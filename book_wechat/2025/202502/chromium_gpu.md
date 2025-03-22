# 浏览器的 GPU 兼容处理

最近 DeepSeek 让 AI 彻底出圈了，同时也让大家认识到了GPU（显卡）的重要性。以前，我们只是觉得显卡就是用来玩游戏的。其实除了 AI 和玩游戏，显卡还有很多重要作用，比如音视频编解码、图形渲染、3D建模、科学计算等等。甚至在浏览器中，GPU 也发挥着重要作用，比如渲染网页，有 GPU 加速不仅能降低 CPU 占用率，还能提升渲染速度，从而提升用户体验。

但是，浏览器碰到的渲染问题，大多是 GPU 兼容性问题导致的，特别是 Linux 下。在我们维护产品的过程中，碰到最多的问题就是渲染问题，比如花屏、黑屏、白屏、字体闪烁、卡顿等等。这些问题，一般来说禁用硬件加速大部分能解决，但很多时候禁用硬件加速又会影响用户体验，比如卡顿。有时候，浏览器一启动就碰到花屏、白屏、黑屏，都没有机会进入设置界面去禁用硬件加速。

面对 GPU 这些棘手的问题，浏览器开发者也会采取一些措施，这里就来分析 Chromium 浏览器是如何处理 GPU 兼容性的。本篇文章以 deepin v23 系统下的系统浏览器为例，同样适用于其它 Linux 发行版下的基于 Chromium 的浏览器。

## GPU 黑名单

通常，如果用户碰到 GPU 渲染问题，我们通常会让他们在浏览器的启动参数中加入 --disable-gpu 参数，看能否解决问题。这里 --disable-gpu 参数就是禁用 GPU 硬件加速。虽然这样大部分情况下能解决问题，但并不是长久之计，因为禁用 GPU 硬件加速会影响用户体验。

Chromium 为了提升渲染性能，充分利用了 GPU 硬件加速，如 DOM 绘制、Canvas2D、WebGL、视频解码等，并设计了独立的 GPU 进程架构。其实说到 GPU 兼容性问题，有可能是某个接口实现不如预期，或者缺少某个接口，并不是所有的硬件加速都无法使用。

为了更细粒度的解决 GPU 兼容问题，Chromium 将使用 GPU 硬件加速的功能模块分为一个个 feature，包括常见的如 Canvas2D、WebGL、视频解码等，这样即使某个 feature 无法启用，只需要禁用这个 feature 就可以了，而不是禁用整个 GPU 硬件加速。这样能够更灵活的解决问题。

Chromium 中 GPU feature 定义见源码：/gpu/config/gpu_feature_type.h。目前有以下值：

  * Canvas 2D：GPU_FEATURE_TYPE_ACCELERATED_2D_CANVAS
  * WebGL 1.0：GPU_FEATURE_TYPE_ACCELERATED_WEBGL
  * 视频硬件解码: GPU_FEATURE_TYPE_ACCELERATED_VIDEO_DECODE
  * 视频硬件编码: GPU_FEATURE_TYPE_ACCELERATED_VIDEO_ENCODE
  * 图块光栅化：GPU_FEATURE_TYPE_GPU_TILE_RASTERIZATION
  * WebGL 2.0：GPU_FEATURE_TYPE_ACCELERATED_WEBGL2
  * Android 平台下的 SurfaceControl: GPU_FEATURE_TYPE_ANDROID_SURFACE_CONTROL
  * 使用 OpenGL 硬件加速：GPU_FEATURE_TYPE_ACCELERATED_GL
  * 使用 Vulkan 硬件加速：GPU_FEATURE_TYPE_VULKAN
  * GPU_FEATURE_TYPE_CANVAS_OOP_RASTERIZATION
  * WebGPU: GPU_FEATURE_TYPE_ACCELERATED_WEBGPU
  * 使用 Skia 图形库的 Graphite 后端: GPU_FEATURE_TYPE_SKIA_GRAPHITE
  * Web 网络神经：GPU_FEATURE_TYPE_WEBNN

