# 在 UOS 下利用 QEMU 搭建飞腾 ARM64 的开发环境

近年来，在政府推动下，国产操作系统（主要是统信UOS和麒麟OS）以及相关软件的市场份额逐步扩大。越来越多的企业和事业单位开始采用国产操作系统和软件，国产替代进程正在如火如荼地进行。目前，信创软硬件百花齐放，就拿国产芯片来说，有麒麟、兆芯、海光、龙芯、飞腾、申威等等，作为信创产业链的一环，软件开发者也面临着一个问题，需要适配各种平台。

前几天有位客户询问我们的软件有没有飞腾架构统信 UOS 下的版本。在这之前，我们做了 Windows、兆芯（x86）UOS、麒麟等几个平台的适配，但没有做过飞腾平台。查询了一下飞腾芯片的架构，FT-D2000 兼容64位ARMv8指令集。看到是 ARM64 架构的，心里有底了。之前做的麒麟 OS ARM 版本的移植，理论上可以直接运行在统信 UOS 的飞腾版本上。即使有些细微差别，做一些适配就可以了。

但是，我们面临一个问题，手头并没有飞腾处理器的机器，在没有明确客户需求的情况下，也不可能去购置一台机器。

因为目前只是做一个验证，看看之前的软件包能否在机器上跑起来，基本功能是否没问题，所以自然想到虚拟机方案。但 VirutalBox 等虚拟机并不支持在 x86 架构的机器上创建 ARM 虚拟机，所以我将目标投向了 QEMU。

## QEMU

QEMU 是一个强大的开源仿真器和虚拟化器，它可以在不同的硬件架构上运行虚拟机。它支持多种硬件架构的仿真，包括 ARM、x86、MIPS、RISC-V 等，可以完全仿真目标系统的 CPU、内存、网络、I/O 设备等。对于 ARM 仿真，QEMU 提供了多种 ARM 处理器和开发板模型，能够运行常见的 ARM 操作系统，如 Linux、Android 等。

在 UOS V20 上安装 QEMU 非常简单，只需执行下面的命令：

```
$ sudo apt install qemu-system qemu-efi-aarch64
```

安装完毕后，使用如下命令，可以查看 QEMU 模拟器的版本。

```
$ qemu-system-aarch64 --version
QEMU emulator version 3.1.0 (Debian 1:3.1+dfsg.1-1+dde)
Copyright (c) 2003-2018 Fabrice Bellard and the QEMU Project developers
```

## 创建模拟器

QEMU 需要一个虚拟硬盘来安装操作系统，所以使用 qemu-img 工具创建一个虚拟硬盘：

```
$ qemu-img create -f qcow2 uos-arm64-disk.qcow2 100G
Formatting 'uos-arm64-disk.qcow2', fmt=qcow2 size=107374182400 cluster_size=65536 lazy_refcounts=off refcount_bits=16
```

这里的 100G 是虚拟硬盘的大小，可以根据需要调整。

QEMU 可以模拟 ARM64 的 UEFI 启动，需要一个 UEFI 镜像，这个在上面安装 QEMU 时已经安装，文件位于 /usr/share/qemu-efi-aarch64/QEMU_EFI.fd。

先检查一下这个文件是否存在，如果不存在，可以用如下命令安装：

```
$ sudo apt install qemu-efi-aarch64
```

接下来下载 UOS 的安装镜像，请前往如下地址：

> https://www.chinauos.com/resource/download-professional

UOS 的版本比较多，一定下载正确的版本，我们选择飞腾适配版本：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/uos_qemu_arm_01.png)

下载完毕后，最好校验一下 md5 值，避免由于下载文件不完整造成安装失败。

```
$ md5sum uos-desktop-20-professional-1070-arm64-202408.iso 
8757bd794ec7f7d6af13ed4053c2c92a  uos-desktop-20-professional-1070-arm64-202408.iso
```

## 安装 UOS V20 飞腾版

前面的准备工作做好之后，接下来就是在模拟器上安装 UOS V20 版本。简化期间，下载的 UOS 系统镜像文件和虚拟硬盘文件都放在当前目录下。

执行如下命令：

