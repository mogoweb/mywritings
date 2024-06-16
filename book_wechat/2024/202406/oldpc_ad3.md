# 老电脑焕发第二春，玩转 Stable Diffusion 3

几年前，我头脑一热，配置了一台顶配级消费 PC（RTX 2080 Ti GPU + i9 CPU），打算用来学习 AI。然而，起初我并没有找到合适的切入点。深度学习早期阶段，消费级显卡根本无法承担训练大模型、微调大模型，甚至连运行大模型都很吃力。结果，这台电脑主要用来学习 TensorFlow、Python 编程等基础知识，但最后从入门到放弃。不过，当时配置的 CPU 和内存还不错，用来编译 Chromium 浏览器和 Android 系统也算是物尽其用，唯独显卡几乎闲置。随着 Nvidia 不断推出新显卡，RTX 2080 Ti 显得越来越落伍了。

近年来，AIGC（人工智能生成内容）技术不断进步，大模型的规模越来越大，参数动辄以十亿（Billion）为单位，但对 GPU 的要求却逐渐降低。比如最近发布的 Stable Diffusion 3，仅需 6G 显存就能很好地运行。此外，语音识别、GPT、ChatTTS、Text to Audio、Text to Video 等大模型也能在消费级 GPU 上运行。

目前，以我的资源和实力，进入 AI 行业进行大模型研发不太现实，只能围绕 AI 技术做一些应用，或者体验一下 AIGC。因此，我最近在研究如何在本地部署大模型。

前几天，Stable Diffusion 3 的大模型开源了，我第一时间在电脑上进行了本地部署。下面是我在 Windows 上部署 Stable Diffusion 3 的总结。

## ComfyUI

Stability AI 开源了 Stable Diffusion 3 Medium 模型，拥有 20 亿参数。要让模型运行，还需要外围框架和 UI 构建应用。在图像生成领域，常用的有 Stable Diffusion WebUI 和 ComfyUI，以及后起之秀 Fooocus。目前只有 ComfyUI 支持 SD3。

第一眼看到 ComfyUI 的界面，我就被它吸引了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/oldpc_sd3_01.png)

这是一个非常适合程序员使用的交互界面，通过浏览器访问。整个过程以工作流形式组织，非常适合根据需求定制工作流，对于重复性工作，可以反复执行工作流，减少重复劳动。工作流可以以 JSON 文件形式保存和加载，方便交流工作流程。

插件支持是现代软件的标配，ComfyUI 在这方面做得很好，已经建立了良好的生态，社区参与度很高，有专门分享插件和交流工作流的。

### 安装 ComfyUI

ComfyUI 支持 Windows、Linux 和 Mac OS 系统。我之前主要在 Linux 系统下开发，写的文章也是在 Linux 下部署。但自去年转向开发应用后，发现绝大多数用户使用的是 Windows 系统。虽然也开发过一些信创系统的应用，但推不动。广大网友虽然口头上支持国产系统，但实际行动上还是更倾向于 Windows。这篇文章就讲述如何在 Windows 下部署 Stable Diffusion 3，准确地说是 ComfyUI 在 Windows 系统下的部署，系统版本为 Windows 11 64 位。

ComfyUI 是开源的，可以在 GitHub 上访问：

> https://github.com/comfyanonymous/ComfyUI

针对 Windows 系统，ComfyUI 有 Release 包，下载即可使用：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/oldpc_sd3_02.png)

作为程序员，我更喜欢使用第二种方式，直接 clone 代码库：

```
git clone https://github.com/comfyanonymous/ComfyUI
```

安装依赖：

```
pip install -r requirements.txt
```

运行：

```
python main.py
```

打开浏览器，地址栏输入 http://127.0.0.1:8188/ 就可以看到 ComfyUI 的界面了。

### 共享 WebUI

由于我之前安装过 SD-WebUI，里面有 Python 虚拟环境、Pip 包以及一些 SD 模型，正好 ComfyUI 也支持与 SD-WebUI 共用一些设施，于是我写了一个 run.bat 脚本：

```
set VENV_DIR=C:\ai\stable-diffusion-webui\venv
set PYTHON="%VENV_DIR%\Scripts\Python.exe"
echo venv %PYTHON%
%PYTHON% main.py
pause
```

