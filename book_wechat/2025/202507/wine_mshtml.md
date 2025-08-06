# IE虽死，枷锁仍在：国产化替代的最后一公里

2022 年 6 月 15 日，微软正式终止了对 Internet Explorer 浏览器的支持。这个曾经统治了互联网近二十年的“蓝色 E”，终于被扫进了历史的故纸堆。

对于第一代前端开发者来说，曾经 IE6 是挥之不去的噩梦。对于今天的年轻开发者来说，可能无法想象，前端开发者为了兼容 IE6，要多做多少无用功。IE6，这个发布于 2001 年的浏览器，在长达近十年的时间里，凭借与 Windows XP 的深度捆绑，占据了绝对的市场霸主地位。然而，它对 W3C Web 标准的支持却极其糟糕，想开发跨浏览器的 Web 应用？对不起，需要翻过 IE6 这座山。由于这是市场占有率第一的浏览器，又不得不翻。

微软虽然后来推出了 IE7、IE8 等后续版本，直到 IE 11，才基本完成与现代浏览器一致的标准实现。不过这也是 IE Trident 引擎的回光返照，因为微软随后开始推出基于 Chromium 的 Edge 浏览器，自家的 Trident 引擎彻底放弃。

然而，当我们欢呼一个旧时代的终结时，却发现 IE 虽死，其“遗毒”仍在，像一个幽灵，继续盘桓在企业应用的各个角落。

因为尽管 Internet Explorer 11（IE11）桌面应用程序已经退役，但其核心渲染引擎MSHTML（也称为Trident）为了向后兼容性，仍然是操作系统的一部分。Microsoft Edge 浏览器包含一个**IE模式**，允许企业和开发者加载需要 IE 特定功能或旧有技术的网站。这个模式的实现，就依赖于系统内置的MSHTML引擎。除了 Edge 的 IE 模式，在过去几十年的软件开发中，无数的桌面应用程序（如银行客户端、企业内部的 ERP/OA 系统、各类报表工具）都使用了 Windows 提供的 WebBrowser 控件，在其窗口中内嵌一个网页视图。而这个控件，调用的正是系统级的 MSHTML 引擎。

由于 MSHTML 仍然是系统的一部分并被积极使用（尤其是在企业环境中），微软会继续为其提供安全更新。微软承诺对 Edge 中的 IE 模式至少支持到 2029 年，这意味着在此期间 MSHTML 会一直得到维护。

如果说 MSHTML 引擎是“遗毒”，那么 ActiveX 控件就是“剧毒”。在手机银行流行之前，在 PC 上使用网上银行，都必须先下载银行提供的专用控件，当然浏览器也必须使用 IE 浏览器，因为 ActiveX 是微软推出的一项基于 COM 的、 IE 专属的技术。这也是当年为什么很多用户对 IE 恨之入骨，但又不得不使用的原因。为什么银行、12306 等偏爱 ActiveX 控件？因为它允许网页直接调用在本地安装的原生代码（通常是 .dll 或 .ocx 文件）。这打破了浏览器的安全沙箱，赋予了网页与操作系统直接交互的强大能力，例如：

* 直接读写本地文件。
* 调用打印机、扫描仪、高拍仪等硬件设备。
* 读取 U 盾、身份证读卡器等加密硬件。
* 调用Office组件进行文档操作。

特别是 U 盾，差不多是银行客户端的标配，当时确实也没有很好的选择。然而，它的弊端也是致命的：完全与Windows和IE绑定，无法在 Linux 或其他浏览器上运行。

随着国产化替代（信创）的浪潮兴起，我们需要在基于 Linux 的国产操作系统（如统信 UOS）上，运行那些过去只能在 Windows + IE 上使用的业务系统。这时，IE 就成了国产系统开发者的噩梦。

要知道，业务系统可不是个人系统，可以随便升级替换的。首先，业务系统一般都是定制的，当年开发业务系统的公司是否存在。即使还在，如果适配新的系统，那又是一笔新的费用。其次，如果一套系统运行得好好的，站在业务方的角度，缺少升级的动力，因为升级就意味着不确定性，意味着风险，谁来承担这个风险？

于是，压力就来到操作系统厂商，国产化替代可以，必须解决这些难题。国产系统为什么这么难做，这些都是难点。我们又不可能像苹果公司那样硬气，强制要求应用升级，否则不给上应用商店。

