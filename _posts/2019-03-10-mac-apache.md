---
layout: post
keywords: mac,apache
description: mac下apache配置
title: mac下apache配置
comments: true
---

因为在godaddy上买了一个域名，想测试一下域名如何指向本机地址

实验了一下，在godaddy上的DNS 管理页面填入本机的`内网地址`即可，如下所示

![godaddy-dns-manage](/assets/img/2019-03-10/godaddy-dns-manage.png)

完成后直接访问域名，呈现的页面结果是

```
It works!
```

本机是mac，然而我并没有在80端口上启动程序。

查了一下原来mac上自带了apache，位置在```/private/etc/apache2```

默认的资源路径是```/Library/WebServer/Documents```

通过以下命令可以关闭apache服务，让80端口给其他程序使用

```
sudo apachectl start/restart/stop
```



