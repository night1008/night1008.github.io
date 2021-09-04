---
layout: post
keywords: php7, mysql8, authentication
description: php mysql8 连接认证问题
title: php mysql8 连接认证问题
comments: true
---

最近发现 laravel 项目连接不上 mysql 数据库了，报了以下错误，

```php
Illuminate\Database\QueryException

SQLSTATE[HY000] [2054] The server requested authentication method unknown to the client (SQL: select * from information_schema.tables where table_schema = game_join and table_name = migrations and table_type = 'BASE TABLE')
```

原因：

在MySQL 8.0.11中，caching_sha2_password是默认的身份验证插件，而不是以往的mysql_native_password。

### 方法一：手动修改用户密码连接类型

```sh
mysql -u root -p

mysql > use mysql

mysql > select user, host, plugin from user;

mysql> alter user 'game_join'@'%' identified with mysql_native_password by 'secret';

mysql> mysql> select user, host, plugin from user;
```

### 方法二：设置 mysql.cnf

```yml
version: "3.1"

services:
  mysql:
    image: mysql:8.0.22
    environment:
      - MYSQL_ALLOW_EMPTY_PASSWORD=1
      - MYSQL_DATABASE=game_join
      - MYSQL_USER=game_join
      - MYSQL_PASSWORD=secret
    ports:
      - 3306:3306
    volumes:
      - mysql:/var/lib/mysql
      - ./mysql.cnf:/etc/mysql/conf.d/mysql.cnf

volumes:
  mysql: {}
```

创建 `mysql.cnf` 文件，放置以下内容

```cnf
[mysqld]
default_authentication_plugin=mysql_native_password
```

这样就能正常连上了。
