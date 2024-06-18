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

而更晚出现的ComfyUI凭借超高的可定制性和复现性迅速火遍全球。有设计师表示SD发布了XL1.0后，ComfyUI用它优秀的底层逻辑率先打击了臃肿不稳定的WebUI1.6，成为更适合“体验”XL的SD生图工具。