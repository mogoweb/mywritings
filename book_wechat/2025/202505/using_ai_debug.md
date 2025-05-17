# 用 AI 解 AI 写的代码 BUG，我的一次 AI 辅助编程实践

我对 AI 的接触算是比较早的，也比较能接受 AI 作为辅助工具。 ChatGPT 出现后，我就使用 ChatGPT 指导我的日常工作，开始的时候是当做搜索引擎在使用。随着 AI 的进化，能力越来越强，我也会让 AI 帮我写一些代码。不过到目前为止，仅限于编写代码片段。比如以 Chat 模式让 C++ 写一个 JSON 解析的函数，然后将代码复制到项目中。修改代码也是将代码片段复制到 Chat 窗口，AI 将代码修改好后，再复制回去。

从去年开始，AI Agent 开始火起来，到今年，又有 MCP Server，让 AI 不仅仅是一个 Chat 工具，而是可以直接下达指令，让 AI 自行完成工作。比如编程方面，可通过 AI Agent，创建代码目录结构，编写代码，写入文件，编译并运行，不需要人来操作。修改代码也是如此，可以将项目文件加载进来，指示 AI 修改代码，AI 会自动找到代码所在的文件，修改、编译、测试。

虽然现在 AI Agent + MCP Server 非常强大，但我在工作中没有采用。因为我现在主要基于 Chromium 开发浏览器产品，Chromium 代码及其庞大，有上千万行代码，我不觉得现在的 AI 水平能够进行这么大的上下文。前段时间看到有一个 DeepWiki 的项目，对开源代码进行分析，但这个分析比较像是静态的，分析过后将结果缓存起来，而不是每次用户请求就启动分析。

不过最近有个事情，让我实践了一把 AI 辅助编程。有一个浏览器插件，在测试过程中发现有点小问题，正好可以用来实践一把 AI 编程。一是因为这个插件比较小，功能相对比较简单，代码总计只有几千行，可以用 AI 来处理。二是我本人虽然懂一点前端知识的皮毛，但看到这个插件的代码还是头疼，光 CSS 样式就写了三千多行，现在去啃前端知识，有点力不从心。

AI 编程工具很多，几乎各大 AI 公司都在做，国内有字节的 TRAE、阿里的通义灵码、百度的文心快码、腾讯的 CodeBuddy 等等，功能大同小异。我这次选择的是腾讯的 CodeBuddy，选它是正好看到有一篇公众号文章介绍它，并不是说它比其它公司的产品好。

CodeBuddy 结合 VS Code 使用，可以上官网查看说明。其实步骤就一步，从 VS Code 插件商店下载 CodeBuddy，第一次使用需要登录，使用微信账号登录就可以使用，其它的无须任何配置，只使用默认的设置即可。

![CodeBuddy](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_01.png)

CodeBuddy 界面上方有 Craft、Chat、Code Review、Unit Test 四个标签页。Crasft 就是 AI Agent，搭配 MCP Server 使用，虽然这个界面看起来还是 Chat，但这里是可以直接下达指令写代码、修改代码的。MCP Server 可以选择，但我现在也无法判断哪一个更好，就没有选，直接用默认的。Chat 标签页是我们最熟悉的 Chat 模式。至于 Code Review 和 Unit Test 可以从字面上理解其用途，这次实践没有用到，等后面使用了，再和大家分享。

## 一、解决插件获取网站的 favico 为空图片的问题

没有看插件的源码，仅仅是通过测试，就发现有些网站的图标会显示成透明图标。为此，我向 CodeBuddy 发出了提问：

> 代码中获取网站favicon图标的逻辑是否正确？

然后 CodeBuddy 就进行了分析，以下是 CodeBuddy 的分析及回复。（回复很长，这里只截取部分）

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_02.png)

可以看出，AI 在读取源码，并进行了分析。看分析过程，也可以了解程序的逻辑，这对于我这种刚接手这个插件，且对前端知识不熟的人很有用。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_03.png)

和 Chat 模式不同之处在于，Craft 模式下，AI 是会直接修改代码的。看看下图的效果，有没有黑客帝国的感觉？

<视频>

代码修改之后，会出现一个对比界面，查看 AI 修改了哪些内容，你可以选择接受或拒绝。

我发现有一点比较有意思，就是如果你没有选择接受，AI 可能会继续思考，继续修改，直到 AI 不知道再怎么修改了。不要认为 AI 多次修改后会更好，我发现第一次修改的范围比较小，基本上已经完成了所需功能，结果经过几次迭代，修改的范围更大，反而出问题了。就如同下面的结果，加了太多优化，反而不是我所需要的。

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

至此，这个问题终于得到了解决。在这个过程中，也是不断调整指令，给出信息，全程我都没进行代码分析，只是使用修改的代码进行测试，看是否达到预期。

关于 AI 的思考过程，还是有些不太明白。AI 似乎也有 Favicon API 的知识，但是在前面的修改方案中，始终没有想到这个，还是我主动提醒它之后，才想到使用这个方案。

## 二、解决插件可能出现的线

这个问题不好描述，而 CodeBuddy 还不支持上传图片和文件，现象就是下面所显示的。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/using_ai_debug_06.png)

为此，我向 AI 进行了如下指示：

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

经过这次实践，我体会到了 AI 的强大，它能够分析代码，给出修改建议，甚至能够直接修改代码。不过，目前 AI 还没有强大到能替代程序员的程度，在大型项目中，还是不太可能直接让 AI 来直接编写代码和修改代码，最多写一下代码片段。只是 AI 的发展速度太快了，也许在不久的将来，AI 就能替代大部分程序员，那个时候我也该退休了。