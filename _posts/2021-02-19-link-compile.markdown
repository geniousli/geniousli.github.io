---
title: link compile
date: 2021-02-19
tags: linker compile gcc
---

## link & compile

### 为什么c语言中的 header file *.h 一定需要 防止重复被include 呢？ 而为什么一定需要 header file的存在呢？ 
  * header file 需要存在原因有： 1） .c 文件 变为可执行文件，需要 编译、链接 两个阶段， 而链接中 是可以将 不同的 类型编译过的文件进行 链接的。 随着程序的复杂， 比如 函数参数的 传递顺序等 原因导致，需要 程序调用者 知道 函数调用的结构， 所以 [[https://www.zhihu.com/question/280665935][链接]] 2) 如果依赖于系统中的系统库，比如标准库中的系统调用，则只能使用inluce header file 来进行使用，即：被调用方 是已经编译好的文件格式
  * 为什么 header file一定需要 预防被重复加载呢？： 经过测试发现，在 header file 中不存在 struct 定义的时候，只是简单的 func 定义的时候， 重复包含并没有问题， 然而在 定义 struct的时候 则出现了 重复定义问题。 而问题也只是出现了 gcc -c 的阶段， 在gcc -E gcc -S 阶段 依然没有任何问题，所以 关于 为什么一定需要 #ifndef #define code ...  #endif 的格式，则需要 在gcc 中寻找


### C 语言 到可执行文件的过程

* 预处理(Preprocessing)
* 编译(Compilation)
* 汇编(Assembly)
* 链接(Linking): 链接过程主要包括了： 
  * 地址和空间分配
  * 符号决议
  * 重定位： 当存在a.c 依赖 b.c 的函数时候， 文件是单独编译的，并非 将b.c 的中内容 插入到 a.c 中，然后进行整体编译。所以导致 a.c 中并不知道 b.c 中函数 变量的存在， 所以 在编译 a.c时候， 函数调用 都会 call 0， 函数地址 需要 在链接时候 进行确定，并修改 （即是重定位）

### elf 文件的类型： 

| type                              | 说明                                                                                                           | 实例                          |
|-----------------------------------+----------------------------------------------------------------------------------------------------------------+-------------------------------|
| 可重定位文件(Relocatable File)    | 可以被可执行文件或 共享目标文件 链接                                                                           | Linux: .o                     |
| 可执行文件(Executable File)       | 可执行程序, 一般没有 扩展名                                                                                    | Linux: /bin/bash Windows .exe |
| 共享目标文件(Shared Object File ) | 1） 可被 其他 可重定位文件 、共享文件 连接成 目标文件 2） 动态链接器 将其与可执行文件结合，映射为 进程的一部分 | Linux: .so, Windows: DLL      |
| 核心转储文件(Core Dump File)      | 进程意外终止时候， 系统可以 将进程的地址空间内容 等信息 转储到  该文件中                                       | Linux: core dump               |


```shell
vagrant@precise64:/vagrant_data/link_test$ file a.out
a.out: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0x91de252bc2c3703aa5c324e5176b05e6b36a5bfa, not stripped

vagrant@precise64:/vagrant_data/link_test$ file main.o
main.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
```

### 深入 .o 文件
* 可用工具有： 1) objdump, 2) readelf, 3) nm
* elf 因为 存在段(section) ，而段 是变长的，导致 没有固定的映射方法，所以 Header 为固定长度 异常重要， Header 中存在 多少个段（Number of section headers），  段表偏移(Start of section headers:) 的关键信息， 来使 c代码能够将 段表（section table） 映射到代码中
* objdump -h main.o

```shell

int main() {
  int a = 10;
  name(a);
}

vagrant@precise64:/vagrant_data/link_test$ objdump -h main.o

main.o:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         00000020  0000000000000000  0000000000000000  00000040  2**2
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE
  1 .data         00000000  0000000000000000  0000000000000000  00000060  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  2 .bss          00000000  0000000000000000  0000000000000000  00000060  2**2
                  ALLOC
  3 .comment      0000002b  0000000000000000  0000000000000000  00000060  2**0
                  CONTENTS, READONLY
  4 .note.GNU-stack 00000000  0000000000000000  0000000000000000  0000008b  2**0
                  CONTENTS, READONLY
  5 .eh_frame     00000038  0000000000000000  0000000000000000  00000090  2**3
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, DATA
vagrant@precise64:/vagrant_data/link_test$ readelf -h main.o
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          296 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
  Number of section headers:         12
  Section header string table index: 9

//  注意 在 文件中添加 了 全局变量之后 .data 文件 的size 变大了。
int b = 10;
int main() {
  int a = 10;
  name(a);
}


vagrant@precise64:/vagrant_data/link_test$ objdump -h main.o

main.o:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         00000020  0000000000000000  0000000000000000  00000040  2**2
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE
  1 .data         00000004  0000000000000000  0000000000000000  00000060  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  2 .bss          00000000  0000000000000000  0000000000000000  00000064  2**2
                  ALLOC
  3 .comment      0000002b  0000000000000000  0000000000000000  00000064  2**0
                  CONTENTS, READONLY
  4 .note.GNU-stack 00000000  0000000000000000  0000000000000000  0000008f  2**0
                  CONTENTS, READONLY
  5 .eh_frame     00000038  0000000000000000  0000000000000000  00000090  2**3
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, DATA

vagrant@precise64:/vagrant_data/link_test$ size main.o
   text	   data	    bss	    dec	    hex	filename
     88	      4	      0	     92	     5c	main.o

```

