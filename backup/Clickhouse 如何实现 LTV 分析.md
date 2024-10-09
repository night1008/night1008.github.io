### 背景

LTV 是 Life Time Value 缩写，即用户在生命周期中贡献的商业价值。
LTV 分析是一种分析用户商业价值的分析模型，可分析特定日期访问的用户群体，在一定时长内贡献的人均价值。
LTV 分析可以帮助回答以下问题：
- 广告投放的效果：哪个渠道来的用户带来的人均价值更高？用户贡献的价值是否高于用户获取的成本？
- 市场营销活动的效果：运营活动是否带来了显著的收益？用户的人均价值是否有所提升？运营活动到底是好还是坏？
- 产品迭代和功能优化：游戏产品是否能够带来足够的人均消费来覆盖开发成本？玩家价值会在多久之后趋于稳定？
- 用户细分：哪个用户群体是我们产品的高价值人群？这些用户有怎么样的特征？

现有的留存分析也能计算累计人均值，为什么要独立设计开发LTV 分析功能？
LTV 分析计算的是累计人均值。留存分析中的同时显示功能，可以计算阶段累计人均值，也能用于计算LTV。但两者计算口径上有细微的区别，共有三个差异点：
1. 初始用户群体的不同点：留存分析中，只要用户发生了初始事件，就会计入到对应的初始日期中。LTV 分析中一个人有多次初始事件时，这个用户只会计入初始事件发生最早的那天。
2. 留存用户群体的不同点：留存分析中计算的是留存用户的累计人均值。当留存分析中的后续事件不是营收事件时，计算的留存用户群体和LTV 分析不相同。LTV 分析中计算的初始用户中发生营收事件的用户。
3. 首日事件发生的先后顺序：留存分析中，第 0 日/周/月的计算中，是不区分初始化行为和后续行为的先后顺序的，因此当日后续行为早于初始行为，也会计入留存用户群体。LTV 分析中，只有营收事件不早于初始事件才会算进营收。


下面介绍各个 LTV 模型如何实现。

先建立 LTV 查询测试表。

```sql
-- 建表语句
CREATE TABLE IF NOT EXISTS ltv_test (
  time DateTime,
  event String,
  user_id UInt64,
  amount Int64
) ENGINE = Memory;


--- 插入数据
INSERT INTO ltv_test VALUES
('2024-08-01 00:00:00', 'user_create', 1, 0),
('2024-08-01 01:00:00', 'user_create', 1, 0),
('2024-08-01 00:00:00', 'user_login', 1, 1),
('2024-08-02 00:00:00', 'user_login', 1, 2),
('2024-08-05 00:00:00', 'user_login', 1, 3),
('2024-08-05 00:00:00', 'user_login', 1, 4),
('2024-08-05 00:00:00', 'user_login', 1, 5),
('2024-08-01 00:00:00', 'user_create', 2, 0),
('2024-08-02 00:00:00', 'user_login', 2, 1),
('2024-08-10 00:00:00', 'user_login', 2, 2),
('2024-08-02 00:00:00', 'user_create', 3, 0),
('2024-08-03 00:00:00', 'user_login', 3, 1),
('2024-08-06 00:00:00', 'user_login', 3, 2),
('2024-08-10 00:00:00', 'user_login', 3, 3),
('2024-08-01 00:00:00', 'user_create', 4, 0);
```

### ARPU (阶段累计人均值)

目的：反映公司从每位用户那里获得的平均收入，可以更好的理解用户价值。
概念：ARPU（Average Revenue Per User），即平均每用户营收，指的是用户首次行为发生后的 N 天内，所有初始用户所产生的总营收除以初始用户人数的平均值。
计算公式：ARPU = 总营收 / 初始用户人数。(总营收：用户在 N 天内产生的所有营收总和 | 初始用户人数：产生初始事件行为的用户总数)

实现思路：
1. 计算每个分析主体触发的回访事件的日期和聚合值的集合
2. 计算每个分析主体在 N 天内的累计总和
3. 计算每个分析主体在 N 天内的阶段累计总和
4. 计算每个初始事件触发日期下的 N 天阶段累计总和除以触发初始事件的用户总数

