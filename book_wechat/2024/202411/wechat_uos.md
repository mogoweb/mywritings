# 点赞！微信4.0原生国产系统版本和 Windows、 MacOS 版本同步发布，还支持龙芯、麒麟等国产芯片

在此前的文章《[国产芯片+国产操作系统打造办公系统](https://mp.weixin.qq.com/s/6vI8zfO2okOyGR1aqNzEAg)》中，我分享了在 UOS 系统上工作的体验。尽管未提及微信，原因是微信主要定位于移动通讯软件。但在工作过程中，微信 PC 版的使用频率还挺高，比如在和外部客户、合作伙伴沟通，很多时候都是通过微信。在电脑上使用，打字，收发文件，都比在手机上方便很多。

尽管 UOS 应用商店早早上线了微信，最初的版本却是基于 Wine 运行的 Windows 版，随后微信虽推出了 Linux 版，但功能十分简陋，几乎让人联想到早期的 QQ for Linux。长期以来，微信的路线是优先支持苹果、Windows 和安卓平台，Linux 版本几乎不受关注。

昨天打开 UOS 应用商店时，看到大幅推荐画面，微信 4.0 原生版竟然同步于 Windows 和 macOS 发布，且功能毫无删减。更值得称赞的是，它还支持 LoongArch 64 和 ARM 架构，这意味着微信已经可以在龙芯、麒麟、飞腾等国产芯片上原生运行。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202411/images/wechat_uos_01.png)

从应用商店安装之后，打开微信，和 Windows 版本一样，界面简洁，功能齐全。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202411/images/wechat_uos_04.png)

查看版本，直接来到了 4.0.0.23，和 Windows 版本一样。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202411/images/wechat_uos_03.png)

龙芯 CPU 发展这么多年，生态一直不太行，比如 UOS 龙芯应用商店，里面的应用比 UOS x86就少很多，更别说国外的软件。这次微信同步发布各种架构的版本，算是对国产 CPU、国产系统的大力支持。

微信这次能够做到全平台的支持，据说是因为采用 QT 进行了重构。之前 QQ 也是进行过重构，但采用的是 Electron 框架。微信和 QQ 是采用不同的框架吗？带着这样的疑问，我进到微信的程序目录下，发现并没有 Qt 相关的动态库，使用 ldd 命令查看：