#### objdump -s -d main.o 其中  -s  将各个段 都打印出来， -d 则将 代码段反汇编

```shell
vagrant@precise64:/vagrant_data/link_test$ objdump -s -d main.o

main.o:     file format elf64-x86-64

Contents of section .text:
 0000 554889e5 4883ec10 c745fc0a 0000008b  UH..H....E......
 0010 45fc89c7 b8000000 00e80000 0000c9c3  E...............
Contents of section .data:
 0000 0a000000                             ....
Contents of section .comment:
 0000 00474343 3a202855 62756e74 752f4c69  .GCC: (Ubuntu/Li
 0010 6e61726f 20342e36 2e332d31 7562756e  naro 4.6.3-1ubun
 0020 74753529 20342e36 2e3300             tu5) 4.6.3.
Contents of section .eh_frame:
 0000 14000000 00000000 017a5200 01781001  .........zR..x..
 0010 1b0c0708 90010000 1c000000 1c000000  ................
 0020 00000000 20000000 00410e10 8602430d  .... ....A....C.
 0030 065b0c07 08000000                    .[......

Disassembly of section .text:

0000000000000000 <main>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	48 83 ec 10          	sub    $0x10,%rsp
   8:	c7 45 fc 0a 00 00 00 	movl   $0xa,-0x4(%rbp)
   f:	8b 45 fc             	mov    -0x4(%rbp),%eax
  12:	89 c7                	mov    %eax,%edi
  14:	b8 00 00 00 00       	mov    $0x0,%eax
  19:	e8 00 00 00 00       	callq  1e <main+0x1e>
  1e:	c9                   	leaveq
  1f:	c3                   	retq
```


* 重定位表：

```shell
  vagrant@precise64:/vagrant_data/link_test$ readelf -S main.o
There are 12 section headers, starting at offset 0x128:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000000000  00000040
       0000000000000020  0000000000000000  AX       0     0     4
  [ 2] .rela.text        RELA             0000000000000000  00000548
       0000000000000018  0000000000000018          10     1     8
  [ 3] .data             PROGBITS         0000000000000000  00000060
       0000000000000004  0000000000000000  WA       0     0     4
  [ 4] .bss              NOBITS           0000000000000000  00000064
       0000000000000000  0000000000000000  WA       0     0     4
  [ 5] .comment          PROGBITS         0000000000000000  00000064
       000000000000002b  0000000000000001  MS       0     0     1
  [ 6] .note.GNU-stack   PROGBITS         0000000000000000  0000008f
       0000000000000000  0000000000000000           0     0     1
  [ 7] .eh_frame         PROGBITS         0000000000000000  00000090
       0000000000000038  0000000000000000   A       0     0     8
  [ 8] .rela.eh_frame    RELA             0000000000000000  00000560
       0000000000000018  0000000000000018          10     7     8
  [ 9] .shstrtab         STRTAB           0000000000000000  000000c8
       0000000000000059  0000000000000000           0     0     1
  [10] .symtab           SYMTAB           0000000000000000  00000428
       0000000000000108  0000000000000018          11     8     8
  [11] .strtab           STRTAB           0000000000000000  00000530
       0000000000000014  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), l (large)
  I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)
```

* 在列表中，有 type 为RELA  或者 REL  的 rela.text, 为重定位表， main.c 中 调用了一个name 函数， name 存在于name.c 文件中， 链接器在 连接时候，需要 将main.o 中的函数调用 处理为 name.o 中的地址， 其中info 为指向需要重定位的 section index
* 每个需要重定位的段，都需要 一个相应的重定位表， 比如 .text 则为 .rela.text, .eh_frmae .rela.eh_frame, 
* 字符串表：因为字符串为变长，比较难映射 所以单独存放一个 段。 .strtab .shstrtab, 类型为 strtab (String table )， .strtab  为 字符串表 用来存储 普通的字符串， 比如符号的名字， .shstrtab 则为 （section header string table） 段表字符串表， 用来保存段表 中 用到的字符串 比如段名
* 对比 header 中的 Section header string table index: 9 项， 正好 为 .shstrtab 在section table中的下标
### 符号
* 符号是， 链接过程中重要的信息数据， 链接过程中  目标文件的相互拼接： 实际上 就是目标文件之间对地址的引用， 即对函数 变量地址的引用。比如 B 需要用到 A中的函数foo， 则是： A中定义了 函数foo， B 引用了 A中的函数foo, 概念同样适用变量，
* 链接中，我们称  函数和变量为符号，函数名 变量名则是 符号名。
* 每个文件中 都包含 函数表， 记录了 文件中使用的所有符号，每个定义的符号都有对应的条目，其中包含符号值， 对于变量和函数来说，符号值 则为 其地址。
* 符号 可能包含的有 以下类型： 
 1. 定义在本文件中的全局符号，函数 等，可以被其他文件 引用。 比如: 函数 main, name,
 2. 外部符号： 再本文件中引用的全局符号，并未在本文件中定义
 3. 段名： 其数值为 该段的其实地址， 编译器所加。
 4. 局部符号： 只在该文件中可见，比如static 变量
 5. 行号信息： 目标文件指令 与 源代码 的对应关系. debug 模式下 会创建 不少的 该类信息 在 符号表中
 
