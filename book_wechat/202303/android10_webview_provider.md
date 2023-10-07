
自 Android Lollipop 以来，WebView 一直是一个可更新的系统组件，实现的可更新部分作为 APK 或 App Bundle 分发。 我们将此可更新包称为设备上的“WebView 提供程序”。

某些操作系统映像允许用户将其 WebView 提供程序更改为非默认包，而其他映像仅支持一个预安装的默认包。 此表捕获了最常见设备配置的 WebView 提供程序选项：

修改 AOSP 的供应商可以配置哪些提供程序与其操作系统映像兼容，尽管他们通常坚持使用默认配置（上面标记为“AOSP”）。

提供包含 GMS 和 Play 商店的操作系统映像的供应商必须使用 Google 提供的 WebView 配置（上面标记为“有 GMS”）。 这是为了确保 Google 可以向用户提供 WebView 更新，我们通过 GTS 测试强制执行此操作。 大多数生产 Android 设备都使用此配置。

WebView 提供者要求
一个包必须满足几个要求才有资格成为 WebView 提供者。 这些要求由 WebView 更新服务强制执行，该服务作为设备上 Android 框架的一部分运行。

安装并启用
在 Android O+ 上，必须为所有用户配置文件安装并启用符合条件的 WebView 提供程序（某些 Android 功能是在后台使用多个用户配置文件实现的）。 在 Android L-N 上，只需要为默认用户配置文件安装和启用该包。

如果您卸载（或禁用）选定的 WebView 提供程序，WebView 更新服务将根据排序的首选项回退到不同的包（该顺序在操作系统映像中预先确定）。 如果没有更多符合条件的包（如果这是唯一的包或用户禁用/删除所有其他包），WebView 将根本无法工作，并且基于 WebView 的应用程序将崩溃，直到用户重新启用其中一个包。

在 Android N-P 上，com.google.android.webview 和 com.android.chrome 由于“回退逻辑”而相互排斥。 禁用（或启用）Chrome 将启用（或禁用）WebView 存根。 有关详细信息，请参阅 N-P 的重要说明。

包裹名字
出于安全原因，Android 将只允许预先确定的包名称列表充当 WebView 提供程序。 WebView 团队提供了几个不同的预设列表，具体取决于 Android 图像的配置方式。

签名（用于用户构建）
出于安全原因，Android 还会检查 WebView 提供者的签名，只允许使用预期发布密钥签名的应用程序。

此要求在 userdebug/eng 设备上被免除，因此我们可以在测试设备上安装本地 WebView 构建（没有发布密钥）。

签名可以在 AOSP 中配置。
targetSdk版本
一个有效的 WebView 提供者必须实现在该版本的 Android 平台中公开的所有 API，否则调用新的 API 将在运行时崩溃。 WebView 更新服务无法可靠地确定提供者实现了哪些 API，因此我们决定使用 targetSdkVersion 作为代理：

对于最终确定的 Android 版本，有效的 WebView 提供程序必须声明大于或等于平台的 Build.VERSION.SDK_INT 值的 targetSdkVersion。
对于预发布（也称为开发中）Android 版本，有效的 WebView 提供程序必须声明 targetSdkVersion 等于 Build.VERSION_CODES.CUR_DEVELOPMENT 并使用相应的预发布 SDK 进行编译。
在 Chromium 存储库中，我们通过设置 android_sdk_release = "x" 在 GN args 中配置它，其中“x”是所需操作系统版本的小写代号字母。 上游 Chromium 代码通常只支持最新的公共 Android 版本，因此您应该为所有公共 Android 操作系统版本使用该值。 使用内部存储库构建的 Google 员工可能能够覆盖它以针对当前预发布的 Android 版本。

注意：仅仅更改 APK 中的 targetSdkVersion 是不够的：新的 API 调用仍然会在运行时崩溃！ 您应该只使用 android_sdk_release = "x" 来配置它，因为这也会引入代码来实现新的 Android API。 有关详细信息，请参阅 AOSP 系统集成商的 WebView。
版本号
我们强制执行最低版本代码以确保安全性（以防止降级攻击）和正确性（这确保包可以在新版本的操作系统中实现所有新的 WebView API）。 这是由 WebView 更新服务在运行时通过采用系统映像上安装的所有有效 WebView 提供程序中的最小值来计算的。

对于本地构建，您通常不会遇到此问题，但如果您尝试安装一个非常旧的 WebView 官方构建，则可能会看到此问题。

声明一个本地库
由于 WebView 部分使用 C++ 实现，因此 Android 框架必须加载其原生库。 在L上，原生库必须叫libwebviewchromium.so。 M开头及以上，原生库必须通过AndroidManifest.xml中的com.android.webview.WebViewLibrary元数据标签声明。 如果您对这个过程的工作原理感到好奇，请参阅使用 RELRO 共享加载本机代码以获取更多详细信息。

你通常不应该遇到这个问题，除非你试图安装一个不支持 WebView 的目标（例如 chrome_public_apk 而不是 monochrome_public_apk）。