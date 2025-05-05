# 这个五一假期，借助 AI，我将国家商用密码算法移植到了 BoringSSL

首先得说明一点，借助 AI，并不是说给 AI 下达一个指令，说一句提示词：“将国家商用密码算法移植到 BoringSSL”，就能完成移植工作的。正如《人月神话》一书中所说的“没有银弹”（No Silver Bullet），软件开发中没有一种单一技术或方法能像银弹一样彻底解决所有难题​​。AI 取代程序员是必然发生的事情，但不是现在。

实现国家商用密码的 SSL 库很多，大约四五年前，我做国密算法移植时，参考的是 GmSSL（https://gmssl.org）项目和 TASSL 项目。不过我又发现一个更好的开源项目，那就是铜锁（https://tongsuo.net）。铜锁（Tongsuo）是由蚂蚁集团开源并捐赠至开放原子开源基金会的基础密码库，前身为BabaSSL，是我国首个通过国家密码管理局商用密码产品认证的开源密码学项目（符合GM/T 0028安全一级标准）。

Tongsuo 被应用在阿里巴巴的多款商业产品中，质量有保障，而且还通过了国家密码管理局的商用密码产品认证，能够保证权威性和兼容性。所以在移植到 BoringSSL 项目中，参考的是铜锁项目。

在《[不要重复发明轮子！谷歌：我偏要]()》一文中，有介绍过 BoringSSL，这是谷歌从 OpenSSL fork 出的一个 SSL 库。但是经过谷歌的魔改，已经和 OpenSSL 大不一样，所以铜锁项目虽然和 BoringSSL 同宗，但移植起来不那么容易。

第一步，先使用 DeepWiki 分析铜锁的国密实现。

DeepWiki 是一个分析源码的网站，收录了几乎所有的 github 项目，当然也可以添加自己的项目。在首页中搜索 tongsuo，就可以找到该项目。

![DeepWiki](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_01.png)

由于铜锁项目已经被 DeepWiki 收录，所以打开这个项目的源码分析速度非常快，不用等待。我比较关心的是国密算法和通信流程的实现，这一点 DeepWiki 做得非常棒，有专门的一章分析国密算法（NTLS）的实现，画的流程图也非常专业。

![NTLS实现分析](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_02.png)

非常遗憾的是，该网站只支持英文，可以借助翻译软件或 AI，转成中文文档。下面是 DeepWiki 生成的 NTLS 架构图，有了这张图，就可以对国密算法的实现有一个总体的理解。

![NTLS架构图](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_03.png)

下图则是和 SSL/TLS 类似的握手流程。

![NTLS握手过程](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_04.png)

在阅读了这篇文档后，可以对铜锁项目中国密算法的实现有一个总体的理解，根据文档的说明，能找到对应的代码段，进行进一步的分析。如果你对项目还有什么疑问，还可以在下面的 chat 框中进一步提问。

![针对项目提问](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_05.png)

勾选**Deep Research**，类似于 DeepSeek R1 的深度思考模式，回答问题的速度会慢一些，而且会显示思考的过程。

![铜锁实现的国密套件](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_06.png)

第二步，使用 DeepWiki 分析 BoringSSL

同样，使用 DeepWiki 分析 BoringSSL 开源项目，BoringSSL 虽然是托管在谷歌的代码服务器上，但在 github 上有镜像站点，所以可以直接搜索到。

![BoringSSL](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_07.png)

向 DeppWiki 提问：我想为BoringSSL增加国密算法（SM2、SM3、SM4）实现，以及增加 ECDHE_SM4_CBC_SM3 密码套件，该如何实现？

![BoringSSL](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_08.png)

DeepWiki 给出的答案，左边是实现要点，右边则是具体的源码文件和代码行。虽然不是完全正确，但至少提供了一个思路，知道具体需要修改哪些文件。

做了前面的准备工作后，就可以动手进行移植。移植过程当然不会是那么顺利，特别是在调试 TLCP 统信过程，需要对照协议，分析 client 和 server 之间的通信数据。这个过程可以借助 wireshark 进行抓包分析。最新版本的 wireshark 已经支持 TLCP 协议的分析，使用起来方便很多。

![wireshark](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_09.png)

整个交互过程还是比较清晰的，碰到 wireshark 无法识别的数据格式怎么办？比如上图的 Server Key Exchange，就提示 “Cipher suite not implemented”，这个时候有可以借助 AI 来分析。现在各种 AI 非常多，但总体尝试下来，对 Chromium 项目源码理解最好的是 ChatGPT，所以这里使用 ChatGPT 进行数据分析。从 WireShark 中复制 TLCP 报文，然后向 ChatGPT 提问。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_10.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/poring_ntls_11.png)

ChatGPT 对报文的分析非常详实准确，在调试过程中，可以对照代码，如果碰到哪次交互无法通过，分析起来就方便多了。

小结

在本次移植过程中，AI 只是作为一种辅助工具，帮助我分析代码，找到修改的关键点，并在调试上提供分析帮助，但离自主智能还差得远。现在也有不少的 AI Agent，能够实现提出需求，然后自动写代码的功能，但是对于大型复杂的项目来说，效果并不是很好，所以说 AI 替代程序员，还没那么容易。程序员也不用提心吊胆，害怕被 AI 取代。我更多的是把 AI 视作帮手，的确，有了 AI 后，开发的效率提高了很多。

最后，项目已经开源，项目地址：https://github.com/mogoweb/mojo-ssl。由于时间仓促，SM4 只实现了 CBC 分组模式，SM2 只实现了签名和验签，密码套件只实现了 ECDHE_SM4_CBC_SM3。后续会完善加密算法，实现更多的密码套件。

