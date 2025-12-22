# Wine 中 GDI 绘制的实现原理分析与架构解读

在上一篇文章《[Wine 是如何加载图形驱动的？](https://mp.weixin.qq.com/s/GdydTFRzmYw4xXUeSzSPww)》中，我们探讨了 Wine 如何通过其精巧的架构，适配多种不同的窗口系统与图形后端。本文将在此基础上进一步深入，具体分析 Wine 是如何将 Windows 中的 GDI 绘制功能转换并适配到不同后端实现上的。

GDI（Graphics Device Interface）是伴随着 Windows 而出现的，从现在的眼光看，似乎有些落伍。随着现代 Windows UI 框架 WPF / UWP / WinUI / XAML 等的出现，似乎 GDI 早就该退出历史舞台了。但别忘了，Windows 的核心竞争力之一是：

> 20 年前的程序，今天还能跑。

GDI 是 Win32 API 最稳定、最早被广泛使用的一部分之一。可以说 GDI 是 Windows 桌面图形体系的地基，而 DirectX、WPF、现代 UI 框架，只是在其之上不断扩展性能和表现力，并不是替代之。所以在 Wine、远程桌面、兼容层这些领域里，理解 GDI 依然非常重要。

## 一、GDI 在 Windows 中的作用是什么？

### 1.1 GDI 是传统 2D 桌面绘制接口

GDI 是 Windows 最早期、也是最经典的图形接口之一，其核心目标是：

> 为应用提供一套与具体显示设备无关的 2D 绘制 API。

典型能力包括：

* 基本图形：Line、Rectangle、Ellipse
* 文本绘制：TextOut / DrawText
* 位图操作：BitBlt / StretchBlt
* 画刷、画笔、字体、裁剪区
* 打印机输出（这是 GDI 的重要设计目标之一）

**一次绘制，多种设备输出**是 GDI 设计时的核心理念。

### 1.2 GDI 的关键抽象：HDC（设备上下文）

所有 GDI 绘制都围绕 HDC（Handle to Device Context） 展开。

HDC 抽象了：

* 绘制目标（屏幕 / 窗口 / 内存 / 打印机）
* 当前状态（画笔、画刷、字体、变换、裁剪区）

应用只关心 HDC，不关心底层是：

* 显示器
* 打印机
* 内存位图

这也是 GDI 能长期存在的重要原因。

### 1.3 GDI 不擅长什么？

GDI 的设计限制包括：

* CPU 渲染为主（即使有硬件加速也非常有限）
* 不适合复杂动画
* 不适合 3D
* 不适合高帧率实时渲染
* DPI / 缩放支持是后来补充实现的

### 1.4 GDI+

GDI+ 并不是 GDI 的继任者，而是一个建立在 GDI 之上的补充层。GDI+ 的绘制最终仍然要落到 GDI（或底层设备）上：

```
应用
  ↓
GDI+ API
  ↓
GDI（HDC）
  ↓
显示设备 / 打印机
```

从实现上看：

* GDI 是操作系统内核/系统子系统的一部分
* GDI+ 是一个用户态库（gdiplus.dll）

## 二、`gdi_physdev` 是什么？

在 Wine 中，GDI 的绘制流程并不是直接对 X11 / Wayland / Vulkan 发命令，而是先经过一层物理设备抽象（physical device），这就是 `gdi_physdev`。

它本质上是一个 **函数表（ops 表）+ 设备上下文绑定对象**，用于解决：

> 当前这个 HDC，对应的实际绘制后端是什么？

每一种 GDI 输出目标，几乎都对应一种 gdi_physdev 实现。而且这些实现并非互斥，而是可以以责任链的形式叠加在一个 HDC 上。

常见的 `gdi_physdev` 实现包括：

* Null / Stub physdev（占位实现，作为默认兜底，保证 GDI 调用链不崩溃）
* X11（X11DRV_PDEVICE，x11drv.h）
* XRender / X11 的抗锯齿加速层（xrender_physdev，xrender.c）
* DIB / 设备无关位图（dibdrv_physdev，dibdrv.h）
* PostScript / 打印驱动（PSDRV_PDEVICE，unixlib.c）
* EMF / 增强型图元文件（EMFDRV_PDEVICE，emfdrv.c）：用于图元文件记录，内部保存了一组设备能力参数。
* 字体（font_physdev，font.c）
* 路径（path_physdev，path.c）：用于记录 GDI Path

### 2.1 `gdi_physdev` 是如何被“定位 / 选中”的？

`gdi_physdev` 的选择不是全局的，而是：

* 每一个 HDC 单独决定
* 在 DC 创建或首次使用时动态绑定

典型入口：

* `NtGdiCreateCompatibleDC`（对应 Windows 代码 CreateCompatibleDC）
* `NtGdiCreateDC`（对应 Windows 代码 CreateDC）
* `NtUserBeginPaint`（对应 Windows 代码 BeginPaint）

这些函数最终都会走到 DC 初始化路径。

### 2.2 DC 初始化过程中发生了什么？

在 DC 创建时，Wine 会构建一个 DC 对象，并维护一个 physdev 链表（注意：是链表）：

```c
typedef struct gdi_physdev
{
    const struct gdi_dc_funcs *funcs;
    struct gdi_physdev        *next;
    HDC                        hdc;
} *PHYSDEV;
```

physdev 的基础结构 gdi_physdev 定义包含三个关键字段：函数表指针、指向下一个设备的指针和 HDC 句柄。

再来看 DC（设备上下文）结构：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202512/images/wine_gdi_01.png)

