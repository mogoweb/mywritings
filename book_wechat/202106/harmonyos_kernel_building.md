# 鸿蒙系统研究之二：内核编译

一个操作系统，最重要的部分无疑是内核。鸿蒙系统声称自研了内核，从之前开源的 OpenHarmony OS 代码中可以看到，是一款名为 LiteOS 的面向 IoT 领域构建的轻量级物联网操作系统。LiteOS 又有两个版本：LiteOS-A 和 LiteOS-M。而 OpenHarmony OS 2.0 针对手机、平板等富资源设备，则使用的是 Linux 操作系统。

在 OpenHarmony OS 2.0 源码的 kernel 目录，有四个文件夹，分别是 liteos_m、liteos_a、linux-4.19 和 linux，前三个是三种内核的源码，而 linux 目录则存放的是编译脚本和内核配置。从中也可以看出，鸿蒙系统选用的是 linux 4.19.y LTS 版本。

kernel/linux/patches/kernel_module_build.sh 文件是编译内核的脚本，其中关键的一句：

```
make -f kernel.mk
```

再来看看同一目录下的 kernel.mk 文件：

```
PRODUCT_NAME=$(TARGET_PRODUCT)

ifeq ($(PRODUCT_NAME), Hi3516DV300)
    OHOS_BUILD_HOME := $(OHOS_ROOT_PATH)
    BOOT_IMAGE_PATH = $(OHOS_BUILD_HOME)/device/hisilicon/hispark_taurus/prebuilts
endif

KERNEL_SRC_PATH := $(OHOS_BUILD_HOME)/kernel/linux-4.19
KERNEL_PATCH_PATH := $(OHOS_BUILD_HOME)/kernel/linux/patches/linux-4.19

KERNEL_SRC_TMP_PATH := $(OUT_DIR)/kernel/src_tmp/linux-4.19


PREBUILTS_GCC_DIR := $(OHOS_BUILD_HOME)/prebuilts/gcc

PREBUILTS_CLANG_DIR := $(OHOS_BUILD_HOME)/prebuilts/clang
CLANG_HOST_TOOLCHAIN := $(PREBUILTS_CLANG_DIR)/host/linux-x86/clang-r353983c/bin


CLANG_CC := $(CLANG_HOST_TOOLCHAIN)/clang

KERNEL_HOSTCC := $(CLANG_HOST_TOOLCHAIN)/clang


KERNEL_PREBUILT_MAKE := make

ifeq ($(PRODUCT_NAME), Hi3516DV300)
    KERNEL_ARCH := arm
    KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/arm/gcc-linaro-7.5.0-arm-linux-gnueabi/bin
    KERNEL_TARGET_TOOLCHAIN_PREFIX := $(KERNEL_TARGET_TOOLCHAIN)/arm-linux-gnueabi-
endif

KERNEL_PERL := /usr/bin/perl

KERNEL_CROSS_COMPILE :=
KERNEL_CROSS_COMPILE += CC="$(CLANG_CC)"
KERNEL_CROSS_COMPILE += HOSTCC="$(KERNEL_HOSTCC)"
KERNEL_CROSS_COMPILE += PERL=$(KERNEL_PERL)
KERNEL_CROSS_COMPILE += CROSS_COMPILE="$(KERNEL_TARGET_TOOLCHAIN_PREFIX)"

KERNEL_MAKE := \
    PATH="$(BOOT_IMAGE_PATH):$$PATH" \
    $(KERNEL_PREBUILT_MAKE)

ifeq ($(PRODUCT_NAME), Hi3516DV300)
HI3516DV300_PATCH_FILE := $(OHOS_BUILD_HOME)/device/hisilicon/hi3516dv300/sdk_linux/open_source/linux/hisi_linux-4.19_hos_l2.patch
KERNEL_IMAGE_FILE := $(KERNEL_SRC_TMP_PATH)/arch/arm/boot/uImage

$(KERNEL_IMAGE_FILE):
	$(hide) echo "build kernel..."
	$(hide) rm -rf $(KERNEL_SRC_TMP_PATH);mkdir -p $(KERNEL_SRC_TMP_PATH);cp -arfL $(KERNEL_SRC_PATH)/. $(KERNEL_SRC_TMP_PATH)/
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(HI3516DV300_PATCH_FILE)
	$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) distclean
	$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) hi3516dv300_emmc_smp_hos_l2_defconfig
	$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) -j64 uImage

endif
.PHONY: build-kernel
build-kernel: $(KERNEL_IMAGE_FILE)
```

先指定编译工具链为 clang ，然后为 linux 4.19 内核打上针对具体设备的补丁，Hi3516DV300的补丁位于 device/hisilicon/hi3516dv300/sdk_linux/open_source/linux/hisi_linux-4.19_hos_l2.patch，内核的参数配置文件为 hi3516dv300_emmc_smp_hos_l2_defconfig，最后一句就是生成内核镜像 uImage。

