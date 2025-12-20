# Wine 是如何加载图形驱动的？

在 Wine 语境下，graphics driver（图形驱动）与 Windows 内核态的显卡驱动，或 Linux DRM / KMS / Mesa 这一整套图形栈中的硬件驱动，不是同一个层级的概念。它是一个用户态的窗口系统与图形后端适配层，而不是硬件驱动。其核心职责是把 Windows 程序眼中的 GDI / 窗口 / OpenGL / Vulkan 等图形与窗口操作，映射到宿主系统的图形与窗口系统实现上。

简化后的 Wine 图形相关栈结构如下：

```
Windows 应用
   |
   |  GDI / USER32 / Win32 窗口 API
   v
wineuser / gdi32 / win32u
   |
   |  graphics_driver 接口
   v
-----------------------------------
| Wine 图形驱动（用户态）            |
|                                 |
|  winex11.drv       -> X11       |
|  winewayland.drv   -> Wayland   |
|  winemac.drv       -> macOS     |
|  wineandroid.drv   -> Android   |
-----------------------------------
   |
   |  宿主系统图形栈
   v
X11 / Wayland / Quartz / Cocoa / Android
   |
   v
Mesa / DRM / 显卡内核驱动

```

## wine 图形驱动选择流程

graphics_driver 位于 Wine 内部，以动态库的形式实现，是“Win32 图形语义”与“宿主窗口系统”的桥梁。目前实现的图形驱动有：

* wineandroid.drv
* winemac.drv
* winewayland.drv
* winex11.drv

其中，Android 图形驱动目前仍处于非常早期的阶段，功能并不完整，尚无法正常使用。

既然 Wine 提供了多种图形驱动，那么它是如何决定实际加载哪一个的呢？答案是：基于优先级的回退（fallback）机制。

1. Explorer 进程中的驱动加载

图形驱动的选择最早发生在 explorer 进程中。

explore 模块的 `load_graphics_driver` 函数首先检查注册表项 `HKCU\Software\Wine\Drivers\Graphics`，以确认用户是否显式指定了图形驱动；如果没有设置，则使用默认的驱动列表。

默认的驱动列表定义为：

```
static const WCHAR default_driver[] = L"mac,x11,wayland";
```

注册表中的值也可以像默认值一样，使用逗号分隔，同时指定多个驱动名称。

2. 按顺序尝试加载驱动

函数会遍历以逗号分隔的驱动名称列表，依次尝试加载：

* 将驱动名拼接为 wine{name}.drv（例如 winex11.drv、winewayland.drv）

* 调用 LoadLibraryW 加载对应的驱动 DLL

* 第一个成功加载的驱动即被选中并使用

3. 写入注册表信息

一旦驱动成功加载（或使用了 null 驱动），Wine 会将 GraphicsDriver 注册表值写入以下位置：

> System\CurrentControlSet\Control\Video\{GUID}\0000

如果加载失败，则会记录对应的错误信息。

## 图形驱动初始化流程

1. 桌面窗口创建与驱动加载

当桌面窗口被创建时，win32u 中的 load_desktop_driver 会读取 explorer 进程预先写入的 GraphicsDriver 注册表项。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202512/images/wine_graphics_driver_01.png)

随后，Wine 通过用户态回调机制来触发驱动加载。

2. 回调加载驱动 DLL

* 发起回调

上面代码中 `KeUserModeCallback` 传入的第一个参数 `NtUserLoadDriver` 是回调 ID，info->Data 包含驱动 DLL 的路径，info->DataLength 是路径长度。

* 回调表初始化

在 user32.dll 加载时，process_attach 函数会设置 KernelCallbackTable。

```c
static BOOL process_attach(void)
{
    NtCurrentTeb()->Peb->KernelCallbackTable = kernel_callback_table;
    RegisterWaitForInputIdle( WaitForInputIdle );

    winproc_init();
    ...
}
```

这个回调表 kernel_callback_table 是一个函数指针数组，包含所有的回调函数实现：

```c
static KERNEL_CALLBACK_PROC kernel_callback_table[NtUserCallCount] =
{
#define USER32_CALLBACK_ENTRY(name) User32##name,
    ALL_USER32_CALLBACKS
#undef USER32_CALLBACK_ENTRY
};
```

* 回调分发机制

KeUserModeCallback 通过 KiUserCallbackDispatcher 分发到用户模式。在 x86_64 架构下，分发器会查找 peb->KernelCallbackTable[id] 来调用对应的回调函数。这里面使用了一段汇编指令：

