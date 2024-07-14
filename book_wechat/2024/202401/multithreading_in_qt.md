# QT 中的多线程相关类

多线程程序设计差不多是每一位程序员必备的技能之一。维基百科中，对多线程的定义如下：

> 多线程是一种广泛使用的编程和执行模型，它允许多个线程存在于一个进程的上下文中。这些线程共享进程的资源，但能够独立执行。线程编程模型为开发人员提供了并发执行的有用抽象。多线程也可以应用于一个进程，以便在多处理系统上实现并行执行。

但对于 C++ 程序员而言，写多线程程序却不那么容易，特别是在进行跨平台开发的时候，因为很长一段时间里，C++ 语言和标准库中并没有多线程库。长期以来，Linux/Unix 程序员使用 pthread（POSIX 线程库），Windows 程序员使用 Windows 提供的多线程 API，Mac 程序员使用 GCD 框架。这种分裂导致很多大型软件，比如 Chromium，都有自己的线程封装库。QT 作为一个流行的跨平台开发框架，自然少不了自己的线程封装库。

当然，编写多线程程序难点并非在于使用多线程库，而在于竞争条件、同步、死锁等。多线程程序设计是一个非常大的题目，本文暂不深入讨论，仅仅解析一下 QT 框架提供的线程类。

### QThread

QThread 应该是 QT 程序员使用最多的一个多线程类。QThread提供了一个高层次的抽象，可以让开发人员更容易地进行多线程编程。下面就是一个 QThread 的例子：

```
class Thread : public QThread
{
protected:
    void run()
    {
        // Doing something ...
    };
};

int main()
{
    Thread t;
    t.start();     ①
    t.wait();

    return 0;
}
```

QThread的主要功能包括：

* 创建线程：使用QThread::create()方法可以创建一个线程。
* 启动线程：使用QThread::start()方法可以启动一个线程。
* 停止线程：使用QThread::terminate()方法可以停止一个线程。
* 等待线程结束：使用QThread::wait()方法可以等待一个线程结束。
* 获取线程状态：使用QThread::isRunning()方法可以获取线程是否正在运行。

QThread 可以直接实例化，也可以子类化。一般我们会定义一个 QThread 子类， 重写 run 方法，这里面就是我们希望在线程中执行的代码。当调用 start 方法后，就会执行 run() 方法中的代码。

### QThreadPool 和 QRunnable

线程池不是什么新鲜概念，在某些情况下，比如服务器端程序处理客户机请求，会存在频繁创建和销毁线程的情况。但是一般来说，频繁创建和销毁线程的成本可能会很高。为了减少这种开销，线程池就可以派上用场了。线程池的作用就是将现有线程重新用于新任务，而不是用完销毁。

QThreadPool是可重用的QThread的集合。

要在 QThreadPool 的线程之一中运行代码，请重新实现 QRunnable::run() 并实例化子类 QRunnable。 使用QThreadPool::start()将QRunnable放入QThreadPool的运行队列中。 当线程可用时，QRunnable::run() 中的代码将在该线程中执行。

每个Qt应用程序都有一个全局线程池，可以通过QThreadPool::globalInstance()访问。 该全局线程池根据 CPU 中的核心数量自动维护最佳线程数量。 然而，可以显式地创建和管理一个单独的QThreadPool。