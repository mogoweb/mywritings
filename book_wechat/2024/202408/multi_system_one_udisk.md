# 各种国产操作系统，一个 U 盘搞定

熟悉 Windows 装机的朋友对老毛桃和大白菜这类装机工具应该不陌生。这两款流行的工具可以用来制作启动盘，方便进行系统安装、备份和还原等操作。它们集成了多种磁盘工具，并支持一个启动 U 盘安装多个版本的 Windows 系统，如 Windows 7、Windows 8、Windows 10 等。

在进行国产系统的软件适配时，我们面临一个挑战：需要支持多种国产操作系统，有时甚至需要回溯到不同版本中解决问题。然而，我们的测试电脑只有一台，因此频繁安装不同的国产系统成为必要。为什么不使用虚拟机呢？因为我们的软件与硬件密切相关，虚拟机在硬件支持上存在不足。

在这种情况下，我们希望有一款类似老毛桃的装机工具，可以通过一个 U 盘解决各种国产系统的安装需求。然而，老毛桃等工具只支持 Windows 系统的安装。一次偶然的机会，我发现了一款非常适合我们需求的启动 U 盘制作工具：Ventoy。

访问 [Ventoy官网](https://www.ventoy.net/cn/index.html) https://www.ventoy.net/cn/index.html 后，发现这似乎是一位大神的个人作品，但功能非常强大，使用也很简单。它不仅支持 Windows 和 Linux 操作系统，还完美兼容国产系统。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/multi_system_one_udisk_01.png)

下载最新版的 Ventoy 后，解压缩得到以下文件：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/multi_system_one_udisk_02.png)

插入 U 盘，双击 Ventoy2Disk.exe：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/multi_system_one_udisk_03.png)

点击“安装”按钮即可完成操作。这个步骤会重新分区 U 盘，因此请确保 U 盘上没有重要数据。制作完成后，你会发现 U 盘里空空如也，没有任何文件：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/multi_system_one_udisk_04.png)

不用担心，实际上这个 U 盘还有一个引导分区，只是在插入电脑时不会自动挂载。你只需将国产系统的 ISO 镜像文件复制到 U 盘中即可。只要 U 盘容量足够大，你可以复制多个镜像文件，比如我就一次性复制了 4 个：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/multi_system_one_udisk_05.png)

重启电脑，将启动设备设置为 U 盘，你会看到如下选项：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/multi_system_one_udisk_06.jpg)

界面上显示的正是前面复制的镜像文件名。选择其中一个后，会进入一个选择界面：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/multi_system_one_udisk_07.jpg)

通常选择第一个即可。接下来就会进入国产系统的引导界面：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/multi_system_one_udisk_08.jpg)

参照我之前的文章《[低成本使用国产系统的几种方式](https://mp.weixin.qq.com/s/GHZPyFXVf6f6rAfQbvrkzg)》，你甚至可以使用 LiveUSB 模式，直接在 U 盘上运行系统而无需安装：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202408/images/multi_system_one_udisk_09.png)

如果试用满意，可以双击桌面上的“安装UOS”，进入安装模式。

通过 Ventoy 这个 U 盘启动盘制作工具，你可以轻松解决多个系统的安装问题。以后需要安装新系统时，只需将镜像文件复制到 U 盘中即可。

在前面的演示中，我将所有镜像文件放在 U 盘的根目录下。其实 Ventoy 还支持文件夹管理，这样即使有大量镜像文件，也能井然有序。

通常 U 盘会格式化为 exFAT 格式，因此可以存储大于 4GB 的文件，并在各种系统上访问。除了制作启动盘之外，你还可以将文件复制到其中，当作普通 U 盘使用，是不是一盘多用？

据官网介绍，Ventoy 还支持 LiveUSB 模式的持久化存储，这样我们在做软件测试的时候，甚至不用安装系统，系统就运行在 U 盘上。关于 Ventoy 支持 LiveUSB 模式的持久化存储的方法。我将在后续的文章中探讨，敬请关注！

