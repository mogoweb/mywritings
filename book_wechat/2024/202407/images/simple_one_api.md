# AI 大模型那么多，我全要...

OpenAI 的 ChatGPT 横空出世后，传统科技巨头纷纷推出自己的语言大模型，国内互联网公司也积极参与。开源大模型的涌现让竞争愈发激烈，甚至有人戏称这是“千模大战”。虽然这个说法有些夸张，但在 AI 大模型领域，国内已经有讯飞星火、百度千桨、豆包、DeepSeek 和 Kimi 等多个竞争者。

作为软件开发人员，面对众多的大模型选择，我们陷入了“幸福的烦恼”：

**一方面**，大模型服务提供商之间竞争激烈，服务质量不断提高，价格也越来越低。

**另一方面**，面对这么多选择，我们究竟该选择哪个大模型呢？要找到一个高质量又价格合理的模型，确实很困难。

作为软件方案提供商，我们的选择是**我们全要**。

OpenAI 作为语言大模型领域的开创者，其 OpenAI API 已成为事实上的标准。后来者大多选择兼容 OpenAI API。但每家公司都有自己的特色，因此在实际应用中，我们需要使用不同的 SDK 和参数。如果仅依赖 OpenAI API 的接口是不够的。

为了支持不同的大模型并实现灵活配置，我们有两个主要解决方案：

#### 1. API 封装

在各家 API 的基础上做一层封装，为 AI 应用提供一致的接口。通过这种方式，我们可以根据配置决定最终使用哪个大模型，未来接入新模型时不需修改上层应用代码。这种方案简单易行，适用于大多数情况。

#### 2. 中转代理

架设一个中转服务器，将应用程序的请求转发给不同的大模型服务提供商。这个方法不仅包括简单的请求转发，还涉及接口转换、负载均衡等功能。相比于 API 封装，中转代理有以下优势：

- **灵活性**：增加新的大模型支持时，不会影响客户端应用。API 封装可能需要修改客户端代码，而中转代理只需调整中转服务器配置。
- **负载均衡**：可以将请求分发到不同的大模型或使用不同的 API token，保证负载均衡，避免单一 token 超出并发限制。
- **访问国外大模型**：如果国外的大模型（如 OpenAI ChatGPT）在国内无法直接访问，可以通过将中转服务器部署在国外来解决这一问题。

不过，与 API 封装相比，中转代理对中转服务器的响应速度要求更高，且实现上更复杂。

