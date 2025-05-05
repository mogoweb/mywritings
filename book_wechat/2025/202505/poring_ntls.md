# 这个五一假期，我借助 AI 将国密算法移植到 BoringSSL

首先要说明：借助 AI 并不意味着只要在命令行里敲一句“请把国家商用密码算法移植到 BoringSSL”即可完成全部工作。正如《人月神话》所言，“没有银弹”（No Silver Bullet），任何单一技术都不可能一举解决所有难题。AI 替代程序员是大势所趋，但那一天，还并未到来。

## 为什么选铜锁（Tongsuo）？

早在四五年前，我做过国密算法移植的工作，当时是移植到 firefox 的 NSS 网络库。参考的是 GmSSL（https://gmssl.org）和 TASSL 两个项目。近期我发现了更优秀的开源项目——铜锁（Tongsuo，https://tongsuo.net）。

Tongsuo 原名 BabaSSL，是蚂蚁集团开源后捐赠给开放原子开源基金会的基础密码库，也是国内首个通过国家密码管理局商用密码产品认证（GM/T 0028 安全一级标准）的项目。它不仅在阿里巴巴多款产品中得到了实战检验，还具备权威认证与良好兼容性，因此成为我移植到 BoringSSL 的首选蓝本。

## BoringSSL 与 OpenSSL 的差异

在《[不要重复发明轮子！谷歌：我偏要](https://mp.weixin.qq.com/s/aLAfcVwBDy9dw7hGnTq-ew)》一文中，已有介绍：BoringSSL 是 Google 从 OpenSSL 分叉后演化出的 SSL 库。但经过多次“魔改”，BoringSSL 与 OpenSSL 的内部结构已大相径庭。虽然同为 OpenSSL 的后裔，铜锁与 BoringSSL 在细节上仍有不少差别，直接照搬并不可行，必须因地制宜地对接与调整。

## 步骤一：用 DeepWiki 解读铜锁的国密实现

1. 打开项目分析

DeepWiki 是一个源码分析平台，收录了海量 GitHub 项目，也支持用户自行添加。只要在首页搜索 “tongsuo”，即可快速定位并加载铜锁的全量源码图谱。

![DeepWiki](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_01.png)

2. 重点关注 TLCP（在铜锁中叫 NTLS）章节

DeepWiki 专门列出 “NTLS 实现分析” 这一章节，并附有清晰的流程图，帮助你快速把握 NTLS 在握手、签名、加解密等环节的调用关系。

![NTLS实现分析](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_02.png)

3. 架构与流程一览

有了 DeepWiki 生成的 NTLS 架构图，就能从整体层面理解算法模块的分工；紧接着再看握手流程图，就能明确每一步要调用哪些函数。

![NTLS架构图](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_03.png)

![NTLS握手过程](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_04.png)

4. 实时提问与深度思考

如果对源码细节有疑问，可以在 DeepWiki 提问框中提交。

![针对项目提问](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_05.png)

勾选**Deep Research**，类似于 DeepSeek R1 的深度思考模式，回答问题的速度会慢一些，而且会显示思考的过程。

![铜锁实现的国密套件](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_06.png)

## 步骤二：解析 BoringSSL

1. 同样，使用 DeepWiki 分析 BoringSSL 开源项目，BoringSSL 虽然是托管在谷歌的代码服务器上，但在 github 上有镜像站点，所以可以直接搜索到。

![BoringSSL](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_07.png)

2. 向 DeppWiki 提问：我想为BoringSSL增加国密算法（SM2、SM3、SM4）实现，以及增加 ECDHE_SM4_CBC_SM3 密码套件，该如何实现？

![BoringSSL](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_08.png)

DeepWiki 给出的答案，左边是实现要点，右边则是具体的源码文件和代码行。虽然不是完全正确，但至少提供了一个思路，知道具体需要修改哪些文件。

做了前面的准备工作后，就可以动手进行移植。移植过程当然不会是那么顺利，特别是在调试 TLCP 统信过程，需要对照协议，分析 client 和 server 之间的通信数据。这个过程可以借助 wireshark 进行抓包分析。最新版本的 wireshark 已经支持 TLCP 协议的分析，使用起来方便很多。

![wireshark](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_09.png)

整个交互过程还是比较清晰的，碰到 wireshark 无法识别的数据格式怎么办？比如上图的 Server Key Exchange，就提示 “Cipher suite not implemented”，这个时候有可以借助 AI 来分析。现在各种 AI 非常多，但总体尝试下来，对 Chromium 项目源码理解最好的是 ChatGPT，所以这里使用 ChatGPT 进行数据分析。从 WireShark 中复制 TLCP 报文，然后向 ChatGPT 提问。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_10.png)

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/porting_ntls_11.png)

ChatGPT 对报文的分析非常详实准确，在调试过程中，可以对照代码，如果碰到哪次交互无法通过，分析起来就方便多了。

## 小结

在本次移植过程中，AI 只是作为一种辅助工具，帮助我分析代码，找到修改的关键点，并在调试上提供分析帮助，但离自主智能还差得远。现在也有不少的 AI Agent，能够实现提出需求，然后自动写代码的功能，但是对于大型复杂的项目来说，效果并不是很好，所以说 AI 替代程序员，还没那么容易。程序员也不用提心吊胆，害怕被 AI 取代。我更多的是把 AI 视作帮手。的确，有了 AI 后，开发的效率提高了很多。

最后，项目已经开源，项目地址：https://github.com/mogoweb/mojo-ssl。由于时间仓促，SM4 只实现了 CBC 分组模式，SM2 只实现了签名和验签，密码套件只实现了 ECDHE_SM4_CBC_SM3。后续将补全更多模式与套件，欢迎扫码 Star、PR！
