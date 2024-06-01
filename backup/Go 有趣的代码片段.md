### 可以对 error 进行 switch 判断

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