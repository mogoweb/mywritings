# 鸿蒙系统研究之三：迈出平台移植第一步

OpenHarmony OS 2.0 发布时，标准系统只支持 Hi3516DV300 一种硬件平台，而 Android、IOS 均提供了模拟器供开发人员使用。这也可以理解，毕竟华为长期以来都是设备供应商，专长是硬件，在软件开发方面缺少底蕴。鸿蒙应用开发提供了模拟器，但那是真机模拟器，需要接入到华为的开发平台才能使用。

既然 OpenHarmony OS 2.0 标准系统不支持模拟器，那我们就自己动手，丰衣足食吧！

在本文你将了解到：

1. 如何为 OpenHarmony OS 2.0 标准系统增加新的产品定义；
2. 如何将新的平台移植加入构建系统；
3. 为模拟器编译出 Linux 内核；

常用的模拟器软件有 QEMU，能够模拟多种硬件型号，还支持 ARM、ARM64、RISC-V、X86 等多种指令。现有的嵌入式 Linux 资料和书籍很多是以 Vexpress A9 为例，所以本次移植也以 Vexpress A9 作为目标移植。关于 QEMU 模拟的 VExpress A9 平台介绍如下：

> QEMU/VExpress A9 是 QEMU 模拟器针对 ARM VExpress-A9 FPGA 开发板进行软件模拟的指令级虚拟机。QEMU/VExpress 因为是软件仿真模式，可以配置成多种模式，例如单核 Cortex-A9、多核Cortex-A9、以及多核 Cortex-A15 等，同时也能够模拟出 VExpress FPGA 开发板上大多数的外设。

#### 为 OpenHarmony 增加产品定义

OpenHarmony 系统的 build.sh 编译脚本需要带一个产品名参数 product-name，这里将其命名为 vexpress-a9。

产品定义位于 productdefine/common 目录，在其子目录 products 下有一个 Hi3516DV300.json 文件，这个对应着 Hi3516DV300 型号。复制一份，命名为 vexpress-a9.json，编辑 vexpress-a9.json 文件，将其中的：

```
  "product_name": "Hi3516DV300",
  "product_device": "hi3516dv300",
  "hisilicon_products:hisilicon_products":{},
```

这三行修改为：

```
  "product_name": "vexpress-a9",
  "product_device": "vexpress-a9",
  "qemu_products:qemu_products":{},
```

其中的 product_device 是设备名，在 productdefine/common/device/ 同样需要一个 JSON 文件 vexpress-a9.json，其内容可以从 hi3516dv300.json 复制过来：

```
{
    "target_os": "ohos",
    "target_cpu": "arm"
}
```

#### 为 OpenHarmony 增加子系统定义

鸿蒙系统支持各种形态的设备，从 IoT 设备到手机，其硬件资源千差万别，功能需求也各不一样，所以系统要求可定制、可裁剪，系统被划分为各种模块和子系统。

子系统的定义为 build/subsystem_config.json 文件，可以看到，这里子系统名基本上和 productdefine/common/products/Hi3516DV300.json 一一对应。可以这样理解，这里定义一个比较全的子系统集合，各产品根据自己的需求，定义自己的子系统子集。在这里，我们为 QEMU 增加一个子系统项：

```
    "qemu_products": {
      "project": "hmf/qemu_products",
      "path": "device/qemu/build",
      "name": "qemu_products",
      "dir": "device/qemu"
    },
```

子系统包括名称、路径、子系统构建脚本路径。这里 qemu_products 的编译脚本路径为 device/qemu/build ，首先增加一个 ohos.build，这个文件一定要建立，否则构建脚本就不会执行这个子系统的构建，内容可参考 device/hisilicon/build/ohos.build 文件：

```
{
    "subsystem": "qemu_products",
    "parts": {
        "qemu_products": {
            "module_list": [
                "//device/qemu/build:products_group"
            ]
        }
    }
}
```

