通过间隔分析查看转化时长的分布情况。

假设用户在过去某个时间段内行为序列是：`A1 -> A2 -> A3 -> B1 -> B2 -> B3 -> A4 -> B4`，
初始行为 A 事件，后续行为 B 事件，
以距离初始行为事件最近的规则得到，`(A1 -> B1), (A4 -> B4)`
以距离后续行为事件最近的规则得到，`(A3 -> B1), (A4 -> B4)`

如何用 Clickhouse 语言实现间隔分析？

```sql
-- 建表语句
CREATE TABLE IF NOT EXISTS interval_test (
  time DateTime,
  event String,
  user_id UInt64,
  amount Int64
) ENGINE = Memory;


--- 插入数据
INSERT INTO interval_test VALUES
('2024-08-01 00:00:00', 'A', 1, 0),
('2024-08-01 01:00:00', 'A', 1, 0),
('2024-08-01 02:00:00', 'B', 1, 1),
('2024-08-02 03:00:00', 'B', 1, 2),
('2024-08-05 01:00:00', 'A', 1, 3),
('2024-08-05 02:00:00', 'A', 1, 4),
('2024-08-05 03:00:00', 'A', 1, 5),
('2024-08-05 04:00:00', 'B', 1, 5),
('2024-08-05 05:00:00', 'B', 1, 5),
('2024-08-05 06:00:00', 'B', 1, 5),
('2024-08-01 01:00:00', 'A', 2, 0),
('2024-08-02 02:00:00', 'A', 2, 1),
('2024-08-02 03:00:00', 'B', 2, 1),
('2024-08-10 04:00:00', 'B', 2, 2),
('2024-08-02 00:00:00', 'A', 3, 0),
('2024-08-03 01:00:00', 'B', 3, 1),
('2024-08-06 02:00:00', 'B', 3, 2),
('2024-08-10 03:00:00', 'B', 3, 3),
('2024-08-01 04:00:00', 'A', 4, 0);

--- 以距离初始行为事件最近的规则
select distinct on (t2.date, t2.user_id) * from (
  select date_trunc('day', time) as date, 0 as event_type, * from interval_test where event = 'A'
) as t1 global asof join (
  select date_trunc('day', time) as date, 1 as event_type, * from interval_test where event = 'B'
) as t2 on t1.date = t2.date and t1.user_id = t2.user_id and t1.time < t2.time
order by time

--- 以距离后续行为事件最近的规则
select distinct on (t2.date, t2.user_id) * from (
  select date_trunc('day', time) as date, 0 as event_type, * from interval_test where event = 'A'
) as t1 global asof join (
  select date_trunc('day', time) as date, 1 as event_type, * from interval_test where event = 'B'
) as t2 on t1.date = t2.date and t1.user_id = t2.user_id and t1.time < t2.time
order by time desc
```