# 一个 Windows 窗口的 Linux 系统之旅

之前写过一篇文章《[从Windows示例代码入手，逐步剖析Wine实现原理](https://mp.weixin.qq.com/s/Phkb20xwl9bHDzQueEvmIg)》，从一个简单的 Windows 应用程序是如何通过 Wine 在 Linux 系统上运行起来的。这次，我们将更深入一步，通过分析 Wine 的源码，进一步剖析一个 Windows 应用窗口是如何在 Linux 系统上创建起来的。

在开始之前，我们还是准备一个 Windows 窗口程序，这个程序足够简单，不借助 MFC 或者 QT 这样的框架，仅通过 Windows API 创建一个应用程序窗口。

如下是程序代码：

```
#include <windows.h>

LRESULT CALLBACK WndProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    switch(msg) {
        case WM_CLOSE: PostQuitMessage(0); break;
        default: return DefWindowProc(hwnd, msg, wParam, lParam);
    }
    return 0;
}

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,
                   LPSTR lpCmdLine, int nCmdShow) {
    WNDCLASS wc = {0};
    wc.lpfnWndProc = WndProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = "MyWindowClass";
    RegisterClass(&wc);

    HWND hwnd = CreateWindow("MyWindowClass", "Hello Wine", WS_OVERLAPPEDWINDOW,
                             CW_USEDEFAULT, CW_USEDEFAULT, 400, 300,
                             NULL, NULL, hInstance, NULL);

    ShowWindow(hwnd, nCmdShow);

    MSG msg;
    while(GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    return 0;
}
```

这段代码中使用了 CreateWindow / ShowWindow / GetMessage 等 Windows API，这里仅分析 CreateWindow 的处理流程。

## Win32 API 层

实际上，CreateWindow 不是一个真正的函数，而是一个宏，它最终会展开成 CreateWindowEx。

在 Windows 官方 SDK（winuser.h）中，有这样的定义（Wine 也保持一致，头文件位于 `include/winuser.h` ）：

```
#define CreateWindowA(lpClassName, lpWindowName, dwStyle, x, y, nWidth, nHeight, \
                      hWndParent, hMenu, hInstance, lpParam) \
    CreateWindowExA(0L, lpClassName, lpWindowName, dwStyle, \
                    x, y, nWidth, nHeight, hWndParent, hMenu, hInstance, lpParam)

#define CreateWindowW(lpClassName, lpWindowName, dwStyle, x, y, nWidth, nHeight, \
                      hWndParent, hMenu, hInstance, lpParam) \
    CreateWindowExW(0L, lpClassName, lpWindowName, dwStyle, \
                    x, y, nWidth, nHeight, hWndParent, hMenu, hInstance, lpParam)

#ifdef UNICODE
#define CreateWindow  CreateWindowW
#else
#define CreateWindow  CreateWindowA
#endif
```

由于历史原因，一些 Windows API 有两个版本：一个是 ANSI 版本，以 A 结尾，另一个是 Unicode 版本，以 W 结尾。根据宏定义的不同（是否 UNICODE），展开成对应的版本。

CreateWindowExA 和 CreateWindowExW 的实现 Wine 的 user32.dll 模块中。查看 dlls/user32/win.c 的源码，可以看到 CreateWindowExW 的实现：

```
HWND WINAPI DECLSPEC_HOTPATCH CreateWindowExW( DWORD exStyle, LPCWSTR className,
                                 LPCWSTR windowName, DWORD style, INT x,
                                 INT y, INT width, INT height,
                                 HWND parent, HMENU menu,
                                 HINSTANCE instance, LPVOID data )
{
    CREATESTRUCTW cs;

    cs.lpCreateParams = data;
    cs.hInstance      = instance;
    cs.hMenu          = menu;
    cs.hwndParent     = parent;
    cs.x              = x;
    cs.y              = y;
    cs.cx             = width;
    cs.cy             = height;
    cs.style          = style;
    cs.lpszName       = windowName;
    cs.lpszClass      = className;
    cs.dwExStyle      = exStyle;

    return wow_handlers.create_window( &cs, className, instance, TRUE );
}
```
这里插一句，有一个 user32.spec 文件，在 Wine 项目里很关键。它不是普通的源码文件，而是导出符号表（DLL 导出规范）。

在 Windows 世界里，每个 DLL（例如 user32.dll）都会导出一组 API 函数（例如 CreateWindowExW、MessageBoxW）。

Wine 需要 伪造出一个跟 Windows 一样的 DLL，让 Windows 程序能链接和调用。user32.spec 文件就是告诉 Wine 构建系统：

* 这个 DLL 需要导出哪些函数（名字、序号、参数）

* 这些函数在 Wine 里由哪个实现来处理

* 兼容 Windows 的调用约定和符号导出规则

比如 dlls/user32/user32.spec 里面有这样一行：

```
@ stdcall CreateWindowExW(long wstr wstr long long long long long long long long ptr)
```

Wine 构建系统读取 .spec 文件生成对应的**导出表/跳转桩**，细节这里不用去深究，反正是通过这一步，Windows 应用程序就将 Windows API 调用转向了 Wine 的实现。

继续分析上面的源码实现，wow_handlers 实际上是一个结构，也是一个全局函数表：

```
struct wow_handlers16 wow_handlers =
{
    ...
    WIN_CreateWindowEx,
    ...
};
```
即 create_window 指向 WIN_CreateWindowEx 函数，这个 WIN_CreateWindowEx 函数也定义在 win.c 中。

```
HWND WIN_CreateWindowEx( CREATESTRUCTW *cs, LPCWSTR className, HINSTANCE module, BOOL unicode )
{
    WCHAR nameW[MAX_ATOM_LEN + 1];
    UNICODE_STRING class = RTL_CONSTANT_STRING(nameW), version, window_name = {0};
    HWND hwnd, top_child = 0;
    MDICREATESTRUCTW mdi_cs;
    WNDCLASSEXW info;
    WCHAR name_buf[8];
    HMENU menu;

    init_class_name( &class, className );
    get_class_version( &class, &version, TRUE );
    
    ...

    hwnd = NtUserCreateWindowEx( cs->dwExStyle, &class, NULL, &window_name, cs->style,
                                 cs->x, cs->y, cs->cx, cs->cy, cs->hwndParent, menu, module,
                                 cs->lpCreateParams, 0, cs->hInstance, 0, !unicode );
    if (!hwnd && menu && menu != cs->hMenu) NtUserDestroyMenu( menu );
    if (!unicode && window_name.Buffer != name_buf) RtlFreeUnicodeString( &window_name );
    return hwnd;
}
```

上述代码的关键地方是调用到 NtUserCreateWindowEx 函数。这个 NtUserCreateWindowEx 不是一个普通 API，在 Windows 下是**内核级接口**（Win32k.sys 提供），属于 NT 内核的系统调用（syscall）。内核态的系统调用不能被应用程序直接调用，只能通过 user32.dll / gdi32.dll 等库调用。

在 Linux 下，自然不存在 NT 内核态，但 Wine 也模拟了这一行为，这就是 user32u.spec，与 user32.spec 类似，也有如下定义：

```
@ stdcall -syscall NtUserCreateWindowEx(long ptr ptr ptr long long long long long long long long ptr long long long long)
```

而 `win32syscalls.h` 头文件定义了如下入口：

```
#define ALL_SYSCALLS32 \
    ...
    SYSCALL_ENTRY( 0x136b, NtUserCreateWindowEx, 68 ) \
    ...
```

NtUserCreateWindowEx 的实现代码 位于 `dlls/win32u/window.c`:

```
HWND WINAPI NtUserCreateWindowEx( DWORD ex_style, UNICODE_STRING *class_name,
                                  UNICODE_STRING *version, UNICODE_STRING *window_name,
                                  DWORD style, INT x, INT y, INT cx, INT cy,
                                  HWND parent, HMENU menu, HINSTANCE class_instance, void *params,
                                  DWORD flags, HINSTANCE instance, DWORD unk, BOOL ansi )
{
    ...
    if (!(win = create_window_handle( parent, owner, class_name, class_instance,
                                      cs.hInstance, ansi, style, ex_style )))
        return 0;
    hwnd = win->handle;

    ...

    /* call the driver */

    if (!user_driver->pCreateWindow( hwnd )) goto failed;

    NtUserNotifyWinEvent( EVENT_OBJECT_CREATE, hwnd, OBJID_WINDOW, 0 );

    ...
}
```

从这里开始，就进入了 Linux 系统的窗口创建逻辑，在下一节进行展开。

这里先小结一下到目前为止的调用流程：

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

## X11/Wayland 后端

在 NtUserCreateWindowEx 函数中，有两个关键的调用，其中一个是 create_window_handle。

create_window_handle 函数通过 Wine 的客户端-服务器协议与 wine server 通信，在 server 端创建窗口对象并返回窗口句柄。

这段代码主要通过 SERVER_START_REQ / SERVER_END_REQ 机制向 Wine server 发送 create_window 请求：
```
    SERVER_START_REQ( create_window )
    {
        req->parent    = wine_server_user_handle( parent );
        req->owner     = wine_server_user_handle( owner );
        req->instance  = wine_server_client_ptr( instance );
        req->dpi_context = dpi_context;
        req->style     = style;
        req->ex_style  = ex_style;
        if (!(req->atom = get_int_atom_value( name )) && name->Length)
            wine_server_add_data( req, name->Buffer, name->Length );
        if (!wine_server_call_err( req ))
        {
            handle      = wine_server_ptr_handle( reply->handle );
            full_parent = wine_server_ptr_handle( reply->parent );
            full_owner  = wine_server_ptr_handle( reply->owner );
            extra_bytes = reply->extra;
            dpi_context = reply->dpi_context;
            class       = wine_server_get_ptr( reply->class_ptr );
        }
    }
    SERVER_END_REQ;
```

窗口创建的服务器协议使用特定的请求和响应结构定义，请求中包含：

* 父窗口和所有者窗口句柄

* 窗口类信息

* 样式和扩展样式标志

* DPI 上下文信息

在成功与服务器通信后，函数会执行以下操作：

1. 分配 WND 结构体内存
2. 处理桌面和消息窗口的特殊情况
3. 初始化 WND 结构体

需要注意到，Wine 会启动一个 Wine server 服务来维护全局窗口层次结构，并协调不同客户端进程间的窗口管理。由于 Wine Server 服务是一个相当复杂的组件，这里先不展开，等后面有机会再来单独分析 Wine Server。

Linux 下有两个主流的窗口协议，一个是 X11，一个是 Wayland。虽然 Wayland 是未来的发展方向，但 Wine 对 Wayland 的支持还不够完善，所以通常走的是 X11 的窗口管理。

在 `include/wine/gdi_driver.h` 头文件中，我们可以看到这样的定义：

```
struct user_driver_funcs
{
    ...
    BOOL    (*pCreateWindow)(HWND);
    ...
};
```

而在 `dlls/winex11.drv/init.c` 文件中，则有这样的赋值语句：

```
    .pChangeDisplaySettings = X11DRV_ChangeDisplaySettings,
    .pUpdateDisplayDevices = X11DRV_UpdateDisplayDevices,
    .pCreateDesktop = X11DRV_CreateDesktop,
    .pCreateWindow = X11DRV_CreateWindow,
    .pDesktopWindowProc = X11DRV_DesktopWindowProc,
    .pDestroyWindow = X11DRV_DestroyWindow,
    .pFlashWindowEx = X11DRV_FlashWindowEx,
```

这样，NtUserCreateWindowEx 函数中的 user_driver->pCreateWindow 最终就是走向了 X11DRV_CreateWindow。该函数定义在 `dlls/winex11.drv/window.c` 中：

```
BOOL X11DRV_CreateWindow( HWND hwnd )
{
    if (hwnd == NtUserGetDesktopWindow())
    {
        struct x11drv_thread_data *data = x11drv_init_thread_data();
        XSetWindowAttributes attr;

        /* create the cursor clipping window */
        attr.override_redirect = TRUE;
        attr.event_mask = StructureNotifyMask | FocusChangeMask;
        data->clip_window = XCreateWindow( data->display, root_window, 0, 0, 1, 1, 0, 0,
                                           InputOnly, default_visual.visual,
                                           CWOverrideRedirect | CWEventMask, &attr );
        XFlush( data->display );
        NtUserSetProp( hwnd, clip_window_prop, (HANDLE)data->clip_window );
        X11DRV_DisplayDevices_RegisterEventHandlers();
    }
    return TRUE;
}
```

到这一步，就进入 X11 的创建窗口的函数 XCreateWindow，完成了最终的窗口创建。

至此，整个流程闭环。

## 小结

纵观整个流程，它体现了Wine的分层架构设计：

* 用户API层：提供Windows API兼容性
* 系统调用层：实现窗口管理核心逻辑
* 驱动程序层：处理平台特定的显示实现
* 本地API层：调用Linux/X11原生函数

通过这种设计，Windows应用程序无需修改即可在Linux环境下正常创建和显示窗口，Wine 负责完成所有的API转换和平台适配工作。

整体架构流程图总结如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202509/images/wine_window_01.png)

如果觉得文章不错，点个在看吧！

