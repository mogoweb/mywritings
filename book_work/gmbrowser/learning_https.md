
HTTP的模型很简单，是一个B/S模型，由客户端和服务器组成，交互流程很简单。

* 一个HTTP客户端发送请求至HTTP服务器，然后等待服务器的响应。
* 一个HTTP服务器负责监听端口（默认是80），然后等待客户端的请求，处理完成后，回复给客户端。

大家最熟悉的客户端就是浏览器，比如Chrome和Firefox，另外一种客户端可以称为命令行工具，比如Linux上的Curl工具。服务器软件就更多了，比较流行的就是Nginx和Apache，服务器实现HTTP并不困难，困难的是如何高效地处理客户端请求。

#### 1.2.2 HTTP语义

HTTP消息主要包括两部分，分别是HTTP语义和HTML实体，这里主要讲解HTTP语义信息，了解HTTP语义重点是理解HTTP头部（HTTP Header）。

HTTP消息由三部分组成。

* 请求行或响应行。
* HTTP头部。
* HTML实体，包括请求实体和响应实体。
  
比如访问某个URL，浏览器根据用户的行为和终端环境构建消息结构体，连接上服务器，将消息体发送给服务器，然后等待响应，请求消息结构如下：

```
```

接下来，服务器处理请求并发送HTTP响应。

```
```

HTTP的几个特点:

1）客户端/服务器模型

HTTP是一个客户端/服务器模型，客户端和服务器通过网络交换信息。当然HTTP本身是不能传输的，需要通过网络层中的其他协议进行通信，一般构建在TCP之上。TCP能够提供一个可靠的、面向连接的传输服务，换句话说，客户端和服务器是否正确传输依赖于TCP这个协议。

2）HTTP是无状态的

HTTP是基于TCP的，当一个TCP连接关闭后，所有的HTTP请求/响应信息将全部消失。在HTTP中，客户端通过Socket技术创建一个TCP/IP连接，并连接到服务器，完成信息交换后，就会关闭TCP连接，这种模型简单明了。所谓的无状态就是每次请求完成后，不会在客户端和服务器上保存任何的信息，对于客户端和服务器来说，根本不知道上一次请求的信息是什么，甚至不知道本次连接的对端是不是上次连接的那一端，它的生命周期随着TCP/IP连接的关闭结束了。

3）HTTP是跨平台的

通过上面的讲解，读者知道HTTP就是具备一定规则的纯文本信息，任何开发语言都可以实现HTTP或者基于HTTP进行开发，开发出来的软件也很容易移植，受系统环境的影响非常少。

4）HTTP用途很广泛

Web主要使用HTTP进行传输数据，HTTP更多的是一个数据载体，对于Web应用来说更重要的是浏览器如何处理这些数据，这些本身和HTTP关系并不大。

互联网应用安全包括两部分：

* HTTP本身的安全问题。
* Web应用安全问题。

1）无线WiFi网络的攻击

提供WiFi网络的攻击者可以截获所有的HTTP流量，而可怕的是HTTP流量本身是明文的，攻击者用肉眼就可以知道用户的密码、银行卡信息、浏览习惯，根本不用进行任何的分析就可以获取用户的隐私。

2）垃圾广告攻击

很多用户浏览某个网页的时候，经常发现页面上弹出一个广告，而这个广告和访问的网页根本毫无关系，这种攻击很常见，主要是ISP（互联网服务提供商）发动的一个攻击，用户根本没有任何办法防护。用户访问网站的时候肯定经过ISP, ISP为了一些目的，比如获取广告费用，在响应中插入一段HTML代码，就导致了该攻击的产生。这种攻击称为主动攻击，也就是攻击者知晓攻击的存在。

## 密码学

解决HTTP安全的方法就是采用HTTPS，理解HTTPS之前必须掌握基本的密码学知识，HTTPS本质上就是对密码学算法的组合，很多读者无法充分理解HTTPS的根本原因在于没有掌握密码学的基本知识。

### 2.1.1 基本认知
1）密码学是科学

密码学是科学，有着严格的规范，设计密码学算法需要具备深厚的数学知识，一般情况下开发者不要自行设计密码学算法，因为这存在极大的风险。

2）密码学理论是公开的

密码学算法的实现原理是公开的，读者可能觉得这观点很奇怪，很多开发者喜欢设计千奇百怪的算法，窃以为别人并不知道，其实自行设计的算法根本不具备严格的数学模型，很容易被攻破。流行的密码学算法其算法实现是公开的，经过了长时间的考验。

3）密码学算法是相对安全的

随着时间的推移，计算机的处理速度越来越快，某个密码学算法的数学基础可能受到挑战，现阶段安全的密码学算法，未来可能就是不安全的了。世界上也没有绝对安全的密码学算法，对于算法应用者来说，确保目前使用的密码学算法是安全的就可以了，潜在也说明，应用者应该长期关注密码学算法的安全性，使用最安全、最合适的密码学算法。

4）密码学攻击方法是多样化的

大部分密码学算法需要密钥，最简单的破解方法就是获取密钥，除此之外攻击方式非常多，由于算法实现是公开的，一般不会攻击算法本身。开发者在编写应用的时候，可能会错误地使用密码学算法从而出现一些安全漏洞，而这些漏洞是攻击者的分析目标，一旦攻击成功，应用就会出现安全风险。判断是否真正掌握密码学知识的方法就是成为一个攻击者，因为只有完整地明白密码学原理才能发起攻击。

5）密码学应用标准很重要

很多开发者可能了解某个密码学算法的用途，但由于密码学算法应用有很多陷阱，一旦使用不当，会出现很多问题，为了正确地应用密码学算法，制定了很多应用标准（比如PKCS标准）。开发者可以不了解密码学算法的原理，但是必须掌握应用标准，这样才能编写出更安全的软件。

6）不具备很强的数学知识也能掌握密码学

密码学是基于数学模型的，要真正明白密码学原理，必须具备很好的数学知识，很多密码学算法的创建者都是数学家而非计算机编码专家，就说明了这一点。对于开发者来说，没有掌握密码学算法的原理并不妨碍应用密码学算法。

7）解决特定问题的密码学算法

世界上不存在一种密码学算法，能够解决所有的安全问题，每种算法有特定的应用场景，只能解决特定的问题。在思考安全解决方案的时候，必须具体问题具体分析，比如解决HTTP安全问题的时候，首先分析它存在的核心问题，然后思考每个问题是否能够通过某个算法解决，最终结合这些算法提出了一个解决方案，这个解决方案就是HTTPS，它是协议而非算法，是对多种密码学算法的工程应用。

### 2.1.2 密码学的四个目标

1）机密性（隐私性）

在网络中传递的数据如果具备机密性，那么传输的数据就是一串无意义的数字，只有拥有密钥的才能解释这些数据，密钥是加密算法的关键。在密码学中，对称加密算法和公开密钥算法都能够保证机密性。

2）完整性

完整性表示接收方能够确保接收到的数据就是发送方发送的原始数据，假设数据被中间人篡改，接收方如果有策略知晓数据被篡改了，那么传递的数据就具备完整性。在密码学中，主要使用消息验证码（MAC）算法保证完整性。需要注意的是互联网传输的数据即使是加密的也无法保证完整性，本章后续部分会进行详细的描述。

3）身份验证

互联网应用一般都有发送方和接收方，对于接收方来说，必须确认发送方的身份，才能确保收到的数据就是真实发送方发送的。反之对于发送方来说也是一样的，通信双方必须确保对端就是要通信的对象。在密码学中，一般使用数字签名技术确认身份。

4）不可抵赖性

### 2.1.3 OpenSSL

密码学原理是公开的，在工程上需要实现各种算法，最著名的就是OpenSSL项目，包括了底层密码库和命令行工具，大部分Linux发行版都预装了OpenSSL库。

很多应用软件都不是自行实现各种密码学算法，一般都直接调用OpenSSL密码库。

OpenSSL是密码学中一个非常流行的底层密码库，相对来说是值得信赖的。

## 2.2 随机数

在密码学中随机数的用途非常大，其他密码学算法内部都会用到随机数。

从开发者直观的角度看，随机数就是一串杂乱无序的字母、数字、符号组合，但这不是真正的随机数，更不能用在密码学中，为了理解随机数的本质，先介绍随机数的类型。

### 2.3.1 加密基元

在介绍密码学Hash算法之前，简单介绍加密基元的概念，大部分算法完成的功能是单一的，很少有某个算法能够解决密码学中的所有问题。加密基元就是一些基础的密码学算法，通过它们才能够构建更多的密码学算法、协议、应用程序。加密基元类似于房屋的内部材料（砖头、水泥），基于材料搭建出房屋，才真正对人类有用。

### 2.3.3 密码学Hash算法的特性

密码学Hash算法的主要特性如下：

* 相同的消息总是能得到同样的摘要值，特定的Hash算法，不管消息长度是多少，最终的摘要值长度是相同的。
* 不管多长的消息，Hash运算非常快速，这是非常重要的特性。
* 通过摘要值很难逆向计算出原始消息，Hash算法具备单向性，摘要值是不可逆的，这也是非常重要的特性。为了逆向计算出原始消息，唯一的方法就是采用暴力攻击、字典攻击、彩虹表，对不同的消息组合进行迭代运算，运算的结果如果匹配该消息的摘要值，表示该Hash算法不应该用于密码学。
* 原始消息一旦修改，即使是很轻微的修改，最终的摘要值也会产生变化。
* 很难找出两个不同的消息，并且它们的摘要值是相同的。
  
从密码学的角度考虑，Hash算法能够实现密码学的某个目标，那就是消息防篡改。

密码学Hash算法有很多，比如MD5算法、SHA族类算法，MD5早已被证明是不安全的Hash算法了，目前使用最广泛的Hash算法是SHA族类算法。

SHA（Secure Hash Algorithms）算法是美国国家标准与技术研究院（NIST）指定的算法，SHA算法不是一个算法，而是一组算法，主要分为三类算法。

（1）SHA-1

SHA-1算法类似于MD5算法，输出的长度固定是160比特。

目前SHA-1算法在严谨的加密学中已经被证明是不安全的，但是在实际应用过程中不并代表就有安全问题，在现实世界中要构造出碰撞还是非常困难的，需要经过大量的运算，不过还是尽量使用SHA-2类的Hash算法。

（2）SHA-2

SHA-2算法是目前建议使用的Hash算法，截至目前是安全的，主要有四种算法，分别是SHA-256、SHA-512、SHA-224、SHA-384，输出的长度分别是256比特、512比特、224比特、384比特。

（3）SHA-3

SHA-3算法并不是为了取代SHA-2算法，而是一种在设计上和SHA-2完全不同的算法，主要有四种算法，分别是SHA3-256、SHA3-512、SHA3-224、SHA3-384，输出的长度分别是256比特、512比特、224比特、384比特。

## 2.4 对称加密算法

所谓数据加密，就是将一段数据处理成无规则的数据，除非有关键的密钥，否则谁也无法得知无规则数据的真实含义。在密码学中，用于数据加密的算法主要有两种，分别是对称加密算法（Symmetric-keyAlgorithms）和非对称加密算法（Asymmetrical Cryptography）。

对称加密算法有两种类型，分别是块密码算法（block ciphers）和流密码算法（streamciphers），表2-7和表2-8简单列举了常用的对称加密算法。

既然有这么多块密码算法，使用哪种算法呢？建议使用AES算法，该算法是对称加密算法的标准算法，后续也主要以AES算法讲解。

### 2.4.1 流密码算法

在介绍流密码算法之前，先简单介绍下一次性密码本（one-time pad）的概念，一次性密码本诞生了流密码算法。一次性密码本非常简单，大概原理如下：

* 明文与同样长度的序列进行XOR运算得到密文。
* 密文与加密使用的序列再进行XOR运算就会得到原始明文。

