# Chromium HTML Video 媒体播放代码梳理

经过一番探索（参见[Android 10 WebView 踩坑实录](https://mp.weixin.qq.com/s/jixf2zz-VnLt7oXX8SJU-A)），终于搞定 Chromium WebView 的代码下载和编译问题，加下来就要向 H265 8K 高清播放发起冲锋。

不过在打开 Chromium 源码后，眼前一黑。这还是熟悉的 Chromium 代码吗？虽然我尽量选择了不那么新的代码，但我还是低估了谷歌工程师的努力程度，至少在 HTML Video 方面，代码结构已经改得面目全非。没办法，只能慢慢啃。

在开始梳理代码之前，先上一张图镇楼。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/chromium_media_overview_01.png)

图中，红线框起来的内容是我需要着重关注的。

Chromium HTML Video 相关的代码主要分布在三处。

首先是 third_party/blink ，这个是从之前的 WebKit 演进而来，自从和 WebKit 分道扬镳后，谷歌将新浏览器引擎命名为 blink。经过魔改，现在的代码结构和原来差异已经很大。浏览器的排版和渲染相当复杂，所以 blink 的代码及其庞杂。我们需要重点关注以下代码：

* renderer/core/html/media : 其中最重要的两个类 HTMLVideoElement 和 HTMLMediaElement，对应的是网页中的 video 元素。这里是研究 HTML Video 播放的起点。
* public/platform : 这里面主要定义了一些需要外部实现的接口，比如 WebMediaPlayer。blink 引擎专注于网页的排版和渲染，其它的工作都是 delegate 到外部（主要是 content 模块）完成。

其次是 content/ , content 模块主要处理 Browser 进程和 Renderer 进程之间的交互、层的合成。具体的媒体处理并不在这，Content 只是作为桥梁。这里面的代码虽然非常复杂，但通常不需要修改，只要从总体上把握代码的走向即可。

最后，最重要的模块登场，代码位于 media/ 下，里面还有很多子目录，主要包含与媒体捕获和播放相关的组件集合。以前的实现相对比较简单，将 URI 丢给 Android 的 MediaPlayer 处理。而现在则要处理流的解析、demux、渲染、解码、音视频同步，等等。由于 chromium 支持的平台很多，功能很多，有一些代码是和视频捕捉、cast、加密流等有关，所以下面只列出一些与 HTML Video 播放相关的代码目录：

* audio/ - 音频输入和输出代码。 包括特定于平台的输出和输入实现。

* base/ - meidia 基础类，包含 media 用到的各种枚举、实用程序类和基础类型； 比如 AudioBus、AudioCodec 和 VideoFrame，等等。

* blink/ - 用于与  blink 中的 <video> 和 <audio> 对接。 仅在与 Blink 相同的进程中使用； 通常是渲染进程。

* ffmpeg/ - ffmpeg 是一个非常流行的媒体框架库，代码位于 //third_party/ffmpeg，这里提供封装和辅助方法，这样别的模块不用直接调用 ffmpeg 的接口，也有利于切换到其他的媒体框架库。

* filters/ - 包含用于媒体播放的数据源、解码器、多路分解器、解析器和渲染算法。

* formats/ - 各种媒体格式解析器。

* gpu/ - 包含平台硬件编码器和解码器实现。

* renderers/ - 将音频和视频渲染到输出接收器的代码。

* test/ - 用于测试媒体播放管道的代码和数据。

* tools/ - 独立的媒体测试工具。

* video/ - 抽象硬件视频解码器接口和工具。

一次典型的 HTML Video 播放过程如下：

1. 从 third_party/blink/ 中的 blink::HTMLMediaElement 开始，经过 content::MediaFactory 短暂跳转后到达 media::WebMediaPlayerImpl， media::WebMediaPlayerImpl 实现了 third_party/blink/public/platform/media/ 中定义的 blink:: WebMediaPlayer。 每个 blink::HTMLMediaElement 都拥有一个 media::WebMediaPlayerImpl 来处理诸如播放、暂停、搜索和音量变化（以及其他事情）之类的事情。

2. media::WebMediaPlayerImpl 处理或委托网络上的媒体加载以及多路分解器和管道初始化。 media::WebMediaPlayerImpl 拥有一个 media::PipelineController，它在播放期间管理 media::DataSource、media::Demuxer 和 media::Renderer 之间的协作。

3. 在正常播放期间，WebMediaPlayerImpl 拥有的 media::Demuxer 可能是 media::FFmpegDemuxer 或 media::ChunkDemuxer。 ffmpeg 变体用于标准 src= 播放，其中 WebMediaPlayerImpl 负责通过网络加载字节。 media::ChunkDemuxer 与媒体源扩展 (MSE) 一起使用，其中 JavaScript 代码提供多路复用字节。

3. media::Renderer 通常由 media::RendererImpl 实现，它拥有并协调 media::AudioRenderer 和 media::VideoRenderer 实例。 其中每一个依次拥有一组 media::AudioDecoder 和 media::VideoDecoder 实现。 每个向由 media::Demuxer 公开的 media::DemuxerStream 发出异步读取，media::DemuxerStream 由 media::DecoderStream 路由到正确的解码器。 解码也是异步的，因此解码后的帧会在稍后的某个时间传送到每个渲染器。

4. media/ 库包含所有支持的 Chromium 平台的 media/gpu 中的硬件解码器实现，以及 FFmpeg 和 libvpx 支持的 media/filters 中的软件解码实现。 按照 media::RendererFactory 提供的顺序尝试解码器； 第一个报告成功的将用于播放（通常是视频的硬件解码器）。

5. 每个渲染器分别通过事件驱动的 media::AudioRendererSink 和 media::VideoRendererSink 接口管理音频和视频的计时和渲染。 这些接口都接受回调，当需要新的音频或视频帧时，它们将定期发出回调。

6. 在音频方面，同样在正常情况下，media::AudioRendererSink 是通过浏览器进程拥有的 base::SyncSocket 和共享内存段驱动的。 该套接字由媒体/音频中的 media::AudioOutputStream 的平台级实现定期触发。

7. 在视频方面，media::VideoRendererSink 由合成器向 media::VideoFrameCompositor 发出的异步回调驱动。 media::VideoRenderer 将通过 media::TimeSource 与 media::AudioRenderer 对话，以协调音频和视频同步。

至此，整个播放流程梳理完毕，但其中还牵涉到很多细节。这需要一点一点，耐心阅读源码。更多的时候，遇到问题，找到问题相关的源码，解决问题。

接下来，我将要向 H265 8K 视频解码发起冲锋，敬请关注！