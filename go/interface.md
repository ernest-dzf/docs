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

### eface

eface表示不含 method 的 interface 结构。比如`var emptyInterface interface{}`。

eface的具体结构如下：

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/eface1.png)

代码具体实现如下：

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

一共有两个属性构成，一个是类型信息`_type`，一个是数据信息`data`。

对于 Golang 中的大部分数据类型都可以抽象出来 _type 结构，同时针对不同的类型还会有一些其他信息。

eface的整体结构如下：



![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/eface2.png)

可以看到eface包含两个指针，那么空interface占用字节数是多少呢？

```go
package main

import (
	"fmt"
	"unsafe"
)
type Binary int64
func main() {
	var i interface{}
	var a Binary = 3
	i = a
	fmt.Print(unsafe.Sizeof(i))
}
```

输出:

```
16
Process finished with exit code 0
```

机器为64位，指针长度为8字节，两个指针的话，就是16字节，符合预期。

将一个类型T的变量赋值给一个空interface{}的变量，必然会涉及到两个步骤

- \_type字段需要重新指向另外一个\_type类型的值
- data字段也需要重新指向另外一个值

这其中，将不同类型转化成_type类型万能结构的方法，是运行时的`convT2E`方法，在`runtime`包中。

比如：

```go
import (
	"fmt"
	"strconv"
)

type Binary uint64

func main() {
	b := Binary(200)
	any := (interface{})(b)
	fmt.Println(any)
}
```

赋值后的结构图如下：

