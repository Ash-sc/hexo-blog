---
title: Linux服务器相关设置
date: 2018-12-10 09:56:38
tags: [Linux, ubuntu]
categories: linux
---

一些关于Linux服务器设置的语句，记下来防止遗忘，虽然都是可以查到的。

<!-- more -->

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

### Centos 防火墙

`Centos 7` 使用 `firewall` 作为默认防火墙，`Centos 6` 使用 `iptable`。

iptable:

``` bash
service iptables status # 查看防火墙状态
service iptables stop # 临时关闭防火墙
chkconfig iptables off # 永久关闭防火墙
```

firewall:

``` bash
firewall-cmd --state # 防火墙状态
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```

#### iptables: unrecognized service

原因是iptables未找到

``` bash
sudo yum install iptables
sudo apt-get install iptables
```

#### -bash: service: command not found

遇到此问题，使用 `su - root` 进入root模式再执行命令。


#### vsftp

查看vsftp账号：

linux中，vsftp信息保存在`/etc/vsftpd/`中。

查看所有用户：`cat /etc/vsftpd/ftpusers`。无法查看密码，修改密码：`passwd [ftp_username]`。


