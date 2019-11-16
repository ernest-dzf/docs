# ELF文件解析

ELF文件有三种

1. 可重定向文件：文件保存着代码和适当的数据，用来和其他的目标文件一起来创建一个可执行文件或者一个共享目标文件。
2. 共享目标文件：共享库。文件保存着代码和合适的数据。（linux下后缀为.so的文件）
3. 可执行文件：文件保存着一个用来执行的程序。（例如bash，gcc等）

## ELF视图

ELF文件格式提供了两种视图，分别是链接视图和执行视图。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/elf_view.png)

链接视图以节（section）为单位，执行视图是段（segment）为单位。

链接视图就是在链接时用到的视图，而执行视图则是在执行时用到的视图。

上图左侧的视角是从链接来看的，右侧的视角是从执行来看的。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/elf_view2.png)

分为四个部分：

- ELF header：描述整个文件的组织。
- Program Header Table：描述文件中的各种segments，用来告诉系统如何创建进程映像的。
- sections 或者 segments：segments是从运行的角度来描述elf文件，sections是从链接的角度来描述elf文件，也就是说，在链接阶段，我们可以忽略program header table来处理此文件，在运行阶段可以忽略section header table来处理此程序。从图中我们也可以看出， segments与sections是包含的关系，一个segment包含若干个section。
- Section Header Table：包含了文件各个section的属性信息。

从上图中也可以看到，**一个segment其实就是包含多个section**。

## ELF Header

64位机器上的ELF文件，ELF Header大小为64字节。32位机器则为52字节。

```c
typedef struct
{
  unsigned char e_ident[EI_NIDENT]; /* Magic number and other info */
  Elf64_Half    e_type;         /* Object file type */
  Elf64_Half    e_machine;      /* Architecture */
  Elf64_Word    e_version;      /* Object file version */
  Elf64_Addr    e_entry;        /* Entry point virtual address */
  Elf64_Off e_phoff;        /* Program header table file offset */
  Elf64_Off e_shoff;        /* Section header table file offset */
  Elf64_Word    e_flags;        /* Processor-specific flags */
  Elf64_Half    e_ehsize;       /* ELF header size in bytes */
  Elf64_Half    e_phentsize;        /* Program header table entry size */
  Elf64_Half    e_phnum;        /* Program header table entry count */
  Elf64_Half    e_shentsize;        /* Section header table entry size */
  Elf64_Half    e_shnum;        /* Section header table entry count */
  Elf64_Half    e_shstrndx;     /* Section header string table index */
} Elf64_Ehdr;
```

可以使用`readelf`来读取ELF文件的相关信息。

```shell
[root@localhost]~/code/src/main# ls
main  main.go
[root@localhost]~/code/src/main# readelf -h main
ELF 头：
  Magic：  7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00		//魔法数字
  类别:                              ELF64
  数据:                              2 补码，小端序 (little endian)
  版本:                              1 (current)
  OS/ABI:                            UNIX - System V
  ABI 版本:                          0
  类型:                              EXEC (可执行文件)
  系统架构:                          Advanced Micro Devices X86-64
  版本:                              0x1
  入口点地址：              0x454dd0													//程序的入口地址
  程序头起点：              64 (bytes into file)							//segment表在文件64字节偏移处
  Start of section headers:          456 (bytes into file)
  标志：             0x0
  本头的大小：       64 (字节)																//本头部的大小，也就是ELF头部大小
  程序头大小：       56 (字节)													//segment 头项的大小
  Number of program headers:         7
  节头大小：         64 (字节)
  节头数量：         25
  字符串表索引节头： 3
[root@localhost]~/code/src/main#
```

可执行文件`main`是由`main.go`文件编译得到，文件格式为ELF文件。它的ELF头部大小为64字节。32位机器的话，ELF头部大小为52字节。

ELF头部起始字节为一个16字节的Magic数字，表示这个是ELF文件。我们使用`hexdump`也可以验证一下。

```shell
[root@localhost]~/code/src/main# hexdump main -n 32
0000000 457f 464c 0102 0001 0000 0000 0000 0000
0000010 0002 003e 0001 0000 4dd0 0045 0000 0000
0000020
[root@localhost]~/code/src/main#

```

## 段表（Section Head Table）

