### 搭建个人国密CA(Certification Authority)

在SSL/TLS/HTTPS通信中，证书虽然不是TLS/SSL协议的一部分，却是HTTPS非常关键的一环，网站引入证书才能避免中间人攻击。证书涉及了很多密码学知识，理解证书后，再深入理解TLS/SSL协议，效果会更好。

在前面一篇文章《[搭建国密SSL开发测试环境](https://mp.weixin.qq.com/s/rUeOtFjB3QTPJW3RKccKag)》中，我们制作了一个自签名证书。通常情况下，用作调试简单的客户端/服务器端通信，足够了。然而，现实世界的证书要复杂的多，涉及到CA、证书链、证书的撤销等多种场景。如果我们要实现一个完善的SSL/TLS/HTTPS就需要把这些场景考虑进去，这时仅仅靠自签名证书是不够的。

我们也可以通过CA申请证书，对于个人开发者而言，成本比较高。比如从网上找到的国密证书价格，一年好几千元到几万的都有：

![国密证书参考价格](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202003/images/gmssl_setup_ca_01.png)

那有没有办法自己制作证书呢？答案是可以的。本文将探讨使用GmSSL制作国密证书，包括制作自签名根证书，并使用根证书签发证书。这样在开发中可以调试证书链的处理流程。

本文所使用的方法在Ubuntu 16.04上验证通过，在其它linux发行版本上，可能命令需要稍微做一些调整。关于国密SSL环境，请参考：

[搭建国密SSL开发测试环境](https://mp.weixin.qq.com/s/rUeOtFjB3QTPJW3RKccKag)

现实世界的CA是分级管理的，层级可以有多层。本文简单起见，只模拟到三级CA管理，具体来说：

```
Root CA -> Server CA -> Server
```

Root CA为一级CA，拥有根CA证书，Server CA为二级CA，其CA证书由Root CA签发，Server为最终的用户，其证书由Server CA签发。当然，Root CA也可以直接给Server签发证书，这里是为了演示CA层级需要而做的这样的假定。

### 制作根CA证书

#### 1. 生成SM2私钥：

```
gmssl ecparam -genkey -name sm2p256v1 -text -out rootkey.pem
```

根证书的私钥保存在rootkey.pem中，请妥善保存。

### 2. 创建证书请求：

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
Organization Name (eg, company) [Internet Widgits Pty Ltd]:mogoweb
Organizational Unit Name (eg, section) []:mogoweb
Common Name (e.g. server FQDN or YOUR name) []:mogoweb.com
Email Address []:mogoweb@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

上面的命令会要求提供一些信息，因为证书是自签名证书，只用于开发测试，所以填写什么内容无关紧要。

#### 3.创建一个 certext.ext 文本文件，内容为：

```
[ v3_ca ]
basicConstraints = CA:true
[ usr_cert ]
subjectAltName = DNS:localhost
```
注意： 相比上一篇文章，这个文件多增加了 "CA:true" 这个扩展，用于指定所签发的证书是否CA证书。

#### 4. 生成证书：

```
$ gmssl x509 -req -days 365 -in rootreq.pem -signkey rootkey.pem -extfile certext.ext -extensions v3_ca -out rootcert.pem
Signature ok
subject=C = CN, ST = Hubei, L = Wuhan, O = mogoweb, OU = mogoweb, CN = mogoweb.com, emailAddress = mogoweb@gmail.com
Getting Private key
```

注意：上面的命令行参数多了一个 -extensions v3_ca 参数，指定使用上面 certext.ext 文件 v3_ca 节的扩展项。

### 5. 将key和证书合到一个文件中

**注意：** 这个步骤并非必要，只是为了开发和调试方便。在实际部署时，私钥需要小心保存，绝不能和证书一起分发出去！

```
$ cat rootcert.pem rootkey.pem > root.pem
```

### 签发 Server CA 证书

和上面的步骤一样，这里直接把命令总结一下：

```
$ gmssl ecparam -genkey -name sm2p256v1 -text -out serverCAkey.pem
$ gmssl req -new -key serverCAkey.pem -out serverCAreq.pem
$ gmssl x509 -req -days 365 -in serverCAreq.pem -extfile certext.ext -extensions v3_ca -CA root.pem -CAkey root.pem -CAcreateserial -out serverCAcert.pem
$ cat serverCAcert.pem serverCAkey.pem rootcert.pem > serverCA.pem
```

需要注意第三个命令多了 -CA 和 -CAkey 参数，表示使用根证书签名。因为上面的步骤中，我们把key和证书合成了一个文件，所以这两个参数值给的同一个文件。

### 签发 Server 证书

和上面的步骤一样，这里直接把命令总结一下：

```
$ gmssl ecparam -genkey -name sm2p256v1 -text -out serverkey.pem
$ gmssl req -new -key serverkey.pem -out serverreq.pem
$ gmssl x509 -req -days 365 -in serverreq.pem -extfile certext.ext -extensions usr_cert -CA serverCA.pem -CAkey serverCA.pem -CAcreateserial -out servercert.pem
$ cat servercert.pem serverkey.pem serverCAcert.pem rootcert.pem > server.pem
```

**注意：** 第三个命令的 -extensions 给的参数值为 usr_cert ，对应的是 certext.ext 文件 usr_cert 节的扩展项，通常需要给定服务器的DNS名。

这样，生成的 server.pem 包含了根证书、Server CA证书和Server证书，包含了完整的证书链，可以投入测试使用了。

**再次声明：** 将key和证书打包在一起，只是为了开发和调试方便。在实际部署时，私钥需要小心保存，绝不能和证书一起分发出去！

### 问题

#### 1. 自签名证书出错

```
-Error with certificate at depth: 0
 err 18:self signed certificate
```

原因：CA证书的 Organization Name 值和所签发的证书的 Organization Name 值相同。

解决方法：重新制作server证书，注意其 Organization Name 值不要和Server CA证书相同。

#### 2. 无效的CA证书

```
-Error with certificate at depth: 1
 err 24:invalid CA certificate
```

原因：CA证书的 CA:true 扩展属性没加上
解决方法：重新制作root和Server CA证书，注意 CA:true 扩展属性

### 小结

本文介绍了搭建个人国密CA的方法，这样你也可以签发国密证书了。当然CA的一个重要前提是信任，别人如何信任你签发的证书？所以个人肯定取代不了CA中心的地位，本文仅仅探讨用于开发测试过程中的证书签发。如果真的需要在产品中部署国密证书，还是需要去指定的CA中心去申请。

如果大家有什么问题，欢迎交流。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)