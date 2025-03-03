# 交叉编译 ARM 架构浏览器（补充）

在上一篇文章《[deepin Linux 系统上交叉编译 ARM 架构浏览器](https://mp.weixin.qq.com/s/Ob_bGXtA9xLsAFbh62UFdQ)》中，我们探讨了 Chromium 浏览器交叉编译的基本流程。但在后续的调整编译参数的过程中，发现一些新的问题。为此，这篇文章对上一篇文章进行一个补充，补充一些遗漏的细节。

## 一

在编译 Chromium 源码的过程中，你可能会碰到如下错误：

```
../../chrome/browser/ui/webui/top_chrome/webui_contents_wrapper.h:13:10: fatal error: 'chrome/browser/ui/webui_name_variants.h' file not found
#include "chrome/browser/ui/webui_name_variants.h"
         ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
1 error generated.

In file included from ../../chrome/browser/ui/views/side_panel/customize_chrome/customize_chrome_side_panel_controller.cc:19:
../../chrome/browser/ui/webui/side_panel/customize_chrome/customize_chrome_ui.h:12:10: fatal error: 'chrome/browser/cart/chrome_cart.mojom.h' file not found
#include "chrome/browser/cart/chrome_cart.mojom.h"
         ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
1 error generated.
```

这个问题并不是必现，我在办公室的机器上构建就没有碰到这个错误，但是在家里的机器上会出这个错，使用的都是相同的 deepin V23 系统。在网络上搜了一下，在 Chromium 的某些版本上确实存在这样的问题，对此，debian 构建系统提供了一个 patch， 打上这个 patch 就能解决。针对 Chromium 127.0.6533.99 源码的 patch 如下：

```
diff --git a/chrome/browser/ui/views/side_panel/BUILD.gn b/chrome/browser/ui/views/side_panel/BUILD.gn
index ea57f9b16f6b8..20d8681ee9af2 100644
--- a/chrome/browser/ui/views/side_panel/BUILD.gn
+++ b/chrome/browser/ui/views/side_panel/BUILD.gn
@@ -142,18 +142,30 @@ static_library("side_panel") {
   }
   public_deps = [
     "//base",
+    "//chrome/browser/cart:mojo_bindings",
     "//chrome/browser/companion/core/mojom:mojo_bindings",
     "//chrome/browser/profiles:profile",
     "//chrome/browser/ui/actions:actions_headers",
     "//chrome/browser/ui/color:color_headers",
+    "//chrome/browser/ui/webui/side_panel/customize_chrome:mojo_bindings",
+    "//chrome/browser/ui/webui/side_panel/bookmarks:mojo_bindings",
     "//chrome/browser/ui/webui/side_panel/performance_controls:mojo_bindings",
+    "//chrome/browser/ui/webui/side_panel/reading_list:mojo_bindings",
+    "//chrome/browser/ui:webui_name_variants",
     "//chrome/common",
     "//chrome/common/accessibility:mojo_bindings",
+    "//chrome/common/companion:mojo_bindings",
+    "//components/enterprise/buildflags:buildflags",
+    "//components/enterprise/common/proto:connectors_proto",
     "//components/lens",
     "//components/lens:buildflags",
     "//components/omnibox/browser",
+    "//components/page_image_service/mojom:mojo_bindings",
+    "//components/paint_preview/buildflags:buildflags",
     "//components/prefs",
     "//components/search_engines",
+    "//components/segmentation_platform/public/proto:proto",
+    "//components/webapps/common:mojo_bindings",
     "//content/public/browser",
     "//extensions/browser",
     "//extensions/common",
@@ -165,6 +177,8 @@ static_library("side_panel") {
     "//ui/gfx/geometry",
     "//ui/views",
     "//ui/views/controls/webview",
+    "//ui/webui/resources/cr_components/commerce:mojo_bindings",
+    "//ui/webui/resources/cr_components/help_bubble:mojo_bindings",
     "//url",
   ]
   deps = [
diff --git a/chrome/browser/ui/webui/top_chrome/BUILD.gn b/chrome/browser/ui/webui/top_chrome/BUILD.gn
index 5830f1f5e234a..6749b8fd0da56 100644
--- a/chrome/browser/ui/webui/top_chrome/BUILD.gn
+++ b/chrome/browser/ui/webui/top_chrome/BUILD.gn
@@ -19,6 +19,7 @@ source_set("top_chrome") {
   deps = [
     "//base",
     "//chrome/browser/profiles:profile",
+    "//chrome/browser/ui:webui_name_variants",
     "//components/site_engagement/content:content",
     "//content/public/browser",
     "//ui/webui",
```

如果是其它 Chromium 版本，可能需要稍微做一下调整。

## 二

文章中我只添加了 target_cpu = "arm64" 构建参数，其它的均使用默认参数。默认参数下 is_component_build = true，所以除了拷贝 chrome 和相关资源文件之外，还需要拷贝下面的 so 文件，由于默认是 debug build，总共加起来文件有好几个 g，比较大。

拷贝到 arm64 的系统上运行，出现如下错误：

```
./chrome: error while loading shared libraries: libui_ozone.so: cannot open shared object file: No such file or directory
```

加上 LD_LIBRARY_PATH 指定动态库加载路径也不管用，后来才发现是 so 编译的架构有问题：

```
uos@uos-PC:~/Browser$ file ./chrome
./chrome: ELF 64-bit LSB pie executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, for GNU/Linux 3.7.0, BuildID[xxHash]=e4b0fa4a3ae3d1c4, with debug_info, not stripped
uos@uos-PC:~/Browser$ file ./libui_ozone.so 
./libui_ozone.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[xxHash]=a70856eb28aaad36, with debug_info, not stripped
```

可以看到，chrome 可执行程序是 arm64 架构的，但 so 却是 x86-64 架构，当然就无法加载。

我们可以对编译参数做一下调整，使用如下命令：

```
gn gen out/Default-arm64 --args="target_cpu=\"arm64\" is_component_build=false is_debug=false enable_nacl=false symbol_level=0"
ninja -C out/Default-arm64 chrome
```

这样编译出来的 Chromium 浏览器可以运行在飞腾 CPU 的机器上。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202503/images/cross_compile_chromium_01.png)

## 小结

Chromium 交叉编译出的可执行文件和动态库居然是不同的架构，还存在编译依赖问题。这反映了现代软件工程面临的挑战：随着软件系统复杂度呈指数级增长，以及开发环境的多样化，传统的构建系统已经难以全面覆盖所有可能的使用场景。

这篇技术总结旨在为开发者提供切实可行的解决方案，希望能够帮助大家在国产化系统的浏览器开发过程中少走弯路。如果在实践过程中遇到任何问题，欢迎随时交流探讨，让我们共同推动国产化系统生态的持续优化与发展。