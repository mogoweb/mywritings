# 使用国产操作系统作为开发系统

长期以来，我一直是在 Ubuntu 系统上做开发。近一年来，由于为信创系统（统信 UOS、银河麒麟等）开发应用软件，免不了使用国产操作系统。使用下来，发现国产系统在易用性、稳定性方面已经相当不错，而且用户界面比起 Ubuntu 还美观很多。系统集成的应用商店，里面的应用非常全面，基本上满足了作为系统开发的需求。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/using_deepin_01.png)

某一天，一个念头出现在我的脑海，何不使用国产操作系统作为开发机系统？说干就干。我立即上京东买了一块 500G 的固态硬盘，作为系统盘。至于数据盘，和原来的 Ubuntu 系统共用。之前在 Ubuntu 系统下下载了不少大模型和源码（Chromium、Android等），在新系统可以直接使用。在**统信 UOS** 和**银河麒麟**之间，我选择了统信 UOS，主要是统信 UOS 背后的社区（Deppin Community）更加活跃。

统信 UOS 有多个版本，我选择社区版（Deepin系统）。一般来说社区版本更新比较快，社区更加活跃，作为一名专业开发人员，有问题也方便去社区交流，碰到问题，相信有办法解决。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/using_deepin_02.png)

Deepin 是一个基于 Linux 的操作系统，早期基于 Debian 构建。从 2022 年开始，Deepin 脱离 Debian 社区，从 Linux kernel 和其他开源组件而构建，不依赖上游发行版社区，研发出全新架构的Deepin 23版本。当前 Deepin V23 还是 RC2 版本，离正式发布不远。本着旧不如新的原则，我选择了这个最新版本。

虽然 Deepin v23 不再基于 Debian 系统，但其依然沿用了 Debian 的那套包管理系统，所以从 Ubuntu 切换过来，使用体验上没有感觉差异，常用的软件安装、使用命令都大致相同。

Deepin 的安装过程相对简单，从官方网站下载 ISO 镜像文件，制作启动 U 盘进行安装，这里不赘述。

值得注意的是，在安装方式那一步，要选择高级安装，自己选择分区划分。如果选择推荐的全盘安装，Deepin 系统会自动进行分区，但是分给根分区的空间过小（只有 15G），这对于开发来说远远不够。有很多软件，特别是 deb 包，会将程序和库安装在 /usr 目录下，如果空间不足，会导致安装失败。我选择将 500 G 的空间，除了 32 G 的交换分区，其它全部挂载在根分区。两块数据盘，则挂载在 /data 和 /work 分区下。

## 安装 CUDA

Deepin 系统安装过程中会自动安装 Nvidia 驱动，一般情况下会安装开源的 Nvidia 驱动，如果勾选了安装专有的 Nvidia 驱动，也可能安装的是闭源的 Nvidia 驱动。如果是普通使用，使用系统安装的显卡驱动即可。但对于 AI 应用开发来说， CUDA  是必须安装的。

CUDA 是 **Compute Unified Device Architecture**（计算统一设备架构）的简称。它是由 NVIDIA 公司开发的一种并行计算平台和编程框架，允许软件工程师利用 NVIDIA 图形处理器（GPU）的强大计算能力进行通用计算。

CUDA 由 CUDA核心库、CUDA编译器（nvcc）及 CUDA 运行时（runtime）和驱动程序组成。CUDA 带的驱动程序可能和系统安装的版本不兼容，所以安装 CUDA 时，最好卸载系统的 Nvidia 驱动，使用 CUDA 带的驱动程序。

### 下载 CUDA

Nvidia 提供了 Ubuntu 和 Debian 各种版本的 CUDA deb 包，但没有提供 Deepin 系统的。虽然说 deb 包基本上是通用的，但是对于驱动来说，可能还是有所区别。所以在选择安装包时，选择可执行的二进制包（runfile）。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/using_deepin_03.png)

当前最新版本是 12.5.1，可以使用如下命令下载：

```
wget https://developer.download.nvidia.com/compute/cuda/12.5.1/local_installers/cuda_12.5.1_555.42.06_linux.run
```

### 删除 Nvidia 驱动

```
$ dpkg -l | grep nvidia | awk '{ print $2 }' | xargs sudo apt -y remove
$ sudo apt -y autoremove
```

