# Android 10 WebView 踩坑实录

项目要求支持 8K 高清视频（H265编码）播放，拿到板子后却发现使用 App 可以播放 8K 高清视频，但使用浏览器却不行，即使安装上最新的 Chrome for Android 也不行。根据以往的浏览器内核开发经验，在 Android 平台上，Chromium WebView 最终是调用系统框架层的 MediaPlayer 进行播放。理论上只要系统框架层能够支持 8K 高清播放，那么浏览器应该也支持。实际情况却并非如此，而且 Android 10 预编译 WebView 没任何日志输出，所以需要下载源码编译 Chromium WebView，找出问题所在。

#### 版本选择坑

Chromium 源码更新非常平凡，而且架构也经常变化，不像我们做项目，一套代码恨不得修修补补用上十几年。上一个项目是基于 Chromium V53 进行定制的，这次并不想采用最新的 Chromium 版本，大概浏览了一下最新源码，和 V53 差别太大，以往的定制工作要移植过来相当麻烦。首先想到的是直接使用 V53 的源码，但无法应用到 Android 10 上，主要是 Android 10 的 WebView API 接口发生了一些变化。最终令我放弃的是 Android 10 框架层移除了 HardwareCanvas 类，要知道，在 Android 5.1 中，WebView 中有一个重要的绘制方法：

```
public void callDrawGlFunction(Canvas canvas, long nativeDrawGLFunctor) {
    if (!(canvas instanceof HardwareCanvas)) {
        // Canvas#isHardwareAccelerated() is only true for subclasses of HardwareCanvas.
        throw new IllegalArgumentException(canvas.getClass().getName()
                + " is not hardware accelerated");
    }
    ((HardwareCanvas) canvas).callDrawGLFunction2(nativeDrawGLFunctor);
}
```

不能使用 V53，接下来考虑 Android 10 中预编译的 Chromium Webview 版本，使用 WebView Shell，查看版本号为 74.0.3729.183：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/android10_webview_trouble_01.png)

然而，这里有一个巨大的坑。等我费了九牛二虎之力把代码下载下来，把 Chromium WebView 编译出来并安装到系统，结果浏览器启动就崩溃，查看系统日志，有如下错误信息：

```
03-05 00:41:09.457  3299  4011 W WebViewUpdater: creating relro file timed out
03-05 00:41:09.458 12565 12565 E WebViewFactory: Chromium WebView package does not exist
03-05 00:41:09.458 12565 12565 E WebViewFactory: android.webkit.WebViewFactory$MissingWebViewPackageException: Failed to load WebView provider: No WebView installed
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.webkit.WebViewFactory.getWebViewContextAndSetProvider(WebViewFactory.java:339)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.webkit.WebViewFactory.getProviderClass(WebViewFactory.java:402)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.webkit.WebViewFactory.getProvider(WebViewFactory.java:252)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.webkit.WebView.getFactory(WebView.java:2551)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.webkit.WebView.setWebContentsDebuggingEnabled(WebView.java:1974)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at org.chromium.webview_shell.WebViewBrowserActivity.onCreate(WebViewBrowserActivity.java:230)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.app.Activity.performCreate(Activity.java:7802)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.app.Activity.performCreate(Activity.java:7791)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1306)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:3245)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:3409)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.app.servertransaction.LaunchActivityItem.execute(LaunchActivityItem.java:83)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.app.servertransaction.TransactionExecutor.executeCallbacks(TransactionExecutor.java:135)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.app.servertransaction.TransactionExecutor.execute(TransactionExecutor.java:95)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:2016)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.os.Handler.dispatchMessage(Handler.java:107)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.os.Looper.loop(Looper.java:214)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at android.app.ActivityThread.main(ActivityThread.java:7368)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at java.lang.reflect.Method.invoke(Native Method)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:492)
03-05 00:41:09.458 12565 12565 E WebViewFactory:        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:930)
03-05 00:41:09.459 12565 12565 D AndroidRuntime: Shutting down VM
```

查看了一下框架层代码，一个合格的 WebView Provider，首先其 targetSdk 版本号要大于或等于 29，而查看 chromium v74 源码的 build/config/android/sdk.gni 文件：