```shell
vagrant@precise64:/vagrant_data/link_test$ readelf -s main.o

Symbol table '.symtab' contains 11 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    3
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    6
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    7
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    5
     8: 0000000000000000     4 OBJECT  GLOBAL DEFAULT    3 b
     9: 0000000000000000    32 FUNC    GLOBAL DEFAULT    1 main
    10: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND name  


vagrant@precise64:/vagrant_data/link_test$ objdump -t main.o

main.o:     file format elf64-x86-64

SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000 main.c
0000000000000000 l    d  .text	0000000000000000 .text
0000000000000000 l    d  .data	0000000000000000 .data
0000000000000000 l    d  .bss	0000000000000000 .bss
0000000000000000 l    d  .note.GNU-stack	0000000000000000 .note.GNU-stack
0000000000000000 l    d  .eh_frame	0000000000000000 .eh_frame
0000000000000000 l    d  .comment	0000000000000000 .comment
0000000000000000 g     O .data	0000000000000004 b
0000000000000000 g     F .text	0000000000000020 main
0000000000000000         *UND*	0000000000000000 name
#+end_src

#### 相似段合并：

```shell
vagrant@precise64:/vagrant_data/link_test$ objdump -h main.o

main.o:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         00000020  0000000000000000  0000000000000000  00000040  2**2
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE
  1 .data         00000004  0000000000000000  0000000000000000  00000060  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  2 .bss          00000000  0000000000000000  0000000000000000  00000064  2**2
                  ALLOC
  3 .comment      0000002b  0000000000000000  0000000000000000  00000064  2**0
                  CONTENTS, READONLY
  4 .note.GNU-stack 00000000  0000000000000000  0000000000000000  0000008f  2**0
                  CONTENTS, READONLY
  5 .eh_frame     00000038  0000000000000000  0000000000000000  00000090  2**3
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, DATA
vagrant@precise64:/vagrant_data/link_test$ objdump -h name.o

name.o:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         0000000f  0000000000000000  0000000000000000  00000040  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .data         00000000  0000000000000000  0000000000000000  00000050  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  2 .bss          00000000  0000000000000000  0000000000000000  00000050  2**2
                  ALLOC
  3 .comment      0000002b  0000000000000000  0000000000000000  00000050  2**0
                  CONTENTS, READONLY
  4 .note.GNU-stack 00000000  0000000000000000  0000000000000000  0000007b  2**0
                  CONTENTS, READONLY
  5 .eh_frame     00000038  0000000000000000  0000000000000000  00000080  2**3
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, DATA

ld main.o  name.o -e main -o ab                  
vagrant@precise64:/vagrant_data/link_test$ objdump -h ab

ab:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         0000002f  00000000004000e8  00000000004000e8  000000e8  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .eh_frame     00000058  0000000000400118  0000000000400118  00000118  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .data         00000004  0000000000600170  0000000000600170  00000170  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  3 .comment      0000002a  0000000000000000  0000000000000000  00000174  2**0
                  CONTENTS, READONLY

```

* ld main.o  name.o -e main -o ab
*  注意 VMA LMA:  VMA (Virtual Memory Address) 为 虚拟地址， LMA( Load Memory Address) 即 加载地址， 一般情况下两个数值 都是一样的，有些切入式系统中 两个数值不同。
* 链接器为 目标文件 分配 地址和空间 的理解： 其中地址和空间 含义有两个： 1）输出的可执行文件的空间， 2） 装载后的虚拟地址中的虚拟地址空间， 对于 .text .data 来说， 他们在 文件中 和 虚拟地址 中都有分配空间， .bss 则 仅在 虚拟地址空间中存在。其中 虚拟地址 关系到 链接器对于 地址的重新计算
#### 链接 步骤: 两部链接

* 1. 空间和地址分配： 
  * 扫描所有的输入文件，获得各个段的长度、属性、位置。
  * 将输入文件中的符号 定义 和符号引用 收集起来，统一为一个 全局符号表
  * 合并所有的输入文件的相似段，计算各个合并后的段的长度和位置，并建立映射关系
* 2. 符号解析 与重定位： 链接过程的核心 过程
  * 读取输入文件中的段的数据、重定位信息、进行符号解析与重定位 调整代码中的地址等 

#### 重定位表： 用来提供 给链接器 关于重定位 的相关信息， 比如那些需要被调整，怎样调整？ 重定位表 即用来专门提供这些信息
* 每个 需要 包含需要被重定位符号的 段，都有一个对应的 重定位表。 比如 .text 则赌赢 .rel.text
* 重定位表结构：

```shell
   vagrant@precise64:/vagrant_data/link_test$ objdump -r main.o

main.o:     file format elf64-x86-64

RELOCATION RECORDS FOR [.text]:
OFFSET           TYPE              VALUE
000000000000001a R_X86_64_PC32     name-0x0000000000000004


RELOCATION RECORDS FOR [.eh_frame]:
OFFSET           TYPE              VALUE
0000000000000020 R_X86_64_PC32     .text



vagrant@precise64:/vagrant_data/link_test$ objdump -d main.o