双击 run.bat 脚本即可启动 ComfyUI。

如果要共用 SD-WebUI 中的模型，将代码中的 extra_model_paths.yaml.example 复制一份，重命名为 extra_model_paths.yaml，然后用文本编辑器编辑。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/oldpc_sd3_03.png)

替换其中 path/to/stable-diffusion-webui/ 的路径。

如果你希望将 ComfyUI 的模型放在别处，不想放在程序内，还可以去掉下面 comfyui 的注释行，指定一个路径。

如果运行出现错误：

> AssertionError: Torch not compiled with CUDA enabled

可能是 PyTorch 的版本问题，解决方法是：

```
C:\ai\stable-diffusion-webui\venv\Scripts\pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu121
```

**注意** ：因为我使用的是 WebUI 的 python 虚拟环境，所以 pip 前面需要带上路径，如果你是使用的系统 python：

```
pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu121
```

## 下载模型

模型下载的官方地址是：

> https://huggingface.co/stabilityai/stable-diffusion-3-medium

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/oldpc_sd3_04.png)

模型比较大，如果翻墙不便，可以尝试搜索一下国内的下载站点。

模型下载文件包含三个部分：

* 模型
* 文本编码器
* ComfyUI 工作流

先来看看模型文件，有四个，体积都还不小，到底下哪个呢？还是全部下载？

看看官方对着四个文件的解释：

* sd3_medium.safetensors 包含 MMDiT 和 VAE 权重，但不包含任何文本编码器。
* sd3_medium_incl_clips_t5xxlfp16.safetensors 包含所有必要的权重，包括 T5XXL 文本编码器的 fp16 版本。
* sd3_medium_incl_clips_t5xxlfp8.safetensors 包含所有必要的权重，包括 T5XXL 文本编码器的 fp8 版本，在质量和资源要求之间实现平衡。
* sd3_medium_incl_clips.safetensors 包含除 T5XXL 文本编码器之外的所有必要权重。它需要的资源很少，但如果没有 T5XXL 文本编码器，模型的性能会有所不同。

如果对 text to image 有所了解的话，就会明白所谓的文本编码器就是指的 CLIP 模型，也就是计算机用来理解文字的模型，这个 CLIP 模型可以包含在 SD 模型内部，也可以作为单独的文件，独立加载。

看看 text_encoders 下的文件，我们可以看到里面有四个 CLIP 模型：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/oldpc_sd3_05.png)

目前还没有看到对这四个 CLIP 模型的介绍，不过理解上面对模型的介绍，t5xxl 文本编码器的性能更佳，fp16 版本质量最佳，但对资源的需求更大。

再来看看 comfy_example_workflows 下的文件：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/oldpc_sd3_06.png)

这下面有三个工作流文件，文件比较小，先下载这三个工作流文件。

在 ComfyUI 中打开 sd3_medium_example_workflow_basic.json 文件：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/oldpc_sd3_07.png)

可以看到，这个工作流加载了三个 CLIP 模型（为什么要 3 个？）和一个基础模型。那我们就下载这四个模型吧。

* sd3_medium.safetensors
* clip_g.safetensors
* clip_l.safetensors
* t5xxl_fp8_e4m3fn.safetensors

这几个模型总共有十几个 G，慢慢下载吧。

下载完毕后，将 clip_g.safetensors、clip_l.safetensors、t5xxl_fp8_e4m3fn.safetensors 丢到 ComfyUI 的 models\clip 下，sd3_medium.safetensors 丢到 ComfyUI 的  models\checkpoints 下。

## 执行基础的工作流

在 ComfyUI 中打开上一个步骤中下载的 sd3_medium_example_workflow_basic.json 文件。需要注意这个工作流中的 checkpoint 名称是带有路径的，替换成 sd3_medium.safetensors 即可。

按 **Ctrl + 回车** 快捷键，即可生成一张漂亮的图。整个过程还是相当快，生成的质量也还不错。

给大家看看我在完全没有修改工作流所生成的图。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/oldpc_sd3_08.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/oldpc_sd3_09.png)

这是我初次使用 ComfyUI，还需要继续摸索，希望能用 ComfyUI 和 SD3 生成理想的图片。敬请关注！

