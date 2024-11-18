# QCefView 在信创项目中的应用

虽然使用 C/C++ 编程语言多年，但直到去年才开始使用 Qt 开发软件。使用了 Qt 之后，才感觉使用 C/C++ 开发应用软件也并没有想象中那么困难。Qt 的跨平台特性也非常适合信创项目开发，因为信创项目大多基于国产 Linux 系统进行开发。Linux 系统虽然在服务器端取得了非常大的成功，但桌面端的应用软件开发却一直没有什么起色。采用 Qt 开发，可以兼顾主流的 Windows 和 Mac OS 系统，这对于软件开发商来说是非常必要的，毕竟现在国产 Linux 系统用户量偏少。

Qt 使用 C++ 作为底层实现，具有较高的执行效率，加上 Qt Creator 这个集成开发环境（IDE），支持代码编辑、调试、UI 设计、项目管理等功能，为快速软件开发提供了坚实基础。当前，国产 CPU 性能相对较弱，使用 C++ 能够部分缓解性能问题，提升用户体验。

在应用程序中集成浏览器功能非常常见，例如访问 AI 生成的 markdown 内容，或直接访问 Web 网页。多数情况下，我们并不需要完整的浏览器界面，只需要一个渲染引擎以显示网页，并支持基本的 JavaScript 交互。

在 QT 应用程序中集成浏览器，最简单的方法是使用 QtWebEngine。

## QtWebEngine

QtWebEngine 是 Qt 框架中的一个模块，用于在应用程序中集成现代 Web 技术。它基于 Chromium 开源项目，提供了一个用于显示 Web 内容的浏览器引擎，使开发者能够在他们的 Qt 应用程序中嵌入 Web 内容和功能。QtWebEngine 提供了一个易于使用的 API，开发者可以使用它来创建具有 Web 功能的应用程序，如浏览器、HTML5 游戏、在线帮助系统等。QtWebEngine 允许开发者在应用程序中直接使用 HTML、CSS、JavaScript 等 Web 技术，同时提供了与 Qt 框架的无缝集成，使开发者能够轻松管理和控制 Web 内容的展示和行为。

但是，QtWebEngine 模块基于的 Chromium 版本比较老，并没有随着 Chromium 项目快速迭代。比如我们使用的 Qt 5.15.2 中的 QtWebEngine 模块使用的是 Chromium 78 版本，而最新的 Chromium 版本已经到了 130。大多数情况下我们不需要跟进最新版本，但如果应用程序所访问的网站使用了最新的前端技术，那么 QtWebEngine 可能会出现一些显示异常的问题。

虽然 QtWebEngine 和 Chromium 都是开源的，但这两个项目都相当庞大，要将 QtWebEngine 升级到最新的 Chromium 版本，难度和工作量都相当大。

此外，我们还需要注意，Qt 的一些组件，这其中就包括 QtWebEngine， 是不能应用在商业项目中的。如果要在产品中使用 QtWebEngine，需要获得 Qt 商业许可证。

我们还有第二种选择，就是使用 CEF 框架。

## CEF

CEF (Chromium Embedded Framework) 是一个开源项目，它允许开发者在自己的应用程序中嵌入 Chromium 浏览器引擎。它基于 Google 的 Chromium 项目，提供了一个稳定的、高性能的、现代的浏览器引擎，开发者可以通过 CEF 将其集成到自己的桌面应用程序中。

CEF 的优势之一是它提供了灵活的自定义和扩展性。开发者可以通过添加自定义的 JavaScript 扩展或使用 Chromium 的内置 API 来扩展和定制浏览器的功能。此外，CEF 还提供了一套丰富的 API，用于控制浏览器的行为、处理用户输入、管理 Web 内容等方面。

和 QtWebEngine 不同之处在于，CEF 采用了滚动发布策略，其主要特点包括：

* 持续更新： CEF 通过持续不断地将 Chromium 的最新代码集成到其代码库中，实现持续更新。这意味着 CEF 的开发者可以随时访问和使用最新的 Chromium 特性和改进。

* 频繁发布： CEF 以较短的周期发布新版本，通常每周或每两周发布一个新版本。这使得开发者能够更快地获得最新的功能、性能改进和安全补丁。

* 版本稳定性： 尽管采用了持续更新和频繁发布的策略，CEF 仍然会在每个版本中进行测试和验证，以确保版本的稳定性和可靠性。

* 向后兼容： CEF 的滚动发布策略通常会保持向后兼容性，即在新版本中引入的改进和功能不会破坏现有的应用程序或功能。

在 QT 应用程序中集成 CEF，可以获得较新的 Chromium 内核，升级 Chromium 版本也相对容易一些。

虽然 CEF 对 Chromium 的对外接口进行了封装，对外提供了一个一致的接口，但 CEF 使用起来还是挺复杂，你如果看到过 CEF 的对外头文件就会明白。那有没有类似 QtWebEngine 或者 QtWebView 这样的更简单的封装呢？答案就是 QCefView。

## QCefView

QCefView 是一个与 CEF 集成的 Qt 小部件（Widget），可以使用 QCefView 而无需编写任何与 CEF 代码相关的代码，就像使用其它 Qt Widget 那样。项目采用了 LGPL-2.1 许可，可以放心在商业软件中免费使用。

项目地址： https://github.com/CefView/QCefView

