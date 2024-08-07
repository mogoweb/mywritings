# Chromium 改造实录：8K 来了

2008 年 2 月 16 日，日本东芝公司宣布放弃 HD-DVD 格式，宣告这场大约持续了 6 年时间的高清光碟之战结束。东芝的 HD-DVD 彻底失败，而索尼的 Blu-ray Disc 大获全胜，宣告着高清时代的到来。

还记得初次接触到 1080P 的高清样片，简直不敢相信自己的眼睛。对于从 VCD 时代走过来的我，在大学校园则更多的是接触的 RM 视频。那个时候，能够有 DVD 画质（720 x 480）就非常满足。画质一下子提升到 1920 x 1080，这提升幅度相当惊人。我从来没有见过这么清晰的画面，每一个细节都栩栩如生，每一个颜色都鲜艳夺目。

当然 1080P 高清画质带来的是巨大的体积，动辄几十 G 的影片，别说在线播放（当时家庭宽带主流是 1M ADSL），连存储都是一个巨大问题（当时电脑主流硬盘大小大约一两百 G）。而且刚开始的时候蓝光光驱和光盘都比较贵，所以面对 1080P 的片源，只能是可望不可及，也没有奢望有一天能够在互联网上能够随意点播高清视频。

技术的发展总是出人意料，Sony 的 Blu-ray Disc 也没有笑到最后，因为它面临的是互联网这个强大的对手。随着带宽的提升，视频编解码技术的发展，硬盘容量的不断扩大，人们越来越习惯从互联网获取片源，甚至在线观看。光驱和光盘播放器这种设备也被淘汰进了历史的垃圾堆。现在即使手上有光碟，也很难找得到设备播放。

就在人们逐渐习惯高清视频时，科技仍然在进步，4K 视频已经开始走进人们的生活。充 VIP 会员，就可以在爱优腾这样的视频网站享受 4K 高清画质。但，这仍然不是终点，8K 普及也开始上路。

日本广播协会（NHK）是 8K 技术发展的主要推动者之一。从 2013 年开始，NHK 就利用 8K 技术拍摄和直播了多个重大事件，如女足世界杯、里约奥运会、平昌冬奥会、东京奥运会等。2018年12月，NHK成为全球首个以8K分辨率播放电视节目的电视台。

然而，8K 技术的普及还任重道远。就拿 Chromium 来说，作为最激进的技术派，竟然还不支持 8K。

下面就以 Chromium for Android 为例说明如何支持 8K 视频播放。

#### 一

8K 是一种超高清视频技术，它的分辨率达到了 7680×4320 像素，是 4K 的四倍，是 1080P 的十六倍。由于 8K 内容的数据量非常大，必须采用一些高效的编解码技术才能实现压缩和传输。目前，有以下几种编码格式可以支持8K视频：

* HEVC（High Efficiency Video Coding），也称为 H.265 ，是一种由 ITU-T 和 ISO/IEC 联合制定的视频编码标准，是 H.264 的后继者。HEVC 可以提供与 H.264 相同的视频质量，但是数据量只有一半。HEVC 可以支持最高 8192×4320 像素的分辨率，以及最高 300 帧每秒的帧率。
* AV1（AOMedia Video 1），是一种由联盟开放媒体（AOMedia）制定的开源、免版税的视频编码标准，是 VP9 的后继者。AV1 旨在提供比 HEVC 更高的压缩效率和更低的传输带宽需求。AV1 可以支持最高 7680×4320 像素的分辨率，以及最高 120 帧每秒的帧率。
* VVC（Versatile Video Coding），也称为 H.266 ，是一种由 ITU-T 和 ISO/IEC 联合制定的视频编码标准，是 HEVC 的后继者。VVC 旨在提供比 HEVC 更高的压缩效率和更好的视频质量。VVC 可以支持最高 16384×8192 像素的分辨率，以及最高 300 帧每秒的帧率。
* AVS3（Audio Video Standard 3），是一种由中国制定的视频编码标准，是 AVS2 的后继者。AVS3 是全球首个已推出的面向 8K 及 5G 产业应用的视频编码标准，技术先进，专利清晰，是 5G+8K 中最合适的视频编码标准，将引领未来五到十年 8K 超高清、 VR 视频产业的发展。AVS3 可以支持最高 8192×4320 像素的分辨率，以及最高 120 帧每秒的帧率。

