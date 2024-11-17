# Linux 系统下的硬件视频加速

在浏览器研发中，GPU 硬件加速相关的问题常常令人头疼，而这些问题中，视频播放更是棘手。回顾以往，在基于 Android 系统开发浏览器时，我曾撰写了一系列与浏览器视频播放相关的技术文章：

* [Chromium HTML Video 媒体播放代码梳理](https://mp.weixin.qq.com/s/LQv-I9wAjPXwOKEyZ-69wg)
* [选择最新 Chromium，支持 H264 / H265](https://mp.weixin.qq.com/s/IpioXG-_NaGOc9nnKe6xbQ)
* [Chromium 改造实录：增加 MPEG TS 格式支持](https://mp.weixin.qq.com/s/enrzjVLy_VACqTKavuam7Q)
* [Chromium 改造实录：增加 MP2 音频支持](https://mp.weixin.qq.com/s/ZJjG_JYS51WM-WiY8XejVA)
* [Chromium 改造实录：8K 来了](https://mp.weixin.qq.com/s/iKWBMSjtrM6NAz8s_kV6ag)
* [Chromium 改造实录：国标AVS2 & AVS3 支持起来](https://mp.weixin.qq.com/s/mTCs8Q4PtUUe2M5IDP1Rwg)

转向国产信创系统的开发后，噩梦继续，浏览器中的视频播放的硬件加速问题继续困扰着我。国产信创操作系统通常基于 Linux 内核，而许多国产 CPU 的性能较为有限。一旦无法启用硬解，视频播放的流畅度便难以保证，硬件加速的启用变得至关重要。然而，Linux 系统的开放性使得硬件、驱动、操作系统及应用软件之间的协调成为一大挑战，任何一个环节的问题都可能导致硬件加速失效。

本文将梳理 Linux 系统下硬件视频加速的原理与实现，以 UOS V20 系统为例，测试环境为兆芯 KX-6640MA CPU 和兆芯 C-960 GPU。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202411/images/deepin_vaapi_01.png)

## 概述

使用现代显卡，通常可以将视频编码和解码任务从 CPU 转移给 GPU。与 CPU 相比，GPU 的效率更高。但是，这种转移需要硬件和软件支持。Linux 系统的开放性使得硬件视频加速缺乏统一的标准，目前主流的加速 API 有以下三种：

* VA-API - 在 Intel、AMD 和 NVIDIA 上受支持（仅通过开源 Nouveau 驱动程序）。受到软件的广泛支持，包括 Kodi、VLC、MPV、Chromium 和 Firefox。
* VDPAU - 在 AMD 和 NVIDIA 上完全受支持（专有和 Nouveau）。大多数桌面应用程序（如 Kodi、VLC 和 MPV）均受支持，但在 Chromium 或 Firefox 中完全不受支持。主要限制是对 Intel 的支持不佳且不完整，并且无法与浏览器配合使用以进行网络视频加速。
* NVENC/NVDEC - NVIDIA 独家支持的专有 API。仅在少数主要应用程序中受支持（用于编码的 FFmpeg 和 OBS Studio，用于解码的 FFmpeg 和 MPV）。

在某些 ARM 架构的系统上，比如华为麒麟芯片，也可能通过专用的 API（如 OMX）实现硬件加速。

由于 VA-API 不论在支持程度，还是兼容性上，都比其它几种 API 更加好，VA-API 得到广泛的软件支持，因此本文重点介绍 VA API。

## VA-API

VA-API（Video Acceleration API）是一个开放的跨平台接口，专为支持硬件加速的视频编解码和处理任务而设计。它由 Intel 开发并维护，最初用于 Intel 集成显卡，现已被扩展到支持多种 GPU 平台（如 AMD 和某些 ARM 硬件）。VA-API 提供了用户空间和内核之间的桥梁，使应用程序能够充分利用硬件的视频解码、编码和后处理功能。

需要注意的是， VA-API 只是一套接口标准，包含以下核心组件：

* 用户空间库
  提供了应用程序调用 VA-API 的接口（如 `libva`）。应用程序通过此库发送硬件加速的请求。
* 驱动程序
  不同硬件需要相应的用户空间驱动程序（如 `intel-media-driver`、`mesa` 的 AMD 驱动）。
* 内核接口
  VA-API 的大部分操作依赖于 Linux 内核中的 DRM（Direct Rendering Manager），后者提供了 GPU 资源管理和任务调度。

VA-API（Video Acceleration API）理论上在各种平台上都可以得到支持，比如 [Video acceleration API (VA-API) now available on Windows!](https://devblogs.microsoft.com/directx/video-acceleration-api-va-api-now-available-on-windows/) 这篇文章就介绍了 Windows 下 VA-API 的支持情况。华为麒麟芯片也实现了 VA-API 驱动和 libva。甚至还可以在其它视频加速 API 的基础上实现 VA-API，比如 nvidia-vaapi-driver 就是以 NVDEC 为后端封装 VA-API，使得使用 Nvidia 显卡的系统也可以使用 VA-API。

对于应用开发者来说，一般只需要关心 libva。

## libva

**Libva**（全称为 Video Acceleration (VA) API library）是实现 VA-API 的用户空间库，它充当应用程序和硬件驱动程序之间的桥梁。Libva 的核心职责是为开发者提供统一的 API，隐藏底层硬件实现的复杂性，使应用程序能够无缝地调用硬件加速功能。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202411/images/deepin_vaapi_02.png)

Libva 的架构分为以下几个层次：

1. 应用程序层
   应用程序（如媒体播放器、浏览器）通过 VA-API 调用 Libva 提供的视频处理功能。
2. Libva 核心库
   * 提供符合 VA-API 标准的接口（如 `vaInitialize`、`vaCreateContext`）。
   * 负责加载和管理硬件特定的驱动程序。
3. 硬件驱动程序（Backend）
   Libva 加载硬件驱动（通常为动态链接库，如 `/usr/lib/dri/iHD_drv_video.so`），驱动程序负责与 GPU 通信并执行硬件加速任务。

一般来说，我们可以通过 libva 提供的实用程序 vainfo 来检验系统是否正确正常支持 VA-API。

```
alex@alex-UOS-ZXPC:~$ vainfo
libva info: VA-API version 1.14.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/zx_drv_video.so
libva info: Found init function __vaDriverInit_1_0
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.14 (libva 2.4.0)
vainfo: Driver version: ZX_VA
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointVLD
      VAProfileMPEG4Simple            : VAEntrypointVLD
      VAProfileMPEG4AdvancedSimple    : VAEntrypointVLD
      <unknown profile>               : VAEntrypointVLD
      <unknown profile>               : VAEntrypointEncSlice
      VAProfileH264Main               : VAEntrypointVLD
      VAProfileH264Main               : VAEntrypointEncSlice
      VAProfileH264High               : VAEntrypointVLD
      VAProfileH264High               : VAEntrypointEncSlice
      VAProfileH264ConstrainedBaseline: VAEntrypointVLD
      VAProfileH264ConstrainedBaseline: VAEntrypointEncSlice
      VAProfileVC1Simple              : VAEntrypointVLD
      VAProfileVC1Main                : VAEntrypointVLD
      VAProfileVC1Advanced            : VAEntrypointVLD
      VAProfileH263Baseline           : VAEntrypointVLD
      VAProfileJPEGBaseline           : VAEntrypointVLD
      VAProfileJPEGBaseline           : VAEntrypointEncPicture
      VAProfileNone                   : VAEntrypointVideoProc
      VAProfileHEVCMain               : VAEntrypointVLD
      VAProfileHEVCMain               : VAEntrypointEncSlice
      VAProfileHEVCMain10             : VAEntrypointVLD
      VAProfileHEVCMain10             : VAEntrypointEncSlice
      VAProfileH264MultiviewHigh      : VAEntrypointVLD
      VAProfileH264MultiviewHigh      : VAEntrypointEncSlice
      VAProfileH264StereoHigh         : VAEntrypointVLD
      VAProfileH264StereoHigh         : VAEntrypointEncSlice
      VAProfileVP8Version0_3          : VAEntrypointVLD
      <unknown profile>               : VAEntrypointVLD
      <unknown profile>               : VAEntrypointVLD
      <unknown profile>               : VAEntrypointVLD
```

上面的输出，可以看到支持的 VA-API 版本为 1.14，支持 MPEG2、MPEG4、H264、VC1、HEVC 等的解码。

## 应用程序支持

有了 libva，还需要应用程序支持，一般来说，应用程序为了更多的通用性，会支持多种硬件加速方案以及软解方案，一般通过配置文件或命令行参数来切换。

就拿常用的播放器 mpv 来说，它具有良好的硬件加速支持，但默认情况下未启用。要启用它，需要使用 --hwdec 命令行开关。也可以通过在 mpv 的配置文件（一般位于 $HOME/.config/mpv/mpv.conf）中添加“hwdec”之类的参数值。

在我这台兆芯的机器上，运行如下命令：

```
$ sudo apt install mpv
$ mpv --hwdec=vaapi --log-file=mpv.log 1.mp4
```

查看 log 文件，有如下输出：

```
[   0.204][v][vd] Codec list:
[   0.204][v][vd]     h264 - H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10
[   0.204][v][vd]     h264_omx_dec (h264) - OpenMAX IL H.264 video decoder
[   0.204][v][vd]     h264ftomx (h264) - ft omx H.264 decoder wrapper
[   0.204][v][vd]     h264_v4l2m2m (h264) - V4L2 mem2mem H.264 decoder wrapper
[   0.204][v][vd]     h264_qsv (h264) - H264 video (Intel Quick Sync Video acceleration)
[   0.204][v][vd]     h264_cuvid (h264) - Nvidia CUVID H264 decoder
[   0.204][v][vd] Opening decoder h264
[   0.205][v][vd] Looking at hwdec h264-vaapi...
[   0.205][v][vo/gpu] Loading hwdec driver 'vaapi-egl'
[   0.205][v][vo/gpu/vaapi-egl] using VAAPI EGL interop
[   0.205][v][vo/gpu/vaapi-egl] Trying to open a x11 VA display...
[   0.205][d][vo/gpu/vaapi-egl/vaapi] libva: VA-API version 1.14.0
[   0.205][d][vo/gpu/vaapi-egl/vaapi] libva: Trying to open /usr/lib/x86_64-linux-gnu/dri/zx_drv_video.so
[   0.237][d][vo/gpu/vaapi-egl/vaapi] libva: Found init function __vaDriverInit_1_0
[   0.352][d][vo/gpu/vaapi-egl/vaapi] libva: va_openDriver() returns 0
[   0.352][v][vo/gpu/vaapi-egl/vaapi] Initialized VAAPI: version 1.14
[   0.352][d][ffmpeg] AVHWDeviceContext: VAAPI driver: ZX_VA.
[   0.352][d][ffmpeg] AVHWDeviceContext: Driver not found in known nonstandard list, using standard behaviour.
[   0.352][v][vo/gpu/vaapi-egl] Going to probe surface formats (may log bogus errors)...
[   0.354][d][vo/gpu/vaapi-egl] Supported formats:
[   0.354][d][vo/gpu/vaapi-egl]  nv12
[   0.354][d][vo/gpu/vaapi-egl]  p010
[   0.354][d][vo/gpu/vaapi-egl]  bgra
[   0.354][d][vo/gpu/vaapi-egl]  rgba
[   0.354][d][vo/gpu/vaapi-egl]  rgb0
[   0.354][d][vo/gpu/vaapi-egl]  bgr0
[   0.354][v][vo/gpu/vaapi-egl] Done probing surface formats.
```
播放过程中按 i 键，显示解码信息：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202411/images/deepin_vaapi_03.png)

可以看出，是启用了 vaapi 解码。

mpv 是一个命令行程序，主要用来进行媒体播放验证，在日常使用中，用得比较多的是 VLC 播放器。

VLC 中的硬件加速在界面中通过“工具 → 偏好设置 → 输入/编解码器 → 硬件加速解码”进行控制。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202411/images/deepin_vaapi_04.png)

如果使用硬件加速，VLC 的输出将包含如下行：

```
libva info: VA-API version 1.14.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/zx_drv_video.so
libva info: Found init function __vaDriverInit_1_0
libva info: va_openDriver() returns 0
GLSL: In function 'main':
GLSL:10: warning: 
GLSL:10: warning: 
[00007f9154ce4020] avcodec decoder: Using ZX_VA for hardware decoding
```

## 小结

Linux 下的硬件视频加速是一个复杂的生态，涉及从硬件驱动到应用适配的多个环节。作为 Linux 下视频硬件加速的核心组件之一，VA-API 凭借其广泛的硬件兼容性和丰富的功能支持，在提升视频处理性能方面表现出色。通过 VA-API，开发者和用户可以显著优化视频播放体验，减少卡顿现象。在信创软件中集成 libva，不仅能充分发挥硬件性能，还能显著提升用户体验，成为优化整体生态的关键一环。