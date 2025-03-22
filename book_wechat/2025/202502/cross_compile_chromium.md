# 国产系统上交叉编译 ARM 架构浏览器

随着国产信创系统的逐步发展，越来越多的设备采用了非 x86 架构的 CPU，如 ARM、龙芯、申威和 RISC-V 等。其中，ARM 架构的市场份额最高，主要厂商包括飞腾和华为麒麟。在为这些国产系统开发软件时，通常需要支持上述架构。

在之前的开发中，我们一般选择直接在 ARM 的机器上编译和调试代码，这种方式虽然简单，但对于大型应用程序，尤其是像浏览器这样的复杂系统来说，却面临着不少挑战。以 Chromium 浏览器为例，其代码庞大且复杂，构建时需要大量的计算资源和内存。在搭载 ARM 架构的设备上进行编译时，尤其是在处理器性能相对较弱、内存较小的机器上，可能会遇到编译过程长时间卡顿、内存占用过高等问题。比如，目前不少搭载飞腾处理器的机器通常只有 8GB 内存，编译速度缓慢，甚至可能因为内存不足而导致编译失败。

在嵌入式开发中，一种常见的开发模式是交叉编译，通过在性能更强的机器上进行编译，再将编译好的应用移植到目标 ARM 设备上进行测试和优化。我们也可以采用这种方式，在 x86 架构的开发机上编译 arm 版 Chromium 浏览器。

本文将介绍如何在国产系统上交叉编译 ARM 架构的浏览器。

## 系统要求

- 一台 x86-64 架构的机器，至少配备 8GB 内存（推荐 16GB 以上）。若使用 SSD，建议为 8GB/16GB 内存的机器分别分配 ≥32GB/≥16GB 的交换空间。  
- 至少需要100GB的可用磁盘空间，不强制要求在同一驱动器；建议在 HDD 上分配约 50-80GB 用于构建。  
- 必须已安装 git 和 Python v3.8+（且 `python3` 需指向该版本的可执行文件）。若系统中无合适版本，`depot_tools`会在 `$depot_tools/python-bin` 中捆绑适配版本。  
- 唯一支持的STL为`libc++`，官方推荐编译器为`clang`。

本文选择 deepin V23 作为操作系统，其他操作系统如 Open Kylin、Ubuntu 24.04 也适用。

## 下载 depot_tools

Chromium 使用了自定义的代码管理和构建系统 depot_tools，因此我们首先需要下载它：

1. 克隆 depot_tools 仓库：

```
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

将 depot_tools 添加到 PATH 环境变量的最前端​（建议将其写入 ~/.bashrc 中。假设 depot_tools 克隆到 /path/to/depot_tools）：

```
$ export PATH="/path/to/depot_tools:$PATH"
```

**注意事项**：若 depot_tools 克隆到用户主目录（如 /home/username/depot_tools），请勿在 PATH 中使用 ~ 符号（这会导致 gclient runhooks 命令失败）。应改用 $HOME 或绝对路径：

```
$ export PATH="$HOME/depot_tools:$PATH"
```

## 下载指定版本 chromium 源码

由于 Chromium 的 git 库极其庞大，如果整个 git 库全部 clone 下来，经常会遇到中途中断的问题。由于 git 没有断点续传的功能，几十 G 的 git 库，下载了几个小时后中断，需要重新下，令人崩溃。通常情况下，我们只会基于某个 release 版本做开发，所以通常情况下我们只用 git clone 指定分支的源码，也不用带完整的 git 历史记录。如果需要翻 chromium 的历史记录，可以在线查找，网址为：

> https://source.chromium.org/chromium/chromium/

所以这里下载指定版本（127.0.6533.100）的 Chromium 源码，且不带历史记录。这样不仅可以避免中途中断，还可以大幅减少下载代码的时间。

1. 下载 127.0.6533.100 版本源码（不带git log）

```
$ git clone --depth 1 --branch 127.0.6533.100 https://chromium.googlesource.com/chromium/src.git
```

2. 创建分支

```
$ cd src
$ git branch branch-127
```

3. 新建 .gclient 文件，注意 和 src 同一级目录

```
solutions = [
  {
    "name": "src",
    "url": "https://chromium.googlesource.com/chromium/src.git",
    "managed": False,
    "custom_deps": {},
    "custom_vars": {},
  },
]

