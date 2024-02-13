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

### 如何用 ClickHouse 实现查询

参照了以下两个文章，
1. [https://www.jianshu.com/p/dd8de7c795c6](https://www.jianshu.com/p/dd8de7c795c6)
2. [https://www.jianshu.com/p/65f3df30d174](https://www.jianshu.com/p/65f3df30d174)

实现了第一种写法，

```sql
SELECT `$event_traces`, COUNT(1) AS `$session_count` FROM (
  WITH arraySort(x -> x.1, groupArray((`#time`, array(`$event_name`, `$group0`)))) AS `$sorted_events`,
  arrayEnumerate(`$sorted_events`) AS `$event_idxs`,
  arrayFilter((x, y, z) -> (y > 1800000), `$event_idxs`, arrayDifference(`$sorted_events`.1), `$sorted_events`
  ) AS `$temp_gap_idxs`,
  arrayMap(x -> x + 1, `$temp_gap_idxs`) AS `$gap_idxs`,
  arrayMap(x -> if(has(`$gap_idxs`, x), 1, 0), `$event_idxs`) AS `$gap_masks`,
  arraySplit((x, y) -> y, `$sorted_events`, `$gap_masks`) AS `$split_events`
    SELECT `$virtual_user_id`,
           arrayJoin(`$split_events`) AS `$temp_event_traces`,
          `$temp_event_traces`.2 AS `$event_traces`
      FROM (
          SELECT *, multiIf((`#event` IN ('#user_create')), '#user_create', (`#event` IN ('#user_login')), '#user_login', (`#event` IN ('#user_logout')), '#user_logout', (`#event` IN ('#user_new')), '#user_new', (`#event` IN ('#charge')), '#charge', '') AS `$event_name`, '' AS `$group0`, `#user_id` AS `$virtual_user_id`
            FROM (
              SELECT `#time`, `#data_lifecycle`, `#dt`, `#event`, `#user_id`
              FROM events AS event
              WHERE (`#data_lifecycle` = '0')
              AND (`#dt` BETWEEN '2023-10-14' AND '2023-10-20')
              AND ((`#event` IN ('#user_create')) OR (`#event` IN ('#user_login')) OR (`#event` IN ('#user_logout')) OR (`#event` IN ('#user_new')) OR (`#event` IN ('#charge'))))
        )
    WHERE `$virtual_user_id` != ''
    GROUP BY `$virtual_user_id`
    HAVING length(`$event_traces`) >= 1
    )
    WHERE `$event_traces`[1][1] = '#user_create'
GROUP BY `$event_traces`
ORDER BY `$session_count` DESC
LIMIT 1000000;
```

自己思考后实现了第二种写法，

```sql
SELECT `$event_traces`, COUNT(1) AS `$session_count` FROM (
  SELECT `$virtual_user_id`,
        groupArray((`#time`, array(`$event_name`, `$group0`))) AS `$sorted_events`,
        groupArray(`$split_flag`) AS `$gap_masks`,
        arraySplit((x, y) -> y, `$sorted_events`, `$gap_masks`) AS `$split_events`,
        arrayJoin(`$split_events`) AS `$temp_event_traces`,
        `$temp_event_traces`.2 AS `$event_traces`
   FROM (
        SELECT *, any(`#time`) OVER (PARTITION BY `$virtual_user_id` ORDER BY `#time` rows between 1 preceding and 1 preceding) AS `$previous_time`, if(`#time` - `$previous_time` > 1800000, 1, 0) AS `$split_flag`
        FROM (
          SELECT *, multiIf((`#event` IN ('#user_create')), '#user_create', (`#event` IN ('#user_login')), '#user_login', (`#event` IN ('#user_logout')), '#user_logout', (`#event` IN ('#user_new')), '#user_new', (`#event` IN ('#charge')), '#charge', '') AS `$event_name`, '' AS `$group0`, `#user_id` AS `$virtual_user_id`
          FROM (
            SELECT `#time`, `#data_lifecycle`, `#dt`, `#event`, `#user_id`
            FROM events AS event
            WHERE (`#data_lifecycle` = '0')
            AND (`#dt` BETWEEN '2023-10-14' AND '2023-10-19')
            AND ((`#event` IN ('#user_create')) OR (`#event` IN ('#user_login')) OR (`#event` IN ('#user_logout')) OR (`#event` IN ('#user_new')) OR (`#event` IN ('#charge'))))
            ORDER BY `#time`
        )
      ) WHERE `$virtual_user_id` != ''
      GROUP BY `$virtual_user_id`
      HAVING length(`$event_traces`) >= 1
    )
  WHERE `$event_traces`[1][1] = '#user_login'
GROUP BY `$event_traces`
ORDER BY `$session_count` DESC
LIMIT 1000000;
```