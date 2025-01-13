### 手动修改联合主键表索引

原本 `ad_advertisers` 表以 `app_id`, `channel_name`, `account_id` 三个字段为主键，现在如何往中间再加一个主键字段

```sql
CREATE UNIQUE INDEX ad_advertisers_pkey ON public.ad_advertisers USING btree (app_id, channel_name, account_id);

=>

ALTER TABLE public.ad_advertisers DROP CONSTRAINT ad_advertisers_pkey;
UPDATE ad_advertisers SET account_type = '' WHERE account_type IS NULL;
ALTER TABLE public.ad_advertisers ADD CONSTRAINT ad_advertisers_pkey PRIMARY KEY (app_id, channel_name, account_type, account_id);
```

---

### 重置数据库表 ID

```sql
ALTER SEQUENCE users_id_seq RESTART WITH 7;
```

---

### 查看表结构

```sql
pg_dump -h localhost -p 5432 -U postgres -d db_name -t table_name --schema-only
```

---

### 表大小情况查询

```sql
-- 查看每个表的占用空间大小
select
  table_name, 
  pg_size_pretty(pg_total_relation_size(quote_ident(table_name))),
  pg_relation_size(quote_ident(table_name))
from information_schema.tables
where table_schema = 'public'
order by 3 desc;

-- 查看每个表的行数
SELECT 
    schemaname AS schema,
    relname AS table,
    n_live_tup AS estimated_row_count
FROM 
    pg_stat_user_tables
ORDER BY 3 desc;

-- 查看某个数据库磁盘占用情况
select pg_size_pretty(pg_database_size('funnydb_web'));

-- 查看全部数据库磁盘占用情况
SELECT datname as db_name, pg_size_pretty(pg_database_size(datname)) as db_usage FROM pg_database order by db_usage desc;
```

---

### 查询列的字段类型
```sql
select pg_typeof("column1"), pg_typeof("column2") from table1;
```

---

### 文本类型字段的数字匹配
```sql
SELECT value, value::numeric FROM property_values where value ~ '^[-0-9.]+$' limit 1000;

select min(id) from (select '10005' as id union all select '995' as id);  -- 错误写法
```
