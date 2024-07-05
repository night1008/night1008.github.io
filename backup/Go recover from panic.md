```go
package main

import (
	"fmt"
	"runtime/debug"
	"time"

	"github.com/rs/zerolog/log"
)

func Recover() {
	if r := recover(); r != nil {
		fmt.Println(string(debug.Stack()))
		log.Error().Err(fmt.Errorf("%s", string(debug.Stack()))).Msg("recover from panic")
	}
}

func main() {
	go func() {
		defer Recover()
		var ss []string
		fmt.Println(ss[1])
	}()

	time.Sleep(100 * time.Millisecond)

	fmt.Println("===> enter end")
}
```