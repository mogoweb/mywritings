# AI 也会写代码了，但我并不担心

如果你比较关注人工智能，可能会注意到最近圈子的人都在刷屏一个 AI 玩意，叫 ChatGPT 。我一直关注的和菜头在他的公众号**槽边往事**上就接连写了几篇文章。

* [为什么和菜头是男的，读者却要叫他“婶婶”](https://mp.weixin.qq.com/s/WiutcxdJv6pw2cIuBXsaVQ)
* [怎么把AI逼到生气](https://mp.weixin.qq.com/s/OQfmtMmupr_GeBCRZgxSag)
* [水文与干货](https://mp.weixin.qq.com/s/3u9KfqhskEWj0-fq8Xi0BQ)

如果看完上面的第一篇文章，得出结论是：**这不就是 siri、小度，或者京东智能客服吗？**

那我要建议你看第二篇文章，在这篇文章中，作者不断给出提示，让 AI 创作故事，这让我恍惚间有些错觉，AI 也有思想了。

在第三篇文章中，和菜头写道：

> 在我看来， AI 绘画， AI 作曲， AI 写文章，这不是什么杂耍、玩意儿、新奇趣，而是未来的影子。 AI 所谓模仿人类智能的那一部分，也就是人做造物主创造智慧的那一部分，大概还有很长的路要走，现时的思路和方法都可能不对。但是此时此刻， AI 作为人类助手，作为辅助工具的那一部分，已经逐渐成熟，在垂直领域，在专业领域会有大量的应用。

当然，ChatGPT 并非什么新物种，只是属于 NLP（自然语言处理）的一个小分支，目前市场上的智能音箱、语音助手、人工智能客服、聊天机器人，都属于 NLP 这一范畴。但它又和现有的聊天机器人有所不同，相当于 Alpha Go Zero 和 Alpha Go 之间的差距。

ChatGPT 是一种自然语言生成的聊天机器人模型，由 OpenAI 开发，它能够根据用户输入的文本内容，自动生成新的文本内容。它的名称来源于它所使用的技术—— GPT-3 架构，即生成式语言模型的第 3 代。

ChatGPT 的技术原理是基于深度学习和神经网络模型，通过大量的训练数据，学习人类语言的表达方式和语法规则，从而能够模拟人类语言的生成过程。通过这种方式，ChatGPT 可以较为自然地生成文本内容，并提供人机对话和自动回复等功能。

目前关于 ChatGPT 的文章不算多，我比较关注的是，请求 ChatGPT 写一段代码，会是怎样的结果？

先让 AI 写一个入门的 python 程序吧！

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202212/images/ai_can_program_01.png)

效果还不错，上点难度如何？

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202212/images/ai_can_program_02.png)

似乎也难不倒 ChatGPT。

实际上，如果代码是使用高度受规则约束的语言（如查询语言），它可以做得很好。 比如为 Wikidata 构建 SPARQL 查询（如果 Wikipedia 的知识图谱没有深入了解，很难编写。）

看看下面这个例子，相当完美，甚至还提供了代码注释来解释查询的各种元素：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202212/images/ai_can_program_03.png)

从头开始生成代码也不在话下，不如这样：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202212/images/ai_can_program_04.png)

将函数直接翻译成其他语言也没有问题：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202212/images/ai_can_program_05.png)

在网上，已经有很多 ChatGPT 编码能力的例子，确实令人刮目相看，但是 ...

ChatGPT 在逻辑思考方面的能力极其有限，它会出现许多事实错误，并且无法确定其论点何时在逻辑上不一致。 下面这个例子中，它在对一个问题的回答中就存在自相矛盾：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202212/images/ai_can_program_06.png)

在第一个解答中，推断出 m = n = 2，继续追问下去，又得出 m = n = 2 不是答案的结论。

此外，受过训练的 LLM 最大的问题之一是，他们受训的时间和被释放的时间之间存在很大的差距。 比如 ChatGPT 使用的是 2021 年之前的数据，也就是两年前的进行训练，这样如果你问及这两年发生的事情，它就不知道。从目前的测试的情况来看，没有理由相信 AI 真正具有思想，最多是对现有知识的一个汇总和总结。

程序员经常自嘲自己是靠 CTRL + C 和 CTRL + V 写代码，这方面人工智能倒是有先天优势，都不用借助搜索引擎，直接就可以给出答案。但自嘲归自嘲，写代码主要还是一个逻辑思考的过程，现在就担心人工智能写出逻辑严谨、功能完善的代码，为时尚早。我倒是觉得现在的程序员有福了，也许不久的将来，我们只需要说一声：小度小度，帮我写一个 xxx 的函数，连搜索、复制粘贴的步骤都可以省略 ...