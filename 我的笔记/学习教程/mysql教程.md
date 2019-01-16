[TOC]





### _mysql的安装和设置_

```shell

mysql下载
     wget http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.33-linux-glibc2.5-x86_64.tar.gz
解压
    //解压文件并放置到指定的文件目录
    tar -xzvf mysql-5.6.33-linux-glibc2.5-x86_64.tar.gz
    //复制解压后的mysql的文件夹
    cp -r mysql-5.6.33-linux-glibc2.5-x86_64 /usr/local/mysql
添加用户组和用户
    //添加用户组
    groupadd mysql
    //添加用户组mysql到用户组mysql
    useradd -g mysql mysql
安装mysql
    //切换到mysql的文件目录下
    cd /usr/local/mysql
    mkdir ./data/mysql
    chown -R mysql:mysql ./
    ./scripts/mysql_install_db --user=mysql --datadir=/usr/local/mysql/data/mysql
    cp support-files/mysql.server /etc/init.d/mysqld
    chmod 755 /etc/init.d/mysqld
    cp support-files/my-default.cnf /etc/my.cnf
    
 修改启动脚本
     vi /etc/init.d/mysqld
     //修改内容如下
     //basedir = /usr/local/mysql/mysql-5.6.33-linux-glibc2.5-x86_64
     //datadir = /usr/local/mysql/data/mysql
     
启动,查看和关闭服务
    service mysqld start //启动mysql
    service mysqld stop  //关闭mysql
    service mysqld status //查看mysql的状态
```







#### _安装问题记录_

```shell
常见的错误信息
 [root@ssgao mysql_tar]# service mysqld start
 /etc/init.d/mysqld: line 256: my_print_defaults: command not found
 Starting MySQL ERROR! Couldn't find MySQL server (/usr/local/mysql/bin/mysqld_safe)
     
```





#### _mysql 启动命令_

```sql

```

