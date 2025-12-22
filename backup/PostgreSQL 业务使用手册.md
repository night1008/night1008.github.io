### 只读账号授权
```sql
GRANT CONNECT ON DATABASE demo_web TO demo_web_ro;

GRANT USAGE ON SCHEMA public TO demo_web_ro;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO demo_web_ro;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO demo_web_ro;
```

---

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

-- 根据表已有的数据进行重置
SELECT setval('users_id_seq', (SELECT coalesce(max(id), 0) FROM users));
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
  pg_size_pretty(pg_relation_size(quote_ident(table_name)))
from information_schema.tables
where table_schema = 'public'
order by pg_total_relation_size(quote_ident(table_name)) desc;

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
SELECT datname as db_name, pg_size_pretty(pg_database_size(datname)) as db_usage FROM pg_database order by pg_database_size(datname) desc;
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

---

### 复制表
```sql
create table users_old as (select * from users);
```

---

### 计算列（Generated Columns）
PostgreSQL 12+ 支持生成列（Generated Columns），这些列的值会自动计算并存储：

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    price DECIMAL(10,2),
    quantity INTEGER,
    total_price DECIMAL(10,2) GENERATED ALWAYS AS (price * quantity) STORED
);
```
STORED 表示值会被实际存储（物化），而不是每次查询时计算。

---

### 查询含有某个字段的表
```sql
SELECT table_schema, table_name
FROM information_schema.columns
WHERE column_name = 'xxx';
```

---

### values 伪造表
```sql
SELECT * FROM (VALUES (1, 'one'), (2, 'two'), (3, 'three')) AS t (num,letter);
```

---

### 联表更新
```sql
UPDATE users
SET sso_user_id = sso_users.id
FROM sso_users
WHERE users.email = sso_users.login_name and sso_users.id is not null;
```

---

### 查看当前所有查询进程
```sql
SELECT
    pid,
    usename,
    datname,
    client_addr,
    state,
    query_start,
    now() - query_start AS runtime,
    query
FROM
    pg_stat_activity
WHERE
    state <> 'idle'
ORDER BY
    query_start;
```

---

### 仅取消查询（连接还在）
```sql
SELECT pg_cancel_backend(pid);
```
