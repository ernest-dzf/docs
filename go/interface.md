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
var tflag uint8
var nameOff int32
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

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/eface_fuzhi.png)



我们通过实际代码验证下内存布局。

#### 例子1

```go
package main

import (
	"fmt"
	"unsafe"
)
type People struct {
	Age		[2]byte
}
func main() {
	var emptyInterface interface{}
	var peo People
	emptyInterface = peo
	p := unsafe.Pointer(&emptyInterface)
	typePtrVal := unsafe.Pointer(*((*uintptr)(p)))
	structSize := *(*uintptr)(typePtrVal)
	fmt.Print(structSize)
}
```

输出为：2，表示结构体`People`的size大小。

1. `p`表示`emptyInterface`的地址
2. `typePtrVal`表示eface结构体中第一个field的值
3. `structSize`表示_type结构体中第一个field的值，也就是`People`的type size

可以试着将`Age`改为`[4]byte`，最终输出结果也是符合预期的。

#### 例子2

```go
package main

import (
	"fmt"
	"unsafe"
)
type People struct {
	First	int8
	Second	int32
}
func main() {
	var emptyInterface interface{}
	var peo People
	emptyInterface = peo
	p := unsafe.Pointer(&emptyInterface)
	typePtrVal := unsafe.Pointer(*((*uintptr)(p)))
	ptralign := unsafe.Pointer(uintptr(typePtrVal) + uintptr(21))
	align := *(*int8)(ptralign)
	fmt.Print(align)
}
```

输出为：4

例子2打印出结构体`People`字节对齐情况，表示按照4字节对齐。

21表示`fieldalign`字段在结构_type的地址偏移。

#### 例子3

```go
package main

import (
	"fmt"
	"unsafe"
)
func main() {
	var emptyInterface interface{}
	var num int64 = -12
	emptyInterface = num
	dataPtr := *((*uintptr)(unsafe.Pointer(uintptr(unsafe.Pointer(&emptyInterface)) + uintptr(8))))
	fmt.Println("emptyInterface's data = ", *(*int64)(unsafe.Pointer(uintptr(dataPtr))))
}
```

输出：

```
emptyInterface's data =  -12
```

上面例子我们通过指针打印出了emptyInterface中data指针指向的数据的值，刚好和变量`num`的值一样。

#### data数据存在哪儿？

对于`interrace{}`类型的变量，它的data指针指向的数据存在哪儿呢？

**存在堆中的！**

想想也应该存在堆中，设想如果一个变量`var emptyInterface interface{}`在代码中不断地被赋值，那么它的data指针也是不断指向内存中不同区域的。也就是说需要开辟不同的内存区域用于存放真实的数据。所以data指向的数据肯定是在堆中。

data中的内容会根据实际情况变化，因为golang在函数传参和赋值时都是**值传递**的。

- 如果实际类型是一个**值**，那么interface会保存这个值的一份拷贝。interface会在堆中为这个值分配一块内存。
- 如果实际类型是一个**指针**，那么interface会保存这个指针的一份拷贝。由于data的长度恰好能保存这个指针的内容，所以data存储的就是指针的值。它和实际数据指向的是同一个变量。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/eface3.png)

#### nameOff字段

我们看到`_type`类型有一个`str`字段，类型为`nameOff`。`nameOff`底层类型为`int32`。

这个值是链接器负责嵌入的，相对于可执行文件的元信息的偏移量。元信息会在运行期间，加载到`runtime.moduledata`结构体中（`src/runtime/symtab.go`）

```go
// moduledata records information about the layout of the executable
// image. It is written by the linker. Any changes here must be
// matched changes to the code in cmd/internal/ld/symtab.go:symtab.
// moduledata is stored in statically allocated non-pointer memory;
// none of the pointers here are visible to the garbage collector.
type moduledata struct {
	pclntable    []byte
	ftab         []functab
	filetab      []uint32
	findfunctab  uintptr
	minpc, maxpc uintptr

	text, etext           uintptr
	noptrdata, enoptrdata uintptr
	data, edata           uintptr
	bss, ebss             uintptr
	noptrbss, enoptrbss   uintptr
	end, gcdata, gcbss    uintptr
	types, etypes         uintptr

	textsectmap []textsect
	typelinks   []int32 // offsets from types
	itablinks   []*itab

	ptab []ptabEntry

	pluginpath string
	pkghashes  []modulehash

	modulename   string
	modulehashes []modulehash

	hasmain uint8 // 1 if module contains the main function, 0 otherwise

	gcdatamask, gcbssmask bitvector

	typemap map[typeOff]*_type // offset to *_rtype in previous module

	bad bool // module failed to load and should be ignored

	next *moduledata
}
```



### iface

iface表示有方法的interface。

```go
type iface struct {
    tab  *itab
    data unsafe.Pointer
}
type itab struct {
    inter  *interfacetype
    _type  *_type
    link   *itab
    hash   uint32
    bad    bool
    inhash bool
    unused [2]byte
    fun    [1]uintptr
}

// interface数据类型对应的type
type interfacetype struct {
    typ     _type
    pkgpath name
    mhdr    []imethod
}
```

## 参考

1. http://c.biancheng.net/view/5116.html