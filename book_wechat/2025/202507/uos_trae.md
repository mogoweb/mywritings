# 统信 Windows 应用兼容引擎兼容性提升，让我也能用上 TRAE

两个月之前，我写过字节出品的 AI IDE：[字节真的豪横，这款 IDE 接入国内外主流大模型，还免费](https://mp.weixin.qq.com/s/3lfRnTOZzS3P23pEGr3b1g)。当时为了能尝试这款 AI IDE，还特地切换到 Windows 系统。由于 TRAE 到目前也只推出了 Windows 版本和 Mac OS 版本，Linux 版本一直处于 Waitlist。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_01.png)

当时我也尝试过使用 Wine 来运行 Windows 版本的 TRAE，没有成功。

最近，`统信 Windows 应用兼容引擎`推出了新版本，版本来到3.3.1，解决了很多应用兼容问题，再加上听说 TRAE 推出了重要的 SOLO 功能，很厉害的样子，于是我又进行了尝试。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_02.png)

正如我上篇文章[十年磨一剑，让国产系统流畅运行 Windows 应用](https://mp.weixin.qq.com/s/XCnY_C1yaUJ0tJCnBk5GBQ)所写，`统信 Windows 应用兼容引擎`不再是一个简单的工具，开始向构建生态的方向发展。

`统信 Windows 应用兼容引擎`相当于内部集成了一个应用商店，有着许多经过验证的可以运行的 Windows 应用程序，这其中就有 Trae CN。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_03.png)

当然，这个 TRAE 中文版并非我所希望使用的版本，我想使用的是 TRAE 国际版。所以我就前往 TRAE 国际版的官网 https://trae.ai 下载 Windows 版本安装程序 Trae-Setup-x64.exe。

然后打开`统信 Windows 应用兼容引擎`，点击左边分栏的**我的应用**，再点击**添加应用**按钮。在文件选择对话框中选择刚刚下载的 TRAE 安装程序。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_04.png)

安装过程与 Windows 下安装一样，都按照默认选项一步步往下走就可以，这里不赘述。安装完毕后，在**我的应用** 就多了 Trace 这个应用，点击**运行**即可启动。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_05.png)

TRAE 是国际版，也支持简体中文，所以运行后 TRAE 根据系统语言，显示的也是简体中文界面。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_06.png)

再进行几步简单的设置后，就来到了登录界面。这个界面不要跳过，因为使用大模型是一定需要登录的，如果此时跳过，后面使用时还是会要求登录的。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_07.png)

点击登录后，会跳转到浏览器，打开一个页面。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_08.png)

需要注意的是 TRAE 国际版是不面向中国区的，所以如果使用国内 IP 登录，会出现如下提示：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_09.png)

这个问题需要自行解决，最简单的方法是选择谷歌账号登录。登录之后，就可以开始使用 TRAE IDE 了。话说字节也没有那么大方了，只提供了 claude 3.5、claude 3.7 和 DeepkSeek R1 三款模型，如果使用其它的模型，则需要自己填入账号、API key。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202507/images/uos_trae_09.png)

最近 AI IDE 扎堆发布，这几天还有腾讯的 CodeBuddy、阿里的Q