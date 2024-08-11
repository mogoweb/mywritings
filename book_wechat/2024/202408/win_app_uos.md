# Windows应用程序是如何在国产系统上运行的

上一篇文章《[在国产系统上安装 Windows 应用程序](https://mp.weixin.qq.com/s/pu3UFz7H3U2AF9okHOfdNg)》发出来后，很多朋友问能否运行 Windows 下的大型游戏，比如 英雄联盟、穿越火线等，还有的朋友问能否使用 Windows 的驱动。对于这样的问题，很难用一句能或者不能回答。所以本文就尝试解释一下 Windows 应用程序是如何在国产系统上运行起来的，这样才能更好的回答朋友的问题。文章有些偏底层技术细节，如果对技术不感兴趣，可以直接拉到底看结论。

本文主要参考了《How Wine works 101》（原文地址：https://werat.dev/blog/how-wine-works-101/）以及 ChatGPT，文章极大地简化了实现细节，而且本人技术水平有限，并不了解所有细节，如果有不正确的地方，还请指正。

## Wine 并不是模拟器

在国产系统（基于Linux）上运行 Windows 应用程序，离不开 Wine。至于为什么要在国产系统上运行 Windows 应用程序，主要还是针对国产系统开发的应用程序太少，特别是游戏，这个强如苹果的 Mac OS，也没有能很好的解决这个难题，直到如今，Mac OS 下能玩的大型游戏还是很少。

Wine 的出现，至少为国产系统的用户提供了一个选择。有些商业系统，比如 Valve 的 Steam Deck 使用基于 Wine 的解决方案来运行游戏（称为 Proton），获得了不少用户。

但 Wine 和 Virtual Box、QEMU 之类的虚拟机不同，它并不模拟指令，正如 Wine 是 Wine Is Not an Emulator 的缩写。Wine 是一个兼容层，能够在多个符合 POSIX 的操作系统（例如 Linux、macOS 和 BSD）上运行 Windows 应用程序。项目地址：

> https://www.winehq.org

## Linux 是如何运行二进制程序的

在解释如何在 Linux 上运行 Windows 二进制程序之前，让我们先弄清楚如何运行普通的 Linux 二进制程序。

先编写一个经典的 Hello, World! 程序：

```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

使用 clang 编译成二进制程序 hello。

```bash
$ clang hello.c -o hello
```

生成二进制程序后，就可以运行了：

```bash
$ ./hello
Hello, World!
```

进一步分析这个 hello 程序：

```bash
$ ldd hello
        linux-vdso.so.1 (0x00007ffed39eb000)
        /lib/$LIB/liblsp.so => /lib/lib/x86_64-linux-gnu/liblsp.so (0x00007f3b4cc00000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3b4ca1b000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f3b4cf2b000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f3b4cf53000)
$ readelf -l hello

Elf 文件类型为 DYN (Position-Independent Executable file)
Entry point 0x1050
There are 13 program headers, starting at offset 64

程序头：
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000618 0x0000000000000618  R      0x1000
  LOAD           0x0000000000001000 0x0000000000001000 0x0000000000001000
                 0x0000000000000171 0x0000000000000171  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x00000000000000ec 0x00000000000000ec  R      0x1000
  LOAD           0x0000000000002dd0 0x0000000000003dd0 0x0000000000003dd0
                 0x0000000000000248 0x0000000000000250  RW     0x1000
```

首先，我们看到该应用程序是一个动态可执行文件，这意味着它依赖于一些动态库，并且需要它们在运行时存在才能运行。在上面的输出中还包含有 **Requesting program interpreter**（请求程序解释器）部分。C++ 不是一种编译语言吗，怎么还需要解释器？注意，这里的解释器和 Python 之类解释型语言的解释器不同，它只是一个动态加载器。简单说，其作用就是解析并加载其依赖项，然后交出控制权。

从上面的输出可以看出，hello 二进制程序的动态加载器是 /lib64/ld-linux-x86-64.so.2，我们试试用它来加载 hello：

```bashxiao
$ /lib64/ld-linux-x86-64.so.2 ./hello
Hello, World!
```

我们还可以输出更详细的信息，日志很多，下面只截取一部分：

```bash
$ LD_DEBUG=all /lib64/ld-linux-x86-64.so.2 ./hello
     21540:     file=./hello [0];  generating link map
     21540:       dynamic: 0x00007fc958f30de0  base: 0x00007fc958f2d000   size: 0x0000000000004020
     21540:         entry: 0x00007fc958f2e050  phdr: 0x00007fc958f2d040  phnum:                 13
     21540:
     21540:     symbol=__vdso_clock_gettime;  lookup in file=linux-vdso.so.1 [0]
     21540:     binding file linux-vdso.so.1 [0] to linux-vdso.so.1 [0]: normal symbol `__vdso_clock_gettime' [LINUX_2.6]
     21540:     symbol=__vdso_gettimeofday;  lookup in file=linux-vdso.so.1 [0]
     21540:     binding file linux-vdso.so.1 [0] to linux-vdso.so.1 [0]: normal symbol `__vdso_gettimeofday' [LINUX_2.6]
     21540:     symbol=__vdso_time;  lookup in file=linux-vdso.so.1 [0]
     21540:     binding file linux-vdso.so.1 [0] to linux-vdso.so.1 [0]: normal symbol `__vdso_time' [LINUX_2.6]
     21540:     symbol=__vdso_getcpu;  lookup in file=linux-vdso.so.1 [0]
     21540:     binding file linux-vdso.so.1 [0] to linux-vdso.so.1 [0]: normal symbol `__vdso_getcpu' [LINUX_2.6]
     21540:     symbol=__vdso_clock_getres;  lookup in file=linux-vdso.so.1 [0]
     21540:     binding file linux-vdso.so.1 [0] to linux-vdso.so.1 [0]: normal symbol `__vdso_clock_getres' [LINUX_2.6]

```

运行可执行文件时，Linux 内核会检测到它是动态的，并且需要加载程序。然后它执行加载程序，加载程序会完成所有工作。我们可以通过在调试器下运行程序来验证这一点。

```bash
$ lldb ./hello
(lldb) target create "./hello"
Current executable set to '/work/mywork/uos/development/wine/hello' (x86_64).
(lldb) process launch --stop-at-entry
Process 24342 stopped
* thread #1, name = 'hello', stop reason = signal SIGSTOP
    frame #0: 0x00007ffff7fe5810 ld-linux-x86-64.so.2`_start
ld-linux-x86-64.so.2`_start:
->  0x7ffff7fe5810 <+0>: movq   %rsp, %rdi
    0x7ffff7fe5813 <+3>: callq  0x7ffff7fe6400            ; _dl_start at rtld.c:518:1

ld-linux-x86-64.so.2`_dl_start_user:
    0x7ffff7fe5818 <+0>: movq   %rax, %r12
    0x7ffff7fe581b <+3>: movq   (%rsp), %rdx
Process 24342 launched: '/work/mywork/uos/development/wine/hello' (x86_64)
```

我们可以看到，执行的第一条指令是在 ld-linux-x86-64.so.2 中，而不是 hello 二进制文件中。

小结一下，在 Linux 上运行二进制程序的过程大致如下：

1. 内核加载映像（≈二进制文件）并发现它是一个动态可执行文件;
2. 内核加载动态加载器 (ld-linux-x86-64.so.2ld.so) 并赋予其控制权;
3. 动态加载器解析依赖项并加载它们;
4. 动态加载器将控制权交还给原始二进制文件;
5. 原始二进制文件在 _start() 中开始执行并最终到达 main();

如果 Linux 下直接运行一个 Windows 二进制程序，会是什么结果？

```bash
$ x86_64-w64-mingw32-gcc hello.c -o hello.exe
$ ./hello.exe
bash: ./hello.exe: 无法执行：找不到需要的文件
```

很明显，简单地运行 Windows 可执行程序是行不通的，Linux 无法识别 Windows 应用程序的格式，内核根本不知道如何处理它。

但是，Windows 应用程序的格式并非秘密，还是有办法写个程序处理它。

从操作系统的角度来看，**运行**二进制程序意味着什么？

每个可执行文件都有 .text 部分，其中包含序列化的 CPU 指令：

```bas
$ objdump -drS hello

hello：     文件格式 elf64-x86-64


Disassembly of section .init:

0000000000001000 <_init>:
    1000:       48 83 ec 08             sub    $0x8,%rsp
    1004:       48 8b 05 c5 2f 00 00    mov    0x2fc5(%rip),%rax        # 3fd0 <__gmon_start__@Base>
    100b:       48 85 c0                test   %rax,%rax
    100e:       74 02                   je     1012 <_init+0x12>
    1010:       ff d0                   call   *%rax
    1012:       48 83 c4 08             add    $0x8,%rsp
    1016:       c3                      ret

Disassembly of section .plt:

0000000000001020 <printf@plt-0x10>:
    1020:       ff 35 ca 2f 00 00       push   0x2fca(%rip)        # 3ff0 <_GLOBAL_OFFSET_TABLE_+0x8>
    1026:       ff 25 cc 2f 00 00       jmp    *0x2fcc(%rip)        # 3ff8 <_GLOBAL_OFFSET_TABLE_+0x10>
    102c:       0f 1f 40 00             nopl   0x0(%rax)

0000000000001030 <printf@plt>:
    1030:       ff 25 ca 2f 00 00       jmp    *0x2fca(%rip)        # 4000 <printf@GLIBC_2.2.5>
    1036:       68 00 00 00 00          push   $0x0
    103b:       e9 e0 ff ff ff          jmp    1020 <_init+0x20>

Disassembly of section .plt.got:

0000000000001040 <__cxa_finalize@plt>:
    1040:       ff 25 9a 2f 00 00       jmp    *0x2f9a(%rip)        # 3fe0 <__cxa_finalize@GLIBC_2.2.5>
    1046:       66 90                   xchg   %ax,%ax

Disassembly of section .text:

0000000000001050 <_start>:
    1050:       31 ed                   xor    %ebp,%ebp
    1052:       49 89 d1                mov    %rdx,%r9
    1055:       5e                      pop    %rsi
    1056:       48 89 e2                mov    %rsp,%rdx
    1059:       48 83 e4 f0             and    $0xfffffffffffffff0,%rsp
    105d:       50                      push   %rax
    105e:       54                      push   %rsp
    105f:       45 31 c0                xor    %r8d,%r8d
    1062:       31 c9                   xor    %ecx,%ecx
    1064:       48 8d 3d d5 00 00 00    lea    0xd5(%rip),%rdi        # 1140 <main>
    106b:       ff 15 4f 2f 00 00       call   *0x2f4f(%rip)        # 3fc0 <__libc_start_main@GLIBC_2.34>
    1071:       f4                      hlt
    1072:       66 2e 0f 1f 84 00 00    cs nopw 0x0(%rax,%rax,1)
    1079:       00 00 00 
    107c:       0f 1f 40 00             nopl   0x0(%rax)


```

为了**运行**可执行文件，操作系统将二进制文件加载到内存中（特别是 .text 部分），将当前指令指针设置为代码所在的地址，这样可执行文件就可以运行了。我们可以对 Windows 可执行文件做同样的事情吗？

是的！可执行文件中的代码在 Windows 和 Linux 之间是**可移植的**（假设 CPU 架构相同）。如果我们只是从 Windows 可执行文件中取出代码，将其加载到内存中并将 %rip 指向正确的位置 - 处理器会很乐意执行它！

回顾一下在 Linux 上运行二进制程序的 5 个步骤，如果我们能完成步骤 1-4 并以某种方式到达步骤 5，那么理论上应该可以实现在 Linux 下运行 Windows 应用程序。

## Wine 的作用

本质上，wine 是 Windows 可执行文件的**动态加载器**。它是原生 Linux 二进制文件，因此 Linux 下可以正常运行，并且它还知道如何处理 Windows 的 EXE 和 DLL，其作用 相当于 ld-linux-x86-64.so.2：

```bash
# 运行 ELF 二进制文件
$ /lib64/ld-linux-x86-64.so.2 ./hello
Hello, World!
# 运行 PE 二进制文件
$ deepin-wine8-stable ./hello.exe 
wine: created the configuration directory '/home/alex/.wine'
wine version: 8.16
wine: configuration in L"/home/alex/.wine" has been updated.
Hello, World!
```

wine 将 Windows 可执行文件加载到内存中，解析它，找出依赖项，找出可执行代码的位置（即 .text 部分），然后最终跳转到该代码。

实际上，它会跳转到 ntdll.dll!RtlUserThreadStart() 之类的东西，这是 Windows 世界中的**用户空间**入口点。它最终将到达 mainCRTStartup()（相当于 Linux 的 _start），然后最终到达实际的 main()。

这样，Linux 系统就运行起 Windows 应用程序了，看起来很简单很完美啊。但是 ……

## 系统调用

写过代码的朋友应该了解，我们写程序，并非只是写代码，很多时候还会调用操作系统的 API。就拿读写文件来说，虽然有些库屏蔽了操作系统的差异，但最终还是会调用到操作系统 API，因为文件读写由操作系统统一管理。

系统调用不是代码中的常规函数调用。例如，打开文件必须由内核本身执行，因为它需要跟踪文件描述符。因此，应用程序代码需要一种“中断”自身并将控制权交给内核的方法（此操作通常称为上下文切换）。

让问题变得棘手的原因在与，各操作系统提供的系统调用是不一样的。

Linux 上的示例：read、write、open、brk、getpid

Windows 上的示例：NtReadFile、NtCreateProcess、NtCreateMutant

而且，调用方式在每个操作系统上也不相同。例如，在 Linux 上，为了调用 read()，二进制文件会将文件描述符放入寄存器 %rdi，将缓冲区指针放入 %rsi，将要读取的字节数放入 %rdx。然而，在 Windows 上，内核中没有 read() 函数，这两个参数都没有任何意义。因此，为 Windows 编译的二进制文件将使用 Windows 方式执行系统调用，这在 Linux 上不起作用。

看起来，Wine 遇到这种系统调用也没辙，但事情还有挽回的余地。

在 Windows 上，一般应用程序不会直接调用系统调用，因为这涉及到与内核通信，调用上比较繁琐。所以，Windows 提供了 kernel32.dll / kernelbase.dll / ntdll.dll，它们屏蔽了与内核通信的细节。应用程序只需调用一个函数，其余部分由库处理：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/win_app_uos_01.png)

看到这里，很多朋友可能会想到一个方案，在 Linux 下重新实现 ntdll.dll，它是进入内核的“网关”，来个截胡不就可以了？

Wine 确实提供了它的自定义实现。在 Wine 的最新版本中，它由两部分组成：ntdll.dll（这是一个 PE 库）和 ntdll.so（这是一个 ELF 库）。第一个是一个薄层，只是将调用重定向到 ELF 对应部分。 ELF 对应部分包含一个名为 __wine_syscall_dispatcher 的特殊函数，它执行将当前堆栈从 Windows 转换为 Linux 并转回的魔术。

因此，在执行系统调用时，使用 Wine 运行的进程的调用堆栈如下所示：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/win_app_uos_02.png)

看起来，又一难题得到完美解决。问题是 ......

## 不太完美的 Wine

经过前面的分析，Wine 运行 Windows 程序是没有什么问题了，但仍然面临着许多挑战：

1. Windows API 非常多，而且 Windows 是闭源软件，无法通过查看源码的方式去了解内部实现，只能通过公开的文档，但是这些 API 的文档并不那么全。
2. 某些 Windows API 实现上存在错误，实现时要不要保留这些错误？如果不保留，某些应用程序将错就错，针对错误做了某些修正，如果 Linux 下正确实现了，反而应用程序在 Linux 下会出现错误。
3. 虽然大部分应用程序不会去直接调用系统指令，但某些特殊的应用程序（特别是游戏）会这么干，Wine 就很难处理，这也是很多游戏在 Linux 下运行会有各种兼容问题的原因之一。
4. 复杂的大型游戏还会用到 DirectX、音频、输入设备（游戏手柄、操纵杆）等。想想 DirectX 有多少 API，在没有源码的情况下要完美重新实现，是多么困难的一件事情。

当然，我们也不必沮丧。Wine 已经开发了很多年，取得了长足的进步，各种问题已经解决了很多。今天，你可以毫无问题地运行最新的游戏，如 Cyberpunk 2077 或 Elden Ring。有时 Wine 的性能甚至比 Windows 还要好！

至于 Windows 驱动，Wine 对此无能为力，因为 Wine主要是一个应用层的兼容层，而驱动工作在更底层。

* 驱动程序直接与操作系统内核交互。Windows 驱动程序与 Windows 内核有紧密的耦合，而 Wine 运行在用户空间，没有权限或能力与 Linux 内核进行这种低层次的交互。
* 驱动程序通常运行在内核模式下，具有高权限，可以直接访问硬件资源。Wine 运行在用户模式下，没有内核模式的权限和能力。因此，Windows 驱动程序无法在 Wine 提供的用户模式环境中运行。
* 驱动程序需要直接访问硬件设备，而这种访问方式在不同的操作系统之间是不同的。Linux 和 Windows 的硬件访问机制不同，导致 Windows 驱动程序无法在 Linux 上正常工作。

## 小结

Wine（Wine Is Not an Emulator）是一个开源的兼容层，它允许 Windows 应用程序在 Linux 和其他类 Unix 操作系统上运行。Wine 实现了如下功能：

1. **API 翻译**：Windows 程序依赖于 Windows API 来调用操作系统的功能，例如文件操作、图形处理、内存管理等。Wine 通过实现一个兼容的 Windows API，把这些 API 调用翻译成对应的 Linux API 调用，从而使得 Windows 程序可以在 Linux 环境中运行。
2. **动态链接库（DLL）替换**：Windows 程序通常依赖于各种 Windows 动态链接库（DLL）。Wine 提供了一组与 Windows DLL 相对应的开源实现，以满足 Windows 程序对这些DLL 的需求。这些开源实现的 DLL 可以替代原生的 Windows DLL。
3. **EXE 和 PE 格式支持**：Windows 程序的可执行文件（如 .exe 和 .dll ）使用的是 PE（Portable Executable）格式。Wine 包含了一个 PE 加载器，能够读取和执行这些文件格式，并在 Linux 系统上加载和运行 Windows 程序。
4. **图形界面支持**：Wine 实现了对 Windows 图形接口（如 GDI 和 DirectX ）的支持，使得 Windows 程序可以在 Linux 上正常显示图形界面。Wine 通过使用 X Window System（X11）或其他图形系统，将 Windows 的图形调用转换成 Linux 能够理解的图形调用。
5. **注册表支持**：Wine实现了 Windows 注册表的功能，这使得 Windows 程序可以读取和写入注册表设置。Wine 在 Linux 文件系统中创建一个虚拟的 Windows 注册表文件，供 Windows 程序使用。
6. **文件系统映射**：Wine 将 Windows 文件系统路径映射到 Linux 的文件系统路径。例如，C:\ drive 通常被映射到 Linux 上的一个目录（如 ~/.wine/drive_c ）。

通过这些技术， Wine 能够在 Linux 上提供一个 Windows 兼容的运行环境，使得大多数 Windows 应用程序可以在 Linux 上运行，而无需修改程序代码。

但由于 Windows 是闭源操作系统，加上 Windows 和 Linux 操作系统架构之间的差异，导致某些 Windows API 实现在 Linux 下表现和 Windows 下不同，导致一些兼容问题。经过多年的发展，兼容性问题已经得到很大程度的解决，主流的 Windows 应用程序和许多大型游戏都能在 Linux 下完美运行。

由于驱动更加底层，所以是无法通过 Wine 使用 Windows 驱动的。