Chromium 内部维护着一个黑名单，如果测试或使用中发现某个 GPU 产品的某个 feature 开启后会引起渲染问题，就会加入到黑名单中，这样浏览器在检测到该 GPU 产品后会禁用该 feature。黑名单定义在一个 json 文件中，位于 gpu/config/software_rendering_list.json。关于这个定义文件的格式，参考源码中的一个文本文件：gpu/config/gpu_control_list_format.txt。这里就不详细介绍，给大家展示几个示例，应该就明白大致的写法：

```json
{
  "name": "software rendering list",
  "entries": [
    {
      "id": 3,
      "description": "GL driver is software rendered. GPU acceleration is disabled",
      "cr_bugs": [59302, 315217, 1155974],
      "os": {
        "type": "linux"
      },
      "gl_renderer": "(?i).*(software|llvmpipe|softpipe).*",
      "features": [
        "all"
      ]
    },
    {
      "id": 4,
      "description": "The Intel Mobile 945 Express family of chipsets is not compatible with WebGL",
      "cr_bugs": [232035],
      "vendor_id": "0x8086",
      "device_id": ["0x27AE", "0x27A2"],
      "features": [
        "accelerated_webgl",
        "accelerated_2d_canvas"
      ]
    },
    {
      "id": 8,
      "description": "NVIDIA GeForce FX Go5200 is assumed to be buggy",
      "cr_bugs": [72938],
      "vendor_id": "0x10de",
      "device_id": ["0x0324"],
      "features": [
        "all"
      ]
    }
  ]
}
```
其中 vendor_id 和 device_id 对应的就是硬件的 vid 和 pid，每个型号的硬件产品都有唯一的 vid 和 pid。features 就是对应上面的 feature 定义，all 表示所有 feature。更高级的还可以根据操作系统类型、系统版本、GPU型号、驱动版本等多个维度的规则进行匹配。

当前这个文件有 1500 多行，160 多项，可以预见随着 GPU 产品越来越多，这个名单也会越来越长。

注意，这个文件是没法在使用中进行修改的。因为在编译的时候，构建系统使用 Python 脚本将 json 规则文件转换为 C++ 文件(software_rendering_list_autogen.h等)，其中包含 kSoftwareRenderingListEntries 数组常量，即为反序列化后的规则列表，然后在 GpuControlList 类中进行处理，将当前设备 GPU 信息进行规则匹配，从而确定各个 feature 是否需要禁用。

有人可能会问了，如果某个 GPU 型号无法使用某个 feature，只能等待 Chromium 开发者来更新这个黑名单吗？目前看来是的。可能谷歌考虑到普通用户也维护不了这张表，如果直接使用 json 文件允许用户修改，一来可能会影响性能，二来如果配置错误，可能会导致更严重的问题。

## GPU workaround

workaround 对于开发者来说不会陌生，有时在开发中碰到无法解决的问题，会通过一些手段绕过去。

前面的特性黑名单是将 feature 禁用，降级为软件渲染，而 GPU workaround 则是针对特定的驱动 bug 绕过去(workaround)。比如在某些设备上调用 glClear 不生效或闪退，需要通过 draw 来模拟 clear。

和前面的 GPU 黑名单类似，workaround 也维护在一个 json 文件( /gpu/config/gpu_driver_bug_list.json )中。每种 workaround 都有一个 id，如 "gl_clear_broken"、"max_texture_size_limit_4096" 等，id 列表详见 gpu/config/gpu_workaround_list.txt ，目前总计有 120 多项。

下面给一个简单的示例：

```json
    {
      "id": 1,
      "description": "Imagination driver doesn't like uploading lots of buffer data constantly",
      "cr_bugs": [178093],
      "os": {
        "type": "android"
      },
      "gl_vendor": "Imagination.*",
      "gl_type": "gles",
      "gl_version": {
        "op": "<",
        "value": "3.0"
      },
      "features": [
        "use_client_side_arrays_for_stream_buffers"
      ]
    }
```
目前该规则列表总计有 300 多项，且在持续更新中。