上面有谈到ELF Header的结构，其中一个字段`Elf64_Off e_shoff`表示段表的偏移量。

上面使用`readelf`读取的ELF头部信息中，就有段表的偏移值（`Start of section headers`）。

ELF文件中把指令和数据分成了很多段，比如`.text`，`.data`等等。ELF文件中所有这些段的元信息会存放在 **段表**中。ELF Header中的`e_shoff`字段指示这个段表在文件中的哪里。

这样，我们得到了文件头（ELF Header），从文件头中得到段表（Section Head Table）位置，最后从段表中可以找到对应段的数据。

我们可以使用`readelf -S `来读取某个ELF文件的段表信息。

```
[root@localhost]~/code/src/main# readelf -S main
共有 25 个节头，从偏移量 0x1c8 开始：

节头：
  [号] 名称               类型              地址              偏移量
       大小               全体大小          旗标     链接   信息   对齐
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000401000  00001000
       000000000008c265  0000000000000000  AX       0     0     16
  [ 2] .rodata           PROGBITS         000000000048e000  0008e000
       000000000004f617  0000000000000000   A       0     0     32
  [ 3] .shstrtab         STRTAB           0000000000000000  000dd620
       00000000000001a1  0000000000000000           0     0     1
  [ 4] .typelink         PROGBITS         00000000004dd7e0  000dd7e0
       0000000000000c68  0000000000000000   A       0     0     32
  [ 5] .itablink         PROGBITS         00000000004de448  000de448
       0000000000000050  0000000000000000   A       0     0     8
  [ 6] .gosymtab         PROGBITS         00000000004de498  000de498
       0000000000000000  0000000000000000   A       0     0     1
  [ 7] .gopclntab        PROGBITS         00000000004de4a0  000de4a0
       000000000006b6ab  0000000000000000   A       0     0     32
  [ 8] .go.buildinfo     PROGBITS         000000000054a000  0014a000
       0000000000000020  0000000000000000  WA       0     0     16
  [ 9] .noptrdata        PROGBITS         000000000054a020  0014a020
       000000000000d0d8  0000000000000000  WA       0     0     32
  [10] .data             PROGBITS         0000000000557100  00157100
       0000000000007050  0000000000000000  WA       0     0     32
  [11] .bss              NOBITS           000000000055e160  0015e160 
       000000000001b870  0000000000000000  WA       0     0     32
  [12] .noptrbss         NOBITS           00000000005799e0  001799e0
       0000000000002768  0000000000000000  WA       0     0     32
  [13] .zdebug_abbrev    PROGBITS         000000000057d000  0015f000
       0000000000000119  0000000000000000           0     0     8
  [14] .zdebug_line      PROGBITS         000000000057d119  0015f119
       00000000000160d4  0000000000000000           0     0     8
  [15] .zdebug_frame     PROGBITS         00000000005931ed  001751ed
       000000000000609f  0000000000000000           0     0     8
  [16] .zdebug_pubnames  PROGBITS         000000000059928c  0017b28c
       00000000000013f3  0000000000000000           0     0     8
  [17] .zdebug_pubtypes  PROGBITS         000000000059a67f  0017c67f
       00000000000031c1  0000000000000000           0     0     8
  [18] .debug_gdb_script PROGBITS         000000000059d840  0017f840
       000000000000002a  0000000000000000           0     0     1
  [19] .zdebug_info      PROGBITS         000000000059d86a  0017f86a
       000000000002e9c7  0000000000000000           0     0     8
  [20] .zdebug_loc       PROGBITS         00000000005cc231  001ae231
       0000000000015295  0000000000000000           0     0     8
  [21] .zdebug_ranges    PROGBITS         00000000005e14c6  001c34c6
       00000000000079ad  0000000000000000           0     0     8
  [22] .note.go.buildid  NOTE             0000000000400f9c  00000f9c
       0000000000000064  0000000000000000   A       0     0     4
  [23] .symtab           SYMTAB           0000000000000000  001cb000
       000000000000fc48  0000000000000018          24   118     8
  [24] .strtab           STRTAB           0000000000000000  001dac48
       0000000000010b17  0000000000000000           0     0     1
  Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)
```



从上面例子可以看到，段表是从0x1c8偏移量开始。对比前面使用`readelf -h main`读取的信息所展示的`Start of section headers:          456 (bytes into file)`，是一致的（0x1c8=456）。

