---
layout: single
title: "[OS]Linux 的进程内存分配"
categories: [OS]
last_modified_at: 2024-05-10
excerpt: " just a record."
header:
  teaser: https://static.runoob.com/images/mix/life-code-typography-hd-wallpaper-1920x1080-7168.jpg
---


# Linux 的进程内存分配

## 地址空间

进程地址空间从低地址开始依次是代码段(Text)、数据段(Data)、BSS段、堆、内存映射段、栈。


---

###  代码段(text)

代码段也称文本段，通常用于存放程序执行代码(即CPU执行的机器指令)。一般C语言执行语句都编译成机器代码保存在代码段。通常代码段是可共享的，因此频繁执行的程序只需要在内存中拥有一份拷贝即可。

代码段通常属于只读，以防止其他程序意外地修改其指令(对该段的写操作将导致段错误)。某些架构也允许代码段为可写，即允许修改程序。

代码段指令根据程序设计流程依次执行，对于顺序指令，只会执行一次(每个进程)；若有反复，则需使用跳转指令；若进行递归，则需要借助栈来实现。

代码段指令中包括操作码和操作对象(或对象地址引用)。若操作对象是立即数(具体数值)，将直接包含在代码中；若是局部数据，将在栈区分配空间，然后引用该数据地址；若位于BSS段和数据段，同样引用该数据地址。


###  数据段(Data)

数据段通常用于存放程序中已初始化且初值不为0的全局变量和静态局部变量。数据段属于静态内存分配(静态存储区)，可读可写。

运行时数据段和BSS段的整个区段通常称为数据区。

### BSS段

BSS(Block Started by Symbol)段中通常存放程序中以下符号：

- 未初始化的全局变量和静态局部变量
- 初始值为0的全局变量和静态局部变量(依赖于编译器实现)
- 未定义且初值不为0的符号(该初值即common block的大小)

     C语言中，未显式初始化的静态分配变量被初始化为0(算术类型)或空指针(指针类型)。由于程序加载时，BSS会被操作系统清零，所以未赋初值或初值为0的全局变量都在BSS中。BSS段仅为未初始化的静态分配变量预留位置，在目标文件中并不占据空间，这样可减少目标文件体积。

###  堆(heap)

     
堆用于存放进程运行时动态分配的内存段，可动态扩张或缩减。堆中内容是匿名的，不能按名字直接访问，只能通过指针间接访问。当进程调用malloc(C)/new(C++)等函数分配内存时，新分配的内存动态添加到堆上(扩张)；当调用free(C)/delete(C++)等函数释放内存时，被释放的内存从堆中剔除(缩减) 。

分配的堆内存是经过字节对齐的空间，以适合原子操作。堆管理器通过链表管理每个申请的内存，由于堆申请和释放是无序的，最终会产生内存碎片。堆内存一般由应用程序分配释放，回收的内存可供重新使用。若程序员不释放，程序结束时操作系统可能会自动回收

堆的末端由break指针标识，当堆管理器需要更多内存时，可通过系统调用brk()和sbrk()来移动break指针以扩张堆，一般由系统自动调用



### 内存映射段(mmap)


此处，内核将硬盘文件的内容直接映射到内存, 任何应用程序都可通过Linux的mmap()系统调用请求这种映射。内存映射是一种方便高效的文件I/O方式， 因而被用于装载动态共享库。用户也可创建匿名内存映射，该映射没有对应的文件, 可用于存放程序数据。在 Linux中，若通过malloc()请求一大块内存，C运行库将创建一个匿名内存映射，而不使用堆内存。”大块” 意味着比阈值 MMAP_THRESHOLD还大，缺省为128KB，可通过mallopt()调整。

ernel 2.6的32位Linux系统中，malloc申请的最大内存理论值在3GB左右。

### 栈(stack)

栈又称堆栈，由编译器自动分配释放，行为类似数据结构中的栈(先进后出)。堆栈主要有三个用途：

- 为函数内部声明的非静态局部变量(C语言中称“自动变量”)提供存储空间。
- 记录函数调用过程相关的维护性信息，称为栈帧(Stack Frame)或过程活动记录(Procedure Activation Record)。它包括函数返回地址，不适合装入寄存器的函数参数及一些寄存器值的保存。除递归调用外，堆栈并非必需。因为编译时可获知局部变量，参数和返回地址所需空间，并将其分配于BSS段。
- 临时存储区，用于暂存长算术表达式部分计算结果或alloca()函数分配的栈内内存。

---

## /proc/$PID/maps 数据字段含义


![[Pasted image 20240511202725.png]]


- 第一列：7f3d96540000-7f3d96542000 本段内存映射的虚拟地址空间范围，对应vm_area_struct中的vm_start和vm_end。  
  
- 第二列： rw-p 此段虚拟地址空间的属性。每种属性用一个字段表示，r表示可读，w表示可写，x表示可执行，p和s共用一个字段，互斥关系，p表示私有段，s表示共享段，如果没有相应权限，则用’-’代替。  
  
- 第三列：00000000  针对有名映射，指本段映射地址在文件中的偏移，对应vm_pgoff。对匿名映射而言，为vm_area_struct->vm_start。  
  
- 第四列：fd（00）: 03  所映射的文件所属设备的设备号，对应vm_file->f_dentry->d_inode->i_sb->s_dev。匿名映射为0。其中fd为主设备号，00为次设备号。  
  
- 第五列： 27582543    文件的索引节点号，对应vm_file->f_dentry->d_inode->i_ino，与ls –i显示的内容相符。匿名映射为0。  
  
- 第六列： /usr/lib/x86_64-linux-gnu/libc.so.6   所映射的文件名。对有名映射而言，是映射的文件名，对匿名映射来说，是此段内存在进程中的作用。[stack]表示本段内存作为栈来使用，[heap]作为堆来使用。

```shell

-- EXP4省略第六列：
55c4ef8ae000-55c4ef8af000 r--p 00000000 08:03 14052137   
-- 该r--p条目描述一个只能读取的内存块（r权限标志）。是静态数据（常量）
55c4ef8af000-55c4ef8b0000 r-xp 00001000 08:03 14052137   
-- 该r-xp条目描述一块可执行内存（x权限标志）。这就是代码。 “main的地址(代码段):0x55c4ef8af4df ”               
55c4ef8b0000-55c4ef8b1000 r--p 00002000 08:03 14052137                   
55c4ef8b1000-55c4ef8b2000 r--p 00002000 08:03 14052137                   
55c4ef8b2000-55c4ef8b3000 rw-p 00003000 08:03 14052137  
-- 该rw-p条目描述了一个可写的内存块（w权限标志）。这是用于库的全局变量
55c4f032b000-55c4f034c000 rw-p 00000000 00:00 0     [heap]

-- ENTER 后多一行：
7f3d962c0000-7f3d962c3000 rw-p 00000000 00:00 0 -- 新建堆区

```

之后的回车会触发文件映射：

```shell

--- 省略部分内容
7f6db43f6000-7f6db4401000 r--p 0002c000 08:03 27582543                   
7f6db4401000-7f6db4402000 rw-s 00000000 08:03 14052221  /cmake.../map-file.txt
--- file——map函数实现，新增该txt文件并且映射内容
7f6db4402000-7f6db4404000 r--p 00037000 08:03 27582543   

--- 再回车后该映射因为文件关闭而消失。

```
