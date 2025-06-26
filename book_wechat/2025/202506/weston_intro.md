# 万字长文详解 Wayland 协议、架构

年初写过一篇文章《[从 X11 到 Wayland，迈出这一步为何如此艰难？](https://mp.weixin.qq.com/s/_EO5xbY5J1dCukiMhxbQXA)》，分析了从 X11 演进到 Wayland 所面临的困难。直到今天，Wayland 替代 X11 仍不容乐观。虽然 Ubuntu、Debian 等发行版本都将默认的桌面环境设置到 Wayland，但很多用户在遇到诸多兼容问题之后，仍会切换回 X11。比如我前段时间使用 Ubuntu 24.04，实在受不了里面的输入法，为了使用搜狗输入法，不得不切换回 X11。

理论上，X Window System (X11) 服役已逾三十年，设计理念已显陈旧，又有性能、安全和维护性上的一系列问题，作为 X11 的继任者 Wayland，替换 X11 是一件水到渠成的事，为什么阻力这么大？

本文将从 X11 的核心痛点出发，深入剖析 Wayland 为何被设计成如今的模样。我们将系统性地讲解 Wayland 的核心架构、协议机制，并通过分析 libwayland 的客户端与服务端代码，揭示其通信的底层逻辑。无论你是桌面环境开发者、Linux爱好者还是对图形技术感兴趣的工程师，本文都将为你提供一份从理论到实践的详尽指南。

读完本文之后，你就会明白 Wayland 取代 X11，并非技术上做不到，更多的是生态问题，甚至是，有人的地方就有江湖，技术之争在开源世界并不少见。就在本月初，曾深度参与 Xorg 开发的维护者 Enrico Weigelt 又搞了一个 X11 的分支： XLibre Xserver，试图复兴逐渐被边缘化的图形服务器 X11。

本文借助了 AI 工具 DeepWiki 进行了 libwayland 和 Weston 的源码分析，如果有什么错误，那一定是我的问题，还请指正。

## 一、X11 的痛点与 Wayland 的诞生

先看一张 X11 和 Wayland 架构对比图，来自 freedesktop.org：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/weston_intro_01.png)

从上图可以看出，Wayland 相比 X11 架构，就是拿掉了 X Server。因为这个 X Server 曾经扮演着重要的角色，如今却处境尴尬。

现代人可能难以理解，但我们要理解，X 系统诞生于 1980 年代，那时的计算环境是一台强大的中央服务器（比如 VAX 或 Sun SPARCstation）和许多连接到它的“哑终端”（Dumb Terminals）。这些终端几乎没有计算能力，只有一个屏幕、键盘和鼠标。网络速度也相对较慢。

X11 的设计完美地契合了这个场景。当一个应用程序（Client）想要在屏幕上画些什么时，它不会自己去计算每个像素的颜色。相反，它向 X Server 发送一系列绘图指令（Drawing Primitives）。这些指令非常基础，就像是在指挥一个远程的绘图机器人：

* XDrawLine(display, window, gc, x1, y1, x2, y2)：“在窗口的 (x1, y1) 到 (x2, y2) 之间画一条线。”
* XFillRectangle(...)：“用指定的颜色填充一个矩形区域。”
* XDrawString(...)：“在某个位置用指定的字体绘制一段文字。”

这种模式的优点在当时非常明显：

* 网络效率：发送 "画一条线" 这样的短指令，比发送这条线所包含的所有像素数据要节省得多。
* 客户端轻量：应用程序本身不需要复杂的渲染逻辑和大量的内存来存放像素数据，所有繁重的渲染工作都交给了 X Server。
* 硬件抽象：应用程序不需要关心屏幕的具体硬件是什么，X Server 会处理好这一切。

然而，随着计算机技术的发展，到了今天，计算环境发生了翻天覆地的变化：

* 硬件：个人电脑、甚至手机都拥有强大的 CPU 和专门用于图形计算的 GPU。
* 用户界面 (UI)：用户期待的不再是简单的线条和色块，而是平滑的抗锯齿字体、半透明的窗口、细腻的阴影、流畅的动画和复杂的渐变效果。

这些复杂的视觉效果很难用 X11 的基础绘图指令来描述。比如，你该如何用指令告诉 X Server 去画一个带有高斯模糊的半透明阴影？这几乎是不可能的。

因此，现代 GUI 工具包（如 GTK、Qt）和应用程序（如 Chrome、Blender）采取了一种完全不同的策略：客户端渲染（Client-Side Rendering）。

也就是说，客户端负责全部的渲染工作，生成一个完整的位图。它不再指挥服务器去画画，而是直接把画好的成品交给服务器去展示。

在这种工作模式下，一个尴尬而低效的工作流就出现了。我们以一个带有桌面特效（如窗口阴影、透明）的现代 Linux 桌面为例：

* 应用端：Chrome 使用其内置的 Skia 引擎，在自己的内存里精心绘制好了一个网页的完整图像。
* 第一层传递：Chrome 通过 X11 协议对 X Server 说：“嘿，我画好了我的窗口内容，这是一整张图片（Pixmap），你收一下。” （为了效率，通常会使用 MIT-SHM 共享内存扩展，避免通过 socket 拷贝整个图像数据，但这依然是一次 IPC 交互）。
* X Server 接收：X Server 拿到了这张由 Chrome 已经渲染好的图片。放在以前，X Server 会直接把这张图“贴”到屏幕上。但现在不行了...
* 合成管理器的介入：一个叫做**合成管理器（Compositor）**的独立程序（如 Mutter for GNOME, KWin for KDE）正在运行，它负责所有花哨的桌面效果。它早已告诉 X Server：“别自己往屏幕上画东西！任何窗口要更新内容，都先重定向给我。”
* 第二层传递：于是，X Server 只能无奈地将刚从 Chrome 那里收到的图片，再原封不动地转交给合成管理器。这又是一次 IPC 和潜在的内存拷贝。
* 最终合成：合成管理器收集了所有窗口（Chrome、终端、文件管理器等）的图像，像PS图层一样，为它们加上阴影、处理透明度，将它们组合（Composite）成最终的桌面画面。
* 上屏：最后，合成管理器把这张最终合成好的桌面图像，交给 X Server，让它显示到屏幕上。

