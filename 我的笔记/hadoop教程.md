[TOC]

---

### _hadoop的组成和概述_

```java
hadoop
 它是一个分布式计算+分布式文件系统,前者其实就是MapReduce,后者是HDFS,后者可以独立运行,前者可以独立运行,前者可以选择性使用,也可以不使用

hive
    通俗的说是一个数据仓库,仓库中的数据是被hdfs管理的数据文件,它支持类似sql语句的功能,我们可以通过该语句完成分布式环境的计算功能,
    hive将语句转换为MapReduce,然后交给hadoop执行.
    这里的计算仅限于查找和分布,而不是更新,增加和删除.它的优势是对历史数据进行处理,用时下流程的说法是离线计算,因为它的底层是MapReduce
    MapReduce在实时计算上性能很差.它的做法是把数据文件加载进来作为一个Hive表(或者外部表)让我们觉得我们sql操作的是传统的表

Hbase
    通俗说,hbase的作用类似于数据库,传统数据库管理的是集中的本地数据文件,而Hbase基于HDFS实现对分布式数据文件的管理,比如增删改查.
    也就是说,Hbase只是利用Hadoop的hdfs帮助其管理数据的持久化文件(HFile),它跟MapReduce没任何关系.hbase的优势在于实时计算,所有实时数据都直接存入Hbase中,客户端通过API直接访问Hbase,实现实时计算.由于它使用的是nosql,或者说是列式结构,从而提高了查找性能,使其能运用于大数据场景,这是它和MapReduce的区别.

总结:
    hadoop是hive和habase的基础,hive依赖hadoop,而hbase仅依赖Hadoop的hdfs模块
    hive适用于离线数据的分析,操作的是通用格式(如通用的日志文件),被hadoop管理的数据文件,它支持类sql,比编写MapReduce的java来的更加方便
    hive的定位是数据仓库,存储和分析历史数据
    hbase使用于实时计算,采用列式结构的nosql,操作的是自己生成的特使格式的File,被hadoop管理的数据文件,它的定位是数据库或者叫DBMS
```



### _hadoop的操作命令_

```she
hadoop version   查看版本
hadoop fs  文件系统客户端
hadoop jar 运行jar包
hadoop classpath  查看类路径
hadoop checknative 检查本地库并压缩
hadoop distcp  远程递归拷贝文件
hadoop credentical 认证
hadoop trace 跟踪

```



### _hdfs介绍_

#### _hdfs文件操作命令_

