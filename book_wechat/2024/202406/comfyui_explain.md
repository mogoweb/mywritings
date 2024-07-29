# 程序员的视角解析 ComfyUI

目前使用 Stable Diffusion 进行创作的工具主要有三个：SD-WebUI、Fooocus 和 ComfyUI。从用户交互界面上看，三者的设计理念有所不同。

先来看看 Fooocus，其用户界面极简。通常情况下，用户只需要输入提示词，即可生成图。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/comfyui_explain_01.png)

对于有进一步参数控制需求的用户，可以勾选 **Advanced**，出现高级选项。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/comfyui_explain_02.png)

虽然有些的参数可调，但总体而言，比较克制，只放出了有限的容易理解的参数。

Fooocus 不仅用户极简，安装升级都是一个脚本搞定，包括模型下载，主打一个开箱即用。由于 Fooocus 的极简特点，迅速抢了不少 SD-WebUI 的用户，获得广泛欢迎。

再来看看 SD-WebUI 的用户界面，真的是琳琅满目，应有尽有。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/comfyui_explain_03.png)

可以说 SD-WebUI 主打一个全面，将所有控制权都交给用户，插件的支持更是高级玩家的最爱。当然，这也招致不少用户的批评，认为其太过臃肿。

最后看看 ComfyUI 的用户界面，是不是眼前一黑，就这么一个白板：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/comfyui_explain_04.png)

加载一个事先制作好的工作流：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/comfyui_explain_05.png)

完全的极客风，估计很多人看到这种界面，立即放弃。

然而，事实上 ComfyUI 凭借超高的可定制性和复现性迅速获得了设计师的青睐，作为一名程序员，看到这种用户界面，反而觉得亲切。

ComfyUI 将整个图像生成过程分解为多个独立的节点，每个节点都有自己独立的功能，例如加载模型，文本提示，生成图片等等。每个模块通过输入和输出的线连在一起变成一个完整的工作流。

整个过程用户可以灵活的调整和配置不同的功能节点，意味着流程更加自由，控制更加精准。

ComfyUI 设计成这种工作流模式，不是没有原因的。因为 Stable Diffusion 的底层运行逻辑也是如此，所以要更好的理解和掌握 ComfyUI，先了解一些Stable Diffusion 的底层运行逻辑是有帮助的。

## LDM 底层逻辑

Stable Diffusion 的基础模型是 Latent Diffusion Model（LDM），翻译成中文就是**潜在扩散模型**，可以理解为主要的图片生成流程都在一个叫「latent space（潜在空间）」的魔法盒子里进行。

图片在这个空间存在的方式是我们无法识别的向量，我们只需要知道这些我们无法识别的东西所表示的信息和图片相差无几，但是数据尺寸却变得非常小就行，这是一个类似于压缩的过程，所以在这个空间中进行运行可以大大缩小运行内存。

这个过程可以简单理解为，向潜在空间输入文件，数据经过处理生成图片并输出：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/comfyui_explain_06.png)

如果是文本生图，输入就是提示语（Prompts），文本内容。但是文本内容计算机是无法理解的，所以我们需要将文本转换为计算机能够理解的信息，这个过程使用了 Clip 模型。而潜在空间里的图片，我们也无法理解，需要使用 VAE 模型转换成图片格式，所以整个流程就是：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/comfyui_explain_07.png)

控制模型实际生成部分的模型是 KSampler（采样器），在这其中我们可以控制迭代次数，种子数值等等。而这个步骤就发生在潜在空间中。

这就是最基础的文生图过程：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/comfyui_explain_08.png)

再来看一个 ComfyUI的 最基础模型，就会清晰很多。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202406/images/comfyui_explain_09.png)

## 小结

ComfyUI 工作流是一个基于图形节点编辑器的工作流程，通过拖拽各种节点到画布上，连接节点之间的关系，构建一个从加载模型到生成图像的工作流程。

每个节点代表一个 Stable Diffusion 相关的模型或功能，节点之间通过连线传递图片信息。

ComfyUI 工作流从加载模型开始，加载模型节点负责加载训练好的 Stable Diffusion 模型。

然后，通过CLIP Text Encode节点对输入的关键词Prompt进行处理，将文本转换为图像描述，并生成一个初始的Latent Image。

接下来，进入采样器和VAE解码节点，这两个节点的作用是将初始的 Latent Image 进行采样和编码解码，得到生成的图像。

最后，生成的图像会通过连线传递到下一个节点进行进一步处理或输出。

通过将简单的工作流组合，可以实现很多复杂的工作流程，这也是 ComfyUI 的强大之处。