看似很简单，但是如果你直接执行 kernel_module_build.sh 脚本，会出错，因为有些环境变量还未设置。那  kernel_module_build.sh 脚本又是如何调用到的呢？下面就从系统的构建脚本 build.sh 出发，捋一捋是如果调用到内核编译脚本的。

先看一看 build.sh 的关键部分：

```
# preloader
${PYTHON3} ${source_root_dir}/build/loader/preloader/preloader.py \
  --product-name ${product_name} \
  --source-root-dir ${source_root_dir} \
  --products-config-dir "productdefine/common/products" \
  --preloader-output-root-dir "out/build_configs"

source ${source_root_dir}/out/build_configs/${product_name}/preloader/build.prop

# call build
${source_root_dir}/build/build_scripts/build_${system_type}.sh \
  --product-name ${product_name} \
  --device-name ${device_name} \
  --target-os ${target_os} \
  --target-cpu ${target_cpu} \
  ${build_params}

```

preloader 脚本是根据产品名生成产品定义属性，具体过程先略过。Hi3516DV300 的 system_type 为 standard，所以接下来调用的是 build_standard.sh 脚本。

再来看看  build_standard.sh 脚本的关键部分：

```
source ${script_path}/build_common.sh

function main() {
  # build ohos
  do_make_ohos

  # 此处略去若干行

  # build images
  build/adapter/images/build_image.sh --device-name ${device_name} \
    --ohos-build-out-dir ${ohos_build_root_dir}/packages/phone
}

main
```

这里有两个步骤，首先是调用 do_make_ohos 编译整个系统，这个脚本函数定义在 build_common.sh，然后调用 build_image.sh 打包整个系统镜像。

还得看看 build_common.sh 脚本，才能弄清楚 do_make_ohos 做了什么事情。

```
function do_make_ohos() {
  local build_cmd="build/build_scripts/build_ohos.sh"
  build_cmd+=" device_type=${product_name} target_os=${target_os} target_cpu=${target_cpu}"
  
  # 此处略去若干行

  $build_cmd
}
```

build_common.sh 脚本又是调用的 build_ohos.sh 脚本，还得往下跟踪：

```
main()
{
    source ${BUILD_SCRIPT_DIR}/pre_process.sh
    pre_process "$@"

    source ${BUILD_SCRIPT_DIR}/make_main.sh
    do_make "$@"

    source ${BUILD_SCRIPT_DIR}/post_process.sh
    post_process "$@"
    exit $RET
}

main "$@"
```

OpenHarmonyOS 2.0 的编译分三步走，第一步调用 pre_process.sh 脚本进行一些预处理工作，第二步调用 make_main.sh 脚本做主要的系统构建工作，最后 post_process.sh 做一些编译后的善后工作。注意，到了这里，三个脚本位于 build/core/build_scripts 目录。

先略过第一步和第三步，主要来看看第二步  make_main.sh 脚本做了哪些工作：

```
do_make()
{
    # 此处略过若干行

    if [ "${SKIP_GN_PARSE}"x = falsex ]; then
        ${BUILD_TOOLS_DIR}/gn gen ${TARGET_OUT_DIR} \
            --args="target_os=\"${TARGET_OS}\" target_cpu=\"${TARGET_ARCH}\" is_debug=false \
            device_type=\"${DEVICE_TYPE}\" is_component_build=true use_custom_libcxx=true \
            ${GN_ARGS} ${TEST_BUILD_PARA_STRING}  ${IS_ASAN} \
            release_test_suite=${RELEASE_TEST_SUITE}" 2>&1 | tee -a $log

        # 此处略过若干行
    fi

    # 此处略过若干行

    if [ "${USE_NARUTO}"x = "truex" ];then
        ${BUILD_TOOLS_DIR}/naruto -d keepdepfile -p ${BASE_HOME}/.naruto_cache -C ${TARGET_OUT_DIR} ${real_build_target} ${NINJA_ARGS} 2>&1 | tee -a $log
    else
        ${BUILD_TOOLS_DIR}/ninja -d keepdepfile -C ${TARGET_OUT_DIR} ${real_build_target} ${NINJA_ARGS} 2>&1 | tee -a $log
    fi

    # 此处略过若干行
}
```

可以看出，主要做了两个工作：1. 调用 gn 生成 Ninja 编译脚本；2. 调用 ninja 构建系统。

在之前的文章 [聊一聊鸿蒙的构建系统]() 中，我们知道鸿蒙采用了 GN 构建系统。GN 构建系统的规则文件名通常为 BUILD.gn 和 *.gni 文件，通过依赖、包含等方式，将整个系统的规则文件组织成一个整体。我们还是从入口文件着手，入口文件为 build/core/gn/BUILD.gn：

```
# 此处略过若干行

# gn target defined

group("make_all") {
  deps = [
    ":make_inner_kits",
    ":packages",
  ]
}

group("packages") {
  deps = [ "//build/ohos/packages:make_packages" ]
}

group("make_inner_kits") {
  deps = [ "$root_build_dir/build_configs:inner_kits" ]
}

if (build_ohos_sdk) {
  group("build_ohos_sdk") {
    deps = [
      "//build/ohos/sdk:ohos_sdk",
    ]
  }
}
```

