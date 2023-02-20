# Android 10 构建系统实战问题解决

本文记录一下最近项目中遇到的 Android 10 构建系统问题及解决方法。

首先说一下背景，之前的项目都是基于 Android 5.1 (是的，你没看错，相当古老的版本) 打造，近期打算将系统升级到 Android 10。我的工作主要是进行浏览器内核的迁移。

Android 系统的浏览器引擎早期是 WebKit，后来谷歌团队和 WebKit 团队 (主要是苹果公司的员工) 由于开发理念的不同，分道扬镳，独立在 WebKit 源码的基础上 fork 出一个新的分支 blink，经过一番魔改，现在 blink 的源码和苹果公司维护的 WebKit 已经相处甚远。谷歌公司的浏览器产品叫做 Chromium (商业版本为 Chrome)，由于 Chromium 和 blink 紧密的关系，我们一般也称浏览器内核为 Chromium。Android 框架中有个 WebView 组件，从 Android 4.4 开始就切换到 Chromium WebView。

然而，从 Android 6.0 系统开始，Android 源码中不再包含 chromium 的源码，而是以预置的 apk 的形势提供。这之前的 Android 5.1，Chromium WebView 也可以从 Chromium 源码中独立编译，不再依赖于 Android 构建系统。我们的项目采取了一种混合的方式，Chromium 源码没有采用 Android 5.1 附带的源码，但代码依然加入到 Android 5.1 的代码树，和系统一起编译。虽然编译过程中使用的 Chromium 的构建系统 GN，但编译过程中会链接到 Android 系统特有的 so。这也好理解，毕竟我们是基于 Android 深度定制，加入了一些自己特有的东西。

让问题变得棘手的是，Android 从 7.0 版本开始，切换到一套新的构建系统 Soong。但迁移过程估计没有那么顺利，直到 Android 10 版本，还是 Soong 和 Make 两套系统混用。所以现在项目中有的模块是用 Android.bp（Soong），有的模块使用老式 Android.mk 。无语的是，这并非终点，Soong 只是一个临时的方案，Google 最终计划迁移到 Basel 构建系统（Android 应用开发者应该眼熟）。

既然 Android 10 构建系统支持老式的 Android.mk，我窃喜，不用做什么修改就可以用了。可问题没那么简单，原因在于 Google 又引入了 Ninja 构建系统。是不是头大，前面不是说构建系统是 Soong 和 Make 混用吗，怎么又牵涉到 Ninja？

Soong 和 Ninja 关系有点类似于 cmake 和 make。cmake 实际上是从 CMakeLists.txt 先生成 Makefile，再使用 make 进行真正的构建。Soong 构建系统也是这样，先生成 Ninja 文件，最后通过 Ninja 进行构建。而 Android Make 这套体系，也是先从 Makefile 生成 Ninja 文件，和 Soong 生成的 Ninja 组合，最终使用 Ninja 构建整个 Android 系统。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202302/images/aosp_build_procedure.png)

对 Ninja，我并不陌生。其实在 chromium 上也进行过类似的折腾，先是 GYP -> Ninja，后来是 GN -> Ninja，Chromium 最终也是使用 Ninja 构建代码。

也就是说，沿用老式的 Android.mk ，也存在 MK -> Ninja 的过程。进行 mk 文件和 ninja 转换的工具叫做 kati，C++ 实现的版本为 ckati，在 Android 10 中使用的是预编译的 ckati。ckati 对 mk 的检查更加严格，为此引入了新的问题，这也是这几天我与之斗争的原因。

注：后文中的 weblink 及目录就是我们定制的 Chromium，可能会将 Chromium 和 weblink 混用。

#### 1. 忽略某个文件夹下的 Android.mk

Android 10 构建系统会扫描源码所有目录（包括子目录）的 Android.mk 和 Android.bp 文件，生成对应的 Ninja 文件。Chromium 下 third_party 目录下包含的第三方开源库，有些包含 Android.mk。Chromium 构建并没有使用到这些 Android.mk，但会被 Android 构建系统扫描到，并转换成对应的 Ninja 文件，转换过程中会出现诸如以下的错误：

```
FAILED: 
build/make/core/binary.mk:1257: error: component/weblink/src/third_party/android_crazy_linker/src/Android.mk: crazylinker_unittest: Unused source files: minitest/minitest.cc.
09:23:10 ckati failed with: exit status 1
```
一种方法就是将目录下的所有 Android.mk 删除或重命名，但我希望找到一个更好的解决方案：忽略某些目录下的 Android.mk。搜了一下网上的方法，有人给出方法，修改 build/core/config.mk 中的 FIND_LEAVES_EXCLUDES 变量定义。但这个方法是针对老的 Android Make 构建系统，并不适用于Android 10。

在网上搜了一圈，没找到答案，决定还是从 Android 10 构建系统入手，大致浏览了一下 Android Soong 构建系统的源码（使用 Go 语言编写，为此还快速入门了一下 Go 语言），很快找到解决方法。 build/soong/ui/build/finder.go 文件的 FindSources 函数，其作用就是遍历 Android 代码树下的所有 Android.bp、Android.mk 及其它构建文件。所以解决方法很简单，在

