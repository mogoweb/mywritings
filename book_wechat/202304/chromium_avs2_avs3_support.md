# Chromium 改造实录：国标AVS2 & AVS3 支持起来

距离今年的五一长假只有几个小时了，一般重大节日也是项目的里程碑（milestone）节点，我也赶在五一长假之前完成了中国标准 AVS2 和 AVS3 在 Chromium 浏览器上的支持工作。

有句话，“一流企业做标准，二流企业做品牌，三流企业做产品”。在这一点上，中国企业一向做得不够，在很多重要的标准制定上没有话语权，但这种情况慢慢在改进。比如在音视频领域，中国也推出了自己的超高清标准方案：AVS2 和 AVS3。

> AVS2 视频编码是《信息技术高效多媒体编码 第2部分：视频》的简称，是我国具备自主知识产权的第二代信源编码标准，也是数字音视频产业的共性基础标准。AVS2 视频编码支持超高分辨率（至少为4k x 2k）、高动态范围的视频编码，在编码效率方面比当前国际国内标准提高50%以上，适用于地面电视广播、有线电视广播、卫星电视广播等应用。AVS2 视频编码于2016年12月30日颁布为国家标准 GB/T 33475.2-2016，并于 2018 年被全球超高清联盟采纳。
>
> AVS3是数字音视频编解码技术标准工作组制定的第三代音视频编解码技术标准，也是全球首个已推出的面向 8K 及 5G 产业应用的视频编码标准。 AVS3 主要面向超高清广播影视、全景视频、增强现实/虚拟现实等应用，以及自动驾驶、智慧城市、智慧医疗、智能监控等。支持超高分辨率(8K以上)、全景视频、三维视频、屏幕混合内容视频、高动态范围视频的智能压缩和沉浸式音频场景的应用。
> 
> 2021 年 11 月，AVS3 视频标准发布为 IEEE 1857.10 标准。目前正在和欧洲 DVB 组织开展合作，2021 年 9 月，DVB Steering Board 通过电子邮件确认 AVS3 标准可以被 DVB 参考，同时 AVS 知识产权政策和 DVB 的知识产权政策相符。

具体说来，AVS2 主要对标 H264，提供最高 4K 的分辨率，而 AVS3 则对标 H265（HEVC），提供最高 8K 的分辨率。在标准化方面也取得了进展，进入了国际标准组织。

但在实现层面，还有太多工作需要做。目前大多开源库都缺乏对 AVS2 和 AVS3 的支持，比如我正在着手的浏览器项目就没考虑 AVS2 和 AVS3 的存在。

下面就谈谈 Chromium 上支持 AVS2 和 AVS3 的改造思路。

#### 一、限定条件

视频解码分硬解和软件，软解通用性比较好，但对 CPU 要求高，对于大码率高分辨率可能会掉帧或卡顿。在 Android 系统上，一般是使用专门的解码芯片来处理视频编解码。所以这里将条件限定为：

1. Android 操作系统
2. Android 解码芯片支持 AVS2 / AVS3 解码
3. Android MediaCodec 框架支持 AVS2 / AVS3 解码
4. Chromium for Android

这里需要说一下，MediaCodec 是一个新的 Android 媒体框架，可以用来访问低层次的媒体编解码器，它是 Android 低层次多媒体支持基础设施的一部分。MediaCodec 可以处理视频数据包/缓冲区的解码或编码，并负责与编解码器的交互。MediaCodec 从Android 4.1（API级别16）开始引入，这样做的目的是为了提供更高效和灵活的媒体处理能力，让开发者可以直接访问硬件加速的编解码器，而不需要通过高层次的 API，如 MediaPlayer。

新的 Chromium 使用了 MediaCodec 框架，增加灵活性的同时，也意味着在 Chromium 内部要做更多的媒体处理工作（流媒体解析、接收数据、demuxer 等），而调用 MediaPlayer 框架则所有媒体处理的任务都转交出去，这将受限于 MediaPlayer 框架的能力（比如增加媒体格式的支持等）。 

#### 二、增加新的 Codec ID

在 Chromium 中维护了一份视频编码列表，定义在 media/base/video_codecs.h 文件中：

