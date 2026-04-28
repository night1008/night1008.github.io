### 背景

先明确留存分析相关的概念
**留存分析**：衡量不同批次新增用户的数据表现，最常见的就是留存率
**分析主体**：触发事件的主体，比如用户
**初始事件**：代表新增的事件，比如用户注册
**回访事件**：代表活跃的事件，比如用户登录

---

### 需求

产品给出的流失定义为：某天完成了初始事件的分析主体，在初始事件日期后的第 1 日至第 N 日都没有完成“回访事件”，即为第 N 日的“流失用户”。
进一步沟通后的补充定义，当天触发的回访事件不算在流失，第 1 日至第 N 日都算该用户为“流失用户”。
比如用户在 `2024-08-10` 进行了注册，但直到 `2024-08-20` 都没有进行登录，那么我们就说该用户是 `1 ～ 10` 日的流失用户。

---

### 实现思路

1. 计算每个分析主体触发的回访事件的日期集合
2. 计算每个分析主体第一次触发回访事件的日期 (有可能分析主体根本没有触发回访事件)
3. 计算每个分析主体触发初始事件到第一次触发回访事件的事件间隔 (没有触发回访事件时以一个大数替代)
4. 计算每个初始事件触发日期下不同流失天数的分布情况 ({1: 5, 5: 10} 表示流失 1 天的分析主体有 5 个，流失 5 天的分析主体有 10 个)
5. 业务层计算第 N 日的流失数时需要进行累加分析主体数

> 这里可以衍生一个关于 Clikhouse 查询的小考题，如何在不使用 `arrayJoin` 函数的前提下，把一个 `Array(Int64)` 转换成一个 `Map(Int64, UInt64)`，比如 `[1, 1, 1, 2, 2, 2, 3, 3, 3] => {1: 3, 2: 3, 3: 3}`

---

### 示例代码

```sql
--- 建表语句
CREATE TABLE IF NOT EXISTS retention_test (
  time DateTime,
  event String,
  user_id UInt64
) ENGINE = Memory;


--- 插入数据
INSERT INTO retention_test VALUES
('2024-08-01 00:00:00', 'user_create', 1),
('2024-08-01 00:00:00', 'user_login', 1),
('2024-08-02 00:00:00', 'user_login', 1),
('2024-08-05 00:00:00', 'user_login', 1),
('2024-08-01 00:00:00', 'user_create', 2),
('2024-08-10 00:00:00', 'user_login', 2),
('2024-08-02 00:00:00', 'user_create', 3),
('2024-08-03 00:00:00', 'user_login', 3),
('2024-08-10 00:00:00', 'user_login', 3),
('2024-08-01 00:00:00', 'user_create', 4);


--- 查询语句
SELECT init_date, arrayReduce('sumMap', arrayMap(x -> map(x, 1), groupArray(loss_date_count))) AS loss_data
  FROM (
    SELECT t1.init_date AS init_date, t1.user_id AS user_id, return_dates, coalesce(date_diff('day', init_date, arrayFirstOrNull(x -> x > init_date, return_dates)), 999999) - 1 AS loss_date_count
      FROM (
      SELECT DISTINCT ON (time, user_id) time AS init_date, user_id
        FROM retention_test
       WHERE event = 'user_create'
       ORDER BY time
    ) AS t1 GLOBAL LEFT JOIN (
      SELECT user_id, arraySort(groupUniqArray(time)) AS return_dates
        FROM retention_test
       WHERE event = 'user_login'
       GROUP BY 1
    ) AS t2 ON t1.user_id = t2.user_id
  )
  GROUP BY 1
  ORDER BY 1;


-- 查询结果
┌───────────init_date─┬─loss_data──────────┐
│ 2024-08-01 00:00:00 │ {0:1,8:1,999998:1} │
│ 2024-08-02 00:00:00 │ {0:1}              │
└─────────────────────┴────────────────────┘
```