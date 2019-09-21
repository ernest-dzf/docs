# go并发 #
## WaitGroup ##

先来个例子：

```go
func main() {
	var wg sync.WaitGroup

	wg.Add(2)
	go func() {
		time.Sleep(2*time.Second)
		fmt.Println("1号完成")
		wg.Done()
	}()
	go func() {
		time.Sleep(2*time.Second)
		fmt.Println("2号完成")
		wg.Done()
	}()
	wg.Wait()	// Wait blocks until the WaitGroup counter is zero.
	fmt.Println("好了，大家都干完了，放工")
}
```

一个很简单的例子，一定要例子中的2个goroutine同时做完，才算是完成，先做好的就要等着其他未完成的，所有的goroutine要都全部完成才可以。

## channel通知 ##


我们都知道一个goroutine启动后，我们是无法控制它的，大部分情况是等待它自己结束。

那么如果这个goroutine是一个不会自己结束的后台goroutine呢？比如监控，会一直运行的。

这种情况，一种傻瓜式的办法是全局变量，其他地方通过修改这个变量完成结束通知，然后后台goroutine不停地检查这个变量，如果发现被通知关闭了，就自我结束。

这种方式也可以，但是首先我们要保证这个变量在多线程下的安全，基于此，有一种更好的方式：chan + select 。

	func main() {
		stop := make(chan bool)
	
		go func() {
			for {
				select {
				case <-stop:
					fmt.Println("监控退出，停止了...")
					return
				default:
					fmt.Println("goroutine监控中...")
					time.Sleep(2 * time.Second)
				}
			}
		}()
	
		time.Sleep(10 * time.Second)
		fmt.Println("可以了，通知监控停止")
		stop<- true
		//为了检测监控过是否停止，如果没有监控输出，就表示停止了
		time.Sleep(5 * time.Second)
	
	}

有了以上的逻辑，我们就可以在其他goroutine中，给stop chan发送值了，例子中是在main goroutine中发送的，控制让这个监控的goroutine结束。

这种chan+select的方式，是比较优雅地结束一个goroutine的方式，不过这种方式也有局限性，如果有很多goroutine都需要控制结束怎么办呢？如果这些goroutine又衍生了其他更多的goroutine怎么办呢？如果一层层的无穷尽的goroutine呢？

这就非常复杂了，即使我们定义很多chan也很难解决这个问题，因为goroutine的关系链就导致了这种场景非常复杂。

## 初识context ##


上面说的这种场景是存在的，比如一个网络请求Request，每个Request都需要开启一个goroutine做一些事情，这些goroutine又可能会开启其他的goroutine。

所以我们需要一种可以跟踪goroutine的方案，才可以达到控制它们的目的，这就是Go语言为我们提供的Context，称之为上下文非常贴切，它就是goroutine的上下文。

下面我们就使用Go Context重写上面的示例。

	func main() {
		ctx, cancel := context.WithCancel(context.Background())
		go func(ctx context.Context) {
			for {
				select {
				case <-ctx.Done():
					fmt.Println("监控退出，停止了...")
					return
				default:
					fmt.Println("goroutine监控中...")
					time.Sleep(2 * time.Second)
				}
			}
		}(ctx)
	
		time.Sleep(10 * time.Second)
		fmt.Println("可以了，通知监控停止")
		cancel()
		//为了检测监控过是否停止，如果没有监控输出，就表示停止了
		time.Sleep(5 * time.Second)
	
	}


有几个点解释下：

1. context.Background()

	context.Background()返回一个non-nil, empty Context，这个context是所有Context的root，不能够被cancel。

2. context.WithCancel()

	WithCancel返回一个继承的Context,这个Context在父Context被cancel时关闭自己的Done通道（就是说`ctx.Done()`方法返回一个closed的通道），或者在自己被cancel的时候关闭自己的Done通道。父Context为context.Background()，不能被cancel。返回的继承的Context是可以被cancel的。main函数后面的cancel调用就cancel了这个继承的Context，同时在子协程中监控到了这个cancel动作。


