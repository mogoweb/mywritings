# 一个 Windows 窗口的 Linux 系统之旅（Wayland 篇）

在上篇文章《[一个 Windows 窗口的 Linux 系统之旅](https://mp.weixin.qq.com/s/blIp_-uzE2O2NhnO2nMdQw)》中介绍了 Windows 应用程序的窗口是如何调用到 X11 的窗口创建的整个流程。Wayland 被认为是即将取代 X11 的下一代窗口协议，在很多 Linux 发行版上已经默认启用。Wine 项目自然也加快了 Wayland 协议的适配，这篇文章接着上一篇，继续探讨 Windows 应用的窗口是如何通过 Wayland 窗口协议创建起来的。

我们先来回顾一下 Win32 API 层的调用流程：

```
应用程序调用
   ↓
CreateWindow(...)    // 宏 (include/winuser.h)
   ↓
CreateWindowEx(0, ...)   // 宏展开
   ↓
CreateWindowExA / CreateWindowExW   // 函数声明 (dlls/user32/user32.spec)
   ↓
CreateWindowExA/W 实现 (dlls/user32/win.c)
   ↓
WIN_CreateWindowEx(...)   // 核心逻辑 (dlls/user32/win.c)
   ↓
NtUserCreateWindowEx(...) // 内核调用封装 (dlls/user32/syscall.c → win32u)
```

截止到目前这个步骤，X11 / Wayland 协议共用相同的代码逻辑，再往下走，两者所走的路径就开始分叉了。

上篇文章讲到，在 X11 下是通过 user_driver->pCreateWindow 转到 XCreateWindow 创建窗口的。然而查看 `dlls/winewayland.drv/waylanddrv_main.c` 源码，却没有 pCreateWindow 的赋值：

```
static const struct user_driver_funcs waylanddrv_funcs =
{
    .pClipboardWindowProc = WAYLAND_ClipboardWindowProc,
    .pClipCursor = WAYLAND_ClipCursor,
    .pDesktopWindowProc = WAYLAND_DesktopWindowProc,
    .pDestroyWindow = WAYLAND_DestroyWindow,
    .pSetIMECompositionRect = WAYLAND_SetIMECompositionRect,
    .pKbdLayerDescriptor = WAYLAND_KbdLayerDescriptor,
    .pReleaseKbdTables = WAYLAND_ReleaseKbdTables,
    .pSetCursor = WAYLAND_SetCursor,
    .pSetCursorPos = WAYLAND_SetCursorPos,
    .pSetLayeredWindowAttributes = WAYLAND_SetLayeredWindowAttributes,
    .pSetWindowIcon = WAYLAND_SetWindowIcon,
    .pSetWindowStyle = WAYLAND_SetWindowStyle,
    .pSetWindowText = WAYLAND_SetWindowText,
    .pSysCommand = WAYLAND_SysCommand,
    .pUpdateDisplayDevices = WAYLAND_UpdateDisplayDevices,
    .pWindowMessage = WAYLAND_WindowMessage,
    .pWindowPosChanged = WAYLAND_WindowPosChanged,
    .pWindowPosChanging = WAYLAND_WindowPosChanging,
    .pCreateWindowSurface = WAYLAND_CreateWindowSurface,
    .pVulkanInit = WAYLAND_VulkanInit,
    .pOpenGLInit = WAYLAND_OpenGLInit,
};
```

所以，与 X11 驱动不同，Wayland 驱动没有实现 pCreateWindow函数指针。相反，它使用延迟创建的方式，通过 pWindowPosChanged 和 pWindowPosChanging 回调处理窗口创建：

```
BOOL WAYLAND_WindowPosChanging(HWND hwnd, UINT swp_flags, BOOL shaped, const struct window_rects *rects)
{
    struct wayland_win_data *data = wayland_win_data_get(hwnd);

    TRACE("hwnd %p, swp_flags %04x, shaped %u, rects %s\n", hwnd, swp_flags, shaped, debugstr_window_rects(rects));

    if (!data && !(data = wayland_win_data_create(hwnd, rects))) return FALSE;

    wayland_win_data_release(data);

    return TRUE;
}
```
可以看到，在 WAYLAND_WindowPosChanging 函数中，Wayland 驱动创建 wayland_win_data 结构。

```
void WAYLAND_WindowPosChanged(HWND hwnd, HWND insert_after, HWND owner_hint, UINT swp_flags, BOOL fullscreen,
                              const struct window_rects *new_rects, struct window_surface *surface)
{
    ...

    if (!surface)
    {
        if ((client = data->client_surface))
        {
            if (toplevel && NtUserIsWindowVisible(hwnd))
                wayland_client_surface_attach(client, toplevel);
            else
                wayland_client_surface_attach(client, NULL);
        }

        if (data->wayland_surface)
        {
            wayland_surface_destroy(data->wayland_surface);
            data->wayland_surface = NULL;
        }
    }
    else if (wayland_win_data_create_wayland_surface(data, toplevel_surface))
    {
        wayland_win_data_update_wayland_state(data);
    }

    ...
}
```

而在 WAYLAND_WindowPosChanged 函数中，Wayland 驱动创建相应的 Wayland 表面（surface）。

相比 X11，虽然 Wayland 也是客户端-服务器模型，但更轻量，强调客户端自己渲染内容。Wine 的窗口在 Wayland 上通常不映射为真实“窗口”，而是 Wayland 的 surface。

和 X11 的不同之处：

* 客户端完全负责渲染像素内容，服务器（Compositor）只负责合成。

* 没有全局查询机制：客户端不知道其他窗口状态，也无法直接获取屏幕尺寸或其他窗口信息。

* 所有请求（例如 attach buffer、resize）都是异步的，没有直接返回值。

* 窗口管理（移动、最大化、关闭）完全由 compositor 控制。

* 没有“window ID”概念，全靠 Wayland 对象和 role。

## 小结
Wayland 驱动采用了与 X11 驱动不同的实现方式。它不在窗口创建时立即创建 Wayland 资源，而是在窗口真正需要显示时才创建相应的 Wayland 表面，这是一种更符合 Wayland 协议设计理念的延迟创建模式。