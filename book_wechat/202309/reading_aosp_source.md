# 开始撸 Android 源码

启动找工作模式，发现无比困难。搁在往日，大龄程序员找工作都是一件困难的事情，加上今年形势很差，更是难上加难。关键是我这十几年来主攻的浏览器内核方向，需求量更是几乎为零。在 BOSS 直聘上以 Chromium 为关键词，搜到如下两条结果：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202309/images/reading_aosp_source_01.png)

这都让我有点大喜过望，不过仔细一看，第二条工作地点在南昌，只剩下独苗。抱着唯一的希望，尝试和招聘官试着沟通，结果发送了几天还是**未读**状态。实在忍不住，直接去他们办公室去探个究竟。办公室离我之前上班的地方就隔几栋楼，算是轻车熟路。上去之后，前台小姐姐很 nice，让我和里面的技术人员聊了聊，总体来说还是比较匹配的，问题是他们近期没有 HC。没有办法，只能打道回府，等待近期能否放出一些 HC。

换一个招聘网站呢？这次选择了 51 Job，更让人灰心，一条招聘信息也没有：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202309/images/reading_aosp_source_05.png)

听说互联网行业拉勾网用得多，那也要试试。虽然出来了几条招聘信息，但都是前端开发工作：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202309/images/reading_aosp_source_06.png)

再换猎聘网，结果也是差不多：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202309/images/reading_aosp_source_07.png)

另外一个寻找方向就是 Android 系统开发。这些年来一直研发 COS 系统（底层基于Android，Framework 采用 C++ 改写）和 TVOS 系统（基于 Android，加入 TVOS 组件），其实也对 Android 系统有很深的了解。不过之前的 Android 基础版本比较老（4.1 和 5.1），而最近一个 8K 视频盒子使用的 Android 10，开发时间不是很长，还没来得及深入系统内部。粗粗浏览了一下源码，加入过一些 TVOS 组件，但代码相对于 5.1 还是变化很大，有必要再研究一下。

这段时间正好空下来，找工作一时半会也难以搞定，还是静下心来读读 Android 源码吧！

如果哪位朋友有浏览器开发或 Web 引擎方面的工作，也可以帮忙推荐一下，谢谢！
