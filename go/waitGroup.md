# WaitGroup用法

## 解决什么问题？

go支持超大并发。但是如何控制多个并发协程之间的同步关系呢？

比如下面这段代码：

```go
package main

import (
    "fmt"
    "time"
)

func main(){
    for i := 0; i < 100 ; i++{
        go fmt.Println(i)
    }
    time.Sleep(time.Second)
}
```

主协程需要等待1秒，以便让所有100个子协程都执行完毕，然后主协程再退出。不然的话，主协程先退出了，而那100个子协程有的还没有执行完毕，子协程也退出了，导致有些子协程没有打印出数字。

这种等待1秒的方法不是好办法。我们不知道各个子协程需要执行多久，我们也不知道主协程需要sleep多久。

有什么办法解决这种问题呢？

可以利用channel，类似下面这种

```go
func main() {
    c := make(chan bool, 100)
    for i := 0; i < 100; i++ {
        go func(i int) {
            fmt.Println(i)
            c <- true
        }(i)
    }

    for i := 0; i < 100; i++ {
        <-c
    }
}
```

这可以达到目的，但是有点儿大材小用。而且如果协程多的话，我们需要申请很多个channel，对资源消耗比较多。

go中有一个工具`sync.WaitGroup`可以更加方便地达到目的。

## 官方例子

先来看一个官方的例子

```go
package main

import (
	"sync"
	"fmt"
)

type httpPkg struct{}

func (httpPkg) Get(url string){
	fmt.Println(url)
}

var http httpPkg

func main() {
	var wg sync.WaitGroup
	var urls = []string{
		"http://www.golang.org/",
		"http://www.google.com/",
		"http://www.somestupidname.com/",
	}
	for _, url := range urls {
		// Increment the WaitGroup counter.
		wg.Add(1)
		// Launch a goroutine to fetch the URL.
		go func(url string) {
			// Decrement the counter when the goroutine completes.
			defer wg.Done()
			// Fetch the URL.
			http.Get(url)
		}(url)
	}
	// Wait for all HTTP fetches to complete.
	wg.Wait()
}
```

## WaitGroup对象不是一个引用类型

`WaitGroup`对象不是一个引用类型，在通过函数传值的时候需要使用地址。

```go
package main

import (
	"fmt"
	"sync"
)

var printCnt int64
func Print(cnt int64, wg *sync.WaitGroup) {
	fmt.Printf("Print: %d\n", cnt)
	printCnt++
	wg.Done()
}

func main() {

	wg := sync.WaitGroup{}
	for i := int64(1); i <= 100; i++ {
		wg.Add(1)
		go Print(i, &wg)
	}
	wg.Wait()
	fmt.Print(printCnt)
	return
}
```

可以看到函数`Print`的第二个参数类型是`*sync.WaitGroup`。

如果参数类型是`sync.WaitGroup`的话，会有死锁问题。原因是传递的是`sync.WaitGroup`变量的拷贝。

这样在主协程中，`wg`就一直`Add(1)`，但是没有释放，导致死锁。

## 计数器不能为负值

`wg.Add(cnt)`中的cnt可以是负数，效果类似`wg.Done()`，就是在计数器（`WaitGroup counter`）上减掉1。

但是你需要确保，减掉1之后的计数器的值不能是负数，否则会panic。