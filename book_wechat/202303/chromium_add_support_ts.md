# Chromium 改造实录：增加 MPEG TS 格式支持

在《[选择最新 Chromium，支持 H264 / H265](https://mp.weixin.qq.com/s/IpioXG-_NaGOc9nnKe6xbQ)》一文中，记录了我通过升级 Chromium 版本解决了 H264 / H265 视频支持难题。然而难题接踵而至，这次的难题是 MPEG TS 流的支持。

MPEG2-TS 传输流广泛应用于数字电视广播系统，所以是一个不得不支持的格式。通过查询资料，了解到 FFmpeg 是支持 TS 格式的，但 Chromium 中并没有默认开启这个功能。这可能是出于版权、性能或者兼容性的考虑。

关于如何让 Chromium 支持 TS，我请教了一下 AI。AI 给的思路是：

* 修改 FFmpeg 的配置文件，开启 MPEG TS 的解复用器和解码器；
* 修改 Chromium for Android 的媒体框架，添加对 MPEG TS 的支持；
* 修改 Chromium for Android 的网络模块，添加对 MPEG TS 的传输协议的支持。

按照 AI 的建议，我一步步解决了 TS 格式的支持问题。

#### 一

第一步研究 Chromium 的编译参数。媒体有关的编译选项主要位于 media 目录下的 media_options/gni 文件中。研究了一下，发现一个 enable_mse_mpeg2ts_stream_parser 参数，目前的值是：

```
  enable_mse_mpeg2ts_stream_parser =
      proprietary_codecs && (enable_cast_receiver || use_fuzzing_engine)
```

对于 Chromium for Android，上面的值是 false。看起来这个选项是与 MSE(Media Source Extensions) 有关，但实际上也会影响 TS 文件的解析，因为在 mime_util_internal.cc 中有这样的代码：

```
#if BUILDFLAG(ENABLE_MSE_MPEG2TS_STREAM_PARSER)
  CodecSet mp2t_codecs{H264, MPEG2_AAC, MPEG4_AAC, MP3};
  AddContainerWithCodecs("video/mp2t", mp2t_codecs);
#endif  // BUILDFLAG(ENABLE_MSE_MPEG2TS_STREAM_PARSER)
```

TS 容器的 mime type 为 video/mp2t，只有开启了 enable_mse_mpeg2ts_stream_parser，才会加入对 TS 容器的处理。所以首先需要将 enable_mse_mpeg2ts_stream_parser 选项修改为 true。

修改这个参数后，立刻又碰到一个新的错误信息：

```
03-09 07:44:55.322 31903 32010 V chromium: [VERBOSE1:batching_media_log.cc(35)] MediaEvent: {"error":"FFmpegDemuxer: open context failed"}
```

#### 二

稍微捋一下代码，可以发现 open context 的代码实际位于 FFmpegGlue::OpenContext，这个方法中又会调用 FFMPEG 的 avformat_open_input 函数。

接下来的调用流程:

```
avformat_open_input -> init_input -> av_probe_input_buffer2 -> av_probe_input_format2 -> av_probe_input_format3 -> av_demuxer_iterator
```

可以很明显看出，av_demuxer_iterator 就是遍历系统的 demuxer，那这个 demuxer 列表来自哪儿呢？

查看其实现文件 allformats.c，可以看到它有如下包含文件：

```
#include "libavformat/demuxer_list.c"
```

也就是在 demuxer_list.c 这个文件里面进行了定义。搜索 demuxer_list.c 文件，发现这个文件有多份，位于 third_party/ffmpeg/chromium/config 下。

这个 config 下的文件组织有些讲究，最上一层是 branding，也就是我们在 args.gn 下定义的 ffmpeg_branding 选项，默认是 Chromiium。再下一层是按 OS 划分，接下来就是按 CPU 分。找出你的目标平台，修改 demuxer_list.c，增加一行：

```
&ff_mpegts_demuxer,
```

接下来还要修改 FFmpegGlue::OpenContext 方法，在

```
DCHECK_NE(container_, container_names::CONTAINER_UNKNOWN);
```

前面增加容器判断：

```
  else if (strcmp(format_context_->iformat->name, "mpegts") == 0)
    container_ = container_names::CONTAINER_MPEG2TS;
```

解决了上面的错误后，又会碰到新的错误：

```
03-04 09:53:29.573 28173 28233 W [FFMPEG]: Could not find codec parameters for stream 0 (Video: h264 ([27][0][0][0] / 0x001B), none): unspecified size
03-04 09:53:29.573 28173 28233 W [FFMPEG]: Consider increasing the value for the 'analyzeduration' (60000000) and 'probesize' (5000000) options
03-04 09:53:29.573 28173 28233 W [FFMPEG]: Could not find codec parameters for stream 1 (Audio: ac3 (AC-3 / 0x332D4341), 0 channels): unspecified sample rate
03-04 09:53:29.573 28173 28233 W [FFMPEG]: Consider increasing the value for the 'analyzeduration' (60000000) and 'probesize' (5000000) options
03-04 09:53:29.573 28173 28233 D [FFMPEG]: After avformat_find_stream_info() pos: 0 bytes read:36960112 seeks:8 frames:292
03-04 09:53:29.574 28173 28247 V chromium: [VERBOSE1:ffmpeg_common.cc(246)] Unknown profile id: -2659
03-04 09:53:29.575 28173 28247 V chromium: [VERBOSE1:batching_media_log.cc(37)] MediaEvent: {"debug":"Warning, FFmpegDemuxer failed to create a valid/supported video decoder configuration from muxed str}
03-04 09:53:29.575 28173 28247 V chromium: [VERBOSE1:batching_media_log.cc(37)] MediaEvent: {"info":"FFmpegDemuxer: skipping invalid or unsupported video track"}
03-04 09:53:29.575 28173 28247 V chromium: [VERBOSE1:ffmpeg_common.cc(122)] Unknown audio CodecID: 86019
03-04 09:53:29.575 28173 28247 V chromium: [VERBOSE1:ffmpeg_common.cc(294)] Unknown AVSampleFormat: -1
```

#### 三

看日志，似乎是解码器创建失败，有了前面的经验，直接找到 codec_list.cc 文件，加入如下行：

```
&ff_h264_decoder,
```

同时修改 config_components.h 文件， 将：

```
#define CONFIG_H264_DECODER 0
```

修改为：

```
#define CONFIG_H264_DECODER 1
```

经过这样的修改后，我们会发现出现很多链接错误。没有关系，将缺的代码文件加入编译即可。

需要注意的是，有些底层代码是使用汇编语言编写，在 Android 平台下就是那些以 .S 为后缀的文件。非常神奇的是，clang 直接支持编译汇编语言，所以将这些汇编文件和 C 文件一起加入编译列表即可。

经过这样一番操作，TS 流终于播放起来了。需要说明的是，我测试使用的 TS 流，内部视频采用的 H264 编码，如果采用其他格式编码，修改的过程会有所不同，但思路类似。

#### 四

经过上述的改造，是否就完美解决问题了呢？并没有。测试下来，发现视频播放出来了，但是没有声音，从日志可以看出：

```
03-05 21:14:18.706 18936 19047 V chromium: [VERBOSE1:batching_media_log.cc(37)] MediaEvent: {"debug":"Warning, FFmpegDemuxer failed to create a valid/supported audio decoder configuration from muxed str}
03-05 21:14:18.707 18936 19047 V chromium: [VERBOSE1:batching_media_log.cc(37)] MediaEvent: {"info":"FFmpegDemuxer: skipping invalid or unsupported audio track"}
```

解决的方法类似，但真正解决声音输出，又花费了一番精力，这个留在下一篇继续，敬请关注！