# go map 底层实现 #

golang中map的底层实现是一个散列表，因此实现map的过程实际上就是实现散列表的过程。

在这个散列表中，主要出现的结构体有两个，一个叫hmap(a header for a go map)，一个叫bucket。

## hmap ##

先来看看hmap的结构。
	
	type hmap struct {
		count     int // # live cells == size of map.  Must be first (used by len() builtin)
		flags     uint8
		B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
		noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
		hash0     uint32 // hash seed
	
		buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
		oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
		nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)
		overflow *[2]*[]*bmap
	}

其中`B`表示有2^B个buckets。

为了便于我们理解map的架构，我们关注`buckets`这个字段就可以了。

`buckets`指向一个内存连续分配的数组。

golang的map中用于存储的实际上是`bucket`数组。那么`bucket`的结构是怎样的呢？
## bucket结构 ##


	// A bucket for a Go map.
	type bmap struct {
		// tophash generally contains the top byte of the hash value
		// for each key in this bucket. If tophash[0] < minTopHash,
		// tophash[0] is a bucket evacuation state instead.
		tophash [bucketCnt]uint8
		// Followed by bucketCnt keys and then bucketCnt values.
		// NOTE: packing all the keys together and then all the values together makes the
		// code a bit more complicated than alternating key/value/key/value/... but it allows
		// us to eliminate padding which would be needed for, e.g., map[int64]int8.
		// Followed by an overflow pointer.
	}
	……
		// Maximum number of key/value pairs a bucket can hold.
	bucketCntBits = 3
	bucketCnt     = 1 << bucketCntBits
	……

`tophash`是个大小为8(bucketCnt)的数组，存储了8个key的hash值的高八位值。

在对key/value增删改查的时候，先比较key的hash值高八位是否相等，然后再比较具体的key值。

根据官方注释，在tophash数组之后跟着8个key/value对，每一对都对应tophash当中的一条记录。最后bucket中还包含指向链表下一个bucket的指针。bmap内存布局如下图：

	+------------------------+
	|       tophash[0]       |
	+------------------------+
	|       tophash[1]       |
	+------------------------+
	|          ...           |
	+------------------------+
	|       tophash[7]       |
	+------------------------+
	|          key1          |
	+------------------------+
	|          key2          |
	+------------------------+
	|          ...           |
	+------------------------+
	|          key8          |
	+------------------------+
	|         value1         |
	+------------------------+
	|         value2         |
	+------------------------+
	|          ...           |
	+------------------------+
	|         value8         |
	+------------------------+
	| pointer to next bucket |
	+------------------------+

之所以把所有key放一起而不是k1v1，k2v2，...这种，是因为key和value的数据类型内存大小可能差距很大，比如map[int64]int8，考虑到字节对齐，kv存在一起会浪费很多空间。

## map 索引过程 ##

	+-----+  hash function:f()   +------------+     +--------------------------+
	| key | -------------------> | hash value | --> | high 8 bits|low 2^B bits |
	+-----+                      +------------+     +--------------------------+
	


1. 首先使用哈希函数处理key值，得到key的哈希值。
2. 用key的hash值的低2^B（这里的B指的是hmap中的字段B）位去buckets数组找到对应的bucket，暂定为bucket[m]。
3. 利用hash值的高8位去依次比对tophash[0]~tophash[7]。如果找到对应的tophash[i]和key的hash值的高8位一样，然后再根据i和key占用字节的大小，计算出keyi的地址是多少，取出来和key值比对，一样的话，说明确实找到了。

## map 扩容##

当哈希表增长的时候，Go语言会将bucket数组的数量扩充一倍，产生一个新的bucket数组，并将旧数组的数据迁移至新数组。

判断扩充的条件，就是哈希表中的**加载因子**(即loadFactor)。

加载因子是一个阈值，一般表示为：散列包含的元素数 除以 位置总数。是一种“产生冲突机会”和“空间使用”的平衡与折中：加载因子越小，说明空间空置率高，空间使用率小，但是加载因子越大，说明空间利用率上去了，但是“产生冲突机会”高了。

每种哈希表都会有一个加载因子，数值超过加载因子就会为哈希表扩容。

Golang的map的加载因子的公式是：map长度 / 2^B, 阈值是6.5

		// Maximum average load of a bucket that triggers growth is 6.5.
		// Represent as loadFactorNum/loadFactDen, to allow integer math.
		loadFactorNum = 13
		loadFactorDen = 2


当Go的map长度增长到大于加载因子所需的map长度时，Go语言就会产生一个新的bucket数组，然后把旧的bucket数组移到一个属性字段`oldbuckets`中。注意：并不是立刻把旧的数组中的元素转移到新的bucket当中，而是，只有当访问到具体的某个bucket的时候，会把bucket中的数据转移到新的bucket中。


## 参考 ##
源码：runtime/hashmap.go

参考：[这里](https://blog.csdn.net/i6448038/article/details/82057424)