target_cpu="arm64"
```

4. sync 和 chromium 相关联的模块和第三方库源码

```
$ gclient sync --nohooks --no-history
```

5. sync 相关工具或二进制文件

```
$ gclient runhooks
```

理论上这一步会下载 arm64 架构的 sysroot，可以检查 build/linux/debian_bullseye_arm64-sysroot 目录是否存在，如果不存在，可以手动下载：

```
./build/linux/sysroot_scripts/install-sysroot.py --arch=arm64
```

## 安装构建依赖的系统库

chromium 提供了一个脚本 ./build/install-build-deps.sh 下载依赖的系统库，但在 deepin v23 下执行，会出现如下错误：

```
WARNING: The following distributions are supported,
but distributions not in the list below can also try to install
dependencies by passing the `--unsupported` parameter.
EoS refers to end of standard support and does not include
extended security support.
        Ubuntu 20.04 LTS (focal with EoS April 2025)
        Ubuntu 22.04 LTS (jammy with EoS June 2027)
        Ubuntu 24.04 LTS (noble with EoS June 2029)
        Debian 10 (buster) or later
```

这是由于编写这个脚本的开发人员没有考虑到其它 Linux 发行版本，我们可以加上 --unsupported 参数。加上这个参数运行后，还会出现如下错误：

```
Running as non-root user.
You might have to enter your password one or more times for 'sudo'.

