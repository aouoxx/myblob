[TOC]

---

### _maptask并行度_

```java
maptask并行度决定map阶段任务处理并发度,进而影响整个job的处理速度
	一个job的map阶段并行度由客户端在提交job时决定的,客户端对map阶段并行度的规划的基本逻辑为:
将待处理数据执行逻辑切片(即按照一个特定切片大小,将待处理数据划分成逻辑上的多个split)，然后每一个split分配一个maptask,并行实例处理。
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
		file1.txt.split2 -- 128~256M
		file1.txt.split3 -- 256~320M
3) FileInputFormat中切片的大小参数配置
	在FileInputFormat中,计算切片大小的逻辑,Math.max(minSize,Math.min(maxSize,blockSize));切片主要由下面几个值来运算决定:
	minsize:默认值 1  配置参数：mapreduce.input.fileinputformat.split.minsize
	maxsize:默认值Long.Maxvalue 配置参数：mapreduce.input.fileinputformat.split.maxsize
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









