### 禁用开源驱动

编辑 blacklist 文件

```
$ sudo vi /etc/modprobe.d/blacklist-nouveau.conf
```

加入如下内容：

```
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
```

执行如下命令，禁用Nvidia开源驱动

```
$ sudo update-initramfs -u
```

### 安装 CUDA

重启系统，使用如下命令，验证是否禁用成功，无输出即成功禁用。这个时候系统分辨率可能不正常，先暂时忽略。

```
$ lsmod | grep nouveau
```

按 CTRL + ALT + F2，进入命令行控制台，需要输入用户名及密码，然后输入如下命令停止桌面系统：

```
$ sudo service lightdm stop
```

最后一步，安装 CUDA：

```
$ sudo sh cuda_12.5.1_555.42.06_linux.run
```

如果一切顺利，不需要做特别选择，按照默认选项安装即可，完毕后重启系统。

如果系统显示分辨率正常，且执行 nvdia-smi 命令输出如下内容，大功告成。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/using_deepin_04.png)

## 安装 Anaconda

Deepin 系统默认已经安装了 3.10 版本的 Python，但对于一名开发者来说，推荐使用 Anaconda。

Anaconda 是一个开源的 Python 和 R 编程语言的发行版本，常用于科学计算、数据科学、机器学习、以及数据分析等领域。它包括了大量的科学计算库和工具，并提供了一个方便的包管理和环境管理系统。

我选择 Anaconda 主要是看中它提供了强大的包管理和环境管理功能，解决了 Python 生态系统中常见的依赖冲突问题。这个对于不同的 AI 应用程序运行非常重要。通常我们为 AI 应用程序安装 Python 包，可能会指定具体的版本，但不同的 AI 应用指定的版本可能不同，可能产生冲突。Anaconda 通过建立不同的 Python 虚拟环境，解决这一问题，相比 Python 自身提供的 venv，使用上更加方便，也更容易管理。

我之前在 Ubuntu 上安装了 Anaconda，安装在数据盘上，在 Deepin 系统下可以直接使用。比如我之前为 ComfyUI 建立了 Python 虚拟环境，切换到 Deepin 上后，激活 comfyui 虚拟环境，ComfyUI 直接就可以运行了。

```bash
(base) alex@alex-deepin-os:~$ conda activate comfyui
(comfyui) alex@alex-deepin-os:~$ cd /data/ai/ComfyUI/
(comfyui) alex@alex-deepin-os:/data/ai/ComfyUI$ python main.py
[START] Security scan
[DONE] Security scan
## ComfyUI-Manager: installing dependencies done.
** ComfyUI startup time: 2024-07-14 23:25:44.481099
** Platform: Linux
** Python version: 3.10.14 (main, May  6 2024, 19:42:50) [GCC 11.2.0]
** Python executable: /data/ai/anaconda3/envs/comfyui/bin/python
** Log path: /data/ai/ComfyUI/comfyui.log

Prestartup times for custom nodes:
   5.5 seconds: /data/ai/ComfyUI/custom_nodes/ComfyUI-Manager

Total VRAM 10823 MB, total RAM 32031 MB
pytorch version: 2.0.1+cu117
xformers version: 0.0.20
Set vram state to: NORMAL_VRAM
Device: cuda:0 NVIDIA GeForce RTX 2080 Ti : cudaMallocAsync
Using xformers cross attention
/data/ai/anaconda3/envs/comfyui/lib/python3.10/site-packages/diffusers/models/transformers/transformer_2d.py:34: FutureWarning: `Transformer2DModelOutput` is deprecated and will be removed in version 1.0.0. Importing `Transformer2DModelOutput` from `diffusers.models.transformer_2d` is deprecated and this will be removed in a future version. Please use `from diffusers.models.modeling_outputs import Transformer2DModelOutput`, instead.
  deprecate("Transformer2DModelOutput", "1.0.0", deprecation_message)
flash_attn import failed: No module named 'flash_attn'
### Loading: ComfyUI-Manager (V2.41)
### ComfyUI Revision: 2279 [2f360ae8] | Released on '2024-06-22'
Torch version too old for FP8

Import times for custom nodes:
   0.0 seconds: /data/ai/ComfyUI/custom_nodes/websocket_image_save.py
   0.1 seconds: /data/ai/ComfyUI/custom_nodes/ComfyUI_ExtraModels
   0.2 seconds: /data/ai/ComfyUI/custom_nodes/ComfyUI-Manager
   2.0 seconds: /data/ai/ComfyUI/custom_nodes/comfyui-hydit

Starting server

To see the GUI go to: http://127.0.0.1:8188
```

