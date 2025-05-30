# 预置国密根证书到浏览器

由于国密算法没有得到国外的认可，所以 Chromium、Firefox 等浏览器均不支持国密算法。即使我们修改了 Chromium 的源码，增加了国密算法的支持，但还不能在浏览器中正常使用。因为这涉及到证书的信任问题，国密证书都是国内厂商签发的，国密根证书并没有集成到系统和浏览器中。这样在访问国密网站时，浏览器会提示证书不受信任。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/add_nss_db_01.png)

因此，我们需要一种方法，在安装浏览器时，将国密根证书加入到 Chromium 的授信证书库中。

Chromium 在 Linux 上使用 NSS 共享数据库（通常在 `~/.pki/nssdb`）来存储用户证书和信任根证书。默认启动时，Chromium 通过 `NSS_InitReadWrite("sql:$HOME/.pki/nssdb")` 初始化 NSS 并打开用户的持久数据库；之后还会加载内置根证书模块 `libnssckbi.so`。

一种解决思路是将根证书添加到 `~/.pki/nssdb` 中，但在 deb 软件包安装时，使用的是 root 用户，需要比较麻烦的方法获知当前用户，然后写入到当前用户的 `$HOME/.pki/nssdb`。还有一个问题是，如果切换到另外一个用户，那这个预置就会失效。

因此，最好是**保留用户数据库的正常交互**（GUI 导入证书仍写入 `~/.pki/nssdb`），同时在启动时额外加载一个只读的 NSS 证书库目录（例如 `/opt/apps/org.mojo.browser/files/.pki/nssdb`），使得其中的根证书对 Chromium 自动信任，但不在用户的证书管理界面中显示。

## NSS 多数据库加载原理

NSS 支持通过 PKCS#11 模块 API 同时打开多个“用户数据库”（User DB）。官方文档介绍，可调用 `SECMOD_OpenUserDB` 打开新的软件令牌数据库（token）并得到一个 `PK11SlotInfo*`，参数可指定 `configDir`（数据库路径）、`tokenDescription`（令牌描述）、以及可选的 `flags=readOnly` 等。例如，文档指出 `flags` 可设为 `readOnly` 表示以只读方式打开数据库。Chromium 源码中也封装了一个帮助函数 `OpenSoftwareNSSDB(path, description)`，其内部就是调用 `SECMOD_OpenUserDB("configDir='sql:path' tokenDescription='desc'"…)`。Chrome 的 NSS 初始化逻辑会记录打开的 Slot，在之后自动引用并保持它们有效。这表明我们可以在 Chromium 的初始化流程中额外调用一次 `SECMOD_OpenUserDB`（或 `OpenSoftwareNSSDB`）来加载第二个证书库。

## Chromium 源码的关键位置与修改方式

Chromium 在 `crypto/nss_util.cc` 中的 `NSSInitSingleton` 构造函数负责 NSS 的初始化。在那里，它会调用 `NSS_InitReadWrite("sql:$HOME/.pki/nssdb")` 打开用户数据库。接着设置密码回调、初始化内部 key slot，然后调用 `LoadNSSModule("Root Certs", "libnssckbi.so", nullptr)` 加载内置根证书。我们可以在这个初始化流程中插入额外的步骤，用 `SECMOD_OpenUserDB` 或者修改后 `OpenSoftwareNSSDB` 打开另一个数据库槽。示例修改位置在 `NSSInitSingleton()` 末尾、加载完根证书模块之后，如下伪代码所示：

```cpp
// 在加载完 NSS root 模块后，增加加载额外只读数据库的代码
// 注意：插入于 crypto/nss_util.cc 的 NSSInitSingleton 构造函数中
const base::FilePath system_nssdb_dir("/opt/apps/org.mojo.browser/files/.pki/nssdb");
if (!system_nssdb_dir.empty()) {
  // 构造 PKCS#11 模块参数字符串，指定只读打开
  std::string sys_modspec = base::StringPrintf(
      "configDir='sql:%s' tokenDescription='System CA NSSDB' flags=readOnly",
      system_nssdb_dir.value().c_str());
  // 打开只读的 NSS 数据库槽
  PK11SlotInfo* sys_slot = SECMOD_OpenUserDB(sys_modspec.c_str());
  if (sys_slot) {
    // NSS 会要求初始化 PIN；对于只读数据库，PIN 为空，强制初始化
    if (PK11_NeedUserInit(sys_slot))
      PK11_InitPin(sys_slot, nullptr, nullptr);
    // 不需要使用时可以释放引用；数据库仍保持加载状态
    PK11_FreeSlot(sys_slot);
    LOG(INFO) << "Loaded read-only NSS DB: " << system_nssdb_dir.value();
  } else {
    LOG(ERROR) << "Failed to open system NSS DB at " << system_nssdb_dir.value()
               << ": " << GetNSSErrorMessage();
  }
}
```

