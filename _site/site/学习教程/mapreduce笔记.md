[TOC]



### _mapreduce教程_

#### _mapreduce的定义_

```xml
mapreduce是一个分布式运算程序编程框架,是用户开发"基于hadoop的数据分析应用"的核心框架
mapreduce核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完成的分布式运算程序,并发运行在一个hadoop集群上
```

#### _mapreduce的优缺点_

***mapreduce易于编程***

```xml
它简单的实现一些接口,就可以完成一个分布式程序,这个分布式程序可以分布到大量廉价的PC机器上运行。
也就是说我们写一个分布式程序,和写一个简单的串行程序是一模一样的,这个特点使得mapreduce编程变得非常流行。
```



***mapreduce的缺点***

> _mapreduce不擅长做实时计算,流式计算,DAG(有向图)计算_

```
实时计算
  mapreduce 无法向mysql一样,在毫秒或者秒级内返回结果
流式计算
  流式计算的输入数据是动态,而Mapreduce的输入数据集是静态的,不能动态变化。
  mapreduce自身的设计特点决定了数据源必须是静态的。
```



### _mapreduce的核心思想_

```xml
Mapper阶段
	1) 用户自定义的Mapper要继承指定的父类
	2) Mapper的输入数据是KV对的形式(KV的类型可自定义)
    3) Mapper中的业务逻辑写在map()方法中
    4) Mapper的输出数据是KV对的形式(KV的类型可自定义)
    5) map()方法(maptask进程)对每一个<K,V>调用一次
Reducer阶段
    1) 用户自定义的Reducer要继承自己的父类
    2) Reducer的输入数据类型对应Mapper的输出数据类型,也是KV
    3) Reducer的业务逻辑写在reduce()方法中
    4) Reducetask进程对每一组相同K的<K,V>组调用一次reduce()方法
Driver阶段
    相当于yarn集群的客户端,用于提交我们整个程序到yarn集群,提交的是封装了mapreduce程序相关运行参数的job对象
```



### _mapreduce编程_

#### _maven依赖_

```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>2.7.7</version>
</dependency>

<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>2.7.7</version>
</dependency>

<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-hdfs</artifactId>
    <version>2.7.7</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.8.2</version>
</dependency>
```





















































### _总结_

