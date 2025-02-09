# 构建高效系统：Chromium 项目构建工具的演进之路

对于 C/C++ 等编译型语言，源代码需要经过编译转化为目标文件。在简单的开发场景中，开发者可以直接使用如 gcc 等命令进行编译。然而，面对复杂系统时，传统的编译方式显得效率低下，因此各种构建工具应运而生。这些工具可以明确指定构建规则，使编写过程更加简便。例如，make、CMake 等就是常见的构建工具代表。

Chromium 作为一个大型项目，一直致力于探索更高效的构建方法。在其发展过程中，源代码的构建系统经历了多次演变。下面就聊一聊 Chromium 项目构建系统的演进历程。

## 一、GYP

我自2012年起接触 Chromium 项目，那时 Chromium 使用的是 GYP（Generate Your Projects）构建系统。GYP 并不直接指定编译规则，而是生成 Makefile、Visual Studio 的 .sln 等中间文件，随后通过 make 或 Visual Studio 等工具完成最终的构建。

GYP 是一个跨平台的构建系统，能够为不同平台生成项目文件，例如 Visual Studio 的 .sln 文件或 Makefile 等，方便开发者在不同操作系统上进行构建。Google 设计并实现了这个构建系统，主要目的是支持跨平台构建。

GYP 的设计目标之一是实现跨平台支持，使得 Chromium 能够在 Windows、Linux、macOS 等多个操作系统上进行构建。GYP 通过解析 .gyp 和 .gypi 文件来生成平台特定的项目文件。具体来说，.gyp 文件定义了项目的构建目标、源文件和依赖关系，而 .gypi 文件通常包含一些公共配置和变量定义。通过类似 C 语言头文件的包含机制，构建系统实现了模块化管理。

此外，GYP 还支持多种构建配置，如 Debug 和 Release 目标，开发者可以通过修改 .gyp 文件中的配置来满足不同的构建需求。

GYP 虽然比较强大，但其应用范围仅限于 Chromium 及相关联项目，并没有取得很大的成功。

## 二、Ninja

随着 Chromium 项目不断膨胀，传统的 Makefile 和 .sln 文件在处理大量源文件和复杂依赖时逐渐暴露出性能瓶颈，构建过程变得缓慢，严重影响了开发效率。

因此，Google 内部开发了 Ninja 构建系统，Ninja 这个名字源自“忍者”一词。Ninja 并不是为了取代 GYP，而是为了替代 Makefile。与 Makefile 类似，Ninja 也处于构建流程的同一层级，因此手动编写规则仍然较为困难。在这一阶段，GYP 支持同时生成 Makefile 和 Ninja 文件。

Ninja 的核心优势在于高效的构建速度。它通过最小化磁盘 I/O 操作和优化构建任务调度，大大提高了构建过程的效率。在多核处理器支持下，Ninja 能够并行执行多个构建任务，充分利用系统资源，加速构建过程。

Ninja 支持增量构建，根据文件的修改时间判断是否需要重新编译文件。如果源文件没有变化，Ninja 就会跳过该文件的编译，从而避免不必要的构建工作，提高了构建效率。

由于其高效性和快速性，Ninja 获得了广泛的应用，许多其他构建系统（如 CMake）也支持生成 Ninja 构建文件。

## 三、GN

GN（Generate Ninja）是为了解决 GYP 性能问题并提升构建系统灵活性而开发的一款工具。

GN 专为快速构建而设计，通过生成 Ninja 构建文件来实现高效的构建过程。GN 的构建文件（.gn 文件）语法简洁，易于编写和理解，且提供了丰富的预定义变量和函数，方便开发者进行构建配置。

GN 能够精确解析和管理源文件之间的依赖关系，确保只重新编译那些发生变化的文件及其依赖的文件，从而进一步提高构建效率。

与 GYP 一样，GN 也支持跨平台构建，能够满足 Chromium 在不同操作系统上的构建需求。

从 GYP 到 GN 的过渡过程中，Chromium 项目团队投入了大量的人力和时间。首先，他们在内部对 GN 进行了充分的测试和优化，然后逐步将项目中的 .gyp 和 .gypi 文件转换为 .gn 文件。在此过程中，团队开发了工具和脚本来自动化转换，并根据 Chromium 项目的构建需求对 GN 进行了定制化改进。

在 Chromium 的构建流程中，开发者首先使用 GN 生成 Ninja 构建文件，然后通过 Ninja 命令启动构建。例如，在 Linux 平台上，可以使用以下命令来构建 Chromium：

```
gn gen out/Default
ninja -C out/Default chrome
```

其中，`gn gen out/Default` 用于生成 Ninja 构建文件，而 `ninja -C out/Default chrome` 则用于构建 Chromium 的 chrome 目标。

与 GYP 一样，GN 目前主要用于 Chromium 项目，尚未广泛应用于其他项目。

为了进一步提高构建速度，Google 的工程师还对 GN 的依赖解析算法进行了改进，优化了 Ninja 的并行构建策略。此外，他们还引入了一些外部工具，如 Icecc（分布式编译器）和 ccache（编译缓存），以加速构建过程。Icecc 可以将编译任务分发到多台机器进行并行编译，而 ccache 则通过缓存编译结果，避免重复编译相同的源文件，从而进一步提高构建效率。

## 四、小结

在上一篇文章《[不要重复发明轮子！谷歌：我偏要](https://mp.weixin.qq.com/s/aLAfcVwBDy9dw7hGnTq-ew)》中，我提到过谷歌特别喜欢重新发明轮子。就连构建系统，谷歌也总是追求极致，展现了它作为一家伟大公司的创新精神。Chromium 项目未来还可能推出什么新奇的构建系统，谁也无法预料。不过，从 Android 系统的构建系统演变来看，谷歌已经经历了 Android.mk、Soong、Blueprint 和 Bazel 等多个阶段。因此，以后如果发现 Chromium 项目的构建系统又有新变化，也不要感到意外。

有一点可以肯定的是，通过持续的优化和创新，Chromium 的构建系统不断为开发者带来更高效、更灵活的开发体验，就是有些辛苦程序员了。