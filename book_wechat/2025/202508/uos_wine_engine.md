# 统信Windows应用兼容引擎上新，完美兼容企业微信5.0和PhotoShop 2021

2025年8月19日，腾讯发布了企业微信的新版本5.0，这次重磅更新主打 AI 提效。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_01.png)

作为企业微信的重度用户，在日常办公中都依赖企业微信，自然第一时间下载了新版本。

UOS 应用商店的企业微信并没有第一时间更新，但我可以使用统信Windows兼容引擎运行。但使用当时的兼容引擎版本（V3.3.1） 安装运行企业微信5.0，发现存在一些兼容问题。虽然基础的聊天功能以及 5. 0推出的 AI 搜索、AI总结功能都可正常可用，但是会议功能不正常，邮件功能和文档功能也存在问题，只能看到列表，邮件详情和文档详情无法预览。

作为重点支持的办公软件，deepin-wine 团队当然不会落后，在第一时间进行了适配优化，开发了 V3.3.2 版本，经过充分验证，可以正常使用会议、邮件、文档等功能，目前已经发布到应用商店，deepin / UOS 用户可以去应用商店更新。这次更新不仅完美兼容企业微信5.0，还带来了不少功能优化：

* 新增 适配企业微信5.0，支持智能AI、邮件、会议等功能
* 新增 适配 Adobe Photoshop
* 新增 我的应用界面支持拖exe文件安装
* 新增 组件下载支持断点续传
* 新增 菜单内添加社区入口方便交流和反馈问题
* 新增 适配deepin V25 ARM
* 优化 左侧应用分类列表
* 优化 高级调试界面交互优化
* 优化 完善组件库，优化自动修复逻辑
* 优化 提升各个场景响应速度
* 优化 快捷方式生成显示逻辑

## 企业微信5.0

企业微信的会议功能提供了共享屏幕、等候室、互动批注、美颜、虚拟背景，深度整合企业通讯录，提供了良好的集成体验和便捷性，这项功能在公司内部使用得越来越多。兼容引擎解决了企业微信5.0会议功能的兼容性问题，让我们第一时间用上新版本。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_02.png)

虽然邮件现在大家用得越来越少，但是在企业内部，邮件功能不可缺少，在比较正式的场合，比如内部员工沟通和与外部客户、合作伙伴的沟通，通常需要通过邮件来完成。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_03.png)

企业微信中的文档功能我个人使用得比较少，因为我们公司内部有统信畅写系统。但对于很多用户来说，企业微信的文档功能​​深度整合于聊天、通讯录和组织架构中​​，实现了​​便捷创作、高效协作与安全管控​​的平衡。它非常适合作为团队日常办公的​​核心文档协作工具​​，无论是内部沟通还是与外部客户、合作伙伴的有限共享，都能提供良好支持。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_04.png)

## PhotoShop 2021

PhotoShop 作为设计师必备的软件之一，一直都没有开发 Linux 原生版，这也是迁移到国产系统的一大障碍。Wine 社区也做了 PhotoShop 兼容，但是是古早的版本 Photoshop CS6，完全跟不上形势。

deepin-wine 团队经过持续的适配探索，终于完成了对 Photoshop 2021 的 wine 适配。

统信Windows应用兼容引擎并不提供 Photoshop 安装文件下载，请大家去正规渠道购买正版软件，然后使用最新版的兼容引擎进行安装。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_05.png)

就是这个 Photoshop 安装程序，团队都花费了极大力气，才解决兼容问题：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_06.png)

进入到 Photoshop，就可以开始创作了：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_07.png)

随着兼容引擎的成熟，更多热门软件会持续适配上架到兼容引擎，欢迎大家使用兼容引擎进行尝试。如果在使用中遇到问题，可以通过兼容引擎直接反馈问题，也可以通过兼容引擎菜单内“社区”入口直达论坛，在论坛内反馈问题寻求帮助，或者分享你wine成功的软件！