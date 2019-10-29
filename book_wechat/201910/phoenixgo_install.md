# 使用AI指导下围棋

作为一名围棋爱好者，业余时间会花相当的时间下围棋。围棋是属于那种入门容易精通难的体育活动，极费脑筋。惭愧的是，虽然在网上下棋下的多，但一直在1级和1段之间徘徊(腾讯围棋上的排位)。我也知道需要下功夫去记定式、做死活题，水平才能提高，但那太枯燥了，我还是喜欢上网厮杀，那种快感不是枯燥的训练所能得到的。下棋下得多了，经常会碰到那种不按套路的对手，有时明明觉得对方下了无理手，但就是不知道如何反击。有时碰到棋盘太空旷，或者对方围出了模样，不知如何选点，如何打入。这些虽然在书上可以学到一些基本技巧，但一到实战，往往不知如何下手。

有时上腾讯围棋，碰到大赛，会有讲解，通常解说会搬出**绝艺**（腾讯出品的围棋AI软件），对一些关键步骤，给出几种AI推荐的选点，以及随后几步的变化。当然，专业棋手的棋我也看不太懂，虽然看着热闹，实际上对平时对局也没有帮助。我就想，要是我也有绝艺就好了，对我以往下的棋进行复盘，了解对一些无理手的应对方法，以及一些选点提示，无疑可以提高实战水平。然而，上网搜索了一番，似乎**绝艺**只提供给专业棋手使用，普通人虽然也可以申请**绝艺**复盘，但有很多限制，特别是像我这种低水平的棋手，基本上申请不到。

我之前也研究过一些围棋AI软件，比如Leela、ELF OpenGo，也尝试过和AI对战，请参考我之前的一篇文章：

[想和围棋高手过招？火力更猛且开源的围棋AI来了...](https://mp.weixin.qq.com/s?__biz=MzI3NTQyMzEzNQ==&mid=2247484935&idx=1&sn=079cb5510e11fc48d0d5dbe831ca7213&chksm=eb044cf7dc73c5e13a8f7a98e1dbdad5166aee2768de95371cec89c787a7019618f46b7b0ca0&token=504277048&lang=zh_CN#rd)

但这并没有达到目的，我还是希望能够提升一下围棋水平，虽然带着AI，能够在网上大杀四方，那又有什么意义呢？我希望AI具有**绝艺**那样的复盘功能，实实在在提高一下我的围棋水平。

想到之前看到过一篇报道，腾讯开源了一款围棋AI，开始还以为是**绝艺**，仔细一搜索，原来是另外一款AI，曾经以**金毛测试**出现在腾讯围棋对战平台。

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

```
(base) alex@alex-550-279cn:~/work/ai/gochess/PhoenixGo$ scripts/start.sh 
current directory: '/home/alex/work/ai/gochess/PhoenixGo'
mcts_main was built with CUDA support
mcts_main was built with TensorRT support
found 1 GPU(s)
use config file 'etc/mcts_1gpu.conf'
log to '/home/alex/work/ai/gochess/PhoenixGo/log'
start mcts_main
E1029 08:10:59.333669  4649 trt_zero_model.cc:39] The engine plan file is not compatible with this version of TensorRT, expecting library version 5.1.2 got 0.0.0, please rebuild.
E1029 08:10:59.333832  4649 trt_zero_model.cc:91] load cuda engine error: File exists [17]
F1029 08:10:59.339411  4649 mcts_engine.cc:369] Check failed: ret == 0 (-2001 vs. 0) EvalRoutine: model init failed, ret -2001
*** Check failure stack trace: ***
*** Aborted at 1572307859 (unix time) try "date -d @1572307859" if you are using GNU date ***
PC: @                0x0 (unknown)
Aborted (core dumped)
```

```
(py2) alex@alex-550-279cn:~/work/ai/gochess/goreviewpartner$ python main.py
```

Example-linux-GPU-notensorrt-Slow

/home/alex/work/ai/gochess/PhoenixGo/bazel-bin/mcts/mcts_main

--gtp --config_path=/home/alex/work/ai/gochess/PhoenixGo/etc/mcts_1gpu_notensorrt_grp.conf --logtostderr --v=1