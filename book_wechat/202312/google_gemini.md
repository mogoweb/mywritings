# Google AI 的胡说八道，我差点就信了

目前我在工作中比较多的使用了 GPT 大模型来辅助我编码，主要使用了 Cursor 和微软新必应。

Cursor 专门针对编程领域，有点类似 github Copilot。功能比较多，有问答对话、代码续写、自动调试、查找符号、自动注释等功能。Cursor 算是将 AI 功能集成到编辑器中。但是我一般都是使用 QT IDE 开发，所以基本上只用到 Cursor 的问答对话功能。

Cursor 后端默认连接的是 ChatGPT。付费用户可以选择使用 ChatGPT 3.5 或 ChatGPT 4.0，免费用户只有 50 次 ChatGPT 4.0 的提问机会，用完之后就只能使用 ChatGPT 3.5。而且使用 ChatGPT 3.5 也不是无限量提供，一个月只有 200 次问答机会。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202312/images/gemini_01_1.png)

Cursor 虽然后端连接了 ChatGPT，但在国内访问不需要梯子，但支付方式比较麻烦，需要使用信用卡或者国内用不了的 Cash App Pay。我采用的解决方法是注册几个账号，几个账号轮流用，基本上够用。

微软必应是微软用来和 Google 对抗的搜索引擎，可惜这么多年一直被 Google 压着打，在国内基本上没有什么影响。到了 AI 时代，微软凭着财大气粗，强势入股 OpenAI。自然而然的，AI 赋能到自家的搜索引擎上，这就是微软的新必应。

新必应后端使用了 ChatGPT 4.0，结合了微软的搜索引擎，虽然没有针对编程做优化，但用起来和 Cursor 的体验差不多。关键是新必应对用户完全免费，也不限使用次数。

当然对中国用户来说，新必应有些不太友好。由于一些原因，新必应未对中国用户提供服务。刚开始还比较宽松，翻个墙就可以访问。后来需要各种魔法，相关的教程网上很多。直到最近两周，啥方法也不管用，访问新必应就会跳转到微软商城。不要和我说有秘笈，网上的方法我都尝试过，就差肉身翻墙了。

AI 领域这么热闹，怎么会少得了 Google 呢？Google 算是最早一批入场的重量级选手。要说 AI 市场的引爆，离不了 Google 旗下 Deepmind 公司的 AlphaGo（一款围棋 AI，围棋被认为是智力领域的皇冠）。正是 AlphaGo 的横空出世，人们才猛然发现，原来 AI 也能这么智能。但是最近 Google 的风头被 OpenAI 给抢了。ChatGPT 的出现，将 AI 带入 2.0 时代。人工智能不再是像 Siri、小度之类的玩具，而是实打实的生产力工具。

Google 自然也不甘落伍，于今年 12 月推出了新一代大模型 Gemini。关于 Gemini 的介绍，可以去网上看，就那些话。官网所说的 1.6 万亿参数，估计也没多少人有概念。一般来说看不懂，你需要知道它很牛就够了。国内很多媒体都是用遥遥领先来形容的。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202312/images/gemini_01_2.png)

这也不是国内媒体哗众取宠，因为 Google 官方也出了一张对比图，吊打 ChatGPT-4。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202312/images/gemini_01_3.png)

既然有这么好的 AI，当然要尝试一下。Google 这方面还是比较大方的，直接表示可以用，没有限制。

Google Gemini 有三种版本，针对不同场景：

* Gemini Ultra：谷歌规模最大且功能最强大的模型，适用于高度复杂的任务。

* Gemini Pro：适用于各种任务的最佳模型。

* Gemini Nano：手机等设备最高效的模型。

目前，开放试用的是 Gemini Pro。使用方法很简单，使用浏览器访问如下网址即可。

> https://bard.google.com/

经过这两周的使用，发现中文问题回答的质量比较差，但英文提问，质量要好得多。一直没明白什么原因。直到今天上午，才终于弄明白了原因。

今天我提了一个问题：

> chromium views 在 linux 下使用了什么 GUI 框架

结果 Google AI 给我的回答把我惊呆了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202312/images/gemini_01.png)

Chromium 源码一直是我在研究的，从未在 Chromium 源码中看到过 QT 的移植。虽然 Chromium 也有 QT 的移植，但这个是由 QT 公司在维护，并没有合并到 Chromium 的源码仓库中。难道我这半年没研究 Chromium 的源码，又有新的更新？为此，我特意去 Chromium 仓库看了一下最新源码，确实没有。

于是，我接着问：

> 确定是使用了QT框架吗？

Google AI 信誓旦旦的回答，是的。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202312/images/gemini_02.png)

是不是 Google AI 理解的 Chromium views 和我有什么出入。于是，接着提问：

