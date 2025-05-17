# 用 AI 解 AI 写的代码 BUG：一次 AI 辅助编程实践

我对 AI 的接触算是比较早的，也比较能接受 AI 作为辅助工具，并不会因为 AI 可能会取代我们的工作而排斥。自从 ChatGPT 问世以来，我便将它当作“智能搜索引擎”来使用。随着 AI 能力的不断迭代，我开始让它帮我生成代码片段。例如，我会请它用 C++ 实现一个 JSON 解析函数，再将生成的代码复制到项目中；当要修改现有逻辑时，也同样把片段粘到 Chat 窗口，待 AI 给出修改后再拷回项目。

从去年起，AI Agent 概念走红；到了今年，MCP Server 更进一步，让 AI 不再仅是聊天工具，而是能自主接收、执行指令的“编程小伙伴”。借助 Agent + MCP，理论上可以自动创建工程目录、编写并写入文件，甚至完成编译、运行和测试，全程无需人工干预。

虽然现在 AI Agent + MCP Server 非常强大，但我在工作中没有采用。因为我现在主要基于 Chromium 开发浏览器产品，Chromium 代码及其庞大，有几百万行代码，我不觉得现在的 AI 水平能够进行这么大的上下文。虽然后来出现了如 DeepWiki 之类对开源代码做静态索引的项目，它们也只是先行分析再缓存结果，并未实现实时“随查随用”。

但最近，我还是找了个“小而美”的插件实践了一把 AI 辅助编程。挑它有两点原因：一来该插件代码规模仅数千行，功能简单；二来我对前端只略知皮毛，面对数千行的 CSS 也颇感头疼，这正好让我借助 AI 来“扫盲”。

我选用的是腾讯的 CodeBuddy——并非它最强，而是我偶然看到一篇公众号介绍。安装只需在 VS Code 插件市场搜索“CodeBuddy”，之后登录微信账号即可使用，无需任何额外配置。

![CodeBuddy](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_01.png)

CodeBuddy 界面上方依次有 Craft、Chat、Code Review、Unit Test 四个标签页。Crasft 就是 AI Agent，搭配 MCP Server 使用。虽然这个界面看起来还是 Chat，但这里是可以直接下达指令写代码、修改代码的。MCP Server 可以选择，但我现在也无法判断哪一个更好，就没有选，直接用默认的。Chat 标签页是我们最熟悉的对话模式。至于 Code Review 和 Unit Test 可以从字面上理解其用途，这次实践没有用到，等后面使用了，再和大家分享。

## 一、修复 favicon 显示为空图标的问题

在测试中，我发现部分网站的图标显示成透明。没有去研究源码，而是向 CodeBuddy 发出了提问：

> 代码中获取网站favicon图标的逻辑是否正确？

CodeBuddy 立刻对插件源码做了逐行分析，并给出了诊断。（回复很长，这里只截取部分）

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_02.png)

可以看出，AI 在读取源码，并进行了分析。通过它的解读，我迅速理解了原逻辑：它依次尝试 Yandex 等第三方接口，若都失败则返回默认图标。这对于我这种刚接手这个插件，且对前端知识不熟的人很有用。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_03.png)

和 Chat 模式不同之处在于，Craft 模式下，AI 是会直接修改代码的。看看下图的效果，有没有黑客帝国的感觉？

<视频>

代码修改之后，会出现一个对比界面，可以查看 AI 修改了哪些内容，你可以选择接受或拒绝。

我发现有一点比较有意思，如果你没有选择接受，AI 可能会继续思考，继续修改，直到 AI 不知道再怎么修改了。不要认为 AI 多次修改后会更好，我发现第一次修改的范围比较小，基本上已经完成了所需功能，结果经过几次迭代，修改的范围更大，反而出问题了。就如同下面的结果，加了太多优化，反而不是我所需要的。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_04.png)

这里希望 CodeBuddy 做一些改进，在我还没有点接受或拒绝时，不要再继续分析，因为检查修改内容需要时间，往往我检查完一个修改，结果 AI 已经迭代 3 个版本出来了。

当然，在使用 AI 的时候，一定要当心" AI 幻觉"，比如在优化代码的时候，AI 又给我加上了新的建议：

> 在中国，Google和Yandex服务可能不可靠，而iowen.cn是第三方服务。更好的解决方案是使用 Chrome 内置的chrome://favicon/服务，这会更稳定且不需要依赖外部服务。
>
> 修改方案：
>
> 1. 移除对Google/Yandex等外部服务的依赖
> 2. 使用Chrome内置的favicon服务
> 3. 保留原有的缓存机制
> 4. 添加对chrome://协议的支持检查

实际上 Chrome 内置的 chrome://favicon/ 服务是有问题的，而应该使用 Chrome Extensions 提供的 Favicon API。这个是使用 ChatGPT 时，GPT 给我的一个信息。将这个信息给到 CodeBuddy 后，CodeBuddy 马上就领会到了，立即帮我修改了代码。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_05.png)

至此，这个问题终于得到了解决。在这个过程中，我不断调整指令，给出信息，全程我都没进行代码分析，只是使用修改的代码进行测试，看是否达到预期。

关于 AI 的思考过程，还是有些不太明白。AI 似乎也有 Favicon API 的知识，但是在前面的修改方案中，始终没有想到这个，还是我主动提醒它之后，才想到使用这个方案。

## 二、修复页面中偶先的竖线

这个问题不好描述，而 CodeBuddy 还不支持上传图片和文件，现象就是下面所显示的。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_06.png)

为此，我向 AI 发出了如下指示：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_07.png)

看着 AI 分析得头头是道，然后一顿操作猛如虎，经过反复迭代的代码显示效果是这样的：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_08.png)

其实中间几个版本还没有这么离谱，只是在我验证结果的时候，AI 在疯狂迭代，然后就出来这么一个结果。

没有办法，直接拒绝掉，从头来。这次，我决定更详细的描述错误现象。

> 那根竖线显示在正中间，颜色是body的背景色，请问如何解决

这次 AI 分析得比较仔细，很快就给出了修改方案：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_09.png)

修改很小，但是解决了问题。

## 三、小结

这次还解决了一些其它的问题，比如调整窗口大小时，搜索历史下拉框没有跟随窗口变化，等等。限于文章篇幅，所以就不一一描述了。在整个过程中，我忍住去查看源码的冲动，而是完全让 AI 去分析和解决问题，就是想完整体验一下 AI 的能力。花了半天实践，修复了 3 个问题，对于新手我来说，效率还是挺高的，但比起资深的前端开发人员，估计还是差了点。

经过这次实践，我体会到了 AI 的强大，AI 辅助编程已展现出惊人的潜力：它能理解上下文、分析逻辑、给出修改意见，甚至直接动手“改刀”。不过，目前 AI 还没有强大到能替代程序员的程度，在大型项目中，还是不太可能直接让 AI 来直接编写代码和修改代码，最多写一下代码片段。只是 AI 的发展速度太快了，也许在不久的将来，AI 就能替代大部分程序员，那个时候我也该退休了，那时程序员或许该考虑升级自己的生产力角色了。

