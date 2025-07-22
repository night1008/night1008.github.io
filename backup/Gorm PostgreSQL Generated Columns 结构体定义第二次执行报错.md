业务上想通过 PostgreSQL Generated Columns 自动把 JSON 字段生成为新的列，如下所示，

```go
type AdPlatformEventTrackReport struct {    
    // Your regular fields
    Data datatypes.JSON `gorm:"type:jsonb;column:data"`  // Explicit column name
    
    // Generated column with explicit column name
    TrackPayAmount float64 `gorm:"<-:false;column:track_pay_amount;type:float GENERATED ALWAYS AS ((data->>'track_pay_amount')::float) STORED"`
}
```

第一次执行迁移是可以正常创建列的，但是第二次执行就会报如下错误，

```sh
ERROR: syntax error at or near "GENERATED" (SQLSTATE 42601)
[0.821ms] [rows:0] ALTER TABLE "ad_platform_event_track_reports" ALTER COLUMN "track_pay_amount" TYPE float GENERATED ALWAYS AS ((data->>'track_pay_amount')::float) STORED USING "track_pay_amount"::float GENERATED ALWAYS AS ((data->>'track_pay_amount')::float) STORED
```

尝试了几次仍然无法解决，最后只能先用以下方式解决。

```go
type AdPlatformEventTrackReport struct {    
    // Your regular fields
    Data datatypes.JSON `gorm:"type:jsonb;column:data"`  // Explicit column name
    
    // Generated column with explicit column name
   TrackPayAmount    *float64 `json:"track_pay_amount" gorm:"->;-:migration;column:track_pay_amount"`
}

if err := db.Exec(
  `ALTER TABLE "ad_platform_event_track_reports"
     ADD COLUMN IF NOT EXISTS "track_pay_amount" float GENERATED ALWAYS AS ((data->>'track_pay_amount')::float) STORED;`,
).Error; err != nil {
	return err
}
```