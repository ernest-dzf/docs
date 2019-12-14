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
    tflag      tflag   // extra type information flags, 一个字节
    align      uint8   // alignment of variable with this type
    fieldalign uint8   // alignment of struct field with this type
    kind       uint8   // enumeration for C
    alg        *typeAlg  // algorithm table，8字节
    gcdata    *byte    // garbage collection data
    str       nameOff  // string form，4个字节
    ptrToThis typeOff  // type for pointer to this type, may be zero
}
var tflag uint8
var nameOff int32
```

一共有两个属性构成，一个是类型信息`_type`，一个是数据信息`data`。

对于 Golang 中的大部分数据类型都可以抽象出来 `_type` 结构，同时针对不同的类型还会有一些其他信息。

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

这个值在编译期间就被确定了，相对于可执行文件的元信息的偏移量。元信息会在运行期间，加载到`runtime.moduledata`结构体中（`src/runtime/symtab.go`）


为了探究`nameoff`背后的机制，顺带探究下golang 的反射机制，我们以下面这个例子说明。

```go

package main

import (
	"fmt"
	"reflect"
)

type TestNameOff struct {
	Id		int64
	Name	string
}

func main() {
	obj := TestNameOff{}
	var eface  interface{}
	eface = obj
	typeName := reflect.TypeOf(eface).Name()
	fmt.Println("typeName = ", typeName)
}
```

`reflect.TypeOf`返回一个`Type`接口类型，这个接口类型有一系列的Method，通过这些Method可以获取到入参`i`（`interface{}`类型）的一些有关类型方面的特征。（详细的Method见https://golang.org/pkg/reflect/#Type）

比如我们可以通过`reflect.TypeOf(eface).Name()`获取到`eface`的Concrete Type 类型名称是`TestNameOff`（上面的例子）。

我们查看`TypeOf`的实现源码，

```go
//src\reflect\type.go

// TypeOf returns the reflection Type that represents the dynamic type of i.
// If i is a nil interface value, TypeOf returns nil.
func TypeOf(i interface{}) Type {
	eface := *(*emptyInterface)(unsafe.Pointer(&i)) 
	return toType(eface.typ)    
}

func toType(t *rtype) Type {
	if t == nil {
		return nil
	}
	return t
}
```

可以看到其实就是一个强制类型转换，将其转换为`*rtype`类型。（https://golang.org/src/reflect/type.go?h=rtype#L299）

```go
// rtype must be kept in sync with ../runtime/type.go:/^type._type.
type rtype struct {
	size       uintptr
	ptrdata    uintptr  // number of bytes in the type that can contain pointers
	hash       uint32   // hash of type; avoids computation in hash tables
	tflag      tflag    // extra type information flags
	align      uint8    // alignment of variable with this type
	fieldAlign uint8    // alignment of struct field with this type
	kind       uint8    // enumeration for C
	alg        *typeAlg // algorithm table
	gcdata     *byte    // garbage collection data
	str        nameOff  // string form
	ptrToThis  typeOff  // type for pointer to this type, may be zero
}
```

其实reflect包里面的`rtype`就是我们前面提到的`_type`类型。

`rtype`实现了接口`Type`所定义的所有方法，这也是为什么我们上面可以在`toType`函数直接返回`*type`类型的`t`的原因。

我们接下来看`rtype`类型的`Name`方法是如何实现的。

```go
// name is an encoded type name with optional extra data.
//
// The first byte is a bit field containing:
//
//	1<<0 the name is exported
//	1<<1 tag data follows the name
//	1<<2 pkgPath nameOff follows the name and tag
//
// The next two bytes are the data length:
//
//	 l := uint16(data[1])<<8 | uint16(data[2])
//
// Bytes [3:3+l] are the string data.
//
// If tag data follows then bytes 3+l and 3+l+1 are the tag length,
// with the data following.
//
// If the import path follows, then 4 bytes at the end of
// the data form a nameOff. The import path is only set for concrete
// methods that are defined in a different package than their type.
//
// If a name starts with "*", then the exported bit represents
// whether the pointed to type is exported.
type name struct {
	bytes *byte
} //https://golang.org/src/reflect/type.go?h=name#L466
func (t *rtype) Name() string {
	if t.tflag&tflagNamed == 0 {
		return ""
	}
	s := t.String()
	i := len(s) - 1
	for i >= 0 && s[i] != '.' {
		i--
	}
	return s[i+1:]
}//https://golang.org/src/internal/reflectlite/type.go?h=Name%28%29
func (t *rtype) String() string {
	s := t.nameOff(t.str).name()//这里很关键，取了rtype类型的str字段
	if t.tflag&tflagExtraStar != 0 {
		return s[1:]
	}
	return s
}//https://golang.org/src/reflect/type.go?h=rtype#L299

