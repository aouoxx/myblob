### _elasticsearch教程_

```xml
ElasticSearch是一个分布式,可扩展,实时的搜索与数据分析引擎

基于Apache Lucene构建的开源搜索引擎
采用Java编写,提供简单易用的Restful API
轻松的横向扩展,可支持PB级别的结构化或非结构化的数据处理

elasticSearch的使用场景
1) 海量数据分析引擎
2) 站内搜索引擎
3) 数据仓库
```





### _elasticsearch安装_



#### _单实例安装_

```xml
1) 下载安装包
[root@master ~]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.5.3.tar.gz
	解压文件内容
[root@master elasticsearch-6.5.3]# tar -xzvf elasticsearch-6.5.3.tar.gz

```











#### _安装和启动问题_

```shell
[root@master bin]# ./elasticsearch
[2019-02-18T15:44:16,125][WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [unknown] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:140) ~[elasticsearch-6.5.3.jar:6.5.3]
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:127) ~[elasticsearch-6.5.3.jar:6.5.3]
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-6.5.3.jar:6.5.3]
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124) ~[elasticsearch-cli-6.5.
        
es不能以root用户进行启动
修改方法:
1) 在运行的时候加上参数: bin/elasticsearch  -Des.insecure.allow.root=true
2) 修改bin/elasticsearch,加上参数ES_JAVA_OPTS属性 ES_JAVA_OPTS="-Des.insecure.allow.root=true"
上面的方法在es5之后就不能使用了,es5以后的解决方法,建议创建一个单独的用户来运行elasticSearch
[root@master bin]# groupadd elasticsearch
[root@master bin]# useradd elasticsearch -g elasticsearch -p elasticsearch
[root@master software]# chown -R elasticsearch:elasticsearch elasticsearch-6.5.3/
[root@master software]# ll
总用量 12
drwxr-xr-x.  8 elasticsearch elasticsearch 4096 12月  7 04:17 elasticsearch-6.5.3
//切换到elasticsearch的用户下,启动elasticsearch
[root@master software]# su elasticsearch
```











































### _总结_