维护了两个关键的 physdev 相关字段：

* nulldrv: 链表底部的空驱动（null driver）
* physDev: 指向当前驱动堆栈顶部的指针

这是 Wine GDI 中一个非常经典的责任链设计。

```text
DC
 ├─ physdev #0  （最上层）
 ├─ physdev #1
 ├─ physdev #2  （最底层，真正落地）
```

每一层 `physdev` 都可以：

* 拦截 GDI 操作
* 修改参数
* 决定是否继续向下传递

### 2.3 GDI 驱动栈架构

Wine 使用一种基于优先级的驱动栈结构，**驱动优先级决定了驱动入栈时，驱动在栈中的所在的层级**。每个驱动的优先级都有预先定义好的固定值，比如path_driver的优先级为400。

GDI 驱动入栈是由函数 `push_dc_driver` 来实现，入栈规则为：

* 优先级数值更高的驱动会被插入到更靠近链表头部的位置
* 优先级数值更低的驱动会被插入到更靠近链表尾部的位置

多个 physdev 以链表方式串联：

* 每个 DC 底部都有一个 nulldrv（优先级 0）
* 字体驱动优先级为 100
* 图形驱动优先级为 200
* DIB 驱动优先级为 300
* 路径驱动优先级为 400

当调用 GDI 函数时，会从栈顶向下遍历，直到某个驱动处理该调用。

大多数 GDI 驱动都遵循同一种模式：将 struct gdi_physdev dev 作为结构体的第一个成员。

```c
  /* X physical device */
typedef struct
{
    struct gdi_physdev dev;
    GC            gc;          /* X Window GC */
    Drawable      drawable;
    RECT          dc_rect;       /* DC rectangle relative to drawable */
    RECT         *bounds;        /* Graphics bounds */
    HRGN          region;        /* Device region (visible region & clip region) */
    X_PHYSPEN     pen;
    X_PHYSBRUSH   brush;
    int           depth;       /* bit depth of the DC */
    ColorShifts  *color_shifts; /* color shifts of the DC */
    int           exposures;   /* count of graphics exposures operations */
} X11DRV_PDEVICE;
```

这样可以安全地在基础类型与驱动特定类型之间进行类型转换，例如：

* get_x11drv_dev()
* get_xrender_dev()

## 三、Wayland下的 physdev 是如何实现的

前面在列举 `gdi_physdev` 实现时，我们会发现在 Wayland 下并没有类似于 X11 下 X11DRV_PDEVICE 或 xrender_physdev 这样的实现。这是由于 Wayland 的模型决定了：

* 客户端不能直接操作窗口
* 只能提交 buffer，由 compositor 决定何时显示

在 X11 下，x11drv_physdev 可以直接对 X drawable 进行绘制，GDI 调用可以立即生效。而在 Wayland 下，physdev 并不直接对 wl_surface 绘制，而是主要画到中间缓冲（buffer）上。

### 3.1 实际绘制：DIB（Device Independent Bitmap）驱动

前面分析过，Wine 的 GDI 系统采用了一种分层的驱动栈架构：多个 GDI 驱动按照优先级被压入同一个链表中，并在绘制时按顺序分发调用。

我们知道栈结构是最上层的节点先出栈，所以首先调用的的 Null driver，而 Null driver 会将调用转到下一个，在 Wayland 场景下就是 DIB 驱动，所以真正承担软件绘制工作的是 DIB 驱动。

DIB 驱动实现了大多数核心 GDI 绘制操作，例如 BitBlt、LineTo、Rectangle 等。

补充一点，对于窗口 DC，Wine 还额外引入了一层 window driver，它在 DIB 驱动之上进行封装，主要职责是为窗口表面访问提供线程安全的锁机制。

这种设计使得 Wayland 这样的显示驱动可以专注于窗口系统与合成器交互，而将复杂且平台无关的 GDI 绘制逻辑完全交由 Wine 内部处理。

### 3.2 DIB -> wl_buffer

在 Windows 世界，Window（窗口）是整个用户态体系的核心抽象，几乎所有图形、输入和交互能力，最终都围绕 Window 展开。