很明显，这个过程引入了至少一到两次不必要的进程间通信（IPC）和数据拷贝，导致了延迟增加、性能下降和系统资源浪费。

有朋友可能回说，这么理解 X Server，是否太浅薄了，要知道 X Server 不仅是显示服务器，还包含了窗口管理逻辑、输入设备驱动、字体渲染、2D图形加速（通过 XRender 等）等众多功能。

是的，曾经 X Server 是一个巨大而复杂的单体，而且为了适应新功能（如 Compositing、RandR），X11 引入了大量扩展，这使得修复 bug 和添加新功能变得异常困难，所以很多功能已经从 X Server 剥离出来了。

* X Server 拥有窗口管理的底层能力，但具体的管理逻辑和决策早已被抽离到了独立的窗口管理器进程中（如 GNOME 的 Mutter、KDE 的 KWin）。
* X Server 内部几乎不再包含任何具体的输入设备驱动逻辑了。真正的驱动和处理逻辑已经下沉到了内核（evdev）和共享库（libinput）。X Server 只是一个 libinput 的客户端。
* 服务器端字体渲染已经完全被淘汰。现代 Linux 桌面上的所有字体渲染都是客户端侧完成的。
* 应用现在都是自己渲染好一整块图像。X Server 需要加速的不再是“画线”，而是“将一块大图像贴到另一块大图像上”（即blit操作）。

通过上面的逐步剥离，X Server 就像一个逐渐被“掏空”的历史建筑。

如果仅仅是上述问题，那么我们也没有必要推倒重建，就像 X 系统的改良派所坚持的，又不是不能用，修修补补还可再战。但是 X11 的体系架构存在严重的安全漏洞：

* 缺乏隔离：在 X11 中，任何一个客户端原则上都可以监听和注入任意其他客户端的输入事件（键盘、鼠标），以及截取屏幕内容。这使得恶意软件（如键盘记录器）的编写变得轻而易举。
* 信任模型：X11 的安全模型建立在“所有客户端都是可信的”这一过时假设之上，不适应现代多任务和沙盒化的应用环境。

为了甩掉 X Server 这个包袱，Wayland 应运而生，让那些已经实际承担工作的组件（合成器、客户端库）直接对话，不再需要中间这个日渐臃肿、低效且不安全的“传话筒”。

## Wayland 架构与核心概念

当我们说“我的桌面在用 Wayland”，这其实是一种简化说法。准确来说，我们的桌面环境（如 GNOME、KDE）正在使用遵循 Wayland 协议的组件。

什么是协议？ 想象一下 HTTP。HTTP 本身不是浏览器（如 Chrome）也不是 Web 服务器（如 Nginx），它只是一套规则，规定了浏览器和服务器之间应该如何请求和响应数据（GET /index.html HTTP/1.1）。

同样，Wayland 协议定义了一套客户端（应用程序）和服务器（合成器）之间进行通信的语言和规则。它精确地描述了：

* 客户端可以向服务器请求什么？（例如：“请为我创建一个用于显示的表面（surface）”）
* 服务器可以向客户端发送什么通知/事件？（例如：“鼠标指针进入了你的表面范围”）
* 数据应该如何打包和传输？（例如，使用二进制格式通过 Unix domain socket 传输）

Wayland 本身没有一行代码是关于如何画窗口、如何处理鼠标点击的。它只负责 **“传话”**。这意味着，不存在一个叫做“Wayland 显示服务器”的单一、标准的程序。任何软件，只要它能正确地“说”和“听” Wayland 协议定义的这门语言，它就可以成为一个 Wayland 服务器（Compositor）。这就是为什么 GNOME 的 Mutter 和 KDE 的 KWin 都是 Wayland Compositor，但它们是完全不同的软件。

这是 Wayland 相对于 X11 最具颠覆性的变革，也是其简洁高效的根源。在 X11 的世界里，显示服务器（X Server）、窗口管理器（Window Manager）和合成器（Compositor）是三个独立的角色，它们之间通过复杂的协议和扩展进行通信。

而在 Wayland 的世界里，这三个角色被合并成了一个单一实体：Compositor。

这个 Compositor 现在是唯一的权力中心，它直接负责：

* 显示管理：它直接通过内核的 DRM/KMS 接口与显卡对话。它可以查询连接了哪些显示器，设置每个显示器的分辨率和刷新率，并告诉显卡下一帧应该显示哪个图像缓冲区。没有中间商。
* 输入管理：它直接通过内核的 evdev 接口（通常借助 libinput 库）读取键盘、鼠标、触摸屏的原始输入事件。它知道哪个窗口当前拥有焦点，并负责将输入事件直接分发给对应的客户端。这从根本上增强了安全性，因为一个应用无法再轻易地监听到另一个应用的输入。
* 窗口管理：由于它就是窗口的最终合成者，它自然而然地承担了窗口管理的工作。移动窗口、改变大小、堆叠顺序等操作，都是 Compositor 内部的状态改变，不再需要跨进程通信。
* 合成：这是它的本职工作。它从所有客户端那里收集已经渲染好的表面（surfaces），像图层一样将它们组合起来，可能还会加上阴影、动画等效果，最终生成一幅完整的桌面图像，然后交给显示器。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/weston_intro_01.png)

再回顾一下这张架构对比图，Wayland 通过将所有图形和输入相关的职责集中到 Compositor，极大地简化了图形栈。数据流从“应用 -> X Server -> Compositor -> X Server -> 硬件”这条曲折的路径，变成了“应用 -> Compositor -> 硬件”这条直线高速公路。

### 构建 Wayland 世界的砖瓦

