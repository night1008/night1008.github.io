---
layout: post
keywords: mac, install, supervisor, 安装supervisor
description: mac install supervisor using brew, mac如何安装supervisor
title: mac安装supervisor
comments: true
---

```
brew install supervisor
```

使用sudo权限启动，一定要使用sudo

```
sudo brew services start supervisor

supervisorctl 进行查看
```

不使用sudo权限

```
echo_supervisord_conf > supervisord.conf
sudo supervisord -c supervisord.conf
sudo supervisorctl  -c supervisord.conf
```

修改配置，便可在浏览器中查看
```
[inet_http_server]
port=127.0.0.1:9001

[supervisord]
nodaemon=true

[supervisorctl]
serverurl=http://127.0.0.1:9001
```

在 /usr/local/etc/supervisor.d/ 创建foo.ini，即可加入启动

```
[program:foo]
command=/bin/cat
```
