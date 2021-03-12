# 写给开发人员的实用密码学 - 国密哈希算法SM3实现要点

在《[写给开发人员的实用密码学 - Hash算法](https://mp.weixin.qq.com/s/N5drDkTi5IeTNkE3V-Jyiw)》和《[写给开发人员的实用密码学 - MAC](https://mp.weixin.qq.com/s/cp7U88wEvM5a6b2f-QQ4vA)》这两篇文章分别介绍了哈希算法和消息验证码，其中消息验证码使用到了哈希算法。国密标准中也定义了一种哈希算法 SM3，本文就来谈一谈在 libtomcrypt 中实现 SM3 算法的要点。

SM3 算法描述可以参考《GM/T 0004-2012 SM3密码杂凑算法》这个标准文档。实现代码可以参考 GmSSL 项目的 sm3.c 文件。

往 libtomcrypt 中添加一种新的哈希算法，实际上是要定义一个 struct ltc_hash_descriptor 结构：

```c
struct ltc_hash_descriptor {
    /** name of hash */
    const char *name;
    /** internal ID */
    unsigned char ID;
    /** Size of digest in octets */
    unsigned long hashsize;
    /** Input block size in octets */
    unsigned long blocksize;
    /** ASN.1 OID */
    unsigned long OID[16];
    /** Length of DER encoding */
    unsigned long OIDlen;

    /** Init a hash state
      @param hash   The hash to initialize
      @return CRYPT_OK if successful
    */
    int (*init)(hash_state *hash);
    /** Process a block of data
      @param hash   The hash state
      @param in     The data to hash
      @param inlen  The length of the data (octets)
      @return CRYPT_OK if successful
    */
    int (*process)(hash_state *hash, const unsigned char *in, unsigned long inlen);
    /** Produce the digest and store it
      @param hash   The hash state
      @param out    [out] The destination of the digest
      @return CRYPT_OK if successful
    */
    int (*done)(hash_state *hash, unsigned char *out);
    /** Self-test
      @return CRYPT_OK if successful, CRYPT_NOP if self-tests have been disabled
    */
    int (*test)(void);

    /* accelerated hmac callback: if you need to-do multiple packets just use the generic hmac_memory and provide a hash callback */
    int  (*hmac_block)(const unsigned char *key, unsigned long  keylen,
                       const unsigned char *in,  unsigned long  inlen,
                             unsigned char *out, unsigned long *outlen);

};
```

name 和 ID 不用多说，输出的哈希值大小是 32 字节，分组大小为 64 字节。

关于 OID 的定义，参考《M/T 0006-2012 密码应用标识规范》这份标准，可以查到 SM3 的 OID 为 1.2.156.10197.1.401 。所以 OID 字段填入：

```
{ 1, 2, 156, 10197, 1, 401, }
```
其长度是 6 。后面就是初始化、处理以及测试函数。最后一个是 HMAC 的加速实现，我们可以先使用标准的 HMAC 实现，所以这里可以填入 NULL。

测试用例可以参考《GM/T 0004-2012 SM3密码杂凑算法》文档的附录，里面有两个文本及对应的 SM3 哈希值。

最后来说说 HMAC，libtomcrypt 中已经有 HMAC 的实现，在使用 HMAC 时指定哈希算法为 SM3 即可。要添加测试用例，可以使用 gmssl 命令行工具，输入消息文本和密钥，输出 SM3 HMAC 值。这里有一个小技巧，可以在命令行参数中指定16进制数字的密钥：

```bash
$ echo -n "Hi There" |gmssl dgst -sm3 -mac hmac -macopt hexkey:0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b
(stdin)= 51b00d1fb49832bfb01c3ce27848e59f871d9ba938dc563b338ca964755cce70
```

有兴趣的同学也可以参考我的移植代码：https://gitee.com/mogoweb/libtomcrypt-gm.git

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)