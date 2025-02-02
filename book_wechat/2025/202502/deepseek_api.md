# UOS AI 接入 DeepSeek R1 模型

中国有句古话：树大招风，这句话放在 DeepSeek 上是再合适不过了。DeepSeek R1 模型爆火之后，大量用户涌入，导致 DeepSeek 的服务器不堪重负，甚至出现了宕机的情况。DeepSeek 线上服务受到大规模恶意攻击。直到今天，这种状况依然没见好转，比如访问 DeepSeek 的 API Platform，出现如下提示：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_01.png)

DeepSeek 的官方 Chat 应用，联网搜索无法启用，深度思考模式也是经常出现无法访问的情况。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_02.png)

面对这种情况，国内模型云服务提供商硅基流动联合华为云推出基于昇腾云的 DeepSeek R1 & V3 推理服务，为用户提供新的选择，也可以部分缓解 DeepSeek 的压力。目前新用户注册，还可以获赠 2000 万 tokens 的免费额度。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_03.png)

访问硅基流动的官网: https://siliconflow.cn/zh-cn/，点击右上角的 Login 按钮，使用手机账号即可注册/登录。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_04.png)

注册/登录完毕之后，进入模型广场，里面的模型非常多。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_05.png)

我们点击 DeepSeek R1 模型。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_06.png)

可以先点击在线体验按钮体验一下。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_07.png)

相比 DeepSeek 的官方 Chat 应用，界面上多了 Temperature、Top-P、Top-K 之类的参数，先暂时忽略，在对话框中进行对话。

硅基流动没有提供 App，使用起来不甚方便，不过可以通过 API 调用，通过 UOS AI 应用实现类似的效果。

首先创建 API 密钥。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_08.png)

复制这个 API 密钥，稍后需要用到。

打开 UOS AI，进入设置界面，添加模型

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_09.png)

模型类型选择自定义，然后粘贴 API 密钥。模型名输入 deepseek-ai/DeepSeek-R1，请求地址输入 https://api.siliconflow.cn/v1/chat/completions。最后点击确定，模型就添加好了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_10.png)

回到对话界面，开启对话。

由于应用适配的问题，深度思考的内容没有和对话内容分开，不过不影响使用。相信UOS AI 团队会很快修复这个问题。