```
uos@uos-loongsun-PC:/opt/apps/com.tencent.wechat/files$ ldd wechat
        linux-vdso.so.1 (0x000000fffce68000)
        libglib-2.0.so.0 => /lib/loongarch64-linux-gnu/libglib-2.0.so.0 (0x000000ffed278000)
        libatomic.so.1 => /lib/loongarch64-linux-gnu/libatomic.so.1 (0x000000ffed268000)
        libXcomposite.so.1 => /lib/loongarch64-linux-gnu/libXcomposite.so.1 (0x000000ffed25c000)
        libXrender.so.1 => /lib/loongarch64-linux-gnu/libXrender.so.1 (0x000000ffed248000)
        libXrandr.so.2 => /lib/loongarch64-linux-gnu/libXrandr.so.2 (0x000000ffed234000)
        libandromeda.so => /opt/apps/com.tencent.wechat/files/./libandromeda.so (0x000000ffecd68000)
        libconfService.so => /opt/apps/com.tencent.wechat/files/./libconfService.so (0x000000ffecb18000)
        libilink2.so => /opt/apps/com.tencent.wechat/files/./libilink2.so (0x000000ffec924000)
        libilink_network.so => /opt/apps/com.tencent.wechat/files/./libilink_network.so (0x000000ffec108000)
        libilink_protobuf.so => /opt/apps/com.tencent.wechat/files/./libilink_protobuf.so (0x000000ffec0c4000)
        libowl.so => /opt/apps/com.tencent.wechat/files/./libowl.so (0x000000ffebf90000)
        libvoipChannel.so => /opt/apps/com.tencent.wechat/files/./libvoipChannel.so (0x000000ffebd78000)
        libvoipCodec.so => /opt/apps/com.tencent.wechat/files/./libvoipCodec.so (0x000000ffeaa04000)
        libvoipComm.so => /opt/apps/com.tencent.wechat/files/./libvoipComm.so (0x000000ffea9b0000)
        libWxH264.so => /opt/apps/com.tencent.wechat/files/./libWxH264.so (0x000000ffea750000)
        libwxtrans.so => /opt/apps/com.tencent.wechat/files/./libwxtrans.so (0x000000ffe8dc0000)
        libmmmojo.so => /opt/apps/com.tencent.wechat/files/./libmmmojo.so (0x000000ffe8b44000)
        libz.so.1 => /lib/loongarch64-linux-gnu/libz.so.1 (0x000000ffe8b20000)
        libdl.so.2 => /lib/loongarch64-linux-gnu/libdl.so.2 (0x000000ffe8b14000)
        libxkbcommon.so.0 => /lib/loongarch64-linux-gnu/libxkbcommon.so.0 (0x000000ffe8acc000)
        libxkbcommon-x11.so.0 => /lib/loongarch64-linux-gnu/libxkbcommon-x11.so.0 (0x000000ffe8abc000)
        libxcb-glx.so.0 => /lib/loongarch64-linux-gnu/libxcb-glx.so.0 (0x000000ffe8a9c000)
        libxcb-xkb.so.1 => /lib/loongarch64-linux-gnu/libxcb-xkb.so.1 (0x000000ffe8a7c000)
        libxcb-randr.so.0 => /lib/loongarch64-linux-gnu/libxcb-randr.so.0 (0x000000ffe8a64000)
        libxcb-icccm.so.4 => /lib/loongarch64-linux-gnu/libxcb-icccm.so.4 (0x000000ffe8a58000)
        libxcb-shm.so.0 => /lib/loongarch64-linux-gnu/libxcb-shm.so.0 (0x000000ffe8a4c000)
        libxcb-render.so.0 => /lib/loongarch64-linux-gnu/libxcb-render.so.0 (0x000000ffe8a38000)
        libxcb-image.so.0 => /lib/loongarch64-linux-gnu/libxcb-image.so.0 (0x000000ffe8a2c000)
        libxcb-xfixes.so.0 => /lib/loongarch64-linux-gnu/libxcb-xfixes.so.0 (0x000000ffe8a1c000)
        libxcb-shape.so.0 => /lib/loongarch64-linux-gnu/libxcb-shape.so.0 (0x000000ffe8a10000)
        libxcb-sync.so.1 => /lib/loongarch64-linux-gnu/libxcb-sync.so.1 (0x000000ffe8a00000)
        libxcb-render-util.so.0 => /lib/loongarch64-linux-gnu/libxcb-render-util.so.0 (0x000000ffe89ec000)
        libxcb-keysyms.so.1 => /lib/loongarch64-linux-gnu/libxcb-keysyms.so.1 (0x000000ffe89e0000)
        libxcb.so.1 => /lib/loongarch64-linux-gnu/libxcb.so.1 (0x000000ffe89b4000)
        libX11.so.6 => /lib/loongarch64-linux-gnu/libX11.so.6 (0x000000ffe8870000)
        libX11-xcb.so.1 => /lib/loongarch64-linux-gnu/libX11-xcb.so.1 (0x000000ffe8864000)
        libfontconfig.so.1 => /lib/loongarch64-linux-gnu/libfontconfig.so.1 (0x000000ffe8818000)
        libdbus-1.so.3 => /lib/loongarch64-linux-gnu/libdbus-1.so.3 (0x000000ffe87c4000)
        libtiff.so.5 => /lib/loongarch64-linux-gnu/libtiff.so.5 (0x000000ffe8744000)
        libgcc_s.so.1 => /lib/loongarch64-linux-gnu/libgcc_s.so.1 (0x000000ffe86dc000)
        libpthread.so.0 => /lib/loongarch64-linux-gnu/libpthread.so.0 (0x000000ffe86b4000)
        libc.so.6 => /lib/loongarch64-linux-gnu/libc.so.6 (0x000000ffe8548000)
        /lib64/ld.so.1 (0x000000fff6fddc18)
        libpcre.so.3 => /lib/loongarch64-linux-gnu/libpcre.so.3 (0x000000ffe84f8000)
        libXext.so.6 => /lib/loongarch64-linux-gnu/libXext.so.6 (0x000000ffe84dc000)
        libnss3.so => /lib/loongarch64-linux-gnu/libnss3.so (0x000000ffe83b8000)
        libnssutil3.so => /lib/loongarch64-linux-gnu/libnssutil3.so (0x000000ffe8380000)
        libnspr4.so => /lib/loongarch64-linux-gnu/libnspr4.so (0x000000ffe833c000)
        libsmime3.so => /lib/loongarch64-linux-gnu/libsmime3.so (0x000000ffe8308000)
        libstdc++.so.6 => /lib/loongarch64-linux-gnu/libstdc++.so.6 (0x000000ffe8140000)
        libm.so.6 => /lib/loongarch64-linux-gnu/libm.so.6 (0x000000ffe8084000)
        libxcb-util.so.1 => /lib/loongarch64-linux-gnu/libxcb-util.so.1 (0x000000ffe8074000)
        libXau.so.6 => /lib/loongarch64-linux-gnu/libXau.so.6 (0x000000ffe8068000)
        libXdmcp.so.6 => /lib/loongarch64-linux-gnu/libXdmcp.so.6 (0x000000ffe8058000)
        libfreetype.so.6 => /lib/loongarch64-linux-gnu/libfreetype.so.6 (0x000000ffe7f9c000)
        libexpat.so.1 => /lib/loongarch64-linux-gnu/libexpat.so.1 (0x000000ffe7f48000)
        libuuid.so.1 => /lib/loongarch64-linux-gnu/libuuid.so.1 (0x000000ffe7f38000)
        libsystemd.so.0 => /lib/loongarch64-linux-gnu/libsystemd.so.0 (0x000000ffe7e88000)
        libwebp.so.6 => /lib/loongarch64-linux-gnu/libwebp.so.6 (0x000000ffe7e30000)
        libzstd.so.1 => /lib/loongarch64-linux-gnu/libzstd.so.1 (0x000000ffe7d60000)
        liblzma.so.5 => /lib/loongarch64-linux-gnu/liblzma.so.5 (0x000000ffe7d34000)
        libjbig.so.0 => /lib/loongarch64-linux-gnu/libjbig.so.0 (0x000000ffe7d20000)
        libjpeg.so.62 => /lib/loongarch64-linux-gnu/libjpeg.so.62 (0x000000ffe7ccc000)
        libplc4.so => /lib/loongarch64-linux-gnu/libplc4.so (0x000000ffe7cc0000)
        libplds4.so => /lib/loongarch64-linux-gnu/libplds4.so (0x000000ffe7cb4000)
        libbsd.so.0 => /lib/loongarch64-linux-gnu/libbsd.so.0 (0x000000ffe7c94000)
        libpng16.so.16 => /lib/loongarch64-linux-gnu/libpng16.so.16 (0x000000ffe7c58000)
        librt.so.1 => /lib/loongarch64-linux-gnu/librt.so.1 (0x000000ffe7c48000)
        liblz4.so.1 => /lib/loongarch64-linux-gnu/liblz4.so.1 (0x000000ffe7c10000)
        libgcrypt.so.20 => /lib/loongarch64-linux-gnu/libgcrypt.so.20 (0x000000ffe7b40000)
        libgpg-error.so.0 => /lib/loongarch64-linux-gnu/libgpg-error.so.0 (0x000000ffe7b18000)
```

