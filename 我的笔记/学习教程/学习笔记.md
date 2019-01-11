[TOC]

---

### _历史服务器_

#### _历史服务器的配置_

```xml
配置文件mapred-site.xml 

	<property>
		<name>mapreduce.jobhistory.address</name>
   	 	<value>master.ssgao:10020</value>
        <description>数据通信的地址</description>
	</property>
	<property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>master.ssgao:19888</value>
        <description>web页面的地址</description>
	</property>

启动历史服务器的命令
mr-jobhistory-daemon.sh start historyserver

```



#### _配置日志聚集_

```xml
MapReduce是在各个机器上运行的,在运行过程中产生的日志存在于各个机器上,为了能够统一查看各个机器的运行日志,将日志集中存放在HDFS上,这个过程就是日志聚集。Hadoop默认是不启用日志聚集的,
配置yarn-site.xml
	<property>
		<name>yarn.log-aggregation-enable</name>
        <value>true</value>
        <description>日志聚集功能打开</description>
	</property>
	<property>
		<name>yarn.log-aggration.retain-seconds</name>
        <value>604800</value> 单位为秒
        <description>日志保留时间设置为7天</description>
	</property>
```



### _编译hadoop_

```shell


[root@slavea apache-ant-1.9.9]# yum install -y  glibc-headers
[root@slavea apache-ant-1.9.9]# yum install -y  gcc-c++
[root@slavea apache-ant-1.9.9]# yum install -y  make cmake


protobuf的编译安装
[root@slavea protobuf-3.0.0]# ./configure
[root@slavea protobuf-3.0.0]# make
[root@slavea protobuf-3.0.0]# make check
[root@slavea protobuf-3.0.0]# make install
[root@slavea protobuf-3.0.0]# ldconfig
[root@slavea protobuf-3.0.0]# vim /etc/profile
	export LD_LIBRARY_PATH=/root/package/protobuf-3.0.0
	export PATH=PATH:$LD_LIBRARY_PATH
[root@slavea protobuf-3.0.0]# protoc --version
	libprotoc 3.0.0

卸载protoc
	[root@slavea ~]# which protoc
	/usr/local/bin/protoc
	[root@slavea ~]# rm /usr/local/bin/protoc 
	rm：是否删除普通文件 "/usr/local/bin/protoc"？y

解压hadoop源码,执行mvn 命令
	[root@slavea hadoop-2.7.7-src]# mvn package -Pdist.native -DskipTests -Dtar
```







```xml
dataNode3 异常,返回nameNode的时候副本数少了一个, NameNode会异步的复制一份文件

dataNode1 异常,首先会重试连接,如果还是连接不上,重新请求nameNode,重新分配dataNode节点
```



### _HDFS的深入_

#### _namenode的工作机制_