```c
#ifdef __WINE_PE_BUILD
                   "movq %gs:0x60,%rax\n\t"     /* peb */
                   "movq 0x58(%rax),%rax\n\t"   /* peb->KernelCallbackTable */
                   "call *(%rax,%r8,8)\n\t"     /* KernelCallbackTable[id] */
```

* User32LoadDriver 实现

最终，NtUserLoadDriver 回调 ID 对应到 User32LoadDriver 函数，该函数使用 LdrLoadDll 加载驱动 DLL：

```c
static NTSTATUS WINAPI User32LoadDriver( void *args, ULONG size )
{
    const WCHAR *path = args;
    UNICODE_STRING str;
    HMODULE module;

    RtlInitUnicodeString( &str, path );
    return LdrLoadDll( L"c:\\windows\\system32", 0, &str, &module );
}
```

3. winewayland.drv 始化流程

DllMain 初始化 Unix 调用接口，并调用 waylanddrv_unix_init。从 Windows PE 到 Unix 侧的实现，Wine 使用了一套非常精妙的实现方法。简单说就是通过 __wine_unix_call 机制，将 PE 侧调用分发到 Unix 侧函数表，借助汇编调度器完成上下文切换并执行对应的 Unix 实现。有兴趣的读者不妨进一步深入分析这一机制，在理解其实现细节的过程中，往往会由衷感叹这些开发者在设计上的巧思与工程智慧。

我们主要关注 waylanddrv 的初始化，可以先从 waylanddrv_unix_init 入口看起。

```
static NTSTATUS waylanddrv_unix_init(void *arg)
{
    /* Set the user driver functions now so that they are available during
     * our initialization. We clear them on error. */
    __wine_set_user_driver(&waylanddrv_funcs, WINE_GDI_DRIVER_VERSION);

    wayland_init_process_name();

    if (!wayland_process_init()) goto err;

    return 0;

err:
    __wine_set_user_driver(NULL, WINE_GDI_DRIVER_VERSION);
    return STATUS_UNSUCCESSFUL;
}
```

* 直接调用 __wine_set_user_driver 注册 Wayland 驱动函数表

* 随后初始化 Wayland 进程级连接

4. 驱动注册机制

__wine_set_user_driver 是核心的驱动注册函数，其职责包括：

* 校验驱动接口版本是否匹配

* 分配并拷贝驱动函数表

* 对未实现的函数指针填充 null driver 的默认实现

* 使用 InterlockedCompareExchangePointer 原子性地更新全局 user_driver 指针

__wine_set_user_driver 定义在 `dlls/win32u/driver.c`，由图形驱动模块调用。

需要指出的是，Wine 启动时会先使用一个 lazy_load_driver，其中仅包含占位用的 stub 函数。只有当第一次调用窗口系统相关函数时会触发实际的驱动加载流程。

## 为什么 unset DISPLAY 环境变量可以使用 waylanddrv

如果你向 AI 提问，如何使用纯正的 Wine Wayland，而不是通过 XWayland，AI 大概率会给出答案：

```
$ env -u DISPLAY wine your.exe
```

网上的很多资料也没有提到修改注册表，而是通过 unset DISPLAY 来做到的。

为什么这种方法可行，这要提到图形驱动加载的 fallback 机制。前面有讲到，Wine 会遍历以逗号分隔的驱动名称列表，依次尝试加载。默认的加载次序为 mac、x11、wayland，在 Linux 下，不存在 macdrv，肯定加载失败。然后就会尝试加载 x11drv。

在 Wine 的 X11 驱动（winex11.drv）初始化过程中，DISPLAY 环境变量是必须的。一旦 DISPLAY 未设置，XOpenDisplay(NULL) 就会失败，导致 X11 驱动初始化失败。

于是 Wine 自然回退到下一个驱动：winewayland.drv。

如果 Wayland 驱动也无法初始化，Wine 最终会退回到 null_driver，窗口系统将无法正常工作。

## 小结

从图形驱动的选择，到回调加载，再到 Wayland 驱动的初始化过程，可以看到 Wine 在不同系统语义之间搭建了一套高度模块化、可替换的适配层。

正是这种设计，使 Wine 能够在 X11、Wayland、macOS 甚至 Android 等完全不同的窗口系统之上，持续演进并保持兼容性。

随着 Wayland 支持逐步成熟，Wine 的图形驱动体系，也正在经历一次真正意义上的架构升级。
