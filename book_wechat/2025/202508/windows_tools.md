# 介绍几款 Windows 应用调试的小工具

在《[兜兜转转，我又开始研究 Windows 系统](https://mp.weixin.qq.com/s/A79_LI0Vxq5aiod51QFLMw)》一文中，我提到最近开始专注于研究在国产系统上兼容运行 Windows 应用程序。这也意味着我将与各种麻烦为伍，每天都在与 Windows 应用在国产系统上运行不正常的问题作斗争。与麻烦作伴，当然手中要常备几个趁手的工具。这里，我就来给大家介绍几款 Windows 下的小工具。它们不是万能的解决方案，不是那种一上来就能立刻解决问题的“神兵利器”，而是针对特定方向的工具，虽然小巧，但在关键时刻却非常实用。

## spy++

Spy++ 是一个由 Microsoft 提供的 Windows 开发工具，它是 Visual Studio 附带的一部分，主要用于调试和分析 Windows 应用程序的窗口和消息。它可以帮助开发人员查看应用程序的窗口结构、窗口消息和其他窗口属性。通过 Spy++，开发人员可以深入了解应用程序的行为，尤其是关于窗口和消息传递的部分。

这个工具是我使用得最多的，因为 Windows 应用在国产 Linux 系统下运行，经常会有无法输入中文、焦点丢失、窗口看不到等问题。通过 spy++，可以进行如下定位：

* 查看窗口结构

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/windows_tools_01.png)

这是通过 wine 运行 spy++ 的窗口信息，可以看出，Spy++ 可以列出系统中所有正在运行的窗口，以树状显示窗口层次，可以非常清晰的看到窗口的父子关系，并显示它们的窗口句柄、标题等信息。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/windows_tools_02.png)

通过点开窗口的属性，在属性检查器中还可以查看窗口更详细的信息，包括坐标、窗口样式、窗口类、进程等信息。在有 Windows 应用存在显示问题时，通过查看窗口坐标、样式、层级等信息，可以定位问题的方向。

* 查看窗口消息

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/windows_tools_03.png)

在窗口右键菜单中选择"消息"，可以监控发往该窗口的消息。它会捕获各种窗口消息（如 WM_PAINT、WM_KEYDOWN 等）并将它们显示出来。这对于调试窗口消息的处理过程，比如调试窗口交互（按钮点击、键盘输入）非常有帮助非常有用。如果消息比较多，还可以监控的定类型的消息：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/windows_tools_04.png)

* 查找特定窗口

有时在 wine 日志中，会记录窗口句柄，我们可以根据窗口的句柄定位到具体的窗口。此外我们还可以根据窗口的标题、类名等条件来查找特定的窗口。这对于寻找特定的应用程序窗口非常有用，尤其是在窗口数目较多时。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/windows_tools_05.png)

图中用红色方框标记的工具，可以将它拖到屏幕上任一窗口上，然后就可以定位到该窗口，显示出对应的窗口句柄、标题和窗口类。

### 在 Linux 下使用 spy++

spy++ 是一个windows 程序，可以通过 wine 在 Linux 下运行。

首先我们从 Visual Studio 下复制可执行文件，比如我安装的 Visual Studio 2022 Community 版本，就在 `C:/Program Files/Microsoft Visual Studio/2022/Community/Common7/Tools` 目录下。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/windows_tools_06.png)

这下面还有比较多的 dll，最好是将 Tools 整个目录复制过去。

接下来需要指定 wine 容器，需要和要监控的应用程序运行在相同的容器中，这个只需设置一个环境变量即可。比如：

```
export WINEPREFIX=$HOME/.pipelight-wine
```

最用，使用 wine 启动 spy++，比如在 UOS/deepin 系统下，可以使用 `deepin-wine10-stable` 启动：

```
cd Tools
deepin-wine10-stable spy++.exe
```

spy++ 有 32 位和 64 位版本，spy++.exe 是 32 位程序，spyxx_amd64.exe 是 64 位程序，根据要调试的 Windows 应用程序是 32 位还是 64 位，启动对应的 spy++ 程序。