在了解 Wayland 总体蓝图后，现在我们来认识一下构成这个生态系统的具体“砖瓦”。

Wayland 通过一系列 XML 文件，用一种结构化的方式定义了通信的细节。下面是一个摘录的片段：

```xml
  <interface name="wl_display" version="1">
    <description summary="core global object">
      The core global object.  This is a special singleton object.  It
      is used for internal Wayland protocol features.
    </description>

    <request name="sync">
      <description summary="asynchronous roundtrip">
	The sync request asks the server to emit the 'done' event
	on the returned wl_callback object.  Since requests are
	handled in-order and events are delivered in-order, this can
	be used as a barrier to ensure all previous requests and the
	resulting events have been handled.

	The object returned by this request will be destroyed by the
	compositor after the callback is fired and as such the client must not
	attempt to use it after that point.

	The callback_data passed in the callback is undefined and should be ignored.
      </description>
      <arg name="callback" type="new_id" interface="wl_callback"
	   summary="callback object for the sync request"/>
    </request>

    <request name="get_registry">
      <description summary="get global registry object">
	This request creates a registry object that allows the client
	to list and bind the global objects available from the
	compositor.

	It should be noted that the server side resources consumed in
	response to a get_registry request can only be released when the
	client disconnects, not when the client side proxy is destroyed.
	Therefore, clients should invoke get_registry as infrequently as
	possible to avoid wasting memory.
      </description>
      <arg name="registry" type="new_id" interface="wl_registry"
	   summary="global registry object"/>
    </request>

    <event name="error">
      <description summary="fatal error event">
	The error event is sent out when a fatal (non-recoverable)
	error has occurred.  The object_id argument is the object
	where the error occurred, most often in response to a request
	to that object.  The code identifies the error and is defined
	by the object interface.  As such, each interface defines its
	own set of error codes.  The message is a brief description
	of the error, for (debugging) convenience.
      </description>
      <arg name="object_id" type="object" summary="object where the error occurred"/>
      <arg name="code" type="uint" summary="error code"/>
      <arg name="message" type="string" summary="error description"/>
    </event>

    <enum name="error">
      <description summary="global error values">
	These errors are global and can be emitted in response to any
	server request.
      </description>
      <entry name="invalid_object" value="0"
	     summary="server couldn't find object"/>
      <entry name="invalid_method" value="1"
	     summary="method doesn't exist on the specified interface or malformed request"/>
      <entry name="no_memory" value="2"
	     summary="server is out of memory"/>
      <entry name="implementation" value="3"
	     summary="implementation error in compositor"/>
    </enum>

    <event name="delete_id">
      <description summary="acknowledge object ID deletion">
	This event is used internally by the object ID management
	logic. When a client deletes an object that it had created,
	the server will send this event to acknowledge that it has
	seen the delete request. When the client receives this event,
	it will know that it can safely reuse the object ID.
      </description>
      <arg name="id" type="uint" summary="deleted object ID"/>
    </event>
  </interface>
```

* interface（接口）: 代表一个可以交互的对象类型。例如，wl_surface 代表一个客户端可以用来显示内容的矩形区域；wl_seat 代表一组输入设备。
* request（请求）: 客户端可以对一个接口对象发出的命令。例如，wl_surface 接口有一个 attach 请求，意思是“将这个图像缓冲区附加到此表面上，准备显示”。
* event（事件）: 服务器可以向客户端发送的通知。例如，wl_pointer 接口有一个 enter 事件，意思是“鼠标指针已经进入了你管理的某个表面”。
* enum（枚举）: 定义了一组常量值，例如键盘按键代码。

这些 XML 文件是给人读的，也是给机器读的。一个名为 wayland-scanner 的工具会读取这些 XML，并自动生成 C 语言的头文件和胶水代码，供 libwayland 和应用程序使用。

libwayland 又包含两个部分：

* libwayland-client: 供**客户端（应用程序）**使用。当一个应用（如 GTK 程序）调用 wl_surface_attach(...) 时，这个库会：
  1. 将这个函数调用翻译成 Wayland 协议定义的二进制消息格式（序列化）。
  2. 通过底层的 Unix domain socket 将这条消息发送给 Compositor。
  3. 监听来自 Compositor 的事件，将其解析（反序列化），并调用应用注册的回调函数。

* libwayland-server: 供**服务器（Compositor）**使用。它做的事情正好相反：监听 socket，接收来自客户端的消息，解析它们，并调用 Compositor 中相应的处理函数。当 Compositor 要发送事件时，它也负责将事件序列化并发送出去。

需要注意的是，libwayland 自身不包含任何图形逻辑。它不知道什么是像素，什么是窗口，什么是 OpenGL。它只是一个纯粹的、轻量级的 IPC（进程间通信）库，严格按照协议规范来打包和解包数据。

协议的实现者才是是真正干活的那个，也就是 Compositor 的实现。不同的桌面环境有不同的 Compositor 实现，它们共享同一个协议，但内部实现和提供的用户体验千差万别。

* Weston: Wayland 协议的官方参考实现。它的代码相对简洁，设计清晰，主要目的是为了展示如何正确地实现一个 Compositor，是学习 Wayland 内部原理的最佳范例。
* Mutter: GNOME 桌面的 Compositor。它功能复杂，与 GNOME Shell 深度集成，负责 GNOME 所有窗口管理和视觉效果。
* KWin: KDE Plasma 桌面的 Compositor。同样功能强大，并且有一个独特之处：它可以同时作为 X11 窗口管理器和 Wayland Compositor 运行。
* Sway: 一个平铺式（Tiling）的 Wayland Compositor，是 i3 窗口管理器的 Wayland 版本替代品，深受键盘流用户的喜爱。

开发一个完美的 Compositor 并不容易，Compositor 作为系统唯一的图形和输入管理者，需要直接与操作系统内核提供的底层接口打交道。

