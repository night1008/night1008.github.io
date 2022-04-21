---
layout: post
keywords: go json unmarshal int to float64
description: go json unmarshal 整数类型解析问题
title: go json unmarshal 整数类型解析问题
comments: true
---

业务代码中，发现 json 字符串转换成 go 结构体时，整数数值不能被正常解析成 `int` 类型.

测试代码如下，可以看到整数 `1` 的类型被解析成 `float64`了。
这是因为标准的 json 中没有 整数和浮点数之分，都被统一解析成浮点数。

```go
package main

import (
	"encoding/json"
	"fmt"
	"reflect"
)

func main() {
	var jsonContent = []byte(`[1, 1.2, "a"]`)

	var values []interface{}
	if err := json.Unmarshal(jsonContent, &values); err != nil {
		panic(err)
	}

	for _, value := range values {
		fmt.Println(fmt.Sprintf("%v type is %s", value, reflect.TypeOf(value).Name()))
	}
	// 1 type is float64
	// 1.2 type is float64
	// a type is string
}
```