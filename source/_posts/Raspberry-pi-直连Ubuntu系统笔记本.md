---
title: Raspberry pi 直连Ubuntu系统笔记本
date: 2016-06-14 14:09:00 +0800
categories: raspberry
tags: raspberry 
---

想通过ssh连上你的raspberry pi就应该使你的电脑和raspi处在同一个网段下(你的raspi有独立IP当我没说), 那么首先想到的就是把它们都连接到同一个路由器上, 如果手边没有路由器的话, 其实你也可以直接用一根网线直接连接你的电脑和raspi!

## Step 1: 准备一根网线

不就是一根网线么? 有什么好讲的? 是这样的, 笔者之前上网搜索的时候, 看过一篇帖子:[Raspberry与笔记本直连](http://www.eeboard.com/bbs/thread-6476-1-1.html), 里面有个人说

> camark
> 发表于 2013-1-10 15:51:24 |只看该作者
> 肯定不行。双机直连需要特殊的网线，而且要设置在一个网段。。

笔者当时就有点失望, 不过也硬着头皮随便找了一根网线, 最后也成功了, 最后也没有深究那个人说的是否是对的, **因此, 在此强调, 照着本文操作, 可能面临的风险**

## Step 2: 设置你笔记本的网络

其实也就是新建一个网络连接, 以Ubuntu 15.04为例子, 打开网络连接(屏幕右上角双箭头或wifi标志处), 点击编辑连接, 点击新建连接后如下图所示:

![add a new connection](http://ww2.sinaimg.cn/mw690/8932190cgw1f4up1i8h91j20f60ckt9z.jpg)

选择Ethenet(以太网), 点击创建, 在弹出来的框框中点击IPv4 Settings选项卡, Method栏选择Automatic (DHCP), 图如下:

![config your new connection](http://ww4.sinaimg.cn/mw690/8932190cgw1f4up1i8etmj20fu0dc0u4.jpg)

至此, 网络设置就OK了

## Step 3: 插进去!!

把网线的一头插笔记本, 一头插raspi, 点击上步新建的网络连接, 等待连接建立成功, 不出意外, 连接成功了! 出意外的肯定是网络连接没设置好或者网线是坏的(这tm不是废话么...)

## Step 4: Find out your Raspi's IP address

连接建立后你可以查看一下笔记本的连接信息, 找出你的笔记本IP地址, 注意! 是查看你连接raspi的网络连接的IP地址, 而不是如果你连了wifi的IP地址!

假设你的笔记本IP地址是 10.42.0.1 , 使用nmap查找同网段所有主机IP地址, 在终端下键入`nmap 10.42.0.1/24`, 等待片刻你会发现有两台主机, 一台你的笔记本, 一台就是raspi, 还可以看到有哪些端口打开着. 至于nmap, 你需要自行安装, 一般终端键入`sudo apt-get install nmap`即可, 好了, 拿到你的raspi的IP, 就OK了, 接着键入`ssh pi@raspi的IP地址`, 默认密码raspberry



