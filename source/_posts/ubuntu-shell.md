---
title: Linux服务器相关设置
date: 2018-12-10 09:56:38
tags: [Linux, ubuntu]
categories: linux
---

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-22.jpg)

一些关于Linux服务器设置的语句，记下来防止遗忘，虽然都是可以查到的。

<!-- more -->

![img](http://web-site-files.ashshen.cc/blog-header-images/nature-22.jpg)

### 服务器 SSH KEY 登录设置

``` shell
# 本地生成SSH key
ssh-keygen

# 获取本地公钥
cat ~/.ssh/id_rsa.pub

# 使用密码登录到服务器
cd ~/.ssh
vi authorized_keys
# 写入本地公钥

# 服务器文件权限相关修改
chmod 600 authorized_keys
chmod 700 ~/.ssh

# 修改服务器ssh配置
vi /etc/ssh/sshd_config

RSAAuthentication yes
PubkeyAuthentication yes
PermitRootLogin yes

# :wq 后 验证是否能用key登录
# 修改服务器ssh配置， 禁用密码登录
vi /etc/ssh/sshd_config
PasswordAuthentication no

```