理解一次性密钥本之后，就可以大概明白流密码算法的工作原理了，以RC4流密码算法为例，关键就在于算法内部生成了一个伪随机的密钥流（keystream），密钥流的特点如下：

* 密钥流的长度和密钥长度是一样的。
* 密钥流是一个伪随机数，是不可预测的。
* 生成伪随机数都需要一个种子（seed），种子就是RC4算法的密钥，基于同样一个密钥（或者称为种子），加密者和解密者能够获取相同的密钥流。
  
有了密钥流，随后的加密解密就非常简单了，就是XOR运算。

### 2.4.2 块密码算法

块密码算法在运算（加密或者解密）的时候，不是一次性完成的，每次对固定长度的数据块（block）进行处理，也就是说完成一次加密或者解密可能要经过多次运算，最终得到的密文长度和明文长度是一样的。

数据块的长度就称为分组长度（block size），由于大部分明文的长度远远大于分组长度，所有要经过多次迭代运算才能得到最终的密文或明文，块密码算法有多种迭代模式（Block ciphermodes of operation），迭代模式也可以称为分组模式。

### 2.4.3 填充标准

为了正确和安全地使用密码算法，定义了很多标准，指导开发者使用密码学算法，在后面的密码学算法中也会讲解更多的标准。本节会涉及PKCS#7和PKCS#5标准，更确切地说是这两个标准中的填充机制标准。

PKCS#5和PKCS#7处理填充机制的方式其实是一样的，只是PKCS#5处理的分组长度只能是8字节，而PKCS#7处理的分组长度可以是1到255任意字节，从这个角度看，可以认为PKCS#5是PKCS#7标准的子集。AES算法中分组长度没有8字节，所以AES算法使用PKCS#7标准。

标准的好处：

* 约定俗成，通信双方约定AES算法使用标准（比如AES-128-CBC-PKCS#7），填充标准是PKCS#7，密钥长度是128比特，分组模式是CBC, AES算法默认分组长度是128比特，双方基于同样的标准处理加密和解密。
* 标准代表严谨，能够成为标准，必然是经过充分验证的，可以安全使用。

从安全的角度看，初始化向量应该是随机的，不容易预测的，推荐使用随机数生成器生成初始化向量，初始化向量长度等同于分组长度。

```
$ ./gmssl list --cipher-algorithms
AES-128-CBC
AES-128-CBC-HMAC-SHA1
AES-128-CBC-HMAC-SHA256
AES-128-CFB
AES-128-CFB1
AES-128-CFB8
AES-128-CTR
AES-128-ECB
AES-128-OCB
AES-128-OFB
AES-128-XTS
AES-192-CBC
AES-192-CFB
AES-192-CFB1
AES-192-CFB8
AES-192-CTR
AES-192-ECB
AES-192-OCB
AES-192-OFB
AES-256-CBC
AES-256-CBC-HMAC-SHA1
AES-256-CBC-HMAC-SHA256
AES-256-CFB
AES-256-CFB1
AES-256-CFB8
AES-256-CTR
AES-256-ECB
AES-256-OCB
AES-256-OFB
AES-256-XTS
AES128 => AES-128-CBC
AES192 => AES-192-CBC
AES256 => AES-256-CBC
BF => BF-CBC
BF-CBC
BF-CFB
BF-ECB
BF-OFB
CAMELLIA-128-CBC
CAMELLIA-128-CFB
CAMELLIA-128-CFB1
CAMELLIA-128-CFB8
CAMELLIA-128-CTR
CAMELLIA-128-ECB
CAMELLIA-128-OFB
CAMELLIA-192-CBC
CAMELLIA-192-CFB
CAMELLIA-192-CFB1
CAMELLIA-192-CFB8
CAMELLIA-192-CTR
CAMELLIA-192-ECB
CAMELLIA-192-OFB
CAMELLIA-256-CBC
CAMELLIA-256-CFB
CAMELLIA-256-CFB1
CAMELLIA-256-CFB8
CAMELLIA-256-CTR
CAMELLIA-256-ECB
CAMELLIA-256-OFB
CAMELLIA128 => CAMELLIA-128-CBC
CAMELLIA192 => CAMELLIA-192-CBC
CAMELLIA256 => CAMELLIA-256-CBC
CAST => CAST5-CBC
CAST-cbc => CAST5-CBC
CAST5-CBC
CAST5-CFB
CAST5-ECB
CAST5-OFB
ChaCha20
ChaCha20-Poly1305
DES => DES-CBC
DES-CBC
DES-CFB
DES-CFB1
DES-CFB8
DES-ECB
DES-EDE
DES-EDE-CBC
DES-EDE-CFB
DES-EDE-ECB => DES-EDE
DES-EDE-OFB
DES-EDE3
DES-EDE3-CBC
DES-EDE3-CFB
DES-EDE3-CFB1
DES-EDE3-CFB8
DES-EDE3-ECB => DES-EDE3
DES-EDE3-OFB
DES-OFB
DES3 => DES-EDE3-CBC
DESX => DESX-CBC
DESX-CBC
IDEA => IDEA-CBC
IDEA-CBC
IDEA-CFB
IDEA-ECB
IDEA-OFB
RC2 => RC2-CBC
RC2-40-CBC
RC2-64-CBC
RC2-CBC
RC2-CFB
RC2-ECB
RC2-OFB
RC4
RC4-40
RC4-HMAC-MD5
SEED => SEED-CBC
SEED-CBC
SEED-CFB
SEED-ECB
SEED-OFB
SMS4 => SMS4-CBC
SMS4-CBC
SMS4-CCM
SMS4-CFB
SMS4-CFB1
SMS4-CFB8
SMS4-CTR
SMS4-ECB
SMS4-GCM
SMS4-OCB
SMS4-OFB
SMS4-WRAP
SMS4-WRAP-PAD
SMS4-XTS
ZUC
ZUC256
AES-128-CBC
AES-128-CBC-HMAC-SHA1
AES-128-CBC-HMAC-SHA256
id-aes128-CCM
AES-128-CFB
AES-128-CFB1
AES-128-CFB8
AES-128-CTR
AES-128-ECB
id-aes128-GCM
AES-128-OCB
AES-128-OFB
AES-128-XTS
AES-192-CBC
id-aes192-CCM
AES-192-CFB
AES-192-CFB1
AES-192-CFB8
AES-192-CTR
AES-192-ECB
id-aes192-GCM
AES-192-OCB
AES-192-OFB
AES-256-CBC
AES-256-CBC-HMAC-SHA1
AES-256-CBC-HMAC-SHA256
id-aes256-CCM
AES-256-CFB
AES-256-CFB1
AES-256-CFB8
AES-256-CTR
AES-256-ECB
id-aes256-GCM
AES-256-OCB
AES-256-OFB
AES-256-XTS
aes128 => AES-128-CBC
aes128-wrap => id-aes128-wrap
aes192 => AES-192-CBC
aes192-wrap => id-aes192-wrap
aes256 => AES-256-CBC
aes256-wrap => id-aes256-wrap
bf => BF-CBC
BF-CBC
BF-CFB
BF-ECB
BF-OFB
blowfish => BF-CBC
CAMELLIA-128-CBC
CAMELLIA-128-CFB
CAMELLIA-128-CFB1
CAMELLIA-128-CFB8
CAMELLIA-128-CTR
CAMELLIA-128-ECB
CAMELLIA-128-OFB
CAMELLIA-192-CBC
CAMELLIA-192-CFB
CAMELLIA-192-CFB1
CAMELLIA-192-CFB8
CAMELLIA-192-CTR
CAMELLIA-192-ECB
CAMELLIA-192-OFB
CAMELLIA-256-CBC
CAMELLIA-256-CFB
CAMELLIA-256-CFB1
CAMELLIA-256-CFB8
CAMELLIA-256-CTR
CAMELLIA-256-ECB
CAMELLIA-256-OFB
camellia128 => CAMELLIA-128-CBC
camellia192 => CAMELLIA-192-CBC
camellia256 => CAMELLIA-256-CBC
cast => CAST5-CBC
cast-cbc => CAST5-CBC
CAST5-CBC
CAST5-CFB
CAST5-ECB
CAST5-OFB
ChaCha20
ChaCha20-Poly1305
des => DES-CBC
DES-CBC
DES-CFB
DES-CFB1
DES-CFB8
DES-ECB
DES-EDE
DES-EDE-CBC
DES-EDE-CFB
des-ede-ecb => DES-EDE
DES-EDE-OFB
DES-EDE3
DES-EDE3-CBC
DES-EDE3-CFB
DES-EDE3-CFB1
DES-EDE3-CFB8
des-ede3-ecb => DES-EDE3
DES-EDE3-OFB
DES-OFB
des3 => DES-EDE3-CBC
des3-wrap => id-smime-alg-CMS3DESwrap
desx => DESX-CBC
DESX-CBC
id-aes128-CCM
id-aes128-GCM
id-aes128-wrap
id-aes128-wrap-pad
id-aes192-CCM
id-aes192-GCM
id-aes192-wrap
id-aes192-wrap-pad
id-aes256-CCM
id-aes256-GCM
id-aes256-wrap
id-aes256-wrap-pad
id-smime-alg-CMS3DESwrap
idea => IDEA-CBC
IDEA-CBC
IDEA-CFB
IDEA-ECB
IDEA-OFB
rc2 => RC2-CBC
rc2-128 => RC2-CBC
rc2-40 => RC2-40-CBC
RC2-40-CBC
rc2-64 => RC2-64-CBC
RC2-64-CBC
RC2-CBC
RC2-CFB
RC2-ECB
RC2-OFB
RC4
RC4-40
RC4-HMAC-MD5
seed => SEED-CBC
SEED-CBC
SEED-CFB
SEED-ECB
SEED-OFB
sms4 => SMS4-CBC
SMS4-CBC
SMS4-CCM
SMS4-CFB
SMS4-CFB1
SMS4-CFB8
SMS4-CTR
SMS4-ECB
SMS4-GCM
SMS4-OCB
SMS4-OFB
SMS4-WRAP
SMS4-WRAP-PAD
SMS4-XTS
ZUC
ZUC256
```

加密：

```
$ gmssl enc -aes-256-cbc -salt -in temp.txt -out gm_temp.enc -p

enter aes-256-cbc encryption password:
Verifying - enter aes-256-cbc encryption password:
salt=767D5FD89F7402F7
key=E82734FEE48D60FA1377EAE83D02FA800094F12635618DA24B0697785D048BCF
iv =E79E9E6B29A7C7D75A02CE9DC680B77F
```

* AES算法使用的密钥通过口令和Salt生成，同样的口令和Salt会生成同样的密钥。
* Salt的主要作用是为了保证同样的口令可以生成不同的密钥，是明文传输的。

在file.enc文件中包含salt和初始化向量值，这两个值不用加密也没法加密，因为解密的时候要用。

解密：

```
$ gmssl enc -d -aes-256-cbc -in gm_temp.enc
enter aes-256-cbc decryption password:
hello gmssl!
```

如果对于对称加密算法比较了解，可以通过OpenSSL命令行显式地输入初始化向量、密钥，比如：

```
$ gmssl enc -aes-256-cbc -in temp.txt -out gm_temp.enc -iv E9EDACA1BD7090C6 -K 89D4B1678D604FAA3DBFFD030A314B29
$ gmssl enc -d -aes-256-cbc -in gm_temp.enc -iv E9EDACA1BD7090C6 -K 89D4B1678D604FAA3DBFFD030A314B29
```

-iv表示初始化向量，-K表示密钥，密钥长度和初始化向量长度如果输入错误，OpenSSL命令行会报错。

实际上OpenSSL命令行默认使用的填充标准就是PKCS#7标准。

## 2.5 消息验证码

Hash算法能够完成密码学目标之一的完整性校验，但却不能避免消息被篡改，为避免消息被篡改，需要用到消息验证码。消息验证码非常重要，一般结合加密算法一起使用。