之前写过多篇在国产系统上运行 Windows 应用程序的文章，为了解决 IE 遗留下来的技术债，还得通过 Wine 来实现。

## Wine 中的 MSHTML 引擎

Wine 中的 MSHTML 实现将 Internet Explorer 的渲染能力适配到非 Windows 平台上运行，为那些依赖 IE 技术的 Windows 应用程序提供了必要的接口和功能。

具体说来，Wine 的 MSHTML 实现底层借用了 Mozilla 的 Gecko 渲染引擎，在其之上提供了微软的接口层。这样，Windows 应用程序就可以通过熟悉的接口与网页内容交互，同时利用 Gecko 的渲染能力。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/wine_mshtml_01.png)

Wine 的 MSHTML 实现相当于一个桥阶层，并不进行实际的渲染，通常需要进行如下的桥接操作：

* 将 MSHTML 对象映射到 Gecko 的 nsDOM* 对象

* 将 MSHTML 的方法调用转换为 Gecko 的操作

* 将 Gecko 的事件转换为 MSHTML 的事件

* 在两个系统之间管理文档的生命周期

每个 MSHTML 对象通常都持有对其对应 Gecko 对象的引用。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/wine_mshtml_02.png)

由于微软的 Trident 和 Gecko 引擎属于两个完全不同的引擎，在网页渲染、事件处理存在一定的差异，再加上 MSHTML 中大量依赖 COM 的接口在 Wine 中并没有完全实现，或者实现不完善，特别是大量边缘 API 未覆盖，所以 Wine 的 MSHTML 实现属于“足够用”级别，在 UI 复杂、脚本交互多、CSS/DOM 依赖深的场景下，仍会遇到渲染失败、样式丢失、脚本报错。

在国内操作系统厂商的推动下，Wine 的实现越来越完善，但离完美还有距离，什么时候能够完全抛弃 Wine，才是国产操作系统取得了真正的成功。

## Wine 实现 ActiveX 兼容

Wine 实现 ActiveX 兼容的一种方案是采用开源项目 Pipelight，项目地址：https://github.com/keithbowes/pipelight。

Pipelight 项目之初是为了 Linux 浏览器能够加载 Windows 平台上的 NPAPI 插件，主要是微软的 Silverlight 插件，官方不支持 Linux 系统，而当时 Netflix 等流媒体服务当时依赖微软的 Silverlight 技术。为了让 Linux 用户也能访问流媒体服务， Pipelight 项目应运而生。项目也解决了其它一些 Windows 插件需求，比如 Adobe Flash Player（Windows 版）、Widevine DRM 等插件。

随着 Silverlight、Flash 插件的消亡，NPAPI 插件机制也从 Chrome、Firefox 等主流浏览器移除，这个项目其实早就停止了维护。但这个项目提供了一个解决 ActiveX 控件运行的思路，而且国内的浏览器厂商，如360、龙芯浏览器，都保留了 NPAPI 插件的支持，所以这套方案看起来非常老旧，但依然可行。

Pipelight 的核心思路是**通过 Wine 在后台运行 Windows NPAPI 插件，并将其以 Linux 插件的形式暴露给浏览器**。其实现架构大致包括：

* Linux 集成层：libpipelight.so 库，实现了 NPAPI 接口，用于与浏览器兼容

* Windows 执行环境：pluginloader.exe 程序，通过 Wine 托管并运行 Windows 插件

* 进程间通信机制：双向 IPC 系统，实现 Linux 浏览器与 Windows 插件之间的数据无缝交互

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/wine_mshtml_03.png)

当然，要实现 ActiveX 兼容，仅仅通过 Pipelight 是无法加载 ActiveX 控件的，毕竟 ActiveX 控件加载和 NPAPI 插件的处理逻辑不同，还需要进行二次开发。

## 小结

为什么说国产系统难做，除了需要解决操作系统本身的问题外，如何将业务系统平滑迁移到新系统上也需要投入大量的人力。指望国家一声令下，所有软件厂商齐心协力转到新系统上是不现实的。就连IE这种已经淘汰的产品，它在长达二十年的统治中所留下的技术债务和设计思想，都不会自动消失。对于正在大力推进的国产化事业而言，这不仅仅是更换操作系统和硬件，更是一场应用层面的“排毒”与“新生”。啥时能彻底切断这些历史的枷锁，才能真正实现自主、可控与创新。
