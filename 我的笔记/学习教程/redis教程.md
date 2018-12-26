[TOC]

----

### _redis的安装_

####  _redis在linux下安装_

```shell
下载源码，解压缩后编译源码。
$ wget http://download.redis.io/releases/redis-2.8.3.tar.gz
$ tar xzf redis-2.8.3.tar.gz
$ cd redis-2.8.3
$ make
$ make install

当安装完redis之后，就需要进行测试，以下简单做一个测试来验证我们的redis是否安装成功。
b.首先我们启动redis服务，启动和关闭redis服务命令如下：
b.1  src/redis-server       启动redis，加上&表示使redis以后台程序方式运行
b.2 redis-server redis.conf  启动redis
b.3  src/redis-cli shutdown     关闭redis
```

#### _redis服务的启动和关闭_

```shell
按照service的形式来启动redis服务
执行命令 vim /etc/init.d/redis 创建脚本文件
# Date 2015-12-10
# chkconfig: 2345 10 90  
# description: Start and Stop redis   
REDISPORT=6379
EXEC=/root/redis/redis-stable/src/redis-server
REDIS_CLI=/root/redis/redis-stable/src/redis-cli
PIDFILE=/var/run/redis.pid
CONF="/root/redis/redis-stable/redis.conf"
case "$1" in   
        start)   
                if [ -f $PIDFILE ]   
                then   
                        echo "$PIDFILE exists, process is already running or crashed."  
                else  
                        echo "Starting Redis server..."  
                        $EXEC $CONF   
                fi   
                if [ "$?"="0" ]   
                then   
                        echo "Redis is running..."  
                fi   
                ;;   
        stop)   
                if [-f $PIDFILE ]   
                then   
                        echo "$PIDFILE exists, process is not running."  
                else  
                        PID=$(cat $PIDFILE)   
                        echo "Stopping..."  
                       $REDIS_CLI -p $REDISPORT  SHUTDOWN    
                        sleep 2  
                       while [ -x $PIDFILE ]   
                       do  
                                echo "Waiting for Redis to shutdown..."  
                               sleep 1  
                        done   
                        echo "Redis stopped"  
                fi   
                ;;   
        restart|force-reload)   
                ${0} stop   
                ${0} start   
                ;;   
        *)   
               echo "Usage: /etc/init.d/redis {start|stop|restart|force-reload}" >&2  
                exit 1  
esac

> 给文件添加权限,使得脚本可以执行 chmod 755 /etc/init.d/redis
> 以上工作顺利完成并且没有报错,则配置完成
  > service redis start 开启redis服务
  > service redis stop 关闭redis服务
[root@ssgao1987 ~]# service redis start
/etc/init.d/redis: line 1: te: command not found
Starting Redis server...
1601:C 09 Sep 14:14:41.944 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1601:C 09 Sep 14:14:41.944 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1601, just started
1601:C 09 Sep 14:14:41.944 # Configuration loaded
Redis is running...
[root@ssgao1987 ~]# service redis stop
/etc/init.d/redis: line 1: te: command not found
/etc/init.d/redis: line 26: [-f: command not found
cat: /var/run/redis.pid: No such file or directory
Stopping...
Redis stopped
[root@ssgao1987 ~]#   


'配置环境变量'
export REDIS_HOME=/root/redis/redis-stable
export PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER/bin:$ACTIVEMQ_HOME/bin:$REDIS_HOME/src
[root@ssgao1987 src]# redis-cli 
127.0.0.1:6379> 
127.0.0.1:6379> keys
```







### _redis的数据类型_



##### _zset的介绍_

```shell
	Sorted set是set的一个升级版本,它在set的基础上增加了一个顺序属性,这个属性在添加修改元素时候可以指定,每次指定后,zset会自动重新按新的值进行调整顺序.
可以理解为有两列字段的数据表,一类存value, 一列存顺序编号.操作中key理解为zset的名字.
zadd 向名称为key的zset中添加元素member,score用于排序.如果该元素存在,则更新其顺序。(用法: zadd有序集合 顺序编号 元素值)

127.0.0.1:6379> zadd ssgao_zset 1 ssgao
(integer) 1
127.0.0.1:6379> zadd ssgao_zset 2 ssgao_2
(integer) 1
127.0.0.1:6379> 
127.0.0.1:6379> zrange ssgao_zset 0 -1
1) "ssgao"
2) "ssgao_2"


127.0.0.1:6379> zcard ssgao_zset   (zcard 返回集合中元素个数)
(integer) 2
127.0.0.1:6379> zcount ssgao_zset 0 -1  (zcount: 返回集合中score在给定区间的数量)
(integer) 0
127.0.0.1:6379> zcount ssgao_zset 0 1
(integer) 1


zcard 	 查看zset集合中成员的个数
127.0.0.1:6379> zcard ssgao_zset
(integer) 5
zcount   获取zset集合中指定分支之间存在的成员个数
127.0.0.1:6379> zcount ssgao_zset
(error) ERR wrong number of arguments for 'zcount' command
127.0.0.1:6379> 
127.0.0.1:6379> zcount ssgao_zset 2 5  ---数值表示分数的意思
(integer) 0

```























































































