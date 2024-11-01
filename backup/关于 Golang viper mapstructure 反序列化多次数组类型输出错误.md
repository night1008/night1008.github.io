业务程序使用了 Golang 中的 [viper](github.com/spf13/viper) 进行加载配置，
但在实际使用过程中因为读取了两次配置，同时反序列化了两次，
导致一些数组类型的字段出现了不符合预期的结果，如下测试程序所示。

经过排查才知道是因为 [mapstructure decodeSlice](https://github.com/mitchellh/mapstructure/blob/8508981c8b6c964e6986dd8aa85490e70ce3c2e2/mapstructure.go#L1136)
使用反射保留了第一次的数组大小，如果第一次的数组长度大于第二次的数组长度，多余的数组元素仍然会保留。

---

```go
package main

import (
	"bytes"
	"fmt"

	"github.com/spf13/viper"
)

var defaultConfigContent []byte = []byte(`
values:
  - item1
  - item2
  - item3
  - item4
server:
  endpoint: http://example11.com
  values:
    - item1
    - item2
    - item3
    - item4
`)

var overrideConfigContent []byte = []byte(`
values:
  - item11
  - item22
server:
  endpoint: http://example22.com
  values:
    - item11
    - item22
`)

type Config struct {
	raw    *viper.Viper
	Values []string `mapstructure:"values"`
	Server struct {
		Endpoint string   `mapstructure:"endpoint"`
		Values   []string `mapstructure:"values"`
	} `mapstructure:"server"`
}

func main() {
	raw := viper.NewWithOptions()
	raw.SetConfigType("yaml")

	config := &Config{
		raw: raw,
	}

	if err := config.raw.MergeConfig(bytes.NewBuffer(defaultConfigContent)); err != nil {
		panic(err)
	}

	// 两次序列化需要开启该注释代码
	// if err := config.raw.Unmarshal(&config); err != nil {
	// 	panic(err)
	// }

	if err := config.raw.MergeConfig(bytes.NewBuffer(overrideConfigContent)); err != nil {
		panic(err)
	}

	if err := config.raw.Unmarshal(&config); err != nil {
		panic(err)
	}

	fmt.Println("===> config ", config)

	// 日志打印
	// 序列化一次 (正确)
	// ===> config  &{0xc00014c380 [item11 item22] {http://example22.com [item11 item22]}}

	// 序列化两次 (错误)
	// ===> config  &{0xc0001f2380 [item11 item22 item3 item4] {http://example22.com [item11 item22 item3 item4]}}
}
```