func (t *rtype) nameOff(off nameOff) name {
	return name{(*byte)(resolveNameOff(unsafe.Pointer(t), int32(off)))}
}//https://golang.org/src/reflect/type.go?h=rtype#L299

func resolveNameOff(ptrInModule unsafe.Pointer, off nameOff) name {
	if off == 0 {
		return name{}
	}
	base := uintptr(ptrInModule)
	for md := &firstmoduledata; md != nil; md = md.next {
		if base >= md.types && base < md.etypes {
			res := md.types + uintptr(off)
			if res > md.etypes {
				println("runtime: nameOff", hex(off), "out of range", hex(md.types), "-", hex(md.etypes))
				throw("runtime: name offset out of range")
			}
			return name{(*byte)(unsafe.Pointer(res))}
		}
	}

	// No module found. see if it is a run time name.
	reflectOffsLock()
	res, found := reflectOffs.m[int32(off)]
	reflectOffsUnlock()
	if !found {
		println("runtime: nameOff", hex(off), "base", hex(base), "not in ranges:")
		for next := &firstmoduledata; next != nil; next = next.next {
			println("\ttypes", hex(next.types), "etypes", hex(next.etypes))
		}
		throw("runtime: name offset base pointer out of range")
	}
	return name{(*byte)(res)}
}//https://golang.org/src/runtime/type.go?h=resolveNameOff#L180
```

注意`rtype`类型的`String`方法。取了`rtype`类型的`str`字段。

`str`字段决定了我们调用`reflect.TypeOf().Name()`返回的类型名字。

这也是我们在本节开头提到的`nameOff`类型的字段`str`。

**`str`表示什么含义呢？**

`str`表示与某个`rtype`类型变量对应的Concrete Type类型的名称在某个区块中相对于区块起始地址的偏移。

这里的**区块**（moduledata）指的是啥呢？

我们再来看上面`resolveNameOff`的实现。涉及到了一个变量`firstmoduledata`。

```go
var firstmoduledata moduledata  // linker symbol
// moduledata records information about the layout of the executable

// image. It is written by the linker. Any changes here must be

// matched changes to the code in cmd/internal/ld/symtab.go:symtab.

// moduledata is stored in statically allocated non-pointer memory;

// none of the pointers here are visible to the garbage collector.

