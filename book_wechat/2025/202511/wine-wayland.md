# 探索纯正的 wine-wayland： 如何让 Windows 应用运行在纯 Wayland 桌面环境

在《[干得漂亮，Ubuntu终于干掉了 X11]()》这篇文章中，介绍了 Ubuntu 25.10 正式弃用 GNONE 的 Xorg/X11 会话，只支持 Wayland。无独有偶，另一大主流 Linux 发行版 Fedora 也宣布从 Fedora 43 起，桌面环境切换为 Wayland-only。 目前，大多数 Linux 版本已经将 Wayland 设为默认，经过了这几年的检验，终于可以向 X11 说声再见。

作为 Linux 桌面的两大支柱，GNOME 和 KDE Plasma 都已完成了对 Wayland 的适配与优化。那么问题来了：桌面准备好了，**应用程序是否也准备好了？**

本文将带你一起探索 Wine 项目在纯 Wayland 环境下的表现，看看它如今的 Wayland 支持究竟到了什么程度。

这次测试，我使用的是 Ubuntu 24.04 的系统，Wine 则是自行编译的最新版本 10.18：

```
$ wine --version
wine-10.18-229-g0fc31f45624
```

Ubuntu 24.04 没有完全移除 X11，首先确认一下当前桌面是运行在 Wayland 下：

```
$ echo $XDG_SESSION_TYPE
wayland
```

运行一个自行编译的 Windows 窗口应用程序：

```
$ wine wine-wayland.exe 
```

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202511/images/wine_wayland_01.png)

程序是运行起来了，但是使用 xwininfo 一检查，还是运行在 X11 下：

```
$ xwininfo

xwininfo: Please select the window about which you
          would like information by clicking the
          mouse in that window.

xwininfo: Window id: 0x1c00003 "wine wayland"

  Absolute upper-left X:  257
  Absolute upper-left Y:  283
  Relative upper-left X:  14
  Relative upper-left Y:  49
  Width: 392
  Height: 266
  Depth: 24
  Visual: 0x40
  Visual Class: TrueColor
  Border width: 0
  Class: InputOutput
  Colormap: 0x1a00001 (not installed)
  Bit Gravity State: NorthWestGravity
  Window Gravity State: NorthWestGravity
  Backing Store State: NotUseful
  Save Under State: no
  Map State: IsViewable
  Override Redirect State: no
  Corners:  +257+283  -1271+283  -1271-531  +257-531
  -geometry 392x266+257+283
```

这是怎么一回事？

这是因为 GNOME 在“移除 X11”的同时，仍保留了 **XWayland** ——一个运行在 Wayland 下的 X11 兼容层。它的存在意义在于：当桌面环境过渡到 Wayland 时，旧的 X11 应用仍能继续运行。

这种“过渡性兼容”是必要的，否则就会陷入“桌面等应用、应用等桌面”的循环。可以预见，**XWayland 将在很长一段时间内继续存在**，直到绝大多数桌面应用都完成向 Wayland 的迁移。届时，它的去留恐怕又会掀起一场争论，就像今天 GNOME 和 KDE Plasma 移除 X11 支持一样。

那 Wine 对 Wayland 的支持如何呢？目前 Wine 项目对 Wayland 支持标记为实验性支持，也就是说还不完善，但可以用。其实 Wine 对 WOW64 也是标记为实验性支持，但我们项目组使用下来，完成度已经很高。Wine 对 Wayland 的支持程度如何？让我们开启验证一下。

要开启纯 Wayland 支持，先将如下注册表信息保存为文件 (比如 wine-wayland.reg)：

```
Windows Registry Editor Version 5.00
[HKEY_CURRENT_USER\Software\Wine\Drivers]
"Graphics"="wayland"
```

然后运行 wine regedit，打开注册表编辑器，导入上述文件：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202511/images/wine_wayland_02.png)

然后再运行

```
$ wine wine-wayland.exe
0168:err:waylanddrv:wayland_process_init Wayland compositor doesn't support optional zwlr_data_control_manager_v1 (clipboard functionality will be limited)
0168:err:waylanddrv:wayland_process_init Wayland compositor doesn't support xdg_toplevel_icon_manager_v1 (window icons will not be supported)
0160:err:waylanddrv:wayland_process_init Wayland compositor doesn't support optional zwlr_data_control_manager_v1 (clipboard functionality will be limited)
0160:err:waylanddrv:wayland_process_init Wayland compositor doesn't support xdg_toplevel_icon_manager_v1 (window icons will not be supported)
```

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202511/images/wine_wayland_03.png)

窗口是显示出来了，但是存在两个问题：

1. 窗口样式并未遵循 Ubuntu 系统主题，而是使用了 Windows 自绘边框，看起来格格不入；
2. 我随意拉伸了一下窗口，就成这个样子了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202511/images/wine_wayland_04.png)

如果说这么简单的窗口都还存在问题，那说明 Wine 对 Wayland 的支持还相当不完善。这显然并不是应用程序一方的问题。这涉及到 Wayland 的设计哲学。

Wayland 的核心协议（`wayland.xml`）只是提供最基本的接口，比如：

* 创建窗口（surface）
* 接收输入（keyboard, pointer）
* buffer 提交

但像窗口管理（`xdg_shell`）、桌面 portal、拖放、输入法、色彩管理等，都通过 **扩展协议**（extension protocols）实现。

客户端在连接 Wayland socket 后，会通过：

```c
wl_registry_bind()
```

向合成器注册它所支持的接口。
合成器只会暴露自己支持的接口列表（如 `xdg_wm_base`, `wl_data_device_manager`, `zwp_linux_dmabuf_v1` 等）。
客户端要根据是否能 `bind` 成功，来决定启用哪些特性。

不同桌面环境/合成器（如 GNOME Mutter、KDE KWin、Sway、Weston）往往支持**不同的协议扩展版本**，例如：

* KDE 可能实现了 `xdg_decoration_unstable_v1`
* GNOME 则使用 `gtk_shell` 自家协议
* Sway 使用 `layer_shell` 来支持面板和 dock

这导致同一个应用在不同桌面上行为不一致。开发者不得不为多个协议写适配层，甚至要“猜”当前合成器的行为。

Wayland 没有像 X11 那样的“强制核心协议”，而是一个**框架**，这会导致一系列问题：

* 协议处于 `unstable` 或 `private` 状态时，要不要支持；
* 客户端支持了 unstable 版本后，协议演变到正式版本后，应用程序要不要保留 unstable 版本的支持，是否需要降级处理，服务器端也面临同样的抉择；
* 应用的行为表现依赖于合成器的实现。

目前 Wayland 仍在快速演化中，新的实验性协议层出不穷，旧的接口也在陆续稳定。或许，要等 Wayland 像 X System 一样迭代到“第 11 版”，Wayland 才能迎来真正的稳定。

## 小结

Wayland 无疑是 Linux 图形系统的未来，但它仍在成长之中。Wine 的 Wayland 支持虽已初见雏形，但距离完全替代 X11 还有一段路要走。

因此，**在现阶段，仍不建议在纯 Wayland 环境下运行 Wine 应用**。对 Wayland 的探索值得期待，但现在，它更像是一场未完的实验。