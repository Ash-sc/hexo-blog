---
title: 使用docker安装和启动mongo
date: 2018-02-28 20:32:45
tags: [docker, mongodb]
---

鉴于在windows下安装mongo的曲折经历（无力吐槽），最终决定使用docker来安装mongo。docker中运行的mongo与本地安装的mongo并没有什么区别，同时也可以实现在本地维护db文件，安装过程更加简洁（不用满世界的找安装文件了、也不用去官网填写表单注册账号了）。

## 首先

首先，你需要去docker官网下一个docker客户端，安装完成之后，设置docker镜像源为国内镜像（国外镜像的速度嘛，呵呵哒…）。

我这边的话，加了这三个镜像源：

``` bash
registry.docker-cn.com
https://docker.mirrors.ustc.edu.cn
https://hub-mirror.c.163.com
```
![docker-install](http://web-site-files.ashshen.cc/blog/docker/docker-mirrors.png)

应用，然后重新启动。

## mongo镜像与新建容器

安装完docker之后，通过docker pull mongo获取到mongo镜像。

pull完成之后可以通过docker images查看本地拥有的镜像列表。

![windows-shell](http://web-site-files.ashshen.cc/blog/docker/docker-images.png)

然后我们就可以通过mongo镜像来新建一个容器了。

这里，如果你本地有安装git的话，可以在文件夹中或者桌面“右键”，然后使用“git bash”来打开git命令行工具；如果本地没有安装git的话，那么你需要安装一个git，git的安装也比较简单，直接官网下载安装包一件安装即可。

PS：为什么要使用git bash？ 因为windows下的git客户端会自带一个叫**mingw**的命令行工具，可以执行简单的shell命令，之后用到的命令需要它。

在git bash中使用以下命令新建一个mongo的容器：

``` bash
docker run --name <自定义容器名> -p 27017:27017 -v F:/docker/mongodb:/data/db -d mongo --auth // （mac用户可以加上sudo）
```

`-p 27017:27017`表示：前面的27017是映射到本地主机的端口（如果你本地27017被占用，可以选择其他端口），后面的27017是docker容器中运行的mongo端口，这个是固定的。

`-v F:/docker/mongodb:/data/db`表示：冒号前面是本地维护的数据库文件所在，后面的是容器中mongo服务数据库文件，实际应用中一般都会将数据库文件映射到本地，方便维护。

新建容器完成后，可以使用`docker ps`查看当前运行的容器列表，使用`docker ps -a`可以查看当前所有的容器列表。

![window-shell-2](http://web-site-files.ashshen.cc/blog/docker/docker-ps.png)

可以看到，我们刚刚运行的容器在列表中（mongoTest），使用`docker stop mongoTest`命令停止运行容器，更多常见docker命令，可以看[这篇文章](http://www.youruncloud.com/docker/1_37.html)。

## 配置mongodb

mongodb服务已经启动起来了，接下来需要做的就是mongodb配置相关了（配置用户等）。

首先在git bash中执行以下命令：（容器名称请自行替换）

``` bash
docker exec -it <自定义容器名> mongo admin
```

PS：在这里，windows用户执行上面的命令时会遇到这样的问题。

![shell-3](http://web-site-files.ashshen.cc/blog/docker/windows-error.png)

这时候在命令前面加上“mintty ”变成“mintty docker ......”，就可以正常新建容器了。

然后，会进入到mongodb的命令行状态。执行以下命令，创建管理员账号：

``` bash
db.createUser({ user: '<USER>', pwd: '<PASSWORD>', roles: [ { role: 'userAdminAnyDatabase', db: 'admin' } ]});
```
mongodb提示用户创建成功，以后想要使用管理员身份登录mongo就可以使用命令
``` bash
docker exec -it <YOUR-NAME> mongo -u <USER> -p <PASSWORD> --authenticationDatabase admin
```
假设我们项目中用到的数据库为test，上面创建的管理员是没有test数据库的读写权限的，所以我们还需要再建一个用户。

在命令行中依次输入：
``` bash
use test; // 新建test数据库
db.createUser({user: '<USER>',pwd: '<PASSWORD>', roles: [{role: 'readWrite', db: 'test'}]}); // 创建新用户
db.auth('<USER>', '<PASSWORD>'); // 切换到新建的用户
db.test.insert({name: 'test db'}); // 确定用户是否新建成功
```
新建用户完成后，就可以使用新建的用户密码连接操作mongo了。

## mongo客户端

mongo的客户端的话，windows和mac都可以使用Robo 3T，下载地址：https://robomongo.org/；当然，你如果有使用IDEA家族的编辑器的话，可以直接装mongo插件：`mongo4idea`。


