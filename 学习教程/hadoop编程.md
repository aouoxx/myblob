[TOC]

---



### _hdfs的编程_



#### _hdfs的编程_

```java

    @Test
    public void getClient02(){
        Configuration configuration = new Configuration();
        try {
           // 获取hdfs的客户端
           FileSystem fileSystem =
                   FileSystem.get(new URI("hdfs://192.168.0.108:9000"),configuration,"root");
            // 创建一个目录
           fileSystem.mkdirs(new Path("/ssgao_b"));
           // 关闭资源
           fileSystem.close();
        } catch (Exception e) {
            e.printStackTrace();
        } 
    }
    public FileSystem getFileSystem(){
        Configuration configuration = new Configuration();
        try {
            // 获取hdfs的客户端
            FileSystem fileSystem =
                    FileSystem.get(new URI("hdfs://192.168.0.108:9000"),configuration,"root");
            return fileSystem;
        } catch (Exception e) {
            e.printStackTrace();
        } 
        return null;
    }
    /**
     * 释放资源
     * @param fileSystem
     */
    public void releaseResource(FileSystem fileSystem){
        if(fileSystem!=null){
            try {
                fileSystem.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    @Test
    public void putFile(){
        // 获取hdfs的客户端
        FileSystem fileSystem = getFileSystem();
        // 上传文件
        try {
            fileSystem.copyFromLocalFile(new Path("/Users/ssgao/Downloads/14_wordcount案例.avi"),new Path("/ssgao_a/avi.avi"));
        } catch (IOException e) {
            e.printStackTrace();
        }
        //关闭资源
        releaseResource(fileSystem);
    }

    @Test
    public void getFile(){
        FileSystem fileSystem = getFileSystem();
        try {
            /**
             * delSrc 是否删除源文件
             * 源文件路径
             * 目标路径
             * 是否使用原始本地文件系统,是否开启文本校验
             */
            fileSystem.copyToLocalFile(false,new Path("/ssgao_a/avi.avi"),new Path("/Users/ssgao/Downloads/test_test.avi"),false);
        }catch (Exception e){
            e.printStackTrace();
        }
        releaseResource(fileSystem);
    }

    @Test
    public void delFile(){
        FileSystem fileSystem = getFileSystem();
        try {
            // 删除文件路径，参数二 是否递归删除,即递归删除文件夹的数据
            fileSystem.delete(new Path("/ssgao_b"),false);
        }catch (Exception e){
            e.printStackTrace();
        }

        releaseResource(fileSystem);
    }

    @Test
    public void renameFile(){
        FileSystem fileSystem = getFileSystem();
        try {
            // 修改文件名称
            fileSystem.rename(new Path("/ssgao_a/avi.avi"),new Path("/ssgao_a/avi666.avi"));
        } catch (IOException e) {
            e.printStackTrace();
        }
        releaseResource(fileSystem);
    }

    @Test
    public void lsPath() throws IOException {
        FileSystem fileSystem = getFileSystem();
        // 查询文件夹路径下的所有文件
        RemoteIterator<LocatedFileStatus> files = fileSystem.listFiles(new Path("/ssgao_a"),true);
        // 迭代输出信息
        while (files.hasNext()){
            LocatedFileStatus fileStatus =  files.next();
            System.out.println(fileStatus.getBlockSize());
            System.out.println(fileStatus.getPath().getName());
            Arrays.asList(fileStatus.getBlockLocations()).forEach(location-> {
                String[] arrs = new String[0];
                try {
                    arrs = location.getNames();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                System.out.println(JSON.toJSONString(arrs));
            });
        }
        releaseResource(fileSystem);
    }

    @Test
    public void testFile(){
        FileSystem fileSystem = getFileSystem();
        // 文件和文件夹的判断
        try {
            FileStatus[] fileStatuses = fileSystem.listStatus(new Path("/ssgao_a"));
            Arrays.asList(fileStatuses).forEach(fileStatus -> {
                if(fileStatus.isDirectory()){
                    System.out.println("目录: "+fileStatus.getPath().getName());
                }else{
                    System.out.println("文件: "+fileStatus.getPath().getName());
                }
            });
        } catch (IOException e) {
            e.printStackTrace();
        }

        releaseResource(fileSystem);
    }

    @Test
    public void testWrite() throws Exception {
        FileSystem fileSystem = getFileSystem();
        // 获取输入流信息
        FileInputStream fileInputStream = new FileInputStream("/Users/ssgao/Downloads/15_hdfs的wordcount案例.avi");
        // 获取输出流信息, 使用create()方法
        FSDataOutputStream outputStream = fileSystem.create(new Path("/ssgao_c/15_test.avi"));
        // 流的拷贝, 操作完成后会关闭输入和输出流信息
        IOUtils.copyBytes(fileInputStream,outputStream,4096);
        releaseResource(fileSystem);
    }

    @Test
    public void testRead() throws Exception {
        FileSystem fileSystem = getFileSystem();
        // 获取输入流信息
        FSDataInputStream inputStream =  fileSystem.open(new Path("/ssgao_c/15_test.avi"));
        // 获取输出流信息
        FileOutputStream fileOutputStream = new FileOutputStream("/Users/ssgao/Downloads/15_15.avi");
        // 流的拷贝,操作完成后会关闭输入和输出流信息
        IOUtils.copyBytes(inputStream,fileOutputStream,4096);
        releaseResource(fileSystem);
    }

    @Test
    public void getBlockFile() throws Exception {
        FileSystem fileSystem = getFileSystem();
        // 获取输入流
        FSDataInputStream inputStream =  fileSystem.open(new Path("/ssgao_c/15_test.avi"));
        // 获取输出流
        FileOutputStream outputStream = new FileOutputStream("/Users/ssgao/Downloads/15_15.avi.part01");
        // 流的拷贝
        byte[] buf = new byte[1024];
        for(int i=0;i<1024*128;i++){
            inputStream.read(buf);
            outputStream.write(buf);
        }
        // 关闭资源
        IOUtils.closeStream(inputStream);
        IOUtils.closeStream(outputStream);
        releaseResource(fileSystem);
    }

    @Test
    public void getBlockNFile() throws Exception {
        FileSystem fileSystem = getFileSystem();
        // 获取输入流
        FSDataInputStream inputStream =  fileSystem.open(new Path("/ssgao_c/15_test.avi"));
        // 获取输出流
        FileOutputStream outputStream = new FileOutputStream("/Users/ssgao/Downloads/15_15.avi.part01");
        // 流的拷贝,跳过第一块的内容
        inputStream.seek(1024*1024*128);
        IOUtils.copyBytes(inputStream,outputStream,4096);
        // 关闭资源
        IOUtils.closeStream(inputStream);
        IOUtils.closeStream(outputStream);
        releaseResource(fileSystem);
    }
```