中文文档：https://cefview.github.io/QCefView/zh/

### 构建 QCefView

QCefView 没有提供二进制发布版文件，所以在使用之前必须从源码构建。好在 QCefView 的构建非常简单，提供了几个脚本来帮助完成 CEF 的下载，编译以及 QCefView 的构建。

下面介绍 QCefView 在 Linux 下的构建。

1. 下载代码

```
git clone https://github.com/CefView/QCefView.git
```

QCefView 项目实际包含了两个 git 库，一个是 QCefView，另一个则是 QCefViewCore。

2. 修改CEF配置

构建 QCefView 可以选择 CEF 的版本，CEF 发布版本可以访问 CEF Automated Builds 网站 https://cef-builds.spotifycdn.com/index.html 去选择。

一般情况下，使用 QCefView 指定的 CEF 版本即可满足要求，如果对 CEF 版本有特殊要求，可以修改 cmake/CefViewCoreConfig.cmake 文件。

通常，只需要修改第一行中的指令

> set(CEF_SDK_VERSION "127.3.5+g114ea2a+chromium-127.0.6533.120")

设置 CEF_SDK_VERSION 为要使用的目标版本。

** 注意! 这里的格式为 127.3.5+g114ea2a+chromium-127.0.6533.120，非 cef_binary_127.3.5+g114ea2a+chromium-127.0.6533.120_windows64，因为 QCefView 本身是跨平台，支持 Windows、Linux 和 Mac OS 系统，脚本会根据系统去下载对应的 CEF 。**

如果网络环境环境较差，使用自动脚本没法下载 CEF，也可以手动下载 CEF binary，放到QCefView\CefViewCore\dep目录即可，不需要改文件名。

3. 设置编译环境

QCefView通过CMake管理项目构建，所以请先安装CMake。最低要求版本为3.19.1，推荐使用最新版本。

设置 QTDIR 环境变量，使其指向 Qt 工具链目录，对于 Linux 系统，使用如下命令：

```
export QTDIR=/work/Qt/5.15.2/gcc_64
```

4. 构建 QCefView

```
./generate-linux-x86_64.sh
cmake --build .build/linux.x86_64
```

生成的二进制文件在 .build/linux.x86_64/output/Release/bin/ 下，包含如下文件：

```
$ tree .build/linux.x86_64/output/Release/bin
.build/linux.x86_64/output/Release/bin
├── CefViewWing
├── chrome-sandbox
├── icudtl.dat
├── libcef.so
├── libEGL.so
├── libGLESv2.so
├── libQCefView.so
├── libvk_swiftshader.so
├── libvulkan.so.1
├── resources
│   ├── chrome_100_percent.pak
│   ├── chrome_200_percent.pak
│   ├── locales
│   │   ├── af.pak
│   │   ├── am.pak
│   │   ├── ar.pak
│   │   ├── bg.pak
│   │   ├── bn.pak
│   │   ├── ca.pak
│   │   ├── cs.pak
│   │   ├── da.pak
│   │   ├── de.pak
│   │   ├── el.pak
│   │   ├── en-GB.pak
│   │   ├── en-US.pak
│   │   ├── es-419.pak
│   │   ├── es.pak
│   │   ├── et.pak
│   │   ├── fa.pak
│   │   ├── fil.pak
│   │   ├── fi.pak
│   │   ├── fr.pak
│   │   ├── gu.pak
│   │   ├── he.pak
│   │   ├── hi.pak
│   │   ├── hr.pak
│   │   ├── hu.pak
│   │   ├── id.pak
│   │   ├── it.pak
│   │   ├── ja.pak
│   │   ├── kn.pak
│   │   ├── ko.pak
│   │   ├── lt.pak
│   │   ├── lv.pak
│   │   ├── ml.pak
│   │   ├── mr.pak
│   │   ├── ms.pak
│   │   ├── nb.pak
│   │   ├── nl.pak
│   │   ├── pl.pak
│   │   ├── pt-BR.pak
│   │   ├── pt-PT.pak
│   │   ├── ro.pak
│   │   ├── ru.pak
│   │   ├── sk.pak
│   │   ├── sl.pak
│   │   ├── sr.pak
│   │   ├── sv.pak
│   │   ├── sw.pak
│   │   ├── ta.pak
│   │   ├── te.pak
│   │   ├── th.pak
│   │   ├── tr.pak
│   │   ├── uk.pak
│   │   ├── ur.pak
│   │   ├── vi.pak
│   │   ├── zh-CN.pak
│   │   └── zh-TW.pak
│   └── resources.pak
├── snapshot_blob.bin
├── v8_context_snapshot.bin
└── vk_swiftshader_icd.json
```

这下面的文件都需要复制到使用 QCefView 的 Qt 应用程序的相同目录下。

在 .build/linux.x86_64/example/QCefViewTest/output/Release/bin/ 下还有一个 QCefViewTest 测试程序，可以用来验证 QCefView 能否正常工作。

## 小结

本文介绍了在 Qt 应用程序中集成浏览器的一种方法：集成 CEF，接着介绍了 QCefView 这个开源项目，并详细给出了在 Linux 下编译 QCefView 的方法。

现有的很多资料都是基于 Windows 系统的，Linux 系统资料相对少一些，上面的方法适用于国产信创系统，比如统信 UOS 系统，希望对大家有所帮助。
