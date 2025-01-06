---
title: linux-interface-07
date: 2018-12-30
tags: linux, books
---
The linux programming interface
----------

## 内存分配
1. 在堆上分配内存， 进程可以通过增加堆的大小来分配内存， 堆就是一段长度可变的连续的虚拟内存，初始于 进程未初始化的数据段末尾，随着内存的分配和释放而增减。通常将堆当前内存边界成为 program break
   * brk(vodi \* end_data_segment), sbrk(int increment), 两个系统调用可以改变 program break 的位置， 位置调升以后，程序可以访问新分配区域内的任何内存地址。内核会在进程首次访问新分配的地址时，会自动分配实际的物理内存页。brk 直接改变 program break 的地址， sbrk 增量的改变 break 地址， 在原有的 break 地址上 增加increment 的空间，函数返回之前的break地址，也就是新分配的地址空间的起始处，sbrk(0) 返回现有的 program break 地址。
   * malloc(size_t size), free(void \* ptr)： 库函数(建立在系统调用， brk, sbrk的基础上封装而成)，比较与系统调用， 库函数拥有不少的优点， 明显的有 **允许随意的释放内存块，他们被维护于一张空闲的内存列表中，在后续的内存分配调用时候循环使用**,
     1. malloc: 分配成功返回void* 类型指针， 因为void类型所以可以随意使用， 调用失败可能是因为program break 已经触顶，（已经没有堆空间可以分配） 则返回NULL， 虽然出错的概率很小，但是依然需要进行错误检查。
     2. free： 函数释放ptr所指向的内存块，一般情况下， free并不会降低 program break 的位置， 而是将该内存块放入到空闲的内存列表中，以便供后续的malloc使用。有如下的好处， 1）尽量的减少了 sbrk的系统调用此处
   * 调用free还是不呢？: 当进程终止时， 所有的内存都会返回给操作系统，基于内存的这一自动释放机制，对于那些分配内存并持续使用的程序而言，可以忽略free，因为在多次调用free时候不但消耗大量的cpu时间，还是使代码趋向于复杂。
   * malloc, free 的实现: 数据结构为 双向链表， len| pre | next | space | 其中len 表示该空闲内存块 的大小， pre,next 为双向链表指针，指向上一个下一个空闲内存块， space为空闲内存
     1. malloc(size_t size) 会扫描空闲链表，以找到适合大小的内存块
        * 空闲链表中的len == size 时，则直接返回给z调用者
        * len > size: 对其切割（将会出现一个合适大小的内存块+一个空闲的内存块）
        * len < size: 没有找到符合要求的内存块时，调用sbrk 重新分配适合的内存块（为了更少的系统调用sbrk, 通常mallock 会以更大的increment 调用sbrk ）
        * 更新 空闲块链表
     2. free(void * ptr) : free函数通过 ptr 内存块中 len来知道内存块大小，然后加入到 空闲块链表中。
     3. 因为free， malloc的实现导致， 1）ptr指针需要完全正确，以避免对空闲链表的错误操作。（非malloc返回的指针，绝不能调用free）， 2）不能重复释放同一个指针
     4. 除了mallock, C函数库还提供了其他的 内存分配算法版本的 内存分配函数实现。 calloc, realloc, memalign, posix_memalign, alloca（该函数从栈上分配内存，因为栈的特殊性使其有两个特点 1）只有当调用函数的位于顶部时候可用 2）不需要free， 因为函数返回时代码会重置栈指针。）

## 用户和组

1. 每个用户都存在唯一的UID， 并可以归属于多个GID
2. UID， GID 的主要用途有 1）确定各种系统资源的所有权， 2）对进程的操作资源的权限加以控制
3. /et/passwd, 用于记录用户相关的UID， home, shell etc等。 /etc/shadow 维护对应UID的加密密码。组文件 /etc/group, 维护GID， 以及对应的用户列表，

## 进程凭证
1. 每个进程都有一套数字表示UID和GID 具体如下：
    1. real user id, real group id, 实际用户id，实际组id， 确定了进程所属的用户和组，作为登陆过程之一，登陆shell 从/etc/passwd中读取相应用户密码记录的3，4字段，设定为其实际用户id & 组id，当创建进程时，将从父进程中继承这些
    2. effective user id, effective group id, 有效用户id， 有效组id。 系统通常通过结合有效用户id，组id 连同辅助组id 来授予进程权限。
    3. saved user id, saved group id, 保存的用户id， 保存的组id
    4. file-system user id, file-system group id, 文件系统用户id， 文件系统组id
    5. 辅助组id
