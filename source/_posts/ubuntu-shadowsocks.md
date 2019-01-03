---
title: ubuntu下搭建shadowsocks服务
date: 2017-05-14 17:50:54
tags: [ubuntu, shadowsocks, 科学上网]
categories:
  - linux
  - shadowSocks
---

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-21.jpg)

俗话说得好：不会用Google的程序员不是好程序员。倒不是刻意贬低百度，实在是在技术问题搜索结果上，百度和Google搜索出来的内容质量差别太大。

对我而言，百度只用来日常生活的搜索，至于工作中技术问题的搜索，还是google合适。

本篇主要介绍ubuntu下如何搭建shadowsocks服务，也就是我们经常说的“翻墙”。

<!-- more -->

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-21.jpg)

## 首先

首先，你需要一个服务器，一个在Great Firewall (of China)之外的服务器。（国外的或者香港的喽）。

至于服务器的供应商的话：阿里云和腾讯云貌似可以买到香港的服务器，但是价格方面的话……，当然，碰上双十一或者其他活动，还是可以买到价格比较优惠的服务器的。

国外的VPS供应商的话，大概有这一些：[linode](https://www.linode.com/)、[DigitalOcran](https://www.digitalocean.com/)、[搬瓦工](https://bandwagonhost.com/)、[Vultr](https://www.vultr.com/)等。

之前用过DigitalOcean家的服务器，最低配的$5/month。不过不论是新加坡的节点还是美国的节点，都经常会有网络波动的情况，加上他们家卡控制台实在要疯掉，所以后来也就弃用了。

后来转到了Vultr，同样配置的$2.5/month，不过经常会没有货，所以需要看脸去抢（贴一个邀请链接：想用他们家服务的朋友可以用这个链接注册，这样我就能拿到一部分报酬了，笑:-)  [https://www.vultr.com/?ref=7300371](https://www.vultr.com/?ref=7300371)）。

搬瓦工的话，貌似有更便宜的版本（$19.99/year）不过他们家的我没有试过，10GB的硬盘感觉还是会有点不够用。选来选去，还是选择了Vultr。

PS：至于下载速度的话，我只能说，肯定是没法跟专业付费的VPS相比的，不过，即使不做其余的加速措施，我这边Vultr的服务器下载速度最高也能达到2MB/s，用用google，上上油管，那是绝对够的。一家子人用的话，也差不多够用了，毕竟平均下来一天也就5毛钱。

## 然后

有了服务器，接下来的事情就是安装shadowsocks了。

使用ssh连接到购买的服务器。当然……也可以直接使用网页版的console。

然后输入如下命令：

``` bash
sudo apt-get update
sudo apt-get install python-gevent python-pip python-m2crypto python-wheel python-setuptools
sudo pip install shadowsocks
```

shadowsocks安装完成。

安装完成后，修改一下shadowsocks的配置，如下：

``` bash
sudo vim /etc/shadowsocks.json
然后添加：
{
    "server":"my_server_ip", // 替换为你的服务器ip
    "server_port":8000,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"my_password",  // 设置密码
    "timeout":300,
    "method":"rc4-md5"
}
```

然后保存并退出（:wq）。

## 启动与停止

``` bash
sudo ssserver -c /etc/shadowsocks.json -d start
sudo ssserver -c /etc/shadowsocks.json -d stop
```

## 使用chacha20加密方式

我个人一般是使用chacha20来加密的，各种加密方式的区别以及优劣势这里就不细说了。

添加chacha20加密方式的方法：

``` bash
wget https://github.com/jedisct1/libsodium/releases/download/1.0.10/libsodium-1.0.10.tar.gz
tar xf libsodium-1.0.10.tar.gz && cd libsodium-1.0.10
apt-get install build-essential
./configure && make && make install
ldconfig
```

然后修改shadowsocks配置中的”method”值为”chacha20″，重新启动shadowsocks服务即可。

## 客户端下载

Mac：[https://github.com/shadowsocks/ShadowsocksX-NG/releases/](https://github.com/shadowsocks/ShadowsocksX-NG/releases/)

Win：[https://github.com/shadowsocks/shadowsocks-windows/releases](https://github.com/shadowsocks/shadowsocks-windows/releases)

客户端设置的话，按照你在服务端shadowsocks配置中的来就行。

