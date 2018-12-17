[TOC]

---



## _GIT的基本使用_

### _git的基本命令_

```java
git中的基本命令
mkdir xx 创建一个空目录，xx表示目录名
git init 把当前目录变成可以管理的git仓库,生成隐藏的.git文件
git add xx 把xx文件添加到暂存区
git commit -m “xx” 提交文件-m后面是注释
git diff xx 查看xx文件修改了那些内容
git log 查看历史记录

git branch 查看当前所有的分支
git checkout master 切换回master分支
git merge dev 在当前的分支上合并dev分支
git branch -d dev 删除dev分支
git branch name 创建分支

git reset --hard HEAD~ n 回退版本
git reset --hard HEAD~ 回退到上一个版本,如果想回到到100版本 git reset --hard HEAD~ 100
 
git remote 查看远程库的信息
git remote -v 查看远程库的详细信息

git push origin master GIT会把master分支推送到远程库对应的远程分支上
```



### _git生成多个key_

```xml
之前只使用一个ssh-key 在github上提交代码,目前需要在添加一个ssh key 在github上提交代码,下面为具体的配置过程:
	1) 第一次使用ssh生成key，默认会在用户~(根目录下)生成id_rsa,id_rsa.pub两个文件，所以需要添加多个ssh key时也会生成对应的私钥和公钥.
		ssh-keygen -t rsa -C "email@xx.com"
	2）生成并添加第二个ssh-key
		ssh-keygen -t rsa -C "email2@xx.com" 注意这里不要一路回车,需要给这个文件起一个名字,比如id_rsa_github 所以相应的也会生成一个id_rsa_github.pud文件.

gaoshuoshuo@HIH-D-15005 MINGW64 ~/.ssh
$ ssh-keygen -t rsa -C "aouo1987@163.com"
Generating public/private rsa key pair.
// 这里输入新的文件名
Enter file in which to save the key (/c/Users/gaoshuoshuo/.ssh/id_rsa): id_rsa_github 
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa_github.
Your public key has been saved in id_rsa_github.pub.
The key fingerprint is:
SHA256:JzFmQLDAe1s0AsZRK8QTA9k/hX3hrCgFE6g88RU1jSE aouo1987@163.com

```



### _git的fork的介绍_

```xml

```

















































