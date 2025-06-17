# Waydroid 应用播放视频黑屏问题分析

俗话说“好记性不如烂笔头”，趁着记忆还新鲜，记录下这次排查 Waydroid 中视频播放黑屏问题的过程，希望能对你有所帮助。

## 背景说明

在 Linux 系统上运行 Android 应用程序有多种方案，具体可参考我之前的文章：

先交代一下背景，在 Linux 系统上运行 Android 应用程序，有很多种方案，请参考我之前的文章：

* [Linux系统运行Android应用的几种方案](https://mp.weixin.qq.com/s?__biz=MzI3NTQyMzEzNQ==&mid=2247489308&idx=1&sn=6dc8bde808b83c7077fdb11e5f03eb81&scene=21#wechat_redirect)

由于 Anbox 项目已经停止维护，我们采用了其推荐的替代方案——Waydroid。然而在实际使用过程中，我们发现一个棘手的问题：许多 Android 应用在播放视频时没有图像，呈现黑屏，只有极少数应用可以正常显示视频画面。如今几乎每个 Android App 都集成了视频功能，因此解决视频播放黑屏的问题势在必行。

## 初步分析

Android 应用播放视频主要有两种实现方式：

* 使用 Android 系统内置的媒体框架；

* 采用自研的媒体播放引擎（例如基于 FFmpeg）。

像字节跳动、哔哩哔哩这类以视频为核心的大厂，通常会开发自研的视频播放框架，以满足秒开、低延迟、滤镜特效等业务需求。而对于新闻、阅读类等非视频为主的应用，视频功能更多是辅助性质，这类应用往往直接调用 Android 媒体框架来实现。

由于自研框架结构复杂且不可获取源码，本次我们聚焦于 Android 原生媒体框架在 Waydroid 中的兼容问题。为便于分析，我编写了一个简单的 Android 视频播放器应用（其实是 AI 帮我写的），并首先在 Android 模拟器中验证了其正常播放效果，如下图所示：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/waydroid_video_01.png)

然而同样的 App 放到 Waydroid 中运行时，播放视频时却只剩下声音，画面全黑。幸运的是，这个播放器 App 是自己写的，Android 系统镜像是自己编译的，Waydroid 项目也是开源的，整个技术栈尽在掌控之中。于是我开始逐步分析和定位问题。

## 日志分析

为输出更多调试信息，我编译了 AOSP 的 eng 版本镜像，并替换到 Waydroid 环境中。之后通过如下命令抓取日志：

```
sudo waydroid logcat > demo-full.log
```

输出的日志非常多，看起来头晕，这种事情当然可以交给 AI，AI 做这个事情最拿手。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202506/images/waydroid_video_02.png)

找出了问题的核心：图形缓冲区分配失败 (11.583 - 结尾)

* E GraphicBufferAllocator: Failed to allocate (640 x 480) ...
 - 这是最关键的错误日志。GraphicBufferAllocator 是负责为图形渲染分配内存的系统服务。
 - 它尝试分配一个 640 x 480 分辨率的图形缓冲区失败了。
