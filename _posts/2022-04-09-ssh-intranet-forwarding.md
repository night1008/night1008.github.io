---
layout: post
keywords: ssh intranet forwarding
description: 如何通过 ssh 进行内网穿透
title: 如何通过 ssh 进行内网穿透
comments: true
---

最近几天居家办公，需要访问公司电脑上的代码。
当前做法是通过 SSH 端口转发，使内网主机 A 的服务转发至公网主机 B 上进行访问。

## 准备条件
1. 内网主机 (假设地址为: intranet-machine)
2. 公网主机 (假设地址为: aliyun-machine)

### 1. 修改公网主机 ssh 配置

```sh
vim /etc/ssh/sshd_config
```

```
GatewayPorts yes
```

```sh
sudo service sshd restart
```

### 2. 内网主机安装 [autossh](https://linux.die.net/man/1/autossh)

以 ubuntu 系统为例，可能系统已经安装好了 autossh

```
sudo apt update
sudo apt install autossh
```

### 3. 断线免密登录自动重连

1. 内网主机上生成公钥和私钥
> 假设输入名称 ssh-aliyun
```sh
ssh-keygen
```

2. 拷贝秘钥文件到公网主机上

内网主机上执行

```sh
ssh -i ~/Secrets/aliyun-ecs.cer root@aliyun-machine 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/ssh-aliyun.pub
```

公网主机上执行

```sh
sudo service sshd restart
```

> 由于当前公网主机上通过证书登录的，暂时不知道如何通过 ssh-copy-id 复制公钥到公网主机，
> 密码登录的情况下可以执行以下语句
> ssh-copy-id -i ~/.ssh/ssh-aliyun.pub root@aliyun-machine

### 4. 利用 autossh 实现端口转发

内网主机上执行

```sh
autossh -M 1223 -i ~/Secrets/aliyun.cer root@aliyun-machine -R 1222:localhost:22 -Nv
```

- -M 1223 意思是使用内网主机的 1223 端口监视 ssh 连接状态，连接出问题了会自动重连
- -N 意思是不执行远程命令
- -R 意思是将公网主机的某个端口转发到本地指定机器的指定端口，参数格式为`远程主机端口:目标主机:目标主机端口`
- 1222:localhost:22 意思是将内网主机的 22 号端口转发至公网主机的 1222 号端口上

### 5. 监听端口检查

内网主机上执行

```sh
lsof -i:1223
```

公网主机上执行
> 需要检查公网主机的防火墙是否开放响应端口

```
lsof -i:1222
```

### 6. 设置内网主机开机自动启动

mac 上设置开机自动启动

```sh
mkdir -p ~/Startups && cd ~/Startups
touch ssh-intranet-forwarding.sh
chmod 777 ssh-intranet-forwarding.sh
```

把上面内网主机启动 autossh 的命令写到 `ssh-intranet-forwarding.sh` 文件中，即

```sh
autossh -M 1223 -i ~/Secrets/aliyun.cer root@aliyun-machine -R 1222:localhost:22 -Nv
```

然后在系统设置(在`设置->用户与群组->登录项`)的登录项中添加启动文件。

### 7. 本地电脑使用

```sh
ssh root@localhost -p 1222 -J root@aliyun-machine
```