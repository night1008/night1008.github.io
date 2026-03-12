旧查询
```sql
WITH range(-1, 239 + 1) AS date_interval_nums,
arrayEnumerate(date_interval_nums) AS date_interval_nums_enumerate
SELECT
  init_date,
  toString(init_date) AS init_date_str,
  loss_data,
  array(
    retention_data_all [1],
    retention_data_all [3],
    retention_data_all [4],
    retention_data_all [8],
    retention_data_all [15],
    retention_data_all [31],
    retention_data_all [61],
    retention_data_all [121],
    retention_data_all [181],
    retention_data_all [241]
  ) AS retention_data,
  array(
    retention_amount_all [1],
    retention_amount_all [3],
    retention_amount_all [4],
    retention_amount_all [8],
    retention_amount_all [15],
    retention_amount_all [31],
    retention_amount_all [61],
    retention_amount_all [121],
    retention_amount_all [181],
    retention_amount_all [241]
  ) AS retention_amount,
  array(
    total_retention_amount_all [1],
    total_retention_amount_all [3],
    total_retention_amount_all [4],
    total_retention_amount_all [8],
    total_retention_amount_all [15],
    total_retention_amount_all [31],
    total_retention_amount_all [61],
    total_retention_amount_all [121],
    total_retention_amount_all [181],
    total_retention_amount_all [241]
  ) AS total_retention_amount,
  toString(retention_data) AS retention_data_str,
  toJSONString(loss_data) AS loss_data_str,
  toJSONString(retention_amount) AS retention_amount_str,
  toJSONString(total_retention_amount) AS total_retention_amount_str
FROM
  (
    SELECT
      init_date,
      retention_data_all,
      loss_data,
      arrayMap(
        x -> round(
          x / if(
            retention_data_all [1] = 0,
            1,
            retention_data_all [1]
          ),
          2
        ),
        arrayCumSum(base_retention_amount_all)
      ) AS retention_amount_all,
      arrayCumSum(base_retention_amount_all) AS total_retention_amount_all
    FROM
      (
        SELECT
          init_date,
          groupArray(r) AS r_list,
          groupArray(a) AS a_list,
          arrayMap(
            i -> arraySum(arrayMap(x -> x [i], r_list)),
            range(1, length(r_list [1]) + 1)
          ) AS retention_data_all,
          arrayMap(
            i -> round(arraySum(arrayMap(x -> x [i], a_list)), 2),
            range(1, length(a_list [1]) + 1)
          ) AS base_retention_amount_all,
          arrayReduce(
            'sumMap',
            arrayMap(x -> map(x, 1), groupArray(loss_date_count))
          ) AS loss_data
        FROM
          (
            SELECT
              t1.date_time AS init_date,
              t1.virtual_user_id AS virtual_user_id,
              mapFromArrays(
                return_dates,
                arrayWithConstant(length(return_dates), 1)
              ) AS return_dates_map,
              arrayMap(
                x -> if(
                  x == -1,
                  1,
                  mapContains(return_dates_map, date_add(DAY, x, init_date))
                ),
                date_interval_nums
              ) AS r,
              coalesce(
                date_diff(
                  'day',
                  init_date,
                  arrayFirstOrNull(x -> x > init_date, return_dates)
                ),
                999999
              ) - 1 AS loss_date_count,
              date_amounts_map,
              arrayMap(
                x -> multiIf(
                  x == 1,
                  0,
                  r [x],
                  date_amounts_map [date_add(DAY, date_interval_nums[x],init_date ) ],
                  0
                ),
                date_interval_nums_enumerate
              ) AS a
        FROM
          (
            SELECT
              DISTINCT ON (date_time, virtual_user_id) date_time,
              virtual_user_id
            FROM
              (
                SELECT
                  toDateTime(date_trunc('day', `vpc@timezone_#time`)) AS date_time,
                  toString(`#user_id`) AS virtual_user_id,
                  `#time`
                FROM
                  (
                    SELECT
                      *
                    FROM
                      (
                        SELECT
                          `#data_lifecycle`,
                          `#dt`,
                          `#event`,
                          `#time`,
                          `vpc@timezone_#time`,
                          `#user_id`,
                          `#zone_offset`
                        FROM
                          (
                            SELECT
                              `#data_lifecycle`,
                              `#dt`,
                              `#event`,
                              `#time`,
                              timestamp_add(
                                fromUnixTimestamp64Milli(`#time`),
                                interval 8 * 60 minute
                              ) AS `vpc@timezone_#time`,
                              `#user_id`,
                              `#zone_offset`
                            FROM
                              events
                            WHERE
                              (`#event` IN ('#user_create'))
                              AND (
                                (
                                  `#dt` BETWEEN '2025-11-07'
                                  AND '2025-12-08'
                                )
                              )
                          ) AS event
                      )
                    WHERE
                      (
                        (
                          `#dt` BETWEEN '2025-11-07'
                          AND '2025-12-08'
                          AND `vpc@timezone_#time` BETWEEN '2025-11-08 00:00:00.000'
                          AND '2025-12-07 23:59:59.999'
                        )
                      )
                      AND (`#data_lifecycle` = '0')
                      AND (`#event` IN ('#user_create'))
                  )
                WHERE
                  virtual_user_id != ''
                  AND virtual_user_id IS NOT NULL
              )
            ORDER BY
              `#time`
          ) AS t1 GLOBAL
          LEFT JOIN (
            SELECT
              toString(`#user_id`) AS virtual_user_id,
              arraySort(
                groupUniqArray(
                  toDateTime(date_trunc('day', `vpc@timezone_#time`))
                )
              ) AS return_dates
            FROM
              (
                SELECT
                  *
                FROM
                  (
                    SELECT
                      `#data_lifecycle`,
                      `#dt`,
                      `#event`,
                      `#time`,
                      `vpc@timezone_#time`,
                      `#user_id`,
                      `#zone_offset`
                    FROM
                      (
                        SELECT
                          `#data_lifecycle`,
                          `#dt`,
                          `#event`,
                          `#time`,
                          timestamp_add(
                            fromUnixTimestamp64Milli(`#time`),
                            interval 8 * 60 minute
                          ) AS `vpc@timezone_#time`,
                          `#user_id`,
                          `#zone_offset`
                        FROM
                          events
                        WHERE
                          (`#event` IN ('#user_login'))
                          AND (
                            (
                              `#dt` BETWEEN '2025-11-07'
                              AND '2026-08-05'
                            )
                          )
                      ) AS event
                  )
                WHERE
                  (
                    (
                      `#dt` BETWEEN '2025-11-07'
                      AND '2026-08-05'
                      AND `vpc@timezone_#time` BETWEEN '2025-11-08 00:00:00.000'
                      AND '2026-08-04 23:59:59.999'
                    )
                  )
                  AND (`#data_lifecycle` = '0')
                  AND (`#event` IN ('#user_login'))
              )
            WHERE
              virtual_user_id != ''
              AND virtual_user_id IS NOT NULL
            GROUP BY
              virtual_user_id
          ) AS t2 ON t1.virtual_user_id = t2.virtual_user_id GLOBAL
          LEFT JOIN (
            SELECT
              virtual_user_id,
              cast(
                (groupArray(date_time), groupArray(amount)) AS Map(Datetime, Float64)
              ) AS date_amounts_map
            FROM
              (
                SELECT
                  toDateTime(date_trunc('day', `vpc@timezone_#time`)) AS date_time,
                  toString(`#user_id`) AS virtual_user_id,
                  toFloat64(coalesce(SUM(`#vp@amount_yuan`), 0)) AS amount
                FROM
                  (
                    SELECT
                      *,
                      CAST((`#amount` / 100) AS Nullable(Float64)) AS `#vp@amount_yuan`
                    FROM
                      (
                        SELECT
                          `#amount`,
                          `#data_lifecycle`,
                          `#dt`,
                          `#event`,
                          `#time`,
                          `vpc@timezone_#time`,
                          `#user_id`,
                          `#zone_offset`
                        FROM
                          (
                            SELECT
                              `#amount`,
                              `#data_lifecycle`,
                              `#dt`,
                              `#event`,
                              `#time`,
                              timestamp_add(
                                fromUnixTimestamp64Milli(`#time`),
                                interval 8 * 60 minute
                              ) AS `vpc@timezone_#time`,
                              `#user_id`,
                              `#zone_offset`
                            FROM
                              events
                            WHERE
                              (`#event` IN ('#charge'))
                              AND (
                                (
                                  `#dt` BETWEEN '2025-11-07'
                                  AND '2026-08-05'
                                )
                              )
                          ) AS event
                      )
                    WHERE
                      (
                        (
                          `#dt` BETWEEN '2025-11-07'
                          AND '2026-08-05'
                          AND `vpc@timezone_#time` BETWEEN '2025-11-08 00:00:00.000'
                          AND '2026-08-04 23:59:59.999'
                        )
                      )
                      AND (`#data_lifecycle` = '0')
                      AND (`#event` IN ('#charge'))
                  )
                WHERE
                  virtual_user_id != ''
                  AND virtual_user_id IS NOT NULL
                GROUP BY
                  date_time,
                  virtual_user_id
              )
            GROUP BY
              virtual_user_id
          ) AS t3 ON t1.virtual_user_id = t3.virtual_user_id
      )
    GROUP BY
      init_date
  )
)
ORDER BY
  init_date,
  retention_data [1] DESC
