# 使用 RUST 开发 Android 应用

自 2019 年以来，谷歌一直在将用 Rust 编程语言编写的代码集成到其 Android 操作系统中。从 Android 12 开始，Rust 成为一种 Android 平台语言。而在 Android 13 中，大部分新代码都是用内存安全语言编写的： Rust、Java 或 Kotlin 。

当然，重写 Android 的几千万行代码不可行，Rust 主要用于新的开发而不是重写成熟的 C/C++ 代码。而且，RUST 语言主要用于底层开发，还没有迹象表明 RUST 能够取代 Java 或 Kotlin 的地位。但如果 Android 应用程序对性能要求比较高，可以使用 RUST 编写关键部分代码。

本文将介绍如何构建和部署 Rust 库，用于 Android 应用开发。

#### 一 环境设置

Android 应用程序的主流开发工具是 Android Studio，本文假定你已经安装好 Android Studio 。

接下来需要安装 NDK (Native Development Kit)，安装方法如下：

打开 Android Studio。转到 Android Studio > Preferences > Appearance & Behaviour > Android SDK > SDK Tools。勾选以下安装选项，然后单击确定。

> Android SDK Tools
> NDK
> CMake
> LLDB

如下图所示：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202302/images/rust_on_android_01.png)

安装好 NDK 和相关工具后，再设置两个环境变量，第一个用于 SDK 路径，第二个用于 NDK 路径，Android SDK 路径见上图，NDK 路径也取决于你使用的版本，请根据你的实际路径设置 ANDROID_SDK_HOME 和 NDK_HOME 环境变量：

```
export ANDROID_SDK_HOME=/data/android/android-sdk
export NDK_HOME=$ANDROID_SDK_HOME/ndk/25.2.9519653
```

#### 二 安装交叉编译工具链

既然进行 RUST 开发，本文假定你已经安装了 RUST 开发环境，所以直奔主题，创建 NDK 的独立版本，用于编译 so。借助 Android NDK 中的 make_standalone_toolchain.py 脚本即可做到。需要注意 Android 支持多种架构如 ARM、ARM64、x86等，RUST 开发的 so 不能像 Java 库那样跨架构，所以需要为每种架构编译对应的 so。为此，我们为每种架构都创建相应的工具链。

* 创建一个目录来存放独立的工具链：

```
mkdir /data/android/standalone-toolchain
cd /data/android/standalone-toolchain
mkdir NDK
${NDK_HOME}/build/tools/make_standalone_toolchain.py --api 26 --arch arm64 --install-dir NDK/arm64
${NDK_HOME}/build/tools/make_standalone_toolchain.py --api 26 --arch arm --install-dir NDK/arm
${NDK_HOME}/build/tools/make_standalone_toolchain.py --api 26 --arch x86 --install-dir NDK/x86
```

没必要做这一步，直接使用 ndk 下的 toolchan


$NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android26-clang++