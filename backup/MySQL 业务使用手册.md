### 查询数据库大小
```sql
SELECT
  table_schema,
  ROUND(SUM(data_length) / 1024 / 1024 / 1024, 2) AS data_size,
  ROUND(SUM(index_length) / 1024 / 1024 / 1024, 2) AS index_size
FROM
  information_schema.TABLES
GROUP BY
  table_schema
ORDER BY
  2 DESC;
```

---

### 导出导入数据库

```sh
mysqldump --no-tablespaces -h 127.0.0.1 -P 3306 -u root -p operations_system > operations_system.sql

mysql -h 127.0.0.1 -P 3306 -u root -p operations_system < operations_system.sql
```