# 统信UOS / Deepin系统任务栏卡死解决方法

在使用统信 UOS / Deepin 系统时，偶尔会出现任务栏卡死的现象。具体说来就是系统底部的任务栏点了没有反应，此时已经打开的窗口可以响应，但是没法切换窗口。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_taskbar_01.png)

我之前在 Windows 下遇到过类似的问题，解决方法是杀掉任务栏进程，系统会自动再重启一个任务栏进程。在 UOS / Deepin 下是不是也有类似的解决方法？

首先按 Ctrl + Alt + Del 组合键，在界面上点击**启动系统监视器**。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_taskbar_02.png)

在程序进程中找到 org.deepin.dde-shell 这个进程。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_taskbar_03.png)

如果进程比较多，还可以通过搜索的方式

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2024/202409/images/deepin_taskbar_04.png)

找到进程后，鼠标右键点击这个进程，在右键菜单中点击**强制结束进程**。可以看到底部的任务栏会短暂的消失，然后又重新出现。这个时候**系统监视器**中又会看到 org.deepin.dde-shell 这个进程。

这个时候再去点任务栏上的图标，就有响应了。至此，问题得到解决。

至于出现这个问题的原因，太复杂了，可能是某个应用程序在响应操作系统绘制界面预览图的时候卡住，而 DDE 桌面系统没有很好的处理这种情况。连 Windows 这种经过了千锤百炼的系统都有这样的问题，可能造成这种问题的原因比想象中的要复杂。

国产系统出现这样那样的问题，有时确实比较影响体验，希望这样一则小技巧对你来说有用。

你在使用国产系统中碰到哪些问题，如何解决的，欢迎讨论。