段表其实就是一个数组，上面例子中每个section前面的数字（`[0],[1],[2],...`）就是这个段在段表中的下标。可以看到第0个是没有被使用的。



我们说段表其实就是一个数组，那我们就必须关注它的每一项的具体含义，每一项记录了一个段的信息。

段表里面每一项的定义如下：

```c
typedef struct {
        Elf32_Word      sh_name;
        Elf32_Word      sh_type;
        Elf32_Word      sh_flags;
        Elf32_Addr      sh_addr;
        Elf32_Off       sh_offset;
        Elf32_Word      sh_size;
        Elf32_Word      sh_link;
        Elf32_Word      sh_info;
        Elf32_Word      sh_addralign;
        Elf32_Word      sh_entsize;
} Elf32_Shdr;

typedef struct {
        Elf64_Word      sh_name;
        Elf64_Word      sh_type;
        Elf64_Xword     sh_flags;
        Elf64_Addr      sh_addr;
        Elf64_Off       sh_offset;
        Elf64_Xword     sh_size;
        Elf64_Word      sh_link;
        Elf64_Word      sh_info;
        Elf64_Xword     sh_addralign;
        Elf64_Xword     sh_entsize;
} Elf64_Shdr;
```

具体每个字段含义参考[这里](https://docs.oracle.com/cd/E19683-01/817-3677/chapter6-94076/index.html)。

## 程序头（Program Header）

在ELF中，把**权限相同、又连在一起的段（section）**叫做segment。操作系统是按照segment来映射可执行文件的。

描述这些segment的结构叫做**程序头**，它描述了ELF文件如何被映射到内存空间中。

我们可以使用`readelf -l`来读取程序头的类容。

```
[root@localhost]~/code/src/main# readelf -l main

Elf 文件类型为 EXEC (可执行文件)
入口点 0x454dd0						//这个入口点表示程序的入口地址
共有 7 个程序头，开始于偏移量64

程序头：
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000400040 0x0000000000400040
                 0x0000000000000188 0x0000000000000188  R      1000
  NOTE           0x0000000000000f9c 0x0000000000400f9c 0x0000000000400f9c
                 0x0000000000000064 0x0000000000000064  R      4
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x000000000008d265 0x000000000008d265  R E    1000
  LOAD           0x000000000008e000 0x000000000048e000 0x000000000048e000
                 0x00000000000bbb4b 0x00000000000bbb4b  R      1000
  LOAD           0x000000000014a000 0x000000000054a000 0x000000000054a000
                 0x0000000000014160 0x0000000000032148  RW     1000
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     8
  LOOS+5041580   0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000         8
  
  
 Section to Segment mapping:
  段节...
   00
   01     .note.go.buildid
   02     .text .note.go.buildid
   03     .rodata .typelink .itablink .gosymtab .gopclntab
   04     .go.buildinfo .noptrdata .data .bss .noptrbss
   05
   06
```

我们以程序头（Program Header）中第0项为例。

`Type`类型为`PHDR`，偏移量为`0x0000000000000040`，也就是程序头（Program Header）的偏移地址（64字节）。这个段（segment）的大小为`0x0000000000000188`字节（第0段其实就是Program Header）。

我们看到Program Header大小其实为0x188字节（392），总共有7个segment，那么Program Header是不是应该有7个item呢？每个大小为392/7=56字节。

查看内核中对于程序头表项的定义（`elf64_phdr`），确实如此，出处参考[这里](https://code.woboq.org/linux/linux/include/uapi/linux/elf.h.html#Elf64_Xword)。

```c
typedef __u32	Elf64_Word;
typedef __u64	Elf64_Off;
typedef __u64	Elf64_Addr;
typedef __u64	Elf64_Xword;
typedef struct elf64_phdr {
  Elf64_Word p_type;
  Elf64_Word p_flags;
  Elf64_Off p_offset;		/* Segment file offset */
  Elf64_Addr p_vaddr;		/* Segment virtual address */
  Elf64_Addr p_paddr;		/* Segment physical address */
  Elf64_Xword p_filesz;		/* Segment size in file */
  Elf64_Xword p_memsz;		/* Segment size in memory */
  Elf64_Xword p_align;		/* Segment alignment, file & memory */
} Elf64_Phdr;
```

