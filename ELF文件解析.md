# ELF文件解析

ELF文件有三种

1. 可重定向文件：文件保存着代码和适当的数据，用来和其他的目标文件一起来创建一个可执行文件或者是一个共享目标文件。
2. 共享目标文件：共享库。文件保存着代码和合适的数据。（linux下后缀为.so的文件）
3. 可执行文件：文件保存着一个用来执行的程序。（例如bash，gcc等）

## ELF视图

ELF文件格式提供了两种视图，分别是链接视图和执行视图。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/elf_view.png)

链接视图以节（section）为单位，执行视图是段（segment）为单位。

链接视图就是在链接时用到的视图，而执行视图则是在执行时用到的视图。

上图左侧的视角是从链接来看的，右侧的视角是从执行来看的。

分为四个部分：

- ELF header：描述整个文件的组织。
- Program Header Table：描述文件中的各种segments，用来告诉系统如何创建进程映像的。
- sections 或者 segments：segments是从运行的角度来描述elf文件，sections是从链接的角度来描述elf文件，也就是说，在链接阶段，我们可以忽略program header table来处理此文件，在运行阶段可以忽略section header table来处理此程序。从图中我们也可以看出， segments与sections是包含的关系，一个segment包含若干个section。
- Section Header Table：包含了文件各个section的属性信息。



## ELF Header

64位机器上的ELF文件，ELF Header大小为64字节。

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

