# Talk is cheap. Show me the code.如何写一个 Wayland 客户端程序？

前几天写了一篇万字长文《[万字长文详解 Wayland 协议、架构]()》，空谈协议分析有点枯燥，再说，程序员都比较信奉：Talk is cheap. Show me the code.

所以这篇文章就不来长篇大论，从一个简单的 Wayland 客户端程序的编写，来看一看 wayland 存在哪些坑。

我们要开发的 Wayland 客户端程序非常简单，就在一个窗口中显示 “Hello wayland”。通常来说，写图形程序最好使用 GTK、QT 这样的 GUI 框架，这样可以自动适应 X11、Wayland 等后端，但我们为了展示 Wayland 客户端的写法，就手搓一个。当然我们不用亲自动手，这种事交给 AI 就可以了。

