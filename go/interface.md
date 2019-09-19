# golang中的interface

## 两种interface

根据interface是否包含method，有两种interface。

```go
package main

type IBird interface {
	Fly()
	Sing()
}
func main() {
	var emptyInterface interface{}
	var bird IBird
}
```

上面`emptyInterface`和`bird`就是两种interface。`emptyInterface`是空的interface，`bird`是包含method的interface。

## interface底层结构

eface表示不含 method 的 interface 结构。

对于 Golang 中的大部分数据类型都可以抽象出来 _type 结构，同时针对不同的类型还会有一些其他信息。