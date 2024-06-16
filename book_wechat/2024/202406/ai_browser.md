# AI 大模型的风，吹到了浏览器

Chrome 浏览器是谷歌最重要的产品之一。在互联网时代，掌握着流量的入口，帮助谷歌建立起了互联网霸主的地位。Chrome 浏览器不仅在市场占有率方面遥遥领先，还成为了许多用户接入互联网的首选工具。凭借其速度、安全性和丰富的扩展功能，Chrome 浏览器在全球范围内积累了庞大的用户基础。

进入 AI 时代，谷歌也在不断强化其重要性，致力于将前沿技术融入到浏览器中。谷歌不遗余力地开发了 TensorFlow.js，这是一个可以在浏览器中运行的机器学习框架，使得前端开发者能够在客户端实现复杂的 AI 算法。此外，谷歌还推出了 WebGPU，这项技术大大提升了浏览器在图形和计算处理方面的性能，展示了前端技术的无限潜力。

与此同时，以 ChatGPT 为代表的 GPT 大模型近年来风光无限，推动了人工智能的迅猛发展。OpenAI，这家成立不到十年的公司，凭借其创新和技术实力，迅速成为 AI 领域的领导者。相比之下，昔日的互联网霸主谷歌在这一波浪潮中显得光芒暗淡了一些。

然而，谷歌并未在大模型领域停滞不前。相反，它积极推出了 Gemini 系列模型，力图在这一领域重新夺回优势。Gemini 系列模型是谷歌在大规模语言模型方面的重要布局，显示了谷歌在 AI 研究和应用方面的雄心壮志。

如今，谷歌将大模型的能力进一步扩展到了浏览器中，通过将 AI 集成到 Chrome 浏览器，谷歌希望为用户提供更加智能化和个性化的上网体验。无论是搜索、信息获取还是在线服务，AI 技术的融入都将显著提升浏览器的功能和用户体验。

在大多数人的印象中，AI 模型庞大且需要大量计算资源，通常依赖云端服务器来运行。传统的客户端/服务器开发模式简化了应用开发，但带来了高昂的部署成本，并存在数据安全和用户隐私问题。如果能够充分利用用户主机的计算资源，不仅能发挥当前 AI PC 的性能，还能降低成本，并缓解数据安全和隐私问题。

由于 Web 开发的特殊性，通常无法直接访问计算机硬件资源，只能通过插件或服务转接来实现。那么，有没有更好的方法呢？

为了解决这一问题，Chrome 浏览器在最新的 Canary 版本（开发版本，本文发布时的最新版本为 127.0.6526.0）中，首次将 AI 大模型直接内置到浏览器中，实现了 Web AI 的触手可及。这一创新使得 AI 应用不再局限于云端，用户可以在本地直接体验 AI 的强大功能。

下面就带大家体验一下 Chrome 内置大模型。（**友情提醒：需要科学上网**）

1. 下载 Chrome Canary 版本，可以通过访问如下链接下载 Windows 64 位版本：

> https://www.google.com/chrome/canary/?platform=win64

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/ai_browser_01.png)

2. 开启相关功能。目前 Web AI 还是实验性功能，默认没有开启。在 Chrome 地址栏输入 chrome://flags/，找到如下两个选项：

* Prompt API for Gemini Nano
* Enables optimization guide on device

将其值都修改为 Enable。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/ai_browser_02.png)

3. 重启 Chrome 浏览器，使设置生效。
4. 下载模型。在 Chrome 地址栏输入 chrome://components/，找到选项：

* Optimization Guide On Device Model

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/ai_browser_03.png)

如果后面显示的版本号为 0.0.0.0，表示模型还未下载下来，可以点击一下下方的**检查是否有更新**按钮，手动启动下载。

然后耐心等待，这个模型下载有点慢。下载完毕后，会显示出版本号。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/ai_browser_04.png)

模型下载完毕后，重启一下 Chrome 浏览器

5. 验证模型能否工作。打开开发者工具，切到控制台，输入以下代码：

```javascript
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

如果一切顺利，你将看到控制台输出一个笑话：

> Why did the scarecrow cross the road?
>
> To get to the other side!

这代表成功调用了 Chrome 内置的 Gemini Nano 模型。

今天是端午节，来一段端午节祝福吧！

```javascript
(async () => {
  try {
    if (!window.ai || (await window.ai.canCreateTextSession()) !== "readily") {
      throw new Error("Gemini Nano not ready or not supported.");
    }

    const startTime = performance.now();
    let charCount = 0;

    const session = await window.ai.createTextSession();
    const stream = session.promptStreaming("撰写一段简短的端午节祝福，100字以内。");
    let message = "";
    for await (const chunk of stream) {
      message = chunk;
      charCount = chunk.length;
    }

    const endTime = performance.now();
    const timeElapsed = (endTime - startTime) / 1000;

    console.log(message);

    session.destroy();
  } catch (error) {
    console.error(error);
  }
})();
```

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/ai_browser_05.png)

## 写在最后

Chrome 浏览器内置的 AI 大模型名为 Gemini Nano，它是 Google 最新推出的 Gemini 系列中最轻量级的版本，专为在设备端高效运行而设计。这意味着它无法像 ChatGPT 4.0 或 Gemini Pro 那样提供强大的功能。同时，由于 Chrome 浏览器运行的设备各不相同，能否启用这一功能还取决于硬件配置的多样性，这也是 Chrome 浏览器未默认开启该功能的原因。此外，Web AI API 何时能成为标准也需要一个过程。

Web AI 为我们描绘了一个美好的愿景，但离真正实用还有漫长而艰难的道路。
