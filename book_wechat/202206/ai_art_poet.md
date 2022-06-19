# 根据中国古诗词作画，AI 可以做到吗？

AI 作画工具 DD （Disco Diffusion）面市后，不少大神做出了惊艳的作品。玩家以外国人居多，网上的资料也是英文资料较多。现在国内也有人尝试，比如和菜头在他的「槽边往事」微信公众号就写了好几篇关于 AI 作画的文章，现在他的公众号配图也使用自己生成的图。西乔在她的「神秘的程序员们」微信公众号也写了好几篇教程，值得一看。

这段时间我也尝试了一些 AI 作画，但做出的画总是不太理想。作为一名程序员，其实我关注的并不是做出艺术性比较强的作品，而是想探索一些可能，也就是探索让 AI 做出一些比较新奇的东西出来。在「[AI 能理解和表达古诗意境吗？来看看西乔用人工智能辅助创作的古风插画（第一辑）](https://mp.weixin.qq.com/s/oMQ_HR1dJQXLhXv-9qwizA)」这篇文章中，作者做出了效果非常不错的古风插画，其插画也比较贴近诗词的意境，但作者并没有说明其作画的参数，使用了怎样的提示句子（prompts）。看到这些作品后，我就在思考，能否直接根据古诗词来作画呢？

下面就说说我的尝试过程，虽然最后的结果不太满意，但过程还是值得记录一下。

DD 作为一款外国人开发的软件，并不支持中文。 text prompts 必须输入英文，对于国内用户来说，可以借助**谷歌翻译**（或其它翻译软件），先将中文翻译成英文，然后在送给 DD。所以我首先想到的是如何将翻译功能加入到 DD 中。

搜索了一下 Python 的机器翻译资料，发现了 translate 这个 Python 包。 translate 提供的是一种云翻译方式，集成了几家云翻译的产品，包括微软、MyMemory、LibreTranslate。但是试用下来，效果并不理想。这三家功能提供者虽然都提供了免费的使用接口，但对于调用次数、调用频次都有要求。比如我在使用 MyMemory 的云翻译功能时，就碰到开始使用得好好的，突然之间就不能翻译的情况，然后过了一段时间，有可以使用的情况。

于是我就寻找本机翻译的软件，好在随着人工智能的发展，机器翻译的质量也越来越高。经过一番搜索，找到了 Huggingface Transformers 。这款软件能够做的事情非常多，包括文本分类、文本生成、自动问答、文本翻译、自动摘要等等。这里我只用到了其中的文本翻译功能。

Huggingface Transformers 使用了一种非常开放的架构，让更多的人参与进来，并提供了许多预训练的模型下载。这样，对于普通用户而言，并不需要过多的人工智能知识，也不需要经过复杂而繁琐的模型训练，把它当做一个黑盒，集成到软件中即可。比如，我为 DD 增加的中文翻译成英文功能，就这么几行代码：

```python
from transformers import AutoModelWithLMHead, AutoTokenizer, pipeline

chinese_prompts = "碧绿的莲叶无边无际，一直延伸到水天相接的远方，在阳光的照映下，荷花显得格外艳丽鲜红。"

mode_name = 'liam168/trans-opus-mt-zh-en'
model = AutoModelWithLMHead.from_pretrained(mode_name)
tokenizer = AutoTokenizer.from_pretrained(mode_name)
translation = pipeline("translation_zh_to_en", model=model, tokenizer=tokenizer)
translate_result = translation(chinese_prompts, max_length=400)
translated_text = translate_result[0]['translation_text']
print(translated_text)
```

在这段代码中，模型选择了 liam168/trans-opus-mt-zh-en，第一次运行的时候，会从网上下载，非常方便，当然也可以使用事先下载好的预训练模型。

要使用 Huggingface Transformers，请事先安装如下 python 包：

```
pip install transformers==4.4.2 datasets==1.6.2 sklearn scipy matplotlib torchtext seaborn spacy sentencepiece
```

下面看看使用古诗词创作的画：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_poet_01.png)

**海上生明月，天涯共此时** (The sea is full of the moon, and the world is full of it.)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_poet_05.png)

**枯藤老树昏鸦，小桥流水人家，古道西风瘦马，中国水墨画风格** (The old trees of the dead, the old trees of the dead, the little bridges of the water, the old wind of the past, the lean horses. China's ink-painting style.)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_poet_02.png)

**大漠孤烟直，长河落日圆** (The desert's straight, the river's setting sun full.)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_poet_06.png)

**白日依山尽，黄河入海流** (By day, by day, the yellow river flows into the ocean.）

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_poet_03.png)

**接天莲叶无穷碧，映日荷花别样红** (There's a lot of leaves in the sky, a lot of red.)

这里翻译出了一点问题，将英文翻译修改为：

The green leaves of the lotus, unbridled, extend to the distance between the water and the sky, and, in the light of the sun, the flowers look extraordinary and red.

做出的画作如下：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202206/images/ai_art_poet_04.png)

依据中国古诗词作画，面临着两道难题。首先是翻译到英文，可能意思相差太远，比如上面的**接天莲叶无穷碧，映日荷花别样红**翻译到英文，意思完全变了。其次，即使完整翻译成英文，但古诗词的意境如何表现出来，上面的**接天莲叶无穷碧，映日荷花别样红**这句诗，即使人工修改为比较贴近字面意思的英文，但做出来的画依然不太满意。

古诗词如何翻译成现代汉语都是一个难题，更别说翻译成英文，还有很长的一段路需要走，这也需要更多的中国工程师参与其中。此外，DD 中使用的训练数据可能中国的画作比较少，我尝试在 prompts 中加入中国著名画家的名字，没有什么效果。如果是用的梵高之类的外国画家，其生成画作的风格就很像这些画家的作品。

综合试用下来，根据古诗词作画依然困难重重，需要更多的中国人参与其中。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)