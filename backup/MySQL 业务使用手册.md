### 查询数据库大小
```sql
SELECT table_schema AS "Database", SUM(data_length + index_length) / 1024 / 1024 / 1024 AS "Size (GB)" 
FROM information_schema.TABLES GROUP BY table_schema;
```