其中 module_list 指定依赖目标，所以还需要在该目录下增加一个 BUILD.gn 文件：

```
import("//build/ohos.gni")

group("products_group") {
  if (device_type == "vexpress-a9") {
    deps = [
      "//device/qemu/vexpress-a9:vexpress-a9_group",
    ]
  }
}
```

该构建文件又依赖于 device/qemu/vexpress-a9/ 下的构建目标 vexpress-a9_group。到这里，就进入了新平台移植的步骤。

新平台的移植包括很多内容，如内核编译、驱动开发、根文件系统、生成镜像等等，庞杂而且工作量都很大，所以这里先说一说内核编译。

#### 为 Vexpress A9 编译内核

关于嵌入式 Linux 内核编译，网上的资料很多，这里探讨的是如何在鸿蒙系统的构建系统中加入内核编译步骤。

参考 device/hisilicon/hi3516dv300 下的构建脚本，内核编译主要分三个步骤：

1. 为 Linux 4.19 内核打上针对 Hi3516DV300 产品的补丁。
2. 编译内核，生成内核镜像 uImage。
3. 打包 Hi3516DV300 的驱动。

针对 Vexpress A9，我们就不搞那么复杂，就在原始的 Linux 4.19 源码上编译内核镜像。

内核镜像分两种：zImage 和 uImage，其中 zImage 可以直接用 QEMU 加载，而 uImage 需要通过 u-boot 加载，我们先编译出 zImage。

1. 在 device/qemu/vexpress-a9/ 添加 BUILD.gn 文件：

```
import("//build/ohos.gni")

print("vexpress-a9_group in")
group("vexpress-a9_group") {
  deps = [
    "kernel:kernel",
  ]
}
```

2. 新建 device/qemu/vexpress-a9/kernel 目录，在该目录下增加 BUILD.gn、kernel.mk 和 build_kernel.sh 文件。

其中 BUILD.gn 为鸿蒙构建系统的规则定义文件：

```
import("//build/ohos.gni")

kernel_build_script_dir = "//device/qemu/vexpress-a9/kernel"
kernel_source_dir = "//kernel/linux-4.19"

action("kernel") {
  script = "build_kernel.sh"
  sources = [ kernel_source_dir ]

  outputs = [ "$root_build_dir/packages/phone/images/zImage" ]
  args = [
    rebase_path(kernel_build_script_dir, root_build_dir),
    rebase_path("$root_out_dir/../KERNEL_OBJ"),
    rebase_path("$root_build_dir/packages/phone/images"),
    device_type,
  ]
}
```

其中定义的构建目标 "kernel"，执行 build_kernel.sh 脚本：

```
echo "call build_kernel.sh"

pushd ${1}

export OHOS_ROOT_PATH=$(pwd)/../../../..
#note out_dir style:out/xx/
export OUT_DIR=$2

LINUX_KERNEL_OUT=${OUT_DIR}/kernel/src_tmp/linux-4.19
LINUX_KERNEL_ZIMAGE_FILE=$LINUX_KERNEL_OUT/arch/arm/boot/zImage

make -f kernel.mk

if [ -f ${LINUX_KERNEL_ZIMAGE_FILE} ];then
    echo "zImage: ${LINUX_KERNEL_UIMAGE_FILE} build success"
else
    echo "zImage build failed!!!"
    exit 1
fi

mkdir -p ${3}
cp ${2}/kernel/src_tmp/linux-4.19/arch/arm/boot/zImage ${3}/zImage
popd
```

在脚本中又使用到了 kernel.mk 文件：