也并没有看到 Qt 相关 so。继续使用 strings 命令搜索代码中的符号：

```
uos@uos-loongsun-PC:/opt/apps/com.tencent.wechat/files$ strings wechat |grep "QApplication"
QShortcut: Initialize QApplication before calling 'setEnabled'.
QCoreApplication::arguments: Please instantiate the QApplication object first
QWidget: Cannot create a QWidget without QApplication
QEventLoop: Cannot be used without QApplication
QApplication: invalid style override '%s' passed, ignoring it.
QShortcut: Initialize QApplication before calling 'setAutoRepeat'.
Must construct a QApplication first.
QApplication::%s: Please instantiate the QApplication object first
WARNING: QApplication was not created in the main() thread.
QShortcut: Initialize QApplication before calling 'QShortcut'.
QShortcut: Initialize QApplication before calling 'setContext'.
QWidget: Must construct a QApplication before a QWidget
QApplicationStateChangeEvent(
QApplication::notify: Unexpected null receiver
QShortcut: Initialize QApplication before calling 'setKey'.
QCoreApplication::applicationDirPath: Please instantiate the QApplication object first
QCoreApplication::applicationFilePath: Please instantiate the QApplication object first
12QApplication
19QApplicationPrivate
QApplication
28QApplicationStateChangeEvent
```

换一个 Qt 的关键字进行搜索：