### 2.5.1 什么是消息验证码

在密码学应用中，很多情况下，传递的消息没有必要加密，只要确保消息是完整且没有被篡改即可。比如开发者开发了一组天气API，接口返回的数据并没有加密，原因可能如下：

* 接口的数据并不重要，对隐私性要求不高。
* 加密和解密过程很消耗性能。
  
攻击者对消息进行拦截，同时修改接口消息和消息的摘要值然后发送给接收方，接收方收到消息后，对接口消息计算摘要值，然后与接收到的摘要值进行比较，如果相同，接收方认为消息是完整的。可实际呢？消息虽然是完整的，但被篡改了，或者说消息被伪装了，但对于接收方来说，仅仅通过摘要值无法验证消息是不是篡改了，这时候需要使用MAC算法。

消息验证码算法的特点：

* 证明消息没有被篡改，这和Hash算法类似。
* 消息是正确的发送者发送的，也就是说消息是经过验证的。

注意，消息验证和身份验证是不同的概念。

如何确保消息是特定人发送的呢？在通信双方可以维护同一个密钥，只有拥有密钥的通信双方才能生成和验证消息验证码，消息验证码算法需要一个密钥，这和对称加密算法是一样的，通信双方在消息传递之前需要获得同样的密钥。

消息验证码的模型很简单：

MAC值=mac（消息，密钥）

MAC值一般和原始消息一起传输，原始消息可以选择加密，也可以选择不加密，通信双方会以相同的方式生成MAC值，然后进行比较。

### 2.5.2 MAC算法的种类

在密码学中，MAC算法有两种形式，分别是CBC-MAC算法和HMAC算法。CBC-MAC算法从块密码算法的CBC分组模式演变而来，简单地说就是最后一个密文分组的值就是MAC值。

HMAC（Hash-based Message Authentication Code）算法使用Hash算法作为加密基元，HMAC结合Hash算法有多种变种，比如HMAC-SHA-1、HMAC-SHA256、HMAC-SHA512。

### 2.5.3 消息验证码算法实践

通过sha-1算法计算摘要值：

```
$ gmssl dgst -sha1 temp.txt 
SHA1(temp.txt)= 49b694f69541253894b784bf1e128d188c7e9287
```

计算摘要值，并保存到digest.txt 文件：

```
$ gmssl sha1 -out digest.txt temp.txt
```

使用HMAC-SHA-1算法，结合密钥"mykey"对文件计算HMAC值

```
$ gmssl dgst -sha1 -hmac "mykey" temp.txt
HMAC-SHA1(temp.txt)= bbe3e59a70670ec477992ab61e5f73aafc8ec17d
```

### 2.5.4 加密算法不能提供完整性

加密算法不能提供完整性，加密的同时必须引入MAC算法避免消息被篡改。

### 2.5.5 AE加密模式

使用者结合对称加密算法和MAC算法，提供机密性和完整性的模式也叫作Authenticated Encryption（AE）加密模式，主要有三种，简单介绍如下。

1）Encrypt-and-MAC (E&M)

这种模式（图2-11）就是对消息分别进行加密运算和MAC运算，然后将两个运算结果结合起来发送给接收方。

2）MAC-then-Encrypt (MtE)

这种模式（图2-12）先对消息进行MAC计算，然后将消息和MAC值组合在一起再进行加密，最终的加密值再发送给接收方。在HTTPS中，一般使用这种模式进行处理，比如AES-128-CBC#PKCS7-HMAC-SHA256模式。

3）Encrypt-then-MAC (EtM)

这种模式（图2-13）先对消息进行加密得到密文，然后对密文再计算MAC值，最终将密文和MAC值组合在一起再发送给接收方。

不管是Encrypt-and-MAC模式还是MAC-then-Encrypt模式，使用不当的话都会存在安全问题，目前建议使用Encrypt-then-MAC模式。

### 2.5.6 AEAD加密模式

AEAD（Authenticated Encryption with Associated Data）是AE加密模式的一种变体，AE模式需要使用者单独处理加密运算和MAC运算，一旦使用不当，就很容易出现安全问题。

AEAD加密模式在底层组合了加密算法和MAC算法，能够同时保证数据机密性和完整性，减轻了使用者的负担，主要有三种模式，接下来分别介绍。

1）CCM模式

CCM（Counter with CBC-MAC）模式也是一种AEAD模式，不过在HTTPS中使用得比较少。这种模式使用CBC-MAC（一种MAC算法）算法保证完整性，使用块密码AES算法CTR模式的一种变种进行加密运算，底层采用的是MAC-then-Encrypt模式。

2）GCM模式

GCM（Galois/Counter Mode）是目前比较流行的AEAD模式。在GCM内部，采用GHASH算法（一种MAC算法）进行MAC运算，使用块密码AES算法CTR模式的一种变种进行加密运算，在效率和性能上，GCM都是非常不错的。

3）ChaCha20-Poly1305

ChaCha20-Poly1305是谷歌发明的一种算法，使用ChaCha20流密码算法进行加密运算，使用Poly1305算法进行MAC运算。

## 2.6 公开密钥算法

公开密钥算法不是一个算法而是一组算法，如果公开密钥算法用于加密解密运算，习惯上称为非对称加密算法。

公开密钥算法和对称加密算法的一些异同。

1）功能不一样

对称加密算法虽然有很多的算法和加密机制，但主要用于加密和解密。而公开密钥算法的功能比较多，可以进行加密解密、密钥协商、数字签名。

2）密钥是一对

对称加密算法中，密钥是一串数字，加密者和解密者使用同样的一个密钥。公开密钥算法之所以包含公开两字，表示密钥可以部分公开，公开密钥算法的密钥是一对，分别是公钥（publickey）和私钥（private key），一般私钥由密钥对的生成方（比如服务器端）持有，避免泄露，而公钥任何人都可以持有，也不怕泄露。

3）运算速度很慢

相比对称加密算法来说，公开密钥算法尤其是RSA算法运算非常缓慢，一般情况下，需要加密的明文数据都非常大，如果使用公开密钥算法进行加密，运算性能会惨不忍睹。公开密钥算法在密码学中一般进行密钥协商或者数字签名，因为这两者运算的数据相对较小。

公开密钥算法最重要和最广泛使用的算法就是RSA算法，该算法是Ron Rivest、Adi Shamir、Leonard Adleman三个人创建的，以三个人名字的首字母命名。RSA算法是一个多用途的算法，可以进行加密解密、密钥协商、数字签名，需要重点理解。

### 2.6.1 理解RSA的内部结构

对称加密算法中的密钥是一串数字，没有太多的其他含义，而RSA算法中的公钥和私钥在生成的时候有很大的关系，公钥和私钥不只是一串数字，由很多参数组成，公钥和私钥一般以文件的形式提供。

加解和解密过程也需要密钥文件中的其他参数，在理解公开密钥算法的时候，首先要掌握密钥文件的内部结构。

1）密钥文件生成过程

密钥文件（包含公钥和私钥）内部结构大概如下：

```
typedef struct rsa_t {
    BIGNUM *p;
    BIGNUM *q;
    BIGNUM *n;
    BIGNUM *e;
    BIGNUM *d;
} RSA;
```

接下来看看如何生成密钥对：

* 选取两个很大的质数p和q。
* 求这两个数的乘积n。
* 取一个公开指数e，这个数的值小于(p-1)(q-1), e对应的值和(p-1)(q-1)的值互质。
* e和n组合起来就相当于公钥。n值的长度就相当于密钥对的长度。
* 通过e、p、q能够计算出私钥d, d和n组合起来就是私钥。一旦计算出私钥，p和q这两个数就可以丢弃，这样更安全。如果不丢弃且和d一同保存在私钥文件中，则运算的时候效率更高。e和d之间存在互逆的关系。

2）加密过程

```
C = M ^ e (mod n)
```
这个过程表示对明文M进行多次幂运算，运算的次数就是公钥，计算出值后再进行模运算（mod n），最终得到的C就是密文。

对密文进行d次的幂运算，然后进行模运算，最终得到明文M。

```
M = C ^ d (mod n)
```

3）RSA算法安全性

幂运算的逆过程就是求对数问题，而模运算可以认为是离散问题，组合起来RSA算法就是离散对数模型，只要密钥长度足够长，离散对数很难破解。

安全性和密钥对的长度有关，也就是和n这个值有关，它可以称为密钥长度，破解私钥有两个方法：

* 公钥持有人有e和n，而要计算出私钥d，需要知道p和q，想通过一个巨大的数字n获取p和q是一个因式分解问题，暴力破解很难。
* 攻击者假如想通过密文和公钥破解私钥，就要求解决离散对数问题，更是难上加难。
  
### 2.6.2 PKCS标准

和对称密钥算法一样，公开密钥算法也有使用标准，公开密钥算法的标准称为PKCS（PublicKey Cryptography Standards），这个标准由很多的子标准组成，指导使用者正确地使用公开密钥算法。

PKCS标准最早是由RSA公司制定的，目前逐步交由标准化组织IETF（Internet EngineeringTask Force）的PKIX工作组来维护。表2-10描述了PKCS标准的所有子标准，致力于密码学的读者可以以此了解密码学知识。

*表2-10 PKCS标准的所有子标准*

和AES算法一样，RSA算法也有填充标准，使用RSA算法进行加密解密的时候，如果使用不当很容易出现安全问题。比如说同样的明文、同样的密钥经过RSA加密，如果每次得到的密文都是相同的，破解的风险就比较大。为了解决类似的安全问题，PKCS#1标准定义了两种机制，用于处理填充问题，从而保证同样的明文、同样的密钥经过RSA加密，每次的密文都是不一样的。

两种填充机制分别是RSAES-PKCS1-V1_5和RSAES-OAEP，两者在命名上显得有点奇怪，RSAES-PKCS1-V1_5其实是PKCS#1 v1.5以前的版本，而RSAES-OAEP可以认为是PKCS#1v2.0以后的版本，只是官方称之为RSAES-OAEP。

目前推荐使用的填充标准是RSAES-OAEP, OpenSSL命令行默认使用的标准是RSAES-PKCS1-V1_5。

### 2.6.3 RSA加密算法的应用场景

1）单步加密

公钥是公开的，很多客户端知道，而私钥必须由服务器端保密，所以一般客户端用公钥加密的方式传递一些关键数据，比如客户端可以对自己的信用卡号加密，然后传递给服务器端，服务器端解密后无须回应，这就是单步加密的模式。

2）双向加密

在单步加密过程中，服务器端无法发送密文，如果服务器端用私钥加密数据，然后发送给客户端，由于公钥是公开的，任何人都能解密，所以这个过程是不成立的。

举个例子来解释什么是双向加密，完成的功能是用户要查询账户（身份证）下还有多少余额：

* 客户端生成一对RSA密钥对，然后连接服务器端，并将自己的公钥（client Public Key）发给服务器端。
* 服务器端接收请求后，保存客户端的公钥，然后生成另外一对RSA密钥对，并将公钥（server PublicKey）发送给客户端。
* 客户端使用服务器的公钥（server Public Key）加密身份证号，加密的数据发送给服务器端，期待服务器端返回自己的账户余额。
* 服务器端接收到数据后，用自己的私钥（server Private Key）解密出客户端的身份证号，然后查询出用户的余额，并用客户端的公钥（client Public Key）加密余额，并发送给客户端。
* 客户端用自己的私钥（Client Private Key）解密接收到的数据，这个数据就是自己账户下的余额。

这就是双向加密，如果不考虑性能问题，RSA算法确实可以完成数据加密。

### 2.6.4 RSA加密算法实践

（1）使用genrsa子命令生成密钥对，密钥对是一个文件：

```
$ gmssl genrsa -out mykey.pem 2048
Using configuration from /home/alex/bin/gmssl/ssl/openssl.cnf
Generating RSA private key, 2048 bit long modulus
..................+++
..................+++
e is 65537 (0x010001)
```

