# 解读国密非对称加密算法SM2

本文先介绍非对称加密算法，然后聊一聊椭圆曲线密码算法（Elliptic Curve Cryptography，ECC），最后才是本文的主题国密非对称加密算法SM2。因为我的数学知识有限，对于算法涉及的一些复杂的理论知识，也是不懂，所以本文不会涉及理论，仅仅从编程的角度解读一下SM2。

在进行国密算法开发的这段时间，我主要参考的书籍是《深入浅出HTTPS：从原理到实战》，微信读书上也有电子版，如果你也是进行网络安全方面的开发，建议你读一读。这篇文章中的密码学基础知识也是来自此书。

对计算机安全有点基础知识的人们应该知道，在密码学中，用于数据加密的算法主要有两种：对称加密算法（Symmetric-keyAlgorithms）和非对称加密算法（Asymmetrical Cryptography）。所谓对称加密算法，就是加密密钥和解密密钥相同，这个很好理解，就像我们给房间上锁，锁门的钥匙和开门的钥匙是一样的。而非对称密钥加密算法则是加密密钥和解密密钥不同，这个有点违反普通常理，但确实存在这样的算法，其背后的理论非常复杂。我们不需要懂得多少其背后的理论，也可以采用非对称密码算法做很多安全方面的工作。

在整个密码学体系中，非对称加密算法用途更广，可以用在加密解密、密钥协商、数字签名等方面。所以本文先介绍一下非对称加密算法。

#### 非对称加密算法

非对称加密算法也称作公开密钥算法（Public Key Cryptography），有着一对密钥：公钥（Public Key）和私钥（Private Key）。

1. 加密解密

非对称加密算法的首要用途是加密解密，通常加密解密过程如下图所示：

![公开密钥算法加密解密过程](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_sm2_overview_01.png)

需要注意的是，上面只是其中一种用法，反过来，采用私钥加密，公钥解密也可以的。具体采用哪种方式，取决于使用场景。

非对称加密算法的安全性建立在复杂的数学模型之上，其背后的理论不用去深究。我们需要知道的是其安全性和密钥对的长度有关，尽量选择较长的密钥长度。例如，对于RSA非对称加密算法来说，一个2048比特长度的密钥对被认为是安全的。但较长的密钥长度会带来运算速度问题，需要综合权衡。

2. 密钥协商

非对称密钥算法存在加解密速度慢的问题，因此不能用于需要频繁加密大量数据的场景，这个时候需要用到对称密钥加密算法。问题是，怎么保证对称密钥的安全呢？特别是互联网网络通信中，密钥如何通过不安全的网络发送，如何保证对方会安全的存储密钥？

这个时候，可以采用动态密钥，也叫作会话密钥：

* 会话密钥的作用就是为了加密解密通信数据，也就是对称加密算法可以使用会话密钥进行加密解密。
* 在加密解密通信数据之前，客户端和服务器端协商出会话密钥，而会话密钥只有服务器端和特定的客户端才能知晓。
* 会话密钥的意思就是该密钥不用存储，一旦客户端和服务器端的连接关闭，该密钥就会消失，由于密钥不用存储，安全性就得到了很大的保障。

会话密钥可通过密钥协商算法完成，但密钥协商算法可以有很多种，目前主要采用的密钥协商算法有**RSA密钥协商算法**和**DH密钥协商算法**。

3. 数字签名

数字签名可以想象为现实世界中的合同签名，其主要作用是防抵赖，比如你给对方发送了一段消息，然后你否认这个消息是你发送的。

在现实世界中，有哪些行为或者约定可以防止人抵赖呢？最明显的就是合同，合同一般需要人签字或者按指纹。有了合同，合同签署人就无法否认合同的合法性，原因就在于法律规定，指纹具备唯一性，每个人的指纹是不同的，或者说指纹就代表了一个人。

在密码学中，如果一个消息也含有特殊的指纹，那么它是否就不能抵赖呢？非对称加密算法中，私钥只有密钥对的生成者持有，如果不考虑密钥泄露的问题，私钥拥有者使用密钥（注意不是加密操作）签署一条消息，然后发送给任意的接收方，接收方只要拥有私钥对应的公钥，就能成功反解签署消息，由于只有私钥持有者才能“签署”消息，不能抵赖说这条签署消息不是他发送的，这就是数字签名技术的全部。

签名生成流程：

![签名流程](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_sm2_overview_02.png)

签名验证流程:

![验签流程](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_sm2_overview_03.png)

需要注意，上面只是总的流程，签名算法也有很多种，各种算法在实现上还是有很多不同，请根据需要再深入到具体的算法中。

非对称密钥算法中最出名、使用最广泛的要数RSA算法，该算法是Ron Rivest、Adi Shamir、Leonard Adleman三个人创建的，以三个人名字的首字母命名。算法自1977年公开，沿用至今，体现出理论知识的威力。然而，前面也说过，RSA的安全性和密钥长度有关，随着计算机运算速度的提升，需要更长的密钥长度保证安全性。问题是，密钥长度越长，生成密钥的时间也越长，加密解密的速度也越慢。

这么些年来，密码学家也没有闲着，设计出新一代的公开密钥算法：椭圆曲线密码算法。

#### ECC

ECC主要的优点就是安全性，极短的密钥能够提供很大的安全性。比如224比特的ECC密钥和2048比特的RSA密钥可以达到同样的安全水平，由于ECC密钥具有很短的长度，运算速度非常快。

ECC基于非常复杂的算法，看了一些这方面的理论知识，也是云里雾里，所以这里只说说和我们使用和编程相关的基础知识。

不理解ECC理论知识没有关系，但需要了解以下这张图：

![ECC模型](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_sm2_overview_04.png)

