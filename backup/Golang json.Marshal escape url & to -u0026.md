Golang 对包含链接地址的数据进行序列化后发现地址不能直接使用了，测试代码如下。

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	str := "https://p6-mapi-sign.creativityeco.com/web.business.image/202503055d0df5342ee12cfe444cbc29~tplv-iq460dd072-origin.image?key=%23cqMGg8EHUcPdvl7lyx75cSmxgtrV7Zt1JYWCH2NEcMpx9LgfapWMYa%2B4m9Ide3O4mMSJ%2F1pPxQEkH3MqXIHxEyQKAfhBdr70klV0Is7iidjK45XGe6cXRr6VkYIt7PImdC%2B2vVGfTiLWKzPhuryx01mGcybPwVFc1AgOnrpSDBVlIhQTFXrGXzsNyxg%3D&lk3s=62ea907e&sign_for=m_api&x-expires=1742499103&x-signature=AcvUBb0BTT3hFAIyueNdTd%2BbHoE%3D"
	strBytes, _ := json.Marshal(str)
	str2 := string(strBytes)
	fmt.Println(str2)
	// 输出结果: "https://p6-mapi-sign.creativityeco.com/web.business.image/202503055d0df5342ee12cfe444cbc29~tplv-iq460dd072-origin.image?key=%23cqMGg8EHUcPdvl7lyx75cSmxgtrV7Zt1JYWCH2NEcMpx9LgfapWMYa%2B4m9Ide3O4mMSJ%2F1pPxQEkH3MqXIHxEyQKAfhBdr70klV0Is7iidjK45XGe6cXRr6VkYIt7PImdC%2B2vVGfTiLWKzPhuryx01mGcybPwVFc1AgOnrpSDBVlIhQTFXrGXzsNyxg%3D\u0026lk3s=62ea907e\u0026sign_for=m_api\u0026x-expires=1742499103\u0026x-signature=AcvUBb0BTT3hFAIyueNdTd%2BbHoE%3D"

	// 解决方法，可以反序列化回去

	var str3 string
	if err := json.Unmarshal([]byte(str2), &str3); err != nil {
		panic(err)
	}
	fmt.Println(str3)
	// 输出结果: https://p6-mapi-sign.creativityeco.com/web.business.image/202503055d0df5342ee12cfe444cbc29~tplv-iq460dd072-origin.image?key=%23cqMGg8EHUcPdvl7lyx75cSmxgtrV7Zt1JYWCH2NEcMpx9LgfapWMYa%2B4m9Ide3O4mMSJ%2F1pPxQEkH3MqXIHxEyQKAfhBdr70klV0Is7iidjK45XGe6cXRr6VkYIt7PImdC%2B2vVGfTiLWKzPhuryx01mGcybPwVFc1AgOnrpSDBVlIhQTFXrGXzsNyxg%3D&lk3s=62ea907e&sign_for=m_api&x-expires=1742499103&x-signature=AcvUBb0BTT3hFAIyueNdTd%2BbHoE%3D
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