而 Wine 作为兼容层，需要考虑到 Linux / macOS 等 并不存在 HWND 这种万能对象，因为不同窗口系统对窗口和绘制目标的定义完全不同，因此 Wine 必须解耦。

于是在 Wine 的实现中：

* Window（逻辑窗口）：由 win32u / user32 管理
* window_surface（窗口表面）：由图形驱动管理

```
struct window_surface
{
    const struct window_surface_funcs *funcs; /* driver-specific implementations  */
    struct list                        entry; /* entry in global list managed by user32 */
    LONG                               ref;   /* reference count */
    HWND                               hwnd;  /* window the surface was created for */
    RECT                               rect;  /* constant, no locking needed */

    pthread_mutex_t                    mutex;        /* mutex needed for any field below */
    RECT                               bounds;       /* dirty area rectangle */
    HRGN                               clip_region;  /* visible region of the surface, fully visible if 0 */
    DWORD                              draw_start_ticks; /* start ticks of fresh draw */
    COLORREF                           color_key;    /* layered window surface color key, invalid if CLR_INVALID */
    UINT                               alpha_bits;   /* layered window global alpha bits, invalid if -1 */
    UINT                               alpha_mask;   /* layered window per-pixel alpha mask, invalid if 0 */
    HRGN                               shape_region; /* shape of the window surface, unshaped if 0 */
    HBITMAP                            shape_bitmap; /* bitmap for the surface shape (1bpp) */
    HBITMAP                            color_bitmap; /* bitmap for the surface colors */
    /* driver-specific fields here */
};
```

可以这样理解，window_surface 是 Wine 内部的一个抽象结构，用来表示**一个可被绘制、可被提交、可被合成的像素表面**。

Wayland 驱动实现了自己的一种窗口表面类型，称为 wayland_window_surface。该结构在基础的 window_surface 之上进行了扩展，并额外引入了一个 wayland_buffer_queue，用于管理 Wayland 侧的缓冲区。

```c
struct wayland_window_surface
{
    struct window_surface header;
    struct wayland_buffer_queue *wayland_buffer_queue;
    BOOL layered;
};
```

wayland_buffer_queue 负责维护一组 wayland_shm_buffer 对象的链表，而每一个 wayland_shm_buffer 内部都包含一个实际的 wl_buffer。

```c
struct wayland_buffer_queue
{
    struct wl_event_queue *wl_event_queue;
    struct wl_list buffer_list;
    int width;
    int height;
    uint32_t format;
};

struct wayland_shm_buffer
{
    struct wl_list link;
    struct wl_buffer *wl_buffer;
    int width, height;
    uint32_t format;
    void *map_data;
    size_t map_size;
    BOOL busy;
    LONG ref;
    HRGN damage_region;
};
```

每个 wayland_shm_buffer 包含以下信息：

* 实际的 wl_buffer
* 缓冲区宽高
* 像素格式
* 映射到用户态的共享内存地址
* 状态与生命周期跟踪信息（包括 busy 标志与引用计数）

缓冲队列会根据需要动态创建最多 3 个缓冲区。

当需要缓冲区时，系统会先查找一个未被占用（非 busy）的缓冲区；如果不存在可用缓冲区，则尝试创建新的缓冲区，或者阻塞等待 compositor 释放已有缓冲区。

从 DIB 到 wl_buffer 的具体流程为：

* 从缓冲队列中获取一个空闲的缓冲区

* 将 DIB（color_bits）中的像素数据拷贝到缓冲区映射的内存中

* 拷贝过程基于 region（区域）进行，仅更新发生变化的部分

这种架构实现了一种清晰的职责分离：

* DIB 驱动完全不了解任何 Wayland 相关细节，它只负责向标准的 window_surface 中的 color_bitmap 进行软件绘制。

* Wayland 特有的逻辑全部封装在 wayland_window_surface 中，由它负责将 DIB 的更新转换为 Wayland buffer 的 attach 与 commit 操作。

## 小结

Wine 中 GDI 绘制的实现原理展现了分层抽象与责任链模式的巧妙应用。通过物理设备驱动栈（gdi_physdev）的设计，Wine 成功地将 Windows GDI API 适配到多种不同的图形后端，包括 X11、Wayland 等。

这一架构的核心优势在于其灵活性和可扩展性：每个驱动只需关注特定功能的实现，通过驱动栈的优先级调度机制，各种驱动可以协同工作而互不干扰。在 Wayland 环境下，通过 DIB 驱动完成软件渲染，再结合 wayland_window_surface 的缓冲区管理，实现了与 Wayland 合成器的有效交互。

值得注意的是，尽管现代图形技术如 DirectX 和 Vulkan 日益普及，GDI 作为 Windows 图形基础设施的地位依然不可忽视。Wine 对 GDI 的精心实现不仅保证了传统应用的兼容性，也为理解跨平台图形兼容层设计提供了宝贵的技术范例。