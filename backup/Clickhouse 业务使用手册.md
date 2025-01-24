### 变更语句是否执行完
```sql
select hostname(),* from clusterAllReplicas('default','system.mutations') where is_done = 0;
```
---

### 清理数据库表
```sql
truncate table demo.events_local;
truncate table demo_global.users_local;
truncate table demo_global.devices_local;
```
---

### 根据 query 查询 clickhouse 日志
```sql
select * from cluster('all-sharded', 'system', 'query_log') where query_id  = '4efbbb9b-b853-44ca-84b2-95ff066895af';
```
---

### 数据导出成CSV
```sql
SELECT * FROM events
SETTINGS join_use_nulls=1, allow_experimental_analyzer=0
INTO OUTFILE '新玩家首日首场战场数据.csv.gz'
FORMAT CSVWithNames;
```
---

### 数据去重验证
```sql
select `_part`, `#dt`, `#event`, `#log_id`, count(1), groupArray(hostname()) from events where `#dt` = '2024-07-28' group by 1, 2, 3, 4 order by 5 desc limit 100;
```