```shell
hadoop hdfs系统的一些常用命令
 操作hdfs系统可以使用hadoop fs也可以使用hdfs dfs,两者效果一样(hadoop dfs 命令已不再建议使用)
常用命令
  hadoop fs (等价于hdfs dfs)文件操作
  1) 显示目录下的所有文件或者文件夹
     >> hadoop fs -ls [uri形式目录]
        示例: hadoop fs -ls / (显示根目录下的所有文件和目录)
    [root@master ~]# hadoop fs -ls  /
    Found 2 items
    drwxr-xr-x   - root supergroup          0 2018-12-08 01:45 /mydata
    -rw-r--r--   1 root supergroup        911 2018-12-07 02:00 /root
    [root@master ~]# 
    [root@master ~]# hadoop fs -ls -R  /
    drwxr-xr-x   - root supergroup          0 2018-12-08 01:45 /mydata
    -rw-r--r--   1 root supergroup        911 2018-12-07 20:03 /mydata/log.log
    -rw-r--r--   1 root supergroup        911 2018-12-08 01:45 /mydata/log.txt
    -rw-r--r--   1 root supergroup        911 2018-12-07 02:00 /root
    [root@master ~]# 
    [root@master ~]# hadoop fs -ls -R  /mydata
    -rw-r--r--   1 root supergroup        911 2018-12-07 20:03 /mydata/log.log
    -rw-r--r--   1 root supergroup        911 2018-12-08 01:45 /mydata/log.txt
  2) cat查看文件内容
     >> hadoop fs -cat URI [uri ...]
        示例: hadoop fs -cat /in/test2.txt
  3) mkdir创建目录
     >> 使用方法 hadoop fs -mkdir /test
        如果创建多级目录,加上-p
        示例: hadoop fs -mkdir -p /a/b/c
    //--使用时需要先创建目录,否则会报错--
    [root@master ~]# hadoop fs -put log.log hdfs://master.ssgao:9000/mydata/log.log
    put: `hdfs://master.ssgao:9000/mydata/log.log': No such file or directory
    [root@master ~]# hadoop fs -mkdir hdfs://master.ssgao:9000/mydata
    [root@master ~]# hadoop fs -put log.log hdfs://master.ssgao:9000/mydata/log.log
  4) rm 删除目录或者文件
     >> 使用方法 hadoop fs -rm [文件路径] 删除文件夹加上 -r
        hadoop fs -rm /test1.txt
        删除文件夹,加上-r
        hadoop fs -rm -r /test
        hadoop fs -rmr dir 删除目录dir
  5) put 复制文件
     >> 将文件复制到hdfs 系统中,也可以从标准输入中读取文件,此时的dst是一个文件
        使用方法: hadoop fs -put <localsrc>  <dst>
        示例: hadoop fs -put /usr/wisedu/temp/test1.txt  /
        从标准输入中读取文件: hadoop fs -put -/in/myword    
  6) cp 复制系统内文件
     >> 使用方法: hadoop -cp URI [URI....] <dest>
        将文件从源路径复制到目标路径,这个命令允许有多个源路径,此时目标路径必须是一个目录
        hadoop fs -cp /in/myword/word
  7) copyFromLocal复制本地文件到hdfs
     >> 使用方法: hadoop fs-copyFromLocal <localsrc> URI
        除了限定源路径是一个本地文件外,和put命令相似
  8) get复制文件到本地系统
     >> 使用方法: hadoop fs -get[-ignorecrc] [-crc] <src> <localdst>
        复制文件到本地文件系统,可用 -ignorecrc选项复制CRC校验失败的文件,使用-crc选项复制文件以及CRC信息
        hadoop fs -get /word  /usr/wisedu/temp/word.txt
        hadoop fs -get /usr/hadoop/file localfile
        hadoop fs -get hdfs://host:port/user/hadoop/file  localfile
  9) copyToLocal复制文件到本地系统
     >> 使用方法: hadoop fs -copyToLocal[-ignorecrc] [-crc] <src> <localdst>
        除了限定目标路径是一个本地文件外,和get命令类似
        hadoop fs -copyToLocal/word /usr/wisedu/temp/word.txt
  10) mv
      >> 将文件从源路径移动到目标路径, 这个命令允许有多个源路径,此时目标路径必须是一个目录,不允许在不同的文件系统间移动文件。 
      使用方法: hadoop fs -mv URI[URI...] <dest>
      示例: hadoop fs -mv /in/test2.txt /test2.txt
  11) du 显示文件大小
     >> 显示目录中所有文件的大小
        使用方法: hadoop fs -du URI[URI...]
        示例: hadoop fs -du /
              hadoop fs -du /user/hadoop/dir1 /user/hadoop/file1 hdfs://host:port/user/hadoop/dir1
        显示当前目录或者文件夹的大小可加选项 -s
     >>> dus 显示文件大小
         hadoop fs -dus <args>
  12) touchz 创建空文件
     >> 使用方法 hadoop fs -touchz URI [URI ...]
        创建一个0字节的空文件
        示例 hadoop fs -touchz /empty.txt
  13) chmod 改变文件权限
  14) chown 改变文件所有者
  15) chgrp 改变文件所在组
  *) tail 将文件尾部1k字节的内容输出到stdout
      hadoop fs -tail URI
      hadoop fs -tail pathname
  *) test 检查信息
      hadoop fs -test -[ezd] URI
      -e 检查文件是否存在,如果存在则返回0
      -z 检查文件是否是0字节,如果是则返回0
      -d 如果路径是一个目录,则返回1, 否则返回0
   *) text 将源文件输出为文本格式,允许的格式是zip和TextRecordInputStream
      hadoop fs -text <src>
   *) 清空回收站
      hadoop fs -expunge

[root@master ~]# hadoop fs -put /root/log.log hdfs://master.ssgao:9000/root
[root@master ~]# hadoop fs -cat hdfs://master.ssgao:9000/root
eth0      Link encap:Ethernet  HWaddr 00:0C:29:47:3F:D4  
[root@master ~]# hadoop fs -ls /
Found 1 items
-rw-r--r--   1 root supergroup        911 2018-12-07 02:00 /root

