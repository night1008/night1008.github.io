写了一个去除 `map` 中的零数值的方法，如下所示，  
发现当有些值为 `int64` 类型时，即使为零数值也不会去除，

```go
func RemoveMapZeroNumValue(m map[string]interface{}, excludeKeysMap map[string]struct{}) map[string]interface{} {
	for k, v := range m {
		if _, ok := excludeKeysMap[k]; ok {
			continue
		}
		switch val := v.(type) {
		case int, int8, int16, int32, int64,
			uint, uint8, uint16, uint32, uint64,
			float32, float64:
			if val == 0 || val == 0.0 {
				delete(m, k)
			}
		}
	}
	return m
}
```

原来Go 的多类型 case 不会把 val 的类型统一，它只保证你进入了这个 case，
Go 的 type switch 规则：
- val 的静态类型为所有 case 的「公共类型」
- 多个类型的公共类型就是 interface{}
- 所以 val 的静态类型是 interface{}

写法 | val 的实际类型 | 比较结果
-- | -- | --
case int: | int | val == 0 正常
case int, float64: | interface{} | val == 0 永远 false

解决方法：
1. 使用 `reflect.ValueOf(val).IsZero()` 判断
2. 拆分所有 case 