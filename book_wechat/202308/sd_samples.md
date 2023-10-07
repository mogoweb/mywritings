# 稳定扩散采样器：综合指南

AUTOMATIC1111 中提供了多种采样方法。 Euler a、Heun、DDIM……什么是采样器？ 它们如何工作？ 它们之间有什么区别？ 您应该使用哪一个？ 您将在本文中找到答案。

我们将讨论 AUTOMATIC1111 稳定扩散 GUI 中可用的采样器。 您可以在 Windows、Mac 或 Google Colab 上使用此 GUI。

### 什么是抽样？

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_sampler_01.png)

采样器负责执行去噪步骤。

为了生成图像，稳定扩散首先在潜在空间中生成完全随机的图像。 然后噪声预测器估计图像的噪声。 从图像中减去预测的噪声。 这个过程重复十几次。 最后，你会得到一个干净的图像。

这个去噪过程称为采样，因为稳定扩散在每个步骤中都会生成一个新的样本图像。 抽样所采用的方法称为抽样器或抽样方法。

采样只是稳定扩散模型的一部分。 阅读文章“稳定扩散如何工作？” 如果你想了解整个模型。

下面是一个正在运行的采样过程。 采样器逐渐产生越来越清晰的图像。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_sampler_02.gif)

虽然框架是相同的，但有许多不同的方法可以执行此去噪过程。 这通常是速度和准确性之间的权衡。

#### 噪音表

您一定已经注意到，嘈杂的图像逐渐变成清晰的图像。 噪声表控制每个采样步骤的噪声水平。 噪声在第一步最高，在最后一步逐渐降至零。

在每个步骤中，采样器的工作是生成噪声水平与噪声表相匹配的图像。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_sampler_03.png)

15 个采样步骤的噪声表。

增加采样步数有什么影响？ 每个步骤之间的噪音降低幅度较小。 这有助于减少采样的截断误差。

比较下面 15 步和 30 步的噪声表。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_sampler_04.png)

30 个采样步骤的噪声表。

### 采样器概述

截至撰写本文时，AUTOMATIC1111 中有 19 个采样器可用。 随着时间的推移，这个数字似乎在增长。 有什么区别？

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_sampler_05.png)

AUTOMATIC1111 中的采样器。

您将在本文的后面部分了解它们是什么。 技术细节可能令人难以承受。 所以我在本节中包含了鸟瞰图。 这应该可以帮助您大致了解它们是什么。

老式 ODE 求解器
让我们先剔除简单的。 列表中的一些采样器是一百多年前发明的。 它们是常微分方程 (ODE) 的老式求解器。

Euler – 最简单的求解器。
Heun – 欧拉的更准确但速度较慢的版本。
LMS（线性多步法）——与欧拉速度相同，但（据说）更准确。
祖先采样器
您是否注意到某些采样器的名字有一个字母“a”？

欧拉
DPM2a
DPM++ 2S a
DPM++ 2S 卡拉斯
他们是祖先的采样者。 祖先采样器在每个采样步骤向图像添加噪声。 它们是随机采样器，因为采样结果具有一定的随机性。

请注意，许多其他采样器也是随机采样器，尽管他们的名字中没有“a”。

使用祖先采样器的缺点是图像不会收敛。 比较使用 Euler a 和下面的 Euler 生成的图像。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_sampler_06.gif)

欧拉 a 不收敛。 （示例步骤 2 – 40）

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_sampler_07.gif)

欧拉收敛。 （采样步骤2-40）

使用 Euler a 生成的图像在高采样步长时不会收敛。 相比之下，欧拉的图像收敛得很好。

为了再现性，希望图像会聚。 如果你想产生轻微的变化，你应该使用变分种子。

#### 卡拉斯噪音表

带有“Karras”标签的采样器使用 Karras 文章中推荐的噪声表。 如果仔细观察，您会发现噪声步长在接近末尾时较小。 他们发现这提高了图像质量。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_sampler_08.png)

默认噪声表和 Karras 噪声表之间的比较。

DDIM 和 PLMS
DDIM（去噪扩散隐式模型）和 PLMS（伪线性多步方法）是原始稳定扩散 v1 附带的采样器。 DDIM 是最早为扩散模型设计的采样器之一。 PLMS 是 DDIM 更新、更快的替代方案。

它们通常被认为已经过时并且不再广泛使用。

DPM 和 DPM++
DPM（扩散概率模型求解器）和 DPM++ 是为 2022 年发布的扩散模型设计的新采样器。它们代表了一系列具有相似架构的求解器。

DPM 和 DPM2 类似，只是 DPM2 是二阶（更准确但速度更慢）。

DPM++ 是对 DPM 的改进。

DPM自适应自适应调整步长。 它可能会很慢，因为它不能保证在采样步骤数内完成。

统一电脑
UniPC（Unified Predictor-Corrector）是2023年发布的新采样器。受到ODE求解器中的预测器-校正器方法的启发，它可以在5-10步内实现高质量图像生成。

k扩散
最后，您可能听说过 k 扩散这个术语并想知道它的含义。 它只是指 Katherine Crowson 的 k-diffusion GitHub 存储库以及与其关联的采样器。

该存储库实现了 Karras 2022 文章中研究的采样器。

基本上，AUTOMATIC1111 中除 DDIM、PLMS 和 UniPC 之外的所有采样器都是从 k-diffusion 借用的。

图像融合
在本节中，我将使用不同的采样器（最多 40 个采样步骤）生成相同的图像。 第 40 步的最后一个图像用作评估采样收敛速度的参考。 欧拉方法将被用作参考。

Euler、DDIM、PLMS、LMS Karras 和 Heun
首先，让我们将 Euler、DDIM、PLMS、LMS Karras 和 Heun 作为一组来看看，因为它们代表了老式的 ODE 求解器或原始扩散求解器。 DDIM 收敛于欧拉的步长，但有更多变化。 这是因为它在采样步骤中注入了随机噪声。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202308/images/sdxl_sampler_09.png)

Euler、DDIM、PLMS、LMS Karras 和 Heun 的图像收敛（越低越好）。

