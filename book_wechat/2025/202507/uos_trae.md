# 统信 Windows 应用兼容引擎兼容性提升，让我也能用上 TRAE！

两个月前，我写过一篇文章介绍字节出品的 AI IDE —— [《字节真的豪横，这款 IDE 接入国内外主流大模型，还免费》](https://mp.weixin.qq.com/s/3lfRnTOZzS3P23pEGr3b1g)。当时为了试用这款 IDE，我特地切换到了 Windows 系统，因为 TRAE 目前仅推出了 Windows 和 MacOS 版本，Linux 版本仍停留在 Waitlist 阶段。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_01.png)

那时我也尝试过通过 Wine 来运行 Windows 版 TRAE，但遗憾的是，并未成功。由于我主要的工作环境是国产操作系统，在简单尝试了 TRAE 后，就没有继续使用。

最近，`统信 Windows 应用兼容引擎`发布了全新版本 —— **3.3.1**，修复并优化了大量应用兼容性问题。与此同时，TRAE 推出了新功能「SOLO」，看上去非常强大，于是我再次动了尝试的念头。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_02.png)

正如我在前一篇的文章[《十年磨一剑，让国产系统流畅运行 Windows 应用》](https://mp.weixin.qq.com/s/XCnY_C1yaUJ0tJCnBk5GBQ)中提到的，`统信 Windows 应用兼容引擎`早已不只是一个简单的兼容工具，它正逐步向生态构建方向迈进。

兼容引擎本身相当于一个内嵌的“应用商店”，收录了大量经过验证可稳定运行的 Windows 程序，其中就包括 **Trae CN**。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_03.png)

不过，这个 TRAE 版本是中文版，而我更想体验的是官方推出的国际版。于是，我前往 TRAE 官网 [https://trae.ai](https://trae.ai) 下载了最新的安装程序 `Trae-Setup-x64.exe`。

接下来，打开 `统信 Windows 应用兼容引擎`，在左侧导航栏点击 **“我的应用”**，然后选择 **“添加应用”**，在文件选择对话框中定位到刚刚下载的安装程序即可。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_04.png)

安装过程与在 Windows 上无异，一路“下一步”即可。安装完成后，你会在“我的应用”中看到新增的 TRAE 应用，点击 **“运行”** 就可以启动了。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_05.png)

TRAE 国际版支持简体中文，因此运行后会自动根据系统语言显示为中文界面，非常友好。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_06.png)

完成几个基础设置后，即可进入登录界面。**注意：这个登录步骤不要跳过！** 因为后续调用大模型接口必须完成登录，否则使用时仍会弹出提示要求登录。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_07.png)

点击登录后，会跳转浏览器打开一个认证页面。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_08.png)

需要特别提醒的是：**TRAE 国际版目前不支持中国大陆地区的用户直接使用**。如果你使用的是国内 IP，会看到如下提示：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_09.png)

解决方案嘛……你懂的。一个比较简单的方法是选择使用 Google 账号登录。

完成登录后，就可以畅快地使用 TRAE IDE 了。值得一提的是，字节现在似乎也“没那么大方”了，目前仅默认接入了 Claude 3.5、Claude 3.7 以及 DeepSeek R1 三个模型。想用其他模型的用户，则需要自行填入 API Key 及相关配置信息。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_10.png)

---

最近 AI 编程工具如雨后春笋般涌现，比如腾讯刚发布的 **CodeBuddy**，以及阿里的 **Qwen Coder 编程模型**。可以看出，各大厂都瞄准了一个共同的方向——**“加速开发”乃至“取代程序员”**。

或许，程序员这个职业，正在被程序员亲手干掉。

---