### _maptask并行度_

```java
maptask并行度决定map阶段任务处理并发度,进而影响整个job的处理速度
	一个job的map阶段并行度由客户端在提交job时决定的,客户端对map阶段并行度的规划的
基本逻辑为:
将待处理数据执行逻辑切片(即按照一个特定切片大小,将待处理数据划分成逻辑上的多
个split)，然后每一个split分配一个maptask,并行实例处理。
```

#### _FileInputFormat切片机制_


```java
1) 切片定义在InputFormat类中getSplit()方法
2) FileInputFormat中默认的切片机制:
	a）简单地按照文件的内容长度进行切片
	b）切片大小,默认等于block大小
	c）切片时不考虑数据集整体,而是逐个针对每一个文件单独切片
	比如待处理数据有两个文件:
		file1.txt 320M
		file2.txt 10M
	经过FileInputFormat切片机制运算后,形成的切片信息:
		file1.txt.split1 -- 0~128m
		file1.txt.split2 -- 128~256M<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

		- [_maptask并行度_](#maptask并行度)
			- [_FileInputFormat切片机制_](#fileinputformat切片机制)
			- [_map并行度经验_](#map并行度经验)
		- [_reducetask并行度_](#reducetask并行度)
		- [_mapreduce的方法组成_](#mapreduce的方法组成)

		file1.txt.split3 -- 256~320M
3) FileInputFormat中切片的大小参数配置
	在FileInputFormat中,计算切片大小的逻辑,Math.max(minSize,Math.min(maxSize,blockSize));切片主要由下面几个值来运算决定:
	minsize:默认值 1  
  配置参数：mapreduce.input.fileinputformat.split.minsize
	maxsize:默认值Long.Maxvalue
  配置参数：mapreduce.input.fileinputformat.split.maxsize
	blocksize 默认情况下,切片大小=blocksize
	maxsize(切片的最大值)参数如果调得比blocksize小,则会让切片变小,而且就等于配置的这个参数值
	minsize(切片的最小值)参数如果调的比blocksize大,则会让切片变的比blocksize还大
4) 选择并发数的影响因素
	1) 运算节点的硬件配置
	2) 运算任务的类型,CPU密集型还是IO密集型
	3）运算任务的数据量		
```

