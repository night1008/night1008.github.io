发现 Golang MySQL 下，MySQL 驱动默认把 interface{} 类型解析为 []byte，导致字符串和数字以 Base64 编码形式返回。
测试代码如下，

```go
package main

import (
	"context"
	"database/sql"
	"fmt"
	"time"

	_ "github.com/go-sql-driver/mysql"
)

func main() {
	dsn := "root:secret@tcp(localhost:3306)/?charset=utf8mb4&parseTime=true"
	db, err := sql.Open("mysql", dsn)
	if err != nil {
		panic(err)
	}

	ctx := context.Background()
	rows1, err := db.QueryContext(ctx, "select 100 as a, 'aaa' as b, now() as t")
	if err != nil {
		panic(err)
	}
	defer rows1.Close()

	var a1 int64
	var b1 string
	var t1 time.Time
	for rows1.Next() {
		rows1.Scan(&a1, &b1, &t1)
	}
	fmt.Println(a1, b1, t1) // 100 aaa 2025-03-10 08:48:39 +0000 UTC

	rows2, err := db.QueryContext(ctx, "select 100 as a, 'aaa' as b, now() as t")
	if err != nil {
		panic(err)
	}
	defer rows2.Close()

	var a2 interface{}
	var b2 interface{}
	var t2 interface{}
	for rows2.Next() {
		rows2.Scan(&a2, &b2, &t2)
	}
	fmt.Println(a2, b2, t2) // [49 48 48] [97 97 97] 2025-03-10 08:50:25 +0000 UTC
	if aBytes, ok := a2.([]byte); ok {
		fmt.Println("===> string(aBytes) ", string(aBytes)) // ===> string(aBytes)  100
	}
	if bBytes, ok := b2.([]byte); ok {
		fmt.Println("===> string(bBytes) ", string(bBytes)) // ===> string(bBytes)  aaa
	}
}
```