下面是Context接口的定义：

	type Context interface {
		Deadline() (deadline time.Time, ok bool)
		Done() <-chan struct{}
	
		Err() error
	
		Value(key interface{}) interface{}
	}

可以看到`ctx.Done()`返回一个channel，上面例子中`cancel()`函数触发了`ctx.Done()`返回一个关闭的通道，然后子协程中，监控到了这个事件。（读取一个已关闭的 channel 时，总是能读取到对应类型的零值）


## Context控制多个goroutine ##

使用Context控制一个goroutine的例子如上，非常简单，下面我们看看控制多个goroutine的例子，其实也比较简单。

	func main() {
		ctx, cancel := context.WithCancel(context.Background())
		go watch(ctx,"【监控1】")
		go watch(ctx,"【监控2】")
		go watch(ctx,"【监控3】")
	
		time.Sleep(10 * time.Second)
		fmt.Println("可以了，通知监控停止")
		cancel()
		//为了检测监控过是否停止，如果没有监控输出，就表示停止了
		time.Sleep(5 * time.Second)
	}
	
	func watch(ctx context.Context, name string) {
		for {
			select {
			case <-ctx.Done():
				fmt.Println(name,"监控退出，停止了...")
				return
			default:
				fmt.Println(name,"goroutine监控中...")
				time.Sleep(2 * time.Second)
			}
		}
	}


示例中启动了3个监控goroutine进行不断的监控，每一个都使用了Context进行跟踪，当我们使用cancel函数通知取消时，这3个goroutine都会被结束。

这就是Context的控制能力，它就像一个控制器一样，按下开关后，所有基于这个Context或者衍生的子Context都会收到通知，这时就可以进行清理操作了，最终释放goroutine，这就优雅地解决了goroutine启动后不可控的问题。


## context接口 ##

再来看下context的接口定义。

	type Context interface {
		Deadline() (deadline time.Time, ok bool)
	
		Done() <-chan struct{}
	
		Err() error
	
		Value(key interface{}) interface{}
	}

- `Deadline`

	获取设置的截止时间。第一个返回是截止时间，到了这个时间点，Context会自动发起取消请求；第二个返回值ok==false时表示没有设置截止时间，如果需要取消的话，需要调用取消函数进行取消。

- `Done`

	返回一个只读的chan，类型为struct{}，我们在goroutine中，如果该方法返回的chan可以读取，则意味着parent context已经发起了取消请求，我们通过Done方法收到这个信号后，就应该做清理操作，然后退出goroutine，释放资源。

- `Err`

	返回取消的错误原因，因为什么Context被取消。

- `Value`

	获取该Context上绑定的值，是一个键值对，所以要通过一个Key才可以获取对应的值，这个值一般是线程安全的。

以上四个方法中常用的就是`Done()`了。

Context取消的时候，我们就可以得到一个关闭的chan，关闭的chan是可以读取的，所以只要可以读取的时候，就意味着收到Context取消的信号了，以下是这个方法的经典用法。


	  func Stream(ctx context.Context, out chan<- Value) error {
	  	for {
	  		v, err := DoSomething(ctx)
	  		if err != nil {
	  			return err
	  		}
	  		select {
	  		case <-ctx.Done():
	  			return ctx.Err()
	  		case out <- v:
	  		}
	  	}
	  }


Context接口并不需要我们实现，Go内置已经帮我们实现了2个，我们代码中最开始都是以这两个内置的作为最顶层的partent context，衍生出更多的子Context。


	var (
		background = new(emptyCtx)
		todo       = new(emptyCtx)
	)
	
	func Background() Context {
		return background
	}
	
	func TODO() Context {
		return todo
	}

一个是`Background`，主要用于main函数、初始化以及测试代码中，作为Context这个树结构的最顶层的Context，也就是根Context。

一个是`TODO`,它目前还不知道具体的使用场景，如果我们不知道该使用什么Context的时候，可以使用这个。


