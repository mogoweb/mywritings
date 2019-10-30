# 想提高棋艺？试试这款围棋AI

作为一名围棋渣渣，时不时会上对弈平台下下棋。围棋太博大精深，非常惭愧，虽然在下棋上花的时间很多，但一直处在菜鸟阶段，长期在1级和1段之间徘徊(腾讯野狐围棋上的排位)。要提升水平，需要下功夫去记定式、做死活题，但那太枯燥了，相较而言，我更喜欢上网厮杀，屠龙或被屠，爽一把再说。我等初级选手，经常会碰到那种不按套路的对手，有时明明觉得对方下了无理手，但就是不知道如何反击。再就是棋盘太空旷，不知如何选点。这些虽然在书上可以学到一些基本技巧，但一到实战，往往不知如何下手。

腾讯野狐围棋上经常有直播讲解，和电视节目讲解有所不同，解说会搬出**绝艺**（腾讯出品的围棋AI软件），对一些关键步骤，给出几种推荐的选点，以及随后几步的变化。当然，专业棋手的棋我也看不懂，虽然看着热闹，实际上对平时对局也没有帮助。但绝艺真是一个好工具，如果能对我以往下的棋进行复盘，了解对一些无理手的应对方法，以及一些选点提示，无疑可以提高实战水平。然而，上网搜索了一番，似乎**绝艺**只提供给专业棋手使用，普通人虽然也可以申请**绝艺**复盘，但有很多限制，特别是像我这种低水平的棋手，基本上申请不到。

我之前也研究过一些围棋AI软件，比如Leela、ELF OpenGo，请参考我之前的一篇文章：

[想和围棋高手过招？火力更猛且开源的围棋AI来了...](https://mp.weixin.qq.com/s?__biz=MzI3NTQyMzEzNQ==&mid=2247484935&idx=1&sn=079cb5510e11fc48d0d5dbe831ca7213&chksm=eb044cf7dc73c5e13a8f7a98e1dbdad5166aee2768de95371cec89c787a7019618f46b7b0ca0&token=504277048&lang=zh_CN#rd)

带着AI，能够在网上大杀四方，但那又有什么意义呢？我还是希望能够提升一下自身的围棋水平，之前尝试的几款围棋AI软件，并没有**绝艺**那样的复盘功能。

想到之前看到过一篇报道，腾讯开源了一款围棋AI，开始还以为是**绝艺**，仔细一搜索，原来是另外一款AI，名为PhoenixGo，曾经以**金毛测试**出现在腾讯野狐围棋对战平台，最新的对战记录还停留在2019年5月5日，可能腾讯现在把主要力量都放在**绝艺**上了。看PhoenixGo上的代码提交，最新提交也在8个月之前。不过也不要小瞧PhoenixGo，他在2018年4月以“凤凰围棋”身份参赛，并以全胜战绩获取冠军，对付专业棋手还是绰绰有余。

需要注意的是PhoenixGo是一款围棋AI引擎，也就是说它并不带用户界面，好在它支持围棋界广泛采用的GTP协议，可以配合一些第三方软件使用，比如Sabaki，如果是希望进行对局分析，推荐使用GoReviewPartner，这是一款使用python编写的软件，有用户界面。

到此，即使没有绝艺，我也可以自行构建出类似绝艺的围棋AI。

在Linux平台上，没有预编译好的包，对于我等程序员而言，自然难不倒我，下面就聊聊如何从源码build出PhoenixGo，以及如何配合GoReviewPartner。

## 构建PhoenixGo

### 1. 编译环境准备

所需的编译软件有：

* 带C++ 11支持的GCC，使用系统默认安装的gcc即可
* Bazel 0.19.2，如果你的Bazel版本比这个高，先卸载，然后去Bazel官网下载一个0.19.2的版本
* （可选）CUDA和cuDNN，如果你的机器装的是Nvidia的显卡，强烈建议安装。PhoenixGo对显卡算力要求很高，如果只用CPU，速度会慢很多。
* （可选）TensorRT，这是Nvidia推出的GPU加速库，我开始选择了启用TensorRT，但运行起来却有点问题，回头再研究一下。建议使用上，能加速一点是一点。

### 2. 下载源码并进行编译配置

命令如下：

```
$ git clone https://github.com/Tencent/PhoenixGo.git
$ cd PhoenixGo
$ ./configure
```
运行编译配置命令时，会有一系列的选项让你选择，通常情况下使用默认值即可，但是询问CUDA支持时，记得选y。我的配置选项如下：

```
$ ./configure
WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
You have bazel 0.19.2 installed.
Please specify the location of python. [Default is /home/alex/anaconda3/bin/python]: 


Found possible Python library paths:
  /home/alex/work/ai/tensorflow/models/research
  /home/alex/anaconda3/lib/python3.6/site-packages
  /home/alex/work/ai/tensorflow/models/research/slim
Please input the desired Python library path to use.  Default is [/home/alex/work/ai/tensorflow/models/research]
/home/alex/anaconda3/lib/python3.6/site-packages
Do you wish to build TensorFlow with XLA JIT support? [Y/n]: 
XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: 
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with ROCm support? [y/N]: 
No ROCm support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: y
CUDA support will be enabled for TensorFlow.

Please specify the CUDA SDK version you want to use. [Leave empty to default to CUDA 10.0]: 


Please specify the location where CUDA 10.0 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: 


Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 7]: 


Please specify the location where cuDNN 7 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: 


Do you wish to build TensorFlow with TensorRT support? [y/N]: 
No TensorRT support will be enabled for TensorFlow.

Please specify the locally installed NCCL version you want to use. [Default is to use https://github.com/nvidia/nccl]: 


Please specify a list of comma-separated Cuda compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
Please note that each additional compute capability significantly increases your build time and binary size. [Default is: 5.2]: 


Do you want to use clang as CUDA compiler? [y/N]: 
nvcc will be used as CUDA compiler.

Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]: 


Do you wish to build TensorFlow with MPI support? [y/N]: 
No MPI support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native -Wno-sign-compare]: 


Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: 
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See .bazelrc for more details.
	--config=mkl         	# Build with MKL support.
	--config=monolithic  	# Config for mostly static monolithic build.
	--config=gdr         	# Build with GDR support.
	--config=verbs       	# Build with libverbs support.
	--config=ngraph      	# Build with Intel nGraph support.
	--config=dynamic_kernels	# (Experimental) Build kernels into separate shared objects.
Preconfigured Bazel build configs to DISABLE default on features:
	--config=noaws       	# Disable AWS S3 filesystem support.
	--config=nogcp       	# Disable GCP support.
	--config=nohdfs      	# Disable HDFS support.
	--config=noignite    	# Disable Apacha Ignite support.
	--config=nokafka     	# Disable Apache Kafka support.
	--config=nonccl      	# Disable NVIDIA NCCL support.
Configuration finished
```

### 3. 编译

使用bazel进行编译：

```
bazel build //mcts:mcts_main
```

经过漫长的编译，最后终于build完成。

### 4. 下载并展开模型

```
$ wget https://github.com/Tencent/PhoenixGo/releases/download/trained-network-20b-v1/trained-network-20b-v1.tar.gz
$ tar xvzf trained-network-20b-v1.tar.gz
```
模型及权重值会展开到ckpt目录下。

### 5. 运行PhoenixGo

可以运行如下命令做一个简单的测试：

```
$ scripts/start.sh 
```

如果没有什么错误提示，就万事大吉，当然这个命令也没有任何用户界面，也无法对其进行操作。接下来就要请出GoReviewPartner。

## GoReviewPartner配置及使用

### 配置

GoReviewPartner采用Python语言编写，理论上只要有Python运行时环境就可以运行，但要注意的是GoReviewPartner支持的是Python 2,在Python 3下可能存在兼容问题。通过anaconda可以创建一个python 2.7的python虚拟环境。运行命令非常简单：

```
(py2) alex@alex-550-279cn:~/work/ai/gochess/goreviewpartner$ python main.py
```

界面如下：

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/phoenix_go_02.png)

