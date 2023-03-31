# 选择最新 Chromium，支持 H264 / H265

在做了充分的准备后，我信心满满的向着 H265 8K 视频解码这个目标发起进攻，然而，正打算动手的时候，我突然发现，别说支持 H265 ，自编的 Chromium WebView 连 H264 解码都不支持。使用 WebView Shell 访问测试页面，结果如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/chromium_h264_01.png)

错误日志如下：

```
03-05 23:20:21.731  9061  9124 E chromium: [ERROR:batching_media_log.cc(26)] MediaEvent: MEDIA_ERROR_LOG_ENTRY {"error":"FFmpegDemuxer: no supported streams"}
```

令我不解的是，Android 10 的预编译 Webview 却没这样的问题。

测试的视频是 MP4 格式，从日志上看，大致可以判断是对流的解析出现问题。浏览 media 部分的代码，可以看到有 mp4_stream_parser.h / mp4_stream_parser.cc 文件，基本可以判断 MP4StreamParser 类的作用就是解析 MP4 格式的。StreamParser 使用工厂模式创建，工厂类为 StreamParserFactory，查看源码，可以发现有很多代码被 BUILDFLAG(USE_PROPRIETARY_CODECS) 宏包起来了。

联想到 Chromium 文档中有提到专有解码器的，原话为：

```
此外，您可能希望包括对专有音频和视频编解码器的支持，就像 Google 的 WebView 所做的那样。 这些编解码器可能受到专利或许可协议的保护，在分发包含它们的 WebView 构建之前，您应该寻求法律建议。
```

因为平台是支持硬解 H264 / H265 的，没有联想到和这个有关系。赶紧加到编译选项中：

```
ffmpeg_branding = "Chrome"
proprietary_codecs = true
```

问题得到圆满的解决？答案是没有。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/chromium_h264_02.png)

点击页面上的播放箭头，没有视频画面出现，查看 log，有如下信息：

```
03-04 14:11:37.069 26846 26910 W cr_MediaCodecUtil: HW encoder for video/avc is not available on this device.
```

怎么会找不到硬件解码器？

之前看 Media 模块的文档，文档讲到可通过 chrome://media-internals 可以查看媒体解码信息以及相关日志。WebView Shell 并不支持在地址栏输入 chrome://media-internals 。这也难不倒我，可以在这个代码上编译出一个 Chromium 浏览器。

```
$ autoninja -C out/Default chrome_public_apk
```

安装之后，一个 tab 播放网页，另一个 tab 执行  chrome://media-internals ，可以看到：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/chromium_h264_03.png)

视频解码器为 MojoVideoDecoder ，而使用 Google 官方发布的 Chrome for Android，视频解码器为 MediaCodecVideoDecoder：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/chromium_h264_04.png)

从对比图看似乎是解码器创建错误。但经过一番代码跟踪，其实 MojoVideoDecoder 还是会调用 MediaCodecVideoDecoder。这里不要被 MojoVideoDecoder 给骗了，以为它是对表情符进行解码的。关于 Mojo 的解释如下：

> We use mojom interfaces as the transport layer of each media component to support hosting them remotely. These interfaces are called media player mojo interfaces. They are very similar to their C++ counterparts:

也就是说 Mojo 只是一个抽象层，在 Android 平台上，还是会走到 MediaCodec 组件进行解码。至于这里为什么显示使用的 MojoVideoDecoder，原因在于 MediaCodecVideoDecoder 没有创建成功。从 log 上也印证了这一点：

