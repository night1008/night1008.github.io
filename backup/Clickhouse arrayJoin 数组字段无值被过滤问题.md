Clickhouse 使用 arrayJoin 展开数组字段时，有些数组字段可能是没有值的，这时候这些记录会被过滤掉，如下所示，

```sql
select *, arrayJoin(a) as b_item from (
  select array(1, 2, 3) as a, 1 as b
  union all
  select array(1) as a, 2 as b
  union all
  select array() as a, 3 as b
)

   ┌─a───────┬─b─┬─b_item─┐
1. │ [1,2,3] │ 1 │      1 │
2. │ [1,2,3] │ 1 │      2 │
3. │ [1,2,3] │ 1 │      3 │
   └─────────┴───┴────────┘
   ┌─a───┬─b─┬─b_item─┐
4. │ [1] │ 2 │      1 │
   └─────┴───┴────────┘
```

---

解决方法如下

```sql
select *, arrayJoin(if(empty(a), [null], a)) as b_item from (
  select array(1, 2, 3) as a, 1 as b
  union all
  select array(1) as a, 2 as b
  union all
  select array() as a, 3 as b
)

   ┌─a───┬─b─┬─b_item─┐
1. │ [1] │ 2 │      1 │
   └─────┴───┴────────┘
   ┌─a───────┬─b─┬─b_item─┐
2. │ [1,2,3] │ 1 │      1 │
3. │ [1,2,3] │ 1 │      2 │
4. │ [1,2,3] │ 1 │      3 │
   └─────────┴───┴────────┘
   ┌─a──┬─b─┬─b_item─┐
5. │ [] │ 3 │   ᴺᵁᴸᴸ │
   └────┴───┴────────┘
```

```sql
select *, arrayJoin(range(1, length(if(empty(a), [null], a)) + 1)) AS b_item_index from (
  select array(1, 2, 3) as a, 1 as b
  union all
  select array(1) as a, 2 as b
  union all
  select array() as a, 3 as b
)

   ┌─a───┬─b─┬─b_item_index─┐
1. │ [1] │ 2 │            1 │
   └─────┴───┴──────────────┘
   ┌─a──┬─b─┬─b_item_index─┐
2. │ [] │ 3 │            1 │
   └────┴───┴──────────────┘
   ┌─a───────┬─b─┬─b_item_index─┐
3. │ [1,2,3] │ 1 │            1 │
4. │ [1,2,3] │ 1 │            2 │
5. │ [1,2,3] │ 1 │            3 │
   └─────────┴───┴──────────────┘
```