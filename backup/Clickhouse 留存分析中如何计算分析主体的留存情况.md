## 背景

先明确留存分析相关的概念，
**留存分析**：衡量不同批次新增用户的数据表现，最常见的就是留存率
**分析主体**：触发事件的主体，比如用户
**初始事件**：代表新增的事件，比如用户注册
**回访事件**：代表活跃的事件，比如用户登录
**回访用户同时参与事件**：代表活跃的事件，比如用户充值

## 需求

产品给出的关于留存定义为：**某天完成了初始事件的分析主体，在初始事件日期后的第 1 日至第 N 日有完成“回访事件”，即为第 N 日的“留存用户”**。
比如用户在 `2024-08-10` 进行了注册，在 `2024-08-10`, `2024-08-11`, `2024-08-15` 进行了登录，那么我们就说该用户是第 0，1，5 日的留存用户。

> 目前针对当日(0 日)的留存情况，不去对比初始事件和回访事件的发生的先后顺序。

### 实现思路

1. 计算每个分析主体触发的回访事件的日期集合 和 回访用户同时参与事件的日期和聚合值的集合
2. 计算每个分析主体在指定天数下的留存情况和回访用户同时参与事件的聚合值情况
3. 计算每个初始事件触发日期下不同天数的留存情况

如果使用 Clikhouse 提供的 [留存聚合函数](https://clickhouse.com/docs/en/sql-reference/aggregate-functions/parametric-functions#retention) 最多只支持 32 个条件参数，无法计算任意天数的留存情况，需要注意。

### 示例代码

```sql
--- 建表语句
CREATE TABLE IF NOT EXISTS retention_test (
  time DateTime,
  event String,
  user_id UInt64
) ENGINE = Memory;


--- 插入数据
WITH range(-1, 7) AS date_interval_nums, arrayEnumerate(date_interval_nums) AS date_interval_nums_enumerate
SELECT init_date,
      groupArray(r) AS retention_data_list,
      groupArray(a) AS base_retention_amount_list,
      arrayMap(i -> arraySum(arrayMap(x -> x[i], retention_data_list)), range(1, length(retention_data_list[1])+1)) AS retention_data,
      arrayMap(i -> round(arraySum(arrayMap(x -> x[i], base_retention_amount_list)), 2), range(1, length(base_retention_amount_list[1])+1)) AS retention_amount_data
  FROM (
    SELECT t1.init_date AS init_date, t1.user_id AS user_id, return_dates, mapFromArrays(return_dates, arrayWithConstant(length(return_dates), 1)) AS return_dates_map,
      arrayMap(x -> if(x == -1, 1, mapContains(return_dates_map, date_add(DAY, x, init_date))), date_interval_nums) AS r,
      arrayMap(x -> multiIf(x == 1, 0, r[x], date_amounts_map[date_add(DAY, date_interval_nums[x], init_date)], 0), date_interval_nums_enumerate) AS a
      FROM (
      SELECT DISTINCT ON (time, user_id) date_trunc('day', time) AS init_date, user_id
        FROM retention_test
       WHERE event = 'user_create'
       ORDER BY time
    ) AS t1 GLOBAL LEFT JOIN (
      SELECT user_id, arraySort(groupUniqArray(time)) AS return_dates
        FROM retention_test
       WHERE event = 'user_login'
       GROUP BY 1
    ) AS t2 ON t1.user_id = t2.user_id
    GLOBAL LEFT JOIN (
      SELECT user_id, cast((groupArray(date_time), groupArray(amount)) AS Map(Datetime, Float64)) AS date_amounts_map
       FROM (
        SELECT user_id, date_trunc('day', time) AS date_time, COUNT(1) AS amount
          FROM retention_test
        WHERE event = 'user_login'
        GROUP BY 1, 2
      ) GROUP BY 1
    ) AS t3 ON t1.user_id = t3.user_id
  )
  GROUP BY 1
  ORDER BY 1;


-- 查询结果 
┌───────────init_date─┬─retention_data_list─────────────────────────────────────┬─base_retention_amount_list──────────────────────────────┬─retention_data────┬─retention_amount_data─┐
│ 2024-08-01 00:00:00 │ [[1,1,1,0,0,1,0,0],[1,0,1,0,0,0,0,0],[1,0,0,0,0,0,0,0]] │ [[0,1,1,0,0,3,0,0],[0,0,1,0,0,0,0,0],[0,0,0,0,0,0,0,0]] │ [3,1,2,0,0,1,0,0] │ [0,1,2,0,0,3,0,0]     │
│ 2024-08-02 00:00:00 │ [[1,0,1,0,0,1,0,0]]                                     │ [[0,0,1,0,0,1,0,0]]                                     │ [1,0,1,0,0,1,0,0] │ [0,0,1,0,0,1,0,0]     │
└─────────────────────┴─────────────────────────────────────────────────────────┴─────────────────────────────────────────────────────────┴───────────────────┴───────────────────────┘
```