* DRM (Direct Rendering Manager): Linux 内核中与现代显卡通信的子系统。
* KMS (Kernel Mode Setting): DRM 的一部分，专门负责模式设置（Mode Setting）。Compositor 通过 KMS API 可以查询到有多少个显示器连接，每个显示器支持哪些分辨率和刷新率，并可以命令内核将某个特定的**帧缓冲区（Framebuffer）**的内容显示在屏幕上。这就是 Wayland 实现“每一帧都是完美的”（Every frame is perfect）无撕裂显示的关键。
* evdev: 内核将所有输入设备（键盘、鼠标、触摸板、手柄）的原始输入数据都通过 /dev/input/event* 文件暴露出来，这就是 evdev 接口。
* libinput: 一个位于 Compositor 和 evdev 之间的推荐库。它读取 evdev 的原始数据，并将其处理成更高级、更易用的事件。例如，它能将触摸板上的原始坐标点序列识别为“滚动”或“捏拉缩放”手势。Compositor 只需要和 libinput 对话，就免去了处理各种复杂输入硬件细节的麻烦。

协议的使用者，是我们日常运行的各种应用程序。极少数应用会直接链接 libwayland-client 并手动处理 Wayland 对象，绝大多数应用都是通过高级的 GUI 工具包来与 Wayland 交互的。例如：

* GTK 和 Qt 都有 Wayland 后端。当一个 GTK 应用启动时，工具包会检测到 Wayland 环境，并自动使用其 Wayland 后端来创建窗口（wl_surface）、处理输入等。应用开发者写的代码是平台无关的，是工具包在底层完成了与 Compositor 的所有通信。
* SDL: 游戏和多媒体应用常用的库，也支持 Wayland。
* Electron / Chromium: 浏览器和基于 Web 技术的应用也通过其底层的图形引擎（如 Skia）和窗口系统抽象层来支持 Wayland。

一般情况下，应用程序不需要修改就能运行在 X11 或 Wayland 环境下，但凡是都有例外，比如在代码中直接使用了 X11 的 API。为了获得底层的消息和事件，很多应用程序都这么干。这也导致应用程序在迁移到 Wayland 时会出现各种水土不服的问题。

## libwayland 客户端与服务端源码分析

libwayland 的源码可以访问：https://gitlab.freedesktop.org/wayland/wayland。由于 wayland 代码在不断的进化中，为了和本文的分析能对应上，也为了 DeepWiki 能够分析，我在 github 上 fork 了一份代码：https://github.com/mogoweb/wayland。

### libwayland 架构概览

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/weston_intro_02.png)

Wayland 的架构本质上是协议驱动的，这意味着整个系统围绕 XML 协议定义构建，从而驱动自动代码生成。这种方法确保了客户端和合成器之间的类型安全通信，同时保持了协议版本控制和可扩展性。

### 协议处理流水线

Wayland 系统通过定义明确的流水线将 XML 协议规范转换为可执行的 C 代码：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/weston_intro_03.png)

在 protocol 文件夹下，有两个比较重要的文件，也就是上图中的 wayland.xml 和 wayland.dtd。

* wayland.dtd：DTD 代表 Document Type Definition（文档类型定义）。wayland.dtd 文件本身不定义任何 Wayland 的功能。它的作用是定义编写 Wayland 协议 XML 文件的语法规则。规定了一个合法的 Wayland 协议 XML 文件中可以包含哪些标签（如 <protocol>, <interface>, <request>, <event>, <arg>），这些标签可以有哪些属性（如 name, version, type），以及它们之间如何嵌套。
* wayland.xml：这是 Wayland 的核心协议定义文件，遵循 wayland.dtd 定义的语法。它定义了任何 Wayland 系统都必须实现的最基本、最核心的接口和功能。如果我们打开该文件，会发现它非常精简，只有三千多行。核心的接口主要有：
  1. wl_display: 协议的入口点。用于连接、同步和处理错误。
  2. wl_registry: 服务发现机制。客户端通过它来查询服务器（合成器）支持哪些全局对象（即哪些扩展协议可用）。这是 Wayland 动态性的关键。
  3. wl_compositor: 用于创建 wl_surface 对象。
  4. wl_surface: 一个可以被显示在屏幕上的矩形区域。它本身没有窗口标题、边框或位置的概念，只是一个纯粹的“画布”。
  5. wl_shm: 共享内存机制，允许客户端将像素数据高效地传递给合成器。

在上图中，还有扩展协议，这一部分的内容并不在这个 git 库中，而是在 https://gitlab.freedesktop.org/wayland/wayland-protocols 仓库中。

由于核心协议 wayland.xml 非常精简，几乎所有我们认为理所当然的桌面功能都是通过扩展协议来实现的。扩展协议为 Wayland 添加额外的、非核心的功能。这使得 Wayland 保持了高度的模块化和灵活性。合成器可以只实现它所需要的功能。每一个扩展协议都是一个独立的 XML 文件，其语法同样必须遵循 wayland.dtd。在实现 Compoistor 时需要实现哪个扩展特性，可以查看其对应的 xml 定义文件，这里就不展开。

Wayland 使用一套完善的构建系统，可根据 XML 规范生成协议实现代码。wayland-scanner 工具是此过程的核心，它将协议定义转换为类型安全的 C API。

该系统支持多种输出模式，适用于不同的用例：

* server-header：生成服务器端接口定义
* client-header：生成客户端代理函数
* private-code：生成内部协议实现
* public-code：生成公共 API 符号

此架构确保协议变更自动反映在实现中，从而保持规范与代码之间的一致性，同时为协议交互提供编译时类型安全。

### libwayland-server 的核心逻辑

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/weston_intro_04.png)

合成器（服务器）端以 wl_display 对象及其相关子系统为中心，wl_display 结构定义如下：

