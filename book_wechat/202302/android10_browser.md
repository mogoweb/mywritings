# Android 10 中的浏览器构建

从 Android 4.4 开始，系统浏览器内核开始从 WebKit 切换到 Chromium。为了保持 API 兼容，Chromium 为 Android WebView 提供了 Chromium WebView 封装。最初 Chromium Webview 代码是位于 AOSP 源码树中，和 AOSP 源码一起构建。到了 Android 5.0，Chromium WebView 代码依然在 AOSP 源码树上，只是 Android 5.0 还支持单独升级 Chromium WebView，这时 Chromium WebView 由一个 名为 webview.apk （从 Chromium 源码 build 出来的叫 SystemWebView.apk，文件名不是那么重要）提供。由于是一个 APK，可以像普通应用 APK 那样安装、升级。到了 Android 6.0， AOSP 源码和 Chromium 源码彻底分离，AOSP 中不再包含 Chromium 的源码，取而代之的是一个 prebuilt 的 webview.apk 。

因为项目是基于 Android 10，所以这里说说 Android 10 中的浏览器开发。

1. AOSP 中不再包含原来的 Browser 代码，以前的浏览器是一个全功能浏览器，长这样：
   
   ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202302/images/android10_browser_01.jpg)

   而现在的 AOSP 中只包含了一个 WebView Shell，简陋之极：

   ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202302/images/android10_browser_02.png)

   Webview Shell 的代码 位于 packages/apps/Browser2/ 目录下。

2. AOSP 的预编译 Chromium Webview 位于 external/chromium-webview/ 目录下。该目录还有 arm, arm64, x86, x86_64 几个子目录，这是由于浏览器内核引擎主要使用 C++ 开发，所以针对不同的 CPU 架构有着不同的 apk。如果你的系统是针对特定的平台开发，只需要更新对应架构的 apk 即可。编译到 ROM 中的路径为 product/app/webview/ ，而不是以前的 /sytem/app/webview/ 。
3. Android 10 开始引入动态分区，所以在 ROM 镜像文件中找不到熟悉的 system.img，取而代之的是 super.img，super.img 包含 system, product, vendor 三个子分区。AOSP 预编译的 webview 被打包到 product 子分区。

4. 到了 Android 10，关于 WebView 你又有三种选择。
   * 独立的 Webview，也就是上面提到的预编译的 weview.apk。要在 chromium 中编译出来，构建目标选择 system_webview_apk，生成的输出文件称为 SystemWebView.apk。
   * Monochrome，一个单独的 APK，包含整个 WebView 实现，以及一个完整的基于 Chromium 的网络浏览器。 由于 WebView 和 Chromium 浏览器共享许多通用源代码，因此Monochrome APK 比具有单独的 WebView APK 和浏览器 APK 小得多。如果你的系统需要像 Chrome 这样的全功能浏览器，可以选择这种 WebView 变体。在 chromium 中构建目标称为 monochrome_public_apk，生成的输出文件称为 MonochromePublic.apk。
   * Trichrome， 由三个 APK/AAB 组成：
     * TrichromeWebView 包含特定于 WebView 的代码和数据，并为 Android 应用程序提供 WebView 实现。
     * TrichromeChrome 包含特定于浏览器的代码和数据，并为用户提供基于 Chromium 的网络浏览器。
     * TrichromeLibrary 包含共享代码和数据，仅用作 TrichromeWebView 和 TrichromeChrome 的内部实现细节。

    这三个 Trichrome APK 的大小与 Monochrome 大致相同，具有相同的优势，如果是 Android 10 及以上系统，且需要全功能浏览器，推荐使用这种 WebView 变体。在 Chromium 中的构建目标分别称为 trichrome_webview_apk、trichrome_chrome_bundle 和 trichrome_library_apk，生成的输出文件为 TrichromeWebView.apk、TrichromeChrome.aab 和 TrichromeLibrary.apk。

5. 关于 Webview 版本的选择，官方推荐使用最新的稳定版本，你可以访问

   > https://chromiumdash.appspot.com/releases?platform=Android 

   查询当前的稳定版和测试版版本号。但需要注意的是，Chromium 采用滚动发布的模式，版本更新非常频繁，开发产品，还是稳字当头，没有必要追求最新版本。

关于 Android 10 中的浏览器构建就先谈到这儿，当然最主要的工作还是从 Chromium 源码构建 WebView，以及对 Chromium 的定制，这个话题很大，有需要再说说。