```java
Fsimage: 
	namenode中存储的元数据信息进行序列化以后形成的文件
	fsimage0000000001 (n个零)
edits: 记录对namenode中元数据更新的每一步操作
	edits000000001~edits000000005


请求是否需要checkpoint
	checkpoint触发条件 定时时间到   edits 的数据满了
	
第一阶段 NameNode启动
	1) 第一次启动NameNode格式化后,创建fsimage和edits文件.如果不是第一次启动,直接加载编辑日志和镜像文件到内容
	2) 客户端对元数据进行增删改的请求
	3) NameNode记录操作日志,更新滚动日志
	4) NameNode在内存中对数据进行增删改查
第二阶段 Secondary NameNode工作
	1) Secondary NameNode询问NameNode是否需要checkPoint.直接带回NameNode是否需要检查结果
	2) Secondary NameNode 请求执行checkPoint
	3) NameNode 滚动正在写的edits日志
	4) 将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode
	5) Secondary NameNode加载编辑日志和镜像文件到内存,并合并
	6) 生成新的镜像文件 fsimage.chkpoint
	7) 拷贝fsimage.chkpoint到NameNode
	8) NameNode将fsiamge.chkpoint重新命名成fsimage

NN和2NN工作机制
Fsimage namenode内存中元数据序列化后形成的文件
Edits 记录客户端更新元数据信息的每一步操作(可通过Edits运算出元数据)
namenode启动时, 先滚动edits并生成一个空的edits.inprogress,然后加载edits和fsimage到内存中,此时nameNode内存就持有最新的元数据信息。client开始对nameNode发送元数据的增删改查的请求,这些请求的操作首先会被记录到edits.inprogress中(查询元数据的操作不会被记录在edits中,因为查询操作不会更改元数据信息).如果此时
namenode挂掉,重启后会从edits中读取元数据的信息。然后,namenode会在内存中执行元数据的增删改查的操作。
	由于edits中记录的操作会越来越多,edits文件会越来越大,导致namenode在启动加载edits时会很慢,所以需要对edits和fsimage进行合并(所谓的合并,就是将edits和fsimage加载到内存中,按照edits的操作一步步执行,最终形成新的fsimage)。SecondaryNameNode的作用就是帮助namenode进行edits和fsimage的合并工作。
	secondaryNamde首先会先询问namenode是否需要checkpoint(触发checkpoint需要满足两个条件中任意一个,定时时间到了和edits中数据写满了)。直接带会namenode是否检查结果.secondarynamenode执行checkpoint操作,首先会让namenode滚动edits并生成一个空的edits.inprogress,滚动edits的目的是给edits打个标记,所以所有新的操作都写如edits.inprogress,其他未合并的edits和fsimage会拷贝到secondarynamenode的本地,然后将拷贝的edits和fsimage加载到内存中进行合并,生成fsimage.chkpoint,然后将fsimage.chkpoint拷贝给namenode,重命名为fsimage后替换掉原来的fsimage。namenode在启动时就只需要加载之前未合并的edits和fsimage即可,因为合并过的edits中元数据信息已经被记录在fsimage中。
	

fsimage中没有记录块所对应的datanode: 
	datanote有可能会挂掉,不记录datanode的地址列表,在启动时候datanode将地址汇报给namenode,在内存中维护对应的映射关系,如果挂掉,直接在内存将映射关系修改掉就可以了
	
```



#### _fsimage和edits介绍_

```shell



查看fsimage和edits文件的位置
[root@master current]# 
[root@master current]# pwd
/root/mydata/hadoopname/current
[root@master current]# ls -l
总用量 4168
-rw-r--r--. 1 root root 1048576 12月 19 01:12 edits_0000000000000000001-0000000000000000008
-rw-r--r--. 1 root root      42 12月 20 02:45 edits_0000000000000000009-0000000000000000010
-rw-r--r--. 1 root root      42 12月 20 03:45 edits_0000000000000000011-0000000000000000012
-rw-r--r--. 1 root root 1048576 12月 20 03:45 edits_0000000000000000013-0000000000000000013
-rw-r--r--. 1 root root     728 12月 23 05:30 edits_0000000000000000014-0000000000000000025
-rw-r--r--. 1 root root     128 12月 23 06:30 edits_0000000000000000026-0000000000000000028
-rw-r--r--. 1 root root      42 12月 23 07:30 edits_0000000000000000029-0000000000000000030
-rw-r--r--. 1 root root      42 12月 23 08:30 edits_0000000000000000031-0000000000000000032
-rw-r--r--. 1 root root      42 12月 23 09:30 edits_0000000000000000033-0000000000000000034
-rw-r--r--. 1 root root 1048576 12月 23 09:30 edits_0000000000000000035-0000000000000000035
-rw-r--r--. 1 root root     820 12月 23 13:55 edits_0000000000000000036-0000000000000000049
-rw-r--r--. 1 root root      42 12月 23 14:55 edits_0000000000000000050-0000000000000000051
-rw-r--r--. 1 root root      42 12月 23 15:55 edits_0000000000000000052-0000000000000000053
-rw-r--r--. 1 root root      42 12月 23 16:55 edits_0000000000000000054-0000000000000000055
-rw-r--r--. 1 root root      42 12月 23 17:55 edits_0000000000000000056-0000000000000000057
以上是已经滚动过的edits文件
-rw-r--r--. 1 root root 1048576 12月 23 17:55 edits_inprogress_0000000000000000058 (正在写的edits文件)
-rw-r--r--. 1 root root     856 12月 23 16:55 fsimage_0000000000000000055 镜像
-rw-r--r--. 1 root root      62 12月 23 16:55 fsimage_0000000000000000055.md5 校验
-rw-r--r--. 1 root root     856 12月 23 17:55 fsimage_0000000000000000057
-rw-r--r--. 1 root root      62 12月 23 17:55 fsimage_0000000000000000057.md5
-rw-r--r--. 1 root root       3 12月 23 17:55 seen_txid
-rw-r--r--. 1 root root     206 12月 23 13:02 VERSION

[root@master current]# cat VERSION 
#Sun Dec 23 13:02:41 CST 2018
namespaceID=2083891638
clusterID=CID-44142980-7730-476f-b6af-3b3cfae7c504
cTime=0
storageType=NAME_NODE
blockpoolID=BP-1099844779-192.168.0.108-1545152854774
layoutVersion=-63


[root@master current]# cat seen_txid 
58  --存在的是当前正在写的edits的序号

解码edits
	hdfs oev -p XML -i edits_0000000000000000056-0000000000000000057 -0 editsx.xml
	
手动滚动edits
hdfs dfsadmin -rollEdits 
    [root@master bin]# hdfs dfsadmin -rollEdits  --手动滚动edits
    Successfully rolled edit logs.
    New segment starts at txid 70	
	

hdfs如果确定在下次启动的时候,明确哪些edits是已经合并过了的(合并过的edits不会删除),fsimage最后的数字(fsimage_0000000000000000057 表示fsimage已经合并到了edits58了, edits58到seen_txid之间的号都是没有合并的)
```





