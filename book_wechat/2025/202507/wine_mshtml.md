
# IE虽死，枷锁犹在：国产化替代的最后一公里

2022 年 6 月 15 日，微软正式终止了对 Internet Explorer 浏览器的支持。这个曾经统治互联网近二十年的“蓝色 E”，终于被扫进历史的故纸堆。

对第一代前端开发者来说，IE6 曾是挥之不去的梦魇；而对如今的年轻开发者而言，或许难以想象，为了兼容 IE6，前端工程师曾付出多少额外的精力与妥协。这个发布于 2001 年的浏览器，凭借与 Windows XP 的深度绑定，在近十年的时间里牢牢占据市场主导地位。然而，它对 W3C Web 标准的支持极其糟糕。想开发跨浏览器的 Web 应用？首先得跨过 IE6 这座大山 —— 而它偏偏又是无法绕开的“标准”。

虽然微软后来陆续推出了 IE7、IE8，直到 IE11 才基本实现了与现代浏览器一致的标准支持，但这也是 Trident 引擎的最后回光返照。随后，微软全面转向 Chromium，推出新的 Edge 浏览器，Trident 引擎正式退出历史舞台。

然而，当我们为一个时代的终结欢呼时，却发现 IE 虽死，其“遗毒”仍在 —— 如幽灵般，潜伏于无数企业系统之中。

## 被保留的“遗毒”：MSHTML 与 IE 模式

虽然 IE11 桌面应用已正式退役，但其核心渲染引擎 MSHTML（又称 Trident）仍作为系统组件继续存在，作为兼容旧有系统的保障。微软在 Edge 浏览器中引入了 **IE 模式**，允许加载依赖旧技术的网站，而这正是依赖于系统中的 MSHTML 引擎实现的。

事实上，早在 IE 浏览器之外，大量传统桌面应用也借助 Windows 提供的 WebBrowser 控件，在窗口中嵌入网页视图 —— 而这个控件用的也是 MSHTML。特别是在银行客户端、内部 OA/ERP 系统、报表工具等企业级软件中，这种嵌套网页的方式非常常见。

考虑到这些场景的广泛存在，微软承诺对 IE 模式的支持将持续至少到 **2029 年**，也就是说 MSHTML 引擎仍将在系统中“苟活”相当长时间。

## 比“遗毒”更致命的，是 ActiveX

如果说 MSHTML 是“遗毒”，那么 ActiveX 控件就是“剧毒”。

在智能手机和 移动 App 普及之前，网上银行操作几乎都要依赖 ActiveX 控件。这些控件不仅要求使用 IE 浏览器，还要下载银行等机构专门提供的插件包。

ActiveX 是微软专有的技术，也为 IE 所用，基于 COM 接口，能够直接调用本地的 DLL 或 OCX 组件，实现前所未有的“浏览器-操作系统”融合能力。例如：

* 直接读写本地文件；
* 操作打印机、扫描仪、高拍仪等硬件设备；
* 访问加密硬件（如 U 盾、身份证读卡器）；
* 调用 Office 组件生成、编辑文档。

当年由于确实缺乏替代方案，这些能力对政企系统来说不可或缺。但 ActiveX 也因此完全绑定了 Windows 和 IE，无法在 Linux 或其他浏览器中运行，成为国产替代进程中最棘手的难题之一。

## 国产化的挑战：IE 成了最大拦路虎

随着“信创”浪潮推动操作系统国产化，越来越多单位尝试在国产系统（如统信 UOS）上部署业务系统。然而，原本运行在 Windows + IE 环境下的系统无法直接迁移，IE 又成了国产系统开发者的梦魇。

和个人用户不同，业务系统往往不具备灵活替换的能力：

* 多为定制开发，原开发商可能早已消失；
* 即使开发商尚在，适配新平台也是额外开销；
* 若现有系统运行稳定，业务方通常缺乏升级意愿 —— 毕竟升级意味着风险。

因此，压力就落在了国产操作系统厂商身上。不仅要提供自己的操作系统内核和桌面环境，还必须设法兼容那些过时、复杂、绑定 Windows 的旧有系统。

