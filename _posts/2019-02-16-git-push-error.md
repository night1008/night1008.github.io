---
layout: post
keywords: mac,git push error
description: git push error on mac
title: mac git push error
comments: true
---

mac环境下，

由于修改了github的登录密码，同时还使用着其他的github账号

这时候想要，就会push失败，错误如下。

```
remote: Permission to xxx.git denied to xxx.
fatal: unable to access 'https://github.com/xxx.git/': The requested URL returned error: 403
```

更新密码后，新clone下来的库想要push，也会发生如下错误，

```
! [remote rejected] develop -> develop (permission denied)
error: failed to push some refs to 'https://github.com/xxx.git'
```

在此记录一下解决方法，


