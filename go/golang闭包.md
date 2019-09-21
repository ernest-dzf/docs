# golang闭包

先来看这个例子：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {

	wg := sync.WaitGroup{}
	for i := int64(0); i < 10; i++ {
		wg.Add(1)
		go func(j int64) {
			fmt.Printf("j = %d, i = %d\n", j,i)
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

输出结果是？

```
j = 9, i = 10
j = 6, i = 10
j = 7, i = 10
j = 8, i = 10
j = 1, i = 10
j = 3, i = 10
j = 4, i = 10
j = 2, i = 9
j = 5, i = 10
j = 0, i = 10
```

上面是某一次运行时的结果，是否符合你的预期呢？

变量`i`是在主协程定义的，