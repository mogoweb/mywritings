# Chromium 改造实录：增加 MP2 音频支持

在上一篇文章《[Chromium 改造实录：增加 MPEG TS 格式支持](https://mp.weixin.qq.com/s/enrzjVLy_VACqTKavuam7Q)》中，讲了如何为 Chromium 增加 MPEG TS 流的支持。但这项任务并没有结束，因为 TS 只是一种容器格式，现在只是相当于把盖子打开了，而里面的视频流和音频流能否播放，取决于音视频采用何种编码格式以及这些编码格式是否支持。

在解决了 TS 流中 H264 视频编码的解码问题后，我又碰到了音频解码问题。从 log 上看有如下信息：

```
03-08 06:38:13.096 31080 31932 V chromium: [VERBOSE1:ffmpeg_common.cc(122)] Unknown audio CodecID: 86016
03-08 06:38:13.096 31080 31932 V chromium: [VERBOSE1:batching_media_log.cc(37)] MediaEvent: {"debug":"Warning, FFmpegDemuxer failed to create a valid/supported audio decoder configuration from muxed str}
```

这里的 audio CodecID 是指定义在 third_party/ffmpeg/libavcodec/codec_id.h 文件中的 。这个文件中的 ID 以十六进制形式定义，简单转换一下，可以知道 log 中的 ID 对应的是 AV_CODEC_ID_MP2。

又是一个比较古旧的音频格式，而不是我们熟知的 MP3 和 AAC。关于 MP2 格式，介绍如下：

> MP2 是 MPEG-1 Layer II 的缩写，它是一种有损压缩的音频格式，文件扩展名为 .mp2 。对于广播电视制作行业的人来说， MP2 是很常见的音频文件格式。 MP2 主要应用在标准化数字广播和数字电视广播（DAB，DMB，DVB）的数字音频和视频编码。 标准 MPEG II 音频格式支持 32, 44.1 和 48 kHz 采样率以及 32 至 320 kbps / 秒的比特率。
>
> 当MP2的比特率达到 256 kbps及以上时，可具有很好的错误恢复能力和更好的音质，是广播电视行业的主导音频标准。
>
> 相比 MP3 ，MP2 格式具有更好的音质（数据压缩率较小）。

FFmpeg 作为一个全能型的开源媒体库，对 MP2 格式有着完善的支持，问题在于谷歌的工程师对一些老旧的格式不待见，所以在 Chromium 中并没有打算支持。好在 Chromium 代码的可扩展性还不错，所以往 Chromium 中增加一种新格式的支持比较简单。

有资料说 MP3 的解码器一样能兼容解码 MP2，但我尝试过不行，至少在 Chromium 中是不行的，所以还是得把 MP2 单列出来进行处理。

在 Chromium 中维护了一份音频编码列表，定义在 media/base/audio_codecs.h 文件中：

```
enum class AudioCodec {
  // These values are histogrammed over time; do not change their ordinal
  // values.  When deleting a codec replace it with a dummy value; when adding a
  // codec, do so at the bottom before kMaxValue, and update the value of
  // kMaxValue to equal the new codec.
  kUnknown = 0,
  kAAC = 1,
  kMP3 = 2,
  kPCM = 3,
  kVorbis = 4,
  kFLAC = 5,
  kAMR_NB = 6,
  kAMR_WB = 7,
  kPCM_MULAW = 8,
  kGSM_MS = 9,
  kPCM_S16BE = 10,
  kPCM_S24BE = 11,
  kOpus = 12,
  kEAC3 = 13,
  kPCM_ALAW = 14,
  kALAC = 15,
  kAC3 = 16,
  kMpegHAudio = 17,
  kDTS = 18,
  kDTSXP2 = 19,
  // DO NOT ADD RANDOM AUDIO CODECS!
  //
  // The only acceptable time to add a new codec is if there is production code
  // that uses said codec in the same CL.

  // Must always be equal to the largest entry ever logged.
  kMaxValue = kDTSXP2,
};
```

第一步，在 kMaxValue 枚举值前添加一项 kMP2 = 20，并将 kMaxValue 值修改为等于 kMP2 即可。

第二步，修改 media/base/audio_codecs.cc 文件，建立 name 和 ID 间的对应关系。

第三步，修改 media/base/supported_types.cc 和 media/filters/android/media_codec_audio_decoder.cc
 文件，明确指示 Chromium 将 MP2 格式支持起来。搜索 kMP3 关键字，在有 kMP3 的地方，将 kMP2 也加上。

第四步，修改 media/ffmpeg/ffmpeg_common.cc 文件，建立 FFmpeg Codec ID 和 Chromium 中 Codec 枚举值之间的关联。

最后一步，按照《[Chromium 改造实录：增加 MPEG TS 格式支持](https://mp.weixin.qq.com/s/enrzjVLy_VACqTKavuam7Q)》中的方法，修改 codec_list.c
 文件，添加 ff_mp2_decoder 解码器，修改 config_components.h 文件，开启 CONFIG_MP2_DECODER 宏。

这样，对于 MP2 解码的支持就增加上了。当然，这里只是针对媒体容器中包含 MP2 音频流进行支持。如果是 MP2 视频，那又是一个话题。

另外，TS 流中的音频流也可能是 MP3、AAC、AC3 等编码格式，各种组合加起来非常多。所以在实际工作中，有关音视频的坑非常多。我现在就是将各种音视频格式都支持起来，毕竟在现实中，指不定会冒出怎样的音视频文件。

欢迎各位围观我的挖坑填坑囧途。
