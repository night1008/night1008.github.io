使用 [clickhouse-go](https://github.com/ClickHouse/clickhouse-go) 作为客户端进行 clickhouse 查询时，发现某些查询不会返回查询错误，
1. 查询时间超过最大查询时间参数 (max_execution_time)
2. 非法查询，比如 sleep(300)

下面给示例，

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/ClickHouse/clickhouse-go/v2"
)

func main() {
	db := clickhouse.OpenDB(&clickhouse.Options{
		Addr: []string{"127.0.0.1:9000"},
		Auth: clickhouse.Auth{
			Database: "default",
			Username: "default",
			Password: "",
		},
		Settings: clickhouse.Settings{
			"join_use_nulls": 1,
		},
	})

	ctx := context.Background()
	conn, err := db.Conn(ctx)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	sql := "select sleep(300)"
	rows, err := conn.QueryContext(ctx, sql)
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()

	fmt.Println(rows.Columns())
}
```

期望结果是，
```
code: 160, message: The maximum sleep time is 3000000 microseconds. Requested: 300: while executing 'FUNCTION sleep(300 :: 0) -> sleep(300) UInt8 : 1'
```

执行结果是，
```
[sleep(300)] <nil>
```

---

问了官方，说是，
Since the error is not thrown in ClickHouse at the time of query retrieval but later with the data packet, the client has to process data received from the server explicitly using rows.Next().

See code snippet:
```go
package issues

import (
	"context"
	"testing"

	clickhouse_tests "github.com/ClickHouse/clickhouse-go/v2/tests"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

func Test1268(t *testing.T) {
	conn, err := clickhouse_tests.GetDatabaseSQLConnection("issues", nil, nil, nil)
	require.NoError(t, err)

	rows, err := conn.QueryContext(context.Background(), "select sleep(300)")
	require.NoError(t, err)
	defer rows.Close()

	for rows.Next() {
		if rows.Err() != nil {
			break
		}
	}

	assert.ErrorContains(t, rows.Err(), "code: 160, message: The maximum sleep time is 3000000 microseconds.")
}
```