```c
struct wl_display {
	struct wl_event_loop *loop;
	bool run;

	uint32_t next_global_name;
	uint32_t serial;

	struct wl_list registry_resource_list;
	struct wl_list global_list;
	struct wl_list socket_list;
	struct wl_list client_list;
	struct wl_list protocol_loggers;

	struct wl_priv_signal destroy_signal;
	struct wl_priv_signal create_client_signal;

	struct wl_array additional_shm_formats;

	wl_display_global_filter_func_t global_filter;
	void *global_filter_data;

	int terminate_efd;
	struct wl_event_source *term_source;

	size_t max_buffer_size;
};
```

其中有一个重要的结构 wl_event_loop 和 wl_connection:

```
struct wl_event_loop {
	int epoll_fd;
	struct wl_list check_list;
	struct wl_list idle_list;
	struct wl_list destroy_list;

	struct wl_priv_signal destroy_signal;

	struct wl_timer_heap timers;
};

struct wl_connection {
	struct wl_ring_buffer in, out;
	struct wl_ring_buffer fds_in, fds_out;
	int fd;
	int want_flush;
};
```

这三者之间的关系，可以用一个比喻来概括：

* wl_connection：这是电话线。它负责传输原始的电信号（二进制数据），但它不知道信号代表什么意思。
* wl_display：这是电话机。它连接到电话线，将电信号转换成你能理解的声音（Wayland 消息/事件），并将你的声音转换成电信号（Wayland 请求）。它还管理着你的通讯录（所有已知的 Wayland 对象）。
* wl_event_loop：这是你（或你的秘书）。你负责监听电话机是否响起。当它响起时，你拿起电话（处理事件），然后继续等待下一次铃声。


#### wl_connection - 底层数据传输

wl_connection 是这三者中最低级别的抽象。

* 角色：它直接封装了与 Wayland 合成器（服务器）之间的 Unix 套接字（socket）连接。
* 职责：
  1. 发送数据：将客户端发出的请求（例如，“创建一个表面”）序列化为 Wayland 二进制协议格式，并将其写入套接字。这个过程通常被缓冲，通过 wl_connection_flush() 真正发送出去。
  2. 接收数据：从套接字读取传入的原始字节流。
  3. 解析消息：将读取的字节流解析成一个个独立的 Wayland 消息（事件）。它能识别出消息的长度、目标对象 ID、以及操作码（opcode），但它不理解消息的含义，也不知道该如何处理。它只是把解析好的消息放入一个内部队列中。

简单来说，wl_connection 只关心字节流的发送、接收和正确分包。

#### wl_display - 协议状态机和对象分发中心

wl_display 是客户端应用程序与之交互的主要入口点，也是最高级别的对象。

* 角色：代表了客户端与服务器的整个会话（session）。
* 职责：
  1. 持有连接：一个 wl_display 对象内部包含一个 wl_connection 对象。当你调用 wl_display_connect() 时，它会在后台创建套接字并实例化一个 wl_connection 来管理它。
  2. 对象管理：它维护着一个对象映射表。Wayland 中所有东西都是对象（wl_surface, wl_seat 等），每个对象都有一个唯一的 ID。当 wl_connection 从服务器接收到一个事件时，事件中会包含一个目标对象的 ID。wl_display 就用这个 ID 在自己的映射表中查找对应的本地代理对象（proxy object）。
  3. 事件分发 (Dispatching)：这是它的核心工作。当被要求分发事件时（通过 wl_display_dispatch()），它会：
    * 从其内部的 wl_connection 读取并解析所有待处理的消息。
    * 对于每一个消息（事件），它会查找目标对象。
    * 找到目标对象后，它会调用你为该对象注册的监听器（listener）回调函数。

简而言之，wl_display 赋予了 wl_connection 传来的原始消息以“意义”，并将它们路由到正确的处理程序。

#### wl_event_loop - 事件等待与调度器

wl_event_loop 解决了“何时”去检查新事件的问题。如果没有它，你的程序就得在一个死循环里不停地调用 wl_display_dispatch()，这会消耗 100% 的 CPU。

* 角色：一个通用的事件循环，可以监视多个事件源（不仅仅是 Wayland）。
* 职责：
  1. 监视文件描述符 (File Descriptors)：它的核心功能是使用 epoll 或 poll 等系统调用来高效地等待一个或多个文件描述符（FD）变为“可读”或“可写”。
  2. 集成 Wayland 连接：你可以从 wl_display 获取其底层连接的文件描述符（通过 wl_display_get_fd()），然后将这个 FD 添加到 wl_event_loop 的监视列表中。
  3. 触发回调：当操作系统通知 wl_event_loop 那个 Wayland 连接的 FD 上有数据可读时，wl_event_loop 就会被唤醒，并调用你为这个事件源注册的回调函数。通常，这个回调函数就是 wl_display_dispatch()。

简单来说，wl_event_loop 让你的程序可以“睡眠”，直到有事情发生时才被唤醒，从而高效地处理事件。

这三者协同工作的完整流程：
```
+---------------------------------+
|      你的应用程序代码           |
|  (注册监听器, 发送请求)         |
+---------------------------------+
      | wl_display_roundtrip()
      | wl_surface_attach()
      V
+---------------------------------+  <-- 主要交互层
|          wl_display             |
| (对象管理, 事件分发)            |
|                                 |
|  内部持有:                      | ----> (通过 wl_display_get_fd() 关联) ----+
|  +---------------------------+  |                                         |
|  |      wl_connection        |  |                                         |
|  | (序列化/反序列化, 读/写)  |  |                                         |
|  +---------------------------+  |                                         |
|     |  (拥有 Socket FD)         |                                         V
|     V                         |                                 +------------------------+
+---------------------------------+                                 |      wl_event_loop     |
      |                                                             | (等待 FD 事件, 调用回调) |
      V                                                             +------------------------+
+---------------------------------+
|      Unix Socket (OS 内核)      |
+---------------------------------+
      |  <---- 网络/IPC ---->     |
+---------------------------------+
|       Wayland Compositor        |
+---------------------------------+
```

### libwayland-client 的核心逻辑

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/weston_intro_05.png)