请输入密码:
验证成功
命中:1 https://app-store-files.uniontech.com/250228174341123/appstorev23 beige InRelease        
忽略:2 https://community-packages.deepin.com/beige beige InRelease                              
命中:3 https://cdn-community-packages.deepin.com/driver-23 driver InRelease
命中:4 https://community-packages.deepin.com/beige beige Release
正在读取软件包列表... 完成
Finding missing packages...
Building apt package list.
Traceback (most recent call last):
  File "/work/browser/deepin-browser/branch-127/build/install-build-deps.py", line 939, in <module>
    sys.exit(main())
  File "/work/browser/deepin-browser/branch-127/build/install-build-deps.py", line 932, in main
    install_packages(options)
  File "/work/browser/deepin-browser/branch-127/build/install-build-deps.py", line 844, in install_packages
    packages = find_missing_packages(options)
  File "/work/browser/deepin-browser/branch-127/build/install-build-deps.py", line 818, in find_missing_packages
    packages = package_list(options)
  File "/work/browser/deepin-browser/branch-127/build/install-build-deps.py", line 757, in package_list
    packages = (dev_list() + lib_list() + dbg_list(options) +
  File "/work/browser/deepin-browser/branch-127/build/install-build-deps.py", line 272, in dev_list
    if package_exists("realpath"):
  File "/work/browser/deepin-browser/branch-127/build/install-build-deps.py", line 32, in package_exists
    return package_name in build_apt_package_list()
  File "/work/browser/deepin-browser/branch-127/build/install-build-deps.py", line 23, in build_apt_package_list
    output = subprocess.check_output(["apt-cache", "dumpavail"]).decode()
UnicodeDecodeError: 'utf-8' codec can't decode byte 0x92 in position 61529773: invalid start byte
```

这个错误修改起来还比较麻烦，一个比较直接的方法是直接安装构建所需的依赖包，在 deepin v23 下，可以执行如下命令：
```
sudo apt install -y \
  debhelper \
  devscripts \
  lld-16 \
  clang-16 \
  clang-format-16 \
  libclang-rt-16-dev \
  libc++-16-dev \
  libc++abi-16-dev \
  rustc-web \
  bindgen \
  python3 \
  pkg-config \
  ninja-build \
  python3-jinja2 \
  python3-pkg-resources \
  ca-certificates \
  elfutils \
  wget \
  flex \
  yasm \
  xvfb \
  wdiff \
  gperf \
  bison \
  nodejs \
  rollup \
  valgrind \
  xz-utils \
  x11-apps \
  xcb-proto \
  xfonts-base \
  libdav1d-dev \
  libx11-xcb-dev \
  libxshmfence-dev \
  libgl-dev \
  libglu1-mesa-dev \
  libegl1-mesa-dev \
  libgles2-mesa-dev \
  libopenh264-dev \
  generate-ninja \
  mesa-common-dev \
  rapidjson-dev \
  libva-dev \
  libxt-dev \
  libgbm-dev \
  libpng-dev \
  libxss-dev \
  libelf-dev \
  libpci-dev \
  libcap-dev \
  libdrm-dev \
  libffi-dev \
  libhwy-dev \
  libkrb5-dev \
  libexif-dev \
  libflac-dev \
  libudev-dev \
  libpipewire-0.3-dev \
  libopus-dev \
  libxtst-dev \
  libjpeg-dev \
  libxml2-dev \
  libgtk-3-dev \
  libxslt1-dev \
  liblcms2-dev \
  libpulse-dev \
  libpam0g-dev \
  libdouble-conversion-dev \
  libxnvctrl-dev \
  libglib2.0-dev \
  libasound2-dev \
  libsecret-1-dev \
  libspeechd-dev \
  libminizip-dev \
  libhunspell-dev \
  libharfbuzz-dev \
  libxcb-dri3-dev \
  libusb-1.0-0-dev \
  libopenjp2-7-dev \
  libmodpbase64-dev \
  libnss3-dev \
  libnspr4-dev \
  libcups2-dev \
  libevent-dev \
  libevdev-dev \
  libgcrypt20-dev \
  libcurl4-openssl-dev \
  libzstd-dev \
  fonts-ipafont-gothic \
  fonts-ipafont-mincho
```

## 构建 arm64 版 chromium

为 arm64 架构交叉编译 chromium，需要给 gn 传递编译参数:

```
gn gen out/Default-arm64 --args="target_cpu = \"arm64\""
```

接下来就是编译 chromium 了，执行如下命令：
```
autoninja -C out/Default-arm64 chrome
```

这个编译时间有点长，喝杯茶、听听音乐，慢慢等吧。


编译完成后，在 `out/Default-arm64` 目录下会生成一个 `chrome` 可执行文件，这就是我们需要的 arm64 版 chromium。

将 `chrome` 可执行文件及相关资源文件拷贝到目标设备上，可以用一个脚本完成拷贝工作：

```
cp -a ${build_dir}/chrome ${TARGET}
cp -a ${build_dir}/chrome_sandbox ${TARGET}
cp -a ${build_dir}/chrome_100_percent.pak ${TARGET}
cp -a ${build_dir}/chrome_200_percent.pak ${TARGET}
cp -a ${build_dir}/chrome_crashpad_handler ${TARGET}
cp -a ${build_dir}/icudtl.dat ${TARGET}
cp -a ${build_dir}/libEGL.so ${TARGET}
cp -a ${build_dir}/libGLESv2.so ${TARGET}
cp -a ${build_dir}/libvk_swiftshader.so ${TARGET}
cp -a ${build_dir}/libvulkan.so.1 ${TARGET}
cp -r ${build_dir}/locales/ ${TARGET}
cp -a ${build_dir}/resources.pak ${TARGET}
cp -a ${build_dir}/v8_context_snapshot.bin ${TARGET}
cp -a ${build_dir}/vk_swiftshader_icd.json ${TARGET}
```

然后就可以在目标设备上运行 arm64 版 chromium 了。

## 总结

通过本文的步骤，你可以成功在国产系统上交叉编译 ARM 架构的浏览器，并将其部署到目标设备上进行使用。