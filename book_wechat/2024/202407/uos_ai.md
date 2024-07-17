# 国产系统上的 Copilot 初体验

2023 年，微软发布了 Windows Copilot，到了 2024 年更进一步，将 Copilot 植入到 Windows 11 系统中，供用户免费使用，震动了业界。

非常遗憾的是，Windows Copilot 不对中国区用户开放，中国用户无法体验到 Windows Copilot。

在上一篇文章《[使用国产操作系统作为开发系统](https://mp.weixin.qq.com/s/pvwNQHaguXs2X1XnTDE7QA)》中讲到我使用国产操作系统 Deepin 作为开发系统。安装后发现，在系统栏多了一个 UOS AI：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_ai_01.png)

点开一看，这不就是 UOS 版的 Copilot 吗？

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_ai_02.png)

微软财大气粗，入股 Open AI，Windows Copilot 接入 ChatGPT-4，供用户免费使用。相较而言，UOS AI 要麻烦一点，需要自己配置 AI 账号。

对于个人用户而言，国产大模型基本上都提供了免费的用量，日常使用没有什么问题。所以接下来，先去申请大模型 API token。

国内提供大模型服务的厂家很多，比如讯飞星火、百度千帆、豆包、Kimi 等等，我基本上都试用过。综合下来，我更喜欢使用 Kimi。Kimi 产品背后的公司叫做月之暗面，进入 MoonShot AI 开放平台：

> https://platform.moonshot.cn/console/api-keys

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_ai_03.png)

创建一个 API Key，需要注意，**这个 API Key 只会在新建后显示一次**。记住这个 Key，在后面的步骤中需要用到。

接下来在 UOS AI 中添加 AI 账号。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_ai_05.png)

UOS AI 的模型类型内置有百度千帆、讯飞星火、360 智脑以及智谱，并没有 MoonShot，选择**自定义**就可以。

API Key 填入上一步骤申请的 MoonShot AI 开放平台 API Key。模型名填入 moonshot-v1-8k，请求地址填入：https://api.moonshot.cn/v1。

模型名和请求地址是怎么来的呢？这个需要看各厂家开放平台的文档，比如月之暗面开放平台的文档：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_ai_04.png)

可以看到请求地址和模型名，由于 UOS AI 会自动在 url 后加上 chat/completions，所以在填入请求地址时，不要带上 url 后面的 chat/completions。

配置完成后，就可以和 UOS AI 对话了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_ai_06.png)

看到这里，你可能会觉得，这和 ChatGPT、Kimi 助手、豆包等应用没啥两样啊。

的确，现在大模型领域很卷，现在 AI 助手基本上都是
