---
layout: post
keywords: mac,laravel, 安装laravel
description: how to install laravel on mac, 如何在mac上安装laravel
title: 如何在mac上安装laravel
comments: true
---

由于mac自带的php版本上5.6，目前最新的版本上7.1，因此先要升级php版本。

```
brew tap homebrew/dupes
brew tap homebrew/versions
brew tap homebrew/homebrew-php
brew install --with-openssl curl
brew install php71
```

安装完后还需要一个用于切换php版本的包，选择 brew-php-switcher

```
brew install brew-php-switcher
brew-php-switcher 71
```

之后要安装laravel，有两种方法，

其一，全局安装laravel，需要先安装composer

```
brew install composer
composer global require "laravel/installer"
export PATH=$PATH:$HOME/.composer/vendor/bin
(或者
echo 'export PATH=/usr/local/bin:$PATH' >>~/.bash_profile
source ~/.bash_profile
)
laravel new blog
```

其二，局部安装laravel，需要先下载composer.phar

```
curl -sS https://getcomposer.org/installer | php (或者 php -r "readfile('https://getcomposer.org/installer');" | php)
php composer.phar create-project --prefer-dist laravel/laravel blog
mv composer.phar to blog
php composer.phar install
php artisan key:generate
php artisan serve
```

建议采用第二种，比较方便。
