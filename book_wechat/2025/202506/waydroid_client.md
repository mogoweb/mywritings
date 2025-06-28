# Talk is cheap. Show me the code.手搓一个 Wayland 客户端程序

前几天我写了一篇万字长文《[万字长文详解 Wayland 协议、架构]()》，但光讲协议分析难免有些枯燥。毕竟，程序员更信奉那句名言：Talk is cheap. Show me the code.

所以这篇文章不打算长篇大论，而是通过编写一个简单的 Wayland 客户端程序，带大家实际体验一下 Wayland 的“坑”与门道。

我们要开发的 Wayland 客户端非常简单，只需在窗口中显示一句 “Hello wayland”。其实，写图形界面程序一般推荐用 GTK、QT 这样的 GUI 框架，这样可以自动适配 X11、Wayland 等后端。但为了演示 Wayland 客户端的底层写法，这次我们选择“手搓”一个。当然，具体实现就交给 AI 助手来完成了。

很快，AI 就给出了第一个版本的 Wayland 客户端程序。

## 第一个版本

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <wayland-client.h>
#include <wayland-client-protocol.h>
#include <cairo.h>
#include <sys/mman.h>
#include <unistd.h>
#include <fcntl.h>

// 包含我们刚刚生成的头文件
#include "xdg-shell-client-protocol.h"

// 用于管理我们所有Wayland对象和状态的结构体
struct state {
    struct wl_display *display;
    struct wl_registry *registry;
    struct wl_compositor *compositor;
    struct wl_surface *surface;
    struct wl_shm *shm;

    struct xdg_wm_base *xdg_wm_base;
    struct xdg_surface *xdg_surface;
    struct xdg_toplevel *xdg_toplevel;

    struct wl_buffer *buffer;
    void *shm_data;

    int width, height;
    _Bool running;
};

// 绘制函数
static void draw_frame(struct state *state) {
    int stride = cairo_format_stride_for_width(CAIRO_FORMAT_ARGB32, state->width);
    int size = stride * state->height;

    // 清空缓冲区内存
    memset(state->shm_data, 0, size);

    // 使用Cairo在共享内存上创建表面
    cairo_surface_t *cairo_surface = cairo_image_surface_create_for_data(
        state->shm_data, CAIRO_FORMAT_ARGB32, state->width, state->height, stride);
    cairo_t *cr = cairo_create(cairo_surface);

    // 绘制背景 (淡蓝色)
    cairo_set_source_rgba(cr, 0.8, 0.9, 1.0, 1.0);
    cairo_paint(cr);

    // 绘制文字
    cairo_set_source_rgb(cr, 0.1, 0.1, 0.1);
    cairo_select_font_face(cr, "sans-serif", CAIRO_FONT_SLANT_NORMAL, CAIRO_FONT_WEIGHT_BOLD);
    cairo_set_font_size(cr, 40);

    cairo_text_extents_t extents;
    cairo_text_extents(cr, "Hello World!", &extents);
    cairo_move_to(cr, state->width/2.0 - extents.width/2.0, state->height/2.0);
    cairo_show_text(cr, "Hello World!");

    // 清理Cairo资源
    cairo_destroy(cr);
    cairo_surface_destroy(cairo_surface);

    // 将绘制好的缓冲区附加到表面
    wl_surface_attach(state->surface, state->buffer, 0, 0);
    // 告诉合成器表面的哪个区域被更新了 (这里是整个表面)
    wl_surface_damage_buffer(state->surface, 0, 0, state->width, state->height);
    // 提交更改，让合成器显示
    wl_surface_commit(state->surface);
}


// --- xdg_toplevel 事件监听器 ---
static void xdg_toplevel_handle_configure(void *data, struct xdg_toplevel *xdg_toplevel,
                                          int32_t width, int32_t height, struct wl_array *states) {
    struct state *state = data;
    if (width > 0 && height > 0) {
        state->width = width;
        state->height = height;
    }
    // 注意: 我们不在这里绘图，因为我们会在xdg_surface的configure事件后绘图
}

