# 写给开发人员的实用密码学 - CA

在上一篇文章《[写给开发人员的实用密码学 - 数字证书](https://mp.weixin.qq.com/s/IEaQFRqpq4AQS2pqwkXK6A)》中介绍了数字证书，但要让用户信任颁发的数字证书，这里就需要引入 CA 中心。

所谓CA（Certificate Authority）认证中心，即采用PKI（Public Key Infrastructure）公开密钥基础架构技术，专门提供网络身份认证服务的机构。CA可以是民间团体，也可以是政府机构。CA负责签发和管理数字证书，且具有权威性和公正性，它的作用就像我们现实生活中颁发证件的公司，如护照办理机构。

#### 根CA信任模型

面对全球这么广泛的用户，仅仅一个CA显然不够。PKI引入“信任模型”，用于描述和分析同一CA管理域内部或不同CA管理域之间信任关系的建立和传递过程。PKI信任模型主要采用根CA信任模型，也称作严格层次信任模型。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/practical_cryptography_for_developers_ca_01.jpg)

该信任模型下，CA中心可以分为多级，各级CA中心之间呈现严格的层级关系，最上级CA中心只有一个，称作根CA，其它CA称作子CA。根CA的数字证书由自己签发，属于自签名证书，子CA的数字证书由上级CA签发。信任锚可以是根CA，也可以是子CA。

上图中，用户X的信任锚为根CA，因此它可以信任子CA1，从而信任用户A证书，信任链为 **根CA -> 子CA1 -> 用户A证书**。用户Y的信任锚为子CA2，因此它可信任子CA4，从而信任用户D证书，信任链为 **子CA2 -> 子CA4 -> 用户D证书**。

注意，不要被上图误导，全球的根CA不止一个，应该说现实世界的CA是**多根CA信任模型**。

要建立信任锚，就需要获得CA的证书（根据前面的描述，可以是根证书，也可以是二级、三级证书，也可以是用户证书），那这个证书怎么获得呢？会不会获得假冒的证书？其实这个问题的答案非常简单，也可能出乎我们的意料，那就是预置根证书。

#### 预置根证书

根据前面的CA模型，应用程序并不需要预置所有的CA证书，而只需要预置最顶层CA的证书（通常称作根证书）即可，而全球顶层CA中心数量有限，大概十来个，所以不会存在存储上的问题。当然，如果我们去查看系统预置的根证书，发现数量好像远远不止，那是因为为了程序处理的方便，我们也可能预置一些二级CA证书。比如中国的CA中心在全球可能只是二级CA中心，而我们经常会验证中国CA中心颁发的证书，这个时候预置这些二级CA证书，可以避免证书验证时验证链过长，提高效率。

根证书存为怎样的格式，存放在什么位置，与应用程序的实现有关。就拿浏览器来说，Chrome Windows版本采用的Windows的系统证书存储区，关于证书的处理也是调用Windows的 CryptoAPI。而Chrome Linux版本的根证书则是存储在 NSS 数据库中。到了Android 版本的Chrome浏览器，又使用了Android系统的预置证书。而且随着版本的升级，这些策略还可能调整。

预置根证书的方式也存在一定的不足，那就是假定全球的顶级CA中心是不变的。实际上，随着互联网对数字证书的需求越来越大，顶层CA中心也在扩容，这就导致新CA中心签发的证书，可能在现有系统得不到承认。此外，有时候虽然证书虽然不是这些权威的CA中心签发的，但你也信任它。比如早年的12306网站，就采用了自签名证书，而没有采用CA中心签发的证书。

因此，大部分软件都提供了证书管理功能，你可以导入证书，下图就是chrome浏览器的证书管理界面：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/practical_cryptography_for_developers_ca_02.png)

有些软件更加灵活方便，比如 chrome 浏览器，还在遇到可疑的证书时，让你选择是否信任。如果你选择信任，就可以继续访问。

