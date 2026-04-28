最近尝试使用阿里云函数计算，发现使用`定时器触发器`的方式传入的参数内容无法正确反序列化，如下所示，

```go
func HandleRequest(ctx context.Context, event SomeStruct) error {
	log.Println("===> param: ", event)
}
```

后面把 event 为指定为 `map[string]interface{}`，打印后发现，不同调用方式的传入参数内容不同，如下所示，

```go
func HandleRequest(ctx context.Context, event map[string]interface{}) error {
	log.Println("===> param: ", event)
}
```

### 定时器触发器
```
map[payload:{"debug":true,"app_id":74,"account_ids":[1823998827268096,1823998807388364,1823998819034986,1823998841668699,1823998805359619,1823998842198409]} triggerName:funnyads_ad_material_monitor triggerTime:2025-04-24T09:46:00Z]
```

### 网页测试接口
```
map[account_ids:[1823998827268096,1823998807388364,1823998819034986,1823998841668699,1823998805359619,1823998842198409] app_id:74 debug:true]
```

可以用以下方式做兼容处理

```go
func HandleRequest(ctx context.Context, event map[string]interface{}) error {
  var param SomeStruct
	// 先简单判断是否来自定时触发器
	if _, ok := event["payload"]; ok {
		if err := json.Unmarshal([]byte(event["payload"].(string)), &param); err != nil {
			return err
		}
	} else {
		eventData, err := json.Marshal(event)
		if err != nil {
			return err
		}
		if err := json.Unmarshal(eventData, &param); err != nil {
			return err
		}
	}
  ...
}
```
