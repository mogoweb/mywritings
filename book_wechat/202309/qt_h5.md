# QT 应用程序 H5 化方案研究

前段时间一直在在解决几个非常棘手的 BUG，趁着假期空闲时间，思考一下 QT 应用程序 H5 化的方案。

应用程序 H5 化，并不是改成完全意义上的 WEB App，对用户来讲依然是一个 App，只不过 App 包含了一个 WEB 引擎，界面展现采用网页，但主要的实现依然采用 C++。

应用程序 H5 化似乎是很多公司的选择，特别是像 Electron 这样的方案出现后，桌面应用程序采用 H5 技术来开发成为了可能。

我们考虑 H5 化，主要是出于成本的考虑。QT的界面主要是由程序员设计与开发，但程序员实际上并不擅长做界面，让美工去装一个 QT 来设计界面也不现实。很多美工设计懂网页设计，所以由网页设计师把基本网页做出来，由程序员去实现代码逻辑，是一个成本较低的选择。如果网页展现和业务逻辑分离，程序员更可以把精力放在业务逻辑功能实现上。

另外 QT 开发的界面，在多种分辨率下的展现有差异，且难以适配，也是我们考虑 H5 化的原因。

上网查了一下资料，主流的方案有如下几种：

* QT WebEngine
* Felgo 框架
* Electron 框架
* CEF 框架

#### QT WebEngine

QT WebEngine 是 QT 自带的一个模块，是一个基于 Chromium 的浏览器引擎，可以在 QT 应用程序中嵌入 web 内容，并实现 C++ 和 JavaScript 之间的交互。这种方案可以保留原有的 QT 代码，只需要在 UI 层使用 HTML5 和 CSS3 来构建界面，并通过 QWebChannel 类来注册 C++ 对象，使其可以在 JavaScript 中调用。

其优点是：

* 可以复用原有的 QT 代码和逻辑，减少开发成本和风险。
* 可以利用 Chromium 的强大功能和稳定性，提高应用程序的质量和用户体验。
* 可以轻松地将 web 内容集成到 QT 应用程序中，实现混合开发。

缺点也比较明显：

* 需要额外安装和配置 QT WebEngine 模块，增加应用程序的大小和复杂度。
* 需要处理 C++ 和 JavaScript 之间的通信和转换问题，增加开发难度和出错可能。
* QT WebEngine 并没有同步更新 Chromium 版本，特别是我们为了能够支持 Win 7，不能选择最新的 QT 版本，所包含的 Chromium 版本非常老。在之前的开发中就发现显示网页存在问题。

因为我们在之前开发中，发现 QT WebEngine 访问某些网页存在问题，所以在应用程序中包含了 CEF 框架，所以这种方案 Pass。

#### Felgo 框架

Felgo 框架是一个基于 QT 的跨平台开发工具，可以使用 QT Quick（QML 和 JavaScript）来开发移动和桌面应用程序。这种方案可以利用 QT Quick 的声明式语法和动画特性来创建出色的用户界面。

其优点：

* 可以使用 QML 和 JavaScript 来开发应用程序，降低开发难度和门槛。
* 可以使用 Felgo 提供的一些高级组件和插件，如游戏引擎、地图、图表、广告、分析等，增加应用程序的功能和价值。
* 可以使用 Felgo 提供的一些便捷服务，如云编译、热更新、测试、发布等，提高开发效率和质量。

其缺点：

* 需要额外安装和配置 Felgo 框架，增加应用程序的大小和复杂度。
* 需要支付 Felgo 的许可费用，可能会增加开发成本。
* 需要学习 QML，这对设计师来说有点为难。

我们的应用程序并没有那么复杂，也不希望 Felgo 框架的额外功能，引入框架可能会造成不必要的复杂性，这种方案也不会考虑。

* Electron 框架

Electron 框架是一个基于 Node.js 和 Chromium 的桌面应用程序开发工具，可以使用 HTML5、CSS3 和 JavaScript 来构建界面，并且可以调用 Node.js 的模块和 API。这种方案可以充分利用 web 技术的优势和生态系统，但是可能会牺牲一些性能和内存占用。

优点：

* 可以使用 HTML5、CSS3 和 JavaScript 来开发应用程序，无需学习 C++ 或 QML，降低开发难度和门槛。
* 可以使用 Node.js 的模块和 API 来实现后端逻辑和功能，无需依赖 QT 的组件或服务。
* 可以使用 web 技术的丰富资源和社区，如框架、库、插件、教程等，提高开发效率和质量。

缺点：

* 打包后的应用体积巨大，因为需要包含整个 Chromium 内核和 Node.js 运行时。
* 需要处理 Node.js 和 Chromium 之间的通信和转换问题，增加开发难度和出错可能。
* 过于庞大，小公司很难 hold 住。

这种方案需要完全颠覆我们现有的代码架构，没法考虑，咱们还得小步快跑。

#### CEF 框架

CEF 是一个基于 Google Chromium 的开源项目，可以在本地应用程序中嵌入一个符合 HTML5 标准的 Web 浏览器控件。它提供了一个稳定的 API 来隔离用户与底层 Chromium 和 Blink 代码的复杂性。

简单说，CEF 并不是一个应用程序开发框架，只提供在应用程序中嵌入浏览器的方案。

CEF 支持 Windows、MacOS 和 Linux 等多个平台，并且采用 BSD 许可协议，对商业应用非常友好。

因为我们的应用程序中出于别的原因，已经集成了 CEF 框架，所以选择 CEF 做 H5 方案是一个自然的选择。

接下来需要解决的是，H5 界面如何与 C++ 业务逻辑代码交互，主要的方案有：

* 使用 CEF 提供的 JavaScript 绑定功能，可以在 C++ 代码中注册一个对象或者一个函数，然后在 H5 界面中通过 JavaScript 调用该对象或者函数，实现 C++ 与 JavaScript 的双向通信。
* 使用 CEF 提供的自定义协议功能，可以在 C++ 代码中注册一个自定义的 URL 协议（如 myapp:// ），然后在 H5 界面中通过该协议发送请求，实现 H5 界面向 C++ 代码发送数据或者命令。
* 使用 CEF 提供的 IPC（进程间通信）功能，可以在 C++ 代码中发送或者接收消息，然后在 H5 界面中通过 JavaScript 监听或者触发消息，实现 C++ 与 JavaScript 的异步通信。

接下来就是具体的实现工作，还有一些研究工作需要做，比如 CEF 中 JavaScript 绑定功能如何实现，如何自定义 JavaScript 对象，总体上挑战应该不大，主要是一些开发工作量。

真正难搞的是一些兼容性问题，特别是面对 C 端用户，Windows 版本不同、其它 App 的干扰等等，在这段时间解决 BUG 的过程中，对此有很深的认识。这些问题也没有办法提前预判，只能见招拆招，接下来会有什么奇遇呢？

后续我会继续分享。