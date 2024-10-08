# 在 UOS 下利用 QEMU 搭建飞腾 ARM64 的开发环境

近年来，在政府的推动下，国产操作系统（主要是统信 UOS 和麒麟 OS）以及相关软件的市场份额不断扩大。越来越多的企业和事业单位开始采用国产操作系统和软件，国产化替代进程正如火如荼地进行。目前，信创产业链上下游百花齐放，国产芯片领域更是群雄并起，如麒麟、兆芯、海光、龙芯、飞腾、申威等。作为产业链中的一环，软件开发者也面临一个普遍问题：需要适配多种硬件平台。

前几天，一位客户询问我们是否提供飞腾架构统信 UOS 下的软件版本。我们之前已经适配了 Windows、兆芯（x86）UOS 和麒麟等平台，但尚未支持飞腾架构。经过查询得知，飞腾 FT-D2000 兼容 64 位 ARMv8 指令集。看到是 ARM64 架构时，我心中有了底，因为此前我们已经做过麒麟 OS ARM 版本的移植，理论上咱们的软件可以直接在飞腾版本的统信 UOS 上运行。即使存在细微差异，经过简单适配也能解决。

然而，我们面临一个现实问题：手头没有搭载飞腾处理器的设备，而在需求尚不明确的情况下，采购一台新设备并不现实。

由于仅需验证软件包是否能够在该平台上运行，且主要关注基本功能的正常性，我们决定使用虚拟机方案。然而，像 VirtualBox 等常见虚拟机并不支持在 x86 架构的设备上创建 ARM 虚拟机，所以我们将目光投向了 QEMU。

## QEMU 介绍

QEMU 是一个功能强大的开源仿真器和虚拟化工具，能够在不同硬件架构上运行虚拟机。它支持多种硬件架构的仿真，包括 ARM、x86、MIPS 和 RISC-V 等，能够完全仿真目标系统的 CPU、内存、网络和 I/O 设备等。对于 ARM 仿真，QEMU 提供了多种 ARM 处理器和开发板模型，可以运行常见的 ARM 操作系统，如 Linux、Android 等。

在 UOS V20 系统上安装 QEMU 非常简单，只需执行以下命令：

```bash
$ sudo apt install qemu-system qemu-efi-aarch64
```

安装完成后，可通过以下命令查看 QEMU 版本信息：

```bash
$ qemu-system-aarch64 --version
QEMU emulator version 3.1.0 (Debian 1:3.1+dfsg.1-1+dde)
```

## 创建虚拟机

QEMU 需要一个虚拟硬盘来安装操作系统。使用 `qemu-img` 工具创建虚拟硬盘：

```bash
$ qemu-img create -f qcow2 uos-arm64-disk.qcow2 100G
```

这里的 100G 是虚拟硬盘的大小，可根据实际需求调整。

QEMU 支持 ARM64 的 UEFI 启动，相关的 UEFI 镜像文件会在安装 QEMU 时自动安装，默认路径为 `/usr/share/qemu-efi-aarch64/QEMU_EFI.fd`。如文件缺失，可执行以下命令重新安装：

```bash
$ sudo apt install qemu-efi-aarch64
```

接下来，请从统信官网上下载 UOS 安装镜像，地址如下：

> https://www.chinauos.com/resource/download-professional

UOS 的版本比较多，一定下载正确的版本，我们选择飞腾适配版本：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/uos_qemu_arm_01.png)

下载完毕后，最好校验一下 md5 值，避免由于下载文件不完整造成安装失败。

```bash
$ md5sum uos-desktop-20-professional-1070-arm64-202408.iso
```

## 安装 UOS V20 飞腾版

准备工作完成后，即可在虚拟机中安装 UOS V20 飞腾版。将下载的系统镜像和虚拟硬盘文件置于同一目录，执行以下命令启动虚拟机：

```bash
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

参数解释：
- `-M virt`：指定 ARM 虚拟机类型。
- `-cpu cortex-a72`：指定虚拟 CPU 类型。
- `-smp 8`：分配 8 个虚拟 CPU 核心。
- `-m 8096`：分配 8GB 内存。
- `-bios /usr/share/qemu-efi-aarch64/QEMU_EFI.fd`：使用 UEFI 镜像文件。
- `-device virtio-gpu-pci`：启用 virtio 图形设备，并设置分辨率。
- `-cdrom`：挂载 ISO 镜像文件。
- `-netdev user`：配置虚拟机网络支持。

虚拟机启动成功后，将显示第一个安装界面，按下回车键继续。图形界面安装过程与在 PC 上安装 UOS 基本一致，建议选择默认安装选项。

完成安装后，可通过以下命令启动已安装的系统：

```bash
qemu-system-aarch64 -M virt -cpu cortex-a72 -smp 8 -m 8096 \
    -bios /usr/share/qemu-efi-aarch64/QEMU_EFI.fd \
    -device virtio-gpu-pci,xres=1920,yres=1080 \
    -drive if=none,file=uos-arm64-disk.qcow2,id=hd0 \
    -netdev user,id=net0 -device virtio-net-device,netdev=net0
```

建议将该命令保存为脚本文件，便于日后快速启动。

## 小结

通过 QEMU 安装飞腾版 UOS 系统虽有一定的局限性，但在某些场景下，尤其是硬件资源有限的情况下，依然是一个不错的验证手段。然而，由于 x86 平台上 ARM 指令的纯软件仿真效率较低，运行过程中会出现明显的延迟。此方案更适合进行简单的测试和验证。

本文记录了安装过程，供有类似需求的开发者参考。如果有更好的方案，欢迎留言讨论。