不像苹果那样可以强制开发者配合迁移，国产系统要“稳中求进”，更显艰难。

## Wine 中的 MSHTML 实现

为解决这一“技术债”，Wine 提供了在 Linux 上运行 Windows 应用的能力，其中就包括对 IE 渲染引擎的适配：**Wine 中的 MSHTML 实现**。

具体来说，Wine 并未复刻 Trident，而是采用 Mozilla 的 Gecko 引擎作为底层渲染核心，并在其上实现与微软 MSHTML 接口兼容的封装。这样，Windows 应用就可以继续调用熟悉的接口，与网页内容交互，而渲染由 Gecko 负责。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/wine_mshtml_01.png)

Wine 的 MSHTML 实现实际上是一个“桥接层”，主要承担以下职责：

* 将 MSHTML 对象映射为 Gecko 的 `nsDOM*` 对象；
* 转换方法调用，如 DOM 操作、脚本执行等；
* 将 Gecko 的事件翻译为 MSHTML 可识别的事件；
* 管理两者之间的文档生命周期。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/wine_mshtml_02.png)

不过，由于 Trident 与 Gecko 本质上是两套不同的浏览器内核，事件模型、CSS 支持、DOM 实现均存在差异，加之 MSHTML 中大量依赖 COM 接口 —— 而 Wine 对这些接口的支持仍不完全 —— 所以这个兼容层在实际应用中属于“能用但不完美”的程度。特别是在 UI 复杂、脚本交互密集的业务场景下，容易出现页面渲染失败、样式错乱、脚本报错等问题。

尽管国内操作系统厂商正持续推动 Wine 的完善，但距离 Windows 原生仍有不小的差距。

## Wine 实现 ActiveX 控件兼容：Pipelight 方案

Wine 实现 ActiveX 兼容的一种方案是采用开源项目 Pipelight，项目地址：https://github.com/keithbowes/pipelight。

Pipelight 项目之初是为了 Linux 浏览器能够加载 Windows 平台上的 NPAPI 插件，主要是微软的 Silverlight 插件，官方不支持 Linux 系统，而当时 Netflix 等流媒体服务当时依赖微软的 Silverlight 技术。为了让 Linux 用户也能访问流媒体服务， Pipelight 项目应运而生。项目也解决了其它一些 Windows 插件需求，比如 Adobe Flash Player（Windows 版）、Widevine DRM 等插件。

随着 Silverlight 与 Flash 的终结，以及主流浏览器（Chrome、Firefox）彻底移除 NPAPI 支持，Pipelight 项目已经停止维护。

但这个项目提供了一个解决 ActiveX 控件运行的思路：**通过 Wine 后台运行 Windows 插件，并通过本地 NPAPI 接口暴露给浏览器。**。而且国内的浏览器厂商，如360、龙芯浏览器，都保留了 NPAPI 插件的支持，所以这套方案看起来非常老旧，但依然可行。

其架构如下：

* **Linux 集成层**：`libpipelight.so`，实现 NPAPI 接口，与浏览器通信；
* **Windows 执行环境**：`pluginloader.exe`，通过 Wine 执行 Windows 插件；
* **进程间通信**：双向 IPC 系统，在浏览器与插件之间同步操作与数据。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/wine_mshtml_03.png)

当然，ActiveX 控件的机制与 NPAPI 插件不同，Pipelight 本身无法直接加载 ActiveX，需要在此基础上进行二次开发与兼容处理。

## 小结：真正的国产化，是从“排毒”开始的

为什么说国产系统难做？不仅仅是因为系统层面的问题，更在于如何**让现有业务系统“无痛迁移”**。指望一纸政令就让所有开发商配合是不现实的 —— 哪怕是一个“已死”的 IE，其留下的历史包袱也足以让人头疼。

这不仅是操作系统的国产化，更是一场**应用生态的断舍离与重构**。只有当我们彻底切断对 IE、ActiveX 等旧有技术的依赖，才能真正实现自主、可控与创新。
