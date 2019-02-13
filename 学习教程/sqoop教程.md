[TOC]

### _sqoop的介绍_

```xml
Sqoop 是一个用于在hadoop和关系数据库服务器之间传输数据的工具。它用于从关系数据库(如mysql，Oracle)导入数据到hadoop hdfs，并从hadoop文件系统导出到关系数据库

sqoop = sql -> hadoop
 导入数据: mysql,oracle 导入数据到hadoop的HDFS,Hive,Hbase等数据存储系统
 导出数据: 从Hadoop的文件系统中导出数据到关系数据库

sqoop导入
	导入工具从RDBMS向HDFS导入单独的表,表中的每一行都被视为HDFS中的记录。
	所有记录都以文本文件的形式存储在文本文件中或作为Avro和Sequence文件中的二进制数据存储。
sqoop导出
	导出工具将一组文件从HDFS导出回到RDBMS。
    给Sqoop输入的文件包含记录,这些记录在表中被称为行。这些被读取并解析成一组记录并用用户指定的分隔符分隔 
```



### _sqoop的安装_

```xml

```







### _sqoop的使用介绍_

```


https://blog.csdn.net/jsjsjs1789/article/details/70093104
```

