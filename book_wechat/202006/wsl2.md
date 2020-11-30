# 还在安装双系统？ 试试 Windows 和 Linux 合体

作为一个长期使用 Linux 作为主力系统的开发人员，我经常向周围的朋友安利 Linux （ Ubuntu ）系统。但非常尴尬的是，大部分人都是浅尝辄止，最后还是会回到 Windows 系统，布道成功的并不多。毕竟习惯的力量非常强大，要从一个熟悉的系统转向陌生的系统，需要有足够的动力才行。一句“ Linux 系统上能够玩游戏吗？”就能让我哑口无言，还有诸如网课、办公、图像处理软件之类的问题。的确，Linux 系统在应用程序支持方面（特别是娱乐休闲类）一直是软肋。虽然经过这么多年的努力，但具有全平台支持（Windows、Linux、Mac OS等）的软件还是相当少，特别是国内的一些办公软件、网课客户端，通常只开发了 Windows 版本。

在科技领域，向来是强者恒强， Linux 系统的生态要赶超 Windows，还有很长的路要走。这个时候，我们通常会选择一些折中的方案，兼收两系统之长处。这其中最常用的方法就是在电脑上安装双系统。娱乐办公的时候进 Windows 系统，开发的时候进 Linux 系统。然而，电脑上的双系统，并没有做到像手机那样“双卡双待”，在一个时间点，我们只能在其中一个系统中。比如如果要在写代码之余，玩两把游戏，就需要退出 Linux 系统，然后进入 Windows系统。反之依然，要进入 Linux 系统，就需要退出当前的系统。这样在两个系统之间切换，甚是不便。

#### WSL

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202006/images/wsl2_01.png)

因为战略转型，微软开始积极拥抱 Linux 系统，目前已经是 Linux 社区最大的贡献者。他们也意识到这一问题，所以在 Windows 10 中增加了 WSL（Windows Subsystem for Linux）。顾名思义， WSL 就是 Windows 系统的 Linux 子系统，但并非固化在 Windows 10中，而是作为 Windows 组件， 出现在 Windows 10 系统中（1607 版本之后）。

**重点：要体验 WSL，需要将 Windows 10 更新到 1607 之后的版本。**

关于 WSL，有几点需要澄清：

1.  WSL 并不是 Linux发行版， 它本质上是系统层面对 Linux 内核的支持，为了在 Windows 中使用，还需要在 Win10 的应用商店内搜索下载我们喜欢的 Linux 发行版。目前， WSL 支持Ubuntu， Kali Linux，OpenSUSE 等，后续会有更多 Linux 发行版支持 WSL 。
2.  WSL 作为系统层的一部分，相较于应用层（虚拟机）会消耗更少的资源，并且与系统锲合度更高。事实上，我们只需要打开一个类似 CMD 的Bash命令行窗口，就可以开始使用 WSL（相当于建立了一个 Session ，因为 WSL 会一直伴随 Win10 的运行而运行），相对于从虚拟机启动既省时又省力。
3.  由于 WSL 子系统依附于“系统”，所以“子系统”会有一些限制。不过也不用担心，虽然 WSL 不是完整的 Linux 系统，绝大多数在完整 Linux 系统能做的事，在 WSL 中也可以做到。

#### WSL2

在试水了 WSL 之后，微软迅速推出了 WSL2，这是一个全新的 WSL 版本。技术演进从来都不是一蹴而就的，所以目前 WSL 和 WSL2 是并存的。秉承着旧不如新的原则，建议大家选择 WSL2 这个版本。为什么呢？

> WSL2 使用了全新的体系结构，该体系结构可运行真正的 Linux 内核，可在 Windows 上运行 ELF64 Linux 二进制文件。它提高了文件系统性能，增加了完整的系统调用兼容性。

当然，你也可以选择将 Linux 发行版作为 WSL 或 WSL2 运行。而且，你可以随时在这些版本之间切换。

#### 安装

Microsoft承诺在不久的将来为 WSL2 提供流畅的安装体验，并能够通过 Windows 更新来更新 Linux 内核。就目前而言，安装过程稍显复杂，但也不要畏惧，按照步骤操作即可。

本文介绍在 Windows 10 上安装 Ubuntu 20.04，但这个过程对于微软应用商店中的任何发行版都适用。