这里定义了若干构建目标，通过之前的脚本处理，最后确定构建目标是 packages ，该构建目标又依赖于 //build/ohos/packages:make_packages，对应的 GN 规则文件为 build/ohos/packages/BUILD.gn 。这个脚本有些复杂，先只用关注 make_packages 这个目标：

```
group("make_packages") {
  deps = []
  foreach(_platform, target_platform_list) {
    deps += [
      ":${_platform}_parts_list",
      ":${_platform}_install_modules",
      ":gen_required_modules_${_platform}",
    ]
  }
}
```

它的依赖目标又是动态生成的，这里先不展开具体依赖哪些目标，和 build/subsystem_config.json 这个 JSON 文件有关，有兴趣的同学可以自行去分析其中的处理过程。查看 build/subsystem_config.json 文件，可以看到 hisilicon_products 这个目标依赖，其 GN 规则文件位于 device/hisilicon/build/BUILD.gn。

```
device_type = "hi3516dv300"
group("products_group") {
  if (device_type == "hi3516dv300") {
    deps = [
      "//device/hisilicon/hi3516dv300:hi3516dv300_group",
    ]
  } else if (device_type == "hi3559av100") {
    deps = [
      "//device/hisilicon/hi3559av100:hi3559av100_group",
    ]
  }

  deps += [
    "//device/hisilicon/hardware:hardware_group",
    "//device/hisilicon/modules/middleware:middleware_group",
  ]
}
```

按图索骥，找到 device/hisilicon/hi3516dv300/BUILD.gn 文件：

```
group("hi3516dv300_group") {
  deps = [
    "sdk_linux/mpp:sdk_linux_mpp_group",
    "kernel:kernel",
    "kernel:all_modules",
    "build:rc_files",
    #"sdk_linux/osdrv:sdk_linux_osdrv_group"
  ]
}
```

这里面又是依赖 "kernel:kernel" 这个目标，继续追踪 device/hisilicon/hi3516dv300/kernel/BUILD.gn 文件：

```
action("kernel") {
  script = "build_kernel.sh"
  sources = [ kernel_source_dir ]

  outputs = [ "$root_build_dir/packages/phone/images/uImage" ]
  args = [
    rebase_path(kernel_build_script_dir, root_build_dir),
    rebase_path("$root_out_dir/../KERNEL_OBJ"),
    rebase_path("$root_build_dir/packages/phone/images"),
    device_type,
  ]
}
```

可以看到，这里调用了一个 build_kernel.sh 脚本，这个文件调用了 kernel_module_build.sh 脚本，终于回到前面的内核编译脚本，至此，整个内核编译的脚本调用过程就此结束。总结下来，整个链路为：

> build.sh -> build_standard.sh => build_common.sh -> build_ohos.sh -> make_main.sh -> GN -> build/core/gn/BUILD.gn -> build/ohos/packages/BUILD.gn -> device/hisilicon/build/BUILD.gn -> device/hisilicon/hi3516dv300/BUILD.gn -> device/hisilicon/hi3516dv300/kernel/BUILD.gn -> build_kernel.sh -> kernel_module_build.sh

华为把构建过程整的无比复杂，又是脚本，又是 python，然后掺杂着 JSON 文件，最后是 GN 构建系统，GN 构建系统又掺杂着脚本，看起来头大。不过如果要单独编译内核，也可以不通过脚本。官方文档给出了简单的步骤：

1.  准备工作

    准备编译环境，可以使用开源arm clang/gcc编译器，或者使用工程自带编译器。

    进入工程主目录配置环境变量：

    ```
    export TARGET_PRODUCT=Hi3516DV300 # HDF驱动需要
    export PATH=`pwd`/prebuilts/clang/host/linux-x86/clang-r353983c/bin:`pwd`/prebuilts/gcc/linux-x86/arm/gcc-linaro-7.5.0-arm-linux-gnueabi/bin/:$PATH # 配置编译环境
    MAKE_OPTIONES="ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- CC=clang HOSTCC=clang" # 使用工程项目自带的clang环境
    ```

2.  直接进入整编过的内核目录out/KERNEL_OBJ/kernel/src_tmp/linux-4.19下修改内核代码或config （OpenHarmony提供对应平台的defconfig供参考）。
3.  生成内核.config。

    ```
    make ${MAKE_OPTIONES} hi3516dv300_emmc_smp_hos_l2_defconfig # 使用自带的默认config 构建内核
    ```

4.  编译生成对应的内核Image。

    ```
    make ${MAKE_OPTIONES} -j32 uImage # 编译uImage内核镜像
    ```

当然，上面的对构建脚本的分析也不是没有用处，因为接下来要为使用 QEMU 模拟 vexpress-a9 硬件，为其编译内核，敬请关注。