mykey.pem文件中包含e、n、d等相关信息，在加密解密的时候都要加载密钥文件。

对pem文件使用3DES算法进行加密保护：

```
$ gmssl genrsa -des3 -out mykey2.pem 2048
Using configuration from /home/alex/bin/gmssl/ssl/openssl.cnf
Generating RSA private key, 2048 bit long modulus
...............................................+++
....................................................................................+++
e is 65537 (0x010001)
Enter pass phrase for mykey2.pem:
Verifying - Enter pass phrase for mykey2.pem:
```

-des3表示mykey2.pem文件使用3DES算法进行加密保护，3DES算法的密钥是通过口令生成的，关于口令的概念后续会讲解，目前可以简单地认为口令就是一个弱密钥。

（2）从密钥对中分离出公钥：

```
$ gmssl rsa -in mykey.pem -pubout -out mypubkey.pem
Using configuration from /home/alex/bin/gmssl/ssl/openssl.cnf
writing RSA key
```

mykey2.pem文件需要口令才能分离出公钥：

```
$ gmssl rsa -in mykey2.pem -pubout -out mypubkey2.pem
Using configuration from /home/alex/bin/gmssl/ssl/openssl.cnf
Enter pass phrase for mykey2.pem:
writing RSA key
```

（3）校验密码对文件是否正确：

```
$ gmssl rsa -in mykey.pem -check
Using configuration from /home/alex/bin/gmssl/ssl/openssl.cnf
RSA key ok
writing RSA key
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAopgaLlcMsTGXqN/mwZb9qkM3xI0bW3peWs/n0tTu68pdNwAj
dRbNkaaKSvkHiY+imf+0KlNSukx2B3raWE20nONgLulgVAtoj6SZKl/P0eG9MwFn
DIiAjEkQ9ABYBn6FLQkxYX/9IaVpq08KgQr9NDfLshj2GnZK74ZQ+gkac5u19Ukp
zdddrFNyCeLChUGamygboBc/Z2jcwUdIMUCDVHOrdLs69SOa/XifLqifmV+AlZQH
4P/7xewxj5q+WtXVaupUt9/j11BfMtfQuui/yNVNsTObmBlt9zTdZJuUdjSp3a+y
m1ZBQ41kc89CtgXSk8bgru7HY+UMcs5C+y8MTQIDAQABAoIBAHrvplwDVYVkidcV
89PL5mAmErR6PIUeMNlY+V9fwIZnq7M6V5TgeO33jfjB8gEAqWDnBcI63gIebd+Z
9B1wI2+8O608p6jkN1rkiSqQ5wE6PWOjW9cOHqPzWu4ilGvUnb0/hibCLxKRjyQH
z1lihbBFv64ZUXsQlfglBnUHDQj7LjYa1+Dv5QIHXotBrxhAtDzBJuWIbY32VX+i
jmKLTlgHxBWZboTDv2P5SGgX1IJXcyBJ+QgxGeBG+2x5aMVUaTqho8hs1YeLugSO
dnwbobiEhTbX8cA5ldE5eH7AYL6H0gmMMgJlvKPn0eB2FDim5J8cPgHQQ045Vl/m
5jhL6AECgYEAz96lOPbySHHgMBnEAou8eKn1Hn0DPI8f2FNJF8VIjTllIznpyWuN
+sndszSCZVacmexFPc2p99zdgNfJNmkvOwSYRUpyh+fTFEbTapE2LsgyoH35Rqkm
m6qhPbrKj53xgWRpL14qsr6cuulj61CXkwnQFIT6mdDJmUwNTYa5ne0CgYEAyD3G
VoioYRnRZ7NH3IYgeFXvRVkPHkFTqaMvTL99ClcelqBWlqlQgAG3SwefLXp+19M7
UOJEETYs4jbCoAE0nx3FYUrKCEKjuMbGrVybBPQ0QPBJlF/jFbHBx+BoyVWyaphd
br8SmSH45gFo1l6YVigsC71t6SOysez88FWIW+ECgYB2lXI2JBKlp2kYp6o9NZBI
WdS/Ftwg0Rl+pEyfZel0v1hmFyS6xkPR3RU/pWX5/8YIvVPm5QvgnbwzQ2bDRpAu
H/nqFYVu6J5vA9SaB8scNxNCoXryh47B4T5o48Wo1paulSS4ZAUBwWHR81EQLgK6
XC+7dP0tgIFxlYRFROVhJQKBgDtuMj6eorLnEcKgcDSgTmTIxJIlg5osM2OGvlQe
BUObZcW44tomeHD1kWwgX/sEfz8ZP2KbNS6SkLG3JP6OPQr4sAtXQi0/cg42WOM9
N/k5bYTUjFIQP3rB3kyvawpOd/yxKhHjfeabMZ86Td5KBxaTJ7d4SnXGlZO/Tbca
+7ShAoGBAJrQ7bus+9Hfd1auhiPIaC1Ilgly6BEdvu+T0dQ0FhMVjM+d9I15qVAC
iu7f4M6WArZRViyxuU0tvd269TOsX+ttYDyuzBIL756Zl9aeT6868YbSAqKMZedn
kMv/FsaxqFpmIALVPVBIUeFkj4jt2ZzhFmRoX0sNITyIS0VirsZs
-----END RSA PRIVATE KEY-----
```

（4）显示公钥信息：

```
$ openssl rsa -pubin -in mypubkey.pem -text
Using configuration from /home/alex/bin/gmssl/ssl/openssl.cnf
Public-Key: (2048 bit)
Modulus:
    00:a2:98:1a:2e:57:0c:b1:31:97:a8:df:e6:c1:96:
    fd:aa:43:37:c4:8d:1b:5b:7a:5e:5a:cf:e7:d2:d4:
    ee:eb:ca:5d:37:00:23:75:16:cd:91:a6:8a:4a:f9:
    07:89:8f:a2:99:ff:b4:2a:53:52:ba:4c:76:07:7a:
    da:58:4d:b4:9c:e3:60:2e:e9:60:54:0b:68:8f:a4:
    99:2a:5f:cf:d1:e1:bd:33:01:67:0c:88:80:8c:49:
    10:f4:00:58:06:7e:85:2d:09:31:61:7f:fd:21:a5:
    69:ab:4f:0a:81:0a:fd:34:37:cb:b2:18:f6:1a:76:
    4a:ef:86:50:fa:09:1a:73:9b:b5:f5:49:29:cd:d7:
    5d:ac:53:72:09:e2:c2:85:41:9a:9b:28:1b:a0:17:
    3f:67:68:dc:c1:47:48:31:40:83:54:73:ab:74:bb:
    3a:f5:23:9a:fd:78:9f:2e:a8:9f:99:5f:80:95:94:
    07:e0:ff:fb:c5:ec:31:8f:9a:be:5a:d5:d5:6a:ea:
    54:b7:df:e3:d7:50:5f:32:d7:d0:ba:e8:bf:c8:d5:
    4d:b1:33:9b:98:19:6d:f7:34:dd:64:9b:94:76:34:
    a9:dd:af:b2:9b:56:41:43:8d:64:73:cf:42:b6:05:
    d2:93:c6:e0:ae:ee:c7:63:e5:0c:72:ce:42:fb:2f:
    0c:4d
Exponent: 65537 (0x10001)
writing RSA key
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAopgaLlcMsTGXqN/mwZb9
qkM3xI0bW3peWs/n0tTu68pdNwAjdRbNkaaKSvkHiY+imf+0KlNSukx2B3raWE20
nONgLulgVAtoj6SZKl/P0eG9MwFnDIiAjEkQ9ABYBn6FLQkxYX/9IaVpq08KgQr9
NDfLshj2GnZK74ZQ+gkac5u19UkpzdddrFNyCeLChUGamygboBc/Z2jcwUdIMUCD
VHOrdLs69SOa/XifLqifmV+AlZQH4P/7xewxj5q+WtXVaupUt9/j11BfMtfQuui/
yNVNsTObmBlt9zTdZJuUdjSp3a+ym1ZBQ41kc89CtgXSk8bgru7HY+UMcs5C+y8M
TQIDAQAB
-----END PUBLIC KEY-----
```

其中Public-Key表示密钥的长度。Modulus的值表示公开密钥系数，就是RSA结构中的n。Exponent的值表示公钥，就是RSA结构中的e。-----BEGIN PUBLIC KEY-----和-----END PUBLICKEY-----之间表示公钥具体的值。

（5）RSA加密

用公钥加密：

```
$ openssl rsautl -encrypt -pubin -inkey mypubkey.pem -in temp.txt -out gm_rsa_cipher.txt
Using configuration from /home/alex/bin/gmssl/ssl/openssl.cnf
```

用私钥解密：

```
$ openssl rsautl -decrypt -inkey mykey.pem -in gm_rsa_cipher.txt 
Using configuration from /home/alex/bin/gmssl/ssl/openssl.cnf
hello gmssl!
```

rsautl命令默认的填充机制是PKCS#1 v1.5，可以指定-oaep参数使用PKCS#1 OAEP机制

## 2.7 密钥

密钥是密码学中非常关键的概念，一般情况下，密钥一旦被截获，密文就能够被解密。密钥最重要的属性就是密钥的长度，密钥长度决定了密钥空间的大小，如果密钥长度过短，很容易受到暴力攻击。密码学的算法是公开的，一般很难破解，密钥是攻击者可能采取的攻击手段，攻击者可以不断猜测密钥，最多经过216次运算（生成各种不同的密钥）就能破解密文，暴力攻击就是不断地迭代密钥进行攻击。

为了避免暴力破解，不同密码学算法的密钥应该保证一定长度，比如AES算法安全的密钥长度是128比特，密钥长度足够长也不代表安全，密钥应该是随机、无法预测的。

那么密钥到底是什么？从两个维度考虑：

* 对称加密算法、MAC算法使用的密钥就是一串数字。
* 公开密钥算法中的密钥是一对，由多个部分组成，但本质上也可以认为由多个数字组成。

密钥虽然是简单的数字，但在实际使用过程中还是复杂的，涉及密钥生成、存储、传输等一系列的工作。

### 2.7.1 生成密钥

密钥最关键的特性：

* 足够的长度，达到一定长度才能保证算法安全性。
* 不可预测性，不能是简单的数字、字母组合，否则即使长度足够，密钥本身也容易被破解。
 
在密码学中，为了生成密钥，一般采用两种方法：

* 基于伪随机生成器生成密钥。
* 基于口令的加密（Password-based Encryption，简称PBE）算法产生密钥。

使用伪随机数生成器（PRNG）生成的密钥足够随机，很难预测，对于人类来说很难记住，同时由于不具备可预测性，攻击者很难对密钥本身进行攻击，除非该密钥泄露了。

PBE算法生成的密钥一般情况下无须存储，因为使用同样的口令就能生成同样的密钥，这是其优点之一，PBE算法生成的密钥有时候并不是为了使用该密钥，而有其他用途，这是非常重要的一个概念。接下来重点讲解口令的概念、口令和密钥的区别、口令的作用、各种PBE算法。

### 2.7.2 口令和PEB算法

口令（password或者passphrase）也可以认为是一种密钥，都需要保密，不能泄露。口令和密钥最大的区别在于口令更容易生成、更容易记忆，一般情况下口令记录在人脑中，口令可以认为是一种弱密钥，由固定的字母、数字、符号组成，长度也有一定的限制。

在密码学中很少直接用口令进行加密，容易受到暴力攻击和字典攻击，暴力攻击的原理在于口令都是由固定的字母、数字、符号组成的，攻击者可以生成所有可能的口令，然后使用口令迭代去解密，一旦成功解密，就表示口令被暴力破解了。