main.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <main>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	48 83 ec 10          	sub    $0x10,%rsp
   8:	c7 45 fc 0a 00 00 00 	movl   $0xa,-0x4(%rbp)
   f:	8b 45 fc             	mov    -0x4(%rbp),%eax
  12:	89 c7                	mov    %eax,%edi
  14:	b8 00 00 00 00       	mov    $0x0,%eax
  19:	e8 00 00 00 00       	callq  1e <main+0x1e>
  1e:	c9                   	leaveq
  1f:	c3                   	retq

因为  main中 存在对 外部函数(name.c 中name ) 的调用，所以 call name 命令 的地址 需要被 ld 调整修改，即是 .rel.text 段中需要记录的信息. 在  重定位表 中， offset 代表 改需要被重定位的符号，在段中的偏移，
main.o 中的 rel.text name中的offset 为1a， 即是 main.o 中的代码段 callq 的地址部分。
```
  
##### common  块
* 符号抉择： 如果链接的输入文件中， 存在多个相同的符号类型，该如何选择？  因为链接器本身并不支持符号类型， 变量，函数定义对于编译器 是透明的。
* 符号类型 不一致的情况 主要由如下三点：
  * 1. 两个或者两个以上的 强符号类型不一致
  * 2. 有一个强符号，其他都是 弱符号， 出现类型不一致
  * 3. 两个或两个以上 弱符号类型不一致
  * 对于1  无额外处理，即： 链接器直接抛出错误。 2， 3 情况 则需要 common块（common block）机制 来进行处理， common块 很简单， 当不同的 目标文件需要的common块 空间大小不一致时， 以最大的那块为准
  * 弱符号： 即是非强符号， 强符号有： 函数定义， 全局变量， 初始化的内部变量，

* ABI： ld 能不能 链接 两个不同编译器 编译出来的目标文件呢？ 能够 满足的条件的目标文件 需要满足如下条件：
  * 采用相同的文件格式
  * 同样的符号修饰标准
  * 变量的内存分布方式相同(大端、小端)
  * 函数的调用方式相同 （参数push stack的顺序， 是否 使用 寄存器代替stack ）
  *  etc...
  * 规范这些的即是 二进制兼容性 ABI  (Application Binary Interface)

#### 静态链接: 一种语言的开发环境 往往带有语言库，这些苦 就是对操作系统的api 封装，其实一个静态库可以简单的堪称一组目标文件的集合，很多目标文件经过压缩打包后 形成的一个文件， 比如linux 下 /usr/lib/libc.a 
* glic 本身是c语言开发的， 他有上千个 C语言源代码文件组成， 编译完成后 有相同数量的 目标文件（上千个） 比如 printf.o scanf.o fread.o 
* 把这些零散的目标文件直接提供给 开发者，造成组织上的不便， 人们采用 ar 压缩程序将目标文件组合到一起，并对其进行编号索引，以便于查找和检索，形成了lib.a 静态库文件.
* 可以使用ar -t libc.a 来查看文件
* gcc -static --verbose main.c 可以打印详细的 编译过程 

#### 链接脚本控制： 即是 控制ld 参数的方式： 
* 使用命令行指定参数， 比如ld -e -o
* 将连接指令存放在目标文件中， 编译器 会通过此类方法 向链接器 传递指令，比如visual c++ 将参数 放在 PE目标文件的 .drectve 段
* 链接控制脚本， 单独将连接配置放到一个 .lds 文件中， 详细的内容 需要查看参考资料

### 动态连接: 页映射，
#### 装载的方式： 程序执行时所需要的指令和数据 都必须在内存中 才能够正常运行。将程序的指令和数据 装载到内存中 就称为 装载。覆盖装入（overlay） 和页面映射 (Paging) 是典型的两种装载方法。
##### 页映射：
* 将内存和磁盘中的 数据和指令 按照 页（page） 为单位划分为若干个页，即 装载和操作的单位是页。 
* 操作系统角度看 可执行文件的装载：
* 在 程序中使用物理地址直接进行操作时，都需要硬件的MMU 进行 虚拟地址 到 page 的地址转换功能，

#### 进程的建立：
* 1. 创建一个独立的虚拟地址空间
  * 只需要分配一个页目录 即可， 甚至不需要设定页面映射关系，可以等到后面程序发生页错误的时候 在进行设置 
* 2. 读取可执行文件头， 并建立 虚拟空间到 可执行文件的映射关系
  *  虚拟空间 与 可执行文件的映射关系。 当 发生缺页错误时，操作系统 应该知道 当前所需页 在可执行文件中的哪一个位置，这就是这步骤的目的
  * 该数据结构 只保存在 操作系统内部，Linux 将 进程虚拟空间中的一个段 叫做 虚拟内存区域(virtual Memory Area)  操作系统创建进程后
  * 会在进程相应的数据结构中设定一个 .text 段的VMA 
* 3. 将CPU 的PIC 寄存器 设定为 可执行文件的文件入口地址， 启动运行
  * 操作系统通过设定CPU的执行寄存器将控制权交给 进程，由此进程开始执行。 看似简单，操作系统层面比较复杂， 涉及到内核堆栈到用户堆栈的切换，CPU运行权限的切换， 等
  * 进程角度则 认为操作系统执行了一个跳转指令 到 可执行文件的入口地址， ELF 文件头中的 入口地址
  
#### 进程虚拟空间分布：
* elf 文件 链接视图 与 执行视图 
* elf 映射到 内存时， 当段的数量增多，就会 产生空间浪费问题， 因为 是以系统的页大小问映射单位，段 映射的长度都是系统页的整数倍。如果不是，那么段 多余的部分也将映射 一个页，那么如何减少空间浪费呢？
* 操作系统视角看 可执行文件，我并不关心 文件的 各个段的内容，我们只关心与装载相关的 权限问题 （即 段是否可以被修改，共享等）， 对于相同权限的段，我们将他们合并到一起当做 一个段进行映射，比如.text .init。 段的权限组合有以下几种：
* 以代码段 为代表的权限为 可读可执行的段
* 以数据段和bss段为代表的权限为 可读可写的段
* 以只读数据段为代表的权限为 只读的段
* Segment: elf 引入segment，一个segment 包含一个或多个属性类型的section，将.text 与 .init 段合并在一起看做一个 segment
* 装载时候就可以 一起映射。也就说 映射以后 在进程的虚拟空间中 只有一个对应的VMA， 这样的好处就是 明显的减少了 页面的内部碎片
* Segment 概念实际上上从 装载的角度 重新划分elf 的各个段， 将目标文件连接成可执行文件的时候，链接器会尽量将相同权限属性的各个段 分配在同一Sgemtn中
* 而系统正式按照Segment 而不是section进行映射的
* 再看了通篇 概念之后，我们可能会问， 为什么 最后要按照 segment 把文件组合呢？ 在前面的 重定位 章节中，我们以为 地址的计算是按照 合并各个段 然后进行计算的，
* 其实不是的， <em> VMA 即是最后的 虚拟地址， 重定位的 地址计算也是按照 VMA 得来的。也即是说 链接最后的结果 就是 装载视图。链接视图只是其中的过程而已。</em>
* 描述 segment 的 结构叫做程序头 (Program Header ) 描述了 elf文件该如何被操作系统 映射到进程的 虚拟空间。

##### 堆栈：在进程的虚拟空间中，同样是以 VMA 存在的， 很多情况下，一个进程的堆和栈 都有一个对应的VMA， Linux可以通过/proc 来查看进程的 虚拟空间分布


```shell
vagrant@precise64:/vagrant_data/link_test$ sleep 100 &
[1] 3834
vagrant@precise64:/vagrant_data/link_test$ cat /proc/3834/maps
00400000-00406000 r-xp 00000000 fc:00 2359461                            /bin/sleep
00605000-00606000 r--p 00005000 fc:00 2359461                            /bin/sleep
00606000-00607000 rw-p 00006000 fc:00 2359461                            /bin/sleep
01ee5000-01f06000 rw-p 00000000 00:00 0                                  [heap]
7fba0d791000-7fba0da96000 r--p 00000000 fc:00 3413573                    /usr/lib/locale/locale-archive
7fba0da96000-7fba0dc4d000 r-xp 00000000 fc:00 2752737                    /lib/x86_64-linux-gnu/libc-2.15.so
7fba0dc4d000-7fba0de4c000 ---p 001b7000 fc:00 2752737                    /lib/x86_64-linux-gnu/libc-2.15.so
7fba0de4c000-7fba0de50000 r--p 001b6000 fc:00 2752737                    /lib/x86_64-linux-gnu/libc-2.15.so
7fba0de50000-7fba0de52000 rw-p 001ba000 fc:00 2752737                    /lib/x86_64-linux-gnu/libc-2.15.so
7fba0de52000-7fba0de57000 rw-p 00000000 00:00 0
7fba0de57000-7fba0de79000 r-xp 00000000 fc:00 2756455                    /lib/x86_64-linux-gnu/ld-2.15.so
7fba0e06c000-7fba0e06f000 rw-p 00000000 00:00 0
7fba0e077000-7fba0e079000 rw-p 00000000 00:00 0
7fba0e079000-7fba0e07a000 r--p 00022000 fc:00 2756455                    /lib/x86_64-linux-gnu/ld-2.15.so
7fba0e07a000-7fba0e07c000 rw-p 00023000 fc:00 2756455                    /lib/x86_64-linux-gnu/ld-2.15.so
7fff892a4000-7fff892c5000 rw-p 00000000 00:00 0                          [stack]
7fff89388000-7fff89389000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]       
对于 输出结构 的描述：

