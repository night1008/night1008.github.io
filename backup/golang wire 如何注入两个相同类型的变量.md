如果使用 wire 初始化两个相同类型的变量，会报以下错误，

> BaseProviderSet has multiple bindings for *gorm.io/gorm.DB

---

解决方式，先对变量定义新类型，最后创建的时候再转换成原来的类型。

伪代码如下，

```go
type DB1 *gorm.DB

func NewDB1() DB1{
	var db *gorm.DB
	return db
}

type DB2 *gorm.DB

func NewDB2() DB2{
	var db *gorm.DB
	return db
}
```

```go
type Injector struct {
	DB1  DB1
	DB2  DB2
}

var BaseProviderSet = wire.NewSet(
	NewDB1,
	NewDB2,
)
```

```go
var AppSvcProvider = wire.NewSet(NewAppSvc)

type AppService struct {
	DB1            *gorm.DB
	DB2            *gorm.DB
}

func NewAppSvc(db1 DB1, db2 DB2) (*AppService, error) {
	cronParser := cron.NewParser(cron.Minute | cron.Hour | cron.Dom | cron.Month | cron.Dow | cron.Descriptor)

	svc := &AppService{
		DB1:          db1,
		DB2:          db2,
	}
	return svc, nil
}
```
