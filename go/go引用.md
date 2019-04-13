# go语言没有引用传递 #

> There is no pass-by-reference in Go

**go中是没有引用的。**

## 什么是引用变量？ ##

在像 C++ 这样的语言中你可以给一个已经存在的变量定义一个别名，这个别名就被称为引用变量。

	#include <stdio.h>
	
	int main() {
	        int a = 10;
	        int &b = a;
	        int &c = b;
	
	        printf("%p %p %p\n", &a, &b, &c); // 0x7ffe114f0b14 0x7ffe114f0b14 0x7ffe114f0b14
	        return 0;
	}

你可以看到上面 a、b、c 都指向了相同的内存地址，向 a 写入数据将会更改 b、c 的内容。这当你想在不同作用域（即函数调用）里定义引用变量的时候就显得十分的关键。

## Go 语言中并没有引用变量 ##

**与 C++ 不同，Go 语言中每一个定义的变量都占据一个唯一的内存地址**

	package main
	
	import (
	    "fmt"
	)
	
	func main() {
	    var a int
	    var b = &a
	    var c = &a
	
	    fmt.Println(&a, &b, &c) //0xc0420361d0 0xc04204e018 0xc04204e020
	}

在 Go 语言中，不可能创建 2 个变量而这 2 个变量却拥有相同的内存地址。

Go 语言是允许创建 2 个变量它们的内容是同一个指向相同地址的指针，但是这与两个变量共享同一个内存地址完全是两回事。

	package main
	
	import "fmt"
	
	func main() {
	        var a int
	        var b, c = &a, &a
	        fmt.Println(b, c)   // 0x1040a124 0x1040a124
	        fmt.Println(&b, &c) // 0x1040c108 0x1040c110
	}

在这个例子中 b、c 持有的相同的值是 a 的地址，然而 b、c 他们自己的值则存储在各自唯一的地址中。更新 b 的内容 是不会对 c 的内容产生影响的。

## 但 Maps 与 Channels 是引用变量总该对了吧？ ##

错！ Maps 与 Channels 依然不是引用变量。如果他们是引用变量，那么下面这段程序将打印 false.


	package main
	
	import "fmt"
	
	func fn(m map[int]int) {
	        m = make(map[int]int)
	}
	
	func main() {
	        var m map[int]int
	        fn(m)
	        fmt.Println(m == nil)
	}

但是我们确实看到，传参如果传递的是map变量，在函数内部更改map变量，能够影响到函数外部。

	package main
	
	import "fmt"
	
	func fn(m map[int]int) {
		m[3] = 4
	}
	
	func main() {
		var m map[int]int = make(map[int]int)
		fn(m)
		fmt.Printf("m = %v", m)
		//m = map[3:4]
	}


但其实不是。golang中没有引用，map是传值的。它的原理和数组切片、channel类似，map内部维护着一个指针，该指针指向真正的map存储空间。所以即使传参时是对值的复制，但都指向同一块存储空间。