#### _namenode故障处理_

##### _checkpoint时间设置_

```xml
1) 通常情况下,SecondaryNameNode每隔一个小时执行一次 (hdfs-default.xml)
	<property>
		<name>dfs.namenode.checkpoint.period</name>
        <value>3600</value>
	</property>
2) 一分钟检查一次操作,当操作次数达到一百万时,SecondaryNamdeNode执行一次
	<property>
		<name>dfs.namenode.checkpoint.txns</name>
        <value>1000000</value>
        <description>操作动作次数</description>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.check.period</name>
        <value>60</value>
        <description>一分钟检查一次操作次数</description>
	</property>
	
```



##### _namenode故障处理_

```xml
NameNode 故障后,采用如下方式恢复数据

方法一:
将SecondaryNameNode中数据拷贝到NameNode存储数据的目录
[root@master current]# pwd
/root/mydata/hadooptmp/dfs/namesecondary/current
[root@master current]# ll
总用量 88
-rw-r--r--. 1 root root  42 12月 20 02:45 edits_0000000000000000009-0000000000000000010
-rw-r--r--. 1 root root  42 12月 20 03:45 edits_0000000000000000011-0000000000000000012
-rw-r--r--. 1 root root 728 12月 23 05:30 edits_0000000000000000014-0000000000000000025
-rw-r--r--. 1 root root 128 12月 23 06:30 edits_0000000000000000026-0000000000000000028
-rw-r--r--. 1 root root  42 12月 23 07:30 edits_0000000000000000029-0000000000000000030
-rw-r--r--. 1 root root  42 12月 23 08:30 edits_0000000000000000031-0000000000000000032
-rw-r--r--. 1 root root  42 12月 23 09:30 edits_0000000000000000033-0000000000000000034
-rw-r--r--. 1 root root 820 12月 23 13:55 edits_0000000000000000036-0000000000000000049
-rw-r--r--. 1 root root  42 12月 23 14:55 edits_0000000000000000050-0000000000000000051
-rw-r--r--. 1 root root  42 12月 23 15:55 edits_0000000000000000052-0000000000000000053
-rw-r--r--. 1 root root  42 12月 23 16:55 edits_0000000000000000054-0000000000000000055
-rw-r--r--. 1 root root  42 12月 23 17:55 edits_0000000000000000056-0000000000000000057
-rw-r--r--. 1 root root  42 12月 23 18:55 edits_0000000000000000058-0000000000000000059
-rw-r--r--. 1 root root  42 12月 23 19:55 edits_0000000000000000060-0000000000000000061
-rw-r--r--. 1 root root  42 12月 23 20:55 edits_0000000000000000062-0000000000000000063
-rw-r--r--. 1 root root  42 12月 23 21:55 edits_0000000000000000064-0000000000000000065
-rw-r--r--. 1 root root  42 12月 23 22:55 edits_0000000000000000066-0000000000000000067
-rw-r--r--. 1 root root 856 12月 23 21:55 fsimage_0000000000000000065
-rw-r--r--. 1 root root  62 12月 23 21:55 fsimage_0000000000000000065.md5


方法二:
 使用-importCheckpoint选项启动NameNode守护进程,从而将SecondaryNameNode中数据拷贝到NameNode目录中
 使用该命令,需要修改配置文件hdfs-site.xml
	<property>
		<name>dfs.namenode.checkpoint.period</name>
        <value>120</value>
	</property>
	<property>
		<name>dfs.namenode.name.dir</name>
        <value>/opt/module/hadoop-2.7.2/data/name</value>
	</property>

如果SeondaryName和NameNode不在一个主机节点上,需要将SecondaryNameNode存储数据的目录拷贝到NameNode存储数据的平级目录,并删除in_use.lock文件。然后执行hdfs namenode -importCheckpoint(等待一会ctrl+c结束掉),启动namenode(hadoop-daemon.sh start namenode)
```