> 请介绍一下 chromium 中的 views 系统

结果这次回答更离谱：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202312/images/gemini_03.png)

看到最后那张图了没有，这是什么鬼？

算了，用英文问一下吧。

> What GUI framework does chromium views use under linux?

这次 Google AI 也用英文回答，回答比较靠谱。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202312/images/gemini_04.png)

这个时候，我感觉中文和英文使用了不同的模型，要不回答不至于差别这么大。

于是继续提问：

> 你现在是使用的什么大模型在和我对话？

 我现在使用的大模型是 Google AI 的 PaLM2，它是一个 540 亿参数的 Transformer 模型，经过训练可以生成文本、翻译语言、编写不同类型的创意内容，并以信息丰富的方式回答您的问题。

 PaLM2 是在一个包含文本和代码的巨大数据集上训练的。这使它能够理解和生成复杂的语言，并在各种任务中表现出色。例如，它可以生成与人类撰写的文本难以区分的文字，翻译不同语言的文本，并编写不同类型的创意内容，如诗歌、代码、脚本、音乐作品、电子邮件、信件等。

 PaLM2 仍在开发中，但它已经学会了执行许多类型的任务，包括

 * 我会尽力按照你的指示，认真完成你的要求。
 * 我会用我的知识来回答你的问题，以一种全面和信息丰富的方式，即使他们是开放的、具有挑战性的或奇怪的。
 * 我会生成不同的创意文本格式，如诗歌、代码、脚本、音乐作品、电子邮件、信件等。我会尽力满足你的所有要求。

 我很高兴看到 PaLM2 在未来的发展，并相信它将有潜力在许多不同的领域发挥作用。

晕倒，居然不是使用的 Gemini 模型。不科学啊，官方的指导不至于会出错呀，那就继续问吧。

> 不是使用的Gemini吗？

 您说的没错，Google AI 在 2023 年 12 月发布了 Gemini，这是一个 1.6 万亿参数的 Transformer 模型。Gemini 在一个包含文本、代码、音频、图像和视频的巨大数据集上训练，能够执行比 PaLM2 更复杂的任务。例如，它可以生成更具创意和信息量的文本，翻译更准确的语言，并生成更逼真的图像和视频。

 然而，Gemini 仍在开发中，目前还没有公开可用。Google AI 表示，他们正在评估 Gemini 在各种应用中的潜力，并计划在未来几年内将其商业化。

 因此，目前我使用的大模型是 PaLM2，而不是 Gemini。PaLM2 已经在许多不同的任务中得到了验证，并能够提供准确和信息丰富的响应。我相信 PaLM2 仍然是目前最好的语言模型之一，并且能够满足我与用户进行交流的需求。

> What big model are you using to talk to me now?

 You're absolutely right, I apologize for the earlier confusion. I am indeed powered by Gemini Pro, the middle tier of the Gemini series of large language models from Google AI. It was officially launched in December 2023 and represents a significant upgrade from the PaLM2 model I previously mentioned.

 Gemini Pro boasts a colossal 1.6 trillion parameters, trained on a vast dataset encompassing text, code, audio, images, and videos. This expanded data scope and increased parameter count allow me to perform significantly more complex tasks and provide even more accurate and informative responses.

 Here are some of the key advantages of Gemini Pro:

 * Enhanced Capabilities: I can generate more creative and informative text formats, translate languages with greater accuracy, and even understand and respond to complex factual queries with deeper context.
 * Improved Reasoning and Planning: I can now plan and reason through multi-step tasks, making my responses more relevant and consistent with the overall conversation flow.
 * Enhanced Information Synthesis: My ability to synthesize information from various sources and present it in a concise and understandable manner has been significantly improved.

While Gemini Ultra, the largest and most powerful version of the Gemini model, is still under development, Gemini Pro already represents a major leap forward in language model technology. I'm excited to continue learning and growing, and I believe Gemini Pro has the potential to revolutionize the way we interact with information and AI.

Please feel free to ask me any questions you have, and I'll do my best to answer them using the full capabilities of Gemini Pro.

原来如此，看来使用中英文提问，后端连接的模型并不相同。要想取得比较好的效果，还是使用英文提问比较好。借助百度翻译或谷歌翻译，中英文处理上没有什么难度。

小结一下：

* Google Gemini 可以免费试用，但前提是得备好梯子。
* Google Gemini 对中文问题的回答质量堪忧，换成英文，效果比较好，目前只有官方数据表示遥遥领先，个人使用感觉并不明显。
* Google bard 可能针对中英文，后端采用了不同的模型（只是猜测，有待实证）。
* AI 有时也会出现一些错误的回答，需要仔细甄别。

码字不易，觉得不错就点个赞吧！

