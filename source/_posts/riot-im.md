---
title: 使用docker十分钟搭建一个局域网IM工具 —— Roit
date: 2020-01-03 16:31:58
tags: [im]
categories: 即时通讯
---

`Riot im` 是一个以 `Matrix` 协议为基础的开源软件，使用 `React` 开发，有 `android`、`ios`、`mac`、`windows`、`linux`多平台客户端，支持文本、markdown、文件、视音频等传输。

`Riot im` 只需要服务器提供注册能力就可以与在不同服务器注册的用户进行通信，并且支持私信端到端加密，因此大受隐私爱好者喜爱。

本篇主要介绍如何使用 `docker` 来搭建一个 `synapse` 服务器供 `Roit im` 使用。

<!-- more -->

## 获取最新镜像
``` shell
docker pull matrixdotorg/synapse
```

## 生成配置文件

这里更改 `your-domain` 为你的服务器域名或者ip。
``` shell
docker run -it --rm \
    --mount type=volume,src=synapse-data,dst=/data \
    -e SYNAPSE_SERVER_NAME=your-domain \
    -e SYNAPSE_REPORT_STATS=yes \
    matrixdotorg/synapse:latest generate

```

## 修改配置文件

生成的配置文件中是默认关闭了注册功能的。

首先找到上一步中生成的配置文件：
``` shell
find / -name homeserver.yaml
```
然后，更新配置文件中：`registration_enable` 为 `true`。

## 启动镜像
``` shell
docker run -d --restart=always --name synapse \
    --mount type=volume,src=synapse-data,dst=/data \
    -p 8008:8008 \
    matrixdotorg/synapse:latest
```

服务器地址即为：`your-domain:8008`。

可以在浏览器中直接打开验证：

![示例](http://web-site-files.ashshen.cc/blog/riot/synapse-page.png)

然后，使用客户端注册时，选择使用自定义服务器注册，服务器填写上面的服务器地址即可。

当然，也可以使用nginx等转换成https协议。