一个典型的 Wayland 客户端（应用程序）遵循一个清晰的、事件驱动的生命周期。可以将其分为以下几个核心步骤：

1. 连接 (Connection)

  * 入口点: 客户端首先调用 wl_display_connect() 来建立与 Wayland 合成器（服务器）的 Unix 套接字连接。

  * 结果: 这会返回一个 wl_display 对象，它是客户端与服务器通信的根对象和主上下文。

2. 发现与绑定 (Discovery & Binding)

  * 核心机制: Wayland 是高度模块化的。客户端在连接后并不知道服务器支持哪些具体功能（如窗口管理、输入设备等）。

  * wl_registry: 客户端从 wl_display 获取 wl_registry（注册表）对象，并为其设置一个监听器（listener）。

  * 接收“广告”: 合成器通过 wl_registry 的 global 事件，向客户端“广播”它所支持的所有全局接口（例如 wl_compositor, xdg_wm_base, wl_seat 等）。

  * 绑定: 当客户端在监听器中收到它感兴趣的 global 事件时，它会使用 wl_registry_bind() 来请求一个这些接口的本地实例（代理对象）。例如，它会绑定 wl_compositor 以便能够创建绘图表面。

3. 创建对象 (Object Creation)

  * 绘图表面: 客户端使用在第2步中绑定的 wl_compositor 对象，调用 wl_compositor_create_surface() 来创建一个 wl_surface。这只是一个空白的、没有边框和标题的矩形“画布”。

  * 赋予“窗口”身份: 仅有 wl_surface 是不够的。客户端需要使用一个 Shell 协议（通常是 xdg-shell）来将这个表面变成一个真正的窗口。它会使用绑定的 xdg_wm_base 对象，为 wl_surface 创建一个 xdg_toplevel 角色，这使得窗口可以被移动、缩放，并拥有标题栏等属性。

  * 获取输入: 客户端使用绑定的 wl_seat 对象来获取输入设备，如 wl_pointer (鼠标) 和 wl_keyboard (键盘)，并为它们注册监听器以接收输入事件。

4. 渲染与更新 (Rendering & Updating) - "Attach & Commit" 模型

  * 创建缓冲区: 客户端创建一块共享内存（通常通过 wl_shm 接口），这块内存将作为它的绘图缓冲区。

  * 绘图: 客户端使用各种库（如 Cairo, Skia, OpenGL/Vulkan）将自己的 UI 绘制到这个缓冲区中。这个过程完全在客户端完成，Wayland 服务器不关心你如何绘图。

  * 附加 (Attach): 客户端调用 wl_surface_attach()，将填充好内容的缓冲区“附加”到它的 wl_surface 上。这相当于告诉合成器：“这是我下一帧要显示的内容。”

  * 提交 (Commit): 客户端调用 wl_surface_commit()。这会原子性地将所有对 wl_surface 的状态更新（包括新附加的缓冲区、尺寸变化等）提交给合成器。合成器收到 commit 后，才会在下一个刷新周期将新的内容显示在屏幕上。

5. 事件循环与处理 (Event Loop & Handling)

  * 核心循环: 客户端的主线程进入一个事件循环。它调用 wl_display_dispatch() 或类似的函数，这个函数会阻塞程序，直到有来自服务器的事件。

  * 事件驱动: 当服务器发送事件时（例如，鼠标移动、键盘按下、窗口需要关闭的请求），wl_display_dispatch() 会被唤醒。

  * 分发: 它会自动查找该事件对应的对象（如 wl_pointer），并调用你之前为该对象注册的监听器中的相应回调函数。

  * 响应: 你的回调函数被执行（例如，处理鼠标点击的逻辑），可能会更新应用状态，并触发一次新的渲染与更新（返回第4步）。

6. 清理与断开 (Cleanup & Disconnection)

  * 当应用程序准备退出时，它必须按相反的顺序销毁它创建的所有 Wayland 对象（xdg_toplevel, wl_surface, 等等）。

  * 最后，调用 wl_display_disconnect() 来优雅地关闭与服务器的连接。

### 消息编组（Marshalling）和传输

Wayland 使用二进制线路协议实现客户端和合成器之间的高效通信：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/weston_intro_06.png)

Wayland 的消息编组（Marshalling）和传输是其高性能和强类型安全设计的核心。

Wayland 的设计哲学是：协议规范（XML 文件）应该能被机器读取并自动生成代码。它摒弃了像 X11 那样复杂的、可扩展的文本协议，转而使用一种紧凑、高效的二进制协议。

这个过程可以分为两大部分：

* 消息编组 (Marshalling)：将一个函数调用（例如 wl_surface_attach(...)）及其参数转换成一段二进制数据。
* 消息传输 (Transport)：将这段二进制数据通过底层机制从客户端发送到合成器（反之亦然）。

假设客户端要创建一个新窗口表面：

1. 开发者调用: struct wl_surface *surface = wl_compositor_create_surface(compositor);
2. 编组 (Marshalling): libwayland-client 库的内部实现执行以下操作：
  * 为新 surface 分配一个 ID（例如 12）。
  * 构建一条 12字节 的二进制消息：
    - Header (8 bytes):
      * 对象 ID: compositor 对象的 ID (例如 2)。
      * 大小: 12 (8字节头 + 4字节参数)。
      * Opcode: 0 (代表 create_surface)。
    - Argument (4 bytes):
      * 新 ID: 12。
    - 将 [0x02000000, 0x0c000000, 0x0c000000]（简化表示）这样的二进制数据放入发送缓冲区。
3. 传输 (Transport):
  * 在某个时间点（例如调用 wl_display_flush()），整个缓冲区的内容通过 write() 系统调用写入 Unix Domain Socket。
