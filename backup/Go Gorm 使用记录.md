### 先 union 再聚合

```go
var results []struct {
    Category string
    SumTotal int
}

err := db.Table("(?) as t", 
    db.Model(&Table1{}).Select("category, COUNT(*) as total").Group("category").
    Union(db.Model(&Table2{}).Select("category, COUNT(*) as total").Group("category")),
).
Select("category, SUM(total) as sum_total").
Group("category").
Find(&results).Error

if err != nil {
    // 处理错误
}
```