LIMIT
  100000
settings use_query_cache = 0, enable_writes_to_query_cache = 0;
```

<img width="1280" height="661" alt="Image" src="https://github.com/user-attachments/assets/832ea9c2-35f6-422a-b564-f764c5df9bb5" />

---

### 慢的原因
聚合阶段使用了太多中间数组，导致处理很慢。
在最小改动的情况下，去掉中间层数组聚合处理，查询可以从 600 秒提升到 6 秒内。

<img width="806" height="386" alt="Image" src="https://github.com/user-attachments/assets/3c16df86-6fcf-43ce-a6fe-0eb379db88cf" />

---

新查询
```sql
WITH range(-1, 239 + 1) AS date_interval_nums,
arrayEnumerate(date_interval_nums) AS date_interval_nums_enumerate
SELECT
  init_date,
  toString(init_date) AS init_date_str,
  loss_data,
  retention_data_all AS retention_data,
  retention_amount_all AS retention_amount,
  total_retention_amount_all AS total_retention_amount,
  toString(retention_data) AS retention_data_str,
  toJSONString(loss_data) AS loss_data_str,
  toJSONString(retention_amount) AS retention_amount_str,
  toJSONString(total_retention_amount) AS total_retention_amount_str
FROM
  (
    SELECT
      init_date,
      retention_data_all,
      loss_data,
      arrayMap(
        x -> round(
          x / if(
            retention_data_all [1] = 0,
            1,
            retention_data_all [1]
          ),
          2
        ),
        base_retention_amount_all
      ) AS retention_amount_all,
      base_retention_amount_all AS total_retention_amount_all
    FROM
      (
        SELECT
          init_date,
          [
                count(),
                countIf(mapContains(return_dates_map, date_add(DAY,   1, init_date))),
                countIf(mapContains(return_dates_map, date_add(DAY,   2, init_date))),
                countIf(mapContains(return_dates_map, date_add(DAY,   6, init_date))),
                countIf(mapContains(return_dates_map, date_add(DAY,  13, init_date))),
                countIf(mapContains(return_dates_map, date_add(DAY,  29, init_date))),
                countIf(mapContains(return_dates_map, date_add(DAY,  59, init_date))),
                countIf(mapContains(return_dates_map, date_add(DAY, 119, init_date))),
                countIf(mapContains(return_dates_map, date_add(DAY, 179, init_date))),
                countIf(mapContains(return_dates_map, date_add(DAY, 239, init_date)))
          ] AS retention_data_all,
          [
            0,
            sum(arraySum(mapValues(mapFilter((k, v) -> k >= init_date AND k <= date_add(DAY, 1, init_date), date_amounts_map)))),
            sum(arraySum(mapValues(mapFilter((k, v) -> k >= init_date AND k <= date_add(DAY, 2, init_date), date_amounts_map)))),
            sum(arraySum(mapValues(mapFilter((k, v) -> k >= init_date AND k <= date_add(DAY, 6, init_date), date_amounts_map)))),
            sum(arraySum(mapValues(mapFilter((k, v) -> k >= init_date AND k <= date_add(DAY, 13, init_date), date_amounts_map)))),
            sum(arraySum(mapValues(mapFilter((k, v) -> k >= init_date AND k <= date_add(DAY, 29, init_date), date_amounts_map)))),
            sum(arraySum(mapValues(mapFilter((k, v) -> k >= init_date AND k <= date_add(DAY, 59, init_date), date_amounts_map)))),
            sum(arraySum(mapValues(mapFilter((k, v) -> k >= init_date AND k <= date_add(DAY, 119, init_date), date_amounts_map)))),
            sum(arraySum(mapValues(mapFilter((k, v) -> k >= init_date AND k <= date_add(DAY, 179, init_date), date_amounts_map)))),
            sum(arraySum(mapValues(mapFilter((k, v) -> k >= init_date AND k <= date_add(DAY, 239, init_date), date_amounts_map))))
          ] AS base_retention_amount_all,
          arrayReduce(
            'sumMap',
            arrayMap(x -> map(x, 1), groupArray(loss_date_count))
          ) AS loss_data
        FROM
          (
            SELECT
              t1.date_time AS init_date,
              t1.virtual_user_id AS virtual_user_id,
              return_dates,
              mapFromArrays(return_dates, arrayWithConstant(length(return_dates), 1)) AS return_dates_map,
              coalesce(
                date_diff(
                  'day',
                  init_date,
                  arrayFirstOrNull(x -> x > init_date, return_dates)
                ),
                999999
              ) - 1 AS loss_date_count,
              date_amounts_map
        FROM
          (
            SELECT
              DISTINCT ON (date_time, virtual_user_id) date_time,
              virtual_user_id
            FROM
              (
                SELECT
                  toDateTime(date_trunc('day', `vpc@timezone_#time`)) AS date_time,
                  toString(`#user_id`) AS virtual_user_id,
                  `#time`
                FROM
                  (
                    SELECT
                      *
                    FROM
                      (
                        SELECT
                          `#data_lifecycle`,
                          `#dt`,
                          `#event`,
                          `#time`,
                          `vpc@timezone_#time`,
                          `#user_id`,
                          `#zone_offset`
                        FROM
                          (
                            SELECT
                              `#data_lifecycle`,
                              `#dt`,
                              `#event`,
                              `#time`,
                              timestamp_add(
                                fromUnixTimestamp64Milli(`#time`),
                                interval 8 * 60 minute
                              ) AS `vpc@timezone_#time`,
                              `#user_id`,
                              `#zone_offset`
                            FROM
                              events
                            WHERE
                              (`#event` IN ('#user_create'))
                              AND (
                                (
                                  `#dt` BETWEEN '2025-11-07'
                                  AND '2025-12-08'
                                )
                              )
                          ) AS event
                      )
                    WHERE
                      (
                        (
                          `#dt` BETWEEN '2025-11-07'
                          AND '2025-12-08'
                          AND `vpc@timezone_#time` BETWEEN '2025-11-08 00:00:00.000'
                          AND '2025-12-07 23:59:59.999'
                        )
                      )
                      AND (`#data_lifecycle` = '0')
                      AND (`#event` IN ('#user_create'))
                  )
                WHERE
                  virtual_user_id != ''
                  AND virtual_user_id IS NOT NULL
              )
            ORDER BY
              `#time`
          ) AS t1 GLOBAL
          LEFT JOIN (
            SELECT
              toString(`#user_id`) AS virtual_user_id,
              arraySort(
                groupUniqArray(
                  toDateTime(date_trunc('day', `vpc@timezone_#time`))
                )
              ) AS return_dates
            FROM
              (
                SELECT
                  *
                FROM
                  (
                    SELECT
                      `#data_lifecycle`,
                      `#dt`,
                      `#event`,
                      `#time`,
                      `vpc@timezone_#time`,
                      `#user_id`,
                      `#zone_offset`
                    FROM
                      (
                        SELECT
                          `#data_lifecycle`,
                          `#dt`,
                          `#event`,
                          `#time`,
                          timestamp_add(
                            fromUnixTimestamp64Milli(`#time`),
                            interval 8 * 60 minute
                          ) AS `vpc@timezone_#time`,
                          `#user_id`,
                          `#zone_offset`
                        FROM
                          events
                        WHERE
                          (`#event` IN ('#user_login'))
                          AND (
                            (
                              `#dt` BETWEEN '2025-11-07'
                              AND '2026-08-05'
                            )
                          )
                      ) AS event
                  )
                WHERE
                  (
                    (
                      `#dt` BETWEEN '2025-11-07'
                      AND '2026-08-05'
                      AND `vpc@timezone_#time` BETWEEN '2025-11-08 00:00:00.000'
                      AND '2026-08-04 23:59:59.999'
                    )
                  )
                  AND (`#data_lifecycle` = '0')
                  AND (`#event` IN ('#user_login'))
              )
            WHERE
              virtual_user_id != ''
              AND virtual_user_id IS NOT NULL
            GROUP BY
              virtual_user_id
          ) AS t2 ON t1.virtual_user_id = t2.virtual_user_id GLOBAL
          LEFT JOIN (
            SELECT
              virtual_user_id,
              cast(
                (groupArray(date_time), groupArray(amount)) AS Map(Datetime, Float64)
              ) AS date_amounts_map
            FROM
              (
                SELECT
                  toDateTime(date_trunc('day', `vpc@timezone_#time`)) AS date_time,
                  toString(`#user_id`) AS virtual_user_id,
                  toFloat64(coalesce(SUM(`#vp@amount_yuan`), 0)) AS amount
                FROM
                  (
                    SELECT
                      *,
                      CAST((`#amount` / 100) AS Nullable(Float64)) AS `#vp@amount_yuan`
                    FROM
                      (
                        SELECT
                          `#amount`,
                          `#data_lifecycle`,
                          `#dt`,
                          `#event`,
                          `#time`,
                          `vpc@timezone_#time`,
                          `#user_id`,
                          `#zone_offset`
                        FROM
                          (
                            SELECT
                              `#amount`,
                              `#data_lifecycle`,
                              `#dt`,
                              `#event`,
                              `#time`,
                              timestamp_add(
                                fromUnixTimestamp64Milli(`#time`),
                                interval 8 * 60 minute
                              ) AS `vpc@timezone_#time`,
                              `#user_id`,
                              `#zone_offset`
                            FROM
                              events
                            WHERE
                              (`#event` IN ('#charge'))
                              AND (
                                (
                                  `#dt` BETWEEN '2025-11-07'
                                  AND '2026-08-05'
                                )
                              )
                          ) AS event
                      )
                    WHERE
                      (
                        (
                          `#dt` BETWEEN '2025-11-07'
                          AND '2026-08-05'
                          AND `vpc@timezone_#time` BETWEEN '2025-11-08 00:00:00.000'
                          AND '2026-08-04 23:59:59.999'
                        )
                      )
                      AND (`#data_lifecycle` = '0')
                      AND (`#event` IN ('#charge'))
                  )
                WHERE
                  virtual_user_id != ''
                  AND virtual_user_id IS NOT NULL
                GROUP BY
                  date_time,
                  virtual_user_id
              )
            GROUP BY
              virtual_user_id
          ) AS t3 ON t1.virtual_user_id = t3.virtual_user_id
      )
    GROUP BY
      init_date
  )
)
ORDER BY
  init_date,
  retention_data [1] DESC
LIMIT
  100000
  format Vertical
settings use_query_cache = 0, enable_writes_to_query_cache = 0;
```

<img width="1280" height="693" alt="Image" src="https://github.com/user-attachments/assets/67455c87-17d2-4e9e-ae61-22235e1c1149" />