static void xdg_toplevel_handle_close(void *data, struct xdg_toplevel *xdg_toplevel) {
    struct state *state = data;
    // 合成器通知我们用户点击了关闭按钮
    state->running = 0;
}

static const struct xdg_toplevel_listener xdg_toplevel_listener = {
    .configure = xdg_toplevel_handle_configure,
    .close = xdg_toplevel_handle_close,
};

// --- xdg_surface 事件监听器 ---
static void xdg_surface_handle_configure(void *data, struct xdg_surface *xdg_surface, uint32_t serial) {
    struct state *state = data;
    // 必须确认配置事件
    xdg_surface_ack_configure(xdg_surface, serial);
    // 在收到配置后，我们就可以绘图了
    draw_frame(state);
}

static const struct xdg_surface_listener xdg_surface_listener = {
    .configure = xdg_surface_handle_configure,
};

// --- xdg_wm_base 事件监听器 ---
static void xdg_wm_base_handle_ping(void *data, struct xdg_wm_base *xdg_wm_base, uint32_t serial) {
    // 客户端必须响应ping事件，否则合成器会认为客户端无响应
    xdg_wm_base_pong(xdg_wm_base, serial);
}

static const struct xdg_wm_base_listener xdg_wm_base_listener = {
    .ping = xdg_wm_base_handle_ping,
};


// --- wl_registry 事件监听器 ---
static void registry_handle_global(void *data, struct wl_registry *registry, uint32_t name,
                                   const char *interface, uint32_t version) {
    struct state *state = data;
    printf("Got a registry event for %s id %u\n", interface, name);

    if (strcmp(interface, wl_compositor_interface.name) == 0) {
        state->compositor = wl_registry_bind(registry, name, &wl_compositor_interface, 4);
    } else if (strcmp(interface, wl_shm_interface.name) == 0) {
        state->shm = wl_registry_bind(registry, name, &wl_shm_interface, 1);
    } else if (strcmp(interface, xdg_wm_base_interface.name) == 0) {
        state->xdg_wm_base = wl_registry_bind(registry, name, &xdg_wm_base_interface, 1);
        xdg_wm_base_add_listener(state->xdg_wm_base, &xdg_wm_base_listener, state);
    }
}

static void registry_handle_global_remove(void *data, struct wl_registry *registry, uint32_t name) {
    // This space is for rent
}

static const struct wl_registry_listener registry_listener = {
    .global = registry_handle_global,
    .global_remove = registry_handle_global_remove,
};

// 创建共享内存缓冲区
static int create_shm_buffer(struct state *state) {
    // 使用 memfd_create 创建一个匿名的、基于内存的文件
    char tmp_name[] = "/tmp/wayland-shm-XXXXXX";
    int fd = mkstemp(tmp_name);
    if (fd < 0) {
        fprintf(stderr, "mkstemp failed\n");
        return -1;
    }
    // 立即删除文件名，文件描述符依然有效
    unlink(tmp_name);

    int stride = cairo_format_stride_for_width(CAIRO_FORMAT_ARGB32, state->width);
    int size = stride * state->height;

    if (ftruncate(fd, size) < 0) {
        close(fd);
        fprintf(stderr, "ftruncate failed\n");
        return -1;
    }

    // 将文件映射到内存
    state->shm_data = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (state->shm_data == MAP_FAILED) {
        close(fd);
        fprintf(stderr, "mmap failed\n");
        return -1;
    }

    // 从文件描述符创建Wayland共享内存池
    struct wl_shm_pool *pool = wl_shm_create_pool(state->shm, fd, size);
    state->buffer = wl_shm_pool_create_buffer(pool, 0, state->width, state->height, stride, WL_SHM_FORMAT_ARGB8888);
    
    wl_shm_pool_destroy(pool);
    close(fd);
    return 0;
}

