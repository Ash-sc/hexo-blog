---
title: mac下nvm安装与使用
date: 2017-06-11 17:58:11
tags: [mac, nvm]
categories: Node
---

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-14.jpg)

nvm作为当前主流的Node.js版本管理工具，可以做为一个Node.js版本切换工具或者Node.js安装工具使用。

<!-- more -->

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-14.jpg)

### 安装nvm

``` bash
$ curl https://raw.github.com/creationix/nvm/v0.4.0/install.sh | sh
```

如果使用的是是`zsh`，需要在安装完成后使用`source ~/.zshrc`重启配置。

如果使用的是自带的`Teminal`的话，则需要在安装完成后使用`source ~/.profile`重启配置。

重启后，在命令行中输入`nvm`，如果出现`nvm`相关信息则表示安装成功。

### nvm命令

**查看版本：**

安装完nvm之后，我们可以使用`nvm ls-remote`来查看Node.js发布的所有版本。

**安装Node.js：**

然后使用`nvm install v6.2.0`来安装6.2.0版本。

**切换版本：**

切换版本之前，我们需要使用`nvm install`安装至少两个版本；

ps: 不能在使用nvm安装的Node.js版本与其他安装方式安装的Node.js版本之间切换。

然后，使用`nvm use v8.0.0`来切换到8.0.0版本的Node.js。

**查看当前已经安装的版本：**
``` bash
nvm ls
```
**查看正在使用的版本：**
``` bash
nvm current
```
**设置默认版本：**
``` bash
nvm alias default x.x.x
```
**以指定版本执行脚本：**
``` bash
nvm run 6.2.0 app.js
```
**卸载nvm：**
``` bash
rm -rf ~/.nvm
```
PS:关掉终端，或者在其他的终端tab中使用nvm的话，如果出现错误提示：`command not found: nvm`的话，使用`source ~/.zshrc`或者`source ~/.profile`即可。
