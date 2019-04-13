# go map 底层实现 #

golang中map的底层实现是一个散列表，因此实现map的过程实际上就是实现散表的过程。

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


参考：[这里](https://blog.csdn.net/i6448038/article/details/82057424)