```
# The default SDK release used by public builds. Value may differ in
# internal builds.
default_android_sdk_release = "p"

# SDK releases against which public builds are supported.
public_sdk_releases = [ "p" ]
```

Android 10 的代号为 Q，也就是说这个版本的 Chromium WebView 实际上只支持到 Android 9，也不知道 Google 使用了什么魔法。可能是当时 Android 10 在开发过程中，使用了内部的 SDK 进行构建。

这个时候，我还是不想使用最新版本，那就采用正式支持 Android Q 的最早版本吧。方法就是查看这个 sdk.gni 的修改记录，看看哪个提交从 "p" 修改到 "q"。因为代码的分支超级多，必须要查看所有分支的 git 提交：

```
$ git log -p  --all -G 'sdk' sdk.gni
```

查到 commit id 为 c2481863282a401926e0ee479334c68ec362d302 ，接下来查询哪些分支包含这个提交：

```
$ git branch -a --contains c2481863282a401926e0ee479334c68ec362d302
  remotes/branch-heads/3967
  remotes/branch-heads/3968
  remotes/branch-heads/3969
  remotes/branch-heads/3970
  remotes/branch-heads/3971
  remotes/branch-heads/3972
  remotes/branch-heads/3973
  remotes/branch-heads/3974
  remotes/branch-heads/3975
  remotes/branch-heads/3976
  remotes/branch-heads/3977
  remotes/branch-heads/3978
  remotes/branch-heads/3979
  remotes/branch-heads/3980
  remotes/branch-heads/3981
  remotes/branch-heads/3982
  remotes/branch-heads/3983
  remotes/branch-heads/3984
  remotes/branch-heads/3985
  remotes/branch-heads/3986
  remotes/branch-heads/3987
  remotes/branch-heads/3987_100
  remotes/branch-heads/3987_137
  remotes/branch-heads/3987_158
  remotes/branch-heads/3987_87
  remotes/branch-heads/3988
  remotes/branch-heads/3989
  remotes/branch-heads/3990
  remotes/branch-heads/3991
  remotes/branch-heads/3992
  remotes/branch-heads/3993
  remotes/branch-heads/3994
  remotes/branch-heads/3995
  remotes/branch-heads/3996
  remotes/branch-heads/3997
  remotes/branch-heads/3998
  remotes/branch-heads/3999
  remotes/branch-heads/4000
  remotes/branch-heads/4001
  remotes/branch-heads/4002
  remotes/branch-heads/4003
  remotes/branch-heads/4004
  remotes/branch-heads/4005
  remotes/branch-heads/4006
  remotes/branch-heads/4007
  remotes/branch-heads/4008
  remotes/branch-heads/4009
  remotes/branch-heads/4010
  remotes/branch-heads/4011
  remotes/branch-heads/4012
```

包含这个提交的分支非常多，数字越大，表明分支代码越新，所以这里搜索这中间的一个稳定发布版本即可。访问如下链接：

> https://chromiumdash.appspot.com/releases?platform=Android 

这里包含了所有曾经发布过的版本，最终我选择的是 V80.0.3987.165 。在趟过 代码下载坑、编译坑、安装坑后，终于成功运行起来：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/android10_webview_trouble_02.png)

#### 代码下载坑

由于众所周知的原因， Chromium 源码不能直接下载，我是挂了代理进行下载。然而在下载时经常出现如下错误：

