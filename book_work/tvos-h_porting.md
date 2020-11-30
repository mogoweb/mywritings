### 

依赖app完成的功能的OCN扩展：

1. VideoPhone标签（VideoPhone app处理）
2. VOD标签（LTVodPlayer处理）
3. 报错处理（browser app处理）
4. a标签媒体播放（传到browser app）

技术难点：

1. 在现有weblink框架下，如果点播不走Overlay，该如何实现网页和视频的合成？
2. 有些处理是在onPause、onResume中处理，在TVOS-H还存在这种场景吗？
3. NGB-H键值转换是在与android相关的代码中处理，在TVOS-H中还需要研究