字典攻击本质上也是一种暴力攻击，只是能够加快破解效率（时间和空间），人类一般使用常见的字母、数字、符号组合成口令（比如很多人喜欢用字母password作为口令），攻击者可以将常见的口令保存在一张字典中，然后用字典中的口令迭代去解密密文。除了字典攻击，还有彩虹表攻击方式，破解的关键点就在于口令相对容易猜测和预测。

1）口令用于身份校验

很多程序员喜欢使用摘要函数处理用户的口令，然后将摘要值（密钥）存储到数据库中，基于的原理就在于摘要算法的摘要值不能计算出原有口令，本质上这也是一种基于口令的密钥生成算法，当然这种算法是不严谨的，存在安全问题，攻击者仍然能够采用字典攻击，攻击方式大概如下：

* 某个系统的用户库被攻击者盗库，用户库包含了所有用户的信息，口令经过摘要运算以密钥的形式保存在用户库中。
* 攻击者虽然获取了用户库，但是不能冒充用户，原因在于攻击者不能使用用户库中的密钥进行登录。
* 攻击者的破解思路就是通过密钥破解原始口令，攻击者猜测系统使用某种摘要算法处理了用户口令。
* 攻击者对所有字典字符（包含了常见的口令）进行摘要计算，然后和用户库的密钥匹配，一旦成功匹配，就代表用户的口令泄露了。

这种简单处理口令的方式没有太大的安全性，相当于在用户库中明文存储口令。

在密码学中，存在一种密码衍生算法（Key Derivation Function, KDF），该算法可以简单理解为通过某些值可以生成任意长度的一个（多个）密钥，常见的KDF算法有很多，比如PBKDF2、bcrypt、scrypt等。

2）PBKDF2算法

PBE算法标准定义在RFC 2898文档中，描述了PBKDF2函数的实现细节：

```
DK = PBKDF2(PRF, Password, Salt, c, dkLen)
```

* PRF是一个伪随机函数，可以简单地理解为摘要算法。
* Password表示口令。
* Salt表示盐值，一个随机数。
* c表示迭代次数。
* dkLen表示最后输出的密钥长度。

OpenSSL命令行工具没有PBKDF2的子命令，写代码时可以引用OpenSSL库的PKCS5_PBKDF2_HMAC_SHA1函数完成运算。

### 2.7.3 密钥存储和传输

生成密钥后，需要考虑的问题就是密钥存储和密钥传输，存储和传输需要结合在一起考虑，接下来通过两个维度进行讲解。

1）静态密钥

有些密钥需要存储，有些密钥不需要存储，密钥可以存储到文件、数据库、专属的设备中。一旦密钥泄露了，面临的核心问题就是隐私被暴露了，密钥和密文是一样重要的，有了密钥相当于密文被解密了。需要存储的密钥是静态不变的，也称为长期密钥，在一定时间内都是生效的，除非主动更新或者废弃密钥。

对于个体、小范围团体的网络通信或非网络通信的应用来说，加密解密的操作者之间都是熟识的，可以通过简单的途径将密钥告诉给密钥使用者，一般有以下几种方式：

* 密钥硬编码在代码中。
* 以口头、邮件的方式传输密钥。

不管是将密钥硬编码在代码中还是通过邮件的方式传输，都要注意安全，静态密钥的有效期相对较长，密钥拥有者很难发现密钥泄露了，所以有必要经常更改密钥。

2）动态密钥

网络通信应用中，安全传输密钥是非常难以解决的问题：

* 对于一个网站来说，客户端用户非常多，服务器不可能给每个客户端分配同样的密钥，因为如果密钥相同，一个客户端就可以解密另外一个客户端传输的数据，如果给每个客户端分配不同的密钥，那么密钥存储的数据库容量就非常大，不存在可操作性。
* 对于一个网站来说，服务器端根本不知道客户端是谁，也不认识这些客户端，无法以邮件这样的方式通知每个客户端。
* 对于一个网站来说，加密解密需要的密钥需要通过其他的方式进行传输，每个TCP连接传输的密钥不一样，这就是动态密钥。

为了在网络通信中传输动态密钥，可以采用密码学中的密钥协商算法。

## 2.8 密钥协商算法

公开密钥算法的另外一种算法就是密钥协商算法。在网络通信应用中，密钥的管理和分配是个难题，尤其是生成一个动态密钥更难，而密钥协商算法就可以解决密钥分配、存储、传输等问题。

在网络通信中，为了加密解密数据，可以采用动态密钥，也叫作会话密钥，这个密钥有以下一些特点：

* 会话密钥的作用就是为了加密解密通信数据，也就是对称加密算法可以使用会话密钥进行加密解密。
* 在加密解密通信数据之前，客户端和服务器端需要协商出会话密钥，而会话密钥只有服务器端和特定的客户端才能知晓，不能泄露，这可以采用密钥协商算法解决。
* 会话密钥的意思就是该密钥不用存储，一旦客户端和服务器端的连接关闭，该密钥就会消失，也就是说密钥存储在客户端和服务器端的内存中，由于密钥不用存储，安全性就得到了很大的保障。

### 2.8.1 RSA密钥协商算法

通过一个例子看看RSA密钥协商算法如何工作的：

* 客户端初始化连接服务器端，服务器发送RSA密钥对的公钥给客户端。
* 客户端生成一个随机值，这个值必须是随机的，不能被攻击者猜出，这个值就是会话密钥。
* 客户端使用服务器RSA密钥对的公钥加密会话密钥，并发送给服务器端，由于攻击者没有服务器的私钥，所以无法解密会话密钥。
* 服务器端用它的私钥解密出会话密钥。
* 至此双方完成连接，接下来服务器端和客户端可以使用对称加密算法和会话密钥加密解密数据。

RSA密钥协商算法有几个优点：

* 每次连接阶段的会话密钥是不同的，无须存储到设备中，连接关闭后会话密钥就会消失。
* 每次连接中的会话密钥是不同的，避免了烦琐的会话密钥分配问题。
* 虽然RSA运算很慢，但由于会话密钥长度相对很小，计算的是数据量并不大，所以性能消耗相对可控。

RSA密钥协商算法也有缺点：

* 获取会话密钥过程其实并不能称为协商，完全是由客户端决定的，只能称为密钥传输。如果客户端生成会话密钥没有使用标准的算法，可能会带来安全隐患。比如说客户端每次随机从26个字母中选取4个字母作为会话密钥，那么很容易受到暴力攻击。攻击者不会去破解RSA加密算法的私钥，直接暴力破解会话密钥就能反解出明文。
* 最大的问题就是不能提供前向安全性，前向安全性是HTTPS中非常重要的概念。

### 2.8.2 DH密钥协商算法

DH算法确切地说，实现的是密钥交换或者密钥协商，DH算法在进行密钥协商的时候，通信双方的任何一方无法独自计算出一个会话密钥，通信双方各自保留一部分关键信息，再将另外一部分信息告诉对方，双方有了全部信息才能共同计算出相同的会话密钥。

客户端和服务器端协商会话密钥的时候，需要互相传递消息，消息即使被挟持，攻击者也无法计算出会话密钥，因为攻击者没有足够的信息（通信双方各自保留的信息）计算出同样的会话密钥。

1）参数文件

在使用DH算法之前，先要生成一些公共参数，这些参数是公开的，无须担心攻击者能够看到这些参数值，这些参数可以由客户端或者服务器端生成，一般由服务器端生成。参数在协商密钥之前必须发给对端。

```
typedef struct dh_st {
    BIGNUM *p;
    BIGNUM *g;
    BIGNUM *pub_key;
    BIGNUM *priv_key;
} DH;
```

参数有两个，分别是p和g, p是一个很大的质数，建议长度在1024比特以上，这个长度也决定了DH算法的安全程度，g表示为一个生成器，这个值很小，可以是2或者5。通过参数，服务器端和客户端会生各自生成一个DH密钥对，私钥需要保密。

2）DH算法处理过程

* 通信双方的任何一方可以生成公共参数p和g，这两个数是公开的，被截获了也没有任何关系，一般情况下由通信双方的服务器端计算。
* 客户端连接服务器端，服务器端将参数发送给客户端。
* 客户端根据公开参数生成一个随机数a，这个随机数是私钥，只有客户端知道，且不会进行发送，然后计算Yc = (g ^ a) mod p, Yc就是公钥，需要发送给服务器端。
* 服务器端根据公开参数生成一个随机数b，这个随机数是私钥，需要服务器端保密，然后计算Ys = (g ^ b) mod p, Ys是公钥，需要发送给客户端。
* 客户端发送Yc数值给服务器端，服务器端计算Z = (Yc ^ b) mod p。
* 服务器端发送Ys数值给发送方，客户端计算Z = (Ys ^ a) mod p。
* 服务器端和客户端生成的Z就是会话密钥，协商完成。

这里的关键点就是私钥a和b不应该泄露，分别由通信双方维护，另外Ys和Yc进行互换才能完成协商，这两个值被截获对攻击者来说没有任何价值。换句话说，只要私钥不发生泄露，攻击者即使有了Ys和Yc也不会计算出会话密钥。

### 2.8.3 DH算法分类

DH算法分为两种类型，分别是静态DH算法和临时DH算法。

1）静态DH算法（DH算法）

静态DH算法，p和g两个参数永远是固定的，而且服务器的公钥（Ys）也是固定的。和RSA密钥协商算法一样，一旦服务器对应的DH私钥泄露，就不能提供前向安全性。静态DH算法的好处就是避免在初始化连接时服务器频繁生成参数p和g，因为该过程是非常消耗CPU运算的。

2）临时DH算法（EDH算法）

在每次初始化连接的时候，服务器都会重新生成DH密钥对，DH密钥对仅仅保存在内存中，不像RSA那样私钥是保存在磁盘中的，攻击者即使从内存中破解了私钥，也仅仅影响本次通信，因为每次初始化的时候密钥对是动态变化的。更安全的是，协商出会话密钥后，a和b两个私钥可以丢弃，进一步提升了安全性，在有限的时间、有效的空间生成了密钥对。

### 2.8.4 DH密钥协商算法实践

1）dhparam和genpkey子命令

生成一个 1024 比特的参数文件：

```
$ gmssl dhparam -out dhparam.pem -2 1024
```

查看参数文件内容：

```
$ gmssl dhparam -in dhparam.pem -noout -C
```

和RSA密钥对生成方式不一样，DH密钥对生成方式分为两步，首先生成参数文件，然后根据参数文件生成密钥对，同一个参数文件可以生成无数多且不重复的密钥对。

基于参数文件生成密钥对：

```
$ gmssl genpkey -paramfile dhparam.pem -out dhkey.pem
```

查看密钥文件内容：