```
remote: Counting objects: 867615, done
remote: Finding sources: 100% (10514569/10514569)
error: RPC failed; curl 56 GnuTLS recv error (-9): Error decoding the received TLS packet.
fatal: the remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

Git 没有断点续传机制，下载失败了需要重来，碰到 Chromium 这样的超级大库，下载到十几 G 的时候断掉，想死的心都有。上网查了很多方法都不管用，后来在一个帖子中找到解决方法：

```
git config --global http.postBuffer 5242880000
git config --global https.postBuffer 5242880000
sudo ifconfig eth0 mtu 9000
```

网上的指导一般是将 http.postBuffer 设为一个比较大的值，对于 Chromium 这样单个 git 库超过 30 G 的还是不够，我将缓冲设成 5G 大小，终于不再出错。另外有指导将 MTU 大小设置为 14000，但在 Ubuntu 上无法设成功，我尝试出来的最大值为 9000。

源码 clone 成功后，执行 gclient runhooks 命令又出现如下错误：

```
NOTICE: You have PROXY values set in your environment, but gsutilin depot_tools does not (yet) obey them.
Also, --no_auth prevents the normal BOTO_CONFIG environmentvariable from being used.
To use a proxy in this situation, please supply those settingsin a .boto file pointed to by the NO_AUTH_BOTO_CONFIG environmentvariable.
```

这是使用了代理的原因，解决的方法很简单，创建一个文本文件（比如 $HOME/.boto），内容为：

```
[boto]
proxy=代理IP
proxy_port=代理端口
```

然后设置环境变量 NO_AUTH_BOTO_CONFIG 指向这个文件：

```
export NO_AUTH_BOTO_CONFIG=$HOME/.boto
```

至此，终于解决了源码下载和二进制文件下载的问题。

#### 代码编译坑

编译代码过程中，出现如下错误：

```
FAILED: gen/build/android/buildhooks/build_hooks_android_java.javac.jar gen/build/android/buildhooks/build_hooks_android_java.javac.jar.info 
python ../../build/android/gyp/javac.py --depfile=gen/build/android/buildhooks/build_hooks_android_java__compile_java.d --generated-dir=gen/build/android/buildhooks/build_hooks_android_java/generated_java --jar-path=gen/build/android/buildhooks/build_hooks_android_java.javac.jar --java-srcjars=\[\] --java-version=1.8 --full-classpath=@FileArg\(gen/build/android/buildhooks/build_hooks_android_java.build_config:deps_info:javac_full_classpath\) --interface-classpath=@FileArg\(gen/build/android/buildhooks/build_hooks_android_java.build_config:deps_info:javac_full_interface_classpath\) --processorpath=@FileArg\(gen/build/android/buildhooks/build_hooks_android_java.build_config:javac:processor_classpath\) --processors=@FileArg\(gen/build/android/buildhooks/build_hooks_android_java.build_config:javac:processor_classes\) --java-srcjars=@FileArg\(gen/build/android/buildhooks/build_hooks_android_java.build_config:deps_info:owned_resource_srcjars\) --bootclasspath=@FileArg\(gen/build/android/buildhooks/build_hooks_android_java.build_config:android:sdk_interface_jars\) --chromium-code=1 --errorprone-path bin/errorprone @gen/build/android/buildhooks/build_hooks_android_java.sources
Traceback (most recent call last):
  File "../../build/android/gyp/javac.py", line 595, in <module>
    sys.exit(main(sys.argv[1:]))
  File "../../build/android/gyp/javac.py", line 590, in main
    add_pydeps=False)
  File "/data/chromium/chromium_v74.0.3729.183/src/build/android/gyp/util/build_utils.py", line 653, in CallAndWriteDepfileIfStale
    pass_changes=True)
  File "/data/chromium/chromium_v74.0.3729.183/src/build/android/gyp/util/md5_check.py", line 87, in CallAndRecordIfStale
    function(*args)
  File "/data/chromium/chromium_v74.0.3729.183/src/build/android/gyp/util/build_utils.py", line 638, in on_stale_md5
    function(*args)
  File "../../build/android/gyp/javac.py", line 584, in <lambda>
    lambda: _OnStaleMd5(options, javac_cmd, java_files, classpath),
  File "../../build/android/gyp/javac.py", line 350, in _OnStaleMd5
    stderr_filter=ProcessJavacOutput)
  File "/data/chromium/chromium_v74.0.3729.183/src/build/android/gyp/util/build_utils.py", line 227, in CheckOutput
    raise CalledProcessError(cwd, args, stdout + stderr)