[root@master ~]# hadoop fs -get /root log.log
[root@master ~]# hadoop fs -get hdfs://master.ssgao:9000/root log1.log
```

```shell
1) -help[cmd] 显示命令的帮助信息
   ./hdfs dfs -help ls
2) -ls(r) 显示当前目录下的所有文件,-R层层循出文件夹
   ./hdfs dfs -ls /log/map
   ./hdfs dfs -lsr /log/ (递归)
   'hadoop 没有当前目录的概念,也没有cd命令,所以在列出文件下的所有文件的时候,需要指定全路径名' 
   [root@master bin]# hdfs dfs -ls /
   Found 1 items
   -rw-r--r--   1 root supergroup        911 2018-12-07 02:00 /root
3) -du(s)显示目录中所有文件大小,或者当只指定一个文件时,显示此文件的大小
   ./hdfs dfs -du /usr/hadoop/dir1 /usr/hadoop/file1 hdfs://host:port/usr/hadoop/dir1
4) -count[-q]显示当前目录下所有文件的大小
5) -mv移动多个文件目录到目标目录
   ./hdfs dfs -mv /usr/hadoop/file1 /usr/hadoop/file2
6) -cp复制多个文件到目录
   ./hdfs dfs -cp /usr/hadoop/file1 /usr/hadoop/file2
   (将文件从源路径复制到目标路径,这个命令有多个源路径,此时目标路径必须是一个目录)
7) -rm(r) 删除文件(夹)
   ./hdfs dfs -rmr /log/map1 (递归删除)
8) -put本地文件复制到hdfs(上传本地文件)
   ./hdfs dfs -put test.txt /log/map/
9) -copyFromLocal 本地文件复制到hdfs
   ./hdfs dfs -copyFromLocal /usr/data/text.txt /log/map1/ 
   (将本地的text.txt 复制到hdfs的/log/map1/下)
10) -moveFromLocal 本地文件移动到hdfs
    ./hdfs dfs -moveFromLocal /usr/data/text.txt /log/map1/
    (将本地的text.txt移动到hdfs的/log/map1/下)
11) -get[-ignoreCrc]复制文件到本地,可以忽略crc校验
    ./hdfs dfs -get /log/map1/* .
    (复制到本地当前目录下)
    ./hdfs dfs -get /log/map1/* /usr/data
    (将hdfs下的/log/map1/下的所有文件全部复制到本地的/usr/data/下)
12) -cat在终端显示文件内容
    ./hdfs dfs -cat /log/map1/part-0000 |head
    (读取hdfs上的/log/map1下的part-0000文件, head参数,代表前10行)
    /hdfs dfs -tail /log/map1/part-0000
    (查看文件的最后一行)

13) -text在终端显示文件内容,将源文件输出为文件格式。
    允许的格式为 zip和TextRecordInputStream
14) -copyToLocal[-ignoreCrc] 复制文件到本地
15) -moveToLocal 移动文件到本地
16) -mkdir 创建文件夹,后跟-p可以创建不存在的父路径
    ./hdfs dfs -mkdir -p /dir1/dir11/dir111
17) -touchz 创建一个空文件
18) -grep 从hdfs上过滤包含某个字符的行内容 (cat 查看文件信息)
    ./hdfs dfs -cat /log/testlog/* | grep 过滤字段
查看HDFS的统计信息
    hdfs dfsadmin -report
```

#### _FileSystem API_

```java
主要命名空间
    org.apache.hadoop.conf.Configuration
    org.apache.hadoop.fs.FileSystem
    org.apache.hadoop.fs.Path
    org.apache.hadoop.fs.FSDataInputStream;
    org.apache.hadoop.fs.FSDataOutputStream;
初始化对象
    Configuration
    FileSystem hdfs
创建文件
    FSDataOutputStream = hdfs.create(path);
    FSDataOutputStream.write(buffer,0,buffer.length)
创建文件夹
    hdfs.mkdirs(Path)
读文件
    FSDataInputStream = hdfs.open(path);
    FSDataInputStream.read(buffer)
写文件
    FSDataOutputStream = hdfs.append(path)
    FSDataOutputStream.write(buffer,0,buffer.length);
删除文件
    FileSystem.delete(path);
列出文件夹内容
    FileSystem.listStatus();
重命名文件
    FileSystem.rename(Path,Path);
```

``` java
实例化FileSystem对象,通过FileSystem类的get方法,这里要传入一个java.net.URL和一个配置Configuration。
String path = "xxx";
COnfiguration conf = new Configuration();
FileSystem fs = FileSystem.get(URI.create(path),conf);
InputStream in = null;
try{
    in = fs.open(new Path(uri));
    /**
     * 调用IOUtils类的copyBytes将hdfs数据流拷贝到标准输出流System.out中
     * copyBytes前两个参数好理解,一个输入,一个输出,第三个是缓存大小,第四个指定拷贝完毕后是否关闭流
     */
    IOUtils.copy(in,System.out,4096,false);
}
```



###  _hadoop配置文件_

#### _core-site.xml_

```xml
修改core-site.xml 配置内容如下:
    <configuration>
       // 指定HDFS老大(namenode)的通信地址
     <property>
       <name> fs.defaultFS </name>
       <value> hdfs://192.168.1.100:9000 </value>
       <description> 192.168.1.100为服务器IP地址,其实也可以使用主机名 </description>
     </property>
     <property>
         <name> io.file.buffer.size </name>
         <value> 131072</value>
         <description>该属性值单位为KB,131072即为默认的64M</description>
     </property>
   </configuration>