#### _map并行度经验_

```
1) 如果硬件配置为2*12core + 64G ，map的并行度是大约每个节点20~100个Map,最好每个Map的执行时间为一分钟
2) 如果job的每个map或者reduce task的运行时间都只有30~40秒,那么就减少该job的map或reduce数,每一个task(map|reduce)的setup和加入到调度器中进行调度,这个中间的过程可能都要花费几秒钟,所以如果每个task都非常快跑完了,就会在task开始和结束的时候浪费太多的时间.
3）配置task的JVM重用可以改善该问题.（JVM重用技术不是指同一个Job的两个或两个以上的task可以同时运行在同一个JVM上） mapred.job.reuse.jvm.num.tasks 默认为1,表示一个JVM上最多可以顺序执行的task数目（属于同一个job是1）。也就是说一个task启一个JVM
4）如果input的文件非常的大,比如1TB,可以考虑将hdfs上的每个blocksize设大,比如设置为256MB或512MB

```



### _reducetask并行度_

```java
	reducetask的并行度同样影响整个job的执行并发度和执行效率,但与maptask的并发数有切片数决定不同,reducetask的数量可以手动设置。
	job.setNumReduceTasks(4); //默认值是1,手动设置为4
	如果数据分布不均匀,就可能在reduce节点产生数据倾斜。
ps:
	reducetask数量并不是任意设置,还要考虑业务逻辑需求,有些情况下,需要计算全局汇总结果,就只能有1个reducetask.

```





### _mapreduce的方法组成_

```java
setup() 此方法被MapReduce框架仅且执行一次,在执行Map任务前,进行相关变量或者资源的几种初始化工作.若是将资源初始化工作放在方法map()中,导致mapper任务在解析每一行输入时都会进行资源初始化工作,导致重复,程序运行效率不高。

cleanup() 此方法被MapReduce框架,仅且执行一次,在执行完毕Map任务后,进行相关变量或资源的释放工作.若是将释放资源工作放入方法map()中,也会导致Mapper任务在解析,处理每一行文本后释放资源,而且在下一行文本解析前还要重复初始化,导致反复重复,程序运行效率不高

```



### _输入/出格式化类_

**_输入格式化类InputFormat_**

```java
	描述MR作业的输入规范,主要功能: 输入规范检查(比如输入文件目录的检查),对数据文件进行输入切分和从输入分块中将数据记录逐一读取出来,并转化为Map输入的键值对。
	getSplits()方法返回List<InputSplit>集合,作用是将输入文件在逻辑上划分为多个输入分片,每个分片数据存放在List集合中。
	createRecordReader()方法返回一个RecordReader对象,该对象用来将InputSplit解析成若干个key/value对。MR框架在Map Task执行过程中，会不断调用RecordReader对象中的方法,迭代获取key/value对并交给map()函数处理【此方法在更为具体的子类中实现,如TextInputFormat】
	
```

**_FileInputFormat类_**

```
主要功能是为子类提供统一的getSplits()方法实现。其中最核心的两个算法是:
1) 文件切分算法
	主要用于确定InputSplit的个数以及每个InputSplit对应的数据段。
	FileInputFormat以文件为单位切分成各个InputSplit,对于每个文件,由三个属性值确定其对应的InputSplit的个数。
2) host选择算法
	待InputSplit切分方案确定后,下一步确定每个InputSplit的元数据信息。
	这通常由四部分组成<file,start,length,hosts>,分别表示InputSplit所在的文件,起始位置,字节长度 以及所在的host(节点)列表.其中前三项很容易确定,难点在于host列表的选择方法
	
```





### _mapreduce编程代码_

#### _mapreduce分区_