```
qemu-system-aarch64 -M virt -cpu cortex-a72 -smp 8 -m 8096 \
    -bios /usr/share/qemu-efi-aarch64/QEMU_EFI.fd \
    -device virtio-gpu-pci,xres=1920,yres=1080 \
    -display gtk \
    -device usb-ehci -device usb-tablet -device usb-kbd \
    -drive if=none,file=uos-arm64-disk.qcow2,id=hd0 \
    -device virtio-blk-device,drive=hd0 \
    -cdrom uos-desktop-20-professional-1070-arm64-202408.iso \
    -netdev user,id=net0 -device virtio-net-device,netdev=net0
```

qemu 命令行参数比较多，这里简单说明一下各个参数的含义：

* -M virt：指定 ARM 的虚拟机类型。
* -cpu cortex-a72：指定虚拟 CPU 类型。
* -smp 8：设置虚拟机 CPU 核心数为 8 个。
* -m 8096：为虚拟机分配 8GB 内存，可根据需要调整。
* -bios /usr/share/qemu-efi-aarch64/QEMU_EFI.fd：UEFI 镜像文件
* -device virtio-gpu-pci,xres=1920,yres=1080：启用 virtio 图形设备，并指定分辨率
* -device usb-ehci -device usb-tablet -device usb-kbd：为虚拟机配置鼠标、键盘支持。
* -drive if=none,file=uos-arm64-disk.qcow2,id=hd0：指定虚拟硬盘文件。
* -device virtio-blk-device,drive=hd0：使用 virtio 作为硬盘驱动，提升虚拟机性能。
* -cdrom uos-desktop-20-professional-1070-arm64-202408.iso：挂载 ISO 文件，虚拟光驱。
* -netdev user,id=net0 -device virtio-net-device,netdev=net0：为虚拟机配置网络支持。

这其中可能需要注意的，虚拟机支持多种显示设备虚拟，如 std, virtio, cirrus, 或 qxl。使用 std 最保险，但可能无法支持较高的分辨率。virtio、cirrus 等并不是在所有机器上都能比较好的虚拟，可能需要根据实际运行情况，调整一下，多尝试几回。

如果启动成功，将出现第一个安装界面：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/uos_qemu_arm_02.png)

敲一下回车键，然后耐心等待（这个过程需要多长时间，取决于你主机的性能，我这边大约等了两三分钟），出现图形安装界面。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/uos_qemu_arm_03.png)

接下来的安装过程和在 PC 上安装 UOS 一样，一般情况下使用默认安装即可。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/uos_qemu_arm_04.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/uos_qemu_arm_05.png)

重新启动，然后又是一阵等待，终于出现 UOS 的界面：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/uos_qemu_arm_06.png)

查看一下系统信息：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/uos_qemu_arm_07.png)

进入控制面板查看系统信息：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/uos_qemu_arm_08.png)

至此，整个系统安装就完成了。后面如果要启动这个虚拟的 UOS 系统，修改上面的命令，去掉挂载光盘参数：

```
qemu-system-aarch64 -M virt -cpu cortex-a72 -smp 8 -m 8096 \
    -bios /usr/share/qemu-efi-aarch64/QEMU_EFI.fd \
    -device virtio-gpu-pci,xres=1920,yres=1080 \
    -display gtk \
    -device usb-ehci -device usb-tablet -device usb-kbd \
    -drive if=none,file=uos-arm64-disk.qcow2,id=hd0 \
    -device virtio-blk-device,drive=hd0 \
    -netdev user,id=net0 -device virtio-net-device,netdev=net0
```

更好的方式是将这个命令放在一个脚本文件中，以后每次运行脚本来启动虚拟系统。

## 小结

使用 QEMU 来装飞腾版的 UOS 系统，实属迫不得已，再实际使用过程中发现，使用模拟器运行飞腾版 UOS，延迟非常严重，整个过程相当不流畅。这主要是因为在 x86 上完全使用软件仿真 ARM 指令，效率非常低。这里没法像 x86 下的虚拟机那样，使用硬件虚拟化。所以模拟器方式只适合做一些简单的测试。

这里将安装过程记录下来，供大家参考，也许某些情况下可以应一下急。如果大家还有什么更好的方法，欢迎留言讨论。

