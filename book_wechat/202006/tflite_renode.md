# 没有硬件，也可以运行与测试 TFLite 应用

提到人工智能和机器学习（Marchine Learning，ML），你的脑海里是否立即会浮现计算中心、高端 GPU、成百上千的 TPU 等等。实际上，随着嵌入式设备、移动终端以及近年来物联网（Internet of Things，IoT）的发展，人工智能离我们越来越近。手机、智能音响、电话手表，甚至控制开关，都配备有一定的人工智能。特别是物联网和智能家居的快速发展，机器学习在微型低功耗设备上应用得越来越广泛。

TensorFlow 作为人工智能领域的前驱，很早就推出了针对资源受限设备（例如Arm Cortex-M MCU，微处理单元）的框架 TensorFlow Lite。现在，已经有成千上万使用 TensorFlow 的开发人员将 ML 模型部署到嵌入式和 IoT 设备上。

有朋友可能会疑惑，就一个 MCU ，内存只有几 M，CPU 速度也慢，能做什么呢？

实际上能做的事情很多，比如关键语检测或手势识别之类。想象一下吧，当你下班回家，回到黑漆漆的家中，第一件事情就是开灯。借助于 IoT 设备，我们只需轻轻说 “开灯”，而不用四处摸索开关。以前装修为了方便冬天关灯，会装上双控开关，现在有了人工智能，你只要简单说一声“关灯”就可以了。再比如智能监控，检测到家里有异常动静就报警，也是应用人工智能的例子。这些设备平常都是安安静静的待机，采用微控制器就非常适合，能耗低，也可以做得很小。

但是，在小型和嵌入式设备上开发软件比较困难，调试不方便，进行大规模的压力测试更是困难。有过嵌入式系统开发经历的朋友可能会理解，即使是有经验的嵌入式开发人员，也会花大量时间在物理硬件上刷固件和测试应用程序，有时仅仅为了实现一个简单的功能。

在嵌入式设备上开发机器学习应用，开发人员面临着更多的挑战：如何在各种硬件上反复可靠地测试各种模型，能自动完成插拔、刷机、运行等流程吗？

为了应对这一挑战，我们可以向 TensorFlow Lite MCU 团队学习，他们选择了 Renode。这是一个 Antmicro 的开源仿真框架，其目标是为嵌入式和 IoT 系统提供无硬件、持续集成驱动的工作流。

接下来，我将说明如何在没有物理硬件的情况下，使用 Renode 虚拟出 RISC-V MCU，在上面运行 TensorFlow Lite 应用。

#### Renode 简介

Renode 刚刚发布了 1.9 版，它是一个开发框架，用来模拟物理硬件系统（包括CPU、外围设备、传感器、环境等等），有了它，可以加速 IoT 和嵌入式系统的开发。

Renode 可以模拟整个系统和动态环境 - 包括将建模的样本数据馈送到模拟的传感器，然后通过自定义软件和算法读取和处理。快速运行软件而无需访问物理硬件的能力使得 Renode 成为在嵌入式和IoT设备上实验和构建 ML 应用程序的理想平台。

#### 安装 Renode 并运行

Renode 支持 Linux、Mac、Windows 平台，因为我使用的开发环境是 Ubuntu，下面就说说在 Ubuntu 18.04 上的安装，其它系统请参考 Renode 的安装说明。

* 安装依赖的 mono 包，这是一个开源的 .NET 运行时环境。

```
sudo apt install gnupg ca-certificates
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
echo "deb https://download.mono-project.com/repo/ubuntu stable-bionic main" | sudo tee /etc/apt/sources.list.d/mono-official-stable.list
sudo apt update
sudo apt install mono-devel
``` 

* 安装其它包依赖。

```
sudo apt-get install policykit-1 libgtk2.0-0 screen uml-utilities gtk-sharp2 libc6-dev
```

* 访问 https://github.com/renode/renode/releases/latest，下载针对 Ubuntu 的安装包，然后安装。

* 运行 Renode

   运行 Renode 的命令如下：
```
renode [flags] [file]
```

   你也可以不加任何参数运行 renode 命令，可以出现如下命令行交互界面：

![Renode 命令行界面](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202006/images/tflite_renode_01.png)

#### 运行 TFLite demo

首先使用下面的命令下载 demo 代码及相关的文件：

```
git clone --recurse-submodules https://github.com/antmicro/litex-vexriscv-tensorflow-lite-demo 
```

在这个库中，包含有预编译的二进制文件，因为从源码 build 还比较麻烦，我们先用该二进制文件体验 Renode 。

二进制文件位于 binaries/magic_wand 目录，但我们先进入 demo 的 renode 目录，加载“litex-vexriscv-tflite.resc“脚本：

```
cd litex-vexriscv-tensorflow-lite-demo
cd renode
renode litex-vexriscv-tflite.resc
```

Renode脚本（.resc 文件）包含有指令，用于创建所需的平台并将应用程序加载到其内存中。加载后，会出现 Renode 的命令行接口，称为“Monitor”，从中可以控制仿真。在命令行接口中，使用 start 命令开始仿真：

```
(machine-0) start
```

在模拟设备的虚拟串行端口（也称为UART-会自动在Renode中作为单独的终端打开）上，你将看到以下输出：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202006/images/tflite_renode_02.gif)

#### renode 的工作原理

Renode模拟硬件（包括 RISC-V CPU 以及 I/O 和传感器），这样二进制文件认为它在实际的硬件板运行。这是通过 Renode 的机器代码转换和全面的 SoC 支持而实现的。

首先， Renode 将应用程序的机器代码转换为本地主机机器语言。

接下来，每当应用程序尝试读取或写入任何外围设备时，该调用都会被拦截并定向到对应的模型。 Renode 模型通常（但不限于）用 C＃ 或 Python 编写，实现寄存器接口，并与实际硬件在行为上保持一致。由于这些模型是抽象的，你可以通过 Renode 命令行接口或使用脚本文件以编程方式与它们进行交互。

在上面的示例中，为虚拟传感器提供了一些离线的、预先记录的数据文件：

```
i2c.adxl345 FeedSample @circle.data
```

Renode 中运行的 TFLite 二进制文件处理数据并检测手势。

#### 小结

在本文中，我们演示了如何在没有硬件的情况下将TensorFlow Lite用于微处理器单元。 尽管本文以基于 RISC-V 的平台为例，但是 Renode 能够仿真针多种体系结构，例如Arm、POWER等，所以本文的方法也可以用在其他硬件上。

如果您想使用 Renode 探索更多的硬件和软件，请访问 https://renode.readthedocs.io/en/latest/introduction/supported-boards.html 获得所支持的主板的完整列表。

最后，仿真软件无法完全替代实际的硬件，就如同做 Android 开发，仅仅使用 Android 模拟器是不够的，最终产品还需要在真正的硬件上测试。但是借助仿真，无疑可以简化开发过程，更加方便调试。

#### 参考

1. [Running and Testing TF Lite on Microcontrollers without hardware in Renode](https://blog.tensorflow.org/2020/06/running-and-testing-tf-lite-on-microcontrollers.html)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)
