[TOC]

----


### _hive的概述_

**_hive是构建在HDFS上的数据仓库_**

```java
	Hive是基于hadoop的一个数据仓库工具,可以将结构化的数据文件映射为一张数据库表,并提供简单的sql查询功能,可以将sql语句转换为MapReduce任务进行运行。其优点可以通过类SQL语句快速实现简单的MapReduce统计,不必开发专门的MapReduce应用,十分适合数据仓库的统计分析。Hive是一个数据仓库基础工具在Hadoop中用来处理结构化数据。它架构在Hadoop之上,总归为大数据,并使得查询和分析更加方便。
	在Hive中,Hive是SQL的解析引擎,它将SQL语句转译为M/R Job然后在hadoop中执行。Hive的表其实就是HDFS的目录/文件,按表名把文件夹分开。如果是分区表,则分区值是子文件夹,可以直接在M/R job中使用这些数据。


关系数据库里, 表的加载模式是在加载时强制确定的(表的介绍模式是指数据库存储数据的文件格式)
    如果加载数据时,发现加载的数据不符合模式,关系数据库则会拒绝加载数据,这个就叫"写模式",写模式会在数据加载时候对数据模型进行校验的操作.
    hive在加载数据时和关系数据库不同,hive在加载数据时不会对数据进行检查,也不会更改被加载的数据文件,而检查数据格式的操作是在查询操作时执行,
这种模式叫'读时模式'.
    在实际应用中,写模式在加载数据的时候会对列进行索引,对数据进行压缩,因此加载数据的速度很慢,但是当数据加载好了,我们查询数据的时候,速度很快.
    但是当我们的数据是非结构化,存储模式也是未知的时候,关系数据这种场景加麻烦多了,这时候hive就会发挥它的优势


```

### _hive的系统架构_

```sql
hive的系统架构图
	用户接口cli   CLI即shell命令行
	JDBC/ODBC   Hive的java与传统数据库JDBC的方式类似
	WebGUI   通过浏览器访问Hive
	解释器,编译器,优化器完成HQL查询语句从词法分析,语法分析,编译,优化以及查询计划(plan)的生成。生成的查询计划存储在HDFS中,并随后又MapReduce调用执行。
	Hive的数据存储在HDFS中,大部分的查询由MapReduce完成( 包含*的查询,比如select * from table不会生成MR任务)
	Hive将元数据存储在数据库中(metastore),目前只支持mysql，derby. 
	Hive中的元数据包括表的名字,表的列和分区以及其属性,表的属性(是否为外部表等),表的数据所在目录等.
```

##### _metadata组件_

```
 	Hive的Metastore组件是hive元数据集中存放处。Metastor组件包含两部分
```







### _hive的安装_

#### _hive的配置文件_

```xml
<configuration>
	<property>
    	<name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql//host:port/hive?createDatabaseIfNotExist=true</value>
    </property>
    <property>
    	<name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    <property>
    	<name>javax.jdo.option.ConnectionUserName</name>
        <value>hadoop</value>
    </property>
    <property>
    	<name>javax.jdo.option.ConnectionPassword</name>
        <value>mysql</value>
    </property>
    
    <property>
    	<name>hive.metastore.local</name>
        <value>false</value>
    </property>
</configuration>

```

```
	默认情况下,Hive元数据保存在内嵌的Derby数据库中,只能允许一个回话连接,只适合简单的测试。
实际情况下生产环境中不使用,为了支持多用户会话,则需要一个独立的元数据库,使用mysql作为元数据库,Hive内部对Mysql提供了很好的支持
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



#### _复杂数据类型查询_

```sql
 数据类型
 `filter_ads` map<string,array<struct<dsp_id:string,deal_id:string,material_id:string,ad_filter_code:int>>>
 
 select filter_ads from iad_preformat_dev.iad_adx_traffic_monitor_di where d='2018-10-23'
 => {"1231":[{"dsp_id":"123","deal_id":"1231","material_id":"123","ad_filter_code":1231}]}
 
 select filter_ads['1231'] from iad_preformat_dev.iad_adx_traffic_monitor_di where d='2018-10-23'
 =>  [{"dsp_id":"123","deal_id":"1231","material_id":"123","ad_filter_code":1231}]
 
 select filter_ads['1231'][0] from iad_preformat_dev.iad_adx_traffic_monitor_di where d='2018-10-23'
 => {"dsp_id":"123","deal_id":"1231","material_id":"123","ad_filter_code":1231}
 
select filter_ads['1231'][0],filter_ads['1231'][0].dsp_id, filter_ads['1231'][0].deal_id from iad_preformat_dev.iad_adx_traffic_monitor_di where d='2018-10-23'
 => {"dsp_id":"123","deal_id":"1231","material_id":"123","ad_filter_code":1231}	123	1231
 
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
    1) 内部表Load数据有两种方式: a） load data *** b）hdfs dfs -put *** 因为在Metastore文件,即mysql的hive数据库的"SDs"表中,保存着Hive表对应的hdfs路径信息
    2) 每一个Table在hive中都有一个相应的目录存储数据， 内部表在Load数据时,如果使用LOCAL关键字,Hive会把本地文件系统的数据文件复制到Hive的/wareHouse目录。反之,则是将HDFS上的数据文件剪切到/warehourse目录
    3) Hive在Load数据时,并不检查目录中的文件是否符合表所声明的模式。只有通过Select查询返回空值NULL,才能确定不匹配
    4) 内部表drop的时候,表的元数据(mysql中)和数据文件(hdfs中)都会被删除

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



