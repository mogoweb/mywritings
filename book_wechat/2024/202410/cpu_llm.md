# 能在 CPU 上运行的开源大模型推理框架

如今，大模型的发展势头迅猛，而且，大模型的演进出现了两种分化趋势：一方面，开源大模型的规模不断扩大，从最初的数十亿参数迅速扩展到数千亿参数，未来甚至可能突破万亿级的超大规模。这类巨型模型通过引入更丰富的语义理解、复杂的推理能力和跨领域知识整合，展现出强大的智能。各大厂商开展军备竞赛，为了追求更高的准确率和更强的泛化能力，大模型的“无上限”扩展不断突破技术和硬件的边界。

另一方面，模型也在向小型化方向发展，这主要是为了满足设备端的应用需求。用户端设备种类繁多，从配备强大计算能力的高端 PC 到资源受限的物联网设备，再到便携式智能设备和嵌入式系统，算力差异巨大。传统的大模型虽然功能强大，但无法在这些设备上高效运行，因此针对用户端的小型化模型也有广阔的市场。小型化模型力求在有限的资源环境中，实现较高的性能表现，以支持用户端的离线推理、实时响应以及低功耗应用。通过参数剪枝、知识蒸馏、量化等技术手段，小型模型可以有效减小体积和计算量，从而适应各类用户设备的运行条件。

这其中，模型量化技术的进步为模型小型化提供了关键支撑。量化技术的原理类似于 MP3 的有损压缩：虽然会带来一定的质量损失，但对用户体验几乎没有影响。MP3 之所以比无损音频格式更受欢迎，正是因为在不显著牺牲音质的前提下，能极大缩小文件体积。量化技术在 AI 模型上也有类似效果，它通过减少数值表示的精度，显著缩减了模型的存储和计算量，使其更适合在低算力设备上运行。常见的量化方法包括将模型参数从原始的 32-bit 浮点数精度压缩为 16-bit、8-bit，甚至 4-bit，这样不仅减少了模型的内存占用，还能大幅降低推理时间，显著提升算力的利用效率。

如今，1-bit 量化技术则进一步将这一进程推向新高度。1-bit 量化的核心在于仅保留模型权重的方向信息而舍弃其大小信息，极大地降低了数据传输和计算的复杂性。1-bit 量化可以大幅加快模型推理速度，同时减少内存使用，使得在 CPU 上推理成为可能。

近日，微软发布了一个全新的开源项目——BitNet.cpp，这是专为 1-bit 大语言模型（LLMs）推理而设计的框架。BitNet.cpp旨在通过优化内核为 CPU 上运行的 1.58-bit 模型提供快速且无损的推理支持，并在未来版本中计划支持 NPU 和 GPU 。

BitNet.cpp的开源为1-bit LLM的普及和大规模推理打开了新的大门，其在CPU上的高效推理性能，极大地扩展了大模型在本地设备上的可行性。

有这么好的开预案项目，当然要尝试一下。下面介绍在 deepin v23 上如何使用 BitNet.cpp。

首先来看看 BitNet.cpp 对环境的要求：

* Python 3.9及以上版本
* CMake 3.22及以上版本
* Clang 18及以上版本
  这几个条件在 deepin v23 上很容易满足。

## 检查 clang 版本并安装 clang 18

首先检查一下 clang 的版本：

```bash
$ clang --version

Deepin clang version 17.0.6 (5deepin4)

Target: x86\_64-pc-linux-gnu

Thread model: posix

InstalledDir: /usr/bin
```

当前版本版本是 clang 17，得升级到 clang 18。在 deepin V23 下升级 clang 非常简单

```
$ sudo apt update
$ sudo apt install clang-18
# 将clang链接到clang 18
$ sudo ln -sf ../lib/llvm-18/bin/clang /usr/bin/clang
```

安装后检查 clang 版本：

```
$ clang --version

Deepin clang version 18.1.7 (1)

Target: x86\_64-pc-linux-gnu

Thread model: posix

InstalledDir: /usr/bin
```

现在版本是 clang 18，满足第一个要求。

## 检查 cmake 版本

使用如下命令检查 cmake 的版本:

```
$ cmake --version
cmake version 3.28.3

CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

可以看出 cmake 已经满足最低版本要求，不需要做额外的操作。

## 创建 python 环境

建议使用 AnaConda 来管理 Python 环境，对个人、教育工作者或者 300 人以下的小企业免费。

使用 conda 命令创建一个 python 3.9 的虚拟环境，并激活这个虚拟环境：

```
$ conda create -n bitnet-cpp python=3.9
$ conda activate bitnet-cpp
```

## 下载源码并安装 Python 包，量化大模型

1. 下载源码：

```
$ git clone --recursive https://github.com/microsoft/BitNet.git
```

2. 安装所需的 Python 包

```
$ cd BitNet
$ pip install -r requirements.txt
```

3. 量化 LLama3 8B 大模型

```
$ python setup_env.py --hf-repo HF1BitLLM/Llama3-8B-1.58-100B-tokens -q i2_s
INFO:root:Compiling the code using CMake.
INFO:root:Downloading model HF1BitLLM/Llama3-8B-1.58-100B-tokens from HuggingFace to models/Llama3-8B-1.58-100B-tokens...
INFO:root:Converting HF model to GGUF format...
INFO:root:GGUF model saved at models/Llama3-8B-1.58-100B-tokens/ggml-model-i2_s.gguf
```

上面的脚本连接到 Hugging Face 下载模型并将其转换为量化格式，当然你也可以手动下载模型后进行推理。

```
huggingface-cli download HF1BitLLM/Llama3-8B-1.58-100B-tokens --local-dir models/Llama3-8B-1.58-100B-tokens
python setup_env.py -md models/Llama3-8B-1.58-100B-tokens -q i2_s
```
## 基本使用

BitNet.cpp提供了简单的命令行接口，便于用户进行推理。以下是一个运行推理的示例：

```
python run\_inference.py -m models/Llama3-8B-1.58-100B-tokens/ggml-model-i2\_s.gguf -p "Where is Mary?" -n 6 -temp 0
```
在这个例子中，模型会根据给定的上下文生成6个token，输出答案为：“Mary is in the garden.”。
其中的关键参数解释：

* `-m`：模型路径。
* `-n`：生成的token数量。
* `-p`：提示词，用于生成文本。
* `-t`：使用的线程数。
* `-temp`：控制生成文本的随机性，值越低，输出越确定。

## 小结

与主流 LLM 推理框架（如 Hugging Face Transformers 或 DeepSpeed）相比，BitNet.cpp 的独特优势在于专注于低比特模型推理，从而显著降低了计算资源需求。不同于传统框架需借助 GPU 才能达到高效推理速度，BitNet.cpp 通过高效的低比特量化技术，仅依赖 CPU 也能实现接近或等同的推理性能。这一优势可以大大推进侧端大模型的普及。


