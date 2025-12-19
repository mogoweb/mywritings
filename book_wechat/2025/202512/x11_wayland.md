# 从 X11 的角度理解 Wayland

在 Linux 桌面领域，Wayland 常被提及为 “X11 的继任者”。事实上，各大主流 Linux 发行版都在加速推进去 “X11 化”，国产桌面系统也在积极转向 Wayland 作为默认显示协议。

然而，与这一趋势形成鲜明对比的是：**Wayland 相关的开发资料却异常稀少**。除了官方文档以及《The Wayland Protocol》等少数教程外，系统性、成体系的资料并不多，这与 Windows 平台下丰富的图形系统开发书籍形成了明显反差。

我认为，造成这一现象的一个重要原因在于：

**绝大多数应用开发者并不直接面对显示协议层**，而是通过 Qt、GTK 等成熟的 GUI 框架完成开发，既不需要理解 X11，也无需关心 Wayland 的内部机制。

但并非所有项目都如此。某些底层或跨平台的开源项目（例如 Wine），为了避免引入体量庞大的 GUI 框架依赖，其图形子系统往往**直接基于 X11 API 或 Wayland 协议实现**。在这种场景下，就必须理解底层显示系统。

在研究 `winewayland.drv` 的实现过程中，很自然地会将其与 `winex11.drv` 的实现进行对比。正是在这种对照之下，Wayland 与 X11 在设计理念上的差异显得尤为突出。因此，本文尝试**站在 X11 的视角**，借助熟悉的概念、对象和调用流程，逐步拆解 Wayland 中的核心对象与接口，希望对读者有所帮助。

## Display vs wl_display

在 X11 程序中，几乎所有代码都会从这样一句开始：

```c
Display *dpy = XOpenDisplay(NULL);
```

在 X11 体系中，`Display` 对象具有非常核心的地位，它代表了：

* 客户端与 X Server 之间的连接
* X11 所有 API 的根对象
* 底层通信通道
* Screen / Visual / Depth 等全局显示状态

可以说，在 X11 编程中，`Display *` 无处不在，绝大多数 Xlib API 都需要将其作为第一个参数传入。

在 Wayland 中，也存在一个在“形式上”与之相似的对象 —— `wl_display`：

```c
struct wl_display *display = wl_display_connect(NULL);
```

但需要特别注意的是，**`wl_display` 的职责要比 X11 中的 `Display` 明显少得多**。它本质上只是客户端与 compositor 之间的一个**纯连接对象**，主要负责：

* Wayland 协议消息的发送与接收
* 协议对象 ID 的分配与管理
* 事件的分发与调度

与 X11 不同的是，`wl_display` **不包含任何显示设备、窗口层级或 UI 语义相关的信息**。在 Wayland 编程中，这一点需要时刻牢记：

**不要期待像 X11 那样，通过一个全局对象就能获取窗口、屏幕或视觉信息。**

## Window vs Surface

在 X11 中，`Window` 是一个极其重要的对象。只要是图形应用程序，几乎必然会与 `Window` 打交道：

```c
Window w = XCreateWindow(...);
XMapWindow(dpy, w);
```

在 X11 语义下，一个 `Window` 同时承担了多种职责：

* 几何属性（位置、大小）
* 绘制目标（Drawable）
* 事件接收器
* 窗口层级关系（parent / child）
* 与窗口管理器（WM）交互的载体

可以说，`Window` 是一个高度“复合”的对象。

而在 Wayland 中，只有一个对象在形式上可以与之类比，即 `wl_surface`：

```c
struct wl_surface *surface = wl_compositor_create_surface(...);
```

初次接触 Wayland 时，最难理解的一点往往在于：

**`wl_surface` 本身既没有位置、也没有大小，更没有窗口类型的概念**。它只表示一块可以向 compositor 提交 buffer 的平面。

那么，Wayland 中的“窗口”究竟是如何定义的？

答案是：**通过为 `wl_surface` 赋予角色（role）**。
这些角色并不定义在 Wayland 核心协议中，而是由扩展协议提供，其中最重要、也是最通用的一个扩展协议就是 **XDG shell**。

XDG（cross-desktop group，跨桌面工作组）shell 是 Wayland 的一个**标准协议扩展**，它为 `wl_surface` 定义了两种最核心的角色（role）：

