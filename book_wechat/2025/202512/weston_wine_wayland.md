# 使用 Weston 调试 Wine 的 Wayland 实现

随着主流 Linux 发行版纷纷加速从 X11 向 Wayland 迁移，GNOME 与 KDE Plasma 这两大桌面环境也相继宣布计划彻底移除 X11 相关代码。从生态趋势看，无论是桌面环境还是应用程序，转向 Wayland 已势不可挡。

Wine 作为 Linux 平台运行 Windows 应用的重要兼容层，在整个 Linux 生态中占据着关键位置。Wine 项目组同样意识到 Wayland 的发展方向，已着手推进 Wayland 驱动的开发。然而，由于 Windows 窗口模型与 Wayland 的设计理念存在显著差异，移植工作面临不少挑战。经过持续投入，目前 Wine 的 Wayland 驱动已有初步实现，但距离完全成熟仍有一定差距。在多数发行版的 Wayland 会话中，Wine 仍主要依赖 XWayland，通过旧的 X11 驱动路径来运行。

在 Wayland 驱动仍不完善的阶段，调试工作尤为关键。

我目前使用 deepin v25 作为开发环境，其已支持 Wayland 会话（treeland）。但 treeland 作为新实现，在日常开发中仍会遇到各种不稳定因素，问题来源难以判断：究竟是 treeland 合成器本身的实现缺陷，还是应用对 Wayland 的支持存在不足？这些问题对开发效率影响较大。多次尝试后，我暂时将显示管理器切回 LightDM。

## Weston：轻量级 Wayland 合成器的最佳选择

然而，要调试 Wine 的 Wayland 驱动，又必须在 Wayland 环境中运行，这便形成了矛盾。我想起自己曾写过一篇介绍 Wayland 客户端开发的文章《[编写 Wayland 客户端（二）](https://mp.weixin.qq.com/s/Mt4Y9RmrwbyMPoMhZqmPDQ)》，其中提到 Weston——用于展示 Wayland 协议与功能的参考实现合成器。Weston 最大的优势在于：

* 可作为独立 Wayland 合成器运行；

* 也可以作为 X11 应用程序在现有桌面环境中运行；

* 稳定、轻量、配置简单，适合作为调试环境。

这正是我需要的：在 deepin 的 X11 桌面中启动 Weston，从而获得一个干净的 Wayland 合成环境，专门用于调试 Wine 的 Wayland 驱动行为，而无需依赖 treeland。

## 编译 Wine 的 Wayland 支持

要编译 wayland 支持，需要额外安装如下开发包：

* libwayland-dev
* wayland-protocols
* libxkbcommon-dev
* linux-libc-dev
* libxkbregistry-dev

然后使用如下命令编译 wine 源码：

```
mkdir build64
cd build64
../configure --prefix=/work/usr/local/wine --enable-win64 --enable-archs=i386,x86_64 --enable-werror --with-mingw
make -j20
make install
```

这里通过 --prefix 参数将目标安装在 /work/usr/local/wine 下，是为了不覆盖系统的 wine，为了使用自己编译的 wine，我定义了如下环境文件 wine.env:

```
# wine.env - 使用自编译 Wine 和库

# 自编译 Wine 可执行文件目录
export WINE_BUILD_DIR="/work/usr/local/wine"

# 优先使用自编译 Wine
export PATH="$WINE_BUILD_DIR/bin:$PATH"

# 自编译 Wine 库目录
export LD_LIBRARY_PATH="$WINE_BUILD_DIR/lib/wine/x86_64-unix:$WINE_BUILD_DIR/lib64:$LD_LIBRARY_PATH"

# Windows DLL 路径 (.dll)
export WINEPATH="$WINE_BUILD_DIR/lib/wine/x86_64-windows:$WINEPATH"

# 默认 Wine 前缀和架构
export WINEARCH=win64
export WINEPREFIX="$HOME/.wine64-build"

# 开启 coredump（调试崩溃用）
ulimit -c unlimited

# 可选调试日志
# export WINEDEBUG=+all

echo "Using custom Wine from $WINE_BUILD_DIR"
echo "Library path: $LD_LIBRARY_PATH"
echo "WINEPREFIX=$WINEPREFIX, WINEARCH=$WINEARCH"
```

在执行了 `source wine.env` 命令后，wine 就是使用的自己编译的版本了：

```
$ which wine
/work/usr/local/wine/bin/wine
$ wine --version
wine-11.0-rc1-57-g9daccb73269
```

为了使用 weston 合成器，先启动 weston 应用程序，Weston 会在当前桌面中新开一个窗口，内部就是一个完整的 Wayland 合成器环境。

然后设置环境变量，启动 wine：

```
$ weston &
$ export WAYLAND_DISPLAY=wayland-1
$ env -u DISPLAY wine notepad
0074:err:waylanddrv:wayland_process_init Wayland compositor doesn't support optional zwp_text_input_manager_v3 (host input methods won't work)
0074:err:waylanddrv:wayland_process_init Wayland compositor doesn't support optional zwlr_data_control_manager_v1 (clipboard functionality will be limited)
0074:err:waylanddrv:wayland_process_init Wayland compositor doesn't support xdg_toplevel_icon_manager_v1 (window icons will not be supported)
00ac:err:waylanddrv:wayland_process_init Wayland compositor doesn't support optional zwp_text_input_manager_v3 (host input methods won't work)
```

可以看到，记事本程序运行在 weston 窗口中：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202512/images/weston_wine_wayland_01.png)

由于中文字体配置问题，记事本应用程序的中文都显示为方框，这个问题，我将在下篇文章中继续阐述，敬请关注。