打开回收站的配置
   删除文件时,其实时放入回收站
   回收站里面的文件可以快速恢复
   可以设置一个时间阈值,当回收站里面存放的文件超过这个阈值,就被彻底删除,并且释放占用的数据块
   <property>
       <name>fs.frash.interval</name>
       <value>10080</value> // 保存的时间
       <description>启用回收站的功能,重启集群 </description>
	</property>
```

#### _hdfs-site.xml_

```xml
配置nameNode
 <configuration>
     /** 分片数量,伪分布式将其配置为1即可*/
     <property>
         <name> dfs.replication </name>
         <value> 1</value>
     </property>
     /** 命名空间和事物在本地文件系统永久存储的路径*/
     <property>
         <name> dfs.namenode.name.dir</name>
         <value> file:/usr/local/hadoop/tmp/namenode</value>
     </property>
     /** datanode1,datanode2分别对应DataNode所在服务器主机名*/
     <property>
         <name>dfs.namenode.hosts</name>
         <value>datanode1,datanode2</value>
     </property>
     /** 大文件系统HDFS块大小为256M,默认值为64M*/
     <property>
         <name>dfs.blocksize</name>
         <value>268435456</value>
     </property>
     /** 更过的NameNode服务器线程处理来自DataNode的RPCS*/
     <property>
         <name>dfs.namenode.handler.count</name>
         <value>100</value>
     </property>
 </configuration>
```

```xml
dfs.namenode.name.dir
    namenode存放fsimage的目录
dfs.datanode.data.dir
    datanode存放数据块文件的目录
dfs.replication
    数据副本数
dfs.blocksize
    文件block块大小
```

#### _yarn-site.xml_

```xml
<configuration>
	<property>
    	<name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
        <!--reduce 获取数据的方式 -->
    </property>
    <property>
    	<name>yarn.resourcemanager.hostname</name>
        <value>master.ssgao</value>
        <!-- 指定yarn的ResourceManager的地址-->
    </property>
</configuration>
```



```xml
yarn.scheduler.minimum-allocation-mb (默认:1024)
yarn.scheduler.maximum-allocation-mb (默认：8192)
	说明:单个容器可申请的最小与最大内存,应用在运行申请内存时不能超过最大值,小于最小值。这两个值一经设定不能动态改变(即: 不能在应用程序运行时改变)
yarn.scheduler.minimum-allocation-vcore
yarn.scheduler.maximum-allocation-vcore
	说明:单个可申请的最小，最大虚拟CPU个数。比如设置为1和4,则运行MapReduce作业时,每个task最少可申请1个虚拟CPU,最多可申请4个虚拟CPU
yarn.nodemanager.resource.memory-mb (默认：8G)
yarn.nodemanager.vmem-pmem-ratio (默认: 2.1)
	说明:每个节点可用的最大内存,RM中的两个值不应该超过此值。此数值可用于计算container最大数目,即此用此值除以RM中的最小容器内存.虚拟内存率是占task所有内存的百分比,默认值为2.1倍;注意第一个参数是不可以修改的,一旦设置,整个运行过程中不可以动态修改,且该值的默认大小是8G,即使计算机内存不足8G也会按照8G内存来使用


