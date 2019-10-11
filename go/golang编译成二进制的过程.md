# golang编译成二进制的过程

golang和c都是系统级编程语言。

我们可以对比一下两者编译成二进制的过程。

## C语言二进制获取过程

先来看下c语言编译的各个过程。

预处理（Preprocessing）---> 编译（Compilation）---> 汇编（Assembly）---> 链接（Linking）。

### 预处理

预处理主要处理那些源代码文件中的"#"开头的预编译指令，比如"#include"，“#define”。

### 编译

编译过程就是把预处理完的文件进行一系列词法分析、语法分析、语义分析及优化后生产相应的汇编代码文件。

目前的GCC一般把预处理和编译两个步骤合并成一个步骤，使用一个叫做cc1的程序来完成。

```shell
$/usr/libexec/gcc/x86_64-redhat-linux/4.8.2/cc1 hello.c
```

或者这样，

```shell
$gcc -S hello.c -o hello.s
```

也可以得到汇编输出文件。

对于C语言，这个预编译和编译的程序是cc1，对于c++来说，对应的程序叫做cc1plus；Objective-C是cc1obj；Java是jc1。

实际上gcc这个命令只是这些后台程序的包装，它会根据不同的参数要求去调用预编译-编译程序、汇编器、链接器。

### 汇编

汇编器就是将汇编代码转换为机器可以执行的指令。

```shell
$as hello.s -o hello.o	
```



### 链接



### 图示

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/bianyi.png)

## 参考

1. http://jizhao.blog.chinaunix.net/uid-31387290-id-5753861.html
2. 

