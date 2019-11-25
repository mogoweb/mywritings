# 在VS Code中编写Jupyter Notebook

> 将Visual Studio Code打造成超酷的机器学习开发工具。

对于在线学习过机器学习有关课程的朋友来说，Jupyter Notebook应该不陌生。Jupyter Notebook提供了基于Web的交互式机器学习环境，用户无需安装任何软件，只需可以上网的浏览器，就可以体验机器学习。Jupyter Notebook在线环境可以让用户编写Notebook，修改代码，并实时执行，查看结果。不过基于Web的编辑器，并没有提供过多的代码编写辅助，对于习惯使用IDE编写代码的开发人员，裸写机器学习代码，的确不太方便。

现在有个好消息，Visual Studio Code提供了对Jupyter Notebook的本机支持。借助于强大的插件系统，VS Code日益成为机器学习工程师喜爱的编程工具，关于VS Code的介绍，可以参考我前面的一篇文章：

[深度学习软件开发环境搭建](https://mp.weixin.qq.com/s/BHjskcSIu407t_xn-lbjTQ)

下面介绍如何在VS Code中编写和使用Jupyter Notebook。

#### 在VS Code中使用Jupyter Notebook

**使用VS Code创建新的Notebook**： 组合键**CTRL + SHIFT + P**，然后运行**Python: Create Blank New Jupyter Notebook**命令。

**在VS Code中打开现有的Notebook**：选择菜单： **File | Open File...**，打开Jypyter Notebook文件（.ipynb后缀）。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/vs_notebook_02.png)

点击单元格左侧的**三角**按钮，可以执行单元格中的代码。

新建或打开Jupyter笔记本文件时，默认情况下，VS Code会自动在本地启动Jupyter服务器。请确保当前的Python环境已经安装了jupyter，可以通过：

```
conda install jupyter # 或 pip install jupyter
```

命令安装。如果说你想使用远程Jupyter服务器，抑或你已经在本地启动了Jupyter服务器，你可以自行指定。组合键**CTRL + SHIFT + P**，然后输入**Python: Specify Jupyter Server URI**:

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/vs_notebook_03.png)

Jupyter中编写Python代码，和在VS Code中编写普通的Python代码一样，其方便之处就在于可以执行一小块代码，并立即看到结果。比如我使用matplotlib绘图，图形可以显示在VS Code编辑器中：

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/vs_notebook_04.png)

非常遗憾的是，VS Code还不支持Jupyter Notebook的调试。要调试Jupyter Notebook，需要首先将其导出为Python文件。导出为Python文件后，即可使用VS Code调试器单步执行代码、设置断点、检查状态并分析问题。关于VS Code调试Python代码，以后有机会再写。

#### 通过插件提升Jupyter Notebook体验

写到这儿，似乎在VS Code中和在Web环境下编写Jupyter Notebook没什么差别。别慌，VS Code的强大就在于其插件。下面介绍一个智能代码补齐插件：IntelliCode。

在插件库中搜索**IntelliCode**，请认准微软出品。安装插件之后，在编写代码时，IntelliSense会在代码单元内为您提供智能代码补齐建议，这里提供的建议是AI基于当前代码上下文提供的自动完成建议，和以前的IntelliSense还不太一样，并不仅仅是包名或者函数名或参数这样的建议。当然，现在的AI还不是特别靠谱，有时提供的一些建议有些智障，期待后续会持续增强吧。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/vs_notebook_05.gif)

使用VS Code的另一个好处是，您可以通过单击笔记本工具栏中的"variable"按钮来浏览变量的当前状态和值，可以实时跟踪变量的值。这个功能并不需要额外安装插件。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201911/images/vs_notebook_06.gif)

VS Code包含着一个巨大的宝库：插件，里面有各种各样的宝贝，等着我们去发现。等我有更多的心得，再和大家分享。

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)