ECC椭圆曲线由很多点组成，这些点由特定的方程式组成的，比如方程式可以是y^2 = x^3 + ax + b，这些点连接起来就是一条曲线，但曲线并不是一个椭圆。

椭圆曲线有个特点，任意两个点能够得到这条椭圆曲线上的另外一点，这个操作称为打点，经过多次（比如d次）打点后，能够生成一个最终点（F）。

在上面的图中，A点称为基点（G）或者生成器。A可以和自己打点从而生成B点，在实际应用的时候，一般有基点就可以了。经过多次打点，就得到了最终点G。

ECC密码学的关键点就在于就算知道具体方程式、基点（G）、最终点（F），也无法知晓一共打点了多少次（d）。

ECC中，打点次数(d)就是私钥，这通常是一个随机数，公钥就是最终点（F)，包含(x，y)两个分量，通常组合成一个数字来传输和存储。

ECC由方程式（比如a、b这样的方程式参数）、基点（G）、质数（P）组成。理论上方程式和各种参数组合可以是任意的，但是在密码学中，为了安全，系统预先定义了一系列的曲线，称为命名曲线（name curve），比如secp256k1就是一个命名曲线。对于开发者而言，在使用ECC密码学的时候，就是选择具体的命名曲线。

说到这儿，和国密SM2算法有什么关系？

#### 国密SM2算法

SM2算法就是一种ECC算法，准确来说，就是设计了一条ECC命名曲线。这算抄袭么？也不是，因为设计一条安全的命名曲线，也是一件非常难的事情，也需要丰富的理论知识。ECC本质上就是一个数学公式，任何人基于公式都可以设计出椭圆曲线，但要注意ECC离散对数问题（Elliptic-Curve Discrete-Logarithm Problem，简称ECDLP），如果实现不当，那么ECC公式就会存在安全风险。一些组织为此还定义了命名曲线的一些设计标准，不同的设计标准有不同的目标，比如有的以安全性为首要目标，有的以效率为首要目标。

在《GMT 0003-2012》这份标准中，有SM2算法的设计背景知识，有兴趣的可以了解，对于开发者而言，最重要的是《GMT 0003.5-2012》标准中的曲线参数：

** p、a、b、G(x,y)和n **

![SM2曲线参数]](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_sm2_overview_04.png)

现在的网络库，比如NSS、OpenSSL、libtomcrypt等，都有ECC算法的支持，要在网络库中加入SM2算法支持，只需加入命名曲线的参数即可。

比如在GmSSL代码的 ec_curve.c 文件中就有 sm2p256v1 命名曲线的参数定义：

```
static const struct {
    EC_CURVE_DATA h;
    unsigned char data[0 + 32 * 6];
} _EC_SM2_PRIME_256V1 = {
    {
        NID_X9_62_prime_field, 0, 32, 1
    },
    {
        /* no seed */
        /* p */
        0xFF, 0xFF, 0xFF, 0xFE, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
        0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0x00, 0x00, 0x00, 0x00,
        0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
        /* a */
        0xFF, 0xFF, 0xFF, 0xFE, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
        0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0x00, 0x00, 0x00, 0x00,
        0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFC,
        /* b */
        0x28, 0xE9, 0xFA, 0x9E, 0x9D, 0x9F, 0x5E, 0x34, 0x4D, 0x5A, 0x9E, 0x4B,
        0xCF, 0x65, 0x09, 0xA7, 0xF3, 0x97, 0x89, 0xF5, 0x15, 0xAB, 0x8F, 0x92,
        0xDD, 0xBC, 0xBD, 0x41, 0x4D, 0x94, 0x0E, 0x93,
        /* x */
        0x32, 0xC4, 0xAE, 0x2C, 0x1F, 0x19, 0x81, 0x19, 0x5F, 0x99, 0x04, 0x46,
        0x6A, 0x39, 0xC9, 0x94, 0x8F, 0xE3, 0x0B, 0xBF, 0xF2, 0x66, 0x0B, 0xE1,
        0x71, 0x5A, 0x45, 0x89, 0x33, 0x4C, 0x74, 0xC7,
        /* y */
        0xBC, 0x37, 0x36, 0xA2, 0xF4, 0xF6, 0x77, 0x9C, 0x59, 0xBD, 0xCE, 0xE3,
        0x6B, 0x69, 0x21, 0x53, 0xD0, 0xA9, 0x87, 0x7C, 0xC6, 0x2A, 0x47, 0x40,
        0x02, 0xDF, 0x32, 0xE5, 0x21, 0x39, 0xF0, 0xA0,
        /* order */
        0xFF, 0xFF, 0xFF, 0xFE, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
        0xFF, 0xFF, 0xFF, 0xFF, 0x72, 0x03, 0xDF, 0x6B, 0x21, 0xC6, 0x05, 0x2B,
        0x53, 0xBB, 0xF4, 0x09, 0x39, 0xD5, 0x41, 0x23
    }
};
```

代码中的order就是参数表中的 n。对于大部分ECC操作来说，不需要该值，但在计算签名的时候会对n取模。

#### 小结

本文从非对称密码算法开始，逐步介绍到国密SM2算法。我们可以看到，SM2并不是一个全新设计的算法，而是借助现有的ECC理论，设计了一条命名曲线。这样，在已经实现了ECC算法的网络库上增加SM2算法的支持就非常简单，只需要将曲线参数添加即可。

但这是否就已经完全实现了SM2算法呢？也不是，因为SM2算法不仅用在加解密，还用在数字签名、密钥协商中，国密标准另外定义了数字签名算法、密钥交换协议、公钥加密算法，所以要把这些都实现完整，才算实现完全了国密SM2算法。

如果大家有什么问题，欢迎交流。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)