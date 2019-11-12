# golang string的底层数据结构

先来看官方的实现，位置位于`src/runtime/string.go`。

```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}	
```

string实际上就是一个结构体。

我们实际验证下：



```go
package main

import (
	"fmt"
	"unsafe"
)

type stringStruct struct {
	str unsafe.Pointer  // 指向一个 [len]byte 的数组
	len int             // 长度
}
func main() {

	test := "hello"
	p := (*stringStruct)(unsafe.Pointer(&test))
	fmt.Printf("p's value is %p\n", p)//p's value is 0xc04203c1d0
	fmt.Printf("p.str = %p\n", p.str)//p.str = 0x4b3c99
	fmt.Printf("p.len = %d\n", p.len)//p.len = 5

	c := make([]byte, p.len)
	for i := 0; i < p.len; i++ {
		tmp := uintptr(unsafe.Pointer(p.str))           // 指针类型转换通过unsafe包
		c[i] = *(*byte)(unsafe.Pointer(tmp + uintptr(i))) // 指针运算只能通过uintptr
	}
	fmt.Printf("c = %v\n", c)         // c = [104 101 108 108 111]
	fmt.Printf("c's values = %s\n", string(c)) // c's values = hello

	test = test + " victor"

	p3 := (*stringStruct)(unsafe.Pointer(&test))

	fmt.Printf("p3's value is %p\n", p3)	//p3's value is 0xc04203c1d0
	fmt.Printf("p3.str = %p\n", p3.str)	//p3.str = 0xc04203c230
	fmt.Printf("p3.len = %d\n", p3.len)//p3.len = 12
}

```



string结构体中的指针指向实际的字符串数据。

如果像`test = test + " victor"`这样使用，实际上涉及到了内存拷贝。先将`test.str`指向的字符串拷贝一份到新的内存空间，然后再补足拼接的字符串` victor`。

也就是说字符串拼接实际上改动的是`stringStruct`结构体中的`str`指针和`len`长度值。