示例代码：
```sql
-- ARPU (阶段累计人均值)
WITH range(-1, 7) AS date_interval_nums, arrayEnumerate(date_interval_nums) AS date_interval_nums_enumerate
SELECT init_date, ltv_data,
      arrayMap(x -> round(x / if(ltv_data[1] = 0, 1, ltv_data[1]), 2), arrayCumSum(base_ltv_amount)) AS ltv_amount,
      arrayCumSum(base_ltv_amount) AS total_ltv_amount
  FROM (
  SELECT init_date,
        groupArray(r) AS ltv_data_list,
        groupArray(a) AS base_ltv_amount_list,
        arrayMap(i -> arraySum(arrayMap(x -> x[i], ltv_data_list)), range(1, length(ltv_data_list[1])+1)) AS ltv_data,
        arrayMap(i -> round(arraySum(arrayMap(x -> x[i], base_ltv_amount_list)), 2), range(1, length(base_ltv_amount_list[1])+1)) AS base_ltv_amount
    FROM (
      SELECT init_date, user_id,
        cast((groupArray(date_time), groupArray(amount)) AS Map(Datetime, Float64)) AS date_amounts_map,
        arrayMap(x -> if(x == -1, 1, mapContains(date_amounts_map, date_add(DAY, x, init_date))), date_interval_nums) AS r,
        arrayMap(x -> multiIf(x == 1, 0, r[x], date_amounts_map[date_add(DAY, date_interval_nums[x], init_date)], 0), date_interval_nums_enumerate) AS a
        FROM (
          SELECT init_date, user_id, date_trunc('day', coalesce(t2.time, toDateTime(0))) AS date_time, toFloat64(coalesce(SUM(t2.value), 0))  AS amount
            FROM (
            SELECT DISTINCT ON (user_id) time AS init_date, user_id, time
              FROM ltv_test
            WHERE event = 'user_create'
            ORDER BY time
          ) AS t1 GLOBAL LEFT JOIN (
            SELECT user_id, time, amount AS value
              FROM ltv_test
              WHERE event = 'user_login'
          ) AS t2 ON t1.user_id = t2.user_id WHERE t1.time < t2.time OR t2.time IS NULL
          GROUP BY 1, 2, 3
        )
        GROUP BY 1, 2
    )
    GROUP BY 1
  )
  ORDER BY 1 SETTINGS join_use_nulls=1;

┌───────────init_date─┬─ltv_data──────────┬─ltv_amount────────┬─total_ltv_amount─────┐
│ 2024-08-01 00:00:00 │ [3,0,2,0,0,1,0,0] │ [0,0,1,1,1,5,5,5] │ [0,0,3,3,3,15,15,15] │
│ 2024-08-02 00:00:00 │ [1,0,1,0,0,1,0,0] │ [0,0,1,1,1,3,3,3] │ [0,0,1,1,1,3,3,3]    │
└─────────────────────┴───────────────────┴───────────────────┴──────────────────────┘
```

---

### 总营收（阶段累计总和）

目的：用户营收追踪，全面的评估用户在特定时间段内对业务的贡献。
概念：总营收是指用户首次行为发生后的 N 天内，所有初始用户产生的总收入总额（N 天的累计值）。
计算公式：用户在观测窗口期 N 天内产生的所有营收的总和。（累计总营收 = ∑(用户营收事件)）

实现思路：
1. 计算每个分析主体触发的回访事件的日期和聚合值的集合
2. 计算每个分析主体在 N 天内的累计总和
3. 计算每个初始事件触发日期下的 N 天的累计总和
4. 计算每个初始事件触发日期下的 N 天的阶段累计总和

