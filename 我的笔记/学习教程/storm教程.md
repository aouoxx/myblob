### _storm教程_

```xml

```

### _storm配置介绍_

```xml
java.library.path
  storm本身依赖的包路径,存在多个时用冒号分隔
  实例
  java.library.path:
  "/usr/local/lib:/opt/local/lib:/usr/lib"

storm.local.dir
    这个目录storm启动的时候可以自动创建,也可以提前创建好
    storm使用的本地文件系统目录'必须存在并且storm进程可读写'

storm.zookeeper.servers
    storm集群对应的zookeeper集群的主机列表
storm.zookeeper.port
    storm集群对应的zookeeper集群的服务端口,zookeeper默认端口为2181
storm.zookeeper.root
    storm的元数据在zookeeper中存储的根目录
storm.zookeeper.session.timeout
    storm客户端连接zookeeper超时时间

storm.cluster.mode
    storm运行模式,集群模式需设置为distributed(分布式的)

nimbus.hosts
    整个storm集群的Nimbus节点 在"storm1.x"版本后弃用了storm
nimbus.seeds
    用于配置主控节点的地址,可以配置多个,在storm1.x版本以后来进行storm HA的配置
    nimbus.seeds: ["mini01", "mini02"]  //nimbus.seeds:空格["node1",空格"node4"]
    
nimbus.thrift.port:6627
    nimbus节点的thrift服务的端口号
nimbus.thrift.threads：64  
    
nimbus.thrift.max_buffer_size：1048576  
   nimbus节点thrift服务的最大缓存字节数
nimbus.childopts："-Xmx1024m"  
    nimbus节点的JVM参数,开启的最大堆栈为1G
nimbus.task.timeout.secs
    判断task存活的心跳超时时间,超过故障时间storm会进行故障转移重新启动这个task
nimbus.supervisor.timeout.secs
    判断supervisor是存活的心跳超时时间
    storm中每个发射出去的消息处理的超时时间,该时间影响到消息的处理,同时在stormUI上杀掉一个topology时的默认时间(kill动作发出后,多长时间才会真正将Topology杀掉)
nimbus.task.launch.secs 
    task启动时的一个特殊超时设置，拓扑启动后,第一次发送心跳前,将用这个值来临时替代心跳
    task第一次启动时需要较长时间的响应,通常这个值会大于task的心跳值




supervisor.slots.ports
    supervisor上能够运行workers的端口列表
supervisor.worker.timeout.secs
    判断worker是否存活的心跳超时时间,超过这个时间storm没有得到响应会进行work重启
supervisor.worker.start.timeout.secs
    supervisor初始心跳超时和task的初始心跳一直

drpc.servers
   drpc服务器列表,以便drpc spout知道和谁通讯
drpc.port
    storm drpc的服务端口
    
ui.port
    storm自带的UI,以Http服务形式支持访问,此处设置该HTTP服务的端口(非root用户端口号,需要大于1024)
ui.childopts
    stormUI 进程的java参数设置 对java进程的约束都可以在此处进行设置,如内存等。

logviewer.port
    此处用于设置logview进程的端口 LogView进程也是为HTTP形式,需要运行在每个storm节点上
logviewer.childopts
    log Viewer进程的参数设置
logview.appender.name
    storm log4j的appender,设置的名字对于文件storm-xxx/logback/cluster.xml中设置的appender,cluster.xml可以控制storm logger的级别
supervisor.solts.ports
    Storm的Solt,最好设置成os核数的整数倍,同时由于storm是基于内存的实时计算
    solt数不要大于每台物理机可运行的solt个数(物理内存-虚拟内存)单个java进程最大占用内存数
worker.childopts
    storm的worker进程的java限制,有效的设置该参数能够在topology异常时进行原因分析
       -Xms1024m -Xmx1024m -XX:+UseConcMarkSweepGC
       -XX: +UseCMSInitiatingOccupancyOnly 
       -XX:CMSInitiatingOccupancyFraction=70 -XX:+HeapDumpOnOutOfMemoryError
       其中xms为单个Java进程最小占用内存数,xmx为最大内存数,设置HeapDumpOnOutOfMemoryError的好处是,当内存使用量操作xmx
       java进程将jvm杀掉同时会生成java_pidxxx.hprof文件
       使用MemoryAnalyzer分析hropf文件将能够分析出内存使用情况,从而进行相应的调整,分析是否有内存溢出等情况
zmp.threads
     storm支持基于ZMQ的消息传递机制,此处对ZMQ的参数设置,建议使用默认值
 
    
storm.messaging.transport
    storm的消息传输机制,使用Netty作为消息传输时设置成backtype.storm.messagin.netty.Context 

storm.messaging.netty.buffer_size netty
    传输的buffer大小,默认为1MB，当Spout发射的消息较大时,此处需要对应调整

storm.messaging.netty.max_wait_ms
    storm.messaging.netty.min_wait_ms
    这几个参数是关于使用Netty作为底层消息传输时,的相关设置,需要重视

storm.scheduler: 全局的任务调度器
storm.cluster.state.store: 指定用于创建 ClusterState 的工厂


topology.debug
   该参数可以在topology中覆盖,表示该topology,是否运行于debug模式,运行与该模式时,storm将激励topology中收发消息等消息信息,线上环境不建议打开
topology.max.spout.pending 
   spout可以缓存的tuple数目
   一个Spout Task中处理pending状态的最大Tuple数量。该配置应用于单个Task,而不是整个Spout或Topology,可以在Topology中进行覆盖。
topology.max.task.paralleism
   每个topology运行时最大的executor数目
topology.workers
   每个topology运行时的worker的默认数目,若在代码中设置,则此选项值被覆盖
topology.ackers
  设置topology中启动的acker任务数。
  Acker保存由spout发送的tuples的记录,并探测tuple何时被完全处理。
  设置为0表示禁用了消息可靠性。  
toplogy.message.timeout.secs
  topology中spout发送消息的最大处理超时时间.
  如果一条消息在该时间窗口内未被成功ack，storm会告知spout这条消息失败。
topology.kryo.register
  注册到Kryo的序列化方案列表
  序列化方案可以是一个类名,或者是com.esotericsoftware.kryo.Serializer的实现类。
```

