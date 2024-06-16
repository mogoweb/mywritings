# 使用不同的编译器编译 Skia，性能差距居然这么大

Skia 是一个开源的 2D 图形库，提供路径、文本、图像和渲染等图形处理功能。它最初由 Skia Inc. 开发，后来被 Google 收购，并用在多个 Google 的产品中，包括 Chrome 浏览器和 Android 操作系统中。从事 Android 系统开发的同学应该对 Skia 不陌生，Skia 小巧高效，提供了一套丰富的API，支持多种 CPU 架构和 GPU 加速渲染，支持 Windows、Linux、Mac OS、Android 等操作系统，是跨平台图应用开发的理想选择，广泛应用于移动应用、游戏和专业图形设计工具中。

之前都是在 Android 系统上使用 Skia，最近由于项目需要，需要在 Windows 上使用 Skia 进行图形处理，所以就按照文档在 Windows 下编译出 Skia 图形库。

在 Skia 的官方文档上有这样一句话：

> Skia uses generated code that is only optimized when Skia is built with clang. Other compilers get generic unoptimized code.

开始看到这样一句话不以为然，想想编译器优化差别能有多大呢？再说官方首先介绍的编译方法也是使用 Visual Studio 2017 或 Visual Studio 2019。在 Windows 下进行 C++ 开发，程序员首先想到的应该是微软的 Visual C++（曾经有 Borland 的 C++ Builder 与之抗衡）。项目中虽然使用的是 Qt，但在 Windows 下，依然使用的是 MSVC 编译器。所以我想也没有想，就选择了使用 Visual C++ 的编译器 来编译 Skia。

按照文档的步骤下载 Skia 源码，选择了 Chrome/m122 这个分支，接下来下载第三方库、工具等等，这里就不展开，最后一步是编译 Skia。前面说过了，Skia 支持多种 CPU 架构和多种 GPU 加速渲染方式，所以支持多种编译参数。Skia 采用了 gn 构建系统，提供了超级多的参数来支持各种操作系统、编译器和各种定制裁剪。比如最开始我编译的 Skia.lib 库有 500 多 M，最后调整一些参数，编译出来的 Skia.lib 只有 20 多 M。下面是我最终使用 MSVC 编译器编译 Skia 的参数：

> bin\gn gen out\Release_msvc --args="extra_cflags=[\"/MT\"] win_sdk=\"C:\\Program Files (x86)\\Windows Kits\\10\" win_sdk_version=\"10.0.20348.0\" win_vc=\"c:\\Program Files\\Microsoft Visual Studio\\2022\\Community\\VC\" is_debug=false is_official_build=true skia_use_system_expat=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false skia_use_system_harfbuzz=false skia_use_system_icu=false skia_enable_skparagraph=true skia_enable_skshaper=true skia_enable_skunicode=true

其中：

> win_sdk: 如果Windows sdk 没有安装在默认位置，需要指定此参数
> win_sdk_version: 指定 sdk 版本，如果没有指定，会使用最高版本
> win_vc: 如果Visual Stuido 没有安装到默认位置，需要指定此参数
> is_official_build: 一般选择false。如果只编译skia库自身，可以选择true，但是jpeg、png等库需要提前准备
> is_component_build: false，编译为静态库。true，编译为动态库

使用编译出来的 Skia，使用开源的一个软件 https://github.com/xland/ScreenCapture 测试了一下，发现有严重的性能问题，鼠标移动有明显的延迟，但是这个软件发布的二进制文件，却流畅得很。

首先怀疑的是 Skia 没有开启 GPU 加速，Skia 编译加上 skia_use_gl=true 开启 OpenGL 加速，也没有提升，后来看了一下这个项目的源码，其实是没有启用 GPU 加速绘制的。

接着尝试调整 Skia 的编译选项，但没有什么效果。为此，我抱着试试的心态问了一下作者，在 github 项目的 discuss 区留言，问了一下作者使用怎样编译出来的 Skia，没想到作者很快给了回复：

按照回复，我下载了 clang 编译器，并使用了如下编译参数：

