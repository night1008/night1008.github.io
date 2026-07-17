### 可以对 error 进行 switch 判断

来源：https://github.com/prashanthpai/sqlcache/blob/4bf943bfd00f02394a480c5437e86af4b5be074c/cache_redis.go#L24

```go
func (r *Redis) Get(ctx context.Context, key string) (*cache.Item, bool, error) {
	b, err := r.c.Get(ctx, r.keyPrefix+key).Bytes()
	switch err {
	case nil:
		var item cache.Item
		if err := msgpack.Unmarshal(b, &item); err != nil {
			return nil, true, err
		}
		return &item, true, nil
	case redis.Nil:
		return nil, false, nil
	default:
		return nil, false, err
	}
}
```

---

### 匿名结构体

来源：https://quii.gitbook.io/learn-go-with-tests/go-fundamentals/concurrency#channels

```go
type result struct {
	string
	bool
}

var r result
fmt.Println(r.string, r.bool)
```

---

### build 时忽略某个目录

比如 storage，这这个目录下加一个 go.mod，声明一个独立的模块名（如 module storage），Go 就会把它当作独立模块，主模块的 ./... 不会再包含它。