---
title: mac OS下配置php-fpm+Nginx服务并运行slim框架项目
date: 2017-07-10 18:18:32
tags: [php, fpm, nginx]
categories: 
  - nginx
  - php
  - fpm
---


最近公司有个项目是用的php写的后端服务，相关的开发和运维人员也没有php的项目部署经验，所以我也就协助着弄了一下关于在Linux环境下部署php项目，顺便学习一下新的知识，拓宽一下自己的知识面。然后事后在自己的电脑中也部署了一套同样的环境。

<!-- more -->

主要包括以下几个步骤：

1. php的安装以及php-fpm的安装

2. Nginx的安装以及配置

3. 安装composer和slim

4. 运行和测试

## php的安装以及php-fpm的安装

mac OS中已经自带php以及php-fpm的环境了，所以也就省去了安装php以及php-fpm的步骤。

使用php -version，可以查看当前安装的php版本。我本机的环境是5.6.30，而slim官网要求的是php5.5或者更高，所以也不需要更新php的版本。

但是，php-fpm还是需要配置之后才可以使用，在shell中直接运行php-fpm会报错找不到配置文件。

![install-fpm](http://web-site-files.ashshen.cc/blog/php-fpm_error.png)

配置php-fpm：

首先复制配置文件：

``` bash
cp /private/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
```

然后修改默认的配置和pid配置

``` bash
vim /usr/local/etc/php-fpm.conf
```

修改pid和error_log为

``` bash
pid = /usr/local/var/run/php-fpm.pid
error_log = /usr/local/var/log/php-fpm.log
```
然后使用以下命令启动php-fpm
``` bash
php-fpm --fpm-config /usr/local/etc/php-fpm.conf
```
当然，也可以不修改默认配置以及pid配置，使用–prefix执行放置运行时文件路径的前缀
``` bash
php-fpm --fpm-config /usr/local/etc/php-fpm.conf --prefix /usr/local/var
```
启动php-fpm之后，可以使用lsof -nP -iTCP:9000查看启动情况 ( php-fpm默认的启动端口是9000 ) 。

*ips：启动php-fpm时，无需使用sudo启动。*

停止php-fpm服务
``` bash
sudo killall php-fpm
```

## Nginx的安装和配置

mac OS下可以使用brew来安装Nginx。

``` bash
brew install nginx --with-http_geoip_module --with-http2
```
安装完成之后需要配置Nginx
``` js
server {
  listen 8011;
  server_name localhost;
  index index.php;
  error_log/XXXX/XXXX/XXX/error.log;
  access_log/XXXX/XXXX/XXX/access.log;
  root /XXXX/XXXX/XXX;

  location / {
    try_files $uri /index.php$is_args$args;
  }

  location ~ \.php {
    try_files $uri =404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME $fastcgi_script_name;
    fastcgi_index index.php;
    fastcgi_pass 127.0.0.1:9000;
  }
}
```
以上是Nginx的示例配置，listen是监听的端口，root是php文件的路径，可根据需求自行更改。

配置完成之后启动Nginx。

Nginx相关命令
``` bash
# 启动
$ sudo nginx
# 重载配置|停止|重启|退出
$ sudo nginx -s reload | stop | reopen | quit
```
启动完Nginx以及php-fpm后，我们可以测试一下是否正确：

在root对应的目录下新建info.php，并输入如下内容
``` php
<?php phpinfo(); ?>
```
然后在浏览器中输入：http://localhost:8011/info.php  ，显示系统安装的php信息则表示Nginx和php-fpm已正确运行。

## 安装composer和slim

composer是一个管理php依赖的工具，类似于Node.js中的npm。

安装方式
``` bash
cd /usr/local/bin
curl -sS https://getcomposer.org/installer | php
###等待下载完成
sudo mv composer.phar composer
sudo chmod a+x composer
```
然后就可以使用composer了，使用composer –version查看当前安装的版本。

Slim是php的一个微框架，可以用来快速编写简单但是功能强大的web应用和API（摘自[slim官网](http://slimphp.net/)介绍）。

安装slim，然后在项目根目录下执行： `composer require slim/slim "^3.0"`

PS：如果安装速度很慢可以使用国内镜像：[https://pkg.phpcomposer.com/](https://pkg.phpcomposer.com/)。

安装完成后在项目根目录下会多出一个composer.json文件以及vendor文件夹，composer.json是composer的配置文件，作用相当于npm中的package.json，vendor文件夹中是安装的依赖。

然后我们可以使用一个简单的例子来测试slim是否已经安装完成，将项目根目录的index.php修改为
``` php
<?php require 'vendor/autoload.php'; $config = ['settings' => [
    'addContentLengthHeader' => false,
]];
$app = new \Slim\App($config);

$app->get('/hello/{name}', function ($request, $response, $args) {
    return $response->write("Hello " . $args['name']);
});

$app->run();
```
保存，然后在浏览器中输入http://localhost:8011/hello/ash ，页面输出Hello ash，slim安装成功。

OK，服务器环境部署完成。
