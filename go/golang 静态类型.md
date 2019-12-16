# golang 静态类型

静态类型(static type)是变量声明时的声明类型。

变量声明、new方法创建结构体(struct)的类型可以认为是静态类型。

接口(interface)类型的变量还有一个动态类型，它是运行时赋值给这个变量的具体的值的类型(当然值为nil的时候没有动态类型)。

一个变量的动态类型在运行时可能改变，这主要依赖它的赋值。

```golang
var x interface{}  // x 为零值 nil,静态类型为 interface{}
var v *T           // v 为零值 nil, 静态类型为 *T
x = 42             // x 的值为 42,动态类型为int, 静态类型为interface{}
x = v              // x 的值为 (*T)(nil)， 动态类型为 *T, 静态类型为 *T
```

