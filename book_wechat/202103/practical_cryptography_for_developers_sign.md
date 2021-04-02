# 写给开发人员的实用密码学 - 数字签名

办理过买房抵押贷款的朋友，应该还记得在贷款合同上不断按手印。这种现实世界的签字画押，一方面是保证合同的文本不会替换，另一方面双方事后都不能否认此次交易（个人方面靠签字和手印，银行方靠公章）。

借助生物特征（指纹）上的唯一性，完成了现实世界中的防篡改和防抵赖。

在计算机世界里，数字签名技术用来达成现实世界中的签名和盖手印相同的目的，所以现数字签名有两个作用，防篡改和防抵赖。

在前面的文章中讲到，MAC和非对称加密也可以用于防篡改，但前面介绍的算法都无法防抵赖。先来说说现有算法为什么无法防抵赖。

先看看对称加密算法，A、B、C三个人共享一个对称加密算法密钥，A向B发送了一条消息，但是A可以抵赖说这条消息并不是他发送的，理由就是C也有同样的密钥，这条加密消息可能是C发送给B的，B无法向第三方证明是A给他发送了消息。即使只有 A、B 两人，A 也可以抵赖说消息是 B 伪造的。

使用 MAC 算法，能保证传递的消息是经过验证的，但不能对消息发送者的身份进行验证，原因就在于消息发送方和接收方拥有同样的密钥，所以双方可以抵赖，否认消息是他发送的。

在公开密钥算法中，如果算法用于加密解密，也同样不能防止抵赖。因为在非对称加密算法中，公钥是完全公开的，私钥拥有者虽然能够解密，但是并不能确认是哪个客户端发送的消息，这样任何人都可以抵赖。

#### 数字签名

在现实世界中，指纹具备唯一性，每个人的指纹是不同的，或者说指纹就代表了一个人，所以在重要的合同中，我们会按上手印，表明这份合同确实是自己签的。

在密码学中，一个消息中包含特殊的指纹（数据），也可以起到现实世界的指纹的作用。具体做法是，私钥拥有者使用密钥签署一条消息，然后发送给任意的接收方，接收方只要拥有私钥对应的公钥，就能成功反解签署消息。由于只有私钥持有者才能“签署”消息，不能抵赖说这条签署消息不是他发送的，这就是数字签名技术。

需要注意的是，虽然使用私钥加密，公钥解密可以达到上述效果，但现有的签名技术采用的算法和加解密时使用的算法并不相同，这一点在实现算法时需要注意。

简单地说，数字签名技术有以下几个特点：

* 防篡改：数据不会被修改，MAC算法也有这个特点。
* 防抵赖：消息签署者不能抵赖。
* 防伪造：发送的消息不能够伪造，MAC算法也有这个特点。

#### 数字签名的流程

不管采用何种数字签名算法，数字签名处理流程是差不多的，主要分为签名生成和签名验证。签名生成流程如下所示。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202103/images/practical_cryptography_for_developers_sign_01.png)

1. 发送者对消息计算摘要值。
2. 发送者用私钥对摘要值进行签名得到签名值。
3. 发送者将原始消息和签名值一同发给接收者。

和签名值一同传递的消息可以是加密过的消息，也可以是未加密的信息，因为这和签名算法并没有什么关系，所以请根据消息的保密程度决定是否需要加密。

签名验证流程也很好理解：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202103/images/practical_cryptography_for_developers_sign_02.png)

1. 接收者接收到消息后，拆分出消息和消息签名值A。
2. 接收者使用公钥对消息进行运算得到摘要值B。
3. 接收者对摘要值B和签名值A进行比较，如果相同表示签名验证成功，否则就是验证失败。

签名算法不直接对消息进行签名，而是对消息的摘要值进行签名，原因是公开密钥算法运行相对缓慢，消息的摘要值的长度固定且短（一般256位），运算速度会比较快。

#### 国密数字签名算法

国密数字签名算法在《GM/T 0003.2-2012 SM2椭圆曲线公钥密码算法第2部分：数字签名算法》这份文档中有详细的描述。其中签名的流程为：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202103/images/practical_cryptography_for_developers_sign_03.png)

国际上比较通用的椭圆曲线数字签名算法成为 ECDSA，定义在 ANSI X9.63 这个标准文档中。国密 SM2 算法虽然也是一种椭圆曲线算法，但其签名流程和 ECDSA 有些不同，这点在实现时需要注意。

上述步骤中的 ZA = H256(ENTLA || IDA || a || b || xG || yG || xA || yA)，ENTLA为两字节表示的 IDA 的比特数（注意不是字节数）。

可以看出，国密签名算法对消息进行了处理，然后才计算摘要。其中 ZA 的计算涉及到命名曲线参数的a、b、G、P（公钥）。

验证签名的流程：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202103/images/practical_cryptography_for_developers_sign_04.png)

如果实现了签名流程，验证签名的流程也比较容易实现，主要是需要关注其中的公式，注意一些细节。

在开发SM2数字签名算法时，我们可以参考《GM/T 0003.2-2012 SM2椭圆曲线公钥密码算法第2部分：数字签名算法》这份文档中附录A的示例，保证每个步骤的数据能对上，这样最终的结果就不会出错。

#### 国密数字签名示例

* 生成 SM2 私钥

```
 $ gmssl sm2 -genkey -out sm2.pem
 Using configuration from /data/gmbrowser/usr/local/gmssl/ssl/openssl.cnf
 generate SM2 key
 writing SM2 key
 Enter PEM pass phrase:
 Verifying - Enter PEM pass phrase:
```

为了保证私钥的安全，一般需要时候口令保护

* 根据私钥导出SM2公钥

```
 $ gmssl sm2 -in sm2.pem -pubout -out sm2Pub.pem
 Using configuration from /data/gmbrowser/usr/local/gmssl/ssl/openssl.cnf
 read SM2 key
 Enter PEM pass phrase:
 writing SM2 key
```

这里需要输入私钥的保护口令，保证即使私钥不小心泄露，也不会被别人利用。

* 将待签名的消息放入 msg.txt 文件

```
 $ echo "this is a sm2 test" > msg.txt
```

* 对消息签名

```
 $ gmssl sm2utl -sign -in msg.txt -inkey sm2.pem -id Alice -out sig.der
 Enter pass phrase for sm2.pem:
```

因为签名使用到了私钥，所以需要输入保护口令。

* 验证签名

```
 $ gmssl sm2utl -verify -in msg.txt -sigfile sig.der -pubin -inkey sm2Pub.pem -id Alice
 Signature Verification Successful
```

使用公钥验签。

有兴趣的同学也可以参考我的移植代码：https://gitee.com/mogoweb/libtomcrypt-gm.git

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)