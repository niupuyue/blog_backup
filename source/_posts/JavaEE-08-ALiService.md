---
title: JavaEE 08 阿里云服务器的搭建
date: 2019-10-31 23:01:51
tags:
  - JavaEE
  - 阿里云
---

趁着马上双十一，阿里云服务器大降价，将自己一直心心向往的与服务器买了回来，不管怎么说，总算是万里长征第一步。先说一下自己的云主机的配置

<!--more-->

- 地域：华北2(北京)
- 实例规格:ecs.t5-lc 1m2.small
- CPU:1核
- 内存:2G
- 操作系统:CentOS 7.3 64位
- 当前使用带宽:1Mbps
- 实例类型:I/O优化

买完之后，因为我是要进行java开发的，所以需要完成一些基本的配置，而且我还想把自己的博客系统移植到与服务器上，所以，有必要去做一下这个配置，下面就是我的配置步骤。

# SSH登录
有了服务器之后，需要登陆，那么如何登录呢？这里我在网上找到一个比较简单的方法

### 本地生成SSH公钥
首先我们需要确认自己是否已经拥有秘钥，默认情况下，用户的SSH秘钥存储在~/.ssh目录下(不管是MacOS还是Windows系统).进入该目录并列出其中的内容，可以快速确认自己是否拥有秘钥

![Windows下判断是否拥有秘钥](/assets/JavaEE/ecs_01.png)

判断是否有密钥的依据就是寻找一对以id_dsa或id_rsa命名的文件，其中一个带有.pub扩展名。.pub文件就是我们的公钥，另一个则是私钥。如果找不到，或者根本没有.ssh目录，则我们可以运行一下代码完成ssh的创建

```
$ ssh-keygen
Generating public/private rsa key pair.
# 输入 enter 键
Enter file in which to save the key (/Users/laohan/.ssh/id_rsa):
Created directory '/Users/laohan/.ssh'.
# 两次输入密码
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/laohan/.ssh/id_rsa.
Your public key has been saved in /Users/laohan/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:XhI9aeGsVJklGyUTvNu+6ABzOZdZL2+y5aMOVQa+ZvI laohan@bogon
The key's randomart image is:
+---[RSA 2048]----+
|         .O*+    |
|         =+X .   |
|        o O.o o  |
|       . =.= =   |
|      o S *o* .  |
|       = =.*.o   |
|        o ..E +  |
|         . o.*.  |
|         .o.=o.. |
+----[SHA256]-----+
```

首先ssh-keygen会确认密钥的存储位置(默认是.ssh/id_rsa)，然后他会让我们重复输入一个密码两次，如果不想输入，直接点击回车键即可

查看公钥的代码如下所示

![windows查看公钥的代码](/assets/JavaEE/ecs_02.png)

这样我们就获取了自己电脑上的公钥。

接下来我们需要先用默认方法连接服务器，连接完成之后。需要对与服务做一些基本的配置，如下所示

```
$ cd
$ mkdir .ssh && chmod 700 .ssh
$ touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

接下来我们需要将刚才在我们自己电脑上生成的ssh公钥添加到authorized_keys文件中
```
vim .ssh/authorized_keys
```

![阿里云服务器将本地的SSH公钥添加到authorized_keys文件中](/assets/JavaEE/ecs_03.png)

这样的话我们就完成了配置工作，当我们想要在自己的电脑上再次登录远程服务器的时候，可以直接在终端中输入
```
ssh root@ip地址
```
这样的方式完成登录远程服务器。

#### 几个可以优化的部分
在将本地的ssh公钥添加到服务器的authorized_keys文件中后，本机登录远程服务器报错，信息如下所示
```
$ ssh root@47.93.236.188
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:HDjXJvu0VYXWF+SKMZjSGn4FQmg/+w6eV9ljJvIXpx0.
Please contact your system administrator.
Add correct host key in /Users/wangdong/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /Users/wangdong/.ssh/known_hosts:46
ECDSA host key for 47.93.236.188 has changed and you have requested strict checking.
Host key verification failed.
```
如果你也出现了这样的问题，那大致是因为你重置过服务器，不管是重装系统还是格式化磁盘，遇到这样的问题解决也和简单，只需要在自己的本地电脑上运行如下的内容即可
```
ssh-keygen -R ip地址
```
这样问题就能解决了。

> 如果想要多台电脑登录，那么我们可以设置在服务端的authorized_keys中添加新电脑的SSH公钥即可

在我们连接远程服务器的时候，会发现我们还是要输入服务器的ip地址，但是这个ip地址有时候并不是很好记，我们可以使用别的方式,也就是别名的方式

在~/.ssh/文件架下面创建一个新的文件config，名字就叫config，建议创建的时候最好使用git brash的方式，这样更加符合linux的命令方式。如下所示创建了一个config文件
```
touch ~/.ssh/config
```
编辑config文件
```
vim ~/.ssh/config
```

编辑的内容如下所示
```vim 
Host paulniu
HostName 47.93.236.188
Port 22
User root
IdentityFile ~/.ssh/id_rsa
```

每个字段的含义如下所示
Host: 是服务器别名，方便记忆。
User：服务器用户名，如 root
Hostname：服务器 IP 地址
Port：服务器端口，如 22
IdentityFile：ssh 秘钥文件本地位置

之后登陆的时候直接使用如下方式即可
```
ssh paulniu
```
![windows使用别名登陆成功](/assets/JavaEE/ecs_04.png)

MacOS的配置方式和Window的配置方式是一样的

# 配置JDK
配置jdk其实就是配置java环境
首先我是在自己的windows电脑上下载好了jdk文件，然后通过xftp将jdk文件上传到了服务器的根目录下/root/packages/。

我的想法是将Java安装到/usr/local/java/这个文件夹中，然后再去配置jdk的环境变量。

首先我需要将jdk文件复制到/usr/local/java/文件夹中
![将jdk文件复制到java的文件中](/assets/JavaEE/ecs_05.png)

将jdk文件解压
```
tar -zxvf jdk-8u231-linux-x64.tar.gz
```
配置环境变量
通过输入:
```
vim /etc/profile
```
在其中设置java环境变量
```
export JAVA_HOME=/usr/local/java/jdk1.8.0_231
export JRE_HOME=/usr/local/java/jdk1.8.0_231/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$JAVA_HOME:$PATH
```

设置完成之后保存执行
```
source /etc/profile
```
之后再终端中输入
```
java -version
```
输出如下所示内容
![配置完成java环境](/assets/JavaEE/ecs_06.png)

# Tomcat环境搭建
将从tomcat官网下载的tomcat文件复制到/usr/local/tomcat文件夹中
将压缩包解压到当前文件夹下
![解压tomcat文件](/assets/JavaEE/ecs_07.png)
进入/usr/local/tomcat/apache-tomcat-8.5.47目录下的bin文件夹，写入配置文件
```
vim setclasspath.sh
```
在文件的最后写入如下内容
```
export JAVA_HOME=/usr/local/java/jdk1.8.0_231
export JRE_HOME=/usr/local/java/jdk1.8.0_231/jre
```
完成之后推出编辑，执行/usr/local/tomcat/apach-tomcat-8.5.47/bin/.startup.sh，如果出现如下所示的内容，说明tomcat启动成功
![启动tomcat](/assets/JavaEE/ecs_08.png)

这时候如果我们在浏览器中输入我们的服务器地址后面加上8080的端口，就能看到如下所示的内容
![启动tomcat成功](/assets/JavaEE/)