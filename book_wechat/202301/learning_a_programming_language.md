# 学习一门新的语言

虽然我大学本科读的不是计算机专业，但当时正处在计算机信息化的时代，工科学生都会安排学习一些计算机相关知识。我学的第一门计算机编程语言是 FORTRAN，此后在学校陆续学习了 C 、汇编、PASCAL 等语言。毕业之后，学习的编程语言更多，总共算下来有十几门。这些语言，有的是自己主动去学习，更多的则是工作需要，边学边做项目。其实我也很羡慕有些同学可以专精一个领域，将某个语言学到极致。不过人在江湖飘，很多时候也是身不由己。做项目的时候，哪个语言做起来快，做起来简便，就会选择哪门语言。此外，不同的公司有不同的语言偏好，如果中间换过几家公司，少不了要去学习新的语言。

虽然学习的语言很多，但大部分都忘记了，比如在学校学习的 FORTRAN、汇编、PASCAL，差不多都忘光了。即使是工作后使用的第一门编程语言 Visual Basic，现在也基本上没有印象，依稀只记得拖拽控件设计界面，双击控件添加事件处理过程。纵观我的编程生涯，主力语言仍然是 C/C++。虽然早在大学就学习了 C 语言，但之后相当长（六七年）时间并没有用。等再使用的时候，早已进入 Windows 编程时代，DOS 时代使用的 Turbo C 2.0、Borland C++ 已经退出了历史舞台。这个时候使用 C++ Builder、Visual C++，基本上是重新学习。再之后是转战嵌入式系统、Linux、移动平台（塞班、安卓），都是围绕着 C/C++ 打转，算是坚持到了现在。

C/C++ 其实也是一门相当悠久的语言，其发展也是伴随着 UNIX 的成功而成功，长期霸占着编程语言榜首，相当长时间里都是程序员的首选语言。直到现在，C 和 C++ 仍然位居 TOIBE 语言排行榜的第二和第三，如果将 C/C++ 合起来，那是当之无愧的 No. 1。

C/C++ 语言相当于独孤求败的玄铁重剑，**重剑无锋，大巧不工**。无所不能，从操作系统、中间件、数据库、嵌入式，到后端开发都可以胜任，甚至不少应用程序也是使用它开发。当然，重剑虽然厉害，要是舞不动也是白搭。C/C++ 语言就是这样，令程序员又爱又恨，学习曲线陡峭，精通更是难上加难。功力稍差，就会被其虐得体无完肤。

当然，我要感谢 C/C++，为大龄程序员建立了一道护城河，难以学习、难以掌握、难以使用，使得这条赛道不那么拥挤。即使过了传说中的 35 岁门槛，我还能继续编程。但感谢归感谢，其实这么多年也是被 C/C++ 折磨得够呛，这里简单罗列一下 C/C++ 语言的罪状：

1. 内存泄露问题。这几乎是每个 C/C++ 程序员面临的最头疼的问题。虽然智能指针、强指针、弱指针一定程度上降低了内存管理的复杂度，但并没有完全达到 GC（垃圾回收机制） 的程度，在使用时依然得小心翼翼。浏览器内核 blink 引入了 oilpan 项目，实现了类似 GC 的类管理器，但使用起来相当复杂，而且用在对象上，raw 指针的内存管理仍然没有解决。
2. 内存访问冲突问题，一般是由于野指针导致的问题，也有因为读写内存越界导致的。C/C++ 碰到最多的问题就是系统 crash、应用 crash，这时需要分析 log、call stack 甚至是操作系统的 coredump，真的是非常痛苦。这个时候就非常羡慕 Java 程序员，内存越界不存在的，野指针也是不可能的，最多就是个空指针异常。
3. 并行计算的支持。且不说难用的 C/C++ 线程库 pthread，而且线程在不同平台上还不同，为了跨平台，不得不在此基础上进行封装。看看 chromium 中的代码，为了多线程、多进程通信，设计了一套复杂的消息循环机制。当前流行的多核心 CPU ，支持得更是差，虽然有很多第三方库弥补这一缺陷，但使用上总有些门槛。GPU的并行支持，则依赖于各厂家对 C/C++ 语言的扩充，没有一个统一的标准规范。
4. 模板、多继承更是程序员的噩梦，我们在写代码的时候，一般都避免使用这些新特性。

很多现代编程语言，号称解决 C/C++ 的这些痛点，比如 Go、RUST。目前，Go 语言在系统编程、服务器端开发取得了很大的成功，但丝毫没有动摇 C/C++ 语言的统治地位。

RUST 语言更加年轻，2015年才首次亮相。但一问世就走上巅峰，已被开发人员广泛接受，在StackOverflow开发人员调查中，连续四年（2016、2017、2018、2019）被评选为最受欢迎的语言。但目前要取代 C/C++ 为时尚早。

在 2023 年，我计划学习一门新的语言，这就是 RUST。为什么选择RUST？

RUST 将 C/C++ 的功能与Java、Haskell的安全性相结合，取得了很大成功。下可编写操作系统内核（下一个版本的 Linux 内核主线，可能就会合并用 Rust 语言提交的 PR 分支），上可写 Web 应用，至于机器学习、游戏、嵌入式、服务端，更不在话下。

RUST 语言的特征：

* 通过所有权和借用概念提供内存安全和并发安全。
* 内存安全和并发安全在编译时确保，即如果程序代码可以编译，那么内存既安全又没有数据竞争。这是Rust最吸引人的功能。
* 它还提供了Haskell中元编程的表现力。凭借不可变的数据结构和功能编程功能，Rust提供了功能并发和数据并发。
* Rust的速度非常快，纯Rust的性能甚至优于纯C。
* 在没有运行时的情况下，Rust可以完全控制现代硬件（TPU、GPU、多核CPU）。
* Rust具有LLVM支持。因此，Rust提供一流的与WebAssembly的互操作性，而且Web代码也非常快。

其实，这个时候学习一门新的语言，主要是想学其思想。一门新的语言，是如何取舍，如何解决现有语言的痛点，又会引起怎样的新问题，这都是我比较关心的。如何在没有 GC 和运行时的情况下实现内存管理的，也是我比较好奇的。在学习的过程中，我希望通过实现国密相关的算法来巩固效果，毕竟，不应用到实际项目中，很难体会到一门语言的优势和劣势。

不知大家对 RUST 语言是否有所了解，希望能和大家一起交流。