示例代码：
```sql
-- 总营收 (阶段累计总和)
WITH range(-1, 7) AS date_interval_nums, arrayEnumerate(date_interval_nums) AS date_interval_nums_enumerate
SELECT init_date, ltv_data,
      arrayCumSum(base_ltv_amount) AS ltv_amount,
      arrayCumSum(base_ltv_amount) AS total_ltv_amount
  FROM (
  SELECT init_date,
        groupArray(r) AS ltv_data_list,
        groupArray(a) AS base_ltv_amount_list,
        arrayMap(i -> arraySum(arrayMap(x -> x[i], ltv_data_list)), range(1, length(ltv_data_list[1])+1)) AS ltv_data,
        arrayMap(i -> round(arraySum(arrayMap(x -> x[i], base_ltv_amount_list)), 2), range(1, length(base_ltv_amount_list[1])+1)) AS base_ltv_amount
    FROM (
      SELECT init_date, user_id,
        cast((groupArray(date_time), groupArray(amount)) AS Map(Datetime, Float64)) AS date_amounts_map,
        arrayMap(x -> if(x == -1, 1, mapContains(date_amounts_map, date_add(DAY, x, init_date))), date_interval_nums) AS r,
        arrayMap(x -> multiIf(x == 1, 0, r[x], date_amounts_map[date_add(DAY, date_interval_nums[x], init_date)], 0), date_interval_nums_enumerate) AS a
        FROM (
          SELECT init_date, user_id, date_trunc('day', coalesce(t2.time, toDateTime(0))) AS date_time, toFloat64(coalesce(SUM(t2.value), 0))  AS amount
            FROM (
            SELECT DISTINCT ON (user_id) time AS init_date, user_id, time
              FROM ltv_test
            WHERE event = 'user_create'
            ORDER BY time
          ) AS t1 GLOBAL LEFT JOIN (
            SELECT user_id, time, amount AS value
              FROM ltv_test
              WHERE event = 'user_login'
          ) AS t2 ON t1.user_id = t2.user_id WHERE t1.time < t2.time OR t2.time IS NULL
          GROUP BY 1, 2, 3
        )
        GROUP BY 1, 2
    )
    GROUP BY 1
  )
  ORDER BY 1 SETTINGS join_use_nulls=1;

┌───────────init_date─┬─ltv_data──────────┬─ltv_amount───────────┬─total_ltv_amount─────┐
│ 2024-08-01 00:00:00 │ [3,0,2,0,0,1,0,0] │ [0,0,3,3,3,15,15,15] │ [0,0,3,3,3,15,15,15] │
│ 2024-08-02 00:00:00 │ [1,0,1,0,0,1,0,0] │ [0,0,1,1,1,3,3,3]    │ [0,0,1,1,1,3,3,3]    │
└─────────────────────┴───────────────────┴──────────────────────┴──────────────────────┘
```

---

### 日营收（单日营收汇总）

目的：了解用户在完成初始事件行为的商业价值，帮助衡量单一时间点的用户贡献，评估市场策略和用户忠诚度。
概念：日营收是指在初始行为后的第 N 日，用户当天发生的所有营收事件的总价值（第 N 日当日的累计总和）
计算公式：总营收 = 第 N 日用户营收事件汇总

实现思路：
1. 计算每个分析主体触发的回访事件的日期和聚合值的集合
2. 计算每个分析主体在 N 天内的累计总和
3. 计算每个初始事件触发日期下的 N 天的累计总和

