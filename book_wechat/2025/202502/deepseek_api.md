# UOS AI 接入 DeepSeek R1 模型

中国有句古话：“树大招风。”这句话用在 DeepSeek 身上再合适不过。自 DeepSeek R1 模型爆火以来，大量用户涌入，导致服务器不堪重负，甚至一度出现宕机现象。此外，DeepSeek 线上服务还遭受了大规模恶意攻击，至今情况仍未完全缓解。例如，访问 DeepSeek 的 API Platform 时，会看到如下提示：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_01.png)

此外，DeepSeek 官方 Chat 应用的联网搜索功能无法启用，深度思考模式也频频遇到无法访问的情况。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_02.png)

面对这一困境，国内模型云服务提供商 **硅基流动** 联合 **华为云**，推出了基于 **昇腾云** 的 DeepSeek R1 & V3 推理服务，为用户提供新的访问渠道，同时也能在一定程度上缓解 DeepSeek 服务器的压力。目前，新用户注册还可获赠 **2000 万 tokens** 的免费额度。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_03.png)

## 注册硅基流动账号  

访问 硅基流动官网：https://siliconflow.cn/zh-cn/，点击右上角的 **Login** 按钮，使用手机账号注册/登录。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_04.png)

注册/登录完毕之后，进入 **模型广场**，这里提供了丰富的模型选择。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_05.png)

我们点击 DeepSeek R1 模型。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_06.png)

可以先尝试 **在线体验** 该模型的对话能力。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_07.png)

与 DeepSeek 官方 Chat 应用相比，硅基流动的界面额外提供了 **Temperature**、**Top-P**、**Top-K** 等参数调节选项，先暂时忽略，在对话框中进行对话。

## 在 UOS AI 中接入 DeepSeek R1  

硅基流动目前没有提供独立的 App，使用上略显不便。不过，我们可以通过 **API 调用**，结合 **UOS AI 应用** 来实现类似的功能。

### 获取 API 密钥  

前往 **API 管理页面**，创建 API Key。 

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_08.png)

复制该 API Key，稍后在 UOS AI 配置时需要用到。

### 配置 UOS AI 

1. 打开 **UOS AI** 应用，进入 **设置** 界面，添加模型。  

   ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_09.png)  

2. **模型类型** 选择 **自定义**，粘贴 **API 密钥**。  
3. **模型名** 输入：`deepseek-ai/DeepSeek-R1`。  
4. **请求地址** 输入：`https://api.siliconflow.cn/v1/chat/completions`。  
5. 点击 **确定**，完成模型接入。

## 开始对话  

回到 **对话界面**，模型选择上一步配置的模型，现在可以使用 DeepSeek R1 进行交流了！

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202502/images/deepseek_api_10.png)

需要注意的是，由于当前适配问题，**深度思考的内容与对话内容尚未分开**，但这并不影响正常使用。相信 **UOS AI 团队** 会在后续更新中尽快优化体验。  

通过以上方法，UOS AI 便可成功接入 **DeepSeek R1 模型**，提供更稳定的 AI 交互体验。
