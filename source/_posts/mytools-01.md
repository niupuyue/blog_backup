---
title: 使用Vultr搭建SSR梯子
date: 2019-10-08 15:17:58
tags:
  - SSR
---

最近一段时间，因为各种原因导致我的“梯子”暂时无法访问外面的世界。外面的世界很精彩，外面的世界很无奈，没办法，只有将之前的购买的服务器删除掉，然后重写搞一个。这次我就学聪明了，在购买服务器之前，我先在网上找了一下各个云服务器的价格。
<!--more-->
其实可供我们选择的服务器还是挺多的，比如阿里云，百度云，腾讯云，京东云等等。反正各个公司提供的服务器由国内的，也有国外的，提供的服务大致相同，不过价格就千差万别。这里给出一张图，这张图也是我借鉴别人的，这里是出处[各大厂商云服务器对比](https://blog.csdn.net/ithomer/article/details/77825664)
![云服务器各大厂商价格对比](/assets/skill/skill_vpn_01.png)
可能仁者见仁，智者见智。我觉得对于国内的开发者而言，最好还是选择腾讯云，不管是价格上，还是方便服务上，都是最佳之选，好像现在小程序的后台可以直接放在腾讯云服务器上的，棒棒哒。像我就很惨，因为我之前买了阿里的域名，后面如果想买腾讯云服务器，可能会比较麻烦。

好了好了，跑远了，其实除了上面介绍的几种云服务器之外，我使用的是另外一种[vultr](http://www.vultr.com)。相比较其他的云服务器而言，这个服务器的好处时计时收费，也就是说如果我最近一段时间不需要使用，则可以将服务器停掉，而不会扣钱。对国内外用户都很友好，而且ip地址还行吧，至少在这个时间段，我还能找到一个可用的IP地址，难道是我运气好？相面就将如何利用Vultr搭建SSR的教程记录下来

## 创建账号

首先登陆vultr的[官方网站](http://www.vultr.com)，如下图所示，点击注册按钮

![官方网站注册账号](/assets/skill/skill_vpn_02.png)

进入到注册账号页面，这里我们需要添加自己的邮箱和密码

![添加邮箱和密码](/assets/skill/skill_vpn_03.png)

创建完成之后，会向我们注册的邮箱发送一个激活链接，我们登录邮箱，点击链接激活即可。

这里我们需要充钱了，关于充钱的这一块比较敏感，为了不引起歧义，我就不说如何充钱了，反正充就完了，不充钱肯定没法用。

## 创建服务器

创建完成账号之后，进入到如下的页面
![首页](/assets/skill/skill_vpn_04.png)

点击添加服务器按钮，开始添加服务器

最上面是服务器类型，这里我直接选择默认的类型即可

![选择服务器类型](/assets/skill/skill_vpn_05.png)

下面是服务器的地址，目前我选择的是芝加哥，其他区域的有一些可以用，有一些用不了，可根据个人喜好，自由选择

![选择服务器地址](/assets/skill/skill_vpn_06.png)

选择服务器类型，一般都是选择centOS系统，但是这里有一个地方需要注意，网上很多教程中使用的命令都是针对CentOS6的，如果选择了其他的版本，不能保证安装一定成功，反正我一开始使用的是CentOS8，然后自己鼓捣了很长时间都没有完全解决兼容问题，这里建议大家使用CentOS6，那么下面的内容也是根据CentOS6写的

![选择服务器操作系统](/assets/skill/skill_vpn_07.png)

选择服务器存储，一般情况下，如果不是作为大中型网站后台，直接使用最低版本即可，反正也就是云服务，后面最多会再去使用作为个人博客后台，所以对性能要求没有这么大

![选择服务器存储性能](/assets/skill/skill_vpn_08.png)

下面的一些操作可有可无，如果感兴趣的，可以去官网查看，这里我直接点击部署按钮

![部署服务器](/assets/skill/skill_vpn_09.png)

部署完成之后，如果操作正确，就会出现如下的内容，注意，ip地址必须是正确的，如果出现乱码，那么你需要将服务器删除掉重新部署一个，部署的步骤和上面的一样

![部署完成](/assets/skill/skill_vpn_10.png)

例如：如果出现这样的情况，说明服务器部署不正确，或者说你的电脑不支持ipv6，所以，你要重新创建

![当前电脑不支持ipv6](/assets/skill/skill_vpn_11.png)

部署完成之后，需要在命令行中测试一下这个服务器是否可用，如果出现下面内容表示服务器部署正常，否则需要删除服务器重新创建

![ping当前服务器地址](/assets/skill/skill_vpn_12.png)

## 配置服务器

我们的服务器部署完成之后，并不代表着我们可以直接使用，如果我们想要使用SSR，那么需要在服务器中安装相应的软件，如何配置服务器？这里我们需要使用特殊的软件，我将软件上传到了百度云盘
[辅助工具](https://pan.baidu.com/s/1--d8fvDY-fESt8oWolvP_A)

这里因为写博客的电脑是windows，只能以XShell为例。将下载好的软件解压之后，找到windows软件，直接双击运行安装即可。安装完成之后，双击图标打开
打开之后的内容如下所示，当然如果你是第一次使用，肯定是没有这么多东西的

![XShell初始化页面](/assets/skill/skill_vpn_13.png)

我们点击新建一个连接

![XShell新建连接](/assets/skill/skill_vpn_14.png)

这里我们先完成基本配置，如下所示

![XShell新建连接的基本配置](/assets/skill/skill_vpn_15.png)

选择用户身份验证，添加用户名和密码

![XShell配置用户名和密码](/assets/skill/skill_vpn_16.png)

用户名一般都是root，密码需要我们登录vultr中的服务器详情页面，去复制

![获取服务器密码](/assets/skill/skill_vpn_17.png)

填写好用户名和密码之后直接点击"链接"按钮

在连接成功之后会出现如图所示的弹窗，这个弹窗表示的是我们使用SSH登录的验证信息，直接接受并保存即可

![XShell链接成功1](/assets/skill/skill_vpn_18.png)

之后如果出现如下所示的内容表示已经连接上远程服务器

![XShell链接成功2](/assets/skill/skill_vpn_19.png)

紧接着，将下面的代码复制到命令行中，并且执行

```
wget --no-check-certificate https://freed.ga/github/shadowsocksR.sh; bash shadowsocksR.sh
```

如果运行报错，可能是没有安装相应的软件，根据提示安装即可

如果没有报错，则会出现如下所示的内容.表示需要我们填写当我们使用SS连接时候的密码

![XShell设置SS连接密码](/assets/skill/skill_vpn_20.png)

点击回车键，出现如下所示内容，表示我们需要填写使用哪个端口作为SSR的输出

![XShell设置端口](/assets/skill/skill_vpn_21.png)

点击回车键，出现如下所示内容,之后点击任意按键开始安装

![XShell开始安装](/assets/skill/skill_vpn_22.png)

后面的安装过程大概持续十分钟左右，如果在这个过程中没有出现任何错误，表示我们的配置已经完成了，并且会出现如下图所示的内容

![XShell安装完成](/assets/skill/skill_vpn_23.png)

这里我们使用的方式是别人写好的脚本，后面如果有机会我也会写一个自己的脚本，尽量兼容更多的版本

到目前为止，基本配置已经完成，我们可以开开心心的打开SS开始外面的世界了。但是这时候的网速会比较差，因为我们没有对SSR进行优化，优化的操作其实就是安装锐速的过程

## 安装锐速

锐速的安装并不是每个服务器都能安装的，像CentOS8好像就不能安装。至于能不能安装，我们需要自己动手检查一下
```
uname -r
```

回车后输出当前系统内核版本。主要分三种情况：

1、结果以 2 开头，例如 2.6.32-696.18.7.el6.x86_64。

这种输出结果说明我们的服务器为 CentOS6 x64 系统。

2、结果以 3 开头，例如 3.10.0-693.11.6.el7.x86_64。

这种输出结果说明我们的服务器为 CentOS7 x64 系统。

3、结果以 4 开头，例如 4.12.10-1.el7.elrepo.x86_64。

这种输出结果说明我们的服务器已经安装 Google BBR 拥塞控制算法，此时已经无法继续安装锐速。

#### CentOS6 安装锐速

使用这段话
```
wget --no-check-certificate -O appex.sh https://raw.githubusercontent.com/hombo125/doubi/master/appex.sh && bash appex.sh install '2.6.32-642.el6.x86_64'
```
点击回车键，出现如下所示内容
![锐速准备开始安装](/assets/skill/skill_vpn_24.png)

之后会要求我们赋予权限，如图中红色方框的内容

![锐速权限赋予](/assets/skill/skill_vpn_25.png)

完成之后如果出现如下所示图片，表名安装成功

![锐速按钮成功](/assets/skill/skill_vpn_26.png)

#### CentOS7 安装锐速

若确定服务器为 CentOS7 x64 系统则看这一步。

按照下图提示，我们继续复制下列命令：
```
wget --no-check-certificate -O rskernel.sh https://raw.githubusercontent.com/hombo125/doubi/master/rskernel.sh && bash rskernel.sh
```

![](/assets/skill/skill_vpn_27.png)

等待内核更换完毕后系统会自动重启并断开连接。然后重新连接，执行下面命令。
```
yum install net-tools -y && wget --no-check-certificate -O appex.sh https://raw.githubusercontent.com/0oVicero0/serverSpeeder_Install/master/appex.sh && bash appex.sh install
```

回车
![](/assets/skill/skill_vpn_28.png)

按回车键继续，系统会自动安装锐速，同时会先后要求我们设置锐速的三项信息。按照下图提示，我们每次都直接回车继续即可。

![](/assets/skill/skill_vpn_29.png)

出现下面信息，就说明锐速安装成功了

![](/assets/skill/skill_vpn_30.png)

## 影梭配置

### android手机

![android手机配置影梭](/assets/skill/skill_vpn_31.png)

### Windows电脑

![Windows电脑配置](/assets/skill/skill_vpn_32.png)

# 参考资料
[用VPS搭建SSR服务](https://www.baishitou.cn/1524.html)