# 巧用Kaggle进行模型训练

> 本文翻译自Medium上的一篇文章，原文标题：Using Kaggle for your Data Science Work，点击文末的阅读原文可以跳转到原文。

数据工程师都喜欢Jupyter Notebook，但是有时候您需要处理非常大的数据集和/或复杂的模型，而您的计算机却无法胜任。好消息来了，您可以将Jupyter Notebook文件导入Kaggle。如果您是数据科学的新手，那么Kaggle对你而言是一个举办有奖金的数据科学竞赛的网站。实际上，Kaggle还是一个拥有丰富信息的伟大社区，非常愿意帮助您提升数据科学水平。

Kaggle的另一个功能是它们具有免费的在线云计算（虽有一些限制）。因此，如果您的计算机温度太高、运行时间太长或没有足够的处理能力或内存无法运行模型，就可以使用Kaggle的核来运行代码！来注册吧！

---

## 使用Kaggle的好处

* 免费！开一个Kaggle帐户，您就可以免费使用他们的服务器。
* 云计算。您可以运行模型，提交更改，然后在另一台计算机上拉取（pull）模型。只要您可以访问互联网，您的工作就可以跟随您（无需使用Git）！
* GPU。对于计算密集型模型，您最多可以使用2个核和13 GB的GPU RAM。那些负担不起昂贵GPU的人，为什么不使用Kaggle的GPU？
* Notebook或脚本。尽可以使用您习惯的方式导入代码！
* 无需使用 *pip install*。Kaggle已预装了大多数python软件包（您甚至可以用 *pip install* 安装Kaggle没提供的软件包）。
* 暗黑模式。使用起来感觉更好。

## 缺点和限制

并非处处都是阳光和彩虹。首先，Kaggle由Google拥有。因此，如果您对Alphabet的服务器上安装的面部识别模型感到不满意，那么Kaggle的核可能不适合您。

另外，在您的网页上运行的核，在无用户输入的情况下，只能在一个小时内运行。因此，如果您在运行模型后走开一个多小时，内核将停止。您将失去所有输出，并且必须重新启动核。您可以通过提交代码来解决此问题，该代码将在与您在网页上看到的不同的核中运行。但是要注意的一点是，只有在核完全运行后才能看到输出。因此，如果您的总运行时间为5个小时，那么您将无法在5个小时内检查已提交的核。这样如果代码有致命错误，那么您要等5个小时才能知道。

以下是使用Kaggle时的**硬件**和**时间限制**：

* 9小时执行时间
* 5 GB自动保存的磁盘空间（/kaggle/正在运行）
* 16 GB的临时暂存磁盘空间（/kaggle/工作区外部）

### CPU规格

* 4个CPU核心
* 16 GB的RAM

### GPU规格

* 2个GPU核心
* 13 GB的RAM

如果您要装一个上述规格的计算机，费用可轻松超过1,000美元。只要确保您的数据少于16GB的磁盘空间（除非您使用的是Kaggle数据集），并且能9小时内跑完。如果您的模型可以在这些限制下运行，那么请上传数据并开始工作！

## Kaggle入门

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/kaggle_01.png)

* 登录您的Kaggle帐户
* 在顶部栏中，单击**Notebooks**
* 然后选择**New Notebook**

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/kaggle_02.png)

* 选择**Python**或**R**
* 选择编码类型
* 如果要使用GPU，请单击**Show Advanced Settings**，然后选择**GPU on**
* 然后点击**Create**

## Kaggle核

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/kaggle_03.png)

*新的在线Jupyter Notebook*

如果您选择的是notebook类型，则应该感觉像Jupyter Notebook一样。要上传数据，请单击右上角 **+ Add data**。您可以选择Kaggle现有数据集或上传自己的数据集。请记住，您最多只能使用16GB的数据。

在右侧栏中，您可以在线跟踪核。**Sessions**选项卡跟踪您拥有多少计算能力。将**Workspace**选项卡视为GUI文件结构。如果您使用的是Kaggle数据集，则文件将位于*/kaggle/input/your-kaggle-dataset*中。如果是上传数据集，则文件将位于*/kaggle/input/your-uploaded-data*中。在**Settings**标签上，您可以更改以前的设置。

现在已经准备就绪！开始编码，享受免费的在线notebook。完成或准备提交后，请点击右上角的**Commit**按钮。您的代码将在单独的核中运行。一旦所有代码运行完毕，它将形成一个版本。您可以返回任何版本或提交的代码，然后查看输出（如果运行正确）。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/kaggle_04.png)

如果您要提交到kaggle竞赛，请前往你的核。在左侧，单击**Outputs**。如果您有 *.csv* 输出，则可以在此处看到。选择您的 *.cvs* 文件，然后单击**Submit to Competition**。

---

Kaggle是数据科学工程师的强大工具。他们甚至有使用pandas、神经网络的python课程，全部使用他们的核。有关另一项免费的在线云服务，请查看Google Colab。

## 译后记

在薅资本主义的羊毛的道路上，我也算是不遗余力，之前写过Google Colab的文章，有兴趣可以了解一下：

1. [Google Colab上安装TensorRT](https://blog.csdn.net/mogoweb/article/details/90231260)
2. [谷歌GPU云计算平台，免费又好用](https://blog.csdn.net/mogoweb/article/details/90216148)
3. [机器学习硬件设施差？免费使用谷歌的GPU云计算平台](https://blog.csdn.net/mogoweb/article/details/79533008)
4. [免费使用谷歌的深度学习云服务](https://blog.csdn.net/mogoweb/article/details/79539535)

薅资本主义的羊毛虽好，可不要沉迷其中。经过我的试用，免费的GPU云服务或多或少都有一些问题，只适用于初学者作为入门平台。作为一名严肃的开发者，还是尽可能自行配一台高性能的计算机。

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)