首先，你应该启用 Windows Subsystem for Linux 可选功能。请以管理员身份打开 PowerShell 并运行以下命令：

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

接下来，将系统更新为 WSL2 。为此，Windows 10 必须更新为 2004 版或这之后的版本，并且必须在**BIOS 设置**中启用英特尔的虚拟化技术。然后，以管理员身份启动 PowerShell 并运行以下命令：

```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

重新启动计算机以完成 WSL 安装并更新到 WSL2 。然后，在安装新发行版时，需要将 WSL2 设置为默认版本。为此，以管理员身份打开 PowerShell 并运行以下命令：

```
wsl --set-default-version 2
```

运行该命令后，你可能会看到此消息： WSL2 需要对其内核组件进行更新。有关信息，请访问[https://aka.ms/wsl2kernel](https://aka.ms/wsl2kernel)。一旦安装了内核，请再次运行该命令，它应该成功完成而不显示消息。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202006/images/wsl2_02.png)

最后一步也很重要，我们应该安装 Linux 发行版。打开微软应用商店，然后搜索 Ubuntu 20.04 LTS 。安装后你应该可以在 Windows 的开始菜单中找到一个新添加的 **Ubuntu应用程序** 。启动它并按照说明（主要是创建一个新的 Linux 用户）完成安装。

要检查 WSL2 上是否安装了 Linux 发行版，请运行:

```
wsl --list --verbose
```

如果结果表明它使用WSL 1，则可以通过运行:

```
wsl --set-version <发行名称> <versionNumber>
```

对其进行更改。

这样，你已经在 Windows 10 中安装了完整的 Ubuntu 发行版！

#### 安装之后

准备好 Ubuntu 之后，我们可以安装所需的任何东西。例如，如果你从事数据分析，则可以安装最新的 Anaconda 发行版；如果你是前端工程师，则可以安装 angular 和 npm 等。

然而，到目前为止，WSL 还没有对 Linux GUI 应用程序的支持，这意味这你只能在 WSL Linux 中使用命令行，关于 Linux 命令行，可以参考我前面的一篇文章：

[掌握基本的命令行，迈向 Linux 第一步](https://mp.weixin.qq.com/s/3gh5bnnLjToh4AK5hZ62Qw)

没法在 WSL Linux 中使用图形应用程序，那编写 Linux 应用程序怎么办？总不至于使用编程神器 vi 或 emacs 吧，这可不是普通程序员能 hold 得住的两大神器。

在这里，向大家隆重介绍 Visual Studio Code。VSC 是许多开发人员首选的 IDE 。借助于**远程开发插件**，我们可以使用在 Windows 下安装的 VSC，通过 SSH 协议编辑位于 WSL2 上的源码。VSC 有各种语言的智能提示，内置 git 支持，还有众多的插件，你所能想到的需求，几乎都可以通过插件来完成。所以编写 Linux 应用程序，同样和编写 Windows 应用程序一样简单方便。

#### 缺憾

WSL2 在不停的演化中，也许过不了多久，又会出现 WSL3、WSL4，对 Linux 系统的全面支持也越来越好。就目前而言，最大的缺憾之一就是前面提到的不支持 Linux GUI。另一个缺憾是对 GPU 的支持。在过去的几年中，WSL、虚拟化、DirectX，Windows 驱动等团队和其他合作伙伴一直在努力开发此项功能，相信要不了多久，就可以得到全面支持。那个时候，我们就可以在 WSL 中进行机器学习相关的开发。

虽然有着这些缺憾，但 WSL 作为 Linux 入门的系统，还是非常推荐。经常有朋友向我抱怨，说在公司就做一些修改 BUG、CURD 操作之类的开发，一点技术含量没有。与其抱怨，不如行动起来。目前几乎所有的服务器后端都采用了 Linux 系统，其它的诸如 Android 系统开发、内核开发、机器学习、大数据，都是以 Linux 作为首选系统。迈出转变的第一步，也许前面的路就开阔了。

#### 参考

1. [What is the Windows Subsystem for Linux?](https://docs.microsoft.com/en-us/windows/wsl/about)
2. [WSL 使用指南——01 WSL入门](https://zhuanlan.zhihu.com/p/34885179)
3. [Dual Boot is Dead: Windows and Linux are now One](https://towardsdatascience.com/dual-boot-is-dead-windows-and-linux-are-now-one-27555902a128)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)