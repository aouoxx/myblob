[TOC]

----

### _hive的概述_

**_hive是构建在HDFS上的数据仓库_**

```java
	关系数据库里, 表的加载模式是在加载时强制确定的(表的介绍模式是指数据库存储数据的文件格式)
    如果加载数据时,发现加载的数据不符合模式,关系数据库则会拒绝加载数据,这个就叫"写模式",写模式会在数据加载时候对数据模型进行校验的操作.
    hive在加载数据时和关系数据库不同,hive在加载数据时不会对数据进行检查,也不会更改被加载的数据文件,而检查数据格式的操作是在查询操作时执行,
这种模式叫'读时模式'.
    在实际应用中,写模式在加载数据的时候会对列进行索引,对数据进行压缩,因此加载数据的速度很慢,但是当数据加载好了,我们查询数据的时候,速度很快.
    但是当我们的数据是非结构化,存储模式也是未知的时候,关系数据这种场景加麻烦多了,这时候hive就会发挥它的优势
    
```





### _数据类型_

```sql
基本数据类型
    > tinyint/smallint/int/bigint :整数类型
    > float/double 浮点数类型
    > boolean 布尔类型
    > string 字符串类型
 复杂数据类型
    > Array 数组类型,由一系列相同数据类型的元素组成
      create table student (sid int, sname string, grade array<float>);
      数据:{1,Tom,[80,90,75]}
    > Map 集合类型,包含key->value键值对,可以通过key来访问元素
      create table student1(sid int,sname string,grade map<string,float>);
      数据:{1,Tom,<'语文成绩',132>}        
    > Struct 结构类型, 可以包含不同数据类型的元素。这些元素可以通过"点语法"的方式来得到所需要的元素。
      create table student4( sid int,info struct<name:string,age:int,sex:string> );
      数据:{1, {'Tom',10,'男'}}
 时间类型
    > Date 从hive0.12.0 开始支持
    > Timestramp 从Hive0.8.0 开始支持
    
```

```sql
map,string,struct类型数据解析
> map类型: 表中的数据类型为map<string,string>
  env
  {"uid":12345,"city":"unknow"}
 解析出city
  select env['city'] from table;
  
> string类型
 表中的数据
 exdata
 {"env_networkType":"4G","env_province":"Unknow","env_countryType":"0","childnum":"0","env_ouid":"",
  "filterlist":"[{\"filtertype\":\"8\",\"filtername\":\"小寨\\/省体育场\",\"filtersubtype\":
 解析出env_province,env_countryType
 select * from tablename a 
 lateral view json_tuple(a.exdata,'env_networkType','env_province') as env_networkType,env_province

>struct类型
 struct<isp:string,longitude:double>
 geo
 ----------------------------------------
 {"isp":"电信","longitude":108.948024}
 要解析出isp
 select geo.isp from tablename;
```





### _hive的数据存储_

```xml
>>> hive的数据存储
     1) hive数据存储基于HDFS
     2) 没有专门的数据存储格式
     3) 存储结构主要包括:数据库,文件,表,视图
     4) 可以直接加载文本文件(.txt文件等)
     5) 创建表的时候,指定hive数据的列分隔符,与行分隔符
 
>>> hive的数据模型(表)
    —— Table 内部表
    —— Partition 分区表
    —— External Table 外部表
    —— Bucket Table 桶表

>>> hive中的视图
```

#### _hive内部表_

```sql
>>> 内部表(Table)
    1) 与数据库中的Table在概念是类似
    2) 每一个Table在hive中都有一个相应的目录存储数据
    3) 所有的Table数据(不包括 External Table)都保存在这个目录中
    4) 删除表时,元数据与数据都会被删除
    
  create table t1( sid int, name string, age int) location '/mytable/hive/t2'; //指定表存在的位置 
  -- 指定分隔符
  create table t1( sid int, name string, age int) row format delimited fields terminated by ',';
  create table t4 as select * from sample_data;
  --------------------------------------------------
  create table t5
  --指定文件中列与列之间的分割符
  row format delimited fields terminated by ','  
  as 
  select * from sample_data;
  --------------------------------------------------
```

#### _hive分区表_

```sql
>>> 分区表(Partition)
   1) Partition 对应于数据库中的Partition列的密集索引
   2) 在Hive中,表中的一个Partition对应于表下的一个目录,所有的Partition的数据都存储在对应的目录中.
    
    
> 创建以性别为分区的表
    create table partition_table
    (sid int,sname string,partition by (gender string))
    row format delimited fields terminated by ',';
   -------------------------------------------------------
   insert into table partition_table partition(gender='M')
   select sid,sname,from sample_data where gender='M';
  
```

####  _hive外部表_

```sql
>>> 外部表(External table)
    1) 指向已经在HDFS中存在的数据,可以创建Partition
    2) 它和内部表在元数据的组织上是相同的,而实际数据存储则有较大的差异
    3) 外部表只有一个过程,加载数据和创建表同时完成,并不会移动到数据仓库目录中
       只是与外部数据建立一个链接.当删除一个外部表的时候,仅仅删除该链接.
    
> 通过指定外部文件创建外部表    
    create external table external_student 
    (sid int, sname string,age int)
    row format delimited fields terminated by ','
    location '/input';
```

#### _hive桶表_

```sql
>>> 桶表(Bucket table)
    -桶表是对数据进行哈希取值,然后放到不同文件中存储.
> 创建桶表
  create table bucket_table
  (sid int, sname string, age int)
  clustered by (sname) into 5 buckets;  //对sname进行哈希运算, 创建5个桶表
```



#### _hive视图_

```sql
>>> 视图(view)
    1) 视图是一种虚表,是一个逻辑概念,可以跨越多张表
    2) 视图建立在已有表的基础上,视图赖以建立的这些表称为基表
    3) 视图是不存储数据的
    4) 视图可以简化复杂查询
    
> 创建视图信息
   -- 视图员工信息: 员工号,姓名,月薪,年薪,部门名称
      create view empinfo
      as 
      select e.empno,e.ename,e.sal,e.sal*12 allSary, d.dname
      from emp e, dept d
      where e.deptno=d.no;
```