4. 解组 (Unmarshalling) 在合成器端:
  * 合成器的事件循环检测到套接字上有数据可读。
  * libwayland-server 读取并解析该二进制消息。
  * 它发现这条消息是发给对象 ID 为 2 的，操作码为 0。
  * 它查询其内部的派发表，知道这对应于 wl_compositor 接口的 create_surface 请求。
  * 它调用合成器为该请求注册的处理函数，并将参数（新 ID 12）传递给它。
5. 完成: 合成器在内部创建一个代表新窗口表面的资源，并将其与 ID 12 关联起来，等待客户端的后续操作（如附加缓冲区和提交）。

通过这个编组 -> 二进制传输 -> 解组的流程，Wayland 实现了高效、类型安全且低延迟的通信。

## Wayland 的隐忧

通过前面的分析，我们会发现，Wayland 核心协议（wayland.xml）本身对窗口管理、输入/输出策略等高级功能几乎没有定义。

它完全没有规定：

* 窗口应该长什么样？ (是否有标题栏、边框？)
* 窗口如何移动、缩放或最大化？
* 如何处理多个窗口的层叠关系（z-order）？
* 如何实现 Alt+Tab 切换窗口？
* 如何定义快捷键？
* 系统的外观和感觉（Look and Feel）是怎样的？

Wayland 将所有这些“策略”性的决策权都下放给了 Wayland Compositor（合成器）。这意味着：

* GNOME 的 Mutter 可以实现一套它认为最佳的窗口管理和交互逻辑。
* KDE 的 KWin 可以实现另一套完全不同的逻辑，提供更多的定制化选项。
* Sway (一个平铺式窗口管理器) 可以实现完全没有浮动窗口的逻辑。
* 一个用于汽车中控系统的合成器可以实现一套专门为触摸优化的、没有传统“窗口”概念的界面。
* 一个用于机顶盒的合成器可能只允许一个全屏应用运行。

这种设计有些优点：

* 极大的灵活性和创新空间：每个桌面环境或嵌入式系统可以根据自己的目标用户和使用场景，自由地创新和实现最合适的交互体验，而不受一个统一、死板的协议限制。这是 Wayland 能够适用于从桌面到移动设备再到嵌入式等多种场景的关键。
* 简洁性和职责分离：核心协议保持稳定和精简。复杂的策略性问题留给更上层的软件（合成器）去解决，这符合软件工程的单一职责原则。
* 避免“委员会式设计”：X11 的很多扩展（如 EWMH）是为了在众多独立的窗口管理器之间达成行为共识而制定的，过程漫长且结果复杂。Wayland 则认为“合成器即是窗口管理器”，所以它自己决定一切，不需要复杂的协商。

这种灵活性也带来了一个显而易见的问题：如果每个合成器都各自为政，应用程序如何知道怎样与它们交互以实现基本功能（如创建一个正常的窗口）？

为了解决这个问题，社区（主要通过 freedesktop.org）制定了一系列被广泛接受的扩展协议，以提供跨桌面环境的通用功能。其中最重要的就是：

* xdg-shell：这个协议是事实上的标准，用于创建传统的桌面应用程序窗口。它定义了“顶层窗口（toplevel）”和“弹出窗口（popup）”的概念，并提供了请求最大化、最小化、设置标题等功能。任何想要在主流桌面（GNOME, KDE, Sway 等）上运行的通用应用程序，都必须支持 xdg-shell。
* xdg-decoration：用于协商窗口边框由谁来绘制（客户端还是服务器）。
* 其他 xdg-* 协议：定义了如激活窗口、启动通知等桌面交互。

前面说过，扩展协议定义在 wayland-protocols 仓库中，查看一下 wayland-protocols 代码结构，就可以看出，扩展协议可太多了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/weston_intro_07.png)

目前稳定的协议只有 5 个，大部分都处在 staging 和 unstable 阶段。这会带来一个新的问题，客户端或服务器端对扩展协议支持不完善，甚至支持的版本不同，两者之间如何和谐相处？

为此，wayland 设计了一套机制来处理客户端和服务器端不匹配的情况，确保它们能够“和谐相处”。这个机制的核心就是 wl_registry 和明确的版本协商规则。

1. 核心机制：wl_registry 服务发现

当一个 Wayland 客户端连接到合成器时，它做的第一件事就是获取 wl_registry 对象并监听它的事件。wl_registry 就像一个“服务黄页”，合成器通过它来广播自己支持的所有功能（全局接口）。

* 服务器广播 (Advertisement)：对于每一个支持的扩展协议（例如 xdg_wm_base），合成器会通过 wl_registry 发送一个 global 事件。这个事件包含三个关键信息：
  - Name (uint): 一个唯一的数字 ID，用于在后续操作中引用这个全局对象。
  - Interface (string): 接口的名称，例如 "xdg_wm_base"。
  - Version (uint): 该服务器支持此接口的最高版本号。

2. 客户端的响应与协商

客户端会监听这些 global 事件。当它看到自己需要或认识的接口时，它会决定是否要使用它。

* 客户端请求 (Binding)：如果客户端决定使用这个接口，它会调用 wl_registry_bind() 函数。这个函数也需要三个关键参数：
  - Name (uint): 从 global 事件中收到的那个唯一 ID。
  - Interface (struct): 客户端代码中对应的接口结构体（例如 &xdg_wm_base_interface）。
  - Version (uint): 客户端希望使用的版本号。通常，这是客户端在编译时所依赖的最高版本。

3. 和谐相处的关键规则：版本协商

现在，最关键的一步发生了。当 wl_registry_bind() 被调用时，客户端和服务器会根据以下简单而明确的规则，就最终使用的协议版本达成一致：

最终使用的版本 = MIN (服务器支持的最高版本, 客户端请求的版本)

这个规则完美地解决了版本不匹配的问题：

场景一：服务器版本更高

* 服务器支持 xdg-shell v4。
* 客户端是旧程序，编译时只知道 xdg-shell v3，于是它请求 v3。
* 结果：协商成功，双方将使用 v3 协议进行通信。客户端可以正常工作，只是无法使用 v4 中新增的功能。

场景二：客户端版本更高

