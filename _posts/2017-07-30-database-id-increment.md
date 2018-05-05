---
layout: post
keywords: database id, increment, 数据库ID自增策略
description: database id, increment, 数据库ID自增策略
title: 数据库ID自增策略
comments: true
---

{{ page.title }}
<p class="meta">30 Jun 2017</p>
<hr>

最近做数据库迁移时要保证旧数据库的数据ID也要迁移到新数据库，所以需要了解一下数据库ID的自增策略。

选取mysql和postgrep进行测试。

先说mysql,

创建个user表，

```
CREATE TABLE `user` (
  	`id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  	`name` varchar(256) DEFAULT NULL,
  	PRIMARY KEY (`id`)
)

(SHOW CREATE TABLE user;)

INSERT INTO user (name) VALUES ('a1');

INSERT INTO user (id, name) VALUES (1000, 'a2');

INSERT INTO user (name) VALUES ('a3');

INSERT INTO user (id, name) VALUES (100, 'a4');

INSERT INTO user (name) VALUES ('a5');
```

以下为插入结果

| id | |name |
| :----------| |:------|
| 1    | | a1 |
| 1000 | | a2 |
| 1001 | | a3 |
| 100  | | a4 |
| 1002 | | a5 |

说明mysql的自增ID策略为在目前最大的ID上进行自增。

可以对user表执行```TRUNCATE  user;```， 刷新ID进行多次测试。

***

再说postgrep,
创建个user表，

```
CREATE TABLE "user" (
	id SERIAL NOT NULL,
	name VARCHAR,
	PRIMARY KEY (id)
)

INSERT INTO "user" (name) VALUES ('a1');

INSERT INTO "user" (id, name) VALUES (1000, 'a2');

INSERT INTO "user" (name) VALUES ('a3');

INSERT INTO "user" (id, name) VALUES (100, 'a4');

INSERT INTO "user" (name) VALUES ('a5');
```

以下为插入结果

| id | | name |
| :----------| |:------|
| 1    | | a1 |
| 1000 | | a2 |
| 2    | | a3 |
| 100  | | a4 |
| 3    | | a5 |


倘若这时手动插入id为4的记录，之后再默认插入时就会报```duplicate key value violates unique constraint "user_pkey"```的错误。

可以对user表执行```TRUNCATE  "user";```， 刷新ID进行多次测试。

说明mysql和postgrep的ID生成策略是不一样的，如果旧数据库的ID迁移的时候也需要的话，还是应该注意一下。