int main(int argc, char **argv) {
    struct state state = {0};
    state.width = 640;
    state.height = 480;
    state.running = 1;

    // 1. 连接到Wayland display
    state.display = wl_display_connect(NULL);
    if (state.display == NULL) {
        fprintf(stderr, "Can't connect to a Wayland display\n");
        return 1;
    }

    // 2. 获取registry，用于发现全局对象
    state.registry = wl_display_get_registry(state.display);
    wl_registry_add_listener(state.registry, &registry_listener, &state);

    // 3. 同步，等待服务器处理我们的请求并发送全局对象事件
    wl_display_dispatch(state.display);
    wl_display_roundtrip(state.display);

    // 检查是否成功绑定了必要的全局对象
    if (state.compositor == NULL || state.shm == NULL || state.xdg_wm_base == NULL) {
        fprintf(stderr, "Can't find compositor, shm or xdg_wm_base\n");
        return 1;
    }

    // 4. 创建Wayland表面
    state.surface = wl_compositor_create_surface(state.compositor);

    // 5. 通过xdg-shell将表面设置为toplevel窗口
    state.xdg_surface = xdg_wm_base_get_xdg_surface(state.xdg_wm_base, state.surface);
    xdg_surface_add_listener(state.xdg_surface, &xdg_surface_listener, &state);
    state.xdg_toplevel = xdg_surface_get_toplevel(state.xdg_surface);
    xdg_toplevel_add_listener(state.xdg_toplevel, &xdg_toplevel_listener, &state);

    // 设置窗口标题
    xdg_toplevel_set_title(state.xdg_toplevel, "Wayland Hello World");

    // 提交表面，让xdg-shell知道我们已经配置好了
    wl_surface_commit(state.surface);

    // 6. 创建共享内存缓冲区用于绘图
    if (create_shm_buffer(&state) < 0) {
        fprintf(stderr, "Failed to create shm buffer\n");
        return 1;
    }

    // 7. 主事件循环
    while (state.running && wl_display_dispatch(state.display) != -1) {
        // 事件处理都在监听器回调中完成
    }

    // 8. 清理资源
    printf("Cleaning up...\n");
    if (state.buffer) wl_buffer_destroy(state.buffer);
    if (state.xdg_toplevel) xdg_toplevel_destroy(state.xdg_toplevel);
    if (state.xdg_surface) xdg_surface_destroy(state.xdg_surface);
    if (state.surface) wl_surface_destroy(state.surface);
    if (state.xdg_wm_base) xdg_wm_base_destroy(state.xdg_wm_base);
    if (state.shm) wl_shm_destroy(state.shm);
    if (state.compositor) wl_compositor_destroy(state.compositor);
    if (state.registry) wl_registry_destroy(state.registry);
    if (state.display) wl_display_disconnect(state.display);

    return 0;
}
```
并且给出了详细的编译步骤。

1. **安装依赖**

   打开终端，输入以下命令安装所有必需的开发包：
   ```bash
   sudo apt update
   sudo apt install libwayland-dev libcairo2-dev cmake build-essential wayland-protocols
   ```

2. **生成 xdg-shell 协议相关文件**

   进入项目目录，执行以下命令：
   ```bash
   # 生成头文件
   wayland-scanner client-header /usr/share/wayland-protocols/stable/xdg-shell/xdg-shell.xml xdg-shell-client-protocol.h
   # 生成C源文件
   wayland-scanner private-code /usr/share/wayland-protocols/stable/xdg-shell/xdg-shell.xml xdg-shell-protocol.c
   ```
   生成的 `xdg-shell-client-protocol.h` 和 `xdg-shell-protocol.c` 文件请放到本项目目录下。

3. **获取源码**

   假设你已经有本项目的源码目录 `WaylandClientDemo`。

4. **编译程序**

   在项目目录下执行：
   ```bash
   cd WaylandClientDemo
   mkdir build && cd build
   cmake ..
   make
   ```
   编译成功后会生成 `hello-wayland-v1` 可执行文件。

5. **运行程序**

   保证你当前在Wayland桌面环境下，执行：
   ```bash
   ./hello-wayland-v1
   ```
   程序会弹出一个窗口，显示“Hello Wayland”文字，10秒后自动关闭。

运行后会创建一个Wayland窗口，窗口中间显示“Hello Wayland”文字。 

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/wayland_client_01.png)

等等，为什么这个窗口程序不是我们常见的窗口样式，没有边框，没有标题，也没有最大化、最小化和关闭按钮？

这是由于在 Wayland 体系下，窗口的“装饰”（即边框、标题栏、按钮等）有两种方式：

1. **服务端装饰（SSD, Server Side Decoration）**
  - 由合成器自动为顶层窗口（`xdg_toplevel`）添加装饰。
  - 但有些合成器默认不加装饰。
2. **客户端装饰（CSD, Client Side Decoration）**
  - 由应用程序自己绘制装饰（如 GTK、Chrome 浏览器等）。

但这就造成一种尴尬的情形，客户端和服务器端都以为对方会绘制窗口装饰，结果都没有绘制。

为了解决这种矛盾，人们就想出了一个新方法，干脆协商一下吧，商量到底谁来绘制这个窗口装饰。于是 Wayland 定义了一个专门的扩展协议，叫 `xdg-decoration`（全名 `xdg-decoration-unstable-v1`），用于**协商窗口装饰的方式**。如果合成器支持该协议，客户端可以请求服务端为窗口添加装饰（SSD），否则需要自己绘制（CSD）。

## 第二个版本

将诉求丢给 AI 后，很快就出了第二个版本：

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <wayland-client.h>
#include <wayland-client-protocol.h>
#include <cairo.h>
#include <sys/mman.h>
#include <unistd.h>
#include <fcntl.h>

// 包含我们刚刚生成的头文件
#include "xdg-shell-client-protocol.h"
#include "xdg-decoration-client-protocol.h" // <<< 引入新生成的头文件

// 用于管理我们所有Wayland对象和状态的结构体
struct state {
    struct wl_display *display;
    struct wl_registry *registry;
    struct wl_compositor *compositor;
    struct wl_surface *surface;
    struct wl_shm *shm;

    struct xdg_wm_base *xdg_wm_base;
    struct xdg_surface *xdg_surface;
    struct xdg_toplevel *xdg_toplevel;

    // <<< 新增：用于xdg-decoration的对象
    struct zxdg_decoration_manager_v1 *decoration_manager;
    struct zxdg_toplevel_decoration_v1 *toplevel_decoration;

    struct wl_buffer *buffer;
    void *shm_data;

    int width, height;
    _Bool running;
};

// 绘制函数
static void draw_frame(struct state *state) {
    int stride = cairo_format_stride_for_width(CAIRO_FORMAT_ARGB32, state->width);
    int size = stride * state->height;

    // 清空缓冲区内存
    memset(state->shm_data, 0, size);

    // 使用Cairo在共享内存上创建表面
    cairo_surface_t *cairo_surface = cairo_image_surface_create_for_data(
        state->shm_data, CAIRO_FORMAT_ARGB32, state->width, state->height, stride);
    cairo_t *cr = cairo_create(cairo_surface);

    // 绘制背景 (淡蓝色)
    cairo_set_source_rgba(cr, 0.8, 0.9, 1.0, 1.0);
    cairo_paint(cr);

    // 绘制文字
    cairo_set_source_rgb(cr, 0.1, 0.1, 0.1);
    cairo_select_font_face(cr, "sans-serif", CAIRO_FONT_SLANT_NORMAL, CAIRO_FONT_WEIGHT_BOLD);
    cairo_set_font_size(cr, 40);

    cairo_text_extents_t extents;
    cairo_text_extents(cr, "Hello World!", &extents);
    cairo_move_to(cr, state->width/2.0 - extents.width/2.0, state->height/2.0);
    cairo_show_text(cr, "Hello World!");

    // 清理Cairo资源
    cairo_destroy(cr);
    cairo_surface_destroy(cairo_surface);

    // 将绘制好的缓冲区附加到表面
    wl_surface_attach(state->surface, state->buffer, 0, 0);
    // 告诉合成器表面的哪个区域被更新了 (这里是整个表面)
    wl_surface_damage_buffer(state->surface, 0, 0, state->width, state->height);
    // 提交更改，让合成器显示
    wl_surface_commit(state->surface);
}


// --- xdg_toplevel 事件监听器 ---
static void xdg_toplevel_handle_configure(void *data, struct xdg_toplevel *xdg_toplevel,
                                          int32_t width, int32_t height, struct wl_array *states) {
    struct state *state = data;
    if (width > 0 && height > 0) {
        state->width = width;
        state->height = height;
    }
    // 注意: 我们不在这里绘图，因为我们会在xdg_surface的configure事件后绘图
}

static void xdg_toplevel_handle_close(void *data, struct xdg_toplevel *xdg_toplevel) {
    struct state *state = data;
    // 合成器通知我们用户点击了关闭按钮
    state->running = 0;
}

static const struct xdg_toplevel_listener xdg_toplevel_listener = {
    .configure = xdg_toplevel_handle_configure,
    .close = xdg_toplevel_handle_close,
};

// --- xdg_surface 事件监听器 ---
static void xdg_surface_handle_configure(void *data, struct xdg_surface *xdg_surface, uint32_t serial) {
    struct state *state = data;
    // 必须确认配置事件
    xdg_surface_ack_configure(xdg_surface, serial);
    // 在收到配置后，我们就可以绘图了
    draw_frame(state);
}

static const struct xdg_surface_listener xdg_surface_listener = {
    .configure = xdg_surface_handle_configure,
};

// --- xdg_wm_base 事件监听器 ---
static void xdg_wm_base_handle_ping(void *data, struct xdg_wm_base *xdg_wm_base, uint32_t serial) {
    // 客户端必须响应ping事件，否则合成器会认为客户端无响应
    xdg_wm_base_pong(xdg_wm_base, serial);
}

static const struct xdg_wm_base_listener xdg_wm_base_listener = {
    .ping = xdg_wm_base_handle_ping,
};

// <<< 新增：xdg_toplevel_decoration 的事件监听器
static void decoration_handle_configure(void *data,
                                        struct zxdg_toplevel_decoration_v1 *decoration,
                                        uint32_t mode)
{
    // 这是协商的核心！合成器通过这个事件告诉我们它最终决定的装饰模式。
    printf("==> Compositor negotiated decoration mode: ");
    switch (mode) {
        case ZXDG_TOPLEVEL_DECORATION_V1_MODE_CLIENT_SIDE:
            printf("Client-Side (we must draw our own!)\n");
            break;
        case ZXDG_TOPLEVEL_DECORATION_V1_MODE_SERVER_SIDE:
            printf("Server-Side (compositor will draw for us!)\n");
            break;
        default:
            printf("Unknown\n");
            break;
    }
}

static const struct zxdg_toplevel_decoration_v1_listener decoration_listener = {
    .configure = decoration_handle_configure,
};


// --- wl_registry 事件监听器 ---
static void registry_handle_global(void *data, struct wl_registry *registry, uint32_t name,
                                   const char *interface, uint32_t version) {
    struct state *state = data;
    // <<< 我们现在打印所有接口，方便调试
    printf("Found global: %s (version %u)\n", interface, version);

    if (strcmp(interface, wl_compositor_interface.name) == 0) {
        state->compositor = wl_registry_bind(registry, name, &wl_compositor_interface, 4);
    } else if (strcmp(interface, wl_shm_interface.name) == 0) {
        state->shm = wl_registry_bind(registry, name, &wl_shm_interface, 1);
    } else if (strcmp(interface, xdg_wm_base_interface.name) == 0) {
        state->xdg_wm_base = wl_registry_bind(registry, name, &xdg_wm_base_interface, 1);
        xdg_wm_base_add_listener(state->xdg_wm_base, &xdg_wm_base_listener, state);
    }
    // <<< 新增：检查合成器是否支持 xdg-decoration 协议
    else if (strcmp(interface, zxdg_decoration_manager_v1_interface.name) == 0) {
        state->decoration_manager = wl_registry_bind(registry, name, &zxdg_decoration_manager_v1_interface, 1);
    }
}

static void registry_handle_global_remove(void *data, struct wl_registry *registry, uint32_t name) {
    // This space is for rent
}

static const struct wl_registry_listener registry_listener = {
    .global = registry_handle_global,
    .global_remove = registry_handle_global_remove,
};

// 创建共享内存缓冲区
static int create_shm_buffer(struct state *state) {
    // 使用 memfd_create 创建一个匿名的、基于内存的文件
    char tmp_name[] = "/tmp/wayland-shm-XXXXXX";
    int fd = mkstemp(tmp_name);
    if (fd < 0) {
        fprintf(stderr, "mkstemp failed\n");
        return -1;
    }
    // 立即删除文件名，文件描述符依然有效
    unlink(tmp_name);

    int stride = cairo_format_stride_for_width(CAIRO_FORMAT_ARGB32, state->width);
    int size = stride * state->height;

    if (ftruncate(fd, size) < 0) {
        close(fd);
        fprintf(stderr, "ftruncate failed\n");
        return -1;
    }

    // 将文件映射到内存
    state->shm_data = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (state->shm_data == MAP_FAILED) {
        close(fd);
        fprintf(stderr, "mmap failed\n");
        return -1;
    }

    // 从文件描述符创建Wayland共享内存池
    struct wl_shm_pool *pool = wl_shm_create_pool(state->shm, fd, size);
    state->buffer = wl_shm_pool_create_buffer(pool, 0, state->width, state->height, stride, WL_SHM_FORMAT_ARGB8888);
    
    wl_shm_pool_destroy(pool);
    close(fd);
    return 0;
}

int main(int argc, char **argv) {
    struct state state = {0};
    state.width = 640;
    state.height = 480;
    state.running = 1;

    // 1. 连接到Wayland display
    state.display = wl_display_connect(NULL);
    if (state.display == NULL) {
        fprintf(stderr, "Can't connect to a Wayland display\n");
        return 1;
    }

    // 2. 获取registry，用于发现全局对象
    state.registry = wl_display_get_registry(state.display);
    wl_registry_add_listener(state.registry, &registry_listener, &state);

    // 3. 同步，等待服务器处理我们的请求并发送全局对象事件
    wl_display_dispatch(state.display);
    wl_display_roundtrip(state.display);

    // 检查是否成功绑定了必要的全局对象
    if (state.compositor == NULL || state.shm == NULL || state.xdg_wm_base == NULL) {
        fprintf(stderr, "Can't find compositor, shm or xdg_wm_base\n");
        return 1;
    }

    // 4. 创建Wayland表面
    state.surface = wl_compositor_create_surface(state.compositor);

    // 5. 通过xdg-shell将表面设置为toplevel窗口
    state.xdg_surface = xdg_wm_base_get_xdg_surface(state.xdg_wm_base, state.surface);
    xdg_surface_add_listener(state.xdg_surface, &xdg_surface_listener, &state);
    state.xdg_toplevel = xdg_surface_get_toplevel(state.xdg_surface);
    xdg_toplevel_add_listener(state.xdg_toplevel, &xdg_toplevel_listener, &state);

    // 设置窗口标题
    xdg_toplevel_set_title(state.xdg_toplevel, "Wayland Hello World");

    // <<< 新增：进行装饰协商
    if (state.decoration_manager) {
        printf("Decoration manager found. Negotiating...\n");
        // 为我们的窗口获取一个装饰对象
        state.toplevel_decoration = zxdg_decoration_manager_v1_get_toplevel_decoration(
            state.decoration_manager, state.xdg_toplevel);
        // 添加监听器以接收合成器的决定
        zxdg_toplevel_decoration_v1_add_listener(
            state.toplevel_decoration, &decoration_listener, &state);
        // *明确地*请求服务器端装饰
        zxdg_toplevel_decoration_v1_set_mode(
            state.toplevel_decoration, ZXDG_TOPLEVEL_DECORATION_V1_MODE_SERVER_SIDE);
    } else {
        printf("Decoration manager not found. Cannot negotiate decorations.\n");
    }

    // 提交表面，让xdg-shell知道我们已经配置好了
    wl_surface_commit(state.surface);

    // 6. 创建共享内存缓冲区用于绘图
    if (create_shm_buffer(&state) < 0) {
        fprintf(stderr, "Failed to create shm buffer\n");
        return 1;
    }

    // 7. 主事件循环
    while (state.running && wl_display_dispatch(state.display) != -1) {
        // 事件处理都在监听器回调中完成
    }

    // 8. 清理资源
    printf("Cleaning up...\n");
    // <<< 新增：清理装饰相关的对象
    if (state.toplevel_decoration) zxdg_toplevel_decoration_v1_destroy(state.toplevel_decoration);
    if (state.decoration_manager) zxdg_decoration_manager_v1_destroy(state.decoration_manager);
    if (state.buffer) wl_buffer_destroy(state.buffer);
    if (state.xdg_toplevel) xdg_toplevel_destroy(state.xdg_toplevel);
    if (state.xdg_surface) xdg_surface_destroy(state.xdg_surface);
    if (state.surface) wl_surface_destroy(state.surface);
    if (state.xdg_wm_base) xdg_wm_base_destroy(state.xdg_wm_base);
    if (state.shm) wl_shm_destroy(state.shm);
    if (state.compositor) wl_compositor_destroy(state.compositor);
    if (state.registry) wl_registry_destroy(state.registry);
    if (state.display) wl_display_disconnect(state.display);

    return 0;
}
```

