DeepSeek R1 简单访问指南



部署和可访问性

开源和许可：DeepSeek-R1 及其变体在 MIT 许可下发布，促进开源协作和商业使用，包括模型提炼。这一举措对于促进创新和降低 AI 模型开发的进入门槛至关重要。

模型格式：
两种模型及其提炼版本均采用 GGML、GGUF、GPTQ 和 HF 等格式，从而可以灵活地在本地部署它们。
1. 通过 DeepSeek 聊天平台进行 Web 访问：
DeepSeek 聊天平台提供用户友好的界面，无需任何设置即可与 DeepSeek-R1 进行交互。

访问步骤：
导航到 DeepSeek 聊天平台
注册一个帐户，或者如果您已经有一个帐户，请登录。
登录后，选择“深度思考”模式，体验 DeepSeek-R1 的逐步推理能力。

2. 通过 DeepSeek API 访问：
对于编程访问，DeepSeek 提供与 OpenAI 格式兼容的 API，允许集成到各种应用程序中。

使用 API 的步骤：

a. 获取 API 密钥：

访问 DeepSeek API 平台以创建帐户并生成您的唯一 API 密钥。

b. 配置您的环境：

将 base_url 设置为 https://api.deepseek.com/v1。

使用您的 API 密钥进行身份验证，通常通过 HTTP 标头中的 Bearer Token 进行身份验证。

c. 进行 API 调用：

利用 API 发送提示并接收来自 DeepSeek-R1 的响应。

DeepSeek API 文档中提供了详细的文档和示例。

