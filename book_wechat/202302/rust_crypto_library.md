# 几个开源 RUST 安全算法库

这段时间把 RUST 语法过了一遍，写一些简单的 Demo 程序没啥问题了，但离掌握这门语言还差的远，需要项目实战才行。我决定从之前研究过的国密算法入手，使用 RUST 实现国密算法。

从头编写算法不太现实，上网搜了一下，已经有一些 开源 RUST 安全算法库，基于现有的开源代码实现更加可行。下面就介绍一下 RUST 加解密库，并选择一个项目作为基础，实现国密算法。

#### Sodiumoxide

项目地址：https://github.com/sodiumoxide/sodiumoxide 

目前项目标记为 [DEPRECATED] , 不再维护。Sodiumoxide 并不是纯 RUST 编写，而是 C 密码库 libsodium 的 Rust 封装，而 libsodium 又是 fork C 密码库NaCl 而来，因此 Sodiumoxide 的大多数文档来源于 NaCl 。

Sodiumoxide 实现的算法有：

* 对称加密算法 
  * 验证加密：aes256gcm, chacha20poly1305
  * 密钥生成：blake2b
  * 密钥交换：x25519blake2b
* 非对称加密算法 
  * curve25519xsalsa20poly1305
* HMAC：hmacsha256/512
* HASH：SipHasher13，SHA256/512

#### Ring

项目地址：https://github.com/briansmith/ring

Ring 采用混合语言编写，包括 Rust、C和汇编语言。Ring 采用轻量设计，小巧但功能全面，特别适用于小型设备、微控制器和IoT应用。

Ring实现的算法有：

* 对称加密算法 
  * 验证加密：aes128/256gcm, chacha20poly1305
  * 密钥生成：HKDF_SHA256/384/512，PBKDF2_HMAC_SHA1，PBKDF2_HMAC_SHA256/384/512
* 非对称加密算法 
  * 数字签名：ECDSA（P-256 Curve）+ SHA256/384，ED25519，RSA_PKCS1_SHA1/256/384/512，RSA_PSS_SHA256/384/512
* HMAC：HMACSHA256/384/512
* HASH：SHA1, SHA256/384/512

#### Dalek

项目地址：https://github.com/dalek-cryptography

Dalek 使用 RUST 语言编写，实际上是一个在 github 上的 RUST 库集合。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202302/images/rust_crypto_library_01.png)

Dalek 专注于椭圆曲线相关的算法 RUST 实现, 实现的椭圆曲线相关算法有：

* X25519
* Curve25519
* ED25519

#### rust-crypto

项目地址：https://github.com/RustCrypto

rust-crypto是一个大集合体，整合了大部分密码学需要使用的模块。

rust-crypto涉及到的算法有：

* 对称加密算法 
  * 验证加密：aesgcm, aessiv, ccm, chacha20poly1305, xsalsa20poly1305, deoxys, eax, mgm
  * 密钥生成：Argon2, bcrypt, PBKDF2, scrypt, SHA-crypt
  * 加解密： 
    * 模式：CFB，CTR, OFB
    * 算法：chacha20, rabbit, salsa20, hc256
* 非对称加密算法 
  * 数字签名：ECDSA，ED25519
  * 椭圆曲线：BP256/384, k256, P-256/384
* HMAC：HMACSHA256/384/512
* HASH：SHA1, SHA256/384/512, blake2, FSB, MD系列, gost94, groestl, k12, ripemd160/256/320, shabal, SM3

#### Rustls

项目地址：https://github.com/rustls/rustls

Rustls 是一个 Rust TLS 库，底层使用 ring ，目标是为 TLS 1.2 或更高版本提供安全功能和组件，号称效率比 OpenSSL 更高。

Rustls支持的算法和协议有：

* TLS1.2 和 TLS1.3。
* 客户端发起的 ECDSA、Ed25519 或 RSA 服务器端身份验证。
* 服务器发起的 ECDSA、Ed25519 或 RSA 服务器端身份验证。
* 使用 curve25519、nistp256 或 nistp384 曲线的 ECDHE 前向保密。
* 使用安全随机数的 AES128-GCM 和 AES256-GCM 批量加密。
* ChaCha20-Poly1305 批量加密 (RFC7905)。
* ALPN 支持。
* SNI 支持。
* 可调片段大小，使 TLS 消息匹配底层传输的大小。
* 可选地使用矢量 IO 来最小化系统调用。
* TLS1.2 会话恢复。
* 通过票证 (RFC5077) 恢复 TLS1.2。
* TLS1.3 通过票据或会话存储恢复。
* 客户端的 TLS1.3 0-RTT 数据。
* 服务器的 TLS1.3 0-RTT 数据。
* 由客户端进行客户端身份验证。
* 服务器端进行客户端身份验证。
* 扩展的主密钥支持 (RFC7627)。
* 导出器 (RFC5705)。
* 服务器端装订 OCSP。
* 服务器端 SCT 装订。
* 客户端的 SCT 验证。

#### rust-openssl

项目地址：https://github.com/sfackler/rust-openssl

这个项目为流行的 OpenSSL 加密库提供了一个安全的接口。 支持 OpenSSL 版本 1.0.1 到 3.x.x，此外还支持 LibreSSL 版本 2.5 到 3.4.1。

该项目被指出代码质量不好，并且缺少文档。

---

上述库中，Sodiumoxide、Rustls、rust-openssl 只是其他库的封装，要增加国密支持，只能修改所封装的库，不予考虑。 ring 则存在大量的汇编代码和 C 代码 ，不便于后期维护和开发，不太合适在上面进行开发。Dalek 实现的算法太少，很多常见加解密算法都没实现，放弃。rust-crypto 由纯 RUST 实现，加解密算法完善，基于 rust-crypto 实现国密算法比较合适。
