# 国产系统上交叉编译 ARM 架构浏览器

国产信创系统中，越来越多的采用了非 x86 架构的 CPU，比如 ARM、龙芯、申威和 RISC-V。这其中，非 x86 架构的 CPU，ARM 架构份额最高，主要厂商有飞腾和华为麒麟。

## 系统要求

- 一台 x86-64 架构的机器，至少配备 8GB 内存（推荐 16GB 以上）。若使用 SSD，建议为 8GB/16GB 内存的机器分别分配 ≥32GB/≥16GB 的交换空间。  
- 至少需要100GB的可用磁盘空间，不强制要求在同一驱动器；建议在 HDD 上分配约 50-80GB 用于构建。  
- 必须已安装 git 和 Python v3.8+（且 `python3` 需指向该版本的可执行文件）。若系统中无合适版本，`depot_tools`会在 `$depot_tools/python-bin` 中捆绑适配版本。  
- 唯一支持的STL为`libc++`，官方推荐编译器为`clang`。

操作系统我选择的是 deepin V23，其它操作系统，比如 Open Kylin、Ubuntu 24.04 同样适用。

## 下载 depot_tools

chromium 采用了自己的一套代码管理系统和构建系统 depot_tools，所以首先需要下载 depot_tools
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

这是由于编写这个脚本的开发人员没有考虑到其它 Linux 发行版本，我们可以加上 --unsupported 参数。

## 构建 arm 版 chromium

新版本