```
$ gmssl pkey -in dhkey.pem -text -noout
DH Private-Key: (1024 bit)
    private-key:
        41:7d:cf:f9:bf:f0:64:db:e8:c9:ca:ec:ef:8c:aa:
        92:8e:e9:16:61:4a:8a:bc:fc:a3:99:bc:41:ee:31:
        a5:e0:6d:d3:7f:a0:c2:8d:e1:75:fb:79:1a:9b:e8:
        3e:3f:ef:d1:ad:0b:e7:6d:c9:2a:19:41:a3:5d:bf:
        25:86:73:36:e0:63:c0:ec:8e:31:8b:53:4f:11:c0:
        15:d1:62:99:d3:d0:59:ec:3a:82:c9:2a:ad:57:44:
        be:b2:38:77:62:55:f1:cc:a3:71:bf:66:67:69:b5:
        85:a6:46:4a:ed:f2:0d:8f:0d:7e:1c:83:fd:c2:f8:
        be:bd:47:6e:4f:c0:e3:5f
    public-key:
        33:ee:e3:47:40:6d:fa:60:e0:cb:76:76:68:4c:6b:
        80:ac:ca:62:fa:f2:c0:ba:ba:1a:9e:18:e0:e7:45:
        32:1c:ce:1c:fd:37:85:72:24:87:f3:ab:b6:4a:71:
        1c:8b:3c:27:29:26:08:81:ff:60:74:28:50:0a:57:
        44:0b:f7:7c:44:81:25:51:2d:50:2b:92:76:6a:c2:
        b5:4b:3d:6f:31:8b:87:70:5c:96:7a:ff:43:19:63:
        5c:fb:5f:64:c9:b1:a8:7b:b5:fe:c4:1c:ae:bb:8c:
        36:89:54:c7:e0:17:a4:17:51:b8:fb:6c:2e:52:83:
        02:51:c6:5c:e2:6b:50:5d
    prime:
        00:ce:20:92:f3:db:88:65:0a:41:48:bc:55:85:bc:
        0a:e1:56:37:6e:fb:ca:5d:75:4d:d0:58:5e:ec:d7:
        e8:60:f3:dd:c3:7d:2b:ea:f9:49:9d:f5:3b:37:d5:
        d0:06:0b:56:a9:39:01:ce:d4:42:85:99:db:e3:28:
        5d:73:fe:0d:0c:cc:49:87:e7:e5:c8:b5:06:00:c9:
        a4:4a:12:8e:73:98:a4:29:4e:88:5e:85:f2:49:eb:
        9b:c0:b0:19:c9:b2:45:b6:3c:36:88:bb:96:2a:72:
        65:08:97:cb:da:2e:6a:49:dd:93:15:b6:e7:92:a2:
        24:a4:52:e8:88:b3:88:5b:53
    generator: 2 (0x2)
```

prime是一个大质数，generator是生成元，同时包含private-key（私钥）和public-key（公钥）。

2）完整的协商例子

1. 通信双方的任何一方生成DH的参数文件，可以对外公开。
2. 发送方A基于参数文件生成一个密钥对。
3. 发送方B基于参数文件生成一个密钥对。
4. 发送方A拆出公钥文件akey_pub.pem，发送给B，私钥自己保存。
5. 发送方B拆出公钥文件bkey_pub.pem，发送给A，私钥自己保存。
6. 发送方A收到B发送过来的公钥，将协商出的密钥保存到data_a.txt文件。
7. 发送方B收到A发送过来的公钥，将协商出的密钥保存到data_b.txt文件。
   
最终会发现data_a.txt文件和data_b.txt文件完全相同，表示协商出同一个会话密钥。

不管客户端还是服务器端每次生成的密钥对是不一样的，也就是能提供前向安全性，这种算法也称为临时DH算法。

## 2.9 椭圆曲线密码学

ECC是新一代的公开密钥算法，主要的优点就是安全性，极短的密钥能够提供很大的安全性。比如224比特的ECC密钥和2048比特的RSA密钥可以达到同样的安全水平，由于ECC密钥具有很短的长度，运算速度非常快。

在具体应用的时候，ECC可以结合其他公开密钥算法形成更快、更安全的公开密钥算法，比如结合DH密钥协商算法组成ECDH密钥协商算法，结合数字签名DSA算法组成ECDSA数字签名算法。

### 2.9.1 ECC算法的基本模型

ECC是比离散对数类算法（比如RSA和DH算法）更复杂的算法，非常难于理解，本身也是很复杂的一个结构体，在理解起来的时候有一定的难度。

ECC密码学的关键点就在于就算知道具体方程式、基点（G）、最终点（F），也无法知晓一共打点了多少次（n），这就是椭圆曲线的关键，很难破解打点过程。

那么ECC到底包含了什么？ECC由方程式、基点（G）、质数（P）组成，当然还有a、b这样的方程式参数。理论上方程式和各种参数组合可以是任意的，但是在密码学中，为了安全，系统预先定义了一系列的曲线，称为命名曲线（name curve），比如secp256k1就是一个命名曲线。

### 2.9.2 使用OpenSSL了解命名曲线

对于开发者说，可以不了解ECC原理，知道命名曲线和参数文件即可。选择一条命名曲线，基于曲线获取参数文件，同样的命名曲线，参数文件是相同的。参数文件结合其他公开密钥算法（比如DH和DSA），能够生成更安全的算法。

查看系统有多少椭圆曲线：

```
$ gmssl ecparam -list_curves
  secp112r1 : SECG/WTLS curve over a 112 bit prime field
  secp112r2 : SECG curve over a 112 bit prime field
  secp128r1 : SECG curve over a 128 bit prime field
  secp128r2 : SECG curve over a 128 bit prime field
  secp160k1 : SECG curve over a 160 bit prime field
  secp160r1 : SECG curve over a 160 bit prime field
  secp160r2 : SECG/WTLS curve over a 160 bit prime field
  secp192k1 : SECG curve over a 192 bit prime field
  secp224k1 : SECG curve over a 224 bit prime field
  secp224r1 : NIST/SECG curve over a 224 bit prime field
  secp256k1 : SECG curve over a 256 bit prime field
  secp384r1 : NIST/SECG curve over a 384 bit prime field
  secp521r1 : NIST/SECG curve over a 521 bit prime field
  prime192v1: NIST/X9.62/SECG curve over a 192 bit prime field
  prime192v2: X9.62 curve over a 192 bit prime field
  prime192v3: X9.62 curve over a 192 bit prime field
  prime239v1: X9.62 curve over a 239 bit prime field
  prime239v2: X9.62 curve over a 239 bit prime field
  prime239v3: X9.62 curve over a 239 bit prime field
  prime256v1: X9.62/SECG curve over a 256 bit prime field
  sect113r1 : SECG curve over a 113 bit binary field
  sect113r2 : SECG curve over a 113 bit binary field
  sect131r1 : SECG/WTLS curve over a 131 bit binary field
  sect131r2 : SECG curve over a 131 bit binary field
  sect163k1 : NIST/SECG/WTLS curve over a 163 bit binary field
  sect163r1 : SECG curve over a 163 bit binary field
  sect163r2 : NIST/SECG curve over a 163 bit binary field
  sect193r1 : SECG curve over a 193 bit binary field
  sect193r2 : SECG curve over a 193 bit binary field
  sect233k1 : NIST/SECG/WTLS curve over a 233 bit binary field
  sect233r1 : NIST/SECG/WTLS curve over a 233 bit binary field
  sect239k1 : SECG curve over a 239 bit binary field
  sect283k1 : NIST/SECG curve over a 283 bit binary field
  sect283r1 : NIST/SECG curve over a 283 bit binary field
  sect409k1 : NIST/SECG curve over a 409 bit binary field
  sect409r1 : NIST/SECG curve over a 409 bit binary field
  sect571k1 : NIST/SECG curve over a 571 bit binary field
  sect571r1 : NIST/SECG curve over a 571 bit binary field
  c2pnb163v1: X9.62 curve over a 163 bit binary field
  c2pnb163v2: X9.62 curve over a 163 bit binary field
  c2pnb163v3: X9.62 curve over a 163 bit binary field
  c2pnb176v1: X9.62 curve over a 176 bit binary field
  c2tnb191v1: X9.62 curve over a 191 bit binary field
  c2tnb191v2: X9.62 curve over a 191 bit binary field
  c2tnb191v3: X9.62 curve over a 191 bit binary field
  c2pnb208w1: X9.62 curve over a 208 bit binary field
  c2tnb239v1: X9.62 curve over a 239 bit binary field
  c2tnb239v2: X9.62 curve over a 239 bit binary field
  c2tnb239v3: X9.62 curve over a 239 bit binary field
  c2pnb272w1: X9.62 curve over a 272 bit binary field
  c2pnb304w1: X9.62 curve over a 304 bit binary field
  c2tnb359v1: X9.62 curve over a 359 bit binary field
  c2pnb368w1: X9.62 curve over a 368 bit binary field
  c2tnb431r1: X9.62 curve over a 431 bit binary field
  wap-wsg-idm-ecid-wtls1: WTLS curve over a 113 bit binary field
  wap-wsg-idm-ecid-wtls3: NIST/SECG/WTLS curve over a 163 bit binary field
  wap-wsg-idm-ecid-wtls4: SECG curve over a 113 bit binary field
  wap-wsg-idm-ecid-wtls5: X9.62 curve over a 163 bit binary field
  wap-wsg-idm-ecid-wtls6: SECG/WTLS curve over a 112 bit prime field
  wap-wsg-idm-ecid-wtls7: SECG/WTLS curve over a 160 bit prime field
  wap-wsg-idm-ecid-wtls8: WTLS curve over a 112 bit prime field
  wap-wsg-idm-ecid-wtls9: WTLS curve over a 160 bit prime field
  wap-wsg-idm-ecid-wtls10: NIST/SECG/WTLS curve over a 233 bit binary field
  wap-wsg-idm-ecid-wtls11: NIST/SECG/WTLS curve over a 233 bit binary field
  wap-wsg-idm-ecid-wtls12: WTLS curve over a 224 bit prime field
  Oakley-EC2N-3: 
	IPSec/IKE/Oakley curve #3 over a 155 bit binary field.
	Not suitable for ECDSA.
	Questionable extension field!
  Oakley-EC2N-4: 
	IPSec/IKE/Oakley curve #4 over a 185 bit binary field.
	Not suitable for ECDSA.
	Questionable extension field!
  brainpoolP160r1: RFC 5639 curve over a 160 bit prime field
  brainpoolP160t1: RFC 5639 curve over a 160 bit prime field
  brainpoolP192r1: RFC 5639 curve over a 192 bit prime field
  brainpoolP192t1: RFC 5639 curve over a 192 bit prime field
  brainpoolP224r1: RFC 5639 curve over a 224 bit prime field
  brainpoolP224t1: RFC 5639 curve over a 224 bit prime field
  brainpoolP256r1: RFC 5639 curve over a 256 bit prime field
  brainpoolP256t1: RFC 5639 curve over a 256 bit prime field
  brainpoolP320r1: RFC 5639 curve over a 320 bit prime field
  brainpoolP320t1: RFC 5639 curve over a 320 bit prime field
  brainpoolP384r1: RFC 5639 curve over a 384 bit prime field
  brainpoolP384t1: RFC 5639 curve over a 384 bit prime field
  brainpoolP512r1: RFC 5639 curve over a 512 bit prime field
  brainpoolP512t1: RFC 5639 curve over a 512 bit prime field
  sm2p256v1 : SM2 curve over a 256 bit prime field
  wapip192v1: WAPI curve over a 192 bit prime field
  sm9bn256v1: SM9 BN curve over a 256 bit prime field
```

生成一个参数文件，-name指定命名曲线：

```
$ gmssl ecparam -name secp256k1 -out secp256k1.pem
```

### 2.9.3 ECDH协商算法

ECC可以结合公开密钥算法，比如DH算法结合ECC可以组成为ECDH算法，接下来看看如何结合。

1. A和B各自生成一个ECDH私钥对文件。
2. A和B分解出公钥，发送给对方。
3. A和B计算出同样的会话密钥。

### 2.9.4 命名曲线

ECC本质上就是一个数学公式，任何人基于公式都可以设计出椭圆曲线，在实现的时候一定要注意ECC离散对数问题（Elliptic-Curve Discrete-Logarithm Problem，简称ECDLP）。

对于大部分开发者来说，如果要使用ECC椭圆曲线，要做的就是选择一条安全且性能高的命名曲线。

在密码学世界中，一些组织定义了命名曲线的一些设计标准，不同的设计标准有不同的目标，比如有的以安全性为首要目标，有的以效率为首要目标。

不同标准的命名曲线命名可能不一样，但是大部分含义是相同的，建议使用NIST标准建立的命名曲线。

在选择命名曲线的时候，有几点需要注意：

* 尽量选择性能更高、安全系数更高的命名曲线，比如P-384命名曲线安全性高于P-256（不考虑命名曲线的优化）。
* 命名曲线的兼容性，这一点非常重要，ECC是相对较新的密码学算法，再加上命名曲线的标准也比较多，大部分操作系统、密码学底层库（比如OpenSSL库）、软件支持的命名曲线可能是不一样的，在Web应用中尽量选择兼容性比较好的命名曲线。
  