如果你有这方面的需求，可以参考一个开源项目：[simple-one-api](https://github.com/your-repo/simple-one-api)。该项目介绍如下：

> Simple-one-api 是一个开源项目，旨在兼容多种大模型接口，如千帆大模型平台、讯飞星火大模型、腾讯混元、MiniMax 和 DeepSeek 等。通过一个单独的可执行文件，配置简单，一键部署，快速集成和调用多种大模型，简化了不同平台接口的差异。

---

该项目旨在兼容多种大模型接口，并统一对外提供 OpenAI 接口。通过该项目，我们可以方便地集成和调用多种大模型，简化了不同平台接口差异带来的复杂性。

项目采用 Go 语言编写，生成一个单一的可执行程序，加上 json 配置文件，就可以工作了。步骤如下：

1. 克隆源代码。
```
$ git clone https://github.com/fruitbars/simple-one-api.git
```

2. 检查 Go 语言的版本，该项目要求 Go 1.21 以上版本。
```
$ go version
go version go1.22.5 linux/amd64
```
3. 编译源码。
```
$ ./quick_build.sh 
Building simple-one-api for linux/amd64...
go: downloading github.com/gabriel-vasile/mimetype v1.4.3
Build completed.
```
4. 启动。
```
$ ./simple-one-api 
2024/07/04 07:51:32 config.go:149: config name: /data/ai/llm/simple-one-api/config.json
2024/07/04 07:51:32 config.go:158: config json
2024/07/04 07:51:32 config.go:179: { false  {     0}  random [] map[] map[] map[hunyuan:[{[hunyuan-lite] true map[secret_id:xxx secret_key:xxx] []  map[] map[] {0 0 0 0 0} false}] minimax:[{[abab6-chat] true map[api_key:xxx group_id:1782658868262748467] [] https://api.minimax.chat/v1/text/chatcompletion_pro map[] map[] {0 0 0 0 0} false}] openai:[{[deepseek-chat] true map[api_key:xxx] [] https://api.deepseek.com/v1 map[] map[] {0 0 0 0 0} false} {[@cf/meta/llama-2-7b-chat-int8] true map[api_key:xxx] [] https://api.cloudflare.com/client/v4/accounts/xxx/ai/v1/chat/completions map[] map[] {0 0 0 0 0} false} {[glm-4 glm-3-turbo] true map[api_key:xxx] [] https://open.bigmodel.cn/api/paas/v4/chat/completions map[] map[] {0 0 0 0 0} false}] qianfan:[{[yi_34b_chat ERNIE-Speed-8K ERNIE-Speed-128K ERNIE-Lite-8K ERNIE-Lite-8K-0922 ERNIE-Tiny-8K] true map[api_key:xxx secret_key:xxx] []  map[] map[] {0 0 0 0 0} false}] xinghuo:[{[spark-lite] true map[api_key:xxx api_secret:xxx appid:xxx] []  map[] map[] {0 0 0 0 0} false}]]}
2024/07/04 07:51:32 config.go:210: read LoadBalancingStrategy ok, random
2024/07/04 07:51:32 config.go:218: read ServerPort ok, :9090
2024/07/04 07:51:32 config.go:223: log level:  
2024/07/04 07:51:32 config.go:98: Models: [spark-lite], Timeout: 0, QPS: 0, QPM: 0, RPM: 0,Concurrency: 0
2024/07/04 07:51:32 config.go:98: Models: [yi_34b_chat ERNIE-Speed-8K ERNIE-Speed-128K ERNIE-Lite-8K ERNIE-Lite-8K-0922 ERNIE-Tiny-8K], Timeout: 0, QPS: 0, QPM: 0, RPM: 0,Concurrency: 0
2024/07/04 07:51:32 config.go:98: Models: [hunyuan-lite], Timeout: 0, QPS: 0, QPM: 0, RPM: 0,Concurrency: 0
2024/07/04 07:51:32 config.go:98: Models: [deepseek-chat], Timeout: 0, QPS: 0, QPM: 0, RPM: 0,Concurrency: 0
2024/07/04 07:51:32 config.go:98: Models: [@cf/meta/llama-2-7b-chat-int8], Timeout: 0, QPS: 0, QPM: 0, RPM: 0,Concurrency: 0
2024/07/04 07:51:32 config.go:98: Models: [glm-4 glm-3-turbo], Timeout: 0, QPS: 0, QPM: 0, RPM: 0,Concurrency: 0
2024/07/04 07:51:32 config.go:98: Models: [abab6-chat], Timeout: 0, QPS: 0, QPM: 0, RPM: 0,Concurrency: 0
2024/07/04 07:51:32 config.go:230: GlobalModelRedirect:  map[]
2024/07/04 07:51:32 config.go:359: other support models: [@cf/meta/llama-2-7b-chat-int8 ERNIE-Lite-8K ERNIE-Lite-8K-0922 ERNIE-Speed-128K ERNIE-Speed-8K ERNIE-Tiny-8K abab6-chat deepseek-chat glm-3-turbo glm-4 hunyuan-lite spark-lite yi_34b_chat]
2024/07/04 07:51:32 config.go:237: SupportMultiContentModels:  [gpt-4o gpt-4-turbo glm-4v gemini-*]
2024/07/04 07:51:32 logger.go:14: level mode 
2024/07/04 07:51:32 logger.go:28: level mode default prod
2024/07/04 07:51:32 logger.go:44: log plain-text format
```

在项目源码的 samples 目录，有各种大模型配置文件的示例：

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/simple_one_api_01.png)

可以看到，支持的大模型还是非常全面的，国内外的主流大模型都支持。我们以 samples/config_xinghuo.json 文件为例：

```
{
  "debug": false,
  "load_balancing": "random",
  "services": {
    "xinghuo": [
      {
        "models": ["spark-lite"],
        "enabled": true,
        "credentials": {
          "appid": "xxx",
          "api_key": "xxx",
          "api_secret": "xxx"
        }
      }
    ]
  }
}
```
结构还是相当清晰简单，只需要填入我们在讯飞星火申请的 appid、api_key、api_secret 填入即可。

你们在项目中是如何集成大模型的？欢迎交流！

