---
layout: post
keywords: ClickHouse, ASOF JOIN, 间隔分析
description: 如何通过 ClickHouse ASOF JOIN 实现间隔分析
title: 如何通过 ClickHouse ASOF JOIN 实现间隔分析
comments: true
---
### 什么是间隔分析？

[间隔分析](https://manual.sensorsdata.cn/sa/latest/zh_cn/%E9%97%B4%E9%9A%94%E5%88%86%E6%9E%90-138871875.html) 主要用于分析用户做两个动作的间隔时长。

产品、运营、市场等人员的日常工作都需要观察业务的转化情况。
衡量转化，除了用漏斗看转化率，还需要看转化时长的分布情况，**间隔分析** 即是解决这类问题和需求的。
通过计算用户行为序列中两个事件的时间间隔，得到业务转化环节的转化时长分布。

### 关键问题：初始和后续行为是如何配对的？

假设用户在过去某个时间段内行为序列是：A → C → A → B → B → A → B → B → A → A → C → D → A → B

此时我们分析用户做了 `A 事件`和 `B 事件`的时间间隔，会按如下的计算规则：
- 在发生 `A 事件`后，找到离 `A 事件` 最近的 `B 事件`，即为第一个间隔。从间隔向后继续找 A 和 B 的配对，间隔与间隔不交叉，依次类推；
- 选择聚合时间单位，按天、周、月聚合，则会限定配对的 `A 事件` 和 `B 事件`发生在同一天、周、月；例如：如果按天聚合，用户发生`A 事件` 的时间是 23:50 ，发生 `B 事件`的时间是次日 00:10 。这两个事件无法完成配对。
- 此时如果不考虑聚合时间单位，则间隔配对结果是：**A** → C → A → **B** → B → **A** → **B** → B → **A** → A → C → D → A → **B**

关于初始行为和后续行为的选择，还分为三种情况，
- 不同事件
- 不同筛选条件的相同事件
- 相同筛选条件的相同事件

### 如何用 ClickHouse 实现查询

最近在看 [ClickHouse ASOF JOIN Usage](https://clickhouse.com/docs/en/sql-reference/statements/select/join#asof-join-usage) 文档，发现是符合实现间隔分析的查询场景的。

You can use any number of equality conditions and exactly one closest match condition.

```sql
CREATE DATABASE demo ENGINE = Memory;

CREATE TABLE interval_test (event String, user_id Int64, group0 String, time DateTime) ENGINE = Memory;

-- 插入测试数据
INSERT INTO interval_test (event, user_id, group0, time) VALUES
('A', 1, 'a', '2024-02-07 01:00:00'),
('C', 1, 'a', '2024-02-07 01:01:00'),
('A', 1, 'b', '2024-02-07 01:02:00'),
('B', 1, 'a', '2024-02-07 01:03:00'),
('B', 1, 'a', '2024-02-07 01:04:00'),
('A', 1, 'b', '2024-02-07 01:05:00'),
('B', 1, 'a', '2024-02-07 01:06:00'),
('B', 1, 'a', '2024-02-07 01:07:00'),
('A', 1, 'b', '2024-02-07 01:08:00'),
('A', 1, 'b', '2024-02-07 01:09:00'),
('A', 1, 'a', '2024-02-07 01:10:00'),
('C', 1, 'a', '2024-02-07 01:11:00'),
('D', 1, 'a', '2024-02-07 01:12:00'),
('A', 1, 'a', '2024-02-07 01:13:00'),
('B', 1, 'a', '2024-02-07 01:14:00');
```

#### 1. 不同事件的查询语句
```sql
SELECT * FROM (
	SELECT *,
	row_number() OVER(PARTITION BY user_id ORDER BY time) AS row_number1,
	row_number() OVER(PARTITION BY t2.time ORDER BY time) AS row_number2 FROM (
	  SELECT * FROM interval_test WHERE event = 'A'
	) AS t1 ASOF JOIN (
	  SELECT * FROM interval_test WHERE event = 'B'
	) AS t2 ON t1.user_id = t2.user_id AND t1.time < t2.time
) WHERE row_number2 = 1;
```

查询结果
```
┌─event─┬─user_id─┬─group0─┬────────────────time─┬─t2.event─┬─t2.user_id─┬─t2.group0─┬─────────────t2.time─┬─row_number1─┬─row_number2─┐
│ A     │       1 │ a      │ 2024-02-07 01:00:00 │ B        │          1 │ a         │ 2024-02-07 01:03:00 │           1 │           1 │
│ A     │       1 │ a      │ 2024-02-07 01:05:00 │ B        │          1 │ a         │ 2024-02-07 01:06:00 │           3 │           1 │
│ A     │       1 │ a      │ 2024-02-07 01:08:00 │ B        │          1 │ a         │ 2024-02-07 01:14:00 │           4 │           1 │
└───────┴─────────┴────────┴─────────────────────┴──────────┴────────────┴───────────┴─────────────────────┴─────────────┴─────────────┘
```

### 2. 不同筛选条件的相同事件

```sql
SELECT * FROM (
	SELECT *,
	any(t2.time) OVER (PARTITION BY user_id ORDER BY time ASC Rows BETWEEN 1 PRECEDING AND CURRENT ROW) AS pre_row_time,
	row_number() OVER(PARTITION BY user_id ORDER BY time) AS row_number1,
	row_number() OVER(PARTITION BY t2.time ORDER BY time) AS row_number2 FROM (
	  SELECT * FROM interval_test WHERE event = 'A' AND group0 = 'b'
	) AS t1 ASOF JOIN (
	  SELECT * FROM interval_test WHERE event = 'A'
	) AS t2 ON t1.user_id = t2.user_id AND t1.time < t2.time
) WHERE row_number2 = 1 AND (time != pre_row_time OR row_number1 % 2 = 1);
```

查询结果
```
┌─event─┬─user_id─┬─group0─┬────────────────time─┬─t2.event─┬─t2.user_id─┬─t2.group0─┬─────────────t2.time─┬────────pre_row_time─┬─row_number1─┬─row_number2─┐
│ A     │       1 │ b      │ 2024-02-07 01:02:00 │ A        │          1 │ b         │ 2024-02-07 01:05:00 │ 2024-02-07 01:05:00 │           1 │           1 │
│ A     │       1 │ b      │ 2024-02-07 01:08:00 │ A        │          1 │ b         │ 2024-02-07 01:09:00 │ 2024-02-07 01:08:00 │           3 │           1 │
└───────┴─────────┴────────┴─────────────────────┴──────────┴────────────┴───────────┴─────────────────────┴─────────────────────┴─────────────┴─────────────┘
```

---

```sql
SELECT * FROM (
	SELECT *,
	any(t2.time) OVER (PARTITION BY user_id ORDER BY time ASC Rows BETWEEN 1 PRECEDING AND CURRENT ROW) AS pre_row_time,
	row_number() OVER(PARTITION BY user_id ORDER BY time) AS row_number1,
	row_number() OVER(PARTITION BY t2.time ORDER BY time) AS row_number2 FROM (
	  SELECT * FROM interval_test WHERE event = 'A' AND group0 = 'a'
	) AS t1 ASOF JOIN (
	  SELECT * FROM interval_test WHERE event = 'A'
	) AS t2 ON t1.user_id = t2.user_id AND t1.time < t2.time
) WHERE row_number2 = 1 AND (time != pre_row_time OR row_number1 % 2 = 1);
```

查询结果：
```
┌─event─┬─user_id─┬─group0─┬────────────────time─┬─t2.event─┬─t2.user_id─┬─t2.group0─┬─────────────t2.time─┬────────pre_row_time─┬─row_number1─┬─row_number2─┐
│ A     │       1 │ a      │ 2024-02-07 01:00:00 │ A        │          1 │ b         │ 2024-02-07 01:02:00 │ 2024-02-07 01:02:00 │           1 │           1 │
│ A     │       1 │ a      │ 2024-02-07 01:10:00 │ A        │          1 │ a         │ 2024-02-07 01:13:00 │ 2024-02-07 01:02:00 │           2 │           1 │
└───────┴─────────┴────────┴─────────────────────┴──────────┴────────────┴───────────┴─────────────────────┴─────────────────────┴─────────────┴─────────────┘
```

---

```sql
SELECT * FROM (
	SELECT *,
	any(t2.time) OVER (PARTITION BY user_id ORDER BY time ASC Rows BETWEEN 1 PRECEDING AND CURRENT ROW) AS pre_row_time,
	row_number() OVER(PARTITION BY user_id ORDER BY time) AS row_number1,
	row_number() OVER(PARTITION BY t2.time ORDER BY time) AS row_number2 FROM (
	  SELECT * FROM interval_test WHERE event = 'A' AND group0 = 'b'
	) AS t1 ASOF JOIN (
	  SELECT * FROM interval_test WHERE event = 'A' AND group0 = 'a'
	) AS t2 ON t1.user_id = t2.user_id AND t1.time < t2.time
) WHERE row_number2 = 1 AND (time != pre_row_time OR row_number1 % 2 = 1);
```

查询结果
```
┌─event─┬─user_id─┬─group0─┬────────────────time─┬─t2.event─┬─t2.user_id─┬─t2.group0─┬─────────────t2.time─┬────────pre_row_time─┬─row_number1─┬─row_number2─┐
│ A     │       1 │ b      │ 2024-02-07 01:02:00 │ A        │          1 │ a         │ 2024-02-07 01:10:00 │ 2024-02-07 01:10:00 │           1 │           1 │
└───────┴─────────┴────────┴─────────────────────┴──────────┴────────────┴───────────┴─────────────────────┴─────────────────────┴─────────────┴─────────────┘
```


#### 3. 相同筛选条件的相同事件
```sql
SELECT * FROM (
	SELECT *,
	row_number() OVER(PARTITION BY user_id ORDER BY time) AS row_number1,
	row_number() OVER(PARTITION BY t2.time ORDER BY time) AS row_number2 FROM (
	  SELECT * FROM interval_test WHERE event = 'A'
	) AS t1 ASOF JOIN (
	  SELECT * FROM interval_test WHERE event = 'A'
	) AS t2 ON t1.user_id = t2.user_id AND t1.time < t2.time
) WHERE row_number1 % 2 = 1 AND row_number2 = 1;
```

查询结果
```
┌─event─┬─user_id─┬─group0─┬────────────────time─┬─t2.event─┬─t2.user_id─┬─t2.group0─┬─────────────t2.time─┬─row_number1─┬─row_number2─┐
│ A     │       1 │ a      │ 2024-02-07 01:00:00 │ A        │          1 │ a         │ 2024-02-07 01:02:00 │           1 │           1 │
│ A     │       1 │ a      │ 2024-02-07 01:05:00 │ A        │          1 │ a         │ 2024-02-07 01:08:00 │           3 │           1 │
│ A     │       1 │ a      │ 2024-02-07 01:09:00 │ A        │          1 │ a         │ 2024-02-07 01:10:00 │           5 │           1 │
└───────┴─────────┴────────┴─────────────────────┴──────────┴────────────┴───────────┴─────────────────────┴─────────────┴─────────────┘
```

### 问题

虽然使用 [ClickHouse ASOF JOIN Usage](https://clickhouse.com/docs/en/sql-reference/statements/select/join#asof-join-usage) 算是可以实现间隔分析，
在`相同筛选条件的相同事件`的场景下需要对相邻两条结果数据取第一条，这个还算可以解决，
但在 `不同筛选条件的相同事件`的场景下是感觉不是很好无法区分结果是等价于`不同事件` 还是 `相同筛选条件的相同事件`。