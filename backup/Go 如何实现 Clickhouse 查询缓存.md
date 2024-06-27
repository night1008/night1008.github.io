**整体思路：先判断是否命中缓存，没有的话再进入查询队列**

依赖的包
```
https://github.com/ClickHouse/ClickHouse

https://github.com/prashanthpai/sqlcache => https://github.com/night1008/sqlcache

https://github.com/DATA-DOG/go-sqlmock

https://github.com/blockloop/scan
```

现在的问题是 clickhouse 的查询结果可能返回复杂类型，比如 Map(String, UInt8)，
如果命中缓存，缓存的结果为 `driver.Rows`，需要转换成 `*sql.Rows` 方便后续使用，
因此想到通过 `sqlmock` 的方式，但是默认的 `value converter` 是 `driver.DefaultParameterConverter`，只能转换基础类型，
复杂类型也没有定义专门用于解析的结构体，会报诸如以下的错误，

> panic: row #1, column #2 ("mapper") type map[string]uint8: unsupported type map[string]uint8, a map

好在 `go-sqlmock` 可以指定 `ValueConverterOption`，这样就可以把 `driver.Rows` 转换成 `*sql.Rows` 了。

以下为测试代码

```go
package main

import (
	"context"
	"database/sql"
	"database/sql/driver"
	"fmt"
	"io"
	"log"
        "regexp"

	"github.com/DATA-DOG/go-sqlmock"
	"github.com/prashanthpai/sqlcache"
	"github.com/prashanthpai/sqlcache/cache"

	"github.com/dgraph-io/ristretto"
	_ "github.com/ClickHouse/clickhouse-go/v2"
)

const (
	defaultMaxRowsToCache = 100
)

func newRistrettoCache(maxRowsToCache int64) (cache.Cacher, error) {
	c, err := ristretto.NewCache(&ristretto.Config{
		NumCounters: 10 * maxRowsToCache,
		MaxCost:     maxRowsToCache,
		BufferItems: 64,
	})
	if err != nil {
		return nil, err
	}

	return sqlcache.NewRistretto(c), nil
}

var querySQL string = `
		-- @cache-ttl 60
		-- @cache-max-rows 10
		SELECT 1 AS id,
		'aaa' AS name,
		CAST((['Ready', 'Steady', 'Go'], [1, 2, 3]), 'Map(String, UInt8)') AS mapper;`

type customConverter struct{}

func (customConverter) ConvertValue(v any) (driver.Value, error) {
	return v, nil
}

func main() {
	cache, err := newRistrettoCache(defaultMaxRowsToCache)
	if err != nil {
		log.Fatalf("newRistrettoCache() failed: %v", err)
	}

	interceptor, err := sqlcache.NewInterceptor(&sqlcache.Config{
		Cache: cache, // pick a Cacher interface implementation of your choice (redis or ristretto)
	})
	if err != nil {
		log.Fatalf("sqlcache.NewInterceptor() failed: %v", err)
	}

	defer func() {
		fmt.Printf("\nInterceptor metrics: %+v\n", interceptor.Stats())
	}()

	dsn := "clickhouse://localhost:9000"

	conn, err := sql.Open("clickhouse", dsn)
	if err != nil {
		log.Fatal(err)
	}
	// install the wrapper which wraps pgx driver
	sql.Register("clickhouse-sqlcache", interceptor.Driver(conn.Driver()))

	db, err := sql.Open("clickhouse-sqlcache", dsn)
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	if err = db.PingContext(context.TODO()); err != nil {
		log.Fatal(fmt.Errorf("db.PingContext() failed: %w", err))
	}

	rows, err := db.QueryContext(context.TODO(), querySQL)
	if err != nil {
		log.Fatal(fmt.Errorf("db.QueryContext() failed: %w", err))
	}
	defer rows.Close()

	for rows.Next() {
		var id int
		var name string
		var mapper interface{} //map[string]uint64
		if err := rows.Scan(&id, &name, &mapper); err != nil {
			log.Fatal(fmt.Errorf("rows.Scan() failed: %w", err))
		}
		fmt.Println("===> ", id, name, mapper)
	}

	db, mock, _ := sqlmock.New(sqlmock.ValueConverterOption(customConverter{}))
	cacheRows := interceptor.CheckCache(context.Background(), querySQL, nil)
	if cacheRows != nil {
		var dests [][]driver.Value
		for {
			dest := make([]driver.Value, len(cacheRows.Columns()))
			if err := cacheRows.Next(dest); err == nil {
				dests = append(dests, dest)
			} else if err == io.EOF {
				break
			} else {
				log.Fatal(err)
			}
		}
		mockRows := mock.NewRows(cacheRows.Columns()).AddRows(dests...)
		mock.ExpectQuery(regexp.QuoteMeta(querySQL)).WillReturnRows(mockRows)
		rows, _ := db.Query(querySQL)
		fmt.Println(rows)
  
                type Person struct {
			ID   int    `db:"id"`
			Name string `db:"name"`
			// Mapper map[string]uint64 `db:"mapper"`
		}
		var persons []Person
		err := scan.Rows(&persons, rows)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Printf("%#v\n", persons)
	}
}
```