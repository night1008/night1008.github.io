之前的业务代码中需要生成 gin swagger 用于和前端联调，且和主流程代码放在一起，
导致打包镜像的时候也需要先生成 swagger，目前看这个只是用于本地和前端联调，
因此独立 gin swagger 启动命令。

代码目录结构

```
/myproject
  /swagger
    docs.go
    swagger.json  # 生成的文档
    swagger.yaml
  /internal
    /server
      server.go   # 主应用代码
    ...
  main.go
```

示例代码如下，

```go
package main

import (
	"flag"
	"fmt"
	"log"
	"os/exec"

	"github.com/gin-gonic/gin"
	swaggerFiles "github.com/swaggo/files"
	ginSwagger "github.com/swaggo/gin-swagger"
)

var port string

func init() {
	flag.StringVar(&port, "port", "8080", "gin swagger http port")
}

func main() {
	flag.Parse()

	startSwaggerServer(fmt.Sprintf(":%s", port))
}

func startSwaggerServer(addr string) {
	cmd := exec.Command("swag", "init", "--parseDependency", "--parseInternal", "-o", "./swagger")
	err := cmd.Run()
	if err != nil {
		log.Fatalf("make swagger failed with %s\n", err)
	}

	r := gin.Default()

	r.Static("/docs", "./swagger")
	r.GET("/swagger/*any", ginSwagger.WrapHandler(
		swaggerFiles.Handler,
		ginSwagger.URL("/docs/swagger.json"),
		ginSwagger.DefaultModelsExpandDepth(-1),
	))

	r.Run(addr)
}
```