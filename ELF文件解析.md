# ELF文件解析

ELF文件有三种

1. 可重定向文件：文件保存着代码和适当的数据，用来和其他的目标文件一起来创建一个可执行文件或者是一个共享目标文件。
2. 共享目标文件：共享库。文件保存着代码和合适的数据。（linux下后缀为.so的文件）
3. 可执行文件：文件保存着一个用来执行的程序。（例如bash，gcc等）

## ELF视图

ELF文件格式提供了两种视图，分别是链接视图和执行视图。

链接视图以节（section）为单位，执行视图是段（segment）为单位。

链接视图就是在链接时用到的视图，而执行视图则是在执行时用到的视图。

上图左侧的视角是从链接来看的，右侧的视角是从执行来看的。

分为四个部分：

- ELF header：描述整个文件的组织。
- Program Header Table：描述文件中的各种segments，用来告诉系统如何创建进程映像的。
- sections 或者 segments：segments是从运行的角度来描述elf文件，sections是从链接的角度来描述elf文件，也就是说，在链接阶段，我们可以忽略program header table来处理此文件，在运行阶段可以忽略section header table来处理此程序。从图中我们也可以看出， segments与sections是包含的关系，一个segment包含若干个section。
- Section Header Table：包含了文件各个section的属性信息。