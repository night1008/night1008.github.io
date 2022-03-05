---
layout: post
keywords: multi tag filter
description: 如何实现标签过滤
title: 如何实现标签过滤
comments: true
---

昨天同事在写业务框架的时候，问了我一个问题，
他构建了如下两只表，如何实现标签过滤，
现在将各个可行的方案整理一下。

```sql
sqlite3 test.db

CREATE TABLE resources (
  id INT PRIMARY KEY NOT NULL,
  name TEXT    NOT NULL
);

INSERT INTO resources (id, name)
VALUES (1, "r1"), (2, "r2"), (3, "r3");

CREATE TABLE labels (
  key TEXT NOT NULL,
  value TEXT NOT NULL,
  resource_id INT,
  PRIMARY KEY (key, value, resource_id)
);

INSERT INTO labels (key, value, resource_id)
VALUES ("foo", "bar", 1), ("x", "y", 1), ("foo", "bar", 2), ("x", "y", 2), ("foo", "bar", 3);
```

### 问题

基于以上库表结构，如果想要查询同时拥有`foo=bar`和`x=y`标签的 resource，应该如何编写查询语句。

#### 方案一
> mysql 不支持 intersect 操作符，sqlite 和 postgresql 可以

```sql
select * from resources where id in (
  select resource_id from labels where key = 'foo' and value = 'bar'
  intersect
  select resource_id from labels where key = 'x' and value = 'y'
)
```

#### 方案二
```sql
select * from resources where id in (
  select resource_id from labels where key = 'foo' and value = 'bar'
) and id in (
  select resource_id from labels where key = 'x' and value = 'y'
)
```

#### 方案三
```sql
select * from resources where (
  select count(1) = 2 from labels
  where resource_id = resources.id and (
    (key = 'foo' and value = 'bar') or (key = 'x' and value = 'y')
  )
)
```

以下两个方案来自 [sql-query-for-multiple-tag-inclusion](https://stackoverflow.com/questions/3798355/sql-query-for-multiple-tag-inclusion)


#### 方案四
```sql
select * from resources
  where exists (
    select * from labels
      where resources.id = labels.resource_id and  labels.key = 'foo' and labels.value = 'bar'
  ) and exists (
    select * from labels
      where resources.id = labels.resource_id and  labels.key = 'x' and labels.value = 'y'
  )
```

#### 方案五
```sql
select resources.*, count(*) as count from resources
  join labels on resources.id = labels.resource_id
  where (labels.key = 'foo' and labels.value = 'bar')
    or (labels.key = 'x' and labels.value = 'y')
  group by resources.id
  having count = 2
```

### 结论
最后处于查询性能和 SQL 语言的通用性考虑，选择了`方案二`。