下面简单了解下不同平台支持ECC的时间点。

* 操作系统：Windows Vista、OS X 10.6、Android 4.0、Red Hat 6.5以后的操作系统支持。
* 浏览器：Firefox 2.0、Chrome 1.0、Safari 4、IE 7以后的版本支持。
* 服务器：Nginx 1.1.0、Apache 2.2.26以后的版本支持。

## 2.10 数字签名

### 2.10.1 数字签名的用途

对称加密算法、公开密钥算法都不能防止抵赖。

先解释为什么消息可能被篡改，比如A、B、C三个人共享一个对称加密算法密钥，现在A和B互相通信，A和B一直认为是双方在发送消息。由于C也有同样的密钥，它可以拦截A发往B的消息，然后篡改消息并用同样的密钥加密后发送给B, B能够正确解密，但是该消息其实已经被篡改。

接下来解释为什么不能防止抵赖，还是用同样的例子说明，A、B、C三个人共享一个对称加密算法密钥，A向B发送了一条消息，但是A可以抵赖说这条消息并不是他发送的，理由就是C也有同样的密钥，这条加密消息可能是C发送给B的，B无法向第三方证明是A给他发送了消息。

在公开密钥算法中，如果算法用于加密解密，也同样不能防止抵赖，以RSA加密算法举例，由于RSA的公钥是完全公开的，RSA私钥拥有者虽然能够解密，但是并不能确认是哪个客户端发送的消息，同理任何人都可以抵赖。

抵赖出现的根本原因就在于通信双方无法确认对方的身份，也就是不能进行身份验证，那么在密码学中有没有对应的解决方案呢？可以使用数字签名技术防抵赖。

在现实世界中，有哪些行为或者约定可以防止人抵赖呢？最明显的就是合同，合同一般需要人签字或者按指纹，考虑签字可以模仿伪造，这里重点用指纹签署的合同来解释。合同一旦由指纹签署了，就可以被复印多份。有了合同，合同签署人就无法否认合同的合法性，原因就在于法律规定，指纹具备唯一性，每个人的指纹是不同的，或者说指纹就代表了一个人。

回到密码学中，如果一个消息也含有特殊的指纹，那么它是否就不能抵赖呢？仔细回忆RSA密钥对，私钥只有密钥对的生成者持有，如果不考虑密钥泄露的问题，私钥拥有者使用密钥（注意不是加密操作）签署一条消息，然后发送给任意的接收方，接收方只要拥有私钥对应的公钥，就能成功反解签署消息，由于只有私钥持有者才能“签署”消息，不能抵赖说这条签署消息不是他发送的，这就是数字签名技术的全部。

简单地说，数字签名技术有以下几个特点。

* 防篡改：数据不会被修改，MAC算法也有这个特点。
* 防抵赖：消息签署者不能抵赖。
* 防伪造：发送的消息不能够伪造，MAC算法也有这个特点。

数字签名技术能够进行身份验证，在2.5节中介绍过MAC算法，它能保证传递的消息是经过验证的，但不能对消息发送者的身份进行验证，原因就在于消息发送方和接收方拥有同样的密钥，所以双方可以抵赖，否认消息是他发送的，读者在理解的时候一定要区分消息验证和身份验证。

### 2.10.2 数字签名的流程

不管是RSA数字签名算法还是后续讲解的DSA数字签名算法，数字签名处理流程是差不多的，主要分为签名生成和签名验证，接下来分别描述这两个过程。

* 发送者对消息计算摘要值。
* 发送者用私钥对摘要值进行签名得到签名值。
* 发送者将原始消息和签名值一同发给接收者。

数字签名技术的本质不是为了加密，所以和签名值一同传递的消息是不用加密的，当然也可以对消息加密后再计算签名值。

* 接收者接收到消息后，拆分出消息和消息签名值A。
* 接收者使用公钥对消息进行运算得到摘要值B。
* 接收者对摘要值B和签名值A进行比较，如果相同表示签名验证成功，否则就是验证失败。

为什么不直接对消息进行签名，而是对消息的摘要值进行签名？签名值除了比较之外并没有其他用途，那么基于消息生成签名和基于消息摘要值生成签名并无区别，考虑到公开密钥算法运行是相对缓慢的，数字签名算法建议对消息摘要值进行签名，因为摘要值的长度是固定的，运算的时候速度会比较快。

### 2.10.3 RSA数字签名算法

RSA算法的用途非常广泛，可以进行数字签名。和RSA加密算法相似，不同的是，RSA加密算法是公钥加密，私钥解密；RSA签名算法是私钥签名，公钥验证签名。

和RSA加密填充一样，RSA签名算法也有填充机制，分别是RSASSA-PKCS1-v1_5和RSASSA-PSS。对于同样的输入值和密钥对，使用RSASSA-PKCS1-v1_5标准生成的签名值是固定不变的，而对于RSASSA-PSS标准来说，生成的签名值每次都是变化的，所以安全性更好一点。

### 2.10.4 RSA数字签名实践

生成一个密钥对，密钥长度1024比特：

```
$ gmssl genrsa -out rsaprivatekey.pem 1024
```

从密钥对中分离出公钥：

```
$ gmssl rsa -in rsaprivatekey.pem -pubout -out rsapublickey.pem
```

对txt文件使用sha256 hash算法和签名算法生成签名文件：

```
$ gmssl dgst -sha256 -sign rsaprivatekey.pem -out signature.txt temp.txt
```

用相同的摘要方法和签名算法校验签名文件，需要对比签名文件和原文件：

```
$ gmssl dgst -sha256 -verify rsapublickey.pem -signature signature.txt temp.txt
```

和RSA公钥加密不一样，RSA实现数字签名技术涉及摘要计算，本例中使用sha256 Hash算法。

OpenSSL命令行进行签名的时候默认使用的是RSAES-PKCS1-V1_5填充标准，也可以指定RSASSA-PSS标准。

## 2.11 DSA数字签名算法

DSA签名算法（Digital Signature Algorithm），是美国国家标准技术研究所（NIST）在1991年提出的签名算法，只能进行签名，不能进行加密解密。

DSA数字签名算法生成签名、验证签名的机制和RSA数字签名算法是一样的。

### 2.11.1 内部结构

理解DSA算法主要了解其参数文件，通过参数文件生成密钥对。

```
typedef struct dsa_st {
    BIGNUM *p;
    BIGNUM *q;
    BIGNUM *g;
    BIGNUM *pub_key;
    BIGNUM *priv_key;
} DSA;
```

p、q、g是公共参数，通过参数会生成密钥对，DSA的公共参数和DH的公共参数很像，通过公共参数能够生成无数个密钥对，这是一个很重要的特性。

p是一个很大的质数，这个值的长度很关键，可以是512到1024比特之间的数（必须是64比特的倍数），这个数的长度建议大于等于1024比特，p-1必须是q的倍数，q的长度必须是160比特。而g是一个数学表达式的结果，数值来自p和q。

DSA的密钥对生成就取决于这三个公共参数，计算签名和验证签名也要依赖参数文件。

1）生成DSA密钥对

* 选取一个随机数作为私钥x,0 < x < q。
* 基于私钥生成公钥，g^x mod p。

2）签名生成

* 生成一个随机数k,1 < k < q。
* 计算r = ( g^k mod p ) mod q。
* 计算s = ( k^(-1) (H(m) + xr)) mod q, H是特定的摘要算法。
* 签名值就是(r, s)，随同原始消息m一起发送。

3）签名验证

* 假如r和s大于q或者小于0，则验证直接失败。
* 计算w = s^(-1) mod q。
* 计算u1 = H(m).w mod q。
* 计算u2 = r.w mod q。
* 计算v = ( g^u1 * y^u2 mod p ) mod q。
* 如果v等于r，则签名验证成功，否则失败。

### 2.11.2 DSA算法实践

### 2.11.3 ECDSA算法

# 3. 宏观理解TLS

解决HTTP三大问题的通用解决方案就是TLS协议。本章宏观上讲解TLS协议是如何工作的，通过本章读者能够明白TLS协议实现的大概原理和关键步骤，体会解决HTTP问题的复杂性。

## 3.1 TLS/SSL协议综述

### 3.1.1 TLS/SSL协议的历史

读者可能听说过TLS（Transport Layer Security）协议，也可能听说过SSL（Secure SocketsLayer）协议，在理解的时候可以认为两者是一样的，TLS协议是SSL协议的升级版，本书使用TLS/SSL协议代表TLS协议或者SSL协议。

网景公司为了解决HTTP的安全问题，1994年创建了SSL协议，作为浏览器的一个扩展，主要应用于HTTP。

网景意识到互联网还有很多其他的应用协议，比如SMTP、FTP，这些协议在实际应用的时候也面临同样的安全问题，于是开始思考是否有统一的方案解决互联网通信安全问题。基于此考虑，SSL协议逐渐成为一个独立的协议，该协议能够保证网络通信的认证和安全问题，SSL协议有三个版本，分别是SSL v1、SSL v2、SSL v3。

1996年，IETF（Internet Engineering Task Force）组织在SSL v3的基础上进一步标准化了该协议，微软为这个新协议取名TLS v1.0，目前比较稳定的版本是TLS v1.2，当然TLS v1.3也发布了。

### 3.1.2 正确认知TLS/SSL协议

说到安全协议，先要正确理解密码学算法，每个密码学算法只能解决特定的问题，实际应用算法的时候可能会有很多陷阱，理解密码学算法的应用标准很重要。

那么什么是协议呢？协议是解决方案、标准，能够解决很多普适性的问题，TLS/SSL协议是一系列算法的组合，相比密码学算法来说，TLS/SSL协议的复杂性就更大了，主要体现在以下方面。

* 协议设计的复杂性：一个完整的解决方案考虑的问题非常多，需要考虑扩展性、适用性、性能等方面，一旦方案设计不充分，攻击者不用攻击特定的密码学算法，而会基于协议进行攻击。
* 协议实现的严谨性：即使协议设计是完美的，在实现协议的时候，也可能犯错误；如果不充分理解密码学算法应用标准，最终实现的协议就会存在安全漏洞。

对于大部分开发者来说，如何正确对待TLS/SSL协议呢？在开发一个应用的时候，安全如果是核心目标之一，就需要考虑是否有标准的解决方案。在互联网开发中，TLS/SSL协议是最常见的安全解决方案，需要注意的是，TLS/SSL协议并不是唯一的解决方案，选择的时候必须清晰地认识到TLS/SSL的优缺点。

TLS/SSL协议是一个独立的协议，完全模块化，读者即使完全不理解原理，也可以在应用中无缝接入。当然实际应用的时候，为了绝对安全，还是必须了解TLS/SSL协议的基本知识，现在有很多的最佳实践指导开发者配置TLS/SSL协议，比如使用Nginx部署TLS/SSL协议的时候，官方就有很详细的指导文档。

学习的层次有三种，以TLS/SSL协议举例来说，第一个层次就是阅读几篇文章搭建出一个安全的Web网站，虽然成功了，但是完全不明白其背后的原理，这个过程是非常痛苦的。第二个层次就是了解密码学算法的原理和作用，也了解TLS/SSL的原理，虽然不知道具体是如何实现的，但能搭建出一个更安全的Web网站。第三个层次就是知道TLS/SSL协议设计思路，研究过OpenSSL源码，这属于专家层次。

TLS/SSL协议在实现上更多考虑安全性，效率相对是弱势，因为密码学算法尤其是公开密钥算法的运算极其缓慢，密码学的重点是安全，必须明白TLS/SSL协议出现的初衷。对于追求高性能的应用来说，如果觉得TLS/SSL协议性能不能满足需求，可以自行设计安全解决方案，但是对于大部分开发者来说，尽量选择TLS/SSL协议，因为其更安全。