util.build_utils.CalledProcessError: Command failed: ( cd /data/chromium/chromium_v74.0.3729.183/src/out/Default; bin/errorprone -g -encoding UTF-8 -sourcepath : -XepDisableAllChecks -source 1.8 -target 1.8 -Xlint:unchecked -Werror -bootclasspath lib.java/third_party/android_tools/android.interface.jar:/usr/lib/jvm/java-8-openjdk-amd64/jre/lib/rt.jar -d /tmp/tmpzNZrEr/classes @/tmp/tmpzNZrEr/files_list.txt )
-Xbootclasspath/p is no longer a supported option.
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

原因是 Ubuntu 20.04 使用的 JDK 版本为 11，但这个版本的 Chromium 编译需要 JDK 8，解决的方法就是将 JDK 切换到 JDK：

```
$ java --version
openjdk 11.0.18 2023-01-17
OpenJDK Runtime Environment (build 11.0.18+10-post-Ubuntu-0ubuntu120.04.1)
OpenJDK 64-Bit Server VM (build 11.0.18+10-post-Ubuntu-0ubuntu120.04.1, mixed mode, sharing)
$ sudo update-alternatives --config java
[sudo] password for alex: 
There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
  2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

Press <enter> to keep the current choice[*], or type selection number: 2
update-alternatives: using /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java to provide /usr/bin/java (java) in manual mode
$ java -version
openjdk version "1.8.0_362"
OpenJDK Runtime Environment (build 1.8.0_362-8u362-ga-0ubuntu1~20.04.1-b09)
OpenJDK 64-Bit Server VM (build 25.362-b09, mixed mode)
```

JDK 切换后，成功编译出 system_webview_apk。

#### WebView 安装坑

编译出  system_webview_apk 后进行安装，出现如下错误：

```
Failure [INSTALL_FAILED_UPDATE_INCOMPATIBLE: Package com.android.webview signatures do not match previously installed version; ignoring!]
- Performing Streamed Install
```

这是由于 Chromium 编译所使用的证书和 Android 10 的 platform 证书不一致导致的。解决这一问题的方法就是将两边的证书弄成一样。

第一步，查看 Android 10 中 weview.apk 的签名信息。使用压缩工具解压出 apk 中的 META-INF/WEBVIEW-.RSA 文件，然后使用如下命令查看签名信息：

```
$ keytool -printcert -file META-INF/WEBVIEW-.RSA
```

第二步，确定 Android 10 使用的证书。一般使用 build/target/product/security/ 下的证书，也有的产品使用自定义的证书，一般放在 device 下的产品目录。如果确定呢？查看一下证书的签名信息，对得上就是的。

```
openssl x509 -in testkey.x509.pem -noout -fingerprint
```

第三步，导入 keystore。Chromium 中使用的是 Java 应用的比较广的 keystore 格式，所以需要将 Android 10 中的证书导入进来。

```
# 私钥格式 pkcs8 转 PEM
$ openssl pkcs8 -inform DER -nocrypt -in $ANDROID_SOURCE/build/target/product/security/testkey.pk8 -out testkey.pem
# 证书从 PEM 转 pkcs12
$ openssl pkcs12 -export -in $ANDROID_SOURC/build/target/product/security/testkey.x509.pem -inkey testkey.pem -out testkey.p12 -name "android_testkey"
Enter Export Password: (输入密码android，默认是android，如是自己制作的key，输入对应的密码)
Verifying - Enter Export Password: (输入密码android)
# 生成 keystore
$ keytool -importkeystore -deststorepass android -destkeypass android -srckeystore testkey.p12 -srcstoretype pkcs12 -srcstorepass android -destkeystore AOSP.keystore -alias android_testkey
```

第四步，修改 gn args，使用自己的 keystore，修改 out/Default/args.gn 文件，加入如下行：

```
# 假设 AOSP.keystore 文件位于 chromium src 目录下
android_keystore_path = "//AOSP.keystore"
android_keystore_name = "android_testkey"
android_keystore_password = "android"
```

第五步，重新 build system_webview_apk

```
autoninja -C out/Default system_webview_apk
```

再次安装，可以成功。

可以预料，后面还会继续踩坑。没办法，只能遇坑填坑，这不就是程序员的工作职责吗？

欢迎关注后续进展！