在使用之前需要配置AI引擎，如下图，选择PhoenixGo：

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/phoenix_go_01.png)

里面有好多个示范profile，分别对应不同的配置，我选择了"Example-linux-GPU-notensorrt-Slow"，然后根据实际情况进行修正,几个框中给的值如下:

Profile: Example-linux-GPU-notensorrt-Slow

Command: /home/alex/work/ai/gochess/PhoenixGo/bazel-bin/mcts/mcts_main

Parameters: --gtp --config_path=/home/alex/work/ai/gochess/PhoenixGo/etc/mcts_1gpu_notensorrt_grp.conf --logtostderr --v=1

Time per move (s): 60

然后点击**Modify profile**，记得一定要点这个按钮，否则修改的值不会保存！然后可以点击**Test**按钮进行启动测试。如果有提示：

```
Error reading "ckpt/checkpoint": No such file or directory [2]
F1030 22:31:01.981541 15943 mcts_engine.cc:369] Check failed: ret == 0 (-1000 vs. 0) EvalRoutine: model init failed, ret -1000
```

这是因为PhoenixGo的配置文件中模型路径中使用了相对路径，现在是从GoReviewPartner的当前路径启动PhoenixGo，在PhoenixGo那边路径就不对，解决的方法是修改PhoenixGo下的etc/mcts_1gpu_notensorrt_grp.conf文件，将其中的：

```
model_config {
    train_dir: "ckpt"
}
```
修改为PhoenixGo的ckpt路径，比如：

```
model_config {
    train_dir: "/home/alex/work/ai/gochess/PhoenixGo/ckpt"
}
```

再进行测试，界面如下：

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/phoenix_go_07.png)

### 使用

从主界面上，我们可以看出有很多复盘的方法，比如加载对局SGF文件，分析完成后，还可以保存为RSGF文件，避免每次加载都需要动用AI引擎进行分析（比较耗时），当然我们还可以进行实时对弈分析，就像大赛解说员那样，现场每下一步棋，输入到GoReviewPartner上，然后AI给出推荐的选点及胜率。选择**Run a live analysis**:

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/phoenix_go_03.png)

**Black player**和**White player**都选择**Human player**，其它就用默认值好了。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/phoenix_go_04.png)

在棋盘上点击，输入选手下的棋，等上一会儿（取决于GPU的算力），会出现一个**Start the review**按钮，点击这个按钮，就可以看出每个步骤的应对推荐选点：

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/phoenix_go_05.png)

将鼠标移到推荐的选点上（红色或蓝色的数字圈圈），还可以看出后续几步的走法：

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201910/images/phoenix_go_06.png)

## 小结

关于PhoenixGo和GoReviewPartner的使用就先介绍到这儿， 其实PhoenixGo还有很多参数可以调，GoReviewPartner的玩法也有好多种，这个就留给大家慢慢摸索吧。

除了可以对对局进行复盘，通过实时分析，它也能成为我们下棋的好帮手，下次碰到棘手的局面时，不妨搬出这套AI，不仅可以打败对手，还可以学学AI的招法哦！

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)