[TOC]

---





### _hadoop编码_

#### _hadoop的基本类型_

```java
NullWritable 当<Key,value>中的key或value为空的时候使用
Text 使用UTF8格式存储的文本
LongWritable 长整型
IntWritable 整型数
FloatWritable 浮点型
DoubleWritable 双字节数值
ByteWritable 单字节浮点型
BooleanWritable 标准布尔整型数
数据类型都实现Writable接口,以便用这个类型定义的数据可以被序列化

writable - value
write() 把每个对象序列化到输入流
readField() 把输入流字节反序列化
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
	
```

