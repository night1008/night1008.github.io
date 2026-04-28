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

---

### 复制 db  查询
```go
db, _ := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
    
// Original query
original := db.Where("name = ?", "Alice")
    
// Create independent copy
copy := original.Session(&gorm.Session{})
    
// Modify the copy without affecting original
copy = copy.Where("age > ?", 20)
    
// Original still only has the name condition
// Copy has both name and age conditions
```