* **toplevel**：用于应用程序的顶层窗口
* **popup**：用于上下文菜单、下拉菜单、工具提示等场景，这类 surface 通常依附于某个顶层窗口

通过这两种角色，客户端可以构建一棵由 surface 组成的树结构：

* 树的根节点是一个 **toplevel**
* 叶子节点可以是 **popup**，也可以是额外的 **toplevel**

需要强调的是：

这种“树结构”并不是 X11 中那种由 Server 维护的全局窗口树，而是**由协议语义明确约束的 surface 关系模型**。

经过 XDG shell 的角色定义之后，Wayland 中才逐步具备了与 X11 类似的“窗口层级”概念，但这种层级是**显式、受限且由 compositor 主导的**，这正是 Wayland 与 X11 在设计理念上的根本差异之一。

## Pixmap vs wl_buffer

在 X11 中，`Pixmap` 由客户端通过如下接口创建：

```c
Pixmap XCreatePixmap(Display*, Drawable, width, height, depth);
```

但需要注意的是，**Pixmap 虽然由客户端发起创建，其实际归属却在 X Server 侧**。它本质上是由服务器管理的一块像素存储资源，可以在 X11 体系中承担多种角色，包括：

* 绘制目标（`Drawable`）
* 图形上下文（GC）的操作对象
* 窗口的后备存储（backing store）
* 复合管理器（Composite）中的 source / mask

在典型的 X11 绘制模型中，客户端并不直接操作像素数据，而是不断向 X Server 发送绘制指令，由服务器在 Pixmap 上执行这些命令，例如：

* `DrawLine`
* `CopyArea`
* `FillRectangle`
* 等等

也就是说，**Pixmap 是一个可被反复修改的服务器端状态对象，其内容由一系列绘制命令逐步累积而成**。

而在 Wayland 中，对应的概念是 `wl_buffer`。与 Pixmap 不同，`wl_buffer` 是**客户端提供给合成器的像素缓冲描述**，它本身不包含任何绘制接口，也不承载绘制语义。

在 Wayland 的设计中，**渲染完全由客户端自行完成**：客户端先在本地将一帧内容绘制到 buffer 中，通常“一帧对应一个 buffer”，然后通过如下方式提交给合成器：

```c
wl_surface_attach(surface, buffer, 0, 0);
wl_surface_commit(surface);
```

对于合成器而言，`wl_buffer` 是一个纯粹的输入结果。
它**不关心这块 buffer 是如何生成的**（CPU 绘制、OpenGL、Vulkan、零拷贝等），只关心“当前这一帧应该被合成并显示成什么样子”。

## XNextEvent vs wl_display_dispatch

在 X11 中，典型的事件循环是这样的：

```
XEvent ev;
while (1) {
    XNextEvent(dpy, &ev);
    handle_event(&ev);
}
```

事件是结构化数据（XEvent），由应用显式拉取并分发。XNextEvent() 做的事情有：

* 如果本地事件队列为空，则阻塞
* 从 X Server 读取事件
* 将事件放入客户端队列
* 取出一个 XEvent 返回给调用者

在 Wayland 中，事件处理模型发生了根本变化。代码看起来和 X11 差不多：

```
while (wl_display_dispatch(state.wl_display)) {
    /* This space deliberately left blank */
}
```

但是，在 Wayland 中没有“事件结构体”，事件不会以返回值的形式交给应用，而是在 dispatch() 内部被直接分发，具体来说，就是调用对应对象上注册的 listener 回调。

所以在 Wayland 上的处理流程是：

* 如果 socket 上没有数据，则阻塞等待
* 从 compositor 读取协议消息
* 解析消息
* 直接调用对应对象上注册的 listener 回调

## XCB / Xlib vs libwayland-client

无论是 X11 还是 Wayland，本质上都只是一套**显示协议**。对于大多数开发者而言，通常并不希望直接面对繁琐、低层的协议交互细节，因此才有了 **Xlib / XCB** 以及 **libwayland-client** 这样的客户端开发库。它们的定位都偏向“**协议访问层**”，而非像 Qt / GTK 那样试图提供完整的 GUI 框架解决方案。

Xlib 诞生于 X11 生态的早期阶段，其设计目标是为应用程序提供一套**完整且易用的客户端 API**，尽可能隐藏 X11 协议细节。它在内部维护了大量隐式状态，使得应用开发变得相对简单，但也因此在可控性、并发模型以及现代硬件适配方面逐渐显露出局限性。