```

##### _内存CPU配置_

```xml
内存的配置
yarn.nodemanager.resource.memory-mb默认值为-1
	表示yarn的nodemanager占总内存的80%。也就是说加入yarn的机器内存为64G,除去非yarn进程需要20%内存,我们nodemanager的内存大概有64*0.8~51G.
    例如: 64GB的机器内存,我们有51G的内存可以用于NodeManager分配,直接将yarn.nodemanager.resource.memory-mb值设置48G(一般container容器的最小内存设置4GB,最大设置为16G)这样当前的NodeManager节点下,我们最多可以有12个container,最少可以有3个container

cpu配置
    此处的CPU指的虚拟CPU(cpu virtual core)，之所以产生虚拟CPU(cpu core)这一概念,因为物理CPU的处理能力的差异,为了平台这种差异,引入这个概念
yarn.nodemanager.resource.cpu-vcores 默认配置为-1
	表示能够分配给container的cpu核数,代表值为8个虚拟cpu,推荐该值设置为和物理CPU的核数数量相同,若不够,则需要调下该值。
yarn.scheduler.minimum-allocation-vcores的默认值为1
	表示每个container容器在处理任务的时候可申请的最小CPU的个数为1个
yarn.scheduler.maximum-allocation-vcores的默认值为1
	表示每个Container容器在处理任务的时候可申请的最大CPU的个数为4个
```

#### _mapred-site.xml_

```xml
mapreduce.job.name 作业名称
mapreduce.job.priority 作业优先级
mapreduce.job.queuename 作业提交到的队列
mapreduce.map.cpu.vcores
mapreduce.reduce.cpu.vcores
	说明: 给Map/reduce的task需要的虚拟cpu的个数
AM 内存配置相关参数,此处以MapReduce为例进行说明,这两个值都是AM特性,应在mapred-site.xml中配置)
mapreduce.map.memory.mb
mapreduce.reduce.memory.mb
	说明: 这两个参数指定用于mapreduce的两个任务(Map和Reduce task)的内存大小,其值应该在RM中的最大,最小container之间,一般reduce应该是map的2倍。这两个值可以在应用启动的时候通过参数改变。
mapreduce.map.java.opts
mapreduce.reduce.java.opts
	说明: 这两个参数主要是为需要运行JVM程序(java,scala等)准备的,通过这两个设置可以向JVM中传递参数，-Xmx,-Xms等。此数值大小,应该在AM中的map.mb和reduce.mb之间。
```

```xml
	hadoop2以上版本中,map和reduce task是运行在container中的。mapreduece.{map|reduce}.memeory.mb 被yarn用来设置container的内存大小.如果container的内存超限,会被yarn杀死，在container中,为了执行map和reduce task，yarn会在container中启动一个jvm来执行task任务. mapreduce.{map|reduce}.java.opts用来设置container启动的jvm相关参数,通过设置XMx来设置map或reduce task的最大内存。
    理论上,{map|reduce}.java.opts设置的最大堆内存要比{map|reduce}.memory.mb小,一般设置为0.75倍的memory.mb即可。因为yarn container这种模式下,JVM进程跑在container中,需要为java code等非JVM的内存使用预留一些空间。
运行中的设置方法例如: xml中也可以设置 hadoop jar -Dmapreduce.reduce.memory.mb=4096 -Dmapreduce.map.java.opts=-Xmx3276
```



#### _配置案例分析_

##### _虚拟内存不足_

```xml
Container [pid=17879,containerID=container_1544560864880_0001_02_000001] is running beyond virtual memory limits. Current usage: 88.1 MB of 256 MB physical memory used; 2.8 GB of 1 GB virtual memory used. Killing container.
yarn-site.xml的配置文件
<configuration>
        ...
        <property>
                <name>yarn.scheduler.minimum-allocation-mb</name>
                <value>256</value>
                <description>container可以申请的最小内存</description>
        </property>
        <property>
                <name>yarn.scheduler.maximum-allocation-mb</name>
                <value>1024</value>
                <description>container可以申请的最大内存</description>
        </property>
        <property>
                <name>mapreduce.map.memory.mb</name>
                <value>128</value>
                <description>map任务申请的内存</description>
        </property>
        <property>
                <name>mapreduce.reduce.memory.mb</name>
                <value>128</value>
                <description>reduce任务申请的内存</description>
        </property>
        <property>
                <name>yarn.nodemanager.resource.memory-mb</name>
                <value>1500</value>
                <description>每个节点可用的最大内存</description>
        </property>
        <property>
                <name>yarn.nodemanager.vmem-pmem-ratio</name>
                <value>4</value>
        </property>
        <property>
                <name>yarn.nodemanager.resource.cpu-vcores</name>
                <value>1</value>
        </property>
        <property>
                <name>yarn.app.mapreduce.am.resource.mb</name>
                <value>200</value>
        </property>
