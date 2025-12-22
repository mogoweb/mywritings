# winewayland.drv 的初始化过程

在前一篇文章《[Wine 是如何加载图形驱动的？](https://mp.weixin.qq.com/s/GdydTFRzmYw4xXUeSzSPww)》中已经分析过：当 Wine 选择加载的图形驱动为 waylanddrv 时，初始化流程会进入 `waylanddrv_unix_init` 函数，完成 winewayland.drv 的整体初始化。

本文将沿着这一入口函数，详细分析 `waylanddrv_unix_init` 内部具体完成了哪些初始化工作，以及 Wine 是如何建立起与 Wayland 合成器之间的基础通信环境的。

`waylanddrv_unix_init` 定义在 `waylanddrv_main.c` 文件中，其核心逻辑可以概括为三个步骤：

* 通过 `__wine_set_user_driver` 注册用户驱动函数表
* 调用 `wayland_init_process_name()` 初始化进程名称
* 调用 `wayland_process_init()`，建立与 Wayland 合成器的连接并完成协议初始化

其中，真正承担 Wayland 环境搭建与协议协商的是 `wayland_process_init`，因此后文将重点围绕该函数展开。

## Wayland 进程初始化（`wayland_process_init`）

`wayland_process_init` 负责完成 Wine Wayland 驱动在进程级别的初始化，其主要步骤包括：

* 连接 Wayland 显示服务器：调用 `wl_display_connect(NULL)`
* 创建专用事件队列：通过 `wl_display_create_queue` 创建独立的 event queue
* 获取 Wayland 注册表：调用 `wl_display_get_registry`
* 注册注册表监听器：添加 `registry_listener`，用于接收和处理全局对象（global）的发布与移除事件

在这一过程中，有一段代码非常值得注意：

```c
    /* We need two roundtrips. One to get and bind globals, one to handle all
     * initial events produced from registering the globals. */
    wl_display_roundtrip_queue(process_wayland.wl_display, process_wayland.wl_event_queue);
    wl_display_roundtrip_queue(process_wayland.wl_display, process_wayland.wl_event_queue);
```

可以看到，`wl_display_roundtrip_queue` 被连续调用了两次。这并非冗余代码，而是 Wayland 初始化阶段中一个非常典型且必要的同步模式。

`wl_display_roundtrip_queue` 的语义是：

> 确保在调用 roundtrip 之前发出的所有 request，都已经被 compositor 处理，并且相关 event 已经进入指定的 event queue。

第一次 roundtrip 的作用在于：

* 确保 compositor 发送的所有 `wl_registry.global` 事件已经到达
* 确保 `registry_listener.global()` 回调已经被调用
* 在回调中完成对各类全局对象的 `bind`

然而问题在于：bind 本身并不是初始化流程的终点。

在 `registry_handle_global` 中，Wine 会对感兴趣的全局对象进行绑定，例如：

```c
seat->wl_seat = wl_registry_bind(registry, id, &wl_seat_interface,
                                 version < 5 ? version : 5);
wl_seat_add_listener(seat->wl_seat, &seat_listener, NULL);
```

这些 `bind` 操作会立即触发 compositor 发送新的事件，例如：

* `wl_seat.capabilities`

这些事件并不属于最初的 registry 查询结果，而是**绑定协议对象之后产生的初始状态事件**。因此，必须通过第二次 roundtrip，确保这些事件也被完整接收和处理。

换句话说：

* 第一次 roundtrip：发现并绑定全局对象
* 第二次 roundtrip：处理绑定动作引发的初始状态事件

这正是 Wayland 初始化中常见的“两次 roundtrip”模式。

## 注册表处理（Registry Handling）

需要注意的是，Wayland 中的“注册表”与 Windows 下的注册表并非同一概念。

在 Wayland 体系中，registry 的本质是：

> 客户端与合成器协商可用协议的一种机制

通过 registry，客户端可以获知 compositor 当前支持哪些全局对象，并选择性地绑定相应的协议接口。

在 winewayland.drv 中，注册表监听器负责识别并绑定多个关键协议，包括但不限于：

* `wl_compositor` —— 基础合成器接口
* `xdg_wm_base` —— 窗口管理协议
* `wl_shm` —— 共享内存缓冲区
* `wl_seat` —— 输入设备抽象（键盘、鼠标等）
* `wp_viewporter` —— 视口裁剪与缩放
* `zwp_pointer_constraints_v1` —— 指针约束（锁定 / 限制）
* `zwp_relative_pointer_manager_v1` —— 相对指针运动
* `zwp_text_input_manager_v3` —— 文本输入法
* 各类剪贴板相关协议

这些协议共同构成了 Wine 在 Wayland 下运行 Windows 应用所需的基础能力集合。

## 输入设备初始化

当 `wl_seat` 的能力发生变化时，Wayland 会通过 `wl_seat.capabilities` 通知客户端。Wine 正是通过该回调，动态初始化或释放输入设备对象：

```c
static void wl_seat_handle_capabilities(void *data, struct wl_seat *seat,
                                        enum wl_seat_capability caps)
{
    if ((caps & WL_SEAT_CAPABILITY_POINTER) && !process_wayland.pointer.wl_pointer)
        wayland_pointer_init(wl_seat_get_pointer(seat));
    else if (!(caps & WL_SEAT_CAPABILITY_POINTER) && process_wayland.pointer.wl_pointer)
        wayland_pointer_deinit();

    if ((caps & WL_SEAT_CAPABILITY_KEYBOARD) && !process_wayland.keyboard.wl_keyboard)
        wayland_keyboard_init(wl_seat_get_keyboard(seat));
    else if (!(caps & WL_SEAT_CAPABILITY_KEYBOARD) && process_wayland.keyboard.wl_keyboard)
        wayland_keyboard_deinit();
}
```

根据 seat 当前支持的能力，分别初始化或销毁：

* 指针设备（鼠标）
* 键盘设备

这种设计也体现了 Wayland 对输入设备动态可变性的建模方式。

## 事件处理线程

完成初始化后，Wine 会创建一个专用线程，用于持续读取 Wayland 事件：

```c
static DWORD WINAPI wayland_read_events_thread(void *arg)
{
    WAYLANDDRV_UNIX_CALL(read_events, NULL);
    /* This thread terminates only if an unrecoverable error occurred
     * during event reading (e.g., the connection to the Wayland
     * compositor is broken). */
    ERR("Failed to read events from the compositor, terminating process\n");
    TerminateProcess(GetCurrentProcess(), 1);
    return 0;
}
```

事件读取的核心循环位于 Unix 层实现中：

```c
static NTSTATUS waylanddrv_unix_read_events(void *arg)
{
    while (wl_display_dispatch_queue(process_wayland.wl_display,
                                     process_wayland.wl_event_queue) != -1)
        continue;
    /* This function only returns on a fatal error, e.g., if our connection
     * to the Wayland server is lost. */
    return STATUS_UNSUCCESSFUL;
}
```

在文章《[从 X11 的角度理解 Wayland](https://mp.weixin.qq.com/s/JXTbg9VsrXqma3LQYSN7cQ)》中曾对比过 `wl_display_dispatch` 与 X11 中的 `XNextEvent`。

这里使用的是 `wl_display_dispatch_queue`，其关键区别在于：**是否使用默认事件队列**。Wine 为 Wayland 驱动单独维护了一条 event queue，从而避免与其他模块产生事件处理上的干扰。

## 初始化流程小结

下图总结了 winewayland.drv 从加载到完成 Wayland 环境初始化的整体流程：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202512/images/winewayland_inital_01.png)

如果有任何疑问，欢迎留言交流。