```java
public class UserEntity implements Writable {
    private Text name;
    private LongWritable count;
	// --getter/setter--
    @Override
    public void write(DataOutput dataOutput) throws IOException {
        name.write(dataOutput);
        count.write(dataOutput);
    }

    @Override
    public void readFields(DataInput dataInput) throws IOException {
        name = new Text();
        name.readFields(dataInput);
        count = new LongWritable();
        count.readFields(dataInput);
    }
}


 */
public class UserMapper extends Mapper<LongWritable,Text,UserEntity,NullWritable> {
    private UserEntity userEntity = new UserEntity();

    /**
     * KeyIn    Mapper的输入数据的key,这里是每行文字的起始位置(0,11,...)
     * ValueIn  Mapper的输入数据的Value,这里是每行文字
     * KeyOut   Mapper的输出数据的key,这里是序列化对象UserEntity
     * ValueOut Mapper的输出数据的Value,这里不返回任何值
     * Writable 接口是一个实现了徐丽华协议的序列化对象
     * 在hadoop中定义了一个结构化对象都要实现Writable接口,使得该结构化对象可以序列化为字节流
     * 字节流也可以反序列化为结构化对象,LongWritable类型,hadoop.io 对Long类型的封装
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // 将每行的数据以空格切分数据,获得每个字段数据 1 135  xxx
        String[] fields = value.toString().split("\t");
        // 赋值userEntity
        userEntity.setName(new Text(fields[0]));
        userEntity.setCount(new LongWritable(Long.valueOf(fields[1])));
        // 将对象序列化
        context.write(userEntity,NullWritable.get());
    }
}

public class UserReduce extends Reducer<UserEntity,NullWritable,UserEntity,NullWritable> {
    /**
     * Reducer需要定义四个输出,输入类型泛型
     * 四个泛型类型分别代表
     * KeyIn    Reducer的输入数据的Key,这里是序列化对象UserEntity
     * ValueIn  Reducer的输入数据的Value,这里是NullWritable
     * KeyOut   Reducer的输出数据的Key，这里是序列化对象UserEntity
     * ValueOut Reducer的输出数据的Value，这里是NullWritable
     */
    @Override
    protected void reduce(UserEntity key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        Long datacount = key.getCount().get()*12;
        key.setCount(new LongWritable(datacount));
        context.write(key,NullWritable.get());
    }
}


public class UserAnalysis {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        // 创建job对象
        Job job = Job.getInstance(new Configuration());
        // 指定程序的入口
        job.setJarByClass(UserAnalysis.class);

        // 指定自定义的Mapper阶段的任务处理类
        job.setMapperClass(UserMapper.class);
        job.setMapOutputKeyClass(UserEntity.class);
        job.setMapOutputValueClass(NullWritable.class);
        // 数据HDFS文件服务器读取数据路径
        FileInputFormat.setInputPaths(job, "/mapper/inputdata.txt");

        // 指定自定义的reducer阶段的任务处理类
        job.setReducerClass(UserReduce.class);
        // 设置输出结构的Key和Value的类型
        job.setOutputKeyClass(UserEntity.class);
        job.setOutputValueClass(NullWritable.class);

        // 设置定义分区的处理类
        job.setPartitionerClass(ProviderPartitioner.class);
        // 默认ReduceTasks的数量为1, 设置为和分区数一致
        job.setNumReduceTasks(2);

        // 执行提交job方法,直到完成,参数true打印进度和详情
        job.waitForCompletion(true);
        System.out.println("==finished==");
    }
}

```





###  _mapreduce排序_

```
排序是Mapreduce框架中最重要的操作之一, MapTask和ReduceTask均会对数据(按照key)进行排序。该操作是属于Hadoop的默认行为。
任何应用程序中的数据均会被排序,而不管逻辑上是否需要,默认排序是按照字典顺序进行排序,且实现该排序的方法是快速排序。

对于Map Task,它会将处理的结果暂时放到一个缓冲区中,当缓冲区使用率达到一定阈值后,再对缓冲区中的数据进行一次排序,并将这些数据写到磁盘上,而当数据处理完毕后,它会对磁盘上所有文件进行一次合并,以将这些文件合并成一个大的有序文件。
```











### _计数器Counter_