type moduledata struct {

	pclntable    []byte		//24 byte

	ftab         []functab //24 byte

	filetab      []uint32  //24 byte

	findfunctab  uintptr	//8 byte

	minpc, maxpc uintptr	//16 byte


	text, etext           uintptr	//16 byte

	noptrdata, enoptrdata uintptr	//16 byte

	data, edata           uintptr	// 16 byte

	bss, ebss             uintptr // 16 byte

	noptrbss, enoptrbss   uintptr // 16 byte

	end, gcdata, gcbss    uintptr // 24 byte

	types, etypes         uintptr // 16 byte


	textsectmap []textsect // 24 byte

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



元信息会在运行期，加载到`runtime.moduledata`结构体中。

`firstmoduledata`是runtime包的符号。

通过`readelf -s main`可以获取它的地址。

```
	 ...
	 164: 00000000004dac70    24 OBJECT  GLOBAL DEFAULT    2 runtime.debugCallWrap.stk
   165: 000000000054f1a0   456 OBJECT  GLOBAL DEFAULT    9 runtime.firstmoduledata
   166: 000000000055e640    24 OBJECT  GLOBAL DEFAULT   11 runtime.envs
   ...
```

那么`firstmoduledata`是位于哪个section呢？

通过`readelf -S main`得知，`.noptrdata`段的起始地址为`000000000054a020`。而`firstmoduledata`的地址为`000000000054f1a0`。猜测`firstmoduledata`位于`.noptrdata`段。

```
  ...
  [ 8] .go.buildinfo     PROGBITS         000000000054a000  0014a000
       0000000000000020  0000000000000000  WA       0     0     16
  [ 9] .noptrdata        PROGBITS         000000000054a020  0014a020
       000000000000d0f8  0000000000000000  WA       0     0     32
  [10] .data             PROGBITS         0000000000557120  00157120
       0000000000007050  0000000000000000  WA       0     0     32
  ...
```

使用gdb调试`main`文件。

```
(gdb) info symbol 0x54f1a0
runtime.firstmoduledata in section .noptrdata
(gdb)
```

验证确实是在`.noptrdata`段。

可以查看每个section的起始地址和大小。

```
(gdb) info files
Symbols from "/root/code/src/main/main".
Local exec file:
        `/root/code/src/main/main', file type elf64-x86-64.
        Entry point: 0x454dd0
        0x0000000000401000 - 0x000000000048d301 is .text
        0x000000000048e000 - 0x00000000004dd758 is .rodata
        0x00000000004dd920 - 0x00000000004de590 is .typelink
        0x00000000004de590 - 0x00000000004de5e0 is .itablink
        0x00000000004de5e0 - 0x00000000004de5e0 is .gosymtab
        0x00000000004de5e0 - 0x0000000000549ca3 is .gopclntab
        0x000000000054a000 - 0x000000000054a020 is .go.buildinfo
        0x000000000054a020 - 0x0000000000557118 is .noptrdata
        0x0000000000557120 - 0x000000000055e170 is .data
        0x000000000055e180 - 0x00000000005799f0 is .bss
        0x0000000000579a00 - 0x000000000057c168 is .noptrbss
        0x0000000000400f9c - 0x0000000000401000 is .note.go.buildid
(gdb)
```

那么`firstmoduledata`这个符号相对于`.noptrdata`段的偏移是多少呢？

0x54f1a0 - 0x54a020 = 0x5180

我们又可知`.noptrdata`在ELF文件`main`中的偏移量为0x14a020，那么`firstmoduledata`在ELF文件`main`的偏移量为0x14a020 + 0x5180 = 0x14f1a0

我们使用hexdump来读取`main`文件偏移量0x14f1a0起始的文件内容，就是读取`firstmoduledata`的内容。

比如我们想要知道`.text`段的起始地址，`text`字段在`firstmoduledata`的偏移量为96字节。

0x14f1a0 + 96 = 0x14F200

```shell
[root@localhost]~/code/src/main# hexdump -s 0x14F200 main -n 8
014f200 1000 0040 0000 0000
014f208
[root@localhost]~/code/src/main#
```

考虑到是小端（低尾，尾数在低字节），我们根据hexdump出来的结果，猜测`.text`段在虚拟地址空间的起始地址为0x401000。

通过`readelf -S main`验证一下。

```shell
[root@localhost]~/code/src/main# readelf -S main |head -n 15
共有 25 个节头，从偏移量 0x1c8 开始：

节头：
  [号] 名称              类型             地址              偏移量
       大小              全体大小          旗标   链接   信息   对齐
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000401000  00001000
       000000000008c203  0000000000000000  AX       0     0     16
  [ 2] .rodata           PROGBITS         000000000048e000  0008e000
       000000000004f757  0000000000000000   A       0     0     32
  [ 3] .shstrtab         STRTAB           0000000000000000  000dd760
       00000000000001a1  0000000000000000           0     0     1
  [ 4] .typelink         PROGBITS         00000000004dd920  000dd920
       0000000000000c70  0000000000000000   A       0     0     32
[root@localhost]~/code/src/main#
```

可以看到`.text`段的起始地址确实是`0x401000`。

我们现在想利用hexdump工具获取`firstmoduledata`结构里面的`types`和`etypes`值。

这两个字段也是在二进制可执行文件生成后就确定了的，在执行二进制文件的时候，随着`firstmoduledata`加载到虚拟内存空间中。

根据前面谈到的`firstmoduledata`结构体，我们知道`types`在`firstmoduledata`的偏移量为200字节。

0x14f1a0 + 200 = 0x14F268

0x14f1a0 + 208 = 0x14F270

```shell
[root@localhost]~/code/src/main# hexdump -s 0x14F268 main -n 8
014f268 e000 0048 0000 0000
014f270
[root@localhost]~/code/src/main#
[root@localhost]~/code/src/main# hexdump -s 0x14F270 main -n 8
014f270 d757 004d 0000 0000
014f278
[root@localhost]~/code/src/main#


```

那么`types`的值为`0x48e000`，`etypes`的值为`0x4dd757`。

发现刚好是`.rodata`段的地址空间。（`rtype`是存放在`.rodata`段的么？）

`.rodata`段在ELF文件main中的偏移量为`0x0008e000`。

```shell
[root@localhost]~/code/src/main# readelf -S main|head -n 15
共有 25 个节头，从偏移量 0x1c8 开始：

节头：
  [号] 名称              类型             地址              偏移量
       大小              全体大小          旗标   链接   信息   对齐
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000401000  00001000
       000000000008c2a3  0000000000000000  AX       0     0     16
  [ 2] .rodata           PROGBITS         000000000048e000  0008e000
       000000000004f7f7  0000000000000000   A       0     0     32
  [ 3] .shstrtab         STRTAB           0000000000000000  000dd800
       00000000000001a1  0000000000000000           0     0     1
  [ 4] .typelink         PROGBITS         00000000004dd9c0  000dd9c0
       0000000000000c74  0000000000000000   A       0     0     32
[root@localhost]~/code/src/main#
```

我们再回过来看前面`resolveNameOff`的实现，关注`res := md.types + uintptr(off)`前后附近代码，我们就可以知道`reflect.TypeOf(eface).Name()`的背后实现原理了。

看下面代码：

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

type TestNameOff struct {
	Id		int64
	Name	string
}

func main() {
	obj := TestNameOff{}
	var eface  interface{}
	eface = obj
	typeName := reflect.TypeOf(eface).Name()
	fmt.Println("typeName = ", typeName)

	p := unsafe.Pointer(&eface)
	typePtrVal := unsafe.Pointer(*((*uintptr)(p)))

	strp := *(*uint32)(unsafe.Pointer(uintptr(typePtrVal) + 8 + 8 + 8 +8 +8))
	fmt.Println("_type.str = ", strp)
}
```

输出为：

```
[root@localhost]~/code/src/main# ./main
typeName =  TestNameOff
_type.str =  29024
[root@localhost]~/code/src/main#

```

那么`res := md.types + uintptr(off)`， `md.types`在ELF文件`main`的偏移量是`8e000`，那么`res`在ELF文件的偏移量为`0x8e000 + 29024 = 610656`。

我们猜想ELF文件`main`在偏移量为610656处，保存的是类型`TestNameOff`的名称，也就是字符串TestNameOff。

```shell
[root@localhost]~/code/src/main# hexdump -s 610656 main -C -n 100
00095160  01 00 11 2a 6d 61 69 6e  2e 54 65 73 74 4e 61 6d  |...*main.TestNam|
00095170  65 4f 66 66 00 00 11 2a  6d 61 70 5b 73 74 72 69  |eOff...*map[stri|
00095180  6e 67 5d 69 6e 74 36 34  00 00 11 2a 72 65 66 6c  |ng]int64...*refl|
00095190  65 63 74 2e 66 75 6e 63  54 79 70 65 01 00 11 2a  |ect.funcType...*|
000951a0  72 65 66 6c 65 63 74 6c  69 74 65 2e 4b 69 6e 64  |reflectlite.Kind|
000951b0  01 00 11 2a 72 65 66 6c  65 63 74 6c 69 74 65 2e  |...*reflectlite.|
000951c0  54 79 70 65                                       |Type|
000951c4
[root@localhost]~/code/src/main#
```

**确实如此！**

#### rtype存放在哪个段里

我们前面谈了这么多`rtype`（或者`_type`），那么这个`rtype`存放在哪个段里面呢？

存放在`.rodata`段里面。

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

type TestNameOffVictor struct {
	Id		int64
	Name	string
}

func main() {
	obj := TestNameOffVictor{}
	var eface  interface{}
	eface = obj
	typeName := reflect.TypeOf(eface).Name()
	fmt.Println("typeName = ", typeName)

	p := unsafe.Pointer(&eface)
	typePtrVal := unsafe.Pointer(*((*uintptr)(p)))

	fmt.Printf("typePtrVal = %p\n", typePtrVal)

	strp := *(*uint32)(unsafe.Pointer(uintptr(typePtrVal) + 8 + 8 + 8 +8 +8))
	fmt.Println("_type.str = ", strp)
}
```

我们打印`typePtrVal`的值，就是打印`_type`类型结构的地址，

```shell
[root@localhost]~/code/src/main# ./main
typeName =  TestNameOffVictor
typePtrVal = 0x4aca20
_type.str =  36685
[root@localhost]~/code/src/main#
```

可以看到其值为`0x4aca20`，这个地址是位于`.rodata`段的。

#### 把一个具体的值赋值给一个eface

当将某个类型赋值给空接口时，由于空接口没有方法，所以空接口`eface`的tab会直接指向数据的具体类型，实际上就是前面谈到的`_type`字段。

在Go的reflect包中，`reflect.TypeOf`和`reflect.ValueOf`的参数都是空接口，因此所有参数都会先转换为空接口类型。这样反射就实现了对所有参数类型获取实际数据类型的统一。

我们看下面这个例子

```go
package main

import (
    "fmt"
)

type MyInterface interface {
    Print()
}

type MyStruct struct{}
func (ms MyStruct) Print() {}

func main() {
    x := 1
    var y interface{} = x
    var s MyStruct
    var t MyInterface = s
    fmt.Println(y, t)
}

```

`var y interface{} = x`，将`x`赋值给一个空的interface。编译器后面替我们干了些什么呢？

### iface

iface表示有方法的interface。

```go
type iface struct {
    tab  *itab
    data unsafe.Pointer
}// 目前这个和go 版本无关，不同go 版本都是一样的

type itab struct {
        inter  *interfacetype
        _type  *_type
        link   *itab
        bad    int32
        inhash int32      // has this itab been added to hash?
        fun    [1]uintptr // variable sized
}// go 版本 go1.8.3，不同版本，itab数据结构可能不一样

// interface数据类型对应的type
type interfacetype struct {
    typ     _type
    pkgpath name
    mhdr    []imethod
}// 目前这个和go 版本无关，不同go版本都是一样的
```



可以看到，`iface`包含2部分，`*itab`类型的`tab`字段和`unsafe.Pointer`类型的`data`字段。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/iface1.png)

`data`和前面谈到的`eface`中的data一样的，指向实际的数据部分。

下面例子可以验证，

```go
package main

import (
	"fmt"
	"unsafe"
)

type MyInterface interface {
	Print()
}

type MyStruct struct{
	Name		string
}
func (ms MyStruct) Print() {}

func main() {
	obj := MyStruct{"victor"}
	var iface MyInterface
	iface = obj

	//dataPtr表示iface中的data指针
	dataPtr := unsafe.Pointer(uintptr(unsafe.Pointer(&iface)) + uintptr(8))

	//取出data指针的值
	dataPtrVal := *(*uintptr)(dataPtr)

	//data指针的值实际上是Mystruct的数据的地址
	assertPtr := *(*string)(unsafe.Pointer(dataPtrVal))
	fmt.Println(assertPtr)//输出 victor
}

```



`tab`字段是`iface`不同于`eface`的关键数据结构。（go版本为go1.11.13）

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/iface_itable.png)



`iface`的整体结构如下：

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/iface_all.png)

## 参考

1. http://c.biancheng.net/view/5116.html