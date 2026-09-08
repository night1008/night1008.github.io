## 现象

某 Go 服务经 `database/sql` + [clickhouse-go/v2](https://github.com/ClickHouse/clickhouse-go) 查询 ClickHouse,偶发:

```
internal query panic: reflect: call of reflect.Value.Interface on zero Value
```

日志附带 `{"panic":{"Method":"reflect.Value.Interface","Kind":0}}`。`Kind:0` = `reflect.Invalid`,
即对一个**未初始化的零 reflect.Value** 调了 `.Interface()`——问题不在业务逻辑,而在某处反射代码。

## 关键排查点:别被表象栈骗了

panic 被上层 `recover` 后作为普通 error 上报,栈顶停在上报调用点,像业务代码的锅。要看的是 **recover 打出的原始 goroutine 栈**,真凶在:

```
lib/column.(*Array).Row(...)  array.go:114   ← 崩溃点
(*stdRows).Next(...)         clickhouse_std.go:428
database/sql.(*Rows).Next(...)
```

> 当 panic 发生在第三方库里,找"谁 recover 了它",看那个 recover 打出的完整 goroutine 栈。

## 根因:驱动吞掉解码错误,又对零值调 `.Interface()`

`lib/column/array.go` 的 `Array.Row`:

```go
func (col *Array) Row(i int, ptr bool) any {
    value, err := col.scan(col.ScanType(), i) // 反射解码某一行
    if err != nil {
        fmt.Println(err)          // ← 错误只打印、被丢弃
    }
    return value.Interface()      // ← value 是零 Value → panic
}
```

`col.scan` 一旦因元素类型无法解码而返回 `(零Value, err)`,这里只把 err 打到 stdout,仍把零值交给
`.Interface()`。**这是所有同类崩溃的最后一步。**

让它返回零值 + err 的主要情形是"**嵌套的无名 tuple**":ClickHouse 无名 tuple 默认扫成 `[]any`,
驱动禁止把无名 tuple 解码成单个 `interface{}`(`Tuple.scan` 对 `!isNamed` 的 interface 目标直接拒绝,
报 `cannot use interface for unnamed tuples, use slice`)。当结果列是 `Array(Tuple(col, Tuple(...)))`
——外层无名 tuple 里再嵌一个无名 tuple——内层的目标类型变成 `interface{}`,触发拒绝,再被 `Array.Row` 吞掉。

次要变体:无名 tuple 内含**非 Nullable 的 Int128/Int256/UInt128/UInt256** 也会解码失败
(`BigInt` 走 `ptr=false` 返回值 `big.Int` 塞不进 `*big.Int` 目标;套 `Nullable` 后走 `ptr=true` 则正常)。

## 最容易踩的 SQL 形态:argMin 打包再被外层聚合

用 `argMin/argMax` 取"一行多字段",再被外层 `groupArray(tuple(...))` 二次包一层,就构成嵌套无名 tuple:

```sql
SELECT user_id, groupArray(tuple(attr_key, row_data)) AS agg
FROM (
  SELECT user_id, attr_key,
         argMin(tuple(field1, field2, field3, field4, field5, field6,
                      field7, field8, field9, field10, field11, field12,
                      field13, field14), tuple(field1, id)) AS row_data
  FROM user_events GROUP BY user_id, attr_key
)
GROUP BY user_id ORDER BY user_id
```

内层 `argMin(...) AS row_data` 本是单层无名 tuple,单独返回没问题;外层 `groupArray(tuple(attr_key, row_data))`
把它包成 `Array(Tuple(UInt64, Tuple(...)))`,命中拒绝逻辑。报错对象正是内层整条 tuple:
`converting Tuple(UInt64, ... Int64) to interface {} is unsupported`.这种写法在"AI 生成 SQL / 复杂聚合"里很常见。

## 实测对照:什么炸 / 什么不炸

| 结果列形态 | 结果 |
| --- | --- |
| `Array(Tuple(简单字段))`(单层无名 tuple) | ✅ 正常 |
| `Array(Tuple(col, Tuple(...)))`(无名 tuple 套无名 tuple) | 🔥 panic |
| `Array(Map(...))` / 命名外层(走 map 目标,不碰 interface 分支) | ✅ 正常 |
| tuple 内非 Nullable `Int128/Int256/UInt128/UInt256` | 🔥 panic(变体) |
| 同类型但被 `Nullable(...)` 包裹 | ✅ 正常 |
| 纯 `Array(Array(...))`、顶层普通简单类型 | ✅ 正常 |

## 最小复现

无需业务表,直连 ClickHouse,用 `database/sql` 把结果逐列 Scan 进 `interface{}`:

```sql
SELECT groupArray(tuple(tuple(toInt64(1)))) AS arr
```

```go
rows, _ := db.QueryContext(ctx, "SELECT groupArray(tuple(tuple(toInt64(1)))) AS arr")
for rows.Next() {
    var v interface{}
    rows.Scan(&v) // ← 在这里 panic
}
```

stdout 先出现驱动打印、随后崩溃:

```
clickhouse [ScanRow]: converting Tuple(Int64) to interface {} is unsupported. cannot use interface for unnamed tuples, use slice
reflect: call of reflect.Value.Interface on zero Value
```

## 影响面与版本

实测 **v2.25.0 与 v2.41.0** 中 `Array.Row` 丢 err 的写法**一字未改**,该问题在很长版本区间内未修复、升级驱动无效。
触发入口不限于某类接口——凡是"把含嵌套无名 tuple 的 Array 列 Scan 进 `interface{}`"的路径都可能踩中,
其中 **用户/AI 可控 SQL** 的查询最容易暴露。

## 小结与经验

1. `reflect.Value.Interface on zero Value` 几乎总是"某库对零反射值调 `.Interface()`";配合 `"Kind": 0` 可快速确认方向。
2. panic 上报栈会被 recover 改写,**看 recover 处的完整 goroutine 栈**才能找到真凶。
3. 本质是第三方驱动**错误处理不严谨**(吞 error、把零值继续往下传),排查时不妨怀疑"库把错误吞了、只留一个零值"。
4. 版本升级不是万能药,这类缺陷可能长期存在。

---

*排查环境:clickhouse-go/v2 v2.25.0(对照至 v2.41.0)· ClickHouse 26.3*