### 3.1.3 TLS/SSL协议的目标

读者在理解TLS/SSL协议的时候，一般都急着想了解TLS/SSL协议到底是怎么工作的，内部细节是怎么样的，其实应该先了解TLS/SSL协议在网络协议中的定位，了解TLS/SSL协议的核心目标。

图3-1所示为TLS/SSL协议在网络层协议中的定位。

通过图3-1可以看出，TLS/SSL协议位于应用层协议和TCP之间，构建在TCP之上，由TCP协议保证数据传输的可靠性，任何数据到达TCP之前，都经过TLS/SSL协议处理。

通过图3-1也可以看出，对于应用层协议来说，它无须过多改变，引入TLS/SSL协议即可保证数据机密性和完整性。任何应用层协议（HTTP、SMTP、FTP、其他自定义应用层协议）都可以结合TLS/SSL协议。

TLS/SSL协议一般构建在TCP之上，也可以构建在UDP之上，称为DTLS（Datagram TransportLayer Security）协议，DTLS协议在Web中使用得比较少。

TLS/SSL协议有四个目标，仔细体会这些目标很重要。

* 数据是机密的：通信两端传输的数据应该是安全的，不可伪造和篡改的。
* 互操作性：TLS/SSL协议是标准的，任何开发者基于TLS/SSL RFC设计规范都可以实现该协议，开发者也很容易在应用中引入TLS/SSL协议。
* 可扩展性：密码学算法是不断迭代的，随着时间的推移，会出现更安全的算法，为了保障持续的安全，TLS/SSL协议允许动态地引入新的算法。由于通信体之间环境是不一样的，协议允许双方协商出都支持的密码学算法，可以说TLS/SSL协议是非常灵活的，TLS/SSL协议也有很多的扩展支持扩展性。
* 效率：解决方案必须是高效的，TLS/SSL协议涉及了很多密码学算法的运算，增加了通信延时和机器负载，这也遭到了很多人的诟病，但TLS/SSL协议发展到现在，有一些新的技术和解决方案在逐步提升TLS/SSL协议的效率。

### 3.1.4 OpenSSL和TLS/SSL的关系

TLS/SSL协议是设计规范，设计规范和设计思路可以通过RFC文档查看，TLS/SSL协议的具体实现有很多，比如OpenSSL、LibreSSL、BoringSSL。虽然OpenSSL名声不太好，但截至目前，它还是最通用的TLS/SSL协议实现。

OpenSSL也实现了TLS/SSL协议，任何和密码学有关的内容都能在OpenSSL中找到。

通过两个维度了解OpenSSL，首先OpenSSL是一个底层密码库，封装了所有的密码学算法、证书管理、TLS/SSL协议实现。

对于开发者来说，要做的就是正确地理解并使用底层OpenSSL库，OpenSSL库包含两种类型的库。

* crypto库函数：具体的密码学算法使用库，比如MD5、RSA、DES算法的实现，开发者可以直接使用这些库，可以理解为底层次库。
* EVP接口：高层次库，基于crypto库函数做了进一步抽象，比如对称加密算法有很多种，为了方便调用，可以直接调用一个简单的EVP接口，通过接口参数就能操作各种对称加密算法。同时EVP接口也会基于CPU模式进行性能优化，比如可以通过AES-NI指令集加速AES运算。

理解OpenSSL的第二个维度就是OpenSSL命令行工具。如果读者不是为了编写代码，而只是为了理解密码学原理和TLS/SSL协议，熟练掌握OpenSSL命令行工具是很好的一种学习方式。

### 3.1.5 HTTPS和TLS/SSL的关系

构建在TCP之上的应用层协议（比如HTTP）都能结合TLS/SSL协议，TLS/SSL协议和应用层协议无关，它只是加密应用层协议（比如HTTP）并传递给下层的TCP。

HTTP和TLS/SSL协议组合在一起就是HTTPS, HTTPS等同于HTTP+TLS/SSL，就是说HTTPS拥有HTTP所有的特征，并且HTTP消息由TLS/SSL协议进行安全保护。

对于服务器端的应用程序来说（比如PHP），无须关心是HTTPS还是HTTP，它完全按照HTTP标准处理HTTP头部，负责输出内容，这也体现了TLS/SSL协议的优势，对开发者来说完全是透明的。

### 3.1.6 TLS/SSL协议的一些实现

表3-1 TLS/SSL协议的一些实现

### 3.2 TLS/SSL协议背后的算法

# 第5章 快速搭建一个HTTPS网站

## 5.1 HTTPS网站构建分析

在构建HTTPS网站之前，需要做一些简单的分析，主要如下。

1）证书、域名、密钥对

证书中包含的一个关键信息就是域名，域名的使用方式间接决定了证书的生成，根据需求可以选择不同的证书类型，比如泛域名证书、SAN证书，这些都和域名的分配方式有关，避免使用多级子域名是一个很好的策略。而从全站HTTPS的角度考虑，必须为每个域名（包括子域名）分配证书。

证书和密钥对是关联保存的，需要避免密钥对尤其是私钥泄露，一旦出现泄露，需要立刻替换证书。

2）迁移还是全新构建HTTPS网站

全新构建HTTPS网站没有历史包袱，可以按照全站HTTPS的策略去实施。

迁移HTTPS花费的时间比较多，处理混合内容的复杂度取决于原有网站应用架构的复杂度。

3）Web应用的类型

Web应用包含API接口和Web网站，API接口主要供手机APP使用，不管是接口迁移还是接口构建相对简单，使用301重定向策略或者强制升级即可。

而Web应用考虑的方面比较多，要替换网站所有能控制的HTTP URL地址，要处理各种情况的混合内容，还要保证暴露在互联网上的HTTP URL地址也能重定向到HTTPS URL地址。

4）混合内容处理

为了绝对的安全，务必实施全站HTTPS策略，而实施全站HTTPS策略，重要的就是处理混合内容以及处理暴露在互联网上的HTTP URL地址，有多种处理策略，包括301重定向、HSTS、CSP策略，需要综合使用，每种解决方案都有优缺点。

5）系统架构

不仅Web服务器，其他一些服务器，包括负载均衡设备、反向代理服务器、CDN，都有可能部署证书和密钥对，对于大中型企业来说，部署的策略取决于网络架构和应用架构，对于只有几台Web服务器的网站来说，直接在Web服务器上部署证书和密钥对即可。

## 5.2 获取证书和密钥对

获取证书有三种途径：

* 自签名证书，如果开发者只是想测试HTTPS，最快速的途径就是生成自签名证书，非常方便。
* Let's Encrypt证书，可以使用免费CA机构签发的证书。
* 使用收费CA机构签发的证书，如果对证书安全性、兼容性、功能有特殊需求，可以向CA机构申请证书。

### 5.2.1 自签名证书

自建证书是自己签发的（表示用户就是一个CA机构），浏览器一般不会集成私有CA机构的根证书，从中也可以看出，即使是具备一定规模的CA机构想将根证书集成到各个浏览器中也并不容易。

由于浏览器没有集成自签名证书的根证书，当浏览器发现一张自签名证书，并不会立刻中断TLS/SSL握手，会提示用户该证书可能是伪造的，存在中间人攻击可能性，用户可以选择信任该证书或者拒绝该证书，一旦拒绝该证书，则整个握手失败，如果用户信任该证书，则进行后续完整的TLS/SSL握手，和正常的握手并无两样，也就是说用户一旦信任自签名证书，后续的数据通信也是处于加密保护的。

自签名证书的用途还是很广泛的，对于一些企业内部系统，由于购买证书需要成本，可以生成自签名证书，企业内部系统的用户一般运行在同一个局域网下，由防火墙保护，风险相对可控，当浏览器提示用户自签名证书存在风险时，用户可以选择信任自签名证书，等同于访问了一个HTTPS网站。

1）生成私钥对和CSR

CSR（Certificate Signing Request）表示证书签名请求，其中包含了服务器的密钥对，CA机构接收到请求后会验证CSR请求的签名。

```
$ gmssl req -newkey rsa:1024-nodes -keyout ilego_key.pem -out ilego_csr.pem
Generating a 1024 bit RSA private key
..........................++++++
................++++++
writing new private key to 'ilego_key.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
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
Organization Name (eg, company) [Internet Widgits Pty Ltd]:China-LianTong
Organizational Unit Name (eg, section) []:China-Liantong
Common Name (e.g. server FQDN or YOUR name) []:ilego.club,www.ilego.club
Email Address []:alexzchen@china_liantong.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:alex123
An optional company name []:China LT
```

输入以上命令后，会有一些交互式提示，最重要选项则是域名信息，可以选择多个，在本例中为ilego.club和www.ilego.club域名生成一张证书。

openssl req命令最终生成两个文件，ilego_key.pem表示密钥对文件，ilego_csr.pem表示CSR文件。

2）生成自签名证书

接下来通过CSR生成证书，对于自签名证书，读者可以认为自己就是一个CA机构，生成证书命令很简单，输入如下命令即可：

```
$ gmssl x509 -signkey ilego_key.pem -in ilego_csr.pem -req -days 365 -out ilego_cert.pem
Signature ok
subject=C = CN, ST = Hubei, L = Wuhan, O = China-LianTong, OU = China-Liantong, CN = "ilego.club,www.ilego.club", emailAddress = alexzchen@china_liantong.com
Getting Private key
Enter pass phrase for ilego_key.pem:
```

-signkey表示密钥对文件，-in表示CSR文件，-days表示证书有效期，-out表示最终的证书文件。

OpenSSL命令非常灵活，生成证书的命令有很多种形式，比如可以通过密钥对直接生成证书文件，开发者要灵活使用命令行，仔细理解其背后的含义。

最终将密钥对文件和证书保存在一个安全的位置。

### 5.2.2 向CA机构申请证书

证书申请的途径可以分为两种：

* 向专门的CA机构申请证书。
* 向代理机构申请证书，比如国内很多云厂商集成了证书签发功能。
 
证书申请的具体方式也分为两种：

* 向CA机构发送CSR文件。
* 由CA机构统一生成证书和密钥对，这种方式很方便，但是存在安全问题，等同于泄露了自己的私钥。

在申请证书的过程中，CA机构会进行审核，以便核实申请者是否拥有域名的所有权，方式也分为两种：

* 申请者增加域名的TXT记录，CA机构校验域名TXT记录，一旦确认无误代表申请者身份审核成功。
* 在域名对应的服务器中放置一个HTTP校验文件，如果CA机构能访问校验文件，代表申请者身份审核成功。

## 5.3 部署证书和密钥对

获取到证书文件后，就可以开始部署HTTPS网站了，主流的Web服务器都集成了TLS/SSL模块。

### 5.3.1 Nginx配置

编辑配置文件 /etc/nginx/nginx.conf：

```
        ssl_certificate     /home/alex/ai/certs/3286328_www.ilego.club.pem;
        ssl_certificate_key /home/alex/ai/certs/3286328_www.ilego.club.key;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;


```

重新启动Nginx：

```
sudo service nginx restart
```

## 5.4 测试HTTPS

1）Curl命令行

Curl命令行是一个非常通用的HTTP（HTTPS）客户端，通过简单的Curl命令就能对HTTPS网站进行测试。使用下列的命令测试某个网站是否支持HTTPS, --verbose参数能够了解详细的信息，包括TLS/SSL握手的详细信息：

```
curl "https://ilego.club" --verbose
```

2）Chrome开发者工具

对于开发者来说，另外一种测试HTTPS网站的工具就是Chrome开发者工具，该工具可以显示三部分信息：

* HTTPS网站的证书信息。
* 协商出的密码套件、TLS/SSL协议版本等信息。
* 页面是否实施全站HTTPS策略。
 
使用Chrome浏览器打开一个HTTPS页面，按F8键打开Chrome开发者工具，选择【Security】菜单，可以了解详细的信息.