## Dependencies

Dependencies 是一款 Windows 平台下的开源工具，用于查看 Windows 可执行文件（.exe、.dll 等）或动态链接库文件的依赖关系。我主要用它来分析一个程序或库在运行时需要哪些其他的库或文件，以及这些库文件之间的相互依赖。

1. 分析依赖关系：Dependencies 可以查看一个可执行文件或库文件依赖的所有 DLL 文件，包括它们的版本和路径。这对于检查程序是否缺少必要的动态库文件、查看程序所依赖的外部库非常有用。

2. 查找缺失的 DLL：如果某个程序无法启动或报错缺少 DLL 文件，Dependencies 可以帮助开发者快速查找缺失的依赖库，进而方便地进行修复。

3. 查看导出函数：Dependencies 允许用户查看 DLL 文件导出的所有函数及其符号。有时编写 Windows Demo 应用时遇到链接问题，就可以派上用场。

4. 检查程序的兼容性：Dependencies 可以检查一个程序是否能够在当前环境中正确运行。例如，如果程序依赖的库文件是较旧版本的 DLL，而系统中安装的是新版 DLL，可能会导致兼容性问题，Dependencies 就能帮助发现这种问题。

Dependencies 程序已经很久没维护了，但还可以用。下载后解压并运行即可。

官方下载地址：https://github.com/lucasg/Dependencies

这个程序同样有 32 位和 64 位版本，请根据需要进行下载。下载的程序包中有命令行版本和带图形界面的版本，一般使用带图形界面的。

这个程序的运行需要 .net 4.8 的环境，所以要确保 wine 容器安装了 dotnet 48 的运行时环境。

```
cd Tools
deepin-wine10-stable DependenciesGui.exe
```

在打开 dll 文件后，显示如下信息：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/windows_tools_07.png)

## Process Monitor

Process Monitor（简称 ProcMon）是微软 Sysinternals 工具集中的一款强大而实用的实时系统监控工具。它结合了前两款工具 Filemon（文件活动监控）和 Regmon（注册表活动监控）的功能，能够全面、细致地监视系统上所有进程的文件系统、注册表、进程/线程活动等信息。

在分析 Windows 应用问题时，通常只有二进制文件，而没有源码，对于其内部实现，基本靠猜。有了这个工具，我们就可以监控应用程序调用哪些 Windows 系统 API，而模拟系统 API 实现正是 wine 的工作。特别是针对文件访问、注册表操作、进程启动/停止、线程创建/销毁等活动的监控，对于排查应用程序程序崩溃、性能问题、资源泄漏等问题也许会有帮助。

Windows API 众多，所以 Process Monitor 提供强大的过滤功能，可以让我们只关注特定的 API 访问。例如，如果我们只关心与文件相关的 API 操作（如 CreateFile、ReadFile 等），可以在过滤器中设置相关条件，过滤出这些操作。

通过 Filter 菜单，我们可以设置复杂的过滤条件，例如过滤特定的进程、路径、操作类型等。通过过滤，我们能够更加精准地监控某些 API 调用。

访问如下微软 Sysinternals 官方网站 下载 Process Monitor 工具:

> https://learn.microsoft.com/en-us/sysinternals/downloads/procmon

解压后，直接运行 Procmon.exe 启动程序（无需安装）。Procmon 同样分为32位版本和64位版本，根据我们要监控的应用是32位还是64位，选择对应的版本。

该工具因为依赖window内核驱动模块，暂时无法通过 wine 运行，但可以在 Windows 下运行，也可以帮助我们分析应用程序的一些行为。

## 小结

以上介绍的几款小工具，虽然看似简单，但在实际调试和排查 Windows 应用程序问题时，往往能发挥关键作用。从窗口结构和消息监控到依赖关系分析，再到系统级的实时监控，它们可以帮助开发人员和系统管理员更好地理解和解决应用程序在运行时出现的各种问题。借助这些工具，我们能够深入分析 Windows 应用在国产系统的问题。