示例代码：
```sql
-- 日营收 (单日营收汇总)
WITH range(-1, 7) AS date_interval_nums, arrayEnumerate(date_interval_nums) AS date_interval_nums_enumerate
SELECT init_date, ltv_data,
      base_ltv_amount AS ltv_amount,
      base_ltv_amount AS total_ltv_amount
  FROM (
  SELECT init_date,
        groupArray(r) AS ltv_data_list,
        groupArray(a) AS base_ltv_amount_list,
        arrayMap(i -> arraySum(arrayMap(x -> x[i], ltv_data_list)), range(1, length(ltv_data_list[1])+1)) AS ltv_data,
        arrayMap(i -> round(arraySum(arrayMap(x -> x[i], base_ltv_amount_list)), 2), range(1, length(base_ltv_amount_list[1])+1)) AS base_ltv_amount
    FROM (
      SELECT init_date, user_id,
        cast((groupArray(date_time), groupArray(amount)) AS Map(Datetime, Float64)) AS date_amounts_map,
        arrayMap(x -> if(x == -1, 1, mapContains(date_amounts_map, date_add(DAY, x, init_date))), date_interval_nums) AS r,
        arrayMap(x -> multiIf(x == 1, 0, r[x], date_amounts_map[date_add(DAY, date_interval_nums[x], init_date)], 0), date_interval_nums_enumerate) AS a
        FROM (
          SELECT init_date, user_id, date_trunc('day', coalesce(t2.time, toDateTime(0))) AS date_time, toFloat64(coalesce(SUM(t2.value), 0))  AS amount
            FROM (
            SELECT DISTINCT ON (user_id) time AS init_date, user_id, time
              FROM ltv_test
            WHERE event = 'user_create'
            ORDER BY time
          ) AS t1 GLOBAL LEFT JOIN (
            SELECT user_id, time, amount AS value
              FROM ltv_test
              WHERE event = 'user_login'
          ) AS t2 ON t1.user_id = t2.user_id WHERE t1.time < t2.time OR t2.time IS NULL
          GROUP BY 1, 2, 3
        )
        GROUP BY 1, 2
    )
    GROUP BY 1
  )
  ORDER BY 1 SETTINGS join_use_nulls=1;

┌───────────init_date─┬─ltv_data──────────┬─ltv_amount─────────┬─total_ltv_amount───┐
│ 2024-08-01 00:00:00 │ [3,0,2,0,0,1,0,0] │ [0,0,3,0,0,12,0,0] │ [0,0,3,0,0,12,0,0] │
│ 2024-08-02 00:00:00 │ [1,0,1,0,0,1,0,0] │ [0,0,1,0,0,2,0,0]  │ [0,0,1,0,0,2,0,0]  │
└─────────────────────┴───────────────────┴────────────────────┴────────────────────┘
```

---

### 日营收人数

目的：用户营收参与度分析，观测在不同的时间节点上，用户群体的参与度和活跃度。
概念：日营收人数是指在初始行为后的第 N 日，所有进行了营收行为的用户的总数（第 N 日）
计算公式：营收人数：第 N 天发生营收事件行为的用户属性进行汇总，以得出该日的营收人数。

实现思路：
1. 计算每个分析主体触发的回访事件的日期集合
2. 计算每个初始事件触发日期下的日营收人数
3. 计算每个初始事件触发日期下的 N 天的日营收人数总和

示例代码：
```sql
-- 日营收人数
WITH range(-1, 7) AS date_interval_nums, arrayEnumerate(date_interval_nums) AS date_interval_nums_enumerate
SELECT init_date, ltv_data,
      base_ltv_amount AS ltv_amount,
      base_ltv_amount AS total_ltv_amount
  FROM (
  SELECT init_date,
        groupArray(r) AS ltv_data_list,
        groupArray(a) AS base_ltv_amount_list,
        arrayMap(i -> arraySum(arrayMap(x -> x[i], ltv_data_list)), range(1, length(ltv_data_list[1])+1)) AS ltv_data,
        arrayMap(i -> round(arraySum(arrayMap(x -> x[i], base_ltv_amount_list)), 2), range(1, length(base_ltv_amount_list[1])+1)) AS base_ltv_amount
    FROM (
      SELECT init_date, user_id,
        cast((groupArray(date_time), groupArray(amount)) AS Map(Datetime, Float64)) AS date_amounts_map,
        arrayMap(x -> if(x == -1, 1, mapContains(date_amounts_map, date_add(DAY, x, init_date))), date_interval_nums) AS r,
        arrayMap(x -> multiIf(x == 1, 0, r[x], date_amounts_map[date_add(DAY, date_interval_nums[x], init_date)], 0), date_interval_nums_enumerate) AS a
        FROM (
          SELECT init_date, user_id, date_trunc('day', coalesce(t2.time, toDateTime(0))) AS date_time, toFloat64(coalesce(any(1), 0)) AS amount
            FROM (
            SELECT DISTINCT ON (user_id) time AS init_date, user_id, time
              FROM ltv_test
            WHERE event = 'user_create'
            ORDER BY time
          ) AS t1 GLOBAL LEFT JOIN (
            SELECT user_id, time, 1 AS value
              FROM ltv_test
              WHERE event = 'user_login'
          ) AS t2 ON t1.user_id = t2.user_id WHERE t1.time < t2.time OR t2.time IS NULL
          GROUP BY 1, 2, 3
        )
        GROUP BY 1, 2
    )
    GROUP BY 1
  )
  ORDER BY 1 SETTINGS join_use_nulls=1;

┌───────────init_date─┬─ltv_data──────────┬─ltv_amount────────┬─total_ltv_amount──┐
│ 2024-08-01 00:00:00 │ [3,0,2,0,0,1,0,0] │ [0,0,2,0,0,1,0,0] │ [0,0,2,0,0,1,0,0] │
│ 2024-08-02 00:00:00 │ [1,0,1,0,0,1,0,0] │ [0,0,1,0,0,1,0,0] │ [0,0,1,0,0,1,0,0] │
└─────────────────────┴───────────────────┴───────────────────┴───────────────────┘
```

