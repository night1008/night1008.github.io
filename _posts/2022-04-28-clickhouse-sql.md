---
layout: post
keywords: clickhouse sql tips
description: clickhouse sql tips
title: clickhouse sql tips
comments: true
---

```sql
SELECT groupArray(arrayJoin([1, NULL, 2]));
---
┌─groupArray(arrayJoin([1, NULL, 2]))─┐
│ [1,2]                               │
└─────────────────────────────────────┘

SELECT groupArray(arrayJoin([1, inf, 2]));
---
┌─groupArray(arrayJoin([1, inf, 2]))─┐
│ [1,inf,2]                          │
└────────────────────────────────────┘

SELECT arrayMap(x -> x = inf ? null : x, groupArray(arrayJoin([1, inf, 2])));
---
┌─arrayMap(lambda(tuple(x), if(equals(x, inf), NULL, x)), groupArray(arrayJoin([1, inf, 2])))─┐
│ [1,NULL,2]                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```


```sql
SELECT group_0,
  CAST((groupArray(dt), groupArray(amount_0)) as Map(String, Nullable(UInt64))) AS data_map_0,
  CAST((groupArray(dt), groupArray(amount_1)) as Map(String, Nullable(UInt64))) AS data_map_1
  from (
  select 'IOS' AS group_0, arrayJoin(['2022-04-10', '2022-04-11', '2022-04-12']) as dt, 10 as amount_0, 0 as amount_1
  union all
  select 'Andriod' AS group_0, arrayJoin(['2022-04-10', '2022-04-11']) as dt, 0 as amount_0, 20 as amount_1
) group by group_0;
---
┌─group_0─┬─data_map_0────────────────────────────────────────┬─data_map_1─────────────────────────────────────┐
│ IOS     │ {'2022-04-10':10,'2022-04-11':10,'2022-04-12':10} │ {'2022-04-10':0,'2022-04-11':0,'2022-04-12':0} │
│ Andriod │ {'2022-04-10':0,'2022-04-11':0}                   │ {'2022-04-10':20,'2022-04-11':20}              │
└─────────┴───────────────────────────────────────────────────┴────────────────────────────────────────────────┘
```

```sql
SELECT group_0, groupArray((dt, amount_0)), groupArray((dt, amount_1)) from (
  select 'IOS' AS group_0, arrayJoin(['2022-04-10', '2022-04-11', '2022-04-12']) as dt, 10 as amount_0, null as amount_1
  union all
  select 'Andriod' AS group_0, arrayJoin(['2022-04-10', '2022-04-11']) as dt, null as amount_0, 20 as amount_1
) group by group_0;
---
┌─group_0─┬─groupArray(tuple(dt, amount_0))─────────────────────────┬─groupArray(tuple(dt, amount_1))───────────────────────────────┐
│ IOS     │ [('2022-04-10',10),('2022-04-11',10),('2022-04-12',10)] │ [('2022-04-10',NULL),('2022-04-11',NULL),('2022-04-12',NULL)] │
│ Andriod │ [('2022-04-10',NULL),('2022-04-11',NULL)]               │ [('2022-04-10',20),('2022-04-11',20)]                         │
└─────────┴─────────────────────────────────────────────────────────┴───────────────────────────────────────────────────────────────┘
```

```sql
SELECT group_0, arrayFilter(x -> x.2 is not null, groupArray((dt, amount_0))) as amount_0, arrayFilter(x -> x.2 is not null, groupArray((dt, amount_1)))  as amount_1 from (
  select 'IOS' AS group_0, arrayJoin(['2022-04-10', '2022-04-11', '2022-04-12']) as dt, 10 as amount_0, null as amount_1
  union all
  select 'Andriod' AS group_0, arrayJoin(['2022-04-10', '2022-04-11']) as dt, null as amount_0, 20 as amount_1
) group by group_0;
---
┌─group_0─┬─amount_0────────────────────────────────────────────────┬─amount_1──────────────────────────────┐
│ IOS     │ [('2022-04-10',10),('2022-04-11',10),('2022-04-12',10)] │ []                                    │
│ Andriod │ []                                                      │ [('2022-04-10',20),('2022-04-11',20)] │
└─────────┴─────────────────────────────────────────────────────────┴───────────────────────────────────────┘
```