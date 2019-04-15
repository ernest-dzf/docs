# golang slice实现 #
## slice结构##

先来看下slice的结构。

	type slice struct {
		array unsafe.Pointer
		len   int
		cap   int
	}
	
	// An notInHeapSlice is a slice backed by go:notinheap memory.
	type notInHeapSlice struct {
		array *notInHeap
		len   int
		cap   int
	}

slice实际就是一个struct，在runtime/slice.go中定义。

由定义可以看出slice底层是基于数组，本质是对数组的封装。由三部分组成：

1. 指针。指向第一个slice元素对应的底层数组元素地址。
2. 长度。slice中元素的数目。
3. 容量。slice开始位置到底层数据的结尾。

内置函数len和cap，分别返回slice的长度和容量。

slice使用下标不能超过len，向后扩展不能超过cap。

多个不同slice之间可以共享底层的数据，起始地址、长度都可以不同，所以slice第一个元素未必是数组的第一个元素。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/slice.png)