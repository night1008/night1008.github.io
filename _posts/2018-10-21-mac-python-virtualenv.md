---
layout: post
keywords: mac python virtualenv
description: mac python virtualenv
title: mac下使用virtualenv创建不同的python版本环境
comments: true
---

创建python2环境

```
mkdir .pyvenv
virtualenv .pyvenv
cd .pyvenv
. bin/activate
```


创建python3环境

```
mkdir .pyvenv
virtualenv -p python3 .pyvenv
cd .pyvenv
. bin/activate
```


