# 写给开发人员的实用密码学 - MAC

这里的MAC，并不是计算机中的MAC地址，而是消息验证码（Message Authentication Code，MAC)。在[写给开发人员的实用密码学 - Hash算法](https://mp.weixin.qq.com/s/N5drDkTi5IeTNkE3V-Jyiw)中讲到的Hash算法能够进行完整性校验，但却不能避免消息被篡改，而MAC正是为了避免消息被篡改而设计。

#### 为什么需要MAC

在[写给开发人员的实用密码学 - Hash算法](https://mp.weixin.qq.com/s/N5drDkTi5IeTNkE3V-Jyiw)这篇文章中，谈到密码学哈希算法用途之一就是保证文档/消息完整性。对于文件下载来说，通过计算下载下来的文件的Hash值，和网站提供的Hash值进行对比，就能确定下载下来的文件是否和网站上的原始文件一致。这对于一些固定的使用场景有效（网站提供下载的文件比较固定），如果是任意消息（文档）的发送，怎么提供原始消息（文档）的Hash值呢？一个比较容易想到的方法是将Hash值附在消息的后面，一起发送给接收方，如下图所示：

![Hash值随消息一起发送](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202012/images/practical_cryptography_for_developers_mac_01.png)

接收方校验Hash值是否和接收到的Hash值相同，看起来没毛病。但如果考虑到如下的场景呢？

![中间人攻击](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202012/images/practical_cryptography_for_developers_mac_02.png)

攻击者对消息进行拦截，修改消息，然后计算该消息的摘要值（摘要算法是公开的），附在消息后面发送给接收方。接收方收到消息后，对消息计算摘要值，然后与接收到的摘要值进行比较，并没有发现有异常，而实际上接收到的消息已经被篡改。

通过对消息加密是一种解决方法。但在很多情况下，传递的消息没有必要加密，只要确保消息是完整且没有被篡改即可，原因可能如下：

* 接口的数据并不重要，对隐私性要求不高。
* 加密和解密过程很消耗性能。

另一种解决方法是MAC算法。MAC是由给定密钥和给定消息计算得出的验证码：

```
auth_code = MAC(key, msg)
```

通信双方维护同一个密钥，只有拥有密钥的通信双方才能生成和验证消息验证码。通常，它的行为类似于哈希函数：

* 消息或密钥中的微小变化导致MAC值完全不同。
* 更改密钥或消息并获得相同的MAC值实际上是不可行的。
* MAC验证码像哈希一样是不可逆的：无法从MAC代码中恢复原始消息或密钥。

MAC算法也称为“键控哈希函数”，因为它们的行为类似于带有密钥的哈希函数。

MAC值一般和原始消息一起传输，原始消息可以选择加密，也可以选择不加密，通信双方会以相同的方式生成MAC值，然后进行比较，如下图所示：

![MAC处理流程](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202012/images/practical_cryptography_for_developers_mac_03.png)

#### MAC算法的种类

现代密码学中存在许多MAC算法。最受欢迎的是基于哈希算法，例如HMAC（基于哈希的MAC，例如HMAC-SHA256）和KMAC（基于Keccak的MAC）。还有的则基于对称加密，例如CMAC（基于加密的MAC），GMAC（Galois MAC）和Poly1305（Bernstein一次性身份验证器）。其他MAC算法包括UMAC（基于通用哈希），VMAC（基于高性能分组加密的MAC）和SipHash（简单、快速、安全的MAC）。

作为开发人员，我们并不需要了解这么多MAC算法，重点了解HMAC（Hash-based Message Authentication Code）即可，它在SSL/TLS通信中得到广泛使用。

HMAC算法使用Hash算法作为加密基元，HMAC结合Hash算法有多种变种，比如HMAC-SHA-1、HMAC-SHA256、HMAC-SHA512，国密标准中则使用SM3 Hash算法。大家不要误以为HMAC算法就是Hash算法加上一个密钥，HMAC算法只是基于Hash算法的，内部的实现还是相当复杂的，我们通常并不需要了解，现有的加密/解密库通常已经实现了HMAC算法。

#### MAC算法实例

借助OpenSSL命令行工具，计算HMAC非常容易：

```
$ echo -n abc | openssl dgst -sha256 -hmac Passw0rd
(stdin)= c12a3b777eaebdc2f98e79418f605f9b0b23064161e83aa19e3cf37c005181f3
```

国密标准中使用SM3作为加密基元，也可以通过命令行计算：

```
$ echo -n abc | gmssl dgst -sm3 -hmac Passw0rd
(stdin)= db1ab0dda0aafbdcd53cbda95b7ecdee4a50586f92696616ab052aceea106212
```

#### MAC算法的用途

MAC算法的主要用途：

1. 证明消息没有被篡改，这和Hash算法类似。
2. 消息是正确的发送者发送的，也就是说消息是经过验证的。

此外MAC算法还可以使用在如下场景。

* 基于HMAC的密钥派生(HMAC-based key derivation，HKDF)

密钥派生功能（KDF）是将可变长度密码转换为固定长度密钥（比特序列）的功能：

```
function(password) -> key
```

一种非常简单的KDF函数，我们可以使用SHA256：仅对密码进行哈希处理。但不要这样做，因为它是不安全的，简单的哈希很容易受到字典攻击。

作为更复杂的KDF函数，我们可以通过使用一些称为“盐”的随机值计算HMAC（salt，msg，SHA256）来生成密码，该随机值与导出的密钥一起存储，以后用于再次从密码中导出相同的密钥。

* 基于MAC的伪随机发生器

我们可以从“盐”（常数或当前日期和时间或其他随机性）和种子（上一次生成的随机数，例如0）开始，计算next_seed：

```
next_seed = MAC(salt, seed)
```

在每次计算上述公式之后，下一个伪随机数将“随机更改”，我们可以使用它来生成特定范围内的下一个随机数。

#### 小结

在本文中我们介绍了消息验证码，消息验证码弥补了单一的Hash算法的不足，确保消息没有被篡改。接下来的部分介绍了MAC算法的种类和用途。在下一篇文章中，我们将介绍密码学中的随机数产生器，敬请关注！

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)