2. set-user-id, set-group-id 程序， set-user-id 程序会将进程的有效用户id设为可执行文件的用户id， 从而获得不具备的权限。set-group-id 程序有类似的效果。可执行文件拥有两个特别的权限位 set-user-id和set-group-id位，（实际上所有文件都有，只有可执行文件比较有用）ls -l program, x 变成 代表 拥有set-user-id or set-group-id. 当运行set-user-id程序时候，内核会将进程的有效用户id变为可执行文件的用户id， set-group-id 执行类似的操作。 linux系统中常用的passwd, mount, unmount, wall(用户向tty组下所有终端写入消息)等都为set-user-id程序(set-user-id-root 来特指 root用户所拥有的 set-user-id 程序)
3. 保存用户id(saved-user-id) 当执行程序时，会发生如下事情：
   1. 可执行文件的set-user-id权限位开启，将进程等的有效用户id 设定为 可执行文件的属主，未设定则进程有效用户id不变
   2. 复制 有效id 到 set-user-id
4. 有不少的系统调用，允许将set-user-id 程序的有效用户id在实际用户id和保存set-user-id之间切换。对于与执行文件用户id相关的任何权限，程序能够随时在两种状态间切换。
5. 文件系统用户id： 在进行linux中 打开文件、改变文件属主 、修改文件权限之类的文件操作时，决定其操作权限的是 文件系统用户id， 而非 有效用户id。通常 文件系统用户id和组id都等于相应的有效用户和组id， 并且只要有效用户id发生变化，相应的文件系统用户id也会发生变化，只有linux特有的两个系统调用setfsuid(), setfsgid()才能刻意的制造出 文件系统用户id 不等于 有效用户id。因此   大部分情况下，可以忽视文件系统用户id，等同于检查 有效用户id

  * [ ] 完成对应的系统调用
  * [ ] 如何在进程中调用 特权程序?


## 时间
大多数计算机体系结构都内置有硬件始终，是的内核得以计算真实时间和进程时间。
1. 日历时间: Unix 系统内部对时间的表示，以 Epoch以来的秒数来度量（UTC 时间）。存储于time_t类型的变量中。（time_t是一个有符号整数 理论上当前许多的32位unix系统都面临着一个2038 的理论问题，如果执行的计算工作涉及到未来时间，那么在2038年问题都会出现，事实上在此之前所有的unix系统可能都已经升级到了64位系统，然而32位嵌入式系统的寿命要长的多，依然面临着这个问题）
  * gettimeofday(timeval \*tv, timezone \*tz): struct timeval { time_t tv_sec; suseconds_t tv_usec;} 其中 tv_usec提供了微秒级别的精度，参数tz是一个历史产物，应该总为NULL，
  * time_t time(time_t \*timep): 函数有两种方式返回同样的结果， UTC秒数， timep 不为NULL，将秒数放在timep 的指针中， timep NULL返回一个数值
  * 时间转换函数： 类型包含如下 time_t， 打印格式， 分解时间(即是： struct tm {int tm_sec; int tm_min; int tm_hour; int tm_mday; int tm_year; int tm_mon; etc})， 转换函数即是 用来在上面的类型中 进行转换的函数，方便使用。其中包括， strftime, mktime, gmtime, localtime 等。
  * 时区： 时区信息被系统使用标准格式保存于文件中。 /usr/share/zoneinfo, 该目录中的每个文件都包含了一个特定国家或地区的时区制度，系统的本地时间在 /etc/localtime 中定义，通常链接到 /usr/share/zoneinfo中的一个文件。使用TZ环境变量来为一个程序指定时区，其值为 ":“ + 时区名称组成的字符串。设定时区会自动影响到 ctime, locatime, mktime, strftime 等，
  * setlocale(int category, char \* locale) : 设定和查询程序的当前地区, category 可选项为 表中的数值 + LC_ALL, LANG, LANGUAGE, 其中，LC_ALL 为设定所有值而准备， LANG为设定所有为明确指定的变量而准备. setlocale 参数中的locale可以为 “”空字符串，表示可以从环境变量中却的地区的设定， 大部分的程序代码 setlocale(LC_ALL, "") 来使用程序中的环境变量设定地区，如果调用被省略，这些环境变量将不会对程序的地区设定生效。

    |---|---|
    | 文件名      | 目的                           |
    | LC_TYPE     | 包含字符分类以及大小写转换规则 |
    | LC_COLLATE  | 包含针对一字符集的排序规则     |
    | LC_MONETARY | 对货币格式化规则               |
    | LC_NUMERIC  | 对货币意外的数字格式化规则     |
    | LC_TIME     | 对日期和时间的格式化规则       |
    | LC_MESSAGES | 针对肯定和否定响应，就格式以及数值做了规定                               |

