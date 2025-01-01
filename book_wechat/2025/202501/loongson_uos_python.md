# 龙芯UOS系统升级Python

Python 是目前最流行的编程语言之一，特别是进入 AI 时代，Python 语言是当之无愧的老大。作为一名 C/C++ 程序员，在工作中也难免用到 Python。比如 Chromium 开源项目中，大量使用了 Python 脚本。

一般 Linux 系统都预装了 Python 环境，比如 UOS V20 版本默认的 python 为 python 2.7.16。

```
$ python --version
Python 2.7.16
```

## Python 2 和 Python 3 切换

Python 2 已经不再维护，Python 3 也老早就计划替代 Python 2，但软件升级就是这么难。因为Python 2 到 Python 3 不兼容，导致 Python 2 顽固的没有退出市场，这种状况估计还会存在很多年。现在的情况就是新的 Python 代码运行需要 python3 版本，但有些老代码仍然还需要 python 2，所以需要一种方便的方法来切换。其实系统的 python 只是一个链接，当前指向的是 python2。

```
$ ls -la /usr/bin/python
lrwxrwxrwx 1 root root 7 3月   4  2019 /usr/bin/python -> python2
```

一个简单的方法，将 python 链接指向 python3 即可将系统 python 版本默认修改为 python 3，但考虑到以后方便切换到 python2，可以借助  update-alternatives 命令：

```
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 2
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1

$ sudo update-alternatives --config python
有 2 个候选项可用于替换 python (提供 /usr/bin/python)。

  选择       路径            优先级  状态
------------------------------------------------------------
* 0            /usr/bin/python2   2         自动模式
  1            /usr/bin/python2   2         手动模式
  2            /usr/bin/python3   1         手动模式

要维持当前值[*]请按<回车键>，或者键入选择的编号：2
update-alternatives: 使用 /usr/bin/python3 来在手动模式中提供 /usr/bin/python (python)
```

再查看 Python 版本，可以看出，已经修改过来了：

```
$ python --version
Python 3.7.3
```

## 升级 Python 3

UOS V20 仓库中最新版本的 Python 是 3.7.3 版本，但是 build 最新的 chromium 要求 python 3.8 以上版本，所以需要升级 python。Python 没有龙芯架构的发布包，所以需要从源码编译。

首先，安装编译 Python 所需要的软件，因为之前已经安装过 gcc/g++ 工具链，这里不需重复安装，再安装一些软件库即可：

```
# 安装 cursor 库
$ sudo apt install libncurses5-dev libncursesw5-dev zlib1g-dev
```

接下来，下载 Python 源码，我这里选择了 3.8.19 版本的源码包。

```
$ wget https://www.python.org/ftp/python/3.8.19/Python-3.8.19.tgz
```

最后，解压，并编译：

```
$ tar xvf Python-3.8.19.tgz
$ cd Python-3.8.19/
$ ./configure --enable-optimizations
$ make -j4
$ sudo make altinstall
```

使用  make altinstall 而不是 make install ，以避免覆盖系统中已有的 python 命令。它会创建一个类似 python3.8 的命令，而不是 python。

在龙芯架构下执行 ./configure 指令，会出现如下错误：

```
$ ./configure --enable-optimizations
checking build system type... ./config.guess: unable to guess system type

This script (version 2018-03-08), has failed to recognize the
operating system you are using. If your script is old, overwrite *all*
copies of config.guess and config.sub with the latest versions from:

  https://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess
and
  https://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub

If ./config.guess has already been updated, send the following data and any
information you think might be pertinent to config-patches@gnu.org to
provide the necessary information to handle your system.

config.guess timestamp = 2018-03-08

uname -m = loongarch64
uname -r = 4.19.0-loongson-3-desktop
uname -s = Linux
uname -v = #7206 SMP Thu Nov 28 14:17:24 CST 2024

/usr/bin/uname -p = unknown
/bin/uname -X     = 

hostinfo               = 
/bin/universe          = 
/usr/bin/arch -k       = 
/bin/arch              = loongarch64
/usr/bin/oslevel       = 
/usr/convex/getsysinfo = 

UNAME_MACHINE = "loongarch64"
UNAME_RELEASE = "4.19.0-loongson-3-desktop"
UNAME_SYSTEM  = "Linux"
UNAME_VERSION = "#7206 SMP Thu Nov 28 14:17:24 CST 2024"
configure: error: cannot guess build type; you must specify one
```

这是由于较早的 autotools 不支持龙芯架构，最新的工具已经支持，解决方法是下载并更新最新版本的 `config.sub` 和 `config.guess`：

```
$ sudo wget -O /usr/share/misc/config.sub "git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD"
$ sudo wget -O /usr/share/misc/config.guess "git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD"
```

然后在 python 源码目录下执行如下命令更新当前目录下的脚本：

```
$ libtoolize -f -i -c
```

再执行如下指令，就可以完成 python 源码的配置、编译和安装：

```
$ ./configure --enable-optimizations
$ make -j4
$ sudo make altinstall
```

## Python 3 版本的切换

现在系统上有两个版本的 Python 3：3.7.3 和 3.8.19。同样是 Python 3，也可能有一定的兼容问题，我们也可以仿照前面的 Python 切换，自由的切换两个 Python 3 版本。

```
$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 0
$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.8 1
```

切换 Python 3 版本的命令：

```
sudo update-alternatives --config python3
```

你会看到一个版本列表，输入对应的编号来选择 Python 3 版本。

确认 python 3 版本：

```
$ python3 --version
Python 3.8.19
```

## 小结

本文详细介绍了在 UOS V20 系统下，如何实现 Python 2 与 Python 3 的切换、从源码编译更高版本的 Python 3，以及在多个 Python 3 版本之间切换的方法。希望对你有所帮助！
