### 背景

我们使用了 `PostgreSQL` 表的记录作为全局任务的全局锁，
并且使用的是 `FOR UPDATE` 的行级别锁，
但在锁已被表持有的情况下，如果要对表结构进行变更字段的话，会执行失败，
场景为旧部署服务已持有了行级别锁，新部署服务就变更不了表结构，
因为 DDL 操作（如添加列）会对整个表加锁，这两者发生了冲突。


### 解决方法：
1. 尽量少变更作为锁机制的表结构
2. 先停止旧部署服务，再启动新部署服务，不要让新旧服务同时存在

> 可以使用如下语句查询锁占用情况
```sql
-- 查看锁占用情况
SELECT * FROM pg_locks WHERE relation = 'service_deployments'::regclass;
```

### 示例代码


`Go` 语言下使用 `Gorm` 库获取 `PostgreSQL` 表的记录作为全局锁的示例代码

```go
const (
	ServiceDeploymentNameGlobalJobLock = "global-job-lock" // 全局任务锁
)

type ServiceDeployment struct {
	Name string `gorm:"primarykey"` // 服务名称

	Config string // 后续添加的字段

	CreatedAt int64 `gorm:"autoCreateTime:milli"` // 记录创建时间，单位 milli
	UpdatedAt int64 `gorm:"autoUpdateTime:milli"` // 记录更新时间，单位 milli
}

func GetServiceDeploymentGlobalJobLock(db *gorm.DB) (bool, error) {
	deployment := ServiceDeployment{
		Name: ServiceDeploymentNameGlobalJobLock,
	}
	if err := db.FirstOrCreate(&deployment, ServiceDeployment{Name: ServiceDeploymentNameGlobalJobLock}).Error; err != nil {
		return false, err
	}

	tx := db.Session(&gorm.Session{Logger: logger.Discard}).Begin() // 不打印错误日志，拿到锁会占用一个数据库连接
	exist, err := gormx.FindOne(tx.Clauses(clause.Locking{Strength: "UPDATE", Options: "NOWAIT"}).Where("name = ?", ServiceDeploymentNameGlobalJobLock), &deployment)
	if err != nil {
		if pgErr, ok := err.(*pgconn.PgError); ok && pgErr.Code == "55P03" {
			err = nil
		}
		_ = tx.Rollback()
	}
	return exist, err
}
```