#### _namenode的多目录配置_

```xml
NameNode的本地目录可以配置成多个,且每个目录存放的内容相同,增加了可靠性
<property>
	<name>dfs.namenode.name.dir</name>
    <value>/name/path1,/name/path2</value>
</property>
配置完成了,需要重新启动集群,重新format (hdfs namenode -format)

```



####  _datanode工作机制_

```java
1) 一个数据块在DataNode上以文件形式存储在磁盘上,包括两个文件,一个数据本身,一个是元数据包括数据块的长度,块数据的校验和以及时间戳
2）DataNode启动后向NameNode注册,通过后,周期性(1小时)的向NameNode上报所有的块信息
3）心跳是每3秒一次,心跳返回结果带有NameNode给该DataNode的命令(如,复制块数据到另一台机器或删除某个数据块)。如果超过10分钟没有收到某个DataNode的心跳,则认为该节点不可用。
4）集群运行中可以安全加入和退出一些机器
```



##### _datanode掉线时限_

```xml
	DataNode 进程死亡或者网络故障造成DataNode无法与NameNode通信,NameNode不会立即包该节点判定为死亡,要经过一段时间,这段时间暂时称为超时时长.HDFS默认的超时时长为10分钟+30秒。如果定义超时时间为timeout,则超时时长的计算公式为:
	timeout = 2*dfs.namenode.heartbeat.recheck-interval+10*dfs.heartbeat.interval
	默认的dfs.namenode.heartbeat.recheck-interval大小为5分钟,dfs.heartbeat.interval默认为3秒

在hdfs-site.xml配置文件中的heart.recheck.interval的单位为毫秒,dfs.heartbeat.interval的单位为秒
<property>
	<name>dfs.namenode.heartbeat.recheck-interval</name>
    <value>30000</value>
</property>
<property>
	<name>dfs.heartbeat.interval</name>
    <value>3</value>
</property>

	
```



##### _datanode添加和删除节点_

```xml
datanode的data/current/VERSION 文件中有个datanodeUuid=xxxx-xxx 为该datanode的唯一标识

指定允许连接namenode的主机
hdfs-site.xml
<property>
	<name>dfs.hosts</name>
    <value></value>  --文件的绝对路径
    <description>执行允许连接到namenode的主机。(必须是文件的全路径名)
    dfs.hosts指定的文件可以更好的来管理集群
    </description>
</property>

不允许连接namenode的datanode节点
<property>
	<name>dfs.hosts.exclude</name>
    <value>绝对路径</value>
    <description>下线时数据会复制到其他节点上,需要等到超时时间才认为datanode死掉了</description>
</property>



刷新namenode,无须重启集群
	hdfs dfsadmin -refreshNodes

```





##### _datanode多目录_

```xml
datanode也可以配置成多个目录,每个目录存储的数据不一样,即数据不是副本,配置完成后重启集群，不需要再次格式化(不需要生成新的namenode,所以不用格式化)
hdfs-site.xml
<property>
	<name>dfs.datanode.data.dir</name>
    <value>file://dfs/data1,file://dfs/data2</value>
</property>
```





#### _数据的完整性_

```
1) 当DataNode读取block的时候,它会计算checkSum
2) 如果计算后的checksum 与block创建时值不一样,说明block已经损坏。
3) client读取其他DataNode上的block
4) datanode在其文件创建后周期验证checksum
```



#### _集群之间的数据拷贝_

```xml
采用discp命令实现两个hadoop集群之间的递归数据复制
bin/hadoop distcp  hdfs://hadoop102.900/user/hello.txt hdfs://hadoop103:9000/user/hel.txt
```