```
androidMks := f.FindFirstNamedAt(".", "Android.mk")
```
语句后面加上字符串过滤

```
	androidMksFiltered := []string{}
	for _, s := range androidMks {
		if !(strings.Contains(s, "weblink/src/third_party")) {
			androidMksFiltered = append(androidMksFiltered, s)
		}
	}
```

还有一种方法是修改该文件的如下定义：

```
	cacheParams := finder.CacheParams{
		WorkingDirectory: dir,
		RootDirs:         []string{"."},
		ExcludeDirs:      []string{".git", ".repo"},
		PruneFiles:       pruneFiles,
		IncludeFiles: []string{
			"Android.mk",
			"AndroidProducts.mk",
			"Android.bp",
			"Blueprints",
			"CleanSpec.mk",
			"OWNERS",
			"TEST_MAPPING",
		},
	}
```
ExcludeDirs 就是要忽略的文件，只不过这个影响要大些，会影响对 Android.bp 等文件的搜索，可以根据实际情况决定采用哪种方法。

#### 2. Error: writing to readonly directory: "weblink"

这个问题也是困扰了我半天，网上也没有搜到满意的答案，Google 的文档也语焉不详。没办法，还是从 Soong 源码入手。以 "writing to readonly directory" 为关键词搜索，找到 build/soong/scripts/build_broken_logs.go 中有定义：

```
{
		name:     "BUILD_BROKEN_PHONY_TARGETS",
		behavior: DefaultFalse,
		warnings: []string{
			"depends on PHONY target",
			"looks like a real file",
			"writing to readonly directory",
		},
	},
```
大致可以猜测因为 Android.mk 中定义了 .PHONY 构建目标。我们 weblink 中的 Android.mk 并不是一般所见的 mk 文件，更多的是类似 Makefile，调用一个脚本执行 weblink 的构建。关于 .PHONY 构建目标，文档中有这样一句话：

> There are several new warnings/errors meant to ensure the proper use of .PHONY targets in order to improve the speed and reliability of incremental builds.
>
> .PHONY-marked targets are often used as shortcuts to provide “friendly” names for real files to be built, but any target marked with .PHONY is also always considered dirty, needing to be rebuilt every build. This isn't a problem for aliases or one-off user-requested operations, but if real builds steps depend on a .PHONY target, it can get quite expensive for what should be a tiny build.

也就是 .PHONY 构建目标无法增量编译，会影响编译速度。但在实际中，这个没法避免。从上面的结构可以看到，其缺省行为是 false，所以现在的问题就是如何将缺省行为改为 true。

以 BUILD_BROKEN_PHONY_TARGETS 为关键字上网搜索，果然找到答案。方法就是修改 device 下的 BoardConfig.mk 文件，增加一行：

```
BUILD_BROKEN_PHONY_TARGETS := true
```

经过这样的修改，错误就变成了警告，不影响编译过程：

```
weblink/Android.mk:27: warning: writing to readonly directory: "weblink"
```

#### 3. error: Automatic variable `$?' isn't supported yet

上网搜索，给出的解决方案是修改 build/kati/command.cc 文件，将如下几行去掉：

```
INSERT_AUTO_VAR(AutoNotImplementedVar, "%");
INSERT_AUTO_VAR(AutoNotImplementedVar, "?");
INSERT_AUTO_VAR(AutoNotImplementedVar, "|");
```

照做之后，问题依然存在。怀疑是 ckati 没有重新编译。在源码下搜索 ckati 文件，果然位于 

> prebuilts/build-tools/linux-x86/bin/ckati

因为是使用的预编译版本，只修改源码当然无效。build/kati/ 下有 Makefile 文件，进入该目录，执行

```
make
```

就可以生成 ckati 在当前目录下，将其复制到 prebuilts/build-tools/linux-x86/bin/ 目录，问题解决。

#### 小结

项目还在进行着，还有不少的问题需要去解决，这在升级之初就预料得到的，我们能做的就是遇山开山，遇水搭桥，解决各种问题也是程序员的价值所在。有人会抱怨技术发展太快，像 Google 这样疯狂刷版本，折腾各种升级。其实只要能够深入进去，很多技术都是相通的。就拿构建系统来说，有 Make、Cmake、GYP、GN、Soong、Ninja、Bazel 等等，以后也不知道会整出什么花样，但只要你熟悉一两个构建系统，其它的也相差不太多。看看文档，上网查查资料，问题并不难解决。

在解决问题的过程中，中科院软件所的**汪辰**写的一系列文章对我帮助很大，加深了我对 Android 构建系统的理解。你可以从下面的地址访问：

> https://gitee.com/aosp-riscv/working-group/tree/master/articles

这里面的文章都写得不错，对 Android 系统开发有很大的帮助，感谢**汪辰**。

对了，在折腾 Android 构建系统时，我又快速入门了一门 Go 语言，但我并没有打算深入研究它，后续还是会继续研究 RUST 语言。

