---
layout: post
keywords: ClickHouse, 路径分析
description: 如何通过 ClickHouse 实现路径分析
title: 如何通过 ClickHouse 实现路径分析
comments: true
---

### 什么是路径分析？

[路径分析](https://manual.sensorsdata.cn/sa/latest/zh_cn/%E7%94%A8%E6%88%B7%E8%B7%AF%E5%BE%84%E5%88%86%E6%9E%90-158433692.html)主要用于分析用户在使用产品时的路径分布情况。

**例如**：用户在访问了某个电商产品首页后，有多大比例的用户进行了搜索？有多大比例的用户访问了分类页？有多大比例的用户直接访问的商品详情页？

Session 间隔为前后两个事件发生的时间间隔，超过指定的时间间隔，则为不同的路径。

### 如何用 ClickHouse 实现查询

```sql
CREATE DATABASE demo ENGINE = Memory;

CREATE TABLE trace_test (event String, user_id Int64, group0 String, time DateTime) ENGINE = Memory;

-- 插入测试数据
INSERT INTO trace_test (event, user_id, group0, time) VALUES
('A', 1, 'a', '2024-02-07 01:00:00'),
('B', 1, 'a', '2024-02-07 01:01:00'),
('C', 1, 'b', '2024-02-07 01:02:00'),
('D', 1, 'a', '2024-02-07 01:13:00'),
('A', 1, 'a', '2024-02-07 01:14:00'),
('A', 1, 'b', '2024-02-07 01:15:00'),
('B', 1, 'a', '2024-02-07 01:16:00'),
('B', 1, 'a', '2024-02-07 01:17:00'),
('A', 1, 'b', '2024-02-08 01:08:00'),
('A', 1, 'b', '2024-02-08 01:09:00'),
('A', 1, 'a', '2024-02-08 01:30:00'),
('C', 1, 'a', '2024-02-08 01:31:00'),
('D', 1, 'a', '2024-02-08 01:32:00'),
('A', 1, 'a', '2024-02-08 01:33:00'),
('B', 1, 'a', '2024-02-08 01:34:00');
```

参照了以下两个文章，
1. [https://www.jianshu.com/p/dd8de7c795c6](https://www.jianshu.com/p/dd8de7c795c6)
2. [https://www.jianshu.com/p/65f3df30d174](https://www.jianshu.com/p/65f3df30d174)

实现了第一种写法，

```sql
SELECT `$event_traces`, COUNT(1) AS `$session_count`
 FROM (
  WITH arraySort(x -> x.1, groupArray((timestamp, array(event, group0)))) AS `$sorted_events`,
      arrayEnumerate(`$sorted_events`) AS `$event_idxs`,
      arrayFilter(
        (x, y, z) -> (y > 300), `$event_idxs`,
        arrayDifference(`$sorted_events`.1),
        `$sorted_events`
      ) AS `$temp_gap_idxs`,
      arrayMap(x -> x, `$temp_gap_idxs`) AS `$gap_idxs`,
      arrayMap(x -> if(has(`$gap_idxs`, x), 1, 0), `$event_idxs`) AS `$gap_masks`,
      arraySplit((x, y) -> y, `$sorted_events`, `$gap_masks`) AS `$split_events`
  SELECT user_id,
        arrayJoin(`$split_events`) AS `$temp_event_traces`,
        `$temp_event_traces`.2 AS `$event_traces`
    FROM (
      SELECT *, toUInt64(time) AS timestamp FROM trace_test
    )
   GROUP BY user_id
   HAVING length(`$event_traces`) >= 1
)
WHERE `$event_traces`[1][1] = 'A'
GROUP BY `$event_traces`
ORDER BY `$session_count` DESC
LIMIT 1000000;
```

查询结果：
```
┌─$event_traces───────────────────────────────────────┬─$session_count─┐
│ [['A','a'],['B','a'],['C','b']]                     │              1 │
│ [['A','b'],['A','b']]                               │              1 │
│ [['A','a'],['C','a'],['D','a'],['A','a'],['B','a']] │              1 │
└─────────────────────────────────────────────────────┴────────────────┘
```

自己思考后实现了第二种写法，

```sql
SELECT `$event_traces`, COUNT(1) AS `$session_count` FROM (
  SELECT user_id,
        groupArray((timestamp, array(event, group0))) AS `$sorted_events`,
        groupArray(`$split_flag`) AS `$gap_masks`,
        arraySplit((x, y) -> y, `$sorted_events`, `$gap_masks`) AS `$split_events`,
        arrayJoin(`$split_events`) AS `$temp_event_traces`,
        `$temp_event_traces`.2 AS `$event_traces`
   FROM (
        SELECT *, any(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp rows between 1 preceding and 1 preceding) AS `$previous_time`, if(timestamp - `$previous_time` > 300, 1, 0) AS `$split_flag`
          FROM (
          SELECT *, toUInt64(time) AS timestamp FROM trace_test
          ORDER BY time
        )
      )
    GROUP BY user_id
    HAVING length(`$event_traces`) >= 1
    )
  WHERE `$event_traces`[1][1] = 'A'
GROUP BY `$event_traces`
ORDER BY `$session_count` DESC
LIMIT 1000000;
```

查询结果：
```
┌─$event_traces───────────────────────────────────────┬─$session_count─┐
│ [['A','a'],['B','a'],['C','b']]                     │              1 │
│ [['A','b'],['A','b']]                               │              1 │
│ [['A','a'],['C','a'],['D','a'],['A','a'],['B','a']] │              1 │
└─────────────────────────────────────────────────────┴────────────────┘
```