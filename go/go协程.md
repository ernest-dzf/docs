# go 协程 #

goroutine的本质是协程，是实现并行计算的核心。

不同的是，Golang 在 runtime、系统调用等多方面对 goroutine 调度进行了封装和处理，当遇到长时间执行或者进行系统调用时，会主动把当前 goroutine 的CPU (P) 转让出去，让其他 goroutine 能被调度并执行，也就是 Golang 从语言层面支持了协程。


在分析原理之前，了解下一些关键性术语的概念。

## 并发 ##

一个cpu上能同时执行多项任务，在很短时间内，cpu来回切换任务执行(在某段很短时间内执行程序a，然后又迅速切换到程序b去执行)，宏观上是同时的，微观仍是顺序执行，这样看起来多个任务像是同时执行，这就是并发。

## 并行 ##

当系统有多个CPU时,每个CPU同一时刻都运行任务，互不抢占自己所在的CPU资源，同时进行，称为并行。

## 进程 ##
cpu在切换程序的时候，如果不保存上一个程序的状态（也就是我们常说的context--上下文），直接切换下一个程序，就会丢失上一个程序的一系列状态，于是引入了进程这个概念，用以划分好程序运行时所需要的资源。

## 线程 ##

cpu切换多个进程的时候，会花费不少的时间，因为切换进程需要切换到内核态，内核态的系统调用耗费资源比较多，耗时也比较久，进程一旦多起来，cpu调度会消耗一大堆资源，因此引入了线程的概念。线程本身几乎不占有资源，他们共享进程里的资源，内核调度起来不会像进程切换那么耗费资源。

这里说的线程指的还是内核线程。或者说是是**轻量级进程**（内核支持的用户线程）。进行切换的时候需要系统调用。

## 协程 ##

协程拥有自己的寄存器上下文和栈（栈建在堆上）。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。因此，协程能保留上一次调用时的状态（即所有局部状态的一个特定组合），每次过程重入时，就相当于进入上一次调用的状态，换种说法：进入上一次离开时所处逻辑流的位置。线程和进程的操作是由程序触发系统接口，最后的执行者是系统；协程的操作执行者则是用户自身程序，goroutine也是协程。

这里协程对于的是恐龙书中的多对多模型。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/lwp2.JPG)

# go 协程示例

## 示例channel的使用

go协程中，使用channel进行通信。

channel 可以看成一个 FIFO 队列，对 FIFO 队列的读写都是原子的操作，不需要加锁。

读取一个已关闭的 channel 时，总是能读取到对应类型的零值，为了和读取非空未关闭 channel 的行为区别，可以使用两个接收值：

	// ok is false when ch is closed
	v, ok := <-ch


当未为channel分配内存时，channel就是nil channel，比如：

	var ch1 chan int

ch1就是一个nil channel，nil channel会永远阻塞对该channel的读写。

所以，可以将某个channel设置为nil，进行强制阻塞，对于select分支来说，就是强制禁用此分支。

golang 中大部分类型都是值类型(只有 slice / channel / map 是引用类型)，读/写类型是值类型的 channel 时，如果元素 size 比较大时，应该使用指针代替，避免频繁的内存拷贝开销。

对于使用select的case，如果多个case同时满足条件，那么会随机选一个去执行，比如：

	package main
	
	import (
	        "fmt"
	        "time"
	)
	
	func main() {
	        ch1 := make(chan int)
	        ch2 := make(chan int)
	
	        go func(ch chan int) { <-ch }(ch1)
	        go func(ch chan int) { ch <- 2 }(ch2)
	
	        time.Sleep(time.Second)
	
	        for {
	                select {
	                case ch1 <- 1:
	                        fmt.Println("Send operation on ch1 works!")
	                case <-ch2:
	                        fmt.Println("Receive operation on ch2 works!")
	                default:
	                        fmt.Println("Exit now!")
	                        return
	                }
	        }
	}
	
可以知道，上面示例中，当执行到select语句块时，两个case（分别是 ch1 <- 1 和 <-ch2）都已经ready了，那么这里

下面是一个使用channel的例子。

	package main
	
	import (
	    "fmt"
	    "time"
	)
	
	func main() {
	
	    /*1.简单的channel队列现进先出特性
	      *在主线程上建立单个协程，完成某项工作后，关闭该管道,关闭与否为可选项。
	      *停止一段阻塞有两种方式，chanInt内能读取到值，即非空;chanInt被关闭
	    */
	    if false {
	        var chanInt chan int = make(chan int, 10)
	        go func() {
	            fmt.Println("新线程完成了某项工作")
	            defer close(chanInt)
	            chanInt <- 1
	            chanInt <- 2
	            chanInt <- 3
	            chanInt <- 4
	
	        }()
	
	        x := <-chanInt
	        fmt.Println(x)
	        x = <-chanInt
	        fmt.Println(x)
	        x = <-chanInt
	        fmt.Println(x)
	        x = <-chanInt
	        fmt.Println(x)
	    }
	
	    /*2.简单的select使用
	      *使用select只会在符合的语句上走一次，即使两个管道都赋值了，先监测到chan2传值以后，select走完case2就已经结束了
	      *管道对象是引用类型，方法传值并不是值的拷贝，而是可以确切地修改队列内容
	    */
	    if false {
	        chan1 := make(chan int)
	        chan2 := make(chan int)
	        go test2(chan1, chan2)
	        for {
	            select {
	            case <-chan1:
	                fmt.Println("chan1收到值")
	            case x := <-chan2:
	                fmt.Println("chan2收到值", x)
	            default:
	                {
	                    fmt.Println("结束了")
	                    break
	                }
	            }
	        }
	
	    }
	
	    /*如何让主gorouting控制单协程结束
	    */
	    if false {
	        stopFlag := make(chan bool)
	        out := make(chan bool)
	
	        go func() {
	            defer close(out)
	            fmt.Println("完成任务ing")
	
	            select {
	            case <-stopFlag:
	                {
	                    fmt.Println("任务结束")
	                }
	            }
	        }()
	        stopFlag <- true
	        <-out
	    }
	
	    /*
	     *select没有满足条件时会被阻塞,一直没有注值就会死锁
	     *引入超时机制，5秒注值显然大于了超时设定3秒，所以超时了
	    */
	    if true {
	        chanInt := make(chan int, 1)
	        out := make(chan int, 1)
	        go func() {
	            go func() {
	                time.Sleep(5 * time.Second)
	                chanInt <- 5
	
	            }()
	            select {
	            case x := <-chanInt:
	                fmt.Println(x)
	            case <-time.After(3*time.Second):
	                fmt.Println("超时了")
	            }
	            out <- 1
	        }()
	
	        <-out
	    }
	}
	func test2(a chan int, b chan int) {
	    b <- 6
	    b <- 7
	    a <- 5
	}


