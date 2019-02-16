---
layout: post
keywords: git push error, update github password
description: git push error because of updating github password, 更新github密码导致推送失败
title: 更新github密码导致推送失败
comments: true
---

mac环境下（当然其他系统下应该也是一样），

由于修改了github的登录密码，原先已经clone的库，想要push，就会失败，错误如下。

```
remote: Permission to xxx.git denied to xxx.
fatal: unable to access 'https://github.com/xxx.git/': The requested URL returned error: 403
```

更新密码后，新clone下来的库想要push，也会发生如下错误，

```
! [remote rejected] develop -> develop (permission denied)
error: failed to push some refs to 'https://github.com/xxx.git'
```

尝试了很多的方法，比如重新生成~/.ssh下的公钥文件，git的重新安装，都不能解决，

在此记录一下解决方法，

[Updating credentials from the OSX Keychain](https://help.github.com/articles/updating-credentials-from-the-osx-keychain/)