### _hive数据导入_

```sql
> 使用load执行数据导入
  load data [local] inpath 'filepath' [overwrite] into table tableName
  [patition (partcol1=val1,partcol2=val2..)]
  -----------------------------------------------------------------------------
     [local]: 表示从操作系统的目录进行导入，如果不写local表示从HDFS数据目录进行数据导入
      inpath: 表示导入的路径'filepath'文件存在的目录
 [overwrite]: 是否覆盖表中存在的数据
 [partition]: 如果表是分区表,通过partition来指明分区,(partcol1=val1,partcol2=val2..)为分区的条件

hive> load data inpath '/root/inner_table.dat' into table t1;
      移动hdfs中数据到t1表中      
hive> load data local inpath '/root/inner_table.dat' into table t1;
      上传本地数据到hdfs中      
hive> !ls 查询当前linux文件夹下的文件
hive> dfs -ls / 查询当前hdfs文件系统下'/'目录下的文件
```



### _hive表相关操作_

```sql
Hive 创建表加上对字段的描述信息
	create table table_name (column1 int comment '第一列',...); //comment表示字段描述
	create table test_table(id bigint comment '序号',name string comment '姓名');
> 查看表信息
	desc person;
> 查看表分区
	show partitions t1; (查看表有哪些分区)
> 删除表信息
	drop table t1;
	drop table t1 if exits t1;
> 新增字段(新增列信息)
    添加列 alter table t1 add columns(english,int);
	alter table tablename add columns (列名 类型 [comment '注释']); --comment 部分可选
	alter table table_test add columns (data string);
	alter table table_test add columns (
    	order_source string comment '订单来源',
        bank_name string comment '银行行名'
    );
> alter table table_name [partition] add|replace columns(col_name data_type comment xx) [cascade|restricts]
	ddl语句最后添加cascade,否则新增的列在旧的分区中不可见,查询数据时为null,重新刷新数据仍为null

> 修改hive表字段注释
	alter hive 表名 change 列名 新的列名 新的列名类型 comment '列注释'
> 修改hive表名
	alter table table_name rename to new_table_name;
> 修改hive表字段名
	alter table 表名 change 旧字段 新字段 类型;

```
#### _hive insert_


```sql
insert into overwrite local directory '/home/hadoop/ssgao/test_demo.txt'
 row format delimited
 fields terminated by '\t'
 select * from test;

insert into 和 insert overwrite的区别
> insert into 语句
	insert into table account select id,age,name form account_tmp;
> insert overwrite 语句
	insert overwrite table account2 select id,age,name from account_tmp;
> 两者的区别,insert overwrite会覆盖已经存在的数据,先将原始表的数据remove,如果存在分区的情况下,insert overwrite 会只重写当前分区数据, insert into只是简单的插入,不考虑原始表数据,直接追加到表中,最后表中的数据是原始数据和新插入的数据。

insert into ... select
> insert into table2(字段1,字段2) select 字段1,字段2 from table1;
> insert into table2(字段1,字段2) partition=xx select 字段1,字段2 from table1 where partition=xx (插入指定的分区)


```





### _hive函数_

#### _get_json_object_

#### _json_tuple_

| data                                                         |
| ------------------------------------------------------------ |
| {"store":{"fruit":[{"weight":8,"type":"apple"},{"weight":9,"type":"pear"}]，“bicycle”:{"price":19.95,"color":"red"}},"email":"amy@only_for_json_udf_test.net","owner":"amy"} |

```sql
get_json_object(string json_string,string path)
返回值 string
说明 解析json的字符串json_string,返回path指定的内容,如果输入的json字符串无效,那么返回null
select  get_json_object(data,'$.owner') from dual;  --输出结果为amy

json_tuple(jsonstr,k1,k2,...)
参数为一组键 k1,k2...和json字符串,返回值为元组。该方法比get_json_object高效,因为可以在一次调用中输入多个键
select a.timestamp,b.* from log a lateral view json_tuple(a.appevent,'eventid','eventname') b as f1,f2;

ps: get_jsob_object和jsob_tuple 解析的对象为string类型,而非结构类型(如：map,array,struct)等
```

```sql
select body from table_test where d='2018-10-23'
{"1231":[{"dsp_id":"123","deal_id":"1231","material_id":"123","ad_filter_code":1231}]}

-- get_json_object的嵌套使用, 解析数组中的元数据
select get_json_object(body,'$.1231'),get_json_object(get_json_object(body,'$.1231[0]'),'$.dsp_id') ,body from table_test

-- json_tuple和get_json_object 联合使用
select json_tuple(get_json_object(body,'$.adContentList[0]'),'ad_type','location','app') from iad_ods_test.tmp_body_sloth_dms_biddings_di_20181106
```





```
















```