---

### ARPPU (阶段人均营收贡献)

目的：用户平均营收贡献分析，反映付费用户的平均消费水平。
概念：ARPPU（Average Revenue Per Paying User），即平均付费用户营收，指的是用户首次行为发生后的第 N 天内，仅针对那些实际产生了营收的用户，总营收除以这些付费用户人数所得的平均值。
计算公式：累计营收人均价值 = 累计总营收 / 累计营收人数 （累计总营收：用户在观测窗口期 N 天内产生的所有营收的总和 ｜ 累计营收人数：在观测窗口期 N 天内至少发生一次营收事件行为的用户总数）

实现思路：
1. 计算每个分析主体触发的回访事件的日期和聚合值的集合
2. 计算每个分析主体在 N 天内的累计总和
3. 计算每个分析主体在 N 天内的阶段累计总和
4. 计算每个初始事件触发日期下的营收人数的阶段累计总和
5. 计算每个初始事件触发日期下的 N 天阶段累计总和除以营收人数的阶段累计总和

示例代码：
```sql
-- ARPPU (阶段人均营收贡献)
WITH range(-1, 7) AS date_interval_nums, arrayEnumerate(date_interval_nums) AS date_interval_nums_enumerate
SELECT init_date, ltv_data,
      base_ltv_amount AS ltv_amount,
      base_ltv_amount AS total_ltv_amount
  FROM (
  SELECT init_date,
        groupArray(r) AS r_list,
        groupArray(a) AS a_list,
        arrayMap(x -> arrayMap((i, y) -> multiIf(i == 1, 1, y >= 1, 1, 0), date_interval_nums_enumerate, x), arrayMap(i -> arrayCumSum(x -> if(x == 1, 0, i[x]), date_interval_nums_enumerate), r_list)) AS r_list_cum_sum,
        arrayMap(i -> arraySum(arrayMap(x -> x[i], r_list_cum_sum)), range(1, length(r_list[1])+1)) AS ltv_data_all,
        arrayMap(i -> round(arraySum(arrayMap(x -> x[i], a_list)), 2), range(1, length(a_list[1])+1)) AS base_ltv_amount_all
    FROM (
      SELECT init_date, user_id,
        cast((groupArray(date_time), groupArray(amount)) AS Map(Datetime, Float64)) AS date_amounts_map,
        arrayMap(x -> if(x == -1, 1, mapContains(date_amounts_map, date_add(DAY, x, init_date))), date_interval_nums) AS r,
        arrayMap(x -> multiIf(x == 1, 0, r[x], date_amounts_map[date_add(DAY, date_interval_nums[x], init_date)], 0), date_interval_nums_enumerate) AS a
        FROM (
          SELECT init_date, user_id, date_trunc('day', coalesce(t2.time, toDateTime(0))) AS date_time, toFloat64(coalesce(SUM(t2.value), 0))  AS amount
            FROM (
            SELECT DISTINCT ON (user_id) time AS init_date, user_id, time
              FROM ltv_test
            WHERE event = 'user_create'
            ORDER BY time
          ) AS t1 GLOBAL LEFT JOIN (
            SELECT user_id, time, amount AS value
              FROM ltv_test
              WHERE event = 'user_login'
          ) AS t2 ON t1.user_id = t2.user_id WHERE t1.time < t2.time OR t2.time IS NULL
          GROUP BY 1, 2, 3
        )
        GROUP BY 1, 2
    )
    GROUP BY 1
  )
  ORDER BY 1 SETTINGS join_use_nulls=1;

┌───────────init_date─┬─ltv_data──────────┬─ltv_amount─────────┬─total_ltv_amount───┐
│ 2024-08-01 00:00:00 │ [3,0,2,0,0,1,0,0] │ [0,0,3,0,0,12,0,0] │ [0,0,3,0,0,12,0,0] │
│ 2024-08-02 00:00:00 │ [1,0,1,0,0,1,0,0] │ [0,0,1,0,0,2,0,0]  │ [0,0,1,0,0,2,0,0]  │
└─────────────────────┴───────────────────┴────────────────────┴────────────────────┘
```