> bin\gn gen out\release_clang --args="clang_win=\"D:\\sdk\\clang+llvm-18.1.6-x86_64-pc-windows-msvc\" clang_win_version=\"18\" cc=\"clang\" cxx=\"clang++\" extra_cflags=[\"/MT\"] is_debug=false is_official_build=true skia_use_system_expat=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false skia_use_system_harfbuzz=false skia_use_system_icu=false skia_enable_skparagraph=true skia_enable_skshaper=true skia_enable_skunicode=true"

然后再编译 ScreenCapture 源码，果然程序流畅了很多。

计算机硬件的发展日新月异，例如 Intel 的酷睿处理器已迈入第 14 代，而现代电脑的标配内存通常起步于 16GB。尽管如此，现代计算机的运行速度虽较十年前提升百倍，用户体验却似乎未见显著改进。这种现象可以通过早年的安迪-比尔定律来解释，该定律揭示了硬件升级与软件需求之间的矛盾：硬件性能的提升往往被新软件的需求迅速消耗。

安迪-比尔定理 （Andy and Bill’s Law）是对IT产业中软件和硬件升级换代关系的一个概括。原话是 “Andy gives, Bill takes away.（安迪提供什么，比尔拿走什么。）” 安迪指英特尔前CEO安迪·格鲁夫，比尔指微软前任CEO比尔·盖茨，这句话的意思是，硬件提高的性能，很快被软件消耗掉了。

不仅仅是 Windows 操作系统如此，应用软件也是如此。虽然新的软件功能比以前的版本强了一些，但是，增加的功能绝对不是和它的大小成比例的。现在随便一个软件都是几百兆，甚至好几个 G。

现代程序员开发软件，不会使用 C/C++ 从头写起，也很少考虑性能，而是采用一大堆框架、叠加很多中间层，这当然会导致软件越来越庞大。当然，从可维护性和开发速度上来讲，这种开发模式没有什么不好。现代软件越来越复杂，要满足的需求越来越多，如果都使用 C/C++ 来写，也不现实。所以现在普遍的模式都是业务层使用 Java、Python、JS 之类的快速开发语言编写，核心功能以及追求高性能的组件仍然使用 C/C++ 编写。在 AI 领域，虽然 Python 语言是当之无愧的 No. 1，但 AI 框架的核心，基本上都是使用 C/C++。

在这里并不是过分强调软件优化，毕竟在软件开发领域，有一句箴言：

> 过早优化是万恶之首。

“过早优化是万恶之首”这句话最初由计算机科学家唐纳德·克努斯（Donald Knuth）提出，完整的说法是：“过早的优化是万恶之首（Premature optimization is the root of all evil）”。这句话强调在软件开发过程中，过早地进行优化可能导致代码复杂度增加、降低代码的可读性和可维护性，而且往往在不了解系统的真正瓶颈前，盲目优化可能会浪费大量的时间和资源。

* 开发者可能在项目需求和系统瓶颈尚不明确时，就开始对代码进行优化。这种情况下，优化往往基于假设而非实际数据，可能导致优化工作偏离了真正需要改进的方向。
* 过早优化可能使代码变得复杂难懂，增加了后续维护和迭代的难度。
* 从成本上考虑，还可能耗费大量的时间和资源，而这些投入在项目早期可能并不划算。

关于软件优化， AI 给出了如下建议：

1. 基于性能分析优化：在进行优化之前，使用性能分析工具来确定系统的实际瓶颈。只有基于实际数据的优化，才是有效和必要的。
2. 逐步优化：在项目开发的早期阶段，可以关注于代码的正确性和功能完整性。待功能稳定后，再根据实际需要逐步进行性能优化。
3. 保持代码的可读性和简洁性：优化不应以牺牲代码的可读性和可维护性为代价。清晰和简洁的代码更容易被理解和维护。
4. 优化的优先级：对于影响用户体验的性能问题优先进行优化，例如加载时间、响应速度等。对于后端处理，如能满足业务需求，可适当延后优化。
5. 使用成熟的工具和库：利用已经过优化的第三方库和工具，可以避免重复造轮子，同时利用社区的力量来提升软件性能。

真的没有想到，编译器对性能有如此大的影响，你在工作中会进行性能优化吗？有哪些优化措施？欢迎留言讨论。

