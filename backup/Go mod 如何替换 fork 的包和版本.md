以  `github.com/prashanthpai/sqlcache` 为例，想替换成 fork 的包 `github.com/night1008/sqlcache`，
假设当前该包的版本为 `v0.0.0`

#### 第一步
```
go mod edit -replace github.com/prashanthpai/sqlcache@v0.0.0=github.com/night1008/sqlcache@master
```

---

#### 第二步
```
go get -u github.com/night1008/sqlcache
```

会有以下提示
```
go: downloading github.com/night1008/sqlcache v0.0.0-20240623031410-4d47c940a2d7
go: github.com/night1008/sqlcache@v0.0.0-20240623031410-4d47c940a2d7: parsing go.mod:
	module declares its path as: github.com/prashanthpai/sqlcache
	        but was required as: github.com/night1008/sqlcache
```

---

#### 第三步

手动把 v0.0.0-20240623031410-4d47c940a2d7 替换到 go.mod `replace github.com/prashanthpai/sqlcache v0.0.0 => github.com/night1008/sqlcache master` 这一行的 master

---

#### 第四步
```
go mod tidy
```