第一列为 VMA 的地址范围， 第二列 为 权限， 第三列 为偏移，标识VMA对应的Segment 在映射文件中的偏移， 
第四列为 映射文件 所在设备的主、次设备号 （可以为0 即没有映射文件 比如stack heap）， 第五列： 映射文件的节点号， 最后为 映射文件路径
```

#### 进程栈初始化： 进程运行存在一些环境变量以及 运行参数， 常见的做法 是 操作系统 在进程启动时 将这些基本信息 提前保存到 进程的虚拟空间栈中

### 动态链接： 
* 为什么需要动态链接： 1） 静态链接 对于计算机内存和磁盘的空间浪费非常严重， 特别是多进程的时候，极大的浪费了内存空间，  2） 更新、部署 困难， 基础库的更新 导致 所有的依赖程序 需要重新编译
* 解决方法即是： 把程序的模块相互分隔开来，形成独立的文件，而不再将他们静态的链接在一起。即是： 不对那些组成程序的目标文件进行连接，等到程序要运行时 才进行连接。 
* 把链接过程推迟到了运行时再进行，这就是动态链接的基本思想
#### 动态链接过程的优势； 动态链接简单过程
* Programe1 依赖  Lib.o
* Programe2 依赖  Lib.o
* 运行 Programe1 时，系统加载 Programe1.o 然后加载依赖 Lib.o 如果Lib.o 依赖其他文件，则依次递归加载所有依赖。
* 当所有依赖关系加载完成之后， 开始链接工作，链接工作 与静态链接非常相似， 包括 符号解析，地址重定位等。
* 完成上述工作之后，将控制权交给Programe1.o 的程序入口处。程序开始运行，

* 当运行Program2时，系统加载Program2.o，但是不在加载 Lib.o 因为系统已经在运行Program1时，加载了 Lib.o
  * 1. 通过以上过程，可以发现 动态链接 节省了一次Lib.o 的依赖 加载。 
  * 2. 在其中如果更新Lib.o 时，不需要 重新编译 Programe1, 与 Programe2 仅仅 更新Lib.o 在下一次重新运行Program1，2时，即可使用最新的依赖
  ##### 程序可扩展性和兼容性：
  * 动态链接 可以在 程序运行时 动态的选择加载各种程序模块，这个特点被人们用来开发 插件功能
  * 动态链接可以加强程序的兼容性，  一个程序可以再不同的平台运行时 可以动态的链接到右操作系统 提供的动态链接库，比如 程序依赖printf 函数， 操作系统 A与 B 只需要提供 一样的printf 接口，即可实现 程序在操作系统AB上的运行
  ##### 动态链接 的基本实现：
  * 动态链接涉及到 各个文件的装载， 需要操作系统的支持， 因为在动态链接的情况下，进程的虚拟地址空间分布 会比 静态链接 情况下更加复杂，
  * 内存管理、内存共享、进程线程等机制，在 动态链接下也会有一些变化，Linux 中动态链接文件 称为 动态共享对象(DSO Dynamic Shared Objects) 以.so 为扩展名， Windows中 动态链接文件称为 动态链接库(Dynamical Linking Library) .dll 为扩展名
  
#### 动态链接 导致的问题： 因为动态链接 将在程序编译阶段的工作推迟到了 运行时间段，导致 程序每次被装载时 都要进行 重新进行连接。导致一些程序性能上的损失，性能损失 大约在5%以下。这些损失 换来 程序在空间上的 节省和程序构建升级上的便利性，是相当值得的。

### 地址无关代码：
* 装载时重定位： 为了能够共享对象在任意地址被装载， 第一个方法为： 装载时重定位， 在链接时，对所有 绝对地址的应用不做重定位，而把这一步推迟到装载时在完成，即：一旦模块装载地址确定 即目标地址确定，name系统就对模块中的所有绝对地址 引用进行重定位，即， 系统需要遍历所有的重定位表，对所有 符号的地址引用进行修改。
* 静态链接中提到的重定位： 链接时重定位， 现在为 装载时重定位。 
* 存在问题： 指令中对绝对地址函数的应用，被修改好，不能很好的在多个进程中共享。
** 地址无关代码 的基本思想： 将指令中那些需要被修改的部分分离出来，跟数据部分放在一起，这样指令部分可以保持不变，而数据部分可以在每个进程中拥有一个副本。 即是地址无关代码技术(PIC position-indenpendent Code)
* 具体解决方法：共享对象模块中的地址应用 分为两类： 模块内部引用、模块外部应用 按照应用类型不同 又分为 函数引用和数据引用， 可分为4类：
  * 1. 模块内部的函数调用、跳转
  * 2. 模块内部的数据访问， 如模块中定义的全局变量、静态变量
  * 3. 模块外部的函数调用、跳转
  * 4. 模块外部的数据访问， 如 其他模块中定义的全局变量
* 条件对应的 解决方法为： 
  * 1. 即是：相对 位置调用， 因为同在模块内，函数间 的地址相对位置是确定的
  * 2. 相对寻址，我们知道， 一个模块前面一般是若干页的代码 后面跟 若干页 的数据，这些页 的相对位置是固定的， 所以 可以使用 指定+ 相对位置 间隔 来访问模块内部的数据 （这里是否存在其他问题呢？ 确定 系统装载时候， 代码与数据段的相对位置是固定的吗？）
  * 4. 模块间的 地址访问 （数据访问、函数地址访问） 比较麻烦， 因为 模块间的地址访问 需要等到装载时候才能决定， 要使该类 代码地址无关，基本思想： 把地址相关的部分放到数据段里面，具体 ELF 的做法就是在数据段里面建立一个指向这些变量的数组指针， 称为 全局偏移表(Global offset Table GOT) 当代吗需要引用该全局变量时候，可以通过GOT中的相对应的项间接引用, 图示如下： 
* 当指令中：需要访问变量b时，程序会找到GOT， 然后根据GOT中的变量所对应的项找到变量的目标地址， 链接器 在装载模块时，会查找每个变量所在的地址，然后填充GOT中的各个项， 以保证 项目地址指向正确。因为GOT 在数据段中，所以他可以在装载时候被修改，并且每个进程都有独立的副本存在。

> gcc -fPIC -shared -o name.so name.c
* 3. 同4中一样， 使用 GOT 来保存 函数地址

|          | 指令跳转、调用      | 数据访问      |   |   |
|----------|---------------------|---------------|---|---|
| 模块内部 | 相对跳转和调用      | 相对地址访问  |   |   |
| 模块外部 | 间接跳转和调用(GOT) | 间接访问(GOT) |   |   |

-fpic 与 -fPIC 参数的区别: -fPIC 产生的代码要更大， 更跨平台， 而-fpic 产生的代码更小， 也更快，但是存在一些限制，更与硬件相关


#### 由上可见： 动态链接 会使程序变慢的 原因有： 1） 动态装载，重定位计算， 2） 共享代码段中 通过GOT来访问数据，调用函数


> 延迟绑定（Procedure Linkage Table）:  共享模块加载时候，程序需要浪费不少时间来 用于解决模块之间的函数应用的符号查找 以及重定位。 而在实际中 大量的函数很少被使用到， 所以 ELF采用了一种叫做延迟绑定的做法， 基本的思想就是 当函数第一次被用到时候 才进行绑定（符号查找，重定位等）


#### 动态链接 文件启动 顺序： 与静态链接情况基本一致， 首先操作系统 读取可执行文件 的头部，检查合法性， 并从 header中读取各个Segment 映射到进程虚拟空间的对应文职。 这时候差异开始显现， 静态链接情况下， 操作系统将 控制权直接交给 可执行文件的入口地址，然后程序开始执行。 动态链接情况下： 操作系统并不能将控制权交给可执行文件， 因为其 依赖的 很多共享对象，还没有加载起来，所以操作系统 先启动一个动态链接器，由动态链接器 来进行一系列的初始化工作，并进行动态链接工作，当所有 链接工作完成以后， 动态联机器将 控制权 交给可执行文件 的入口地址，程序开始正式执行
1. 动态链接器的路径配置：  .interp 段 中 为 ld 的具体路径
2. 动态符号表： 为了完成链接， 需要 所依赖的符号和相关文件的信息， 在静态链接中 有 段为 .symtab (Symbol Table) 动态链接中 则为 Dynmic Symbol Table .dynsym 其中只保存 动态链接相关的符号。很多动态链接库中同时存在.dynsym 以及 .symtab 两个表， .symtab 中 保存了所有的符号。其中的符号 字符串 则保存在 .dynstr (Dynamic String Table) 其中往往还有 符号哈希表 用于加快符号查找速度

readelf -sD  lib.so 


动态链接 需要在装载时候 进行重定位， 对于 使用 PIC 模式编译的（地址无关代码） 则只需要对GOT进行 重定位， 非PIC编译的，则在装载时候 重定位（即 代码段 也被 修改，而无法与其他程序共享） 
重定位结构： 动态链接中 重定位表 为.rel.dyn, .rel.plt  对应于静态链接中的 .rel.text, .rel.data 分别对应  .got （数据段）, 以及 .got.plt （代码段，函数调用地址的修正）



root@precise64:/vagrant_data/link_test# readelf -r main.so

Relocation section '.rela.dyn' at offset 0x420 contains 4 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000201010  000000000008 R_X86_64_RELATIVE                    0000000000201010
000000200fd0  000300000006 R_X86_64_GLOB_DAT 0000000000000000 __gmon_start__ + 0
000000200fd8  000400000006 R_X86_64_GLOB_DAT 0000000000000000 _Jv_RegisterClasses + 0
000000200fe0  000500000006 R_X86_64_GLOB_DAT 0000000000000000 __cxa_finalize + 0

Relocation section '.rela.plt' at offset 0x480 contains 2 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000201000  000200000007 R_X86_64_JUMP_SLO 0000000000000000 name + 0
000000201008  000500000007 R_X86_64_JUMP_SLO 0000000000000000 __cxa_finalize + 0



* 动态链接时候，进程堆栈的初始化信息： 当操作系统将控制权交给 动态链接器的时候， 除了 进程初始化时候的，堆栈里面关于进程执行环境和命令行参数等， 还包含 不少的 动态链接器所需要的辅助信息，比如 可执行文件的入口地址等， 
* 这些信息往往由操作系统传递给动态链接器

#### 动态链接器自举： 因为动态链接器 也是一个共享库，而动态链接器是操作系统的程序入口， 所以动态链接器如何递归的解决自己的依赖关系，完成启动，是一个 非常有意思的事情。
  * 动态链接器完成自举之后，链接器将 可执行文件和链接器本身的符号表合并到一个符号表中， 称为 全局符号表
  * 之后 开始寻找可执行文件所依赖的共享对象， 在.dynamic 段中， 类型为NEEDED 为 可执行文件所依赖的 共享对象，由此 链接器可以获得 可执行文件的所需要的所有的共享对象，其组成的一个集合
  * 称为 装载集合， 从装载集合中取出来一个文件，打开 并读取对应的ELF header 和.dynamic 段，将其所依赖的其他的共享对象 加入到家在集合中，如此递归的加载所有依赖的共享对象
  * 常见的加载算法为 广度优先
  * 当新的共享对象被加载进来的时候，他的符号表会被合并到全局符号表中， 所以 当所有的共享对象 被装载进来的时候，全局符号表里面将函数进程中所有的动态链接所需要的符号
  * 所以动态链接器的加载顺序决定了符号的优先级 
  * 链接器开始变量可执行文件和每个共享对象的重定位表，将他们的 .got.plt 中的每个需要重定位的位置进行修正， 因为此时动态链接器已经拥有了进程的全局符号表，所以这个修正过程会比较容易
  * 重定位完成后，如果某个共享对象 存在.init 段， 则动态链接器 会执行.init 段中的代码， 泳衣实现共享对象特有的初始化代过程， 比如 c++中 全局对象的构造过程， 共享对象中的 .finit 段，则在进程退出时执行 实现共享对象 的析构类型操作
  * 与 链接器执行的.init 对比，则为 可执行文件的 .init, .finit段的执行者， 需要在后面补充
##### 符号优先级： 
  * 动态链接器 按照 各个模块之间依赖关系，对他们进行装载并将他们的符号 并入到 全局符号表。 那么就有可能 两个不同的模块定义了 同一个符号，导致符号歧义，
  * 该现象称为 共享对象全局符号介入 问题，  那么链接器 如何 解决此类问题呢？ 其规则为： 当一个符号需要加入全局符号表时，如果相同的符号名已经存在，对于
  * 则后加的符号 被忽略。由于存在该类问题， 所以 当程序中存在大量的 共享对象 时候，应该非常小心符号的重名问题， 如果 两个符号重名  又执行不同的功能
  * name程序运行时 可能会将所有的该符号的应用解析到 第一个被加入全局符号表的符号，从而导致 莫名其妙的错误

##### 如何 避免全局符号介入问题： 对于 导出 符号（全局符号） 是不可能解决的，然而 内部函数调用， 可以使用 static 关键字 来避免其成为 全局符号 而被 其他模块覆盖，则在内部 程序直接使用 内部 相对地址调用，也加快了 程序的调用速度。 即 不适用 .got.plt 来 进行 间接跳转。
  * 则 更一般的规律为： 文件内部函数调用 都采用的 .got.plt 进行跳转调用，除非 被调用函数为static 则不会成为符号，而是用 内部相对地址调用
  * 
##### 动态链接器 的接个问题：
  * 动态链接器 本身是 动态链接的还是 静态链接的？： 静态链接的， 因为 他不能依赖 与其他共享对象， 其存在的目的 即为 解决 其他elf 动态链接文件 的依赖关系的，所以不能存在依赖
  * 动态链接器 本身必须是PIC 的吗？ 并不是， 动态链接器 的目的是帮助其他elf解决依赖关系，本身是否PIC  的 决定 其代码是否 能够共享。如果 PIC 则会更加简单一些，因为 非PIC的话，链接器在自举的时候 往往 还需要重定位自己的代码段
  * 动态链接器 是否可以被当做可执行文件运行？ 可运行的话，其装载地址是多少？： 可以当做可执行文件来运行， 其装载地址 没有什么区别 几位 0x00000000 这是一个无效地址 ，作为共享库，内核在装载时， 会为期选择一个合适的装载地址
##### 显式 运行时链接(Explicit Run-time Linking) 即是：程序在运行期间 可以主动的加载指定的 动态链接库， 而共享对象 不需要任何的修改 即可 被 运行时装载
  * 这种共享对象往往被叫做 动态装载库 (Dynamic Loading Library) 本质 上与 一般的共享对象 没有任何区别，只是 使用的角度不同而已
  * 其理论上 与 可执行文件 依赖 共享对象 一样的执行方式
  * 主要API有： dlopen  dlsym, dlerror, dlclose 
  
##### 共享库的 组织方式： 
  *  因为共享对象的优点，导致 大量的程序开始使用共享对象，导致系统中存在数量极为庞大的共享对象， 所以操作系统一般会对共享对象的目录组织和使用方式 有一定的规定，即为 共享库，
  *  共享库与共享对象没有任何区别，只是 共享库为共享对象的 组织方式 的另一个称呼 而已 
###### 共享库的兼容性： 
  * 兼容更新：所有的更新只是在原有的共享库基础上添加一些内容， 原接口保持不变
  * 不兼容更新： 共享库改变了原有接口， 使用该共享库的原有接口的程序 无法运行 
  * 这里面讨论的接口为 ABI， 而非其他的接口， 导致ABI发生变化的有以下几种情况：
  * 共享库的版本命名： 用于解决 兼容性问题。 使用命名规范来 明确兼容性。共享库的文件命名如下： libname.so.x.y.z
  * x 为主要版本号(Major version number)， y 为子版本号(Minor version number)， z为发布版本号(Release version number)
  * 主版本号： 库的重大升级， 不同主版本号之间 是不兼容的，依赖旧版本共享库的程序 需要 重新编译 才能 使用最新的共享库版本， 或者 系统保留 旧的版本，旧程序才能正常运行
  * 次版本号： 增量升级， 增加一些新的接口符号，保持原有的符号不变， 高的次要版本号 要 向后兼容低次要版本号的 共享库。 一个依赖 低次要版本号的程序 可以再 高次要版本号的共享库 下运行。
  * 发布版本号： 对一些错误的修正、性能的改进等。  不添加新的接口 也不对 接口进行更改， 相同主版本号，次版本号，不同发布版本号 之间完全兼容
  * 