### 典型流程

1. 客户端通过 `wl_registry` 获取 `zxdg_decoration_manager_v1`。
2. 用 `zxdg_decoration_manager_v1` 为 `xdg_toplevel` 创建 decoration 对象。
3. 调用 `zxdg_toplevel_decoration_v1_request_mode()` 请求 SSD。
4. 合成器响应，决定是否提供装饰。

运行效果图如下：
![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/wayland_client_02.png)

是一个正常的窗口，和我们的预期相符。然而，这个程序在 Ubuntu 24.04 下运行的效果图：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/wayland_client_03.png)

仍然没有窗口装饰。其实从 xdg-decoration 扩展的文件名 xdg-decoration-unstable-v1.xml 就可以看出，该扩展协议还处在 unstable 状态，然后 Ubuntu 使用的合成器 Mutter 没实现该扩展协议。这种情况下，就需要 Wayland 客户端来绘制。

## 小结

本文通过手搓 Wayland 客户端的实践，带你从零体验了 Wayland 协议下窗口程序的开发流程。我们首先实现了一个最基础的“Hello Wayland”窗口，随后又引入了 xdg-decoration 协议，尝试与合成器协商窗口装饰的绘制方式。通过实际运行和对比不同环境下的效果，你可以直观感受到 Wayland 生态中窗口装饰的多样性与复杂性。

Wayland 的窗口装饰机制分为服务端装饰（SSD）和客户端装饰（CSD），而 xdg-decoration 协议则是二者协商的桥梁。但由于协议本身还处于 unstable 阶段，不同合成器的支持情况并不一致，导致实际效果会有差异。比如在 Ubuntu 24.04 的 Mutter 合成器下，仍需客户端自行绘制装饰。

总之，Wayland 生态仍在不断发展，协议和实现也在持续完善。对于开发者来说，理解其原理和流程，有助于更好地适配和优化自己的应用。希望本文的代码和讲解，能为你打开 Wayland 世界的一扇窗。