* E BufferQueueProducer: ... dequeueBuffer: createGraphicBuffer failed
 - BufferQueueProducer 是图形缓冲区的生产者（这里是视频解码器）。它想从队列中取出一个空闲的缓冲区来填充解码后的视频帧，但因为上一步分配失败，所以它也失败了。
 - [TID:51#org.deepin.demo.mediaplayerdemo/...] 这明确指出了问题与你的应用相关联。
* W SoftwareRenderer: Surface::dequeueBuffer returned error -12
 - SoftwareRenderer 是负责将解码后的帧画到 Surface 上的组件。它收到了来自底层的错误码 -12 (即 ENOMEM，内存不足)。
* 重复出现: 你可以看到，这三条错误日志在后面反复出现。这意味着解码器在尝试渲染每一帧视频时，都在请求内存，但每一次都失败了。

这个时候，去看一看源码最高效，不要让 AI 继续分析。

## 源码分析

GraphicBufferAllocator 是 Android 图形系统的核心组件之一，负责分配用于屏幕渲染的图形缓冲区（Graphic Buffers），其代码位于 AOSP 的 frameworks/native/libs/ui/GraphicBufferAllocator.cpp。

在其 GraphicBufferAllocator::allocateHelper 方法中，看到了如下的代码：

```
    status_t error = mAllocator->allocate(requestorName, width, height, format, layerCount, usage,
                                          1, stride, handle, importBuffer);
    if (error != NO_ERROR) {
        ALOGE("Failed to allocate (%u x %u) layerCount %u format %d "
              "usage %" PRIx64 ": %d",
              width, height, layerCount, format, usage, error);
        return NO_MEMORY;
    }
```

这和 log 输出对应上了，就是这个 mAllocator->allocate 出错。

但找到 mAllocator 的实现并不容易，因为 Android 系统要应对各种硬件特性，采取了非常灵活的封装机制，在一层层封装之后，就容易迷路，找不到真正实现的地方。

在理解分配器之前，先了解一下 Gralloc，对于理解后面的代码有好处。

### Gralloc

Gralloc 这个名字是两个词的缩写组合：

* Gr 来自 Graphics (图形)
* alloc 来自 Allocator (分配器)

所以，Gralloc 的全称就是 Graphics Allocator，即图形分配器。Gralloc 在 Android 系统中代表一个非常具体和关键的角色：它是一个硬件抽象层（HAL）模块，专门负责管理和分配用于图形渲染的内存缓冲区（Graphic Buffers）。

Android 的上层应用框架（如 Skia 图形库、视频解码器、相机等）需要内存来存放图像数据，但它不知道底层硬件（GPU、显示控制器）到底需要什么样的内存。例如，某些 GPU 可能要求内存是物理上连续的，或者有特殊的对齐方式，或者位于特定的内存区域（如 VRAM）。

Gralloc 模块由硬件/芯片制造商（如高通、三星、联发科）根据他们自己的硬件特性来实现。它提供了一套标准的 API 给 Android 系统调用。当 Android 系统说“我需要一块 1920x1080 大小、RGBA 格式的缓冲区用于 GPU 渲染”时，Gralloc 模块会把这个高级请求翻译成底层硬件驱动能听懂的具体操作，然后从最合适的内存区域分配一块符合所有硬件要求的内存。

在 Android 8.0 (Oreo) 引入 Project Treble 架构之前，Gralloc 的实现是和 Android 系统框架紧密耦合的。Project Treble 之后，Gralloc 成为了一个标准化的、独立的 HAL 模块。目前有 2、3、4 几个版本。

在 GraphicBufferAllocator 的构造函数，可以看到如下定义：

```
    mAllocator = std::make_unique<const Gralloc4Allocator>(
            reinterpret_cast<const Gralloc4Mapper&>(mMapper.getGrallocMapper()));
    if (mAllocator->isLoaded()) {
        return;
    }
    mAllocator = std::make_unique<const Gralloc3Allocator>(
            reinterpret_cast<const Gralloc3Mapper&>(mMapper.getGrallocMapper()));
    if (mAllocator->isLoaded()) {
        return;
    }
    mAllocator = std::make_unique<const Gralloc2Allocator>(
            reinterpret_cast<const Gralloc2Mapper&>(mMapper.getGrallocMapper()));
    if (mAllocator->isLoaded()) {
        return;
    }

    LOG_ALWAYS_FATAL("gralloc-allocator is missing");
```

也就是说，Android 系统会尽力选择版本最高的那个 Gralloc 分配器。通过加入 log 日志，我们可以确定在 waydroid 中使用的是 Gralloc2Allocator。

接下来看 Gralloc2.cpp 这个文件，可以看到 Gralloc2Allocator::allocate 的定义，

```
    auto ret = mAllocator->allocate(descriptor, bufferCount,
            ...
 
```
又是套娃，找到 mAllocator 的定义：

```
    mAllocator = IAllocator::getService();
    if (mAllocator == nullptr) {
        ALOGW("allocator 2.x is not supported");
        return;
    }
```

这种范式在 Android 系统的代码中非常常见。服务实现和使用分离，虽然非常灵活，但对于阅读代码的人就非常痛苦了，去哪找 IAllocator 的实现呢？

### HAL 清单

在 device/waydroid/waydroid/ 目录下，我发现了一个 manifest.xml 文件，正是这个 xml 文件，揭开了 IAllocator 实现的奥秘。

在 这个 xml 文件中，我看到了如下声明：

```
    <hal format="hidl">
        <name>android.hardware.graphics.allocator</name>
        <transport>hwbinder</transport>
        <version>2.0</version>
        <interface>
            <name>IAllocator</name>
            <instance>default</instance>
        </interface>
    </hal>
```

也就是说，服务由一个实现了这个清单中声明的接口的库提供，并且系统会根据这个清单来加载它。我们可以看到，IAllocator 服务是由一个名为 default 的库提供。当然，这里的 default 也可能只是表示默认的意思，并不是一个真正的名称。为此，我去 AOSP 的构建 out 目录查找所有与 gralloc 有关的 so，如下所示：

```
./out/target/product/waydroid_arm64/system/lib64/libgralloctypes.so
./out/target/product/waydroid_arm64/system/apex/com.android.vndk.current/lib64/libgralloctypes.so
./out/target/product/waydroid_arm64/system/apex/com.android.vndk.current/lib/libgralloctypes.so
./out/target/product/waydroid_arm64/system/apex/com.android.media.swcodec/lib64/libgralloctypes.so
./out/target/product/waydroid_arm64/system/lib/libgralloctypes.so
./out/target/product/waydroid_arm64/vendor/lib64/libminigbm_gralloc_gbm_mesa.so
./out/target/product/waydroid_arm64/vendor/lib64/hw/gralloc.default.so
./out/target/product/waydroid_arm64/vendor/lib64/hw/gralloc.gbm.so
./out/target/product/waydroid_arm64/vendor/lib64/hw/gralloc.minigbm_gbm_mesa.so
./out/target/product/waydroid_arm64/vendor/lib/libminigbm_gralloc_gbm_mesa.so
./out/target/product/waydroid_arm64/vendor/lib/hw/gralloc.default.so
./out/target/product/waydroid_arm64/vendor/lib/hw/gralloc.gbm.so
./out/target/product/waydroid_arm64/vendor/lib/hw/gralloc.minigbm_gbm_mesa.so
```

如何确定到底系统加载的是哪个 so 呢？

最简单的方法是查看某个关键进程（比如 surfaceflinger）已经加载了哪些库。

```
# 进入 Android 系统内部
sudo waydroid shell

# 查找 surfaceflinger 的 PID，从结果可以看出是 102
:/ # ps -A | grep surfaceflinger      
system          102      1 2814640 146668 ep_poll             0 S surfaceflinger

:/ # cat /proc/102/maps | grep .so | grep gralloc      
7f1380c000-7f1380e000 r--p 00000000 07:01 480                            /vendor/lib64/hw/gralloc.default.so
7f1380e000-7f13810000 r-xp 00002000 07:01 480                            /vendor/lib64/hw/gralloc.default.so
7f13810000-7f13811000 r--p 00004000 07:01 480                            /vendor/lib64/hw/gralloc.default.so
7f13811000-7f13812000 rw-p 00004000 07:01 480                            /vendor/lib64/hw/gralloc.default.so
7f9caca000-7f9cad2000 r--p 00000000 07:00 3022                           /system/lib64/libgralloctypes.so
7f9cad2000-7f9cadb000 r-xp 00008000 07:00 3022                           /system/lib64/libgralloctypes.so
7f9cadb000-7f9cadc000 r--p 00011000 07:00 3022                           /system/lib64/libgralloctypes.so
7f9cadc000-7f9cadd000 rw-p 00011000 07:00 3022                           /system/lib64/libgralloctypes.so

```

可以看出，系统加载的就是 gralloc.default.so 这个so。那么，接下来就需要查找 gralloc.default.so 是怎么编译出来的。

### Gralloc HAL 实现

一般 Android 模块的构建都是定义在 Android.bp 文件中，但是有些比较老的模块，可能依然采用 Android.mk，所以我在 AOSP 源码中的 Android.bp 和 Android.mk 文件中搜索 gralloc.default。最后在 hardware/libhardware/modules/gralloc/Android.mk 文件中找到了如下定义：

```
LOCAL_PATH := $(call my-dir)

# HAL module implemenation stored in
# hw/<OVERLAY_HARDWARE_MODULE_ID>.<ro.product.board>.so
include $(CLEAR_VARS)

LOCAL_MODULE_RELATIVE_PATH := hw
LOCAL_PROPRIETARY_MODULE := true
LOCAL_SHARED_LIBRARIES := liblog libcutils

LOCAL_SRC_FILES := 	\
	gralloc.cpp 	\
	framebuffer.cpp \
	mapper.cpp

LOCAL_HEADER_LIBRARIES := libhardware_headers

LOCAL_MODULE := gralloc.default
LOCAL_CFLAGS:= -DLOG_TAG=\"gralloc\" -Wno-missing-field-initializers
ifeq ($(TARGET_USE_PAN_DISPLAY),true)
LOCAL_CFLAGS += -DUSE_PAN_DISPLAY=1
endif

include $(BUILD_SHARED_LIBRARY)
```

所以最终的实现代码就位于 hardware/libhardware/modules/gralloc 目录，在 gralloc.cpp 的 gralloc_alloc 函数中加入日志，可以确认这一点。

其实我们看 gralloc_device_open 这段代码：

```
int gralloc_device_open(const hw_module_t* module, const char* name,
        hw_device_t** device)
{
    int status = -EINVAL;
    if (1) {
        gralloc_context_t *dev;
        dev = (gralloc_context_t*)malloc(sizeof(*dev));

        /* initialize our state here */
        memset(dev, 0, sizeof(*dev));

        /* initialize the procs */
        dev->device.common.tag = HARDWARE_DEVICE_TAG;
        dev->device.common.version = 0;
        dev->device.common.module = const_cast<hw_module_t*>(module);
        dev->device.common.close = gralloc_close;

        dev->device.alloc   = gralloc_alloc;
        dev->device.free    = gralloc_free;

        *device = &dev->device.common;
        status = 0;
    } else {
        status = fb_device_open(module, name, device);
    }
    return status;
}
```

可以看到 gralloc_alloc 给赋值到 dev->device.alloc 方法，最终的调用链条会抵达 gralloc_alloc 方法。我们再来仔细看一下 gralloc_alloc 方法的实现：

```
static int gralloc_alloc(alloc_device_t* dev,
        int width, int height, int format, int usage,
        buffer_handle_t* pHandle, int* pStride)
{
    if (!pHandle || !pStride)
        return -EINVAL;

    int bytesPerPixel = 0;
    switch (format) {
        case HAL_PIXEL_FORMAT_RGBA_FP16:
            bytesPerPixel = 8;
            break;
        case HAL_PIXEL_FORMAT_RGBA_8888:
        case HAL_PIXEL_FORMAT_RGBX_8888:
        case HAL_PIXEL_FORMAT_BGRA_8888:
            bytesPerPixel = 4;
            break;
        case HAL_PIXEL_FORMAT_RGB_888:
            bytesPerPixel = 3;
            break;
        case HAL_PIXEL_FORMAT_RGB_565:
        case HAL_PIXEL_FORMAT_RAW16:
            bytesPerPixel = 2;
            break;
        default:
            return -EINVAL;
    }

    ...
}
```

在入口处加入log，发现视频播放黑屏时，传入的是 format 值是 842094169（0x32315659）。开始看到这么奇怪的值，第一反应是传入的值传错了，好在我去看了一下 HAL_PIXEL_FORMAT_RGBA_8888 等值的定义，它们定义在 system/core/include/system/graphics-base-v1.0.h 文件中：

```
typedef enum {
    HAL_PIXEL_FORMAT_RGBA_8888 = 1,
    HAL_PIXEL_FORMAT_RGBX_8888 = 2,
    HAL_PIXEL_FORMAT_RGB_888 = 3,
    HAL_PIXEL_FORMAT_RGB_565 = 4,
    HAL_PIXEL_FORMAT_BGRA_8888 = 5,
    HAL_PIXEL_FORMAT_YCBCR_422_SP = 16,
    HAL_PIXEL_FORMAT_YCRCB_420_SP = 17,
    HAL_PIXEL_FORMAT_YCBCR_422_I = 20,
    HAL_PIXEL_FORMAT_RGBA_FP16 = 22,
    HAL_PIXEL_FORMAT_RAW16 = 32,
    HAL_PIXEL_FORMAT_BLOB = 33,
    HAL_PIXEL_FORMAT_IMPLEMENTATION_DEFINED = 34,
    HAL_PIXEL_FORMAT_YCBCR_420_888 = 35,
    HAL_PIXEL_FORMAT_RAW_OPAQUE = 36,
    HAL_PIXEL_FORMAT_RAW10 = 37,
    HAL_PIXEL_FORMAT_RAW12 = 38,
    HAL_PIXEL_FORMAT_RGBA_1010102 = 43,
    HAL_PIXEL_FORMAT_Y8 = 538982489,
    HAL_PIXEL_FORMAT_Y16 = 540422489,
    HAL_PIXEL_FORMAT_YV12 = 842094169,
} android_pixel_format_t;
```

最后一个枚举值 HAL_PIXEL_FORMAT_YV12 就对应着上面那个奇怪的 format 值。谜底揭开了，gralloc_alloc 只处理了常见的 RGB 和 RAW 格式，完全没有处理任何 YUV 格式。

我也是第一次碰到 YV12 这种格式，所以下面补充一下相关知识。

### YV12

绝大多数我们在网上看到的视频（MP4, MKV, WebM, AVI等封装格式）其内部的视频流都是使用 YUV 4:2:0 的色彩空间进行编码的。

* YUV 4:2:0 是什么？
 - 这是一种色彩二次采样（Chroma Subsampling）技术。人眼对亮度（Luma, Y）的变化比对色度（Chroma, U 和 V）的变化更敏感。
 - 为了节省带宽和存储空间，编码器会完整地保存每个像素的亮度信息（Y），但每 2x2 的像素块只保存一组色度信息（U 和 V）。
 - 这样，平均每个像素只需要 8 (Y) + 8/4 (U) + 8/4 (V) = 12 bits，相比于完整的 RGB（24 bits）节省了一半的空间。
* YV12 和 YUV 4:2:0 的关系
 - YV12 是 YUV 4:2:0 色彩空间的一种具体内存布局（Planar Format）。
 - "Planar"（平面）意味着 Y、U、V 三个分量是分开存放的。YV12 的内存布局是：先是所有的 Y 分量数据，然后是所有的 V 分量数据，最后是所有的 U 分量数据。
 - 另一种常见的布局是 I420，它的顺序是 Y-U-V。
 - 还有一种半平面（Semi-Planar）布局如 NV12 (Y-UV交错) 和 NV21 (Y-VU交错)。

如果你的视频源文件本身是用 YUV 4:2:0 编码的（这几乎是所有网络视频的标准），那么解码器最自然、最高效的输出就是某种 YUV 4:2:0 的布局，比如 YV12。

在了解到 YV12 格式之后，接下来就是针对这种格式编写内存分配代码，是不是补充上内存分配代码之后，就万事大吉呢？

答案在下一篇文章中揭晓，敬请关注。

码字不易，觉得不错就点个在看吧。