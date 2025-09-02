# 统信Windows应用兼容引擎上新，完美兼容企业微信5.0和PhotoShop 2021

2025年8月19日，腾讯发布了企业微信的新版本5.0，这次重磅更新主打 AI 提效。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_01.png)

作为企业微信的重度用户，我在日常办公中高度依赖企业微信，因此第一时间下载了新版本。

UOS应用商店的企业微信并未第一时间更新，但我可以通过统信Windows兼容引擎来运行新版本。不过，使用当时的兼容引擎版本（V3.3.1）安装运行企业微信5.0时，发现存在一些兼容问题。虽然基础的聊天功能以及5.0推出的AI搜索、AI总结功能均可正常使用，但会议功能无法正常工作，邮件功能和文档功能也存在问题，只能看到列表，无法预览邮件详情和文档详情。

作为重点支持的办公软件，deepin-wine团队迅速响应，第一时间进行了适配优化，开发了V3.3.2版本。经过充分验证，该版本可正常使用会议、邮件、文档等功能，目前已发布到应用商店，deepin/UOS用户可前往应用商店更新。这次更新不仅完美兼容企业微信5.0，还带来了多项功能优化：

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

企业微信的会议功能提供了共享屏幕、等候室、互动批注、美颜、虚拟背景等特性，并深度整合企业通讯录，带来良好的集成体验和便捷性。这项功能在公司内部的使用频率正逐渐提升。兼容引擎成功解决了企业微信5.0会议功能的兼容性问题，让我们能够第一时间用上新版本。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_02.png)

虽然如今邮件的使用频率有所降低，但在企业内部，邮件功能仍不可或缺。在比较正式的场合，如内部员工沟通和与外部客户、合作伙伴的沟通，通常仍需借助邮件完成。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_03.png)

我个人较少使用企业微信中的文档功能，因为我们公司内部已有统信畅写系统。但对于很多用户来说，企业微信的文档功能深度整合于聊天、通讯录和组织架构中，实现了便捷创作、高效协作与安全管控的平衡。它非常适合作为团队日常办公的核心文档协作工具，无论是内部沟通还是与外部客户、合作伙伴的有限共享，都能提供良好支持。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_04.png)

## PhotoShop 2021

PhotoShop作为设计师必备的软件之一，一直未推出Linux原生版，这也成为用户迁移到国产系统的一大障碍。Wine社区也曾对Photoshop进行过兼容，但仅限于较旧的CS6版本，已完全无法满足当前需求。

deepin-wine团队经过持续的适配探索，最终完成了对Photoshop 2021的wine适配。

请注意，统信Windows应用兼容引擎并不提供Photoshop安装文件下载，请大家通过正规渠道购买正版软件，也可以去官网下载试用版本，然后使用最新版的兼容引擎进行安装。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_05.png)

就连Photoshop的安装程序，团队也花费了极大精力才解决兼容问题：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_06.png)

成功进入Photoshop后，就可以开始创作了：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/uos_wine_engine_07.png)

随着兼容引擎的不断成熟，更多热门软件将持续适配并上架。欢迎大家使用兼容引擎进行尝试。如果在使用中遇到问题，可通过兼容引擎直接反馈，也可通过菜单内的“社区”入口进入论坛，在论坛中反馈问题、寻求帮助，或分享你成功运行的应用！
