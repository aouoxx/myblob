[TOC]

----

## _虚拟机的安装_





### _克隆虚拟机_

```xml
vmware克隆虚拟机,修改修改网卡
	vi /etc/udev/rules.d/70-presistent-net.rules;
	# PCI device 0x8086:0x100f (e1000)
	SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:0c:29:00:0a:69", 		ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
	[root@slavea ~]# 

修改设置克隆机器的eth0的静态ip
	[root@slavea ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0 
    DEVICE=eth0
    HWADDR=00:0c:29:00:0a:69 ---(这个地址注意修改 和上面ATTR{address}一致)
    TYPE=Ethernet
    UUID=35e364f9-403c-4929-bbc3-0db3d31a763d
    ONBOOT=yes
    NM_CONTROLLED=yes
    BOOTPROTO=static
    IPADDR=192.168.0.120
    NETMASK=255.255.255.0
    GATEWAY=192.168.0.1
修改虚拟机的hostname
	 linux查看主机名
		[root@master ~]# hostname
		master.ssgao
	 修改network文件
		[root@master ~]# vim /etc/sysconfig/network
		NETWORKING=yes
		HOSTNAME=slave.ssgao
	 修改hosts文件
		[root@master ~]# vim /etc/hosts
		127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
		::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
		192.168.0.108 master.ssgao
		192.168.0.120 slavea.ssgao
	 reboot重启机器生效
```





## _vim编辑器_

```shell
vim 编辑器

单个文件选择复制
	在命令行模式下,输入字段v(小写),便可以进入按字符选择模式,通过h,i,j,k键移动光标选择要进行复制的字符串.
	> 选择完成后按yy进行复制
	> 将光标移动要进行粘贴的地方(比如放到内容的最后),按下p执行粘贴

多个文件复制粘贴
	> 用vim打开一个文本1.txt
	> 在普通模式下输入 :sp 横向切分一个窗口, 或:vsp纵向切分一个窗口
	> 在普通模式下输入 :e /etc/2.txt 在其中一个窗口打开另一个文件
	> 切换文件进行复制这时和单个文件选择复制的操作一样,v进入字符选择模式,h,i,j,k移动光标选择复制的字符串
	 	按yy进行复制， 按p进行粘贴
	> 切换文件 在普通模式下ctrl+w 在按一下 w,完成窗口之间的切换

```





### _linux的常用文件_

#### _linux账户和密码文件_

```shell
linux 系统中,所有用户包括系统管理员的账户和密码都可以在/etc/passwd和/etc/shadow者两个文件中找到
/etc/passwd只有系统管理员才可以修改,其他用户可以查看
/etc/shadow其他用户看不了
```







## _linux的常用命令_



#### _用户管理命令_

```shell
useradd 添加用户
adduser 添加用户
userdel 删除用户
passwd  为用户设置密码
usermod 修改用户命令,可以通过usermod来修改登录名,用户的家目录等等


语法：useradd [options] username
-d 目录       指定用户主目录,(默认是在/home目录下创建和用户名一样的目录)
-g 用户组     指定用户所属的用户组(主组）
-G 用户组     指定用户所属的附加组(这些组必需事先已经增加过了或者是系统中已经存在)
-s Shell     指定用户的登录Shell
-u UID       指定用户的用户号，如果同时有-o选项，则可以重复使用其他用户的标识号
-c 描述       指定一段注释性描述
-m           使用者目录若不存在则自动建立(默认选项)



```

#### _用户组管理命令_

```shell
groupadd 添加用户组
groupdel 删除用户组
groupmod 修改用户组信息
groups 显示用户所属的用户组
newgrp 切换到相应用户组
```









## _shell编程_