```
enum class VideoCodec {
  // These values are histogrammed over time; do not change their ordinal
  // values.  When deleting a codec replace it with a dummy value; when adding a
  // codec, do so at the bottom (and update kMaxValue).
  kUnknown = 0,
  kH264,
  kVC1,
  kMPEG2,
  kMPEG4,
  kTheora,
  kVP8,
  kVP9,
  kHEVC,
  kDolbyVision,
  kAV1,
  // DO NOT ADD RANDOM VIDEO CODECS!
  //
  // The only acceptable time to add a new codec is if there is production code
  // that uses said codec in the same CL.

  kMaxValue = kAVS3,  // Must equal the last "real" codec above.
};
```

第一步，在 kMaxValue 枚举值前添加一项 kAVS2 和 kAVS3，并将 kMaxValue 值修改为等于 kAVS3。

第二步，修改 media/base/video_codecs.cc 文件，建立 name 和 ID 间的对应关系。

第三步，修改 media/base/supported_types.cc 和 media/base/android/media_codec_util.cc 文件，明确指示 Chromium 将 AVS2 和 AVS3 格式支持起来。编译的时候还会提示有些 switch case 分支没有处理，加上相应的即可。

第四步，修改 media/ffmpeg/ffmpeg_common.cc 文件，建立 FFmpeg Codec ID 和 Chromium 中 Codec 枚举值之间的关联。

#### 二、FFmpeg 修改

FFmpeg 本身已经支持了 AVS2 和 AVS3 流的解析，但是解码依赖于第三方开源实现，好在我们并不需要其解码，所以对于 FFmpeg 只需做少量修改。Chromium 中将 AVS2 与 AVS3 相关的编译开关都关掉了，需要将其打开。

按照《[Chromium 改造实录：增加 MPEG TS 格式支持](https://mp.weixin.qq.com/s/enrzjVLy_VACqTKavuam7Q)》中的方法，修改 parser_list.c 文件，添加 ff_avs2_parser 和 ff_avs3_parser 解码器，修改 config_components.h 文件，开启与 AVS2 和 AVS3 宏。

修改 ffmpeg_generated.gni 文件，将与 AVS2 和 AVS3 相关的代码文件加入编译。

注意，在视频解码中还有一个 profile 的值，在 avs2.h 和 avs3.h 中，有 AVS2 和 AVS3 的 profile 限定值定义：

```
enum AVS2Profile {
    AVS2_PROFILE_MAIN_PIC   = 0x12,
    AVS2_PROFILE_MAIN       = 0x20,
    AVS2_PROFILE_MAIN10     = 0x22,
};

#define AVS3_PROFILE_BASELINE_MAIN   0x20
#define AVS3_PROFILE_BASELINE_MAIN10 0x22
```

这个值需要和 Chromium 中的 profile 枚举值定义对应起来。

还要注意的是，AVS2 和 AVS3 的 parser 在解析出 profile 后，并没有将值设到 AVCodecContext 结构中，需要修改相应的代码。

#### 三、修改 Java 代码

这部分代码主要是和 Android 的 MediaCodec 框架交互，因为 AOSP 中的 MediaCodec 并没有对 AVS2 / AVS3 的支持，所以需要增加对 AVS2 / AVS3 的处理。

MediaCodec 框架对 AVS2 / AVS3 的支持不在本文讨论之内，这里只说说 Chromium 中的修改：

在 MediaFormatBuilder.java 中，有针对不同的 mime 类型进行处理，但是缺少 AVS2 和 AVS3。我们需要在 media_codec_util.cc 中增加 AVS2 / AVS3 的 mime 定义，一般定义为 video/avs2 和 video/avs3 即可。所以在 addInputSizeInfoToFormat 中增加 AVS2 和 AVS3 的 case 即可。

#### 四、小结

Chromium 新的媒体框架非常灵活，新的解码器和格式都可以支持起来，但这也意味着开发人员要投入更多的精力去了解媒体播放相关的细节。媒体播放也是一大天坑，各种编解码、容器、协议组合，还有码率、容错之类，会导致问题层出不穷，主要是市面上各种音视频格式太多，要全部支持起来非常困难。就拿 AVS2 / AVS3 来说，虽然按照上面的步骤基本能支撑起来，但是对于高规格的 Profie 的视频，播放还是会出现 crash，这种问题只能去死扣细节，一点点的把 BUG 消除掉。

路漫漫其修远兮，吾将上下求索。