它们两个本质上都是emptyCtx结构体类型，是一个不可取消，没有设置截止时间，没有携带任何值的Context。

	type emptyCtx int
	
	func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
		return
	}
	
	func (*emptyCtx) Done() <-chan struct{} {
		return nil
	}
	
	func (*emptyCtx) Err() error {
		return nil
	}
	
	func (*emptyCtx) Value(key interface{}) interface{} {
		return nil
	}


这就是emptyCtx实现Context接口的方法，可以看到，这些方法什么都没做，返回的都是nil或者零值。


## Context的继承衍生 ##

有了如上的根Context，那么是如何衍生更多的子Context的呢？这就要靠context包为我们提供的With系列的函数了。

	func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
	func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
	func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
	func WithValue(parent Context, key, val interface{}) Context


这四个With函数，接收的都有一个parent参数，就是父Context，我们要基于这个父Context创建出子Context的意思，这种方式可以理解为子Context对父Context的继承，也可以理解为基于父Context的衍生。

通过这些函数，就创建了一颗Context树，树的每个节点都可以有任意多个子节点，节点层级可以有任意多个。

- `WithCancel`函数，传递一个父Context作为参数，返回子Context，以及一个取消函数用来取消Context。
- `WithDeadline`函数，和WithCancel差不多，它会多传递一个截止时间参数，意味着到了这个时间点，会自动取消Context，当然我们也可以不等到这个时候，可以提前通过取消函数进行取消。
- `WithTimeout`函数，和WithDeadline基本上一样，这个表示是超时自动取消，是多少时间后自动取消Context的意思。
- `WithValue`函数，和取消Context无关，它是为了生成一个绑定了一个键值对数据的Context，这个绑定的数据可以通过Context.Value方法访问到，后面我们会专门讲。


大家可能留意到，前三个函数都返回一个取消函数CancelFunc，这是一个函数类型，它的定义非常简单。

	type CancelFunc func()

这就是取消函数的类型，该函数可以取消一个Context，以及这个节点Context下所有的所有的Context，不管有多少层级。

## WithValue传递元数据 ##

通过Context我们也可以传递一些必须的元数据，这些数据会附加在Context上以供使用。

	var key string="name"
	
	func main() {
		ctx, cancel := context.WithCancel(context.Background())
		//附加值
		valueCtx:=context.WithValue(ctx,key,"【监控1】")
		go watch(valueCtx)
		time.Sleep(10 * time.Second)
		fmt.Println("可以了，通知监控停止")
		cancel()
		//为了检测监控过是否停止，如果没有监控输出，就表示停止了
		time.Sleep(5 * time.Second)
	}
	
	func watch(ctx context.Context) {
		for {
			select {
			case <-ctx.Done():
				//取出值
				fmt.Println(ctx.Value(key),"监控退出，停止了...")
				return
			default:
				//取出值
				fmt.Println(ctx.Value(key),"goroutine监控中...")
				time.Sleep(2 * time.Second)
			}
		}
	}

在前面的例子，我们通过传递参数的方式，把name的值传递给监控函数。在这个例子里，我们实现一样的效果，但是通过的是Context的Value的方式。

我们可以使用context.WithValue方法附加一对K-V的键值对，这里Key必须是**等价性**的，也就是具有可比性；Value值要是**线程安全**的。

这样我们就生成了一个新的Context，这个新的Context带有这个键值对，在使用的时候，可以通过Value方法读取ctx.Value(key)。

Context 使用原则：

1. 不要把Context放在结构体中，要以参数的方式传递

2. 以Context作为参数的函数方法，应该把Context作为第一个参数，放在第一位。

3. 给一个函数方法传递Context的时候，不要传递nil，如果不知道传递什么，就使用context.TODO

4. Context的Value相关方法应该传递必须的数据，不要什么数据都使用这个传递

5. Context是线程安全的，可以放心的在多个goroutine中传递



参考文章：[这里](https://www.flysnow.org/2017/05/12/go-in-action-go-context.html)