```
03-07 08:35:05.895 29840 30203 I ACodec  : 0x6fbdb77900 [OMX.uapi.video.decoder.avc] setting nBufferCountActual to 7 *minUndequeuedBuffers: 4  extraBuffers:3
03-07 08:35:05.895  3081 30389 I SOCT_OMXVDEC: set_parameter: setted buf cnt 7 exceed range(0, 3)!
03-07 08:35:05.895  3081 30389 E OMXNodeInstance: setParameter(0xe836c360:uapi.decoder.avc, ParamPortDefinition(0x2000001)) ERROR: Undefined(0x80001001)
03-07 08:35:05.895 29840 30203 W ACodec  : [OMX.uapi.video.decoder.avc] setting nBufferCountActual to 7 failed: -2147483648
03-07 08:35:05.895 29840 30203 I ACodec  : 0x6fbdb77900 [OMX.uapi.video.decoder.avc] setting nBufferCountActual to 6 *minUndequeuedBuffers: 4  extraBuffers:2
03-07 08:35:05.895  3081 30389 I SOCT_OMXVDEC: set_parameter: setted buf cnt 6 exceed range(0, 3)!
03-07 08:35:05.895  3081 30389 E OMXNodeInstance: setParameter(0xe836c360:uapi.decoder.avc, ParamPortDefinition(0x2000001)) ERROR: Undefined(0x80001001)
03-07 08:35:05.895 29840 30203 W ACodec  : [OMX.uapi.video.decoder.avc] setting nBufferCountActual to 6 failed: -2147483648
03-07 08:35:05.895 29840 30203 I ACodec  : 0x6fbdb77900 [OMX.uapi.video.decoder.avc] setting nBufferCountActual to 5 *minUndequeuedBuffers: 4  extraBuffers:1
03-07 08:35:05.895  3081 30389 I SOCT_OMXVDEC: set_parameter: setted buf cnt 5 exceed range(0, 3)!
03-07 08:35:05.896  3081 30389 E OMXNodeInstance: setParameter(0xe836c360:uapi.decoder.avc, ParamPortDefinition(0x2000001)) ERROR: Undefined(0x80001001)
03-07 08:35:05.896 29840 30203 W ACodec  : [OMX.uapi.video.decoder.avc] setting nBufferCountActual to 5 failed: -2147483648
03-07 08:35:05.896 29840 30203 I ACodec  : 0x6fbdb77900 [OMX.uapi.video.decoder.avc] setting nBufferCountActual to 4 *minUndequeuedBuffers: 4  extraBuffers:0
03-07 08:35:05.896  3081 30389 I SOCT_OMXVDEC: set_parameter: setted buf cnt 4 exceed range(0, 3)!
03-07 08:35:05.896  3081 30389 E OMXNodeInstance: setParameter(0xe836c360:uapi.decoder.avc, ParamPortDefinition(0x2000001)) ERROR: Undefined(0x80001001)
03-07 08:35:05.896 29840 30203 W ACodec  : [OMX.uapi.video.decoder.avc] setting nBufferCountActual to 4 failed: -2147483648
03-07 08:35:05.896 29840 30203 E ACodec  : Failed to allocate buffers after transitioning to IDLE state (error 0x80000000)
03-07 08:35:05.896 29840 30203 E ACodec  : signalError(omxError 0x80001001, internalError -2147483648)
03-07 08:35:05.896 29840 30202 E MediaCodec: Codec reported err 0x80001001, actionCode 0, while in state 5
```

在梳理了一遍又一遍的流程，跟踪调试了一遍又一遍的代码，两眼发花。这样持续了三天，一点没找到头绪。再 google 一把，有人说从 Chromium 105 之后的版本开始，对于 H265 的支持比较完善。那编译一个最新的版本试试吧，看看具体是什么情况。

由于前面坑都踩过了一遍，在现有代码上切换新版本很顺利。编译运行后发现，H264 / H265 的支持都没有问题。

选择不那么新的版本，主要是考虑想更快的熟悉代码。但即使不是那么新的中间版本，代码已经改得面目全非。而最新版本解决了大麻烦，那还是选择最新的版本吧。最终我选择的是一个稳定版本 111.0.5563.49 。

最后需要说明一下，不能简单说 Chromium 105 之后的版本支持 H264 / H265，在 Android 上，还取决于 MediaCodec 组件的解码能力，Chromium 只是把上面的流程走通了，但实际解码还是依赖底层硬件。

如何判断系统的 MediaCodec 对各种视频编码格式的支持，这里需要介绍 google 的开源播放器 exoplayer。

> ExoPlayer 是适用于 Android 的应用程序级媒体播放器。 它不基于 MediaPlayer API 开发，所以 ExoPlayer 支持 Android 的 MediaPlayer API 目前不支持的功能，包括 DASH 和 SmoothStreaming 自适应播放。 与 MediaPlayer API 不同，ExoPlayer 易于定制和扩展，并且可以通过 Play Store 应用程序更新进行更新。

exoplayer 的项目地址：

> https://github.com/google/ExoPlayer

构建和安装 exoplayer 后，可以使用命令行来播放指定的视频：

```
$ adb shell am start -a com.google.android.exoplayer.demo.action.VIEW -d <url>
```

如果某种格式在 chromium 中无法播放，先使用 exoplayer 确认一下，在 MediaCodec 这一层面上时候支持，可以更好的确认问题是在 chromium 还是在 Android 系统层。
