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

go的interface是由两种类型来实现的：`iface`和`eface`。

eface表示不含 method 的 interface 结构。比如`var emptyInterface interface{}`。

eface的具体结构如下：

```go
type eface struct {
    _type *_type
    data  unsafe.Pointer
}

type _type struct {
    size       uintptr // type size
    ptrdata    uintptr // size of memory prefix holding all pointers
    hash       uint32  // hash of type; avoids computation in hash tables
    tflag      tflag   // extra type information flags
    align      uint8   // alignment of variable with this type
    fieldalign uint8   // alignment of struct field with this type
    kind       uint8   // enumeration for C
    alg        *typeAlg  // algorithm table
    gcdata    *byte    // garbage collection data
    str       nameOff  // string form
    ptrToThis typeOff  // type for pointer to this type, may be zero
}
```



对于 Golang 中的大部分数据类型都可以抽象出来 _type 结构，同时针对不同的类型还会有一些其他信息。