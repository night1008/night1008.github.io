---
layout: post
keywords: mac,node
description: how to install node on mac
title: 如何在mac上安装node
comments: true
---

{{ page.title }}
<p class="meta">21 Sep 2017</p>
<hr>

首先安装nvm

```
brew install nvm
mkdir ~/.nvm
```

其次, 需要在shell的配置文件(~/.bashrc, ~/.profile, or ~/.zshrc)中添加如下内容：


```
# for nvm
export NVM_DIR=~/.nvm
source $(brew --prefix nvm)/nvm.sh
```

目前node的LTS上v6.11.3
```
nvm install 6.11.3
node -v
npm -v
```

由于nvm , npm 默认是从国外的源获取和下载包信息，很慢，建议通过以下命令安装。
```
npm --registry=https://registry.npm.taobao.org install
```

大功告成。