### _集群安全模式_

```xml
 	NameNode启动时,首先将映射文件(fsimage)载入内存,并执行编辑日志(edits)中的各项操作(即将fsimage和edits合并)。一旦在内存中成功建立文件系统元数据的映像。则创建一个新的fsimage文件和一个空的编辑日志。此时,NameNode开始监听DataNode请求,但是此刻,NameNode运行在安全模式,即NameNode的文件系统对客户端来说是只读的。
 	系统中的数据块的位置并不是由NameNode来进行维护的,而是以块列表的形式存储在DataNode。在系统的正常操作期间,NameNode会在内存中保留所有块位置的映射信息(即DataNode在启动的时候上报信息到NameNode,NameNode将信息写入到内存中)。在安全模式下,各个DataNode会向NameNode发送最新的块列表信息,NameNode了解到足够多的块位置信息之后,即可高效运行文件系统。

	如果满足"最小副本条件",NameNode会在30秒钟之后就退出安全模式。所谓的最小副本条件指的是在整个文件系统中99.9%的块(不是100%,因为有可能有些块是坏的)满足最小副本级别(默认值: dfs.replication.min=1).在启动一个刚刚格式化的HDFS集群时,因为系统还没有任何块,所以NameNode不会进入安全模式。
	
	安全模式其实就是datanode向namenode汇报,告诉nameNode,当前datanode上管理了哪些块的id,namenode在内存中做块的id和datanode的映射关系,有了这个映射关系后,namenode就可以正常接收我们client的一些请求.

```



#### _安全模式相关的命令_

```shell

集群处理安全模式,不能执行重要操作(写操作),集群启动完成后,自动退出安全模式
1) hdfs dfsadmin-safemode get    -- 功能描述,查看安全模式状态
2) hdfs dfsadmin-safemode enter  -- 功能描述,进入安全模式状态
3) hdfs dfsadmin-safemode leave  -- 功能描述,离开安全模式状态
4) hdfs dfsadmin-safemode wait	 -- 功能描述,等待安全模式离开

```







### _网络拓扑概念_

```
在本地网络中,两个节点被称为"彼此近邻"是什么意思
  在海量数据处理中,其主要限制因素是节点之间数据的传输速率——带宽根稀缺,两个节点之间的带宽作为距离的衡量标准.
  节点距离: 两个节点到达最近的共同祖先的距离总和
```



#### _hdfs的机架感知_

```
低版本的hadoop副本节点选择
	第一个副本在client所处的节点上,如果客户端在集群外,随机选择一个
	第二个副本和第一个副本位于不同机架的随机节点上
	第三个副本和第二副本位于相同的机架,节点随机
	

高版本的副本节点选择
	第一个副本在client所处的节点上,如果客户端在集群外,随机选择一个
	第二个副本和第一个副本位于相同的机架,随机节点
	第三个副本位于不同机架,随机节点
```



### _hadoop存档_

***hdfs存储小文件弊端***

```
 每个文件均按块存储,每个块的元数据存储在NameNode的内存中,因此hadoop存储小文件会非常低效。大量的小文件会耗尽NameNode的大部分内存。
但注意存储小文件所需要的磁盘容量和存储这个文件原始内容所需要的磁盘的空间相比不会增多。
 例如, 一个1MB的文件以大小为128MB的块存储,使用的是1MB的磁盘空间,而不是128MB
 
解决存储小文件的方法:
  Hadoop存档文件或HAR文件,是一个更高效的文件存档工具。它将文件存入HDFS块,在减少NameNode内存使用的同时,允许对文件进行透明的访问。
 (注意,这里的归档并不是将多个小文件合并为一个大文件，存档的文件对内还是一个一个独立文件,对NameNode而言却是一个整体,从而减少了NameNode的内存)
```

***归档管理的操作***

