# 写给 Android 开发人员的 RUST 课程

2021 年 4 月 6 日，谷歌发布博客称 AOSP (Android Open Source Project) 开始支持使用 Rust 开发 Android 操作系统。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/202303/images/rust_for_android_developer_01.jpg)

谷歌

在Android 13中，有约21%的新原生码都是以Rust开发，在AOSP中已经有大约150万行的Rust程序代码，涵盖各种功能和组件，包括Keystore2、UWB堆栈、DNS-over-HTTP3、Android虚拟化框架等

大家好，我想分享一下我过去几个月在工作中一直在做的事情：一个新的适合初学者的 Rust 课程，名为 Comprehensive Rust :crab: 105。你可以在这里找到源代码：GitHub - google/comprehensive -生锈 77。

我在 Android 上工作，我们已经支持 Rust 进行操作系统级开发 11 几年了。 为了帮助加快 Rust 在 Android 中的采用，我们编写了为期四天的课程，旨在让开发人员开始使用 Rust。 我们从基本语法、借用、所有权开始，然后向上移动到高级主题，例如泛型、特征和错误处理。 我们甚至触及了不安全的 Rust，并且我们还有一章包含特定于 Android 的信息。 不过，我们并没有涵盖所有内容：我希望尽快涵盖异步 Rust，而且我们还需要更多关于内部可变性的材料。 请使用问题跟踪器或 GitHub 讨论让我知道其他遗漏的主题——我相信我们可以涵盖很多内容 :slight_smile:

该课程可供任何人使用：我一直在内部授课，我们还与其他讲师一起上课。 如果您已经精通 Rust，您应该可以拿起课程材料并作为您的公司教授课程。 如果你这样做了，请告诉我进展如何！

