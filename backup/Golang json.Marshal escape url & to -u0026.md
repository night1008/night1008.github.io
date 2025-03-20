Golang 对包含链接地址的数据进行序列化后发现地址不能直接使用了，测试代码如下。

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	str := "https://example.com?a=1&b=2"
	strBytes, _ := json.Marshal(str)
	fmt.Println(string(strBytes))
	// 输出结果: https://example.com?a=1\u0026b=2

	// 解决方法，可以反序列化回去

	var str2 string
	if err := json.Unmarshal(strBytes, &str2); err != nil {
		panic(err)
	}
	fmt.Println(str2)
	// 输出结果: https://example.com?a=1&b=2
}
```

原来在 Go 语言中，当你使用 json.Marshal 将数据编码为 JSON 字符串时，某些特殊字符（如 &）会被转义为 Unicode 编码形式（例如 \u0026）。这是为了确保生成的 JSON 字符串是有效的、可移植的，并且符合 JSON 规范。

---

### 会被转义的特殊字符
| 字符              | 转义后的形式 | 说明 |
| ---------------- | ------ | ---- |
| `"`        |   `\" `  | 双引号 |
| `\`         |   `\\`   | 反斜杠 |
| `/`    |  `\/`   | 斜杠（可选转义） |
| `\b` |  `\\b`   | 退格符 |
| `\f` |  `\\f`   | 换页符 |
| `\n` |  `\\n`   | 换行符 |
| `\r` |  `\\r`   | 回车符 |
| `\t` |  `\\t`   | 制表符 |
| `<` |  `\u003c`  | 小于号 |
| `>` |  `\u003e`  | 大于号 |
| `&` |  `\u0026`  | 与符号 |
| 控制字符（如 `\x00` 到 `\x1F`）|  `\uXXXX`  | 控制字符会被转义为 Unicode 编码 |