```xml
1）需要启动yarn进程
[root@master sbin]# ./start-yarn.sh 
starting yarn daemons
starting resourcemanager, logging to /root/software/hadoop-2.7.7/logs/yarn-root-resourcemanager-master.ssgao.out
localhost: starting nodemanager, logging to /root/software/hadoop-2.7.7/logs/yarn-root-nodemanager-master.ssgao.out


2) 归档文件
[root@master ~]# hadoop fs -ls hdfs://master.ssgao:9000/ssgao/ssgao-temp
-rw-r--r--   1 root supergroup          9 2019-01-10 18:36 hdfs://master.ssgao:9000/ssgao/ssgao-temp/ssgao_a.txt
-rw-r--r--   1 root supergroup         11 2019-01-10 18:36 hdfs://master.ssgao:9000/ssgao/ssgao-temp/ssgao_b.txt
-rw-r--r--   1 root supergroup         10 2019-01-10 18:36 hdfs://master.ssgao:9000/ssgao/ssgao-temp/ssgao_c.txt
把/ssgao/ssgao-temp目录里面的所有文件归档成一个叫做ssgao-temp.har的归档文件,并把文件文件存储在/ssgao/har路径下
[root@master ~]# hadoop archive -archiveName ssgao-temp.har -p /ssgao/ssgao-temp /ssgao/har
3) 查看归档文件
[root@master ~]# hadoop fs -ls /ssgao/har
drwxr-xr-x   - root supergroup          0 2019-01-10 18:40 /ssgao/har/ssgao-temp.har
[root@master ~]# hadoop fs -ls har:///ssgao/har/ssgao-temp.har
-rw-r--r--   1 root supergroup          9 2019-01-10 18:36 har:///ssgao/har/ssgao-temp.har/ssgao_a.txt
-rw-r--r--   1 root supergroup         11 2019-01-10 18:36 har:///ssgao/har/ssgao-temp.har/ssgao_b.txt
-rw-r--r--   1 root supergroup         10 2019-01-10 18:36 har:///ssgao/har/ssgao-temp.har/ssgao_c.txt

3) 解压归档文件
[root@master ~]# hadoop fs -cp har:///ssgao/har/ssgao-temp.har/*  /ssgao/har
[root@master ~]# hadoop fs -ls /ssgao/har
drwxr-xr-x   - root supergroup          0 2019-01-10 18:43 /ssgao/har/ssgao-temp.har
-rw-r--r--   1 root supergroup          9 2019-01-10 18:44 /ssgao/har/ssgao_a.txt
-rw-r--r--   1 root supergroup         11 2019-01-10 18:44 /ssgao/har/ssgao_b.txt
-rw-r--r--   1 root supergroup         10 2019-01-10 18:44 /ssgao/har/ssgao_c.txt
```



### _快照文件_

> _快照相当于对目录做一个备份。并不对立即复制所有文件,而是指向同一个文件。当写入发生时,才会产生新文件_

```shell
基本语法
1) hdfs dfsadmin -allowSnapshot 路径  开启指定目录的快照功能
2) hdfs dfsadmin -disallowSnapshot 路径 禁用指定目录的快照功能,默认是禁用
3) hdfs dfs -createSnapshot 路径 对目录创建快照
4) hdfs dfs -createSnapshot 路径 名称  指定名称创建快照
5) hdfs dfs -renameSnapShot 路径 旧名称  新名称 重命名快照
6) hdfs lsSnapshottableDir 列出当前用户所有可快照目录
7) hdfs snapshotDiff 路径1  路径2  比较两个快照目录的不同之处
8) hdfs dfs -deleteSnapshot <path> <snapshotName>  删除快照
```



```shell
目录允许快照
[root@master ~]# hdfs dfsadmin  -allowSnapshot /ssgao_a
Allowing snaphot on /ssgao_a succeeded

查看可以进行快照的目录
[root@master ~]# hdfs lsSnapshottableDir
drwxr-xr-x 0 root supergroup 0 2018-12-23 06:24 0 65536 /ssgao_a

对目录创建快照
[root@master ~]# hdfs dfs  -createSnapshot /ssgao_a           
Created snapshot /ssgao_a/.snapshot/s20190110-190607.580
[root@master ~]# hdfs dfs  -renameSnapshot /ssgao_a s20190110-190607.580 ssgao-shot --快照重命名
[root@master ~]# hdfs dfs  -createSnapshot /ssgao_a  ssgao-snapshot  --指定快照名称
Created snapshot /ssgao_a/.snapshot/ssgao-snapshot

[root@master ~]# hdfs dfs  -createSnapshot /ssgao_b  --如何目录不允许快照则无法对其创建快照
createSnapshot: Directory is not a snapshottable directory: /ssgao_b
```

