2. 进程时间:
   **软件时钟： 进程时间受限于 系统软件时钟的分辨率，度量单位 为jiffies（定义在内核代码中的常量HZ, jiffies 为1s内 cpu增加的记数，100HZ(jiffies) 时候， 1 jiffies(hz) 的时间为10毫秒）这是内核按照round-robin的分时调度算法分配cpu进程的单位。因为CPU 的速度大大提高，2.6.0的内核时钟速度已经提高了1000hz， 更高的分辨率意味着更高的时间精度，然而并非可以任意的提高时钟频率，因为每个时钟中断都对耗费少量的CPU时间**
  * 用户CPU时间： 在用户模式下执行所花费的时间数量
  * 系统CCPU时间： 在内核模式中执行所花费的时间数量

## 系统和进程信息
  **为了提供简单的方法来访问内核信息， 现在的UNIX实现提供了一个/proc 虚拟文件系统（并非存储于磁盘上，恶热是内核在进程访问信息时候动态生成的），其中包含了各种用于展示内核信息的文件。并允许进程通过常规的IO系统调用来访问，有些还可以对信息进行修改。**
* /proc/PID： 内核提供了对应PID进程的目录结构，

  |---|---|
  | 文件    | 描述                                       |
  | cmdline | 以 \0分割的命令行参数                      |
  | cwd     | 指向当前工作目录的符号连接                 |
  | Environ | NAME=value 键值对的环境列表                |
  | exe     | 指向正在执行文件的符号连接                 |
  | fd      | 文件目录包含了指向由进程打开文件的符号连接 |
  | maps    | 内存映射                                   |
  | mem     | 进程虚拟内存                               |
  | mounts  | 进程的安装点                               |
  | root    | 指向根目录的符号链接                       |
  | status  | 各类信息                                   |


* /proc/PID/fd： 目录 为进程打开的每个文件描述符都包含了一个符号连接，每个符号连接的名称都与描述符的数值向匹配（/proc/pid/1 为 标准输出）， 任何进程都可以使用符号连接 /proc/self 来访问自己的/proc/PID 目录
* /proc 目录下的系统信息:

  |---|---|
  | 目录              | 描述                   |
  | /proc/net         | 网路和套接字的状态信息 |
  | /proc/sys/fs/     | 文件系统相关的设定     |
  | /proc/sys/kernel/ | 各种常规的内核设定     |
  | /proc/sys/net     | 网络和套接字设定       |
  | /proc/sys/vm/     | 内存管理设定           |

* uname(utsname * utsbuf): 系统调用返回主机系统的标识信息

##  文件IO缓冲
**出于效率的考虑， 系统IO调用，以及 函数库IO函数，都在文件IO操作中对数据进行了缓冲**

###: 内核缓冲区： 缓冲区 高速缓存： read, write 系统调用： 系统调用并不会直接发起磁盘调用，而仅仅是在 用户空间缓冲区 与 内核缓冲区高速缓存(kernel buffer cache) 中间复制数据。 write(fd, "abc", 3) 的详细步骤： 1. 将数据从 用户空间内存传递到 内核 缓冲区中， 2. write 返回， 3）后续的某个时刻， 内核会将其 缓冲区的数据写入到 磁盘上。 read系统调用 则会产生 缺页中断， 进程挂起，知道 内核从磁盘中读取数据 并 存储到 内核缓冲区中，进程执行 从内核缓冲区中读取数据。

### stdio库的缓冲： 缓冲大块数据以减少系统调用， C语言函数库的IO函数， fprintf, fgets,fputs, fputc 等能够使