</configuration>
```

### _yarn的介绍_

![yarn](E:\note_workspace\myblob\mypic\yarn.png)

#### _yarn的构成_

```
ResourceManager
    一个cluster只有一个,负责资源调度,资源分配等工作
NodeManager
    运行DataNode节点,负责启动Application和对资源的管理
JobHistoryServer
    负责查询job运行进度以及元数据管理
Containers
    container通过ResourceManager分配,包括容器的CPU，内存等资源
ApplicationMaster
    ResourceManager将任务给ApplicationMaster，然后ApplicationMaster再将任务给NodeManager。每个Application只有一个ApplicationMaster,运行在NodeManager节点,ApplicationMaster是由ResourceManager指派的
job: 需要执行的一个工作单元,包括输入数据,MapReduce程序和配置信息.job可以叫做Application
task: 一个具体做Mapper或reducer的独立的工作单元.ta

```

##### _ApplicationMaster_

```
task运行在NodeManager的Container中
Client: 一个提交给ResourceManager的一个Application程序
用户提交的每个应用程序均包含一个AM,主要功能包括
    与RM调度器协商以获取资源(用container表示)
    将得到的任务进一步分配给内部的任务
    与NM通信以启动/停止任务
    监控所有任务运行状态,并在任务运行失败时重新为任务申请资源以重启任务
```



#### _yarn的执行流程_

```
当用户向yarn中提交一个应用程序后,yarn将分为两个阶段运行
    第一个阶段:启动ApplicationMaster
    第二个阶段:由ApplicationMaster创建应用程序,为它申请资源,并监控它的整个运行过程,直到运行完成.
yarn的执行流程
    1>client向yarn提交应用程序,其中包括ApplicationMaster程序,启动ApplicationMaster的命令,用户程序等
    2>RM为该应用程序分配第一个Container,并与对应的NodeManager通信,要求它在这个container中启动应用程序的applicationMaster
    3>ApplicationMaster首先向ResourceManager注册,这样用户可以直接通过ResourceManager查询应用程序的运行状态,然后它将未各个人物申请资源,并监控它的运行状态,直到运行结束
    4>ApplicationMaster采用轮询的方式通过RPC协议向ResourceManager申请和领取资源
    5>一般ApplicationMaster申请到资源后边与对应的NodeManager通信,要求它启动任务
    6>ModeManager为任务设置好运行环境(包括环境变量,jar包,二进制程序等)后,将任务启动命令写到一个脚本中,并通过运行该脚本启动任务
    7>各个任务通过某个RPC协议向ApplicationMaster汇报自己的状态和进度,以让ApplicationMaster随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务。在应用程序运行过程中,用户可以随时通过RPC向ApplicationMaster查询应用程序当前的运行状态
    8>应用程序运行完成后,ApplicationMaster向ResourceManager注销关闭自己。
程序运行时,只有ApplicationMaster知道运行信息,rm和nodemanager都不知道,yarn只负责资源的分配.从这一点看出yarn和mr程序是解耦的
maptask执行完毕后,相应的资源会被回收,那之后启动的reduce是如何获取maptask生成的数据.
maptask虽然不存在了,但是有文件,它们被nodemanager管理,reduce可以找nodemanager要,我们在搭建环境时配置过一个参数mapreduced_shuffle就是配合管理这些文件
```

### _hadoop编码_

#### _hadoop的基本类型_

```java
NullWritable 当<Key,value>中的key或value为空的时候使用
	序列化长度为0，不读不写,占位符, 不可以修改,即不能调用其set()方法
	单例模式NullWritable.get()获取实例
	用在MapReduce中,将键/值设置为NullWritable
	在sequencefile中,可以用作sequencefile的键
