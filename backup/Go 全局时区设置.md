正常本地开发环境的时区是东八区，线上服务器的时区是零时区，  
调试一些线上服务器的时区问题的时候，如何通过设置全局时区在本地进行调试呢。

示例代码如下，

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	loc, err := time.LoadLocation("UTC")
	if err != nil {
		panic(err)
	}
	time.Local = loc // -> this is setting the global timezone

	now := time.Now()
	fmt.Println(now)
	// 2025-01-06 01:59:10.767064 +0000 UTC m=+0.000079404
}
```