```
uos@uos-loongsun-PC:/opt/apps/com.tencent.wechat/files$ strings wechat |grep "QWindow"
QWindows cannot be reparented into desktop windows
QWindow::fromWinId(): platform plugin does not support foreign windows.
1focusWindowChanged(QWindow*)
QWindow::setWindowStates does not accept Qt::WindowActive
%s QWindow has no platform window, see QTBUG-32681
QWindowSystemInterface::flushWindowSystemEvents() invoked after QGuiApplication destruction, discarding 
QWindow(0x0)
QWindowStateChangeEvent
passed to QWindow::startSystemResize, ignoring.
2focusWindowChanged(QWindow*)
QWindowsStyle
Manually deleting a QPlatformScreen. Call QWindowSystemInterface::handleScreenRemoved instead.
Attempting to create QWindow-based QOffscreenSurface outside the gui thread. Expect failures.
QWindowsVistaStyle
QWindowContainer: embedded window cannot be null
QWindowContainer
N29QWindowSystemInterfacePrivate17WindowSystemEventE
N29QWindowSystemInterfacePrivate9UserEventE
N29QWindowSystemInterfacePrivate10InputEventE
N29QWindowSystemInterfacePrivate8KeyEventE
N29QWindowSystemInterfacePrivate16ContextMenuEventE
virtual void QXcbBackingStore::flush(QWindow*, const QRegion&, const QPoint&)
16QWindowContainer
23QWindowContainerPrivate
QWindowContainer
QWindow*
20QWindowsStylePrivate
13QWindowsStyle
QWindowsStyle
N29QWindowSystemInterfacePrivate10CloseEventE
N29QWindowSystemInterfacePrivate19GeometryChangeEventE
N29QWindowSystemInterfacePrivate10EnterEventE
N29QWindowSystemInterfacePrivate10LeaveEventE
N29QWindowSystemInterfacePrivate20ActivatedWindowEventE
N29QWindowSystemInterfacePrivate23WindowStateChangedEventE
N29QWindowSystemInterfacePrivate24WindowScreenChangedEventE
N29QWindowSystemInterfacePrivate27SafeAreaMarginsChangedEventE
N29QWindowSystemInterfacePrivate28ApplicationStateChangedEventE
N29QWindowSystemInterfacePrivate16FlushEventsEventE
N29QWindowSystemInterfacePrivate10MouseEventE
N29QWindowSystemInterfacePrivate10WheelEventE
N29QWindowSystemInterfacePrivate10TouchEventE
N29QWindowSystemInterfacePrivate22ScreenOrientationEventE
N29QWindowSystemInterfacePrivate19ScreenGeometryEventE
N29QWindowSystemInterfacePrivate29ScreenLogicalDotsPerInchEventE
N29QWindowSystemInterfacePrivate22ScreenRefreshRateEventE
N29QWindowSystemInterfacePrivate16ThemeChangeEventE
N29QWindowSystemInterfacePrivate11ExposeEventE
N29QWindowSystemInterfacePrivate13FileOpenEventE
N29QWindowSystemInterfacePrivate11TabletEventE
N29QWindowSystemInterfacePrivate25TabletEnterProximityEventE
N29QWindowSystemInterfacePrivate25TabletLeaveProximityEventE
N29QWindowSystemInterfacePrivate18PlatformPanelEventE
N29QWindowSystemInterfacePrivate12GestureEventE
25QWindowSystemEventHandler
QWindow*
7QWindow
14QWindowPrivate
QWindow
QWindow::Visibility
QWindow*
23QWindowStateChangeEvent
```

可以看出 wechat 这个应用程序还是有许多 QT 相关的符号，基本上可以确定是使用了 QT，至于为什么没有加载 Qt 动态库，只有一个解释，那就是静态链接了 Qt 库。这一点倒是挺特别的，因为使用 Qt 开发，一般只会使用 Qt 动态库。

如果使用 ps 命令查看：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202411/images/wechat_uos_05.png)

看到这张图，可能有人会认为微信采用了 Electron 框架，其实这个真不一定。熟悉浏览器的朋友一眼就可以看出，这是在微信中启动了浏览器进程，其实这也很好理解，微信公众号、微信小程序都依赖浏览器内核，所以集成 chromium 不足为奇。

在龙芯的机器上体验了一下微信，确实很流畅，没有卡顿，比原来的 Wine 版本要好多了。因为以前的 Wine 版本其实是 X86 架构的 Windows 应用，存在指令集的转换，Wine 在某些方面的效能也不及 Linux 原生应用。

在办公室，企业微信比微信使用得更多，可惜的是，企业微信还没有退出原生版，依然只能使用 Wine 来运行 Windows 版的企业微信。难道企业微信和微信还不是一个团队开发的？什么时候企业微信也和微信一样，提供全平台全架构版本，那国产系统就更完美了。

你还期待哪些应用推出原生版，欢迎评论区留言！
