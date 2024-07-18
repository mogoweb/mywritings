# 在国产系统上部署开源大模型

在上一篇文章《[国产系统上的 Copilot 初体验](https://mp.weixin.qq.com/s/MDSss_Z1mAJTUV1bzcjB6w)》中，我写到了 UOS AI。UOS 本身并没有提供大模型接入，目前市面上的开源大模型很多，我也具备本地部署大模型的条件，何不在 UOS 系统上部署一下大模型呢？

本地部署大模型的方法很多，一般选择 docker 容器部署，或者使用本地服务框架。这里介绍使用本地服务框架 Ollama 部署。

## Ollama 大模型框架

Ollama 是一个新兴的大模型框架，旨在为机器学习和人工智能研究提供高效、灵活和可扩展的解决方案。随着深度学习模型的复杂性和规模不断增加，开发者和研究人员需要更强大的工具来处理大规模数据和复杂的模型架构。Ollama 正是在这种需求下应运而生的。

### Ollama 的核心特点

1. **高效计算**：Ollama 采用先进的分布式计算技术，可以在多 GPU 、多节点环境中高效运行。这使得它能够处理大规模数据集和复杂的模型训练任务，大大缩短了训练时间。
2. **灵活性**：Ollama 支持多种深度学习框架，如 TensorFlow、PyTorch 等，开发者可以根据项目需要选择最合适的工具。同时，Ollama 还提供了丰富的 API 和库，方便用户进行自定义开发和扩展。
3. **可扩展性**：Ollama 具有强大的扩展能力，可以轻松应对模型和数据规模的增长。无论是初创公司的小型项目，还是大企业的大型应用，Ollama 都能提供稳定和高效的支持。
4. **易用性**：Ollama 注重用户体验，提供了简洁明了的用户界面和详细的文档说明。即使是没有深厚技术背景的用户，也可以快速上手，利用 Ollama 进行模型训练和部署。

## Ollama 安装与运行

在 Deepin 系统下，安装 Ollama 非常简单，只需要如下命令：

```
$ curl -fsSL https://ollama.com/install.sh | sh
>>> Downloading ollama...
######################################################################## 100.0%-=O=#  #   #   #               ######################################################################## 100.0%
>>> Installing ollama to /usr/local/bin...
请输入密码

```

Ollama 默认会安装在 /usr/local/bin 目录下，安装完毕之后，可以在命令行运行 ollama，如果不知道有哪些命令，可以从 ollama help 开始：

```
(base) alex@alex-deepin-os:~$ ollama help
Large language model runner

Usage:
  ollama [flags]
  ollama [command]

Available Commands:
  serve       Start ollama
  create      Create a model from a Modelfile
  show        Show information for a model
  run         Run a model
  pull        Pull a model from a registry
  push        Push a model to a registry
  list        List models
  ps          List running models
  cp          Copy a model
  rm          Remove a model
  help        Help about any command

Flags:
  -h, --help      help for ollama
  -v, --version   Show version information

Use "ollama [command] --help" for more information about a command.

```

可以看到，ollama 的命令行参数和 docker 有些相似。启动一个大模型非常简单，比如我想运行 gemma2 27b 参数的大模型：

```
(base) alex@alex-deepin-os:~$ ollama run gemma2:27b
pulling manifest 
pulling d7e4b00a7d7a...   4% ▕█                                             ▏ 655 MB/ 15 GB 
```

ollama 会自动完成模型文件的下载，容器的创建，并运行起来。ollama 本身提供了命令行交互接口。

```
(base) alex@alex-deepin-os:~$ ollama run gemma2
>>> Send a message (/? for help)
```

此外，Ollama 还提供了和 OpenAI API 兼容的接口服务，本地服务的地址为：

> http://127.0.0.1:11434

## 配置 UOS AI

添加 UOS AI 账号，模型类型还是选择自定义，API Key 不用填，模型名就填写 ollama 运行的大模型名，比如 gemma2，如果运行的是 gemma2 27b 版本，就填写 gemma2:27b，API 地址填写 http://127.0.0.1:11434/v1

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_ollama_02.png)

配置完成后，在下拉框中选择刚配置的账号。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_ollama_03.png)

接下来就可以愉快的和 AI 对话了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_ollama_04.png)

我使用的是 Google 的 Gemma2 9b 大模型，速度还挺快。

如果你想尝试其它的大模型，可以去 ollama 的模型仓库看看。

> https://ollama.com/library

里面收录了很多大模型，比如 [llama3](https://ollama.com/library/llama3)、[qwen2](https://ollama.com/library/qwen2)、[deepseek-coder-v2](https://ollama.com/library/deepseek-coder-v2) 等。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202407/images/uos_ollama_05.png)

## 小结

写到这里，是不是感觉到在 Deepin 系统上部署大模型太简单了？是的，各种服务框架的出现，让我们不用手写代码就能部署大模型，其实本地服务框架远不止 ollama，还有 FastAPI、Streamlit 等等众多框架，甚至还有更多的高级框架，如 Dify，提供的功能更多更强。让我们慢慢探索吧！
