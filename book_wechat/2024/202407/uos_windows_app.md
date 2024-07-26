# 在国产系统上安装 Windows 应用程序

在《[使用国产操作系统作为开发系统](https://mp.weixin.qq.com/s/pvwNQHaguXs2X1XnTDE7QA)》一文说到我将开发系统切换到国产系统 Deepin （统信UOS社区版）上。经过这段时间的使用，非常满意。唯一有点遗憾的是，我平常下棋的围棋软件，在 Deepin 系统上没有。在 UOS 应用商店中搜索**围棋**，倒是有几款围棋软件，但没有我平常使用的对弈软件。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_01.png)

去野狐的官网，有Windows 、Mac OS 和移动端版本，独独缺少了 Linux 版本。没有办法，应用软件开发商也是看菜下碟。国产系统经过大力推广，仍然是小众系统。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_02.png)

作为一名棋臭瘾大的伪棋迷，忍不了，一定要想办法解决这个问题。

事实上，这个问题是有解的，因为 UOS 应用商店上就有一些 Windows 应用程序，这得益于 Linux 下的 Wine.

## Wine

Wine（Wine Is Not an Emulator）是一个在类 Unix 操作系统（如Linux）上运行 Windows 应用程序的软件兼容层。它允许用户运行大多数 Windows 程序而无需安装 Windows 操作系统。以下是关于Wine的一些介绍：

Wine 不是一个模拟器（emulator），它不模拟 Windows 硬件环境，而是将 Windows API 直接转换为 POSIX 调用，从而实现更高效的运行性能。

Wine 支持大量的 Windows 应用程序和游戏，包括 Microsoft Office、Photoshop 以及许多流行的游戏。

Wine 在 Deepin 系统上已经安装，所以不需要费力地去解决安装问题。美中不足的是，使用 Wine 运行 Windows 应用程序，需要从命令行启动。能不能像 UOS 应用商店里的 Windows 应用程序那样，安装到 Deepin 系统中？

经过一番搜索，发现 UOS 应用商店中有一款应用程序** UOS 应用迁移助手**，可以做到将 Windows 应用程序打包成 Deepin 的安装包。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_03.png)

## UOS 应用迁移助手

从应用商店下载 UOS 应用迁移助手后，打开 UOS 应用程序迁移助手。有三种方式选择应用，我选择第二种，直接放野狐围棋的 Windows 版本下载链接，省去了手动下载的步骤。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_04.png)

接下来输入应用程序名、描述、包名、版本等信息，包名在 Linux 中是用来区分不同应用的，取一个独特的名字即可，版本号不一定需要对应真正的版本号，先给一个 1.0.0 就可以。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_05.png)

接下来就是生成安装包的过程，稍等片刻，就可以完毕。在这个过程中，还启动了野狐围棋的安装程序，按照默认安装即可。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_06.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_07.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_08.png)

生成的安装包位于桌面，可以看到生成的有两个包，一个是 x86 架构的，一个是 ARM 架构的。我的电脑是英特尔的 CPU，可以忽略掉 ARM 架构的安装包。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_09.png)

双击桌面上的程序包，即可启动安装过程。注意，和 Windows 有点不同，Linux 系统权限管理比较严格，安装应用程序需要管理员权限，所以会出现提示输入密码。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_10.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_11.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_12.png)

安装完毕后，从启动器里可以看到新装的野狐围棋。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_13.png)

点开之后，就可以愉快的玩耍了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_windows_app_14.png)

## 小结

当前，国产系统应用软件不足是一个客观事实，但通过曲线救国的方式将 Windows 应用程序打包成 Deepin 系统安装包，可以弥补国产系统应用软件不足的问题。而 UOS 应用迁移助手，则让我们自己动手，将一些自己所需的应用程序制作称安装包，在国产系统下运行。
