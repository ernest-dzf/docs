# golang方法接收者为指针和非指针的区别 #

方法接收者为指针，我们可以通过方法改变该接收者的属性
```
	package main
	
	import "fmt"
	
	type T struct {
		Name string
	}
	
	func (t T) M1() {
		t.Name = "name1"
	}
	
	func (t *T) M2() {
		t.Name = "name2"
	}
	
	func main() {
		t1 := T{"t1"}
	
		fmt.Println("M1调用前：", t1.Name)
		t1.M1()
		fmt.Println("M1调用后：", t1.Name)
	
		fmt.Println("M2调用前：", t1.Name)
		t1.M2()
		fmt.Println("M2调用后：", t1.Name)
	}
```
输出为：
		
	M1调用前： t1
	M1调用后： t1
	M2调用前： t1
	M2调用后： name2

上面例子可以看到，`M2`方法更改了接收者的属性；而`M1`方法没有更改接收者的属性。区别是一个接收者是指针，一个接收者不是指针。
	
先来约定一下：接收者可以看作是函数的第一个参数，即这样的：`func M1(t T)`, `func M2(t *T)`。 

当调用 t1.M1() 时相当于 M1(t1) ，实参和行参都是类型 T。此时在M1()中的t只是t1的值拷贝，所以M1()的修改影响不到t1。

当调用 t1.M2() => M2(t1)，这是将 T 类型传给了 *T 类型， **go可能会取 t1 的地址传进去** ： M2(&t1)。所以 M2() 的修改可以影响 t1 。

**T 类型的变量这两个方法都是拥有的。**




下面声明一个 *T 类型的变量，并调用 M1() 和 M2() 。

    t2 := &T{"t2"}
    
    fmt.Println("M1调用前：", t2.Name)
    t2.M1()
    fmt.Println("M1调用后：", t2.Name)
    
    fmt.Println("M2调用前：", t2.Name)
    t2.M2()
    fmt.Println("M2调用后：", t2.Name)

输出结果为：

	M1调用前： t2
	M1调用后： t2
	M2调用前： t2
	M2调用后： name2

t2.M1() => M1(t2)， t2 是指针类型， 取 t2 的值并拷贝一份传给 M1。

t2.M2() => M2(t2)，都是指针类型，不需要转换。

*T 类型的变量也是拥有这两个方法的。

**对于类型A来说，如果他的对象实现了方法F，那么他的指针也实现了方法F；**

**反过来，如果类型A的指针实现了方法F，golang认为他的对象没有实现方法F。**

比如下面这样：


	package main
	
	import "fmt"
	
	type IntF interface {
		M1()
		M2()
	}
	
	type T struct {
		Name string
	}
	
	func (t *T) M1() {
		t.Name = "name1"
		fmt.Print("hhh")
	}
	
	func (t T) M2() {
		t.Name = "name2"
	}
	
	func main() {
		var t2, t3 IntF
		t1 := T{"t1"}
		t2 = t1//会提示错误
		t3 = &t1//正确
		t3.M1()
		t2.M1()
	}

`&t1`作为指针，被认为实现了`M1`和`M2`方法（`M2`方法的接收者为对象，而不是指针）。

`t1`作为对象，并不被认为实现了`M1`方法。

但是：

**无论某个方法的Receiver是对象（值）还是指针，都可以通过对象（值）或者指针来进行方法调用。**

再仔细想一想也是非常合理的，因为Go内部可以轻松地找到一个对象（值）的地址，自然就能构建出指向它的指针（`&obj`），调用Receiver为指针的方法；另外也很容易通过指针找到其指向的对象（值）（`*p`），自然也能轻松调用Receiver为对象（值）的方法。

**但是当你拥有一个对象的时候，你是可能拿不到这个对象的地址的！！**

比如，

```go
 // Sample program to show how you can't always get the
 // address of a value.
 package main

 import "fmt"

 // duration is a type with a base type of int.
 type duration int

 // format pretty-prints the duration value.
 func (d *duration) pretty() string {
     return fmt.Sprintf("Duration: %d", *d)
 }

 // main is the entry point for the application.
 func main() {
	 var x duration
	 x = 42
	 fmt.Println(x.pretty())

     duration(42).pretty()

     // cannot call pointer method on duration(42)
     // cannot take the address of duration(42)
 }
```



## 嵌套类型 ##
声明一个类型 S，将 T 嵌入进去。

	package main
	
	import "fmt"
	
	type IntF interface {
		M1()
		M2()
	}
	
	type T struct {
		Name string
	}
	
	func (t T) M1() {
		t.Name = "name1"
	}
	
	func (t *T) M2() {
		t.Name = "name2"
	}
	
	type S struct {
		T
	}
	
	func main() {
		t1 := T{"t1"}
		s := S{t1}
	
		fmt.Println("M1调用前：", s.Name)
		s.M1()
	
		fmt.Println("M1调用后：", s.Name)
	
		fmt.Println("M2调用前：", s.Name)
	
		s.M2()
	
		fmt.Println("M2调用后：", s.Name)
	
		fmt.Println("t1.Name = ", t1.Name)
	}

将 T 嵌入 S， 那么 T 拥有的方法和属性 S 也是拥有的，但是接收者却不是 S 而是 T。

所以 s.M1() 相当于 M1(t1) 而不是 M1(s)。

最后 t1 的值没有改变，因为我们嵌入的是 T 类型，所以 S{t1} 的时候是将 t1 拷贝了一份。

参考：[这里](https://www.cnblogs.com/zlingh/p/5701785.html)