```java
File System Counter: MR-job执行依赖的数据来自不同的文件系统,这个group表示job与文件系统交互读写统计
// map从hdfs读取数据,包括源文件内容,split元数据。所以这个值比FileInputFormatCounter.BYTES_READ要略大些。
	> HDFS Number of bytes read = 260407668
// map task向本地磁盘中总共写了多少字节(其实,Reduce端的Merge也会写入本地File)
    > FILE Number of bytes written = 984244425
// reduce从本地文件系统读取数据(map结果保存在本地磁盘)
    > FILE Number of bytes read = 655771325
// 最终结果写入到HDFS
    > HDFS Number of bytes written = 17681802
    
Job Counters MR子任务统计,即map tasks和reduce tasks
	> Launched map tasks = 4 //启用map task的个数
	> Launched reduce tasks = 5 //启用reduce task的个数

MAP-REDUCE FRAMEWORK MR框架计数器
	> Map input records 1152870  // map task从HDFS读取的文件总行数
	> Reduce input groups 579532  // Reduce输入的分组个数,如<hello,{1,1}> <me,1><you,1> 
	// 如果有combiner的话,那么这里的数值就等于map端Combiner运算的最后条数,如果没有,那么就应该等于map输出跳出
	> Combine input records=0  //Combiner输入=map输出
	> Spilled Records=67418820 // spill过程在map和reduce端都会发生,这里统计在总共从内存向磁盘中spill了多少条数据

File Input Format Counters 文件输入格式化计数器
	> Bytes Read=0 // map阶段,各个map task的map方法输入的所有value值字节数之和
File Output Format Counters 文件输出格式化计数器
	> Bytes Written // MR 输出总的字节数 包括 【单词】【空格】【单词个数】以及每行的【换行符】
	
```

#### _自定义计数器_

```java
// 自定义计数器<key,value>的形式
Counter counter = Context.getCounter("查找ssgao","ssgao")；
if(string.contrains("ssgao")){
	counter.increment(1L); //出现一次+1
}
```



### _自定义序列化_

```java
自定义bean对象实现序列化接口(Writable)
自定义bean对象要想序列化传输,必须实现序列化接口,需要注意以下7项:
1) 必须实现Writable接口
2) 反序列化,需要反射调用空参构造函数,所以必须有空参构造
	public FlowBean(){ super();}
3) 重写序列化方法
	public void write(Data)
```












### _提交MR-JOB过程_

```java
	一个标准MR-JOB的执行入口
		// 参数true 表示检查并打印Job和Task的运行状况
		System.out.println(job.waitForCompletion(true)?0:1);
```







### _fileInputFormat_

```java
FileInputFormat 源码解析
 1) 找到我们数据存储的目录
 2) 开始遍历处理(规划切片)目录下的每一个文件
 3) 遍历第一个文件
 	  a) 获取文件大小fs.sizeOff(ss.txt)
      b) 计算切片大小 computeSliteSize(Math.max(minSize,Math.min(maxSize,blockSize)))=blocksize=128M
      c) 默认情况下,切片大小=blocksize
      d) 开始切,形成第一个切片:ss.txt-0~128M 第二个切片:ss.txt-128~256M 第三个切片:ss.txt-256~300M 
          (每次切片时,都要判断切完剩下的部分是否大于块的1.1倍,不大于1.1倍就划分一块且切片)
      e) 将切片信息写入到一个切片规划文件中
      f) 整个切片的核心过程在getSplit()方法中完成
      g) 数据切片只是在逻辑上对输入数据进行分片,并不会再磁盘上将器切分成分片进行存储.InputSplit只记录了分片的元数据信息,比如起始位置,长度以及所在的节点列表等
      h) 注意:block是HDFS物理上存储的数据,切片是对数据逻辑上的划分
  4) 提交切片规划文件到yarn中,yarn上的AppMaster就可以根据切片规划文件计算开启maptask个数     
     
```



**_FileInputFormat切片机制_**

