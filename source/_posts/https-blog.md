---
title: 十分钟升级wordpress博客到https
date: 2017-08-19 19:42:38
tags: [https, blog]
categories:
  - 前端
  - https
---

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-11.jpg)

[前面](/2017/08/17/https/)有说到https与http的一些区别以及https的一些好处。当然，对于平时也就用来写写博客的的ashshen.cc来说，升级到https实际上是没有太大必要的 ( PS：可以避免某些网络运营商插入广告 ) ；不过闲着也是闲着，有时间和精力又有条件的话，练练手熟悉熟悉也是极好的。

本文主要是针对使用wordpress（apache2 + php7）搭建的博客如何升级到https。

<!-- more -->

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-11.jpg)

## Let’s Encrypt和Certbot

对于https而言，ssl是最重要的部分。那么，对于https网站而言，ssl的证书也就是必不可少的了，所以想要升级网站到https，我们得先搞一个证书。

现有的ssl证书认证机构大多是收费的，这里推荐一个免费的证书认证机构：[Let’s Encrypt](https://letsencrypt.org/)（只需要每3个月更新一次证书）。

[Certbot](https://certbot.eff.org/)是针对Let’s Encrypt发布的一个客户端，利用它可以完全自动化的获取、部署和更新安全证书。

## 升级步骤

证书搞定，开始升级。

首先ssh连接服务器，并下载安装Certbot
``` bash
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
./certbot-auto
```
然后按照shell中的提示依次输入邮箱、ssl需要绑定的域名等信息
![certbot-install](http://web-site-files.ashshen.cc/blog/certbot-config-1.png)

![certbot-install-2](http://web-site-files.ashshen.cc/blog/certbot-config-2.png)

提示安装成功之后，在浏览器中输入url可以查看https是否升级成功。

![install-check](http://web-site-files.ashshen.cc/blog/https-site.png)

最后，不要忘记将wordpress后台设置中的地址改成https

![wordpress-setting](http://web-site-files.ashshen.cc/blog/change-setting.png)

## 更新Let’s Encrypt证书

如果你在安装certbot的过程中设置了正确的邮箱地址，那么在https证书过期时你将收到提示更新的邮件。

然后你可以使用下面的命令来更新证书

``` bash
certbot renew
// 或者
certbot renew --force-renewal
```

如果更新时提示有apache插件没有安装导致安装失败，可以通过`apt-get install python-certbot-apache`安装插件（安装插件时需要停止Nginx进程） ，然后再进行renew操作。






