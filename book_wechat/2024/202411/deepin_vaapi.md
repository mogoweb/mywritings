# Linux 系统下的硬件视频加速

在做浏览器研发的时候，最头疼与 GPU 硬件加速相关的问题，而 GPU 硬件加速问题最多的又是与视频播放相关。以前做基于 Android 系统的开发浏览器，写过一系列的与浏览器中视频播放有关的文章。

* [Chromium HTML Video 媒体播放代码梳理](https://mp.weixin.qq.com/s/LQv-I9wAjPXwOKEyZ-69wg)
* [选择最新 Chromium，支持 H264 / H265](https://mp.weixin.qq.com/s/IpioXG-_NaGOc9nnKe6xbQ)
* [Chromium 改造实录：增加 MPEG TS 格式支持](https://mp.weixin.qq.com/s/enrzjVLy_VACqTKavuam7Q)
* [Chromium 改造实录：增加 MP2 音频支持](https://mp.weixin.qq.com/s/ZJjG_JYS51WM-WiY8XejVA)
* [Chromium 改造实录：8K 来了](https://mp.weixin.qq.com/s/iKWBMSjtrM6NAz8s_kV6ag)
* [Chromium 改造实录：国标AVS2 & AVS3 支持起来](https://mp.weixin.qq.com/s/mTCs8Q4PtUUe2M5IDP1Rwg)

在转向国产信创系统软件开发后，发现更多的时间花在解决浏览器中视频播放的硬件加速问题上。国产信创操作系统都是基于 Linux 系统开发。由于信创系统采用的国产 CPU，性能一般较弱，一旦不能硬解，视频播放就会有卡顿等不好的体验，所以启用硬件加速通常是非常有必要的。而 Linux 系统是一个开放系统，这里面涉及硬件、驱动、操作系统、应用软件等，只要其中一个环节出了问题，就会导致硬件加速启用不了。

这篇文章就来梳理一下 Linux 下的硬件视频加速，文中使用的系统为 UOS V20，CPU 为兆芯 KX-6640MA， GPU 为兆芯 C-960。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202411/images/deepin_vaapi_01.png)

## 概述

使用现代显卡，通常可以将视频编码和解码任务从 CPU 转移给 GPU。与 CPU 相比，GPU 的效率更高。但是，这种转移需要硬件和软件支持。由于 Linux 系统的开放性，在Linux 世界中对硬件视频加速的支持并没有形成一个统一标准。

目前，Linux 上主流的的三个视频加速 API 是 VA-API、VDPAU 和 NVENC/NVDEC。

* VA-API - 在 Intel、AMD 和 NVIDIA 上受支持（仅通过开源 Nouveau 驱动程序）。受到软件的广泛支持，包括 Kodi、VLC、MPV、Chromium 和 Firefox。
* VDPAU - 在 AMD 和 NVIDIA 上完全受支持（专有和 Nouveau）。大多数桌面应用程序（如 Kodi、VLC 和 MPV）均受支持，但在 Chromium 或 Firefox 中完全不受支持。主要限制是对 Intel 的支持不佳且不完整，并且无法与浏览器配合使用以进行网络视频加速。
* NVENC/NVDEC - NVIDIA 独家支持的专有 API。仅在少数主要应用程序中受支持（用于编码的 FFmpeg 和 OBS Studio，用于解码的 FFmpeg 和 MPV）。

在某些 ARM 架构的系统上，比如华为麒麟芯片，也可能通过专用的 API（如 OMX）实现硬件加速。

由于 VA-API 不论在支持程度，还是兼容性上，都比其它几种 API 更加好，VA-API 得到广泛的软件支持，这里先重点介绍 VA API。

## VA-API

VA-API（Video Acceleration API）是一个开放的跨平台接口，专为支持硬件加速的视频编解码和处理任务而设计。它由 Intel 开发并维护，最初用于 Intel 集成显卡，现已被扩展到支持多种 GPU 平台（如 AMD 和某些 ARM 硬件）。VA-API 提供了用户空间和内核之间的桥梁，使应用程序能够充分利用硬件的视频解码、编码和后处理功能。

需要注意的是， VA-API 只是一套接口标准，包含以下核心组件：

* 用户空间库
  提供了应用程序调用 VA-API 的接口（如 `libva`）。应用程序通过此库发送硬件加速的请求。
* 驱动程序
  不同硬件需要相应的用户空间驱动程序（如 `intel-media-driver`、`mesa` 的 AMD 驱动）。
* 内核接口
  VA-API 的大部分操作依赖于 Linux 内核中的 DRM（Direct Rendering Manager），后者提供了 GPU 资源管理和任务调度。

VA-API（Video Acceleration API）理论上在各种平台上都可以得到支持，比如 [Video acceleration API (VA-API) now available on Windows!](https://devblogs.microsoft.com/directx/video-acceleration-api-va-api-now-available-on-windows/) 这篇文章就介绍了 Windows 下 VA-API 的支持情况。华为麒麟芯片也实现了 VA-API 驱动和 libva。甚至还可以在其它视频加速 API 的基础上实现 VA-API，比如 nvidia-vaapi-driver 就是以 NVDEC 为后端封装 VA-API，使得使用 Nvidia 的显卡也可以使用 VA-API。

对于应用开发者来说，一般只需要关心 libva。

## libva

**Libva**（全称为 Video Acceleration (VA) API library）是实现 VA-API 的用户空间库，它充当应用程序和硬件驱动程序之间的桥梁。Libva 的核心职责是为开发者提供统一的 API，隐藏底层硬件实现的复杂性，使应用程序能够无缝地调用硬件加速功能。

Libva 的架构分为以下几个层次：

1. **应用程序层**
   应用程序（如媒体播放器、浏览器）通过 VA-API 调用 Libva 提供的视频处理功能。
2. **Libva 核心库**
   * 提供符合 VA-API 标准的接口（如 `vaInitialize`、`vaCreateContext`）。
   * 负责加载和管理硬件特定的驱动程序。
3. **硬件驱动程序（Backend）**
   Libva 加载硬件驱动（通常为动态链接库，如 `/usr/lib/dri/iHD_drv_video.so`），驱动程序负责与 GPU 通信并执行硬件加速任务。

一般来说，我们可以通过 libva 提供的实用程序 vainfo 来检验系统是否正确正常支持 VA-API。
