# checksec

## Table of Contents

- [Install](#install)
- [Usage](#usage)
- [扫描结果](#扫描结果)
- [RELRO](#RELRO)
- [STACK CANARY](#STACK_CANARY)
- [NX](#NX)
- [PIE](#PIE)
- [FORTIFY](#FORTIFY)


### Install

```shell
$ sudo apt install checksec
```

### Usage

```shell
#检查单个文件
$ checksec --file /bin/ls

#检查目录下的所有文件
$ checksec --dir /bin/
```

### 扫描结果

|         RELRO|  STACK CANARY|          NX|          PIE|     RPATH|     RUNPATH|     Symbols|  FORTIFY|  Fortified|  Fortifiable|         Filename|
|--------------|--------------|------------|-------------|----------|------------|------------|---------|-----------|-------------|-----------------|
|    Full RELRO|  Canary found|  NX enabled|  PIE enabled|  No RPATH|  No RUNPATH|  No Symbols|      Yes|          1|            2|    /bin/htpasswd|
|    Full RELRO|  Canary found|  NX enabled|  PIE enabled|  No RPATH|  No RUNPATH|  No Symbols|      Yes|          4|            5| /bin/proxychains|
| Partial RELRO|  Canary found|  NX enabled|  PIE enabled|  No RPATH|  No RUNPATH|  No Symbols|      Yes|          3|            5| /bin/qmlprofiler|
|    Full RELRO|  Canary found|  NX enabled|       No PIE|  No RPATH|  No RUNPATH|  No Symbols|      Yes|          6|           12|       /bin/genrb|
| Partial RELRO|  Canary found|  NX enabled|  PIE enabled|  No RPATH|  No RUNPATH|  No Symbols|      Yes|          0|            0|        /bin/look|

### RELRO:

在Linux系统安全领域数据可以写的存储区就会是攻击的目标，尤其是存储函数指针的区域。所以在安全防护的⻆度来说尽量减少可写的存储区域对安全会有极大的好处。

GCC, GNU linker以及glibc-dynamic linker一起配合实现了一种叫做relro的技术: read only relocation。大概实现就是由linker指定binary的一块经过dynamic linker处理过relocation之后的区域为只读。

按照安全防护的⻆度应该尽量使存储区域只读，一般情况下代码里的常量或者经过编译器的分析pass而认为是常量的数据可以直接放入binary里的存储只读区。还有就是const变量，这种变量经过dynamic-linker的重定位处理之后可以位于只读存储区。

relro 这项技术通过gcc/linker/glibc dynamic-linker的协作，最大程度地减少可写存储区的大小，但是如果使用了-z now参数，会使用no-lazy的模式来生成binary，这样就会导致dynamic-linker在将控制权交给 executable 之前做更多的重定位工作，会拖慢启动速度，但是会将只读存储区更大化。

    gcc -o hello test.c             // 默认情况下，是Partial RELRO 
    gcc -z norelro -o hello test.c  // 关闭，即No RELRO 
    gcc -z lazy -o hello test.c     // 部分开启，即Partial RELRO
    gcc -z now -o hello test.c      // 全部开启，即Full RELRO

### STACK_CANARY

栈溢出保护是一种缓冲区溢出攻击缓解手段，当函数存在缓冲区溢出攻击漏洞时，攻击者可以覆盖栈上的返回地址来让shellcode能够得到执行。

当启用栈保护后，函数开始执行的时候会先往栈里插入cookie信息，当函数真正返回的时候会验证cookie信息是否合法，如果不合法就停止程序运行。

攻击者在覆盖返回地址的时候往往也会将cookie信息给覆盖掉，导致栈保护检查失败而阻止shellcode的执行。在Linux中我们将cookie信息称为canary。

gcc在4.2版本中添加了-fstack-protector和-fstack-protector-all编译参数以支持栈保护功能，4.9新增了-fstack-protector-strong编译参数让保护的范围更广。

    gcc -o test test.c						// 默认情况下，不开启Canary保护
    gcc -fno-stack-protector -o test test.c  // 禁用栈保护
    gcc -fstack-protector -o test test.c     // 启用堆栈保护，不过只为局部变量中含有 char 数组的函数插入保护代码
    gcc -fstack-protector-all -o test test.c // 启用堆栈保护，为所有函数插入保护代码


### NX

NX即No-eXecute（不可执行）的意思，NX（DEP）的基本原理是将数据所在内存页标识为不可执行，当程序溢出成功转入shellcode时，程序会尝试在数据页面上执行指令，此时CPU就会抛出异常，而不是去执行恶意指令。

gcc编译器默认开启了NX选项，如果需要关闭NX选项，可以给gcc编译器添加-z execstack参数。

    gcc -z execstack -o test test.c     // 关闭NV选项
    gcc -z noexecstack -o test test.c   // 默认打开NX选项

### PIE

PIE 全称是position-independent executable，中文解释为地址无关可执行文件，该技术是一个针对代码段（.text ）、数据段（.data ）、未初始化全局变量段（.bss ）等固定地址的一个防护技术，如果程序开启了 PIE 保护的话，在每次加载程序时都变换加载地址，从而不能通过 ROPgadget 等一些工具来帮助解题。

内存地址随机化机制（address space layout randomization)，有以下三种情况:
  - 0  表示关闭进程地址空间随机化。
  - 1  表示将mmap的基址，stack和vdso页面随机化。
  - 2  表示在1的基础上增加栈（heap）的随机化。

liunx下关闭PIE的命令：sudo -s echo 0 > /proc/sys/kernel/randomize_va_space

    gcc -o test test.c              // 默认情况下，不开启PIE
    gcc -fpie -pie -o test test.c	// 开启PIE，此时强度为1
    gcc -fPIE -pie -o test test.c	// 开启PIE，此时为最高强度2
    gcc -fpic -o test test.c		// 开启PIC，此时强度为1，不会开启PIE
    gcc -fPIC -o test test.c		// 开启PIC，此时为最高强度2，不会开启PIE"

### FORTIFY

FORTITY其实是非常轻微的检查，用于检查是否存在缓冲区溢出的错误。适用情形是程序采用大量的字符串或者内存操作函数，如memcpy, memset, stpcpy, strcpy, strncpy, strcat, strncat, sprintf, snprintf, vsprintf, vsnprintf, gets 以及宽字符的变体，目的是检查dest变量是否溢出。

    _FORTIFY_SOURCE设为1，并且将编译器设置为优化1(gcc -O1)，以及出现上述情形，那么程序编译时就会进行检查但又不会改变程序功能
    _FORTIFY_SOURCE设为2，有些检查功能会加入，但是这可能导致程序崩溃。

    gcc -D_FORTIFY_SOURCE=1     // 仅仅只会在编译时进行检查 (特别像某些头文件 #include <string.h>)
    gcc -D_FORTIFY_SOURCE=2     // 程序执行时也会有检查 (如果检查到缓冲区溢出，就终止程序)