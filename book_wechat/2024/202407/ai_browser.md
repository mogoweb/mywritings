# 浏览器中集成 AI 的三种方式

进入移动互联网时代，浏览器的重要性有所下降，其地位被几款超级 App 取代。然而，在 PC 端，浏览器依然是最重要的应用，仍是人们踏入互联网世界的关键入口。

到了 AI 时代，几乎所有的应用程序都开始拥抱 AI，纷纷借其之力实现蜕变与升级。浏览器自然也不会置身事外，同样积极地融入这股 AI 的洪流之中。

比如这样一种场景，你在浏览网页时，有一个新的名词不太理解，或者有个知识点想进一步了解，以往的做法是打开搜索引擎，输入关键字，查找相关的资料。但搜索引擎并不知道你现在搜索的语境，也不会根据你现在浏览的内容做一个个性化的推荐。如果在浏览器中集成了 AI，AI 可以根据你现在浏览的网页，作为上下文，提供更为准确的信息。

个性化内容推荐，App 在做，搜索引擎在做，但基本上都是根据浏览历史，根据大数据来做。而浏览器中集成 AI，就可以更进一步，根据正在浏览的内容来做个性化内容推荐。

目前浏览器中的 AI 功能集成还处于起步阶段，还有哪些新的玩法，有待探索，也有待市场检验。哪些是真实需求，哪些是产品经理臆想出来的伪需求，用户说了算。

浏览器中集成 AI，大致有如下几种方式：

第一种，像 Google 这样的高阶玩家，直接从标准入手，提供框架，提供在浏览器中进行 AI 开发的基础设施。

就拿 WebGPU 来说吧，其实在这之前，有一个 WebGL 的标准，可以利用 GPU 的并行计算能力，显著加速某些 AI 算法的执行，例如矩阵运算和卷积操作等，提升模型的训练和推理速度。但 Google 仍然不满足，希望提供比 WebGL 更高效的性能和更丰富的功能，特别是对于计算密集型任务，如机器学习和 AI，这就是 WebGPU。

相较于 WebGL, WebGPU 提供了更低级别的控制，允许开发者更精细地管理内存和计算资源，优化 AI 算法的性能。支持更复杂的计算任务和更大的数据集处理，使其成为 AI 开发的理想选择，尤其是在深度学习和大规模并行计算方面。

对于普通开发者而言，WebGL 和 WebGPU 太过于底层，也不太可能直接基于 WebGPU 开发应用。没有关系，你还可以选择 TensorFlow.js。

**TensorFlow.js** 是一个开源的 JavaScript 库，用于在浏览器和 Node.js 环境中进行机器学习模型的训练和部署。简单说，就是 Google 将 TensorFlow 搬到了前端，TensorFlow.js 为机器学习在 Web 应用中的普及提供了强大的支持。

对于有些开发者而言，TensorFlow.js 仍然太复杂。特别是现在大模型盛行，诸如 Open AI 这样的公司，都提供了 API，只需要几个简单的调用，就可以使用大模型。没有关系，Google 也安排上了。

之前写过一篇文章《[AI 大模型的风，吹到了浏览器](https://mp.weixin.qq.com/s/X6UtvgTDJanEnXkdI1ghZw)》，介绍了 Google 开始在 Chrome 中集成了 Gemini Nano 语言模型。虽然出于各种妥协，集成的是 Gemini 的最小版本，但这是迈向端侧推理（不依赖服务器）的重要一步。这也是 Google 下的一步大棋，将 Web AI 推进下一代 Web 标准中。

请看如下一段代码，有了 Web AI，只需要几行代码，就可以调用大模型。

```
async function testGeminiNano() {
try {
  if (!window.ai) throw new Error("AI API not supported.");
  if ((await window.ai.canCreateTextSession()) !== "readily") {
    throw new Error("Gemini Nano not ready yet.");
  }
  console.log("Gemini Nano is ready to use!");
  const session = await window.ai.createTextSession();
  const stream = session.promptStreaming("Tell me a jok!");
  for await (const chunk of stream) {
    console.log(chunk);
  }
  session.destroy();
} catch (err) {
  console.error(err.message);
}
}

testGeminiNano();
```

像 Google 这样从底层入手，从底向上打通路径，不是一般公司能做到。所以浏览器厂商也想到了自己的解决方法，这就是第二种方法，在浏览器应用层面集成 AI，这其中做得比较好的就是 Brave 浏览器。

**Brave 浏览器** 是一个以隐私和安全为核心设计的 Web 浏览器，在最新的 nightly build 中发现在侧边栏增加了一个名为 Leo 的 AI 助手。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/ai_browser_01.png)

点开就可以也 AI 对话。 Leo 支持多种大模型，也支持添加自定义大模型：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/ai_browser_02.png)

和网页深度集成：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/ai_browser_03.png)

第二种方法简单易行，浏览器引擎不需要自己开发，可以基于 Chromium 定制，然后在应用层面集成 AI。集成 AI 大模型也没有使用浏览器本身的能力，而是通过 API，调用大模型。

第二种方法虽然简单易行，但至少得有自己的浏览器产品，虽然说基于 Chromium 定制一款 Web 浏览器也不是一个难事，但如何说服用户使用你的 Web 浏览器则难上加难。如果没有特别支持，用户为什么不选择 Chrome 或者 Edge 浏览器？

面对这种困境，有人给出了第三种方案，那就是提供浏览器插件，由插件来提供 AI 能力。

我最近发现有一款浏览器插件做得不错，浏览器插件的名称一点也不炫酷，叫做：Monica，但功能确实做得不错。

Chorme 浏览器安装 Monica 插件后，会多一个图标，点开后出现 AI 对话框：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/ai_browser_04.png)

从界面上就可以看出，功能相当齐全，聊天、搜索、写作、画图、翻译，应有尽有。

再看看与网页的集成特性。浏览网页时，如果选择了网页上的文字，或者网页中有文字内容，就会出现一个 Monica 助手的图标，可以对网页做总结、翻译、做笔记等等。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/ai_browser_05.png)

还可以进行重写、扩写等等。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/ai_browser_06.png)

Monica 后端接入的是 ChatGPT-4o、Claude 3.5、Gemini 1.5 几种模型，未来会接入更多的大模型，但不支持自定义接入大模型。Monica 提供免费使用，每天只有 40 个查询量，如果觉得不错，可以订阅套餐，提供了 Pro、Pro+ 和 Unlimited 三种套餐。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/ai_browser_07.png)

通过在浏览器中集成AI技术，可以显著提升用户体验、增强功能和提高效率。智能搜索和推荐、自动化任务执行、增强安全性、提高网页加载速度以及先进的语音和图像识别功能，都是AI赋能浏览器的重要体现。未来，随着AI技术的不断发展和成熟，浏览器将变得更加智能和强大，为用户提供更加优质和高效的互联网体验。