XCB 则被视为对 Xlib 的一次现代化重构。它采用了更“薄”的封装方式，与 X11 协议结构高度贴合，强调**异步通信、显式控制和线程友好性**。通过 XCB，开发者可以更精细地掌控请求、回复和事件的生命周期，这也使其更适合作为底层组件或复杂系统中的基础库。

相比之下，libwayland-client 是 Wayland 体系中的原生客户端库，其设计目标更加克制而明确：**仅提供最小且通用的协议运行与调度能力**。它不试图抽象“窗口”、不提供任何绘制或 GUI 相关接口，而是专注于对象管理、协议消息收发以及回调分发。窗口语义、渲染方式以及交互逻辑，全部由上层协议（如 xdg-shell）或应用自身来定义和实现。

从这个角度看，Xlib / XCB 更像是“围绕 X11 协议逐步堆叠出来的客户端 API”，而 libwayland-client 则是一个**刻意保持最小化的协议运行时**，这也正体现了 X11 与 Wayland 在设计理念上的根本差异。

## X Server vs Compositor

在展开对比之前，事先说明一下，现在 X 系统中也有合成器。

早期 X11 中，X Server 直接将 Window 内容绘制到屏幕，不支持透明、阴影或动画效果。随着桌面体验需求的提升，X11 通过一系列扩展（XComposite、XDamage、XRender 等）引入了合成能力。在这种架构下，合成器与窗口管理器一样，都是运行在 X Server 之上的普通客户端进程：

```
[ X Server ]
     ↑
[ Window Manager + Compositor ]
     ↑
[ Applications ]
```

但本文要对比的，并不是“有没有合成器”，而是 X Server 与 Wayland Compositor 在系统中的架构地位和职责边界。

在传统 X11 模型中，X Server 承担了极其繁重且集中的职责：

1. 绘制执行

  * 接收来自客户端的绘制命令（DrawLine、FillRectangle 等）

  * 在 Window / Pixmap 上执行绘制

  * 管理字体、GC、光栅化逻辑

2. 窗口系统核心

  * 管理 Window 对象及其层级关系

  * 维护几何信息、裁剪区域、可见性

  * 处理 Map / Unmap / Reparent

3. 输入设备管理

  * 键盘、鼠标等输入事件采集

  * 焦点管理

  * 事件分发到对应 Window

4. 显示与输出

  * 屏幕、Visual、Depth

  * 多显示器（RandR）

  * 模式切换

5. 扩展平台

  * XRender / XComposite / XDamage / GLX

  * 通过扩展不断“打补丁”增强能力

Wayland 的设计目标之一，就是彻底削减中心节点的复杂度。在 Wayland 中，合成器的职责被刻意限制为：

1. 缓冲合成

  * 接收客户端提交的 wl_buffer

  * 组合多个 surface

  * 输出到显示设备

2. 输入仲裁

  * 决定输入事件应发送给哪个 surface

  * 不解释应用语义

3. 显示调度

  * 帧时序控制（vsync）

  * 输出设备管理

  * scale / transform

4. 窗口语义的执行者

  * 执行 xdg-shell 等协议定义的规则

  * 但语义本身来自协议，而非内建

在这里，窗口管理器的角色也发生了变化：

* 在 X11 中，窗口管理器是“外部组件”，窗口管理器与 X Server 是松耦合甚至对抗式的关系。
* 在 Wayland 中，合成器即窗口管理器，窗口管理逻辑是其内建职责的一部分。

## 小结

通过以上对照可以看到，Wayland 并不是在 X11 之上“修修补补”，也不是简单地“换一个 API 名字”，而是一次从架构层面重新定义显示系统边界的尝试：

* X11 以 Server 为中心，绘制、状态和策略高度集中

* Wayland 将绘制责任下放到客户端，只保留最小、清晰的合成与调度核心

* X11 依赖隐式状态与扩展演进

* Wayland 依赖显式协议与受控语义

从 X11 的角度理解 Wayland，最大的收获并不在于记住新的 API，而在于意识到：

> 显示系统不再是替应用画图的服务器，而是协调各个应用提交结果的调度者。

正是建立起这样的认知框架，才能避免简单地将 `winex11` 的实现思路机械地平移到 `winewayland` 之上。只有在理解 Wayland 设计初衷的前提下，才能读懂 `winewayland` 中的实现是对 Wayland 架构原则的自然遵循。
