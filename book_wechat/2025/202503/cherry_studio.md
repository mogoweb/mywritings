# 使用 CherryStudio 搭建浏览器知识库

上个周末去参加了一个 WHLUG 的线下活动，了解了 CherryStudio 这个 AI 工具。虽然在 UOS 系统中也有 UOS AI 助手，但作为技术从业者，别的产品也该体验一下，看看别人的产品有什么值得借鉴的地方。

前段时间 CherryStudio 发布了一个里程碑版本 1.0，这标志着 CherryStudio 开始稳定成熟。CherryStudio 是一个开源的跨平台版本，在 Linux 下是 AppImg 格式，对于普通用户来说使用上稍有不便。UOS 应用商店第一时间上架了 CherryStudio，从安装到使用都简便了许多。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cherry_studio_01.png)

打开 CherryStudio，首先是一个聊天对话界面，默认使用硅基流动的服务。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cherry_studio_02.png)

如果第一时间去对话，会提示**身份验证失败，请检查 API 密钥是否正确**。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cherry_studio_03.png)

CherryStudio 只是一个 Chat 客户端，无法像腾讯元宝、DeepSeek 那样提供免费的接入服务，需要用户自行对接大模型服务。在《[UOS AI 接入 DeepSeek R1 模型](https://mp.weixin.qq.com/s/HtCUHedWw_JjfPcrYoIJ8w)》这篇文章中有介绍过如何申请硅基流动的 DeepSeek R1 模型的 API Key，这里不赘述。CherryStudio 提供了几乎所有大模型的接入方式，大家可以根据需要自行选择。当然最务实的策略就是去各个大模型官网申请 API Key，这些服务商一般都提供了免费的 tokens 额度。都配置上来，哪个好用就切换到哪一个，至少在完整体验 AI 之前是可以白嫖的。

将 API Key 填入 CherryStudio：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cherry_studio_04.png)

填入后，最后做一下检查。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cherry_studio_05.png)

如果没有问题，就可以开始和 CherryStudio 对话了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cherry_studio_06.png)

接下来是搭建浏览器知识库。

在 CherryStudio 中创建一个知识库。然后输入名称和嵌入模型。注意这里的嵌入模型并不是 DeepSeek R1 之类的大模型，而是用于 RAG 的模型。该模型并非直接生成内容的LLM，而是承担文档向量化的RAG（检索增强生成）核心组件。通过将非结构化文档转化为高维向量，构建起语义检索的数学基础，这种分层架构有效平衡了计算效率与知识密度。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cherry_studio_07.png)

然后就是添加文档，也可以直接添加目录以及网页。我将项目文档以及 Chromium Docs（markdown 格式）加入其中。由于 Chromium Docs 中的文档比较多，所以解析还花了一点时间。可以将鼠标移到图标上查看进度。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cherry_studio_08.png)

在 CherryStudio 解析文档的过程中，感觉有些卡顿，不知道是不是 RAG 模型需要使用本地算力。在文档解析完成后，在对话框中可以选择知识库。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cherry_studio_09.png)

然后我可以针对项目具体的问题向 AI 提问。比如：

> UOS 浏览器国密需求是从哪个版本开始 支持的？

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cherry_studio_10.png)

不仅给出了答案，还给出了具体的文档位置。

对于个人而言，CherryStudio 的价值不仅在于知识沉淀，更在于创造了人机协同的新型知识迭代模式。随着大模型技术的持续演进，未来或可期待实时知识更新、多源数据融合等进阶功能的实现，真正打造出自我进化的智能知识生态系统。

CherryStudio 还只是一个个人知识库的工具，不知道未来是否支持团队协作。如果能够只是团队协作，使用场景会更加丰富。大家使用怎样的工具搭建自己的知识库呢？欢迎留言讨论。
