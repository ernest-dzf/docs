# golang变量内存布局

## Struct

```go
package main

import (
"fmt"
"unsafe"
)
type Qu struct {
	Uin		uint8
	Sec		int16
	Str		int64
	Forth	int8
}
func main() {
	ptrQu := &Qu{1,1234, 6789000,4}
	ptrUin := (*uint64)(unsafe.Pointer(uintptr(unsafe.Pointer(ptrQu)) + uintptr(8)))
	fmt.Printf("str = %d\n",*ptrUin)
	ptrSec := (*int16)(unsafe.Pointer(uintptr(unsafe.Pointer(ptrQu)) + uintptr(2)))

	fmt.Printf("sec = %d\n", *ptrSec)

	fmt.Printf("size = %d\n", unsafe.Sizeof(*ptrQu))
}
```

输出:

```
str = 6789000
sec = 1234
size = 24
```

虽然Uin字段是一个字节，但是需要字节对齐，所以以8字节对齐后，Qu的size是24字节。整体内存布局如下：

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/struct_layout.png)