## 安装 IDE

Deepin 的应用商店有 Qt Creator，但我更建议去 Qt 官网下载 Qt Community 版本。Qt 是一套开发工具和开发框架的集合，Qt Creator 只是 Qt 的工具之一。使用 Qt 安装程序，可以选择 Qt 库的版本，可以安装 Qt 扩展库、Qt Creator等工具，可以根据需要选择。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/using_deepin_05.png)

Visual Studio Code 差不多是开发人员必备，这个从 Deepin 的应用商店安装即可。

Deepin 应用商店有一个**开发工具**类别，里面有 PyCharm、IDEA、Android Studio、Eclipse等集成开发工具，一键安装，相比去官网下载，还是方便不少。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/using_deepin_06.png)

## 编译 Chromium 浏览器

Deepin 用于基本的软件开发，毫无压力，开发大型软件，能否胜任呢？这里试一试 Chromium 浏览器的编译。

Chromium 浏览器的代码及其庞大，其代码量不亚于 Android 系统。其源码采用自己独特的 depot_tool 进行管理。所以第一步就是下载 depot_tool。

### 安装 depot_tool

克隆 `depot_tools` 库:

```
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

将 `depot_tools` 加入到环境变量`PATH` ，如果不想每次都输入如下命令，可以将其加入到`~/.bashrc`。

```
$ export PATH="/path/to/depot_tools:$PATH"
```

需要注意的是，在 Ubuntu 下我经常将用户特有的设置放在 ~/.profile 文件，但这个在 deepin 下是不生效的。

### 获取 Chromium 源码

Chromium 源码超级庞大，为了节省时间和空间，我就没有全库克隆，没有拉取历史提交，只克隆最新代码：

```
fetch --nohooks --no-history chromium
```

### 运行钩子

Chromium 源码不仅包含代码，还包含一些编译所需的二进制文件，这些是通过钩子脚本从服务器上下载的。在前面的命令中，我加上了 --nohooks，就是下载代码时，不运行钩子脚本。但这个脚本是必须运行的，否则会缺少一些二进制文件和编译器，这个步骤可以单独运行：

```
$ gclient runhooks
```

### 编译代码

在 Chromium 的说明文档中，还需要运行一个 install-build-deps.sh 脚本，用来安装所依赖的系统包，但这个脚本在 deepin 下执行会出错，其实不执行这个脚本也没有关系，因为 deepin 已经安装了比较全的包，仅仅需要再补充以下两个包就可以：

```
$ sudo apt install build-essential gperf
```

Chromium 使用 Ninja 作为其主要构建工具，并使用名为 GN 的工具来生成 .ninja 文件。您可以创建任意数量的具有不同配置的构建目录。要创建构建目录，请运行：

```
$ gn gen out/Default
```

编译 Chromium 浏览器

```
$ autoninja -C out/Default chrome
```

经过漫长的编译，自己的 Chromium 浏览器就生成了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/using_deepin_07.png)

## 小结

总结一下，使用 deepin 作为开发系统，具有如下优点：

* **美观的用户界面**：Deepin Linux 以其优雅和现代的界面设计著称，提供了良好的用户体验。
* **易于使用**：Deepin Linux 注重用户体验，提供了简洁直观的界面和操作方式。
* **丰富的预装软件**：Deepin Linux 自带了许多常用的软件和工具，减少了用户在初期安装和配置软件的时间。
* **良好的中文支持**：作为中国开发的操作系统，Deepin Linux 对中文支持非常好，适合中国开发者使用。
* **基于 Debian**：Deepin Linux 基于 Debian，这意味着你可以访问大量的 Debian 软件包和资源，确保开发所需的工具和库能得到很好的支持。

你会选择国产系统作为开发系统吗？欢迎留言讨论。