```
PRODUCT_NAME=$(TARGET_PRODUCT)

OHOS_BUILD_HOME := $(OHOS_ROOT_PATH)

KERNEL_SRC_PATH := $(OHOS_BUILD_HOME)/kernel/linux-4.19

KERNEL_SRC_TMP_PATH := $(OUT_DIR)/kernel/src_tmp/linux-4.19


PREBUILTS_GCC_DIR := $(OHOS_BUILD_HOME)/prebuilts/gcc

PREBUILTS_CLANG_DIR := $(OHOS_BUILD_HOME)/prebuilts/clang
CLANG_HOST_TOOLCHAIN := $(PREBUILTS_CLANG_DIR)/host/linux-x86/clang-r353983c/bin


CLANG_CC := $(CLANG_HOST_TOOLCHAIN)/clang

KERNEL_HOSTCC := $(CLANG_HOST_TOOLCHAIN)/clang

KERNEL_PREBUILT_MAKE := make

KERNEL_ARCH := arm
KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/arm/gcc-linaro-7.5.0-arm-linux-gnueabi/bin
KERNEL_TARGET_TOOLCHAIN_PREFIX := $(KERNEL_TARGET_TOOLCHAIN)/arm-linux-gnueabi-

KERNEL_PERL := /usr/bin/perl

KERNEL_CROSS_COMPILE :=
KERNEL_CROSS_COMPILE += CC="$(CLANG_CC)"
KERNEL_CROSS_COMPILE += HOSTCC="$(KERNEL_HOSTCC)"
KERNEL_CROSS_COMPILE += PERL=$(KERNEL_PERL)
KERNEL_CROSS_COMPILE += CROSS_COMPILE="$(KERNEL_TARGET_TOOLCHAIN_PREFIX)"

KERNEL_MAKE := \
    PATH="$(BOOT_IMAGE_PATH):$$PATH" \
    $(KERNEL_PREBUILT_MAKE)

KERNEL_IMAGE_FILE := $(KERNEL_SRC_TMP_PATH)/arch/arm/boot/zImage

$(KERNEL_IMAGE_FILE):
	echo "build kernel..."
	rm -rf $(KERNEL_SRC_TMP_PATH);mkdir -p $(KERNEL_SRC_TMP_PATH);cp -arfL $(KERNEL_SRC_PATH)/. $(KERNEL_SRC_TMP_PATH)/
	$(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) distclean
	$(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) vexpress_defconfig
	$(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) zImage
	$(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) dtbs
	$(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) modules

.PHONY: build-kernel
build-kernel: $(KERNEL_IMAGE_FILE)
```

在该 Makefile 中，指定交叉编译工具链，并进行内核编译，最后生成 zImage 镜像。

让我们来使用构建脚本编译系统：

```
./build.sh --product-name vexpress-a9 --ccache
```

然后使用 QEMU 模拟器来启动内核：

```
$ qemu-system-arm -M vexpress-a9 -m 512M -dtb ./out/KERNEL_OBJ/kernel/src_tmp/linux-4.19/arch/arm/boot/dts/vexpress-v2p-ca9.dts -kernel ./out/KERNEL_OBJ/kernel/src_tmp/linux-4.19/arch/arm/boot/zImage -nographic
```

结果如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202106/images/harmonyos_qemu_kernel_01.png)

可以看到，内核加载并启动，但缺少根文件系统。关于根文件系统的制作与加载，在后面再继续聊。

#### 小结

鸿蒙系统的构建系统还是比较复杂的，交织着 bash 脚本、python 脚本、GN 构建系统、make 构建系统、JSON 文件等等，有些文件还是编译过程生成出来的，理解起来相当困难。通过本篇文章，我们可以了解到，移植鸿蒙标准系统的步骤有：

1. 编写产品定义 JSON 文件
2. 编写子系统定义 JSON 文件
3. 为设备增加构建脚本，通常位于 device/<manufactory>/<device_type> 下，包括生成内核镜像、驱动、系统镜像、用户镜像等等

针对 OpenHarmony 2.0 系统源码的修改，我在 gitee 上也 fork 几个 OpenHarmony 2.0 源码库，上述修改均可以在我 fork 的源码库中找到，有兴趣的朋友可以访问：

> https://gitee.com/mogoweb

后续将继续分析鸿蒙系统的移植，敬请关注！

