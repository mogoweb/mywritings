# 专门针对 DeepSeek 的纯 C++ CPU 端推理框架  

在介绍这个项目之前，先声明几点：  

1. **项目尚未成熟** —— 作者明确表示，这是出于兴趣和好奇心开发的。  
2. **对内存需求极高** —— 例如，运行 DeepSeek V3 F8E5M2 版本大约需要 650GB 内存，在尝试之前请掂量一下你的设备。  

尽管如此，这个项目依然值得关注：  

- **纯 C++ 实现**，不依赖任何第三方框架，便于集成，同时也能部署到低端终端设备。  
- **代码量不到 2000 行**，对研究推理框架的开发者而言，这无疑是一个极简、高效的参考。

这个项目就是 **deepseek.cpp**，项目地址：  
> [https://github.com/andrewkchan/deepseek.cpp](https://github.com/andrewkchan/deepseek.cpp)  

## 为什么要关注 deepseek.cpp？  

随着硬件的不断升级，软件却变得越来越庞大、复杂，资源消耗也日益增加。DeepSeek 的崛起给我们带来了一些新的思考：绕过复杂的框架，直接调用底层 API，可以显著降低硬件需求。DeepSeek 通过直接使用 Nvidia 的 PTX 进行底层硬件控制，绕过了部分 CUDA 层的限制，实现了对 GPU 资源的高效利用，从而做到对大模型性能的极致优化。  

除了大模型，推理框架（Inference Framework）在目前大语言模型（LLM）部署中也存在较大的优化空间。大部分开发者使用的推理框架多采用 Python 或 JavaScript，虽然通用性更强，但在极限性能追求下，C++ 方案逐渐崭露头角，例如 **llama.cpp**。不过，随着 **llama.cpp** 逐步支持更多模型，代码量迅速膨胀，如今已超过 25 万行。deepseek.cpp 的作者正是看到这一点，决定专门为 DeepSeek 量身打造一个极简优化的推理引擎 —— 牺牲部分通用性，换取极致性能。  

## 为什么开发这个项目？  

作者的初衷很简单：**好玩、学习！**  

他最初尝试在 **yalm** 中添加 DeepSeek 支持，但发现改动过于庞大，影响了项目的简洁性。最终，他选择将这些改动拆分，创建一个更轻量、更精简的代码库。  

此外，deepseek.cpp 还特别适用于低端 CPU 设备，因为它不依赖 Python 运行时，相比其他推理引擎，代码体积更小（除掉 fmt 和 json 的代码量小于 2 千行）。  

## 支持的 DeepSeek 模型  

| Model | INT4 | F8E5M2 | F8E4M3 | FP16 | BF16 | FP32 |  
|-------|------|--------|--------|------|------|------|  
| DeepSeek-V2-Lite | 进行中 | ✅ | 进行中 | ✅ | 进行中 | ✅ |  
| DeepSeek-V2 | 进行中 | ✅ | 进行中 | ✅ | 进行中 | ✅ |  
| DeepSeek-V2.5 | 进行中 | ✅ | 进行中 | ✅ | 进行中 | ✅ |  
| DeepSeek-V3 | 进行中 | ✅ | 进行中 | - | - | - |  
| DeepSeek-R1 | 进行中 | ✅ | 进行中 | - | - | - |  

其中：

- INT4: 指的是使用 4 位整数来表示数值。由于每个数只占 4 位，相比 FP16 或 FP32 表示而言，存储空间大幅减少，可以将模型参数的存储压缩为原来的1/4甚至更低。使用 INT4 量化后，数据传输速度和缓存效率都会提升，不过代价是数值表示的精度和动态范围受限，因此需要精细设计量化方案（例如通过量化感知训练或后训练量化校准）以减少精度损失。
- F8E5M2: 指的是一种 8 位浮点数格式，用于量化模型权重或激活。在这种格式中，“F8”表示总共使用 8 位存储数据，而“E5M2”则说明其中有 5 位用于表示指数（Exponent），2 位用于表示尾数（Mantissa），剩下一位通常用作符号位。这种 FP8 格式（另一个常见的变体是 F8E4M3）在深度学习中被用来降低模型的内存占用和计算带宽需求，同时在一定程度上保持数值的动态范围，适用于推理阶段的量化优化。

- F8E4M3: F8E4M3 是一种 8 位浮点格式，与之前提到的 F8E5M2 类似。具体来说，F8E4M3 表示使用 8 位总宽度，其中 1 位用于符号，4 位用于指数（Exponent），3 位用于尾数（Mantissa）。这种格式旨在在保持一定数值动态范围的同时，降低表示精度带来的存储和计算开销。与 F8E5M2 相比，F8E4M3 会牺牲部分指数范围（动态范围），但可能在特定应用中能够获得更好的硬件实现效果或更低的能耗。使用 F8E4M3 量化可以进一步降低显存占用，同时依然允许一定程度的浮点运算优势，是一种在追求极致性能优化时可能选用的方案。

## 进展情况  

目前，如果将 F8E5M2 指定为数据转换类型，模型权重会采用 128×128 块量化，而 MoE 门控单元和层归一化仍保持全精度。这种方法相比简单截断量化（例如 yalm 采用的策略）能提供更好的精度，否则 DeepSeek 模型的输出可能会失去意义。  

不过，该模型仍然存在一些问题，例如在低温度下容易陷入无限循环。测试表明，温度设置在 ~1.0 时能够有效避免这种情况，并保持合理的生成质量。  

当前 deepseek.cpp 尚未实现 DeepSeek V3 的部分可选架构优化（例如专家选择的 noaux_tc 方法），因此推理精度可能略低于官方实现。此外，模型运行仍然需要大约 650GB RAM，未来计划加入更激进的量化方法，例如 INT4 或 1.58-bit 量化。  

目前，该框架仅支持解码阶段（即逐步生成 token），不支持预填充（一次性读取整个 prompt）。此外，预填充优化技术（如推测解码、多 token 预测）尚未实现。同样，DeepSeek-V2 论文中提到的 **多潜在注意力优化** 也尚未完全实现。  

---

建议爱好钻研的同学可以关注一下，至于纯 CPU 推理对于内存要求过高的问题，以后也将不是问题，毕竟内存比 GPU 更容易造。