通过 [gorm clickhouse](https://github.com/go-gorm/clickhouse) 进行批量数据插入时，如果超过批大小，就会报以下错误，

```
[clickhouse][conn=1][127.0.0.1:9000][exception] code: 101, message: Unexpected packet Query received from client
[clickhouse-std][conn=0][127.0.0.1:9000] PrepareContext error: code: 101, message: Unexpected packet Query received from client
```

测试代码如下，

```go
package main

import (
	"crypto/md5"
	"fmt"
	"io"
	"time"

	std_ck "github.com/ClickHouse/clickhouse-go/v2"
	"gorm.io/datatypes"
	"gorm.io/driver/clickhouse"
	"gorm.io/gorm"
)

type AdTencentReportCK struct {
	AppID      int64  `json:"app_id"`                            // funnydb 应用ID
	ReportID   string `json:"report_id"`                         // 报表ID
	ReportType string `json:"report_type"`                       // 报表类型
	Level      string `json:"level"`                             // 报表类型级别
	TimeLine   string `json:"time_line"`                         // 时间口径
	AccountId  int64  `json:"account_id"`                        // 账号ID
	Date       string `json:"date"`                              // 日期
	Hour       int64  `json:"hour"`                              // 小时(0-23)
	IngestTime int64  `json:"ingest_time" gorm:"autoCreateTime"` // 采集时间，毫秒级
	Data       datatypes.JSON
}

func (AdTencentReportCK) TableName() string {
	return "ad_tencent_reports_test"
}

func GenerateReportID(keys ...string) string {
	w := md5.New()
	for _, key := range keys {
		_, _ = io.WriteString(w, key)
	}
	return fmt.Sprintf("%x", w.Sum(nil))
}

func main() {
	ckSTDDB := std_ck.OpenDB(&std_ck.Options{
		Addr: []string{"127.0.0.1:9000"},
		Auth: std_ck.Auth{
			Database: "ad_reports",
			Username: "admin",
			Password: "secret",
		},
		Settings: std_ck.Settings{
			"max_execution_time": 3600,
			"final":              1,
		},
		DialTimeout: 5 * time.Second,
		Compression: &std_ck.Compression{
			Method: std_ck.CompressionLZ4,
		},
		Debug: true,
	})

	ckDB, err := gorm.Open(clickhouse.New(clickhouse.Config{
		Conn: ckSTDDB,
	}))
	if err != nil {
		panic(err)
	}

	var reports []*AdTencentReportCK
	for i := 0; i < 110; i++ {
		report := AdTencentReportCK{
			AppID:      1,
			ReportID:   GenerateReportID("1", fmt.Sprintf("%d", i+1)),
			ReportType: "daily_reports",
			Level:      "REPORT_LEVEL_ADVERTISER",
			TimeLine:   "ACTIVE_TIME",
			AccountId:  int64(i + 1),
			Date:       "2025-01-01",
			IngestTime: 1763308800,
			Data:       datatypes.JSON(`{"cost": 1000}`),
		}
		reports = append(reports, &report)
	}

	if err := ckDB.CreateInBatches(reports, 100).Error; err != nil {
		panic(err)
	}
}
```

解决方式，直接使用，
```go
if err := ckDB.Create(reports).Error; err != nil {
	panic(err)
}
```

### 注意事项
在同一个事务中多次提交也会触发这个错误，比如
```go
if err := ckDB.Transaction(func(tx *gorm.DB) error {
	for _, _reports := range reports {
		if err := tx.Create(_reports).Error; err != nil {
			return err
		}
	}
	return nil
}); err != nil {
	panic(err)
}
```