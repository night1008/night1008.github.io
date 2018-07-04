---
layout: post
keywords: mac, php, upgrade, 升级php版本
description: php upgrade on mac, mac升级php版本
title: mac升级php版本
comments: true
---

mac当前版本10.13.2，自带的php版本是7.1.7，最新的是7.1.19

```
php -v  # PHP 7.1.7

brew uninstall php71
brew uninstall hp71-pdo-pgsql
brew install php@7.1

vim /usr/local/etc/php/7.1/conf.d/ext-pdo_pgsql.ini

因为之前独立安装了pdo_pgsql的php扩展
注释掉 extension="/usr/local/opt/php71-pdo-pgsql/pdo_pgsql.so"
否则会报
PHP Warning:  Module 'pdo_pgsql' already loaded in Unknown on line 0

php -v  # PHP 7.1.19
```