* 服务器是旧程序，只支持 xdg-shell v2。
* 客户端是新程序，编译时依赖 xdg-shell v4，于是它请求 v4。
* 结果：协商成功，双方将使用 v2 协议进行通信。

4. 客户端如何处理协商结果？

协商完成后，客户端必须根据实际得到的版本来调整自己的行为。这通常有三种策略：

a) 功能优雅降级 (Graceful Degradation)

这是最常见和最理想的情况。

* 在上面的场景二中，客户端请求了 v4，但实际得到的版本是 v2。
* 客户端代码会检查实际绑定的版本号。
* 如果它发现版本低于预期，它就会禁用那些只有在 v3 或 v4 中才存在的功能。例如，如果 v3 增加了一个“设置窗口最小尺寸”的请求，而当前协商的版本是 v2，那么客户端就不应该去调用这个请求，或者相关的 UI 选项会变灰。
* 结果：应用程序的核心功能仍然可用，只是缺少一些新特性。用户体验依然流畅。

b) 硬性要求与优雅退出 (Hard Requirement & Graceful Exit)

有时候，一个扩展协议或它的某个最低版本对于应用程序是绝对必要的。

* 场景: 一个屏幕录制工具可能硬性要求合成器支持 org_kde_kwin_screencast 或 xdg-desktop-portal 的某个版本，否则它根本无法工作。
* 客户端逻辑:
  1. 客户端在 wl_registry 监听器中检查是否存在所需的接口。
  2. 如果接口存在，客户端尝试绑定它，并检查协商后的版本是否 _>= 它所需的最低版本。
  3. 如果不满足任一条件，客户端不应该崩溃，而应该优雅地退出，并向用户显示一条清晰的错误信息，例如：“启动失败：当前的 Wayland 合成器不支持屏幕录制所需的功能。”
  4. 结果：避免了程序在运行时因调用不存在的函数而崩溃，为用户提供了有用的反馈。

c) 完全不支持 (No Support)

* 场景：客户端想要使用一个可选的增强功能，比如一个自定义的窗口装饰协议，但服务器完全不支持。
* 客户端逻辑：在 wl_registry 的事件中根本找不到对应的接口。
* 结果：客户端 просто忽略它，就像这个功能从未存在过一样，使用默认的回退方案（例如，让服务器自己去画窗口边框）。

虽然这种协商机制允许服务器和客户端以不同的速度演进，但很明显，最终实现效果非常依赖客户端和服务器端的实现，导致不同实现（不同桌面环境）在视觉效果和交互体验上差异很大，甚至无法工作（比如协商结果是 No Support）。

在 Linux 社区，由于缺乏一个强有力的领导者，各自为战，导致了严重的碎片化。GNOME 的 Mutter 和 KDE 的 KWin 在功能实现上差异很大，很多应用在一个桌面环境下能用，在另一个环境下就有问题。

## Wayland 的前景

虽然 Wayland 面临着挑战，总的来说，Wayland 的前景非常光明，它已经越过了最艰难的“引爆点”，正在成为 Linux 桌面的既定未来标准。但这个过程确实比许多人最初预期的要漫长和复杂。

* 两大巨头的成熟：GNOME (Mutter) 和 KDE (KWin) 作为两大主流桌面环境，它们的 Wayland 实现已经非常成熟和稳定。对于绝大多数日常使用场景（办公、影音、网页浏览、轻度游戏），它们提供的体验已经追平甚至超越了 X11。
* 事实标准的形成：xdg-shell 等关键扩展协议已经被所有主流合成器实现，为应用程序提供了一个可靠的、跨桌面环境的开发目标。开发者不再需要为 GNOME 和 KDE 分别适配核心功能。
* NVIDIA 的追赶：在社区和 Valve（为了 Steam Deck）的巨大压力下，NVIDIA 近年来在 Wayland 支持上取得了长足的进步。虽然仍有一些小问题，但已经不再是阻止用户尝试 Wayland 的主要障碍。
* 生态工具的完善：像 PipeWire 这样的项目解决了 Wayland 安全模型下屏幕共享和音频处理的难题，为 OBS、浏览器等应用提供了标准化的接口。

市场已经围绕 GNOME 和 KDE 这两大实现 形成了强大的向心力，它们为整个生态系统设定了基准。

### Wayland 的前景展望

* 不可逆转的趋势：几乎所有主流 Linux 发行版（Fedora, Ubuntu, Arch 等）和桌面环境都在将 Wayland 作为默认选项。X11 已经进入纯维护模式，不会再有新功能开发。这意味着整个 Linux 图形生态的开发重心和资源都在向 Wayland 倾斜。开发者和用户除了跟随，别无选择。
* 游戏体验的提升：Valve 的 Steam Deck 完全基于 Wayland (Gamescope 合成器)，这极大地推动了 Wayland 在游戏领域的优化。Wayland 原生支持可变刷新率（VRR）、更低的输入延迟以及对现代图形 API 的更好集成，长远来看，它将为 Linux 游戏提供比 X11 更优越的平台。
* 安全性的根本优势：随着人们对隐私和安全越来越重视，Wayland 的隔离式架构这一根本优势将变得越来越重要。在 X11 中，任何一个恶意应用都可以轻易地成为键盘记录器，这在 Wayland 中是不可能的。
* 填补最后的功能差距：剩下的难题，如更完善的色彩管理、远程桌面协议、某些特殊的窗口管理功能等，都在通过新的扩展协议逐步解决。虽然需要时间，但方向是明确的。

虽然路途坎坷，但方向已定。Wayland 不是会不会成功的问题，而是还需要多长时间来彻底完善并取代 X11 在所有角落的存在。目前看来，这一天已经不远了。

如果你坚持看到这里了，那一定是这个领域的专家。我个人接触这块工作的时间并不长，很多资料都是查询 AI 得到的，希望 AI 没有胡说八道，如果有什么疏漏之处，还请指点一二。

