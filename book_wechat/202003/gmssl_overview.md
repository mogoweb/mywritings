# 初识国密算法

国密算法是国家商用密码算法的简称，由国家密码管理局管理和发布标准。国家密码管理局的官方网站是：

> http://www.oscca.gov.cn/sca/index.shtml

就如同其它政府网站一样，望着满屏的信息，就是找不到我所想要的，比如国密算法标准和规范文档，网站上只有发布文档的公告，但找不到获取方法。后来在github上找到，网址为：

> https://github.com/guanzhi/GM-Standards.git

这里面有全套的国密算法标准文档，不是由官方维护，不清楚里面是否包含了所有的最新标准文档。文档库中的文档非常多，比如GMT正式标准下就有69个标准文档：

![GMT正式标准](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_overview_01.png)

里面包含了SM2/SM3/SM4等密码算法标准及其应用规范。“SM”代表“商密”，即用于商用的、不涉及国家秘密的密码技术。其中SM2为基于椭圆曲线密码的公钥密码算法标准，包含数字签名、密钥交换和公钥加密，用于替换RSA/Diffie-Hellman/ECDSA/ECDH等国际算法；SM3为密码哈希算法，用于替代MD5/SHA-1/SHA-256等国际算法；SM4为分组密码，用于替代DES/AES等国际算法；SM9为基于身份的密码算法，可以替代基于数字证书的PKI/CA体系。

看到这一堆的标准文档，是不是有点发怵。别慌，目前已经有国密算法的开源实现，这个项目就是GmSSL。该项目有一个官方网站：

> http://gmssl.org/

里面有许多对于程序员而言非常有用的信息。从中我们可以了解到，GmSSL项目由北京大学关志副研究员的密码学研究组开发维护，项目源码托管于GitHub：

> https://github.com/guanzhi/GmSSL.git

GmSSL是一个开源的密码库以及工具箱，支持SM2/SM3/SM4/SM9/ZUC等国密(国家商用密码)算法、SM2国密数字证书及基于SM2证书的SSL/TLS安全通信协议，支持国密硬件密码设备，提供符合国密规范的编程接口与命令行工具，可以用于构建PKI/CA、安全通信、数据加密等符合国密标准的安全应用。

GmSSL是从鼎鼎大名的开源安全库 OpenSSL fork而来，研究其源码可以看出它是基于OpenSSL 1.1.0d开发。GmSSL与OpenSSL保持接口兼容，理论上使用OpenSSL库的产品，可以直接使用GmSSL替换。GmSSL项目采用对商业应用友好的类BSD开源许可证，无论是在开源项目，还是闭源的商业应用中，都可以放心使用，不必担心授权问题。

研究安全算法，必须拥有深厚的数学功底（又是数学，不管是研究深度学习，还是安全算法，数学都是拦路虎:(，阴魂不散），否则读那些算法如同天书。感谢有这些开源实现，否则真不知道如何继续下去。基于这些开源实现，我们可以在几个方面进行深挖：

1. 算法移植，包括硬件和软件两个方面。硬件移植包括移植到更多的平台，比如ARM、MIPS等架构的芯片上，还有就是针对硬件平台进行优化，启用硬件加速。软件则包括将算法移植到其它的语言库，比如Java、JavaScript等等，甚至同样使用C语言，也存在移植的需求，比如firefox家族产品所使用的NSS库。
2. GmSSL库本身的完善。就拿《GM/T 0024-2014: SSL VPN技术规范》的新增密码套件来说，GmSSL并不是都支持。后续如果有新的算法标准公布，也需要加入新的实现。
3. 集成国密算法到产品。国密算法只是一套算法标准，其作用还要体现在具体产品中。要推广国密算法，客户端（如浏览器、邮件客户端、加密卡等）和服务器端（如 Web 服务器、邮件服务器等）都需要推进。

国密算法标准，更多的像是自上而下的政治任务，而不是从产品的实际需求出发，不可避免的带有一些中国特色。就拿浏览器产品来说，虽然这个标准从2012年开始颁布，但我这个从业者之前从没有听说过这个标准，主流的浏览器chrome、firefox，甚至国内的浏览器都不支持该标准。网上搜索了一些号称支持国密的浏览器，除了360，都没达到商用水准。

这套标准中，SM1算法并没有公布，从现代密码学的角度看，这并不安全。就我们普通人看来，算法不公布，不是更难破解吗？所谓道高一尺，魔高一丈，看看Windows系统被攻击的多惨就知道了。而Linux系统采取开源策略，不停的有高手参与进来修复漏洞，反而更安全。当然这只是我的浅见，我对算法这么高深的领域没有发言权。

有句话是这样说的，一流的企业做标准，二流的企业做品牌，三流的企业做产品。要争夺话语权，制定标准无疑是最高级的打法。但从国密算法标准操作来看，似乎并没有想更多的企业参与进来，先不说国际推广吧，对国内参与者也不算友好，发布的标准文档还需要费力从个人维护的github找到，文档也是纸质扫描档，搜索、查找都不方便。参与的企业也是各自为战，并没有很好的交流渠道。

不过吐槽归吐槽，中国推出这套国密算法，也是期望在国际竞争中不受制于人。这套算法标准，目前没看出其先进性。但路总要一步步的走，期望一下子推出领先国际的技术，而忘了我们的基础还比较薄弱，那是自欺欺人。这样的标准推广，需要众多的程序员，做好基础工作，先做出和别人相当的水准，才可能后续慢慢超越。

如果你也在从事相关的工作，欢迎交流！

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)