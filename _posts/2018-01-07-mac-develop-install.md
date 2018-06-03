---
layout: post
keywords: mac, develop, install, 安装开发环境
description: install develop software on mac, mac上安装工作用的开发环境
title: mac上安装工作用的开发环境
comments: true
---

```
App store 内下载Xcode
```

```
安装homebrew:
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bash_profile
```

```
安装python:
由于系统自带的python版本是2.7.10，因此2.7.+版本的需要另外安装

python -V

brew  install python

python2 -V

vim ~/.bash_profile

export PATH="/usr/local/opt/python/libexec/bin:$PATH"

. ~/.bash_profile

python -V

pip install virtualenv -i http://pypi.douban.com/simple/

brew install python3

python3 -V
```

```
安装git:
brew install git

git config --global user.name “yourname”

git config --global user.email “youremail”
```

```
安装node:
brew install node
node -v

brew install n
sudo n lts
node -v
```

```
安装数据库：
brew install mysql

brew instal postgresql  (存储目录为: /usr/local/var/postgres)

brew service start postgresql

pg_ctl -V

psql -d postgres
```

```
安装php:
由于系统自带的php版本是7.1.7，因此7.1.+版本的需要另外安装

php —version

brew tap homebrew/dupes

brew tap homebrew/versions

brew tap homebrew/homebrew-php

brew reinstall php71 --with-postgresql --with-mysql

vim ~/.bash_profile

export PATH="$(brew --prefix homebrew/php/php71)/bin:$PATH"

. ~/.bash_profile

php —version


独立安装php扩展:
brew install homebrew/php(可选， 可查看所有扩展)

brew install homebrew/php/php71-pdo-pgsql

vim /usr/local/etc/php/7.1/conf.d/ext-pdo_pgsql.ini

注释掉 extension="/usr/local/opt/php71-pdo-pgsql/pdo_pgsql.so"
```

```
安装redis:
brew install redis

brew service start redis
```

```
安装iTerm2 + ZSH:
http://www.iterm2.com/downloads.html

brew install zsh zsh-completions

curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh

chsh -s /usr/local/bin/zsh
```

```
安装elasticsearch:
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

brew install elasticsearch

执行 elasticsearch
```

```
安装sublimetext:
https://www.sublimetext.com/3

ln -sv "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/subl

References:
{
    "auto_complete_delay": 5,
    "font_size": 13,
    "ensure_newline_at_eof_on_save": true,
    "translate_tabs_to_spaces": true,
    "trim_trailing_white_space_on_save": true,
}

Packages:
All Autocomplete

Python PEP8 Autoformat
```

```
其他软件：
https://github.com/shadowsocks/shadowsocks-iOS/releases

https://www.google.cn/chrome/browser/desktop/index.html

https://pinyin.sogou.com/mac/
```
