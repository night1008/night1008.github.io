### 可以对 error 进行 switch 判断

来源 https://github.com/prashanthpai/sqlcache/blob/4bf943bfd00f02394a480c5437e86af4b5be074c/cache_redis.go#L24

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