* 在上面代码中，`configDir='sql:/opt/apps/org.mojo.browser/files/.pki/nssdb'` 指定要打开的目录为额外的证书数据库；`tokenDescription` 可自定义名称；`flags=readOnly` 确保该数据库不会被写入。调用 `SECMOD_OpenUserDB` 返回一个 `PK11SlotInfo*`，表示已加载的令牌槽。然后我们检查是否需要初始化 PIN（一般只读无需密码，传入 `nullptr` 即可），最后用 `PK11_FreeSlot` 释放本次引用，但该 slot 仍留存在 NSS 中以供信任决策使用。
* 如果希望使用 Chromium 封装函数，可将其改造为支持 flags。例如修改 `OpenSoftwareNSSDB(path, description)`，在 `modspec` 字符串中追加 `" flags=readOnly"`（参照），然后调用 `SECMOD_OpenUserDB`。或者新增一个类似 `OpenSoftwareNSSDBReadOnly(path, description)` 的接口。在任何情况下，关键是调用 NSS API 打开第二个数据库槽并保持其生效。

经过上述修改后，Chromium 启动时会**同时加载用户 NSSDB 和系统 NSSDB**。NSS 在查找信任根证书时会遍历所有已加载的模块/槽，因此 `/opt/apps/org.mojo.browser/files/.pki/nssdb` 中的根证书会被当作信任 CA，而用户界面 (`chrome://settings/certificates`) 仍然只显示和操作 `~/.pki/nssdb` 中的条目，从而**不干扰原有 GUI**。

## 编译与测试

1. **修改并编译 Chromium**：将上述代码添加到 `crypto/nss_util.cc` 的 `NSSInitSingleton` 构造函数末尾，保证在 `LoadNSSModule("Root Certs", …)` 后执行。重新编译 Chromium，确认 NSS 相关逻辑编译通过。
2. **准备测试证书库**：使用 `certutil` 等工具在 `/opt/apps/org.mojo.browser/files/.pki/nssdb` 创建 NSS 数据库并导入测试根证书。例如：

   ```bash
   certutil -N -d sql:/opt/apps/org.mojo.browser/files/.pki/nssdb    # 创建数据库
   certutil -A -d sql:/opt/apps/org.mojo.browser/files/.pki/nssdb \
     -t "C,," -n "Test CA" -i /path/to/gm-root.pem  # 导入并设置信任
   ```

   证书数据库可以去网上找，比如沃通的根证书可以去[这里](https://wosign.com/WotrusRoot/index.htm)下载。
   ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/add_nss_db_02.png)
3. **启动 Chromium 并验证**：打开 Chromium（确保使用与之关联的 `$HOME` 的 NSSDB），访问一个配置国密的站点, 比如 [https://sm2only.ovssl.cn](https://sm2only.ovssl.cn)，观察是否正常打开。同时，在 `chrome://settings/certificates` 中查看“受信任的根证书”列表，确认该 CA **不出现在界面内**（因为它仅存在只读数据库中）。
![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202505/images/add_nss_db_03.png)
4. **使用命令行验证**：可以分别列出两个数据库的证书：`certutil -L -d sql:$HOME/.pki/nssdb`（用户库）应**无**国密 CA；而 `certutil -L -d sql:/opt/apps/org.mojo.browser/files/.pki/nssdb` 则应含有 “国密 CA”。这确保 GUI 只见用户库内容，Chromium 信任链查询却能看到系统库证书。

通过以上步骤，我们就可以将国密根证书预置到浏览器中并生效，同时仍然支持通过设置界面导入根证书。
