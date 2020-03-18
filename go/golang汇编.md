# golang 汇编

## 函数不被内联

我们有时候不想一个函数被内联，可以这样，

```go
//go:noinline
func appendStr(word string) string {
    return "new " + word
}
```

在函数定义上面添加`//go:noinline`注释，go的编译器就会知道，下面这个函数不要内联展开。

## go文件和汇编文件混合

golang有一个有趣的特性。你可以在一个go文件中**声明**一个函数，这个函数没有函数体。

然后有另外一个汇编文件中，定义了这个函数就可以。	

这样这个汇编文件和go文件都会被编译到最终的binary文件中。

```shell
[root@localhost]~/code/src/main# ls
add.s  main.go
[root@localhost]~/code/src/main#

```

`add.s`文件如下，

```assembly
TEXT ·add(SB),$0
        MOVQ x+0(FP), BX
        MOVQ y+8(FP), BP
        ADDQ BP, BX
        MOVQ BX, ret+16(FP)
        RET
```

`main.go`如下，

```go
package main

import "fmt"

func add(x, y int64) int64

func main() {
        fmt.Println(add(2, 3))
}
```

直接编译，

```shell
[root@localhost]~/code/src/main# go build
[root@localhost]~/code/src/main# ls
add.s  main  main.go
[root@localhost]~/code/src/main# ./main
5
[root@localhost]~/code/src/main#

```

可以看到我们在`main.go`中**声明**的函数`add`没有函数体，函数体放在了`add.s`汇编文件中。（`add.s`需要和`main.go`放在同一个目录）。

## 获取golang 汇编代码的方法

`go build -gcflags="-S"  main.go`

## goloang汇编函数调用

go汇编使用的是`caller-save`模式，因此被调用函数的参数、返回值、栈位置都需要由调用者维护、准备。

## 寄存器

4个核心的伪寄存器（pseudo-register）。

1. FP。Frame pointer: arguments and locals.
2. PC。Program counter: jumps and branches.
3. SB。Static base pointer: global symbols.
4. SP。Stack pointer: top of stack.

这四个寄存器不是我们以前在看8086汇编时，CPU中真实存在的寄存器。他们由golang 的工具链维护。

对于不同的指令集架构，这四个 **伪寄存器**都是存在的，且含义一样。



### SB

先引用一段解释，

> The `SB` pseudo-register can be thought of as the origin of memory, so the symbol `foo(SB)` is the name `foo` as an address in memory. This form is used to name global functions and data. Adding `<>` to the name, as in `foo<>(SB)`, makes the name visible only in the current source file, like a top-level `static` declaration in a C file. Adding an offset to the name refers to that offset from the symbol's address, so `foo+4(SB)` is four bytes past the start of `foo`.

我们可以认为`SB`伪寄存器是 内存的起点。在golang 汇编中，使用`foo(SB)`表示符号`foo`的地址。

通过`SB`，我们可以在golang 汇编中引用全局的函数或者数据。

我们可以给`foo`添加一个offset，比如`foo+4(SB)`表示以`foo`地址为基准，偏移4个字节。

### SP

golang汇编中有一个伪寄存器，但是我们也知道同时存在一个真实的寄存器，名字也叫SP。

真寄存器`SP`对应的是栈的顶部（栈顶）。考虑到栈是从高地址向低地址生长的，相比于栈帧中的局部变量地址来说，真寄存器`SP`的值应该比较小。所以我们通过真寄存器`SP`去引用局部变量的时候，offset一般是正数。比如这样，

```
MOVUPS X0, 0x58(SP)
```

那伪寄存器SP呢？引用官方的一段解释，

> The `SP` pseudo-register is a virtual stack pointer used to refer to frame-local variables and the arguments being prepared for function calls. It points to the top of the local stack frame, so references should use negative offsets in the range [−framesize, 0): `x-8(SP)`, `y-4(SP)`, and so on.

### FP

`FP`始终是一个伪寄存器，不是硬件寄存器。即使在一个有`hardware frame pointer`的架构体系中，`FP`也表示伪寄存器。

我们知道，当代码执行到某个函数B中间时，有一个对应的栈帧。我们假设是函数A调用函数B，那么函数A调用函数B的参数我们怎么获得呢？

可以通过伪寄存器`FP`获得。我们可以使用`FP`来引用调用函数B的参数。

因此，`0(FP)`是调用函数B的第一个参数，`8(FP)`是调用函数B的第二个参数，以此类推。这个是当我们已经进入到函数B的栈帧时，才可以通过这种方法去引用。

当我们通过`FP`去引用函数参数的时候，有必要在前面加一个名称。比如`first_arg+0(FP)`，`second_arg+8(FP)`。

这里的offset 和前面谈到的`SB`伪寄存器的offset不一样。类似`foo+4(SB)`这样的表示方法，offset（此处为4）指的是相对于符号`foo`的偏移。