Text 使用UTF8格式存储的文本
LongWritable 长整型
IntWritable 整型数
FloatWritable 浮点型
DoubleWritable 双字节数值
ByteWritable 二级制数据数组的封装
BooleanWritable 标准布尔整型数
数据类型都实现Writable接口,以便用这个类型定义的数据可以被序列化

ObjectWritable和GenericWritable
ObjectWritable 对java基本类型的通用封装,用于RPC中对方法参数和返回类型进行封装和解封
GenericWritable 一个字段包含多种类型时使用,只写封装类型的名称,通过类型引用,静态类型的数组,加入位置索引提高性能,如sequencefile的值包含多种类型.
    
Writable集合类
	ArrayWritable,TwoDArrayWritable 数组和二维数据
	ArrayPrimitiveWritable Java基本数据类型的封装
	EnumMapWritable 集合的枚举类型
	MapWritable,SortedMapWritable分别实现了java.util.Map<Writable,Writable>和java.util.Map<WritableComparable,Writable>

writable - value
write() 把每个对象序列化到输入流
readField() 把输入流字节反序列化
```

```java
// 使用GenericWritable时,只需要继承它,并通过重写getTypes方法指定那些类型需要支持即可
class MyWritable extends GenericWritable{
    MyWritable(Writable writable){
        set(writable);
    }
    public static Class<? extends Writable>[] CLASSES=null;
    static{
        CLASSES = (Class<? extends Writable>[]) new Class[]{Text.class};
    }
    @Override
    protected Class<? extends Writable>[] getTypes(){
        return CLASSES;
    }
    
}
```



#### _hadoop序列化_

```java
hadoop自身的序列化存储格式就是实现了writable接口的类,它只实现了两点 压缩和快速。但是不容易扩展,也不跨语言.
hadoop实现了Writable接口,Writable接口定义了两个方法:
	1) 将数据写入到二进制流中
	2) 从二进制数据流中读取数据
	public interface Writable{
		void write(java.io.DataOutput p1) throws java.io.IOException;
		void readFields(java.io.DataInput p1) throws java.io.IOException;
	}
	

public class SpendBean implements Writable{
    private Text userName;
    private IntWritable money;
    public  SpendBean(Text userName, IntWritable money){
        this.userName = userName;
        this.money = money;
    }
    /** 反序列化时必须有一个空参构造方法*/
    public SpendBean(){}
    /** 序列化代码*/
    public void write(DataOutput out) throws IOException{
        userName.write(out);
        money.write(out);
    }
    /** 反序列化代码*/
    public void readFields(DataInput in) throws IOException{
        userName = new Text();
        userName=readFields(in);
        money = new IntWritable();
        money=readFields(in);
    }
 	// getter/setter...  
}

public class MyWritable implements Writable{
    private int counter;
    private long timestamp;  
    public void write(DataOutput out) throws IOException{
        out.writeInt(counter)；
        out.writeLong(timestamp);
    }
    public void readFields(DataInput in) throws IOException{
        counter = in.readInt();
        timestamp = in.readLong();
    }  
}

```



### _MapReduce的使用_

#### _mapreduce的运行原理_

```xml

```



#### _mapreduce的相关介绍_

```java
	进行mapreduce计算的时候，输出一般是一个文件夹,而且该文件夹是不能被覆盖的, 文件夹不能被覆盖在MR程序对应的job提交的时候就进行了校验,mapreduce之所以这样设计主要是保证数据可靠性,因为如果输出目录存在,reduce就搞不清楚到底是要追加还是覆盖,不管是追加和覆盖操作都有可能导致最终结果出问题，mapreduce是做海量数据计算,一个生产计算的成本很高,一个job有可能需要几个小时,因此一切影响错误的情况mapreduce是零容忍的.
	mapreduce还有一个inputFormat和outputFormat,在编写map函数时候发现map方法的参数文件行数据,没有牵涉到InputFormat,因为这些事情在new Path()的时候mapreduce计算框架已经做好了,而OutputFormat也是reduce做好了,我们使用什么样的输入文件,就要调用什么样的InputFormat,InputFormat是和我们输入的文件类型相关的,mapreduce中常用的InputFormat有FileInputFormat普通文本文件,SequenceFileInputFormat是指hadoop序列化文件,另外还有KeyValueTextInputFormat.OutputFormat是我们最终存储到hdfs系统上的文件格式，这个可以根据我们的需要进行定义，hadoop支持很多文件格式
	
```













































