规则文件 gpu_driver_bug_list.json 也是通过 Python 脚本生成 C++ 源码文件，将规则列表反序列化到 kGpuDriverBugListEntries 数组常量中，然后提供给底层渲染逻辑进行兼容处理，比如 "gl_clear_broken" 这个 workaround，最终会影响到 skia 的绘制逻辑( /third_party/skia/src/gpu/gl/GrGLCaps.cpp )

```cpp
if (fDriverBugWorkarounds.gl_clear_broken) {
    fPerformColorClearsAsDraws = true;
    fPerformStencilClearsAsDraws = true;
}
```

## GPU 状态&开关

基于 chromium 的浏览器都可通过访问 chrome://gpu 查看详细的GPU信息，包括 Graphics Feature Status、Driver Bug Workarounds、以及各类状态信息等，如：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/chromium_gpu_01.png)

这张表很长，包含的信心量很大，是我们解决浏览器 GPU 渲染问题的重要参考。

前面的 feature 除了通过黑名单禁用，还可以通过访问 chrome://flags/ 手动配置一些GPU相关的开关，如：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/chromium_gpu_02.png)

当然，我们还可以通过命令行参数控制，比如 --ignore-gpu-blacklist 就可以忽略 GPU 黑名单（比如某些产品被误加入了黑名单，强制开启硬件加速），--enable-oop-rasterization 或 --disable-oop-rasterization 可用来启用或禁用进程外光栅化。

## ANGLE 后端

在维护产品是发现，对于某些 GPU 型号（比如 AMD 的 Randon 系列显卡），使用 --use-angle=gl 命令行参数可以解决花屏问题。这个命令行是作用是什么？

原来，Chromium 是一个跨平台的浏览器，必须在 Windows、Android、Linux、macOS 等多个操作系统上运行。为了跨平台，Google 开发了开源图形转换层ANGLE（Almost Native Graphics Layer Engine）。ANGLE 的主要功能是将 OpenGL ES 的 API 调用转换为其他平台上更稳定、性能更优的图形接口调用，例如在 Windows 平台上将 OpenGL ES 调用转换为 DirectX 调用，在 Android 上直接使用 OpenGL ES，在 macOS 上则可能通过 Metal。

使用 ANGLE，开发者可以统一采用 OpenGL ES 作为开发接口，而 ANGLE 根据不同平台自动选择最佳的后端，从而大大简化了跨平台图形渲染的开发和维护工作。

在 Linux 下，默认情况下 Chromium 会直接调用系统提供的原生 OpenGL（通常是基于 Mesa 的实现）。而如果你使用了 --use-angle=gl 参数，则 Chromium 会启用 ANGLE，并用其 OpenGL 后端来处理 OpenGL ES 调用，即将所有的图形 API 调用先经过 ANGLE 的转换层，再调用底层的 OpenGL。

在某些情况下，系统原生 OpenGL 驱动可能存在兼容性或稳定性问题，使用 ANGLE 可能会绕过这些问题，从而提供更加一致的行为。虽然增加了一个中间转换层，使用 ANGLE 可能会引入一定的性能开销，不过这种开销通常是微乎其微的。如果系统的原生驱动存在问题，使用 ANGLE 可能反而会获得更好的整体表现。

如果在 Linux 下碰到 gpu 硬件加速问题，不放尝试加上 --use-angle=gl 命令行参数，看看是否有效。

## 小结

本文总结了一下 Chromium 中对于 GPU 的一些兼容处理，包括黑名单、workaround、状态开关等，希望能对大家解决浏览器渲染问题有所帮助。浏览器的 GPU 加速问题当然不止这些，需要深入研究的地方还有很多，比如 GPU 内存管理、GPU 进程等，后续有机会再和大家分享。

如果大家有什么经验，也欢迎留言交流。