```java
FileInputFormat中默认的切片机制
1) 简单地按照文件的长度进行切片
2) 切片大小,默认等于block大小
3) 切片时不考虑数据集整体,而是逐个针对每一个文件进行单独切片
	比如待处理数据有两个文件
		file1.txt 320M
		file2.txt 10M
	经过FileInputFormat的切片机制运算后,形成的切片信息如下:
		file1.txt.split1--- 0~128
		file1.txt.split2--- 128~256
		file2.txt.split3--- 256~320
	
FileInputFormat切片大小的配置参数
	通过分析源码,在FileInputFormat中,计算切片大小的逻辑:Math.max(minSize,Math.min(maxSize,blockSize));
切片主要由下面几个值来运算决定:
	mapreduce.input.fileinputformat.split.minsize=1 默认值为1
	mapreduce.input.fileinputformat.split.maxsize=Long.MAXValue 默认值为Long.MAXValue
	因此,默认情况下,切片大小为=blocksize
	maxsize(切片最大值): 参数如果调得比blocksize小,则会让切片变小,而且就等于配置的这个参数的值
	minsize(切片最小值): 参数如果调得比blockSize大,则会切片变得比blocksize还大.
              
```

#### _获取切片信息_

```java
// 根据文件类型获取切片信息
FileSplit inputSplit = (FileSplit) context.getInputSplit();
// 获取切片的文件名称
String name = inputSplit.getPath().getName();
```







#### _FileInputFormat的实现类_

```java
MapReduce任务的输入文件一般是存储在HDFS里面。
输入文件的格式包括:基于行的日志文件,二进制格式文件等。这些文件一般会很大,达到数十GB,甚至更大。
MapReduce是FileInputFormat接口来读取这些数据

常见的FileInputFormat的接口实现类包括:
	TextInputFormat,KeyValueInputFormat,NLineInputFormat
CombineTextInputFormat和自定义的InputFormat等。

TextInputFormat
	TextInputFormat是默认的InputFormat,每条记录是一行输入。
	键是LongWritable类型,存储该行在整个文件中起始字节偏移量。值是这行的内容,不包括任何终止符(换行符和回车符)
```







#### _NLineInputFormat_

```java
如果使用NlineInputFormat,代表每个map进程处理的InputSplit不再按block块去划分,而是按NlineInputFormat指定的行数N来划分。
即输入文件的总行数/N=切片数,如果不整除,切片数=相除后的结果+1;

```







### _OutputFormat_

```java

```







### _Join的应用_

```java
ReduceJoin
	原理: Map端的主要工作为来自不同表(文件)的key/value对打标签以区别不同来源的记录。
	然后用连接字段作为key,其余部门和新加的标志作为value,最后进行输出。
reduce端的主要工作: 在reduce端以连接字段作为key,分组已经完成,我们只需要在每一个分组中将那些来源于不同文件的记录(在map阶段已经打标记)分开，最后进行合并就OK了

缺点:
这种方式的缺点很明显就是会造成map和reduce端也就是shuffle阶段出现大量的数据传输,效率很低
```







### _reduceTask工作机制_

```java
设置reduceTask并行度(个数)
 reduceTask的并行度同样影响整个job的执行并发度和执行效率,但与maptask的并发数由切片书决定不同,reduceTask数量的决定可以直接手动设置
//默认值为1,手动设置为4
job.setNumReduceTasks(4);

***注意***
1) reduceTask=0,表示没有reduce阶段,输出文件个数和map个数一致
2) reduceTask 默认值为1,所以输出文件个数为一个
3) 如果数据分布不均匀,就有可能在reduce阶段产生数据倾斜
4) reducetask 数据并不是任意设置,还要考虑业务逻辑需求,在有些情况下,需要计算全局汇总结果,就只能有一个reduceTask
5) 具体多少个reducetask,需要根据集群性能而定
6) 如果分区个数不是1,但是reducetask为1,是不执行分区过程。
   因为在maptask的源码中,执行分区的前提是先判断reduceNum个数是否大于1,不大于1肯定不执行分区。
```









```java
copy阶段
  reduceTask从各个maptask上远程copy一片数据,并针对某一片数据,如果其大小超过一定阈值,则写到磁盘上,否则直接放到内存中。
 
merge阶段
  在远程copy数据的同时,reducetask启动了两个后台线程对内存和磁盘上的文件进行合并,以防止内存使用过多或磁盘上文件过多
  
sort阶段
   按照mapreduce语义,用于编写reduce()函数输入数据是按key进行聚集的一组数据。(即grouping)
   为了将key相同的数据聚在一起,hadoop采用了基于排序的策略。由于各个maptask已经实现对自己的处理结果进行了局部排序
   因此,reducetask只需对所有数据进行一次归并排序即可。
  
reduce阶段
	reduce()函数将计算结果写到HDFS上。
```







### _总结信息_

