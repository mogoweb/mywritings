# 新的挑战：WebGL

这段时间一直在死磕 Chromium 的 8K 高清视频播放，虽然之前写过一些关键技术的实现，主要难点差不多攻破，但投入到产品中，依然还要解决很多实际中的问题，比如卡顿、格式支持、音视频不同步等等。前期的相关文章：

* [Chromium 改造实录：国标AVS2 & AVS3 支持起来](https://mp.weixin.qq.com/s/mTCs8Q4PtUUe2M5IDP1Rwg)
* [Chromium 改造实录：增加 MP2 音频支持](https://mp.weixin.qq.com/s/ZJjG_JYS51WM-WiY8XejVA)
* [Chromium 改造实录：增加 MPEG TS 格式支持](https://mp.weixin.qq.com/s/enrzjVLy_VACqTKavuam7Q)
* [选择最新 Chromium，支持 H264 / H265](https://mp.weixin.qq.com/s/IpioXG-_NaGOc9nnKe6xbQ)

就在我准备歇口气的时候，又遇到了新的麻烦，这次的主角是 WebGL。

具体来说，运营方上线了一个业务，结果在浏览器中现实成这样：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202306/images/webgl_basic_01.png)

一调查，这个业务的页面是通过 WebGL 实现的。

对我来说，WebGL 是一个全新的东西。从名字上来讲，这个 WebGL 和 OpenGL 有很大关系，事实也是如此。

> WebGL 是一种基于 Web 的 3D 绘图标准，它可以让 Web 开发者利用 JavaScript 和 OpenGL ES 2.0 来创建和展示 3D 场景和模型。 WebGL 通过在 HTML5 Canvas 元素上提供一个 JavaScript 绑定，可以让浏览器直接调用系统的显卡，实现硬件加速的 3D 渲染。这样， WebGL 不仅可以提高 Web 页面的交互性和视觉效果，还可以用于开发复杂的导航和数据可视化应用，甚至是 3D 网页游戏。

问题是我对 OpenGL 也不熟。

当前的情况比较尴尬，使用相同 chromium 代码编译出来的 Chromium Browser 或者 Content Shell，显示 WebGL 页面没有问题，但是基于 WebView 的浏览器就存在问题。更为尴尬的是，基于相同 Chromium 代码编译出的 SystemWebView， 放到其它的硬件平台上也没有问题。

所以接下来的调查方向有两个：

1. WebView 和 Content 层在 WebGL接入上有所差异，具体的差异在哪里？
2. OpenGL ES 驱动上是否存在问题，为什么不同的硬件平台存在不同表现？

两条路都很难，本来编写 OpenGL 程序就是一个相当难得事情，现在还要去弄清楚背后是怎么去实现的。调查硬件驱动更是难，和具体的硬件绑定太深。

说起 WebGL，估计很多人比较陌生，现实中似乎也应用得不多。但实际上，WebGL 还是一个不错的技术：

* WebGL 是一个开放的、跨平台的、免费的标准，它由 Khronos Group 制定和维护，得到了多个浏览器厂商和硬件厂商的支持。
* WebGL 不需要安装任何插件或外部库，只要浏览器支持 WebGL，就可以在任何设备上运行WebGL应用。
* WebGL 可以与其他 Web 技术和 API 结合使用，例如 HTML、CSS、SVG、DOM、Web Audio、WebRTC 等，实现丰富的多媒体和网络功能。
* WebGL 可以利用现有的 OpenGL ES 2.0 或 OpenGL ES 3.0（WebGL 2.0）的知识和资源，开发者可以使用熟悉的图形编程语言（GLSL）和工具。

现实中也存在许多 WebGL 的应用场景：

* 地图：WebGL 最广为人知的例子是谷歌地图的地形视图。不管是何种形式的地形图或空间排列，都可以从 3D 展示中获益。你可以在浏览器中旋转、缩放、平移地图，看到不同的角度和细节。你甚至可以切换到街景模式，体验一下虚拟现实的感觉。
* 游戏：游戏是 WebGL 的最大应用领域，有很多优秀的 WebGL 游戏可以在浏览器中玩。比如说，Unity 是最流行的游戏开发平台，并提供 WebGL 构建选项。你可以在网页上玩一些 Unity 制作的游戏，例如《坦克大战》、《死亡之屋》、《疯狂的出租车》等等。
* 在线展览：WebGL 可以让你在浏览器中创建和浏览 3D 的虚拟展厅，展示各种类型的作品和内容，例如艺术品、服装、汽车、建筑等等。WebGL 的优势是它不需要安装任何插件或软件，只要有一个支持 WebGL 的浏览器，就可以直接访问在线展览的网址，享受身临其境的体验。

这次暴露问题的业务就是在线博物馆，遇到问题也没法逃避，只能迎头直上，下来需要恶补一些 OpenGL 和 WebGL 的知识了。

后续有收获，会和大家一起分享，欢迎围观！

