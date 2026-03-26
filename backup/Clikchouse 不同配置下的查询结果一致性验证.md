随着升级 Clikchouse 版本，需要检查业务代码中的查询语句的结果是否一致

比如发现下面的语句在不同设置下结果不一样
```sql
SELECT toDateTime(date_trunc('day', `vpc@timezone_#time`)) AS date_time, 'event_0' AS event_index, 'compare_0' AS compare_index, coalesce(formatDateTimeInJodaSyntax(toDateTime(date_trunc('day', `vpc@timezone_#time`)), 'yyyy-MM-dd'), 'null') AS group_0, round(toFloat64(coalesce(COUNT(1), 0)), 2) AS `$amount` FROM (SELECT * FROM (SELECT `#data_lifecycle`, `#dt`, `#event`, `#time`, `vpc@timezone_#time`, `#zone_offset` FROM (SELECT `#data_lifecycle`, `#dt`, `#event`, `#time`, timestamp_add(fromUnixTimestamp64Milli(`#time`), interval 8 * 60 minute) AS `vpc@timezone_#time`, `#zone_offset` FROM events WHERE (`#event` IN ('#user_login')) AND ((`#dt` BETWEEN '2026-03-18' AND '2026-03-20'))) AS event) WHERE (`#data_lifecycle` = '0') AND (`#dt` BETWEEN '2026-03-18' AND '2026-03-20' AND `vpc@timezone_#time` BETWEEN '2026-03-19 00:00:00.000' AND '2026-03-19 23:59:59.999' ) AND (`#event` IN ('#user_login'))) GROUP BY GROUPING SETS((date_time, group_0),(group_0));
```

`set allow_experimental_analyzer=0;`
<img width="851" height="184" alt="Image" src="https://github.com/user-attachments/assets/afd33513-7308-4c8f-899b-ec263e3c4ab8" />

`set allow_experimental_analyzer=1;`
<img width="856" height="192" alt="Image" src="https://github.com/user-attachments/assets/0a133a69-175f-4355-89e5-de894fe2e0c1" />


原因是
<img width="864" height="705" alt="Image" src="https://github.com/user-attachments/assets/a6af1766-f18c-4f2e-93db-b9250c5870cb" />


所以需要编写脚本对比不同配置下的查询结果，
可以从用户的查询日志作为检查对象，
需要注意以下问题，
1. 浮点数 => 需要固定精度，比如使用 round 函数
2. map 类型 => 可以使用 mapSort 函数固定顺序
3. 时间范围包含当前时间 => 可以使用正则替换大于等于当前时间为过去时间