#### _配置文件注意_

```xml
> storm的配置文件为yaml文件,配置项后面必须更一个空格才能跟配置值
> storm除了storm.yaml配置文件,还有两个配置需要注意
   1> log4j2/cluster.xml文件,其中可以配置storm的日志级别矩阵信息等
   2> 操作系统的配置'通过 ulimit -a 查看',有两项信息需要配置
      open files:当前用户可以打开的文件描述符数
      max user processes:当前用户可以运行的进程数,此参数太小将引起storm的一个错误(java.lang.OutOfMemoryError: unable to create new native thread  )
> 默认情况下storm启动worker进程时,jvm的最大内存是768M,如果需要调大内存可以设置'worker.childopts: "-Xmx2048m"'    
```



#### _配置实例_

```xml
#Storm集群对应的ZooKeeper集群的主机列表
storm.zookeeper.servers:
     - "bops-10-183-93-129"
     - "bops-10-183-93-131"
     - "bops-10-183-93-132"
#Storm集群对应的ZooKeeper集群的服务端口，ZooKeeper默认端口为21818
storm.zookeeper.port: 21818
#Storm使用的本地文件系统目录（必须存在并且Storm进程可读写）
storm.local.dir: /data/hadoop/data5/storm-workdir
#Storm运行模式，集群模式需设置为distributed（分布式的）
storm.cluster.mode: distributed
#storm.local.mode.zmq    true
#Storm的元数据在ZooKeeper中存储的根目录
storm.zookeeper.root: /storm
storm.zookeeper.session.timeout: 60000
#整个Storm集群的Nimbus节点
nimbus.host: bops-10-183-93-129
storm.log.dir: /data/hadoop/data4/storm-logs
#Storm的Slot，最好设置成OS核数的整数倍；
//同时由于Storm是基于内存的实时计算，Slot数不要大于每台物理机可运行Slot个数：（物理内存－虚拟内存）/单个Java进程最大可占用内存数
supervisor.slots.ports:
        - 6700
        - 6701
        - 6702
        - 6703
```



#### _storm的参数介绍_

##### _storm中slot的介绍_

```xml
storm下worker和solt的关系
	worker和slot是1:1的关系, 可以通过调整storm配置参数supervisor.slots.ports来修改slot的数量
storm程序中使用下面语句指定同时处理worker数
	conf.setNumWorkers(5)
worker数量受限于slot数量,一个worker消耗一个slot,当slot全部分配完,就不能再加载新的topology。slot数量实在supervisor.slots.ports中设置,每个端口提供一个slot，这样supervisor数量乘以ports数量,就是storm集群可以使用的worker数量
```























### _storm组成介绍_

#### _并行度介绍_

```xml
Workers(JVMs)
  在一个节点上可以运行一个或多个独立的JVM进程。
  一个Topology可以包含一个或多个worker(并行的跑在不同的machine上),所以worker process就是执行一个topology的子集,并且worker只能对应一个topology
  worker processes的数目可以通过配置文件和代码中配置,worker就是执行进程,所以考虑并发的效果,数量至少应该大于machines的数目

Executors(Threads)
  在一个worker JVM进程中运行着多个Java线程。
  一个executor线程可以执行一个或多个task任务,但一般默认每个executor只执行一个task。一个worker可以包含一个或多个executor,每个component(spout或bolt)至少对应于一个executor,所以可以说executor执行一个component的子集,同时一个executor只能对应于一个component；
一个executor可以执行该component的一个或多个task实例。
executor的数目,component的并发线程数只能在代码中配置(通过setBolt和setSpout的参数)

Tasks(bolt/spout instances)
  Task就具体的处理逻辑对象,每一个Spout和Bolt会被当作很多task在整个集群里面执行。
  topology启动后,一个component(spout或bolt)的task数目是固定不变的,但该component使用的executor线程数是可以动态调整
 <!--*即: 一个executor线程可以执行该compnent的1个或多个task实例，对于一个compnent存在这样的条件, #threads<=#tasks 线程数小于等于task数，当执行多个task实例时executor的数量就减少了 -->
  可以调用TopologyBuilder.setSpout和Topology.setBolt来设置并行度(即设置多少个task的数目)tasks的数目
  可以不配置,默认和executor是1:1(即一个executor线程只运行一个task),也可以通过setNumTasks()配置
```

```java
 TopologyBuilder builder = new TopologyBuilder();
 // 下面代码，线程数大于了任务数,这种情况不正常,导致一个线程(executor)没有任务可以运行
 builder.setSpout("spout", new RandomSentenceSpout(), 5).setNumTasks(4);//executors数目设置为5，即线程数为5，task为4
 builder.setBolt("split", new SplitSentence(), 8).shuffleGrouping("spout");//executors数目设置为8，即线程数为8，task默认为1
 builder.setBolt("count", new WordCount(), 4).fieldsGrouping("spout", new Fields("ming"));   //executors数目设置为4，即线程数为4
 
 Config conf = new Config();
 conf.setDebug(false);
 conf.setNumWorkers(3);//这里是设置Topology的Workers数,slots
 StormSubmitter.submitTopology("word-count", conf, builder.createTopology());
```















































### _总结_