所以证书本质上还是一个信任问题，浏览器和操作系统为什么预置证书，是因为开发者信任这些CA中心，信任他们颁发的证书。其它情况下，有些软件可能会让用户做选择，比如浏览器，而有些软件则不会给用户选择，比如播放器，会直接拒绝不信任的证书。

#### 搭建国密CA中心

CA中心听起来名字就很高大上，其实个人也可以整个CA中心，并签发证书，当然这些作为测试用途是没问题的，至于在实际中能否行得通，就要看忽悠能力了。

现实世界的CA是分级管理的，层级可以有多层。本文简单起见，只模拟到三级CA管理，具体来说：

> Root CA -> Server CA -> Server

Root CA为一级CA，拥有根CA证书，Server CA为二级CA，其CA证书由Root CA签发，Server为最终的用户，其证书由Server CA签发。当然，Root CA也可以直接给Server签发证书，这里是为了演示CA层级需要而做的这样的假定。

##### 制作根CA证书

1. 生成SM2私钥：

```
$ gmssl ecparam -genkey -name sm2p256v1 -text -out rootkey.pem
```

根证书的私钥保存在rootkey.pem中，请妥善保存。

2. 创建证书请求：

```
$ gmssl req -new -key rootkey.pem -out rootreq.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [CN]:
State or Province Name (full name) [Some-State]:Hubei
Locality Name (eg, city) []:Wuhan
Organization Name (eg, company) [Internet Widgits Pty Ltd]:China LianTong      
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:www.china_liantong.com
Email Address []:
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

上面的命令会要求提供一些信息，因为证书是自签名证书，只用于开发测试，所以填写什么内容无关紧要。

3. 创建一个 certext.ext 文本文件，内容为：

```
[ v3_ca ]
basicConstraints = CA:true
[ usr_cert ]
subjectAltName = DNS:localhost
```

"CA:true" 这个扩展，用于指定所签发的证书是否CA证书。

4. 生成证书：

```
$ gmssl x509 -req -days 365 -in rootreq.pem -signkey rootkey.pem -extfile certext.ext -extensions v3_ca -out rootcert.pem
Signature ok
subject=C = CN, ST = Hubei, L = Wuhan, O = China LianTong, CN = www.china_liantong.com
Getting Private key
```

生成的根证书文件为 rootcert.pem。

注意：上面的命令行参数多了一个 -extensions v3_ca 参数，指定使用上面 certext.ext 文件 v3_ca 节的扩展项。

##### 签发 Server CA 证书

和上面的步骤一样，这里直接把命令总结一下：

```
$ gmssl ecparam -genkey -name sm2p256v1 -text -out serverCAkey.pem
$ gmssl req -new -key serverCAkey.pem -out serverCAreq.pem
$ gmssl x509 -req -days 365 -in serverCAreq.pem -extfile certext.ext -extensions v3_ca -CA rootcert.pem -CAkey rootkey.pem -CAcreateserial -out serverCAcert.pem
```
需要注意第三个命令多了 -CA 和 -CAkey 参数，表示使用根证书以及CA的私钥。

这样我们就可以使用 Server CA证书和私钥签发用户证书。

##### 签发 Server 证书

和上面的步骤一样，这里直接把命令总结一下：

```
$ gmssl ecparam -genkey -name sm2p256v1 -text -out serverkey.pem
$ gmssl req -new -key serverkey.pem -out serverreq.pem
$ gmssl x509 -req -days 365 -in serverreq.pem -extfile certext.ext -extensions usr_cert -CA serverCAcert.pem -CAkey serverCAkey.pem -CAcreateserial -out servercert.pem
```

注意： 第三个命令的 -extensions 给的参数值为 usr_cert ，对应的是 certext.ext 文件 usr_cert 节的扩展项，通常需要给定服务器的DNS名。由此也可以看出，叶子节点的证书和CA证书还是有些属性不同。

这样，生成的 server.pem 包含了根证书、Server CA证书和Server证书，包含了完整的证书链，可以投入测试使用了。你可以尝试在 chrome 浏览器中导入根证书：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/practical_cryptography_for_developers_ca_03.png)

#### 证书链

在浏览器中，我们可以查看证书信息，一般来说，证书通常是呈现出多级的状态。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/practical_cryptography_for_developers_ca_04.png)

服务器配置的是CA颁发的服务器实体证书，而客户端（浏览器或操作系统）预置的是根证书，现在的问题是，中间证书怎么获取？

根据X.509标准，服务器应该发送完整的证书链（不包含根证书）。如果服务器端发送的证书不完整，某些客户端可以去尝试构建完整的证书链，但有些浏览器可能不会执行该操作，这样整个HTTPS协议握手就会失败。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202104/images/practical_cryptography_for_developers_ca_05.png)

根据服务器实体证书寻找完整证书链的方法很简单，浏览器从服务器实体证书中获取CA密钥标识符（Authority Key Identifier），进而获取上一级中间证书文件，然后通过中间证书中的CA密钥标识符不断迭代直到获取根证书。

服务器发送完整证书链的好处在于提高握手速度，浏览器无须额外再去寻找其他中间证书，能够加速HTTPS握手过程。

校验过程：

1. 浏览器从服务器实体证书的上一级证书（比如B证书）获取公钥，用来校验服务器实体证书的签名，校验成功则继续，否则证书校验失败。
2. 浏览器从B证书的上一级证书（比如C证书）获取公钥，用来校验B证书的签名，校验成功则继续，否则证书校验失败。
3. 该校验过程不断迭代，直到浏览器发现某张证书的签发者和使用者是同一个人，代表找到了根证书。校验根证书的签名和校验非根证书的签名不太一样，校验根证书签名使用的公钥就在根证书中，而校验其他非根证书签名使用的公钥来自上一级证书，根证书使用自己的公钥验证签名，如果校验成功就代表完整的证书链校验成功。

#### 证书完整校验

X.509标准没有规定校验证书的标准，不同浏览器校验证书的规则也不一样，一般来说，校验主要包含四个方面：

* 证书链的校验。主要是迭代校验每张证书的签名，最后会找到自签名的根证书，由于浏览器已经集成了根证书，可以充分信任根证书的公钥，完成最后的签名校验。但签名验证成功只能代表某张服务器证书确实是由某个根证书签发的，并不能表示身份验证成功。
* 服务器实体证书的校验。主要校验如下内容：

（1）浏览器访问的域名是不是与证书使用者可选名称（SAN）扩展包含的域名匹配，如果不匹配，校验失败。

（2）日期校验，证书包含有效期，生效时间位于{notBefore, notAfter}区间，如果证书过期，校验失败。

（3）证书扩展校验（可选）。比如，如果扩展Critical被标识为True，客户端必须正确处理该扩展，否则校验失败。再比如，浏览器通常会校验密钥用法（Key Usage）扩展，该扩展对应的值如果不包含数字签名（Digital Signature）和密钥协商（Key Encipherment），校验失败。

* 中间证书。

（1）日期校验，校验证书有效期。

（2）证书扩展Critical如果标识为True，必须校验。

（3）中间证书也包含一个公钥，需要校验该公钥的用途，校验方法就是判断密钥用法（Key Usage）扩展，该扩展对应的值如果不包含Digital Signature（数字签名）、Certificate Sign（签名证书）、CRL Sign（签署CRL），校验失败。

（4）校验基础约束（basic constraints）扩展，校验中间证书是否能够签发证书，如果不允许签发，校验失败。

* 吊销状态校验。校验的逻辑非常复杂，有些浏览器可能会放弃吊销状态的检查。需要注意的是，服务器实体证书和中间证书都需要校验吊销状态，但具体如何校验取决于浏览器，这些并不是TLS/SSL协议的标准。

#### 小结

《写给开发人员的实用密码学》系列文章到这就告一段落，实际上有关网络安全、系统安全远不止加密解密、签名、证书等内容，只是密码学是网络安全的基础。在补充密码学知识的同时，我也对网络安全有了更多的认识，后面我还会围绕国密、网络安全、HTTPS、HTTP/2等内容进行研究，欢迎大家讨论交流。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)