---

### 日新增营收人数

目的：用户营收激活分析。
概念：日新增营收人数是指在初始行为后的 N 天内，那些 0 到 N-1 日中未产生过营收，但在第 N 日首次进行消费的用户总数（第 N 日）。
计算公式：日新增营收人数 = 在观测窗口期第 N 天新增营收事件行为的用户总数）

实现思路：
1. 计算每个分析主体首次触发的回访事件的日期
2. 计算每个分析主体首次触发的回访事件的日期与初始事件的间隔日期
3. 计算每个初始事件触发日期下的日新增营收人数

示例代码：
```sql
-- 日新增营收人数
WITH range(-1, 7) AS date_interval_nums, arrayEnumerate(date_interval_nums) AS date_interval_nums_enumerate
SELECT init_date, ltv_data,
      base_ltv_amount AS ltv_amount,
      base_ltv_amount AS total_ltv_amount
  FROM (
  SELECT init_date,
        groupArray(r) AS ltv_data_list,
        groupArray(a) AS base_ltv_amount_list,
        arrayMap(i -> arraySum(arrayMap(x -> x[i], ltv_data_list)), range(1, length(ltv_data_list[1])+1)) AS ltv_data,
        arrayMap(i -> round(arraySum(arrayMap(x -> x[i], base_ltv_amount_list)), 2), range(1, length(base_ltv_amount_list[1])+1)) AS base_ltv_amount
    FROM (
      SELECT init_date, user_id,
        min(date_time) AS first_return_date,
        coalesce(date_diff('day', init_date, first_return_date), 99999) AS diff_return_date_count,
        arrayMap(x -> multiIf(x == -1, 1, diff_return_date_count == x, 1, 0), date_interval_nums) AS r, r AS a
        FROM (
          SELECT init_date, user_id, date_trunc('day', coalesce(t2.time, toDateTime(0))) AS date_time, toFloat64(coalesce(any(1), 0)) AS amount
            FROM (
            SELECT DISTINCT ON (user_id) time AS init_date, user_id, time
              FROM ltv_test
            WHERE event = 'user_create'
            ORDER BY time
          ) AS t1 GLOBAL LEFT JOIN (
            SELECT user_id, time, 1 AS value
              FROM ltv_test
              WHERE event = 'user_login'
          ) AS t2 ON t1.user_id = t2.user_id WHERE t1.time < t2.time OR t2.time IS NULL
          GROUP BY 1, 2, 3
        )
        GROUP BY 1, 2
    )
    GROUP BY 1
  )
  ORDER BY 1 SETTINGS join_use_nulls=1;

┌───────────init_date─┬─ltv_data──────────┬─ltv_amount────────┬─total_ltv_amount──┐
│ 2024-08-01 00:00:00 │ [3,0,2,0,0,0,0,0] │ [3,0,2,0,0,0,0,0] │ [3,0,2,0,0,0,0,0] │
│ 2024-08-02 00:00:00 │ [1,0,1,0,0,0,0,0] │ [1,0,1,0,0,0,0,0] │ [1,0,1,0,0,0,0,0] │
└─────────────────────┴───────────────────┴───────────────────┴───────────────────┘
```