所以，第一步就是需要至少支持上述编码格式中的一种。目前市面上主流的是采用 HEVC 编码技术，无论是片源，还是播放器，支持的都比较多。

Chromium for Android 由于解码使用了 Android 系统的 MediaCodec 框架，其解码能力取决于 Android 系统的媒体框架。此外，Chromium 自身还使用 FFmpeg 做 demuxer，对于编解码格式的判断则是由 FFmpeg 来完成，所以，对于视频解码的支持还涉及到 FFmpeg 的能力。非常欣慰的是， FFmpeg 已经合入了中国开发者的提交，支持 AVS3，对 HEVC 和 AV1 的支持更不在话下，VVC 也有支持方案，不愧为最全能的媒体库。

然而，有些不太友好的是，Chromium 自身并没有加入对 AVS3 的支持。也就是说即使 FFmpeg 能识别 AVS3 码流，Chromium 也装作不认识，不会交给 MediaCodec 去解码。

目前项目中的 MediaCodec 还不支持 AVS3 解码，所以没有探索如何让 Chromium 支持 AVS3。目前做的工作是先把 HEVC 8K 支持起来。

#### 二

解决了编码格式支持后，还需要解除 Chromium 对于视频大小的限制，不知道是出于何种考虑，Google 开发者对于视频帧大小的控制到 4K。

解决的方法非常简单，找到 media/gpu/android/media_codec_video_decoder.cc 文件，这里面定义了各种支持的视频编解码配置，除了编码的 profile 之外，还有size 限制，比如 HEVC 的代码为：

```
#if BUILDFLAG(ENABLE_PLATFORM_HEVC)
  if (base::FeatureList::IsEnabled(kPlatformHEVCDecoderSupport)) {
    supported_configs.emplace_back(HEVCPROFILE_MIN, HEVCPROFILE_MAX,
                                   gfx::Size(0, 0), gfx::Size(3840, 2160),
                                   true,    // allow_encrypted
                                   false);  // require_encrypted
  }
#endif
```

第一个 size 参数是最小尺寸，第二个 size 参数是 最大尺寸，可以看到目前的限制是 4K（3840 x 2160），将这个参数修改为 gfx::Size(7680, 4320) 即可。

经过这样修改后，就可以在我们的盒子上播放 HEVC 编码的 8K 视频了。

#### 三

在谈及视频是，经常还会谈到码流大小。视频的码流大小取决于多种因素，如帧率、色深、编码格式、压缩率等。

 一般来说，HEVC 编码的 8K 视频码流大小约为 50~100 Mbps，AV1编码的 8K 视频码流大小约为 30~60 Mbps，而 AVS3 编码的 8K 视频码流大小约为 20~40 Mbps。这么高的码率因为着如果在线播放需要很高的带宽。在测试中，我找了一个 120 Mbps 码率的 HEVC 编码视频，需要在千兆以太网下才不会卡顿，而在百兆网环境下出现卡顿、画面闪烁。

 如何在有限的带宽下，在画质和流畅度之间取得平衡，是一件非常无奈又必须做的事情。此外 Chromium 代码本身也有优化的地方，比如增加缓冲区，减少播放过程中的卡顿，但这也会导致起播慢的问题，所以如何权衡也是需要在实践中检验。

#### 小结

要在 Chromium 中增加 8K 视频的支持，需要多方面的支持。首先取决于媒体框架的解码能力，其次，需要解除 Chromium 中的限制，并增加编码格式的支持，最后，还需要有一个好的网络环境，才能流畅的播放一部 8K 视频。


