+++
title = "asio-IOServicePool"
tags = [ "C/C++", "网络编程", "socket", "计算机网络", "Linux",]
categories = [ "技术",]
mathjax = false
date = "2024-08-24T22:51:44+08:00"
+++
## 简介

`IOServicePool`是一个用于管理多个`io_context`实例的多线程模型，每一个线程管理一个`io_context`，它的主要目的是在多线程环境中高效地分发和异步处理I/O操作

## 单线程和多线程对比

单线程模式图
![单线程模型.png](https://s2.loli.net/2024/08/24/1Kw5yAiT2DrJg3l.png)

多线程模式图
![IOServicePool.png](https://s2.loli.net/2024/08/24/j9vaAPC8WMzpce3.png)

### 优点

1. 每一个`io_context`跑在不同的线程里，所以同一个`socket`会被注册在同一个`io_context`里，它的回调函数也会被单独的一个线程回调，那么对于同一个`socket`，他的回调函数每次触发都是在同一个线程里，就不会有线程安全问题，网络io层面上的并发是线程安全的。

2. 但是对于不同的`socket`，回调函数的触发可能是同一个线程(两个`socket`被分配到同一个`io_context`)，也可能不是同一个线程(两个`socket`被分配到不同的`io_context`里)。所以如果两个`socket`对应的上层逻辑处理，如果有交互或者访问共享区，会存在线程安全问题。比如`socket1`代表玩家1，`socket2`代表玩家2，玩家1和玩家2在逻辑层存在交互，比如两个玩家都在做工会任务，他们属于同一个工会，工会积分的增加就是共享区的数据，需要保证线程安全。可以通过加锁或者逻辑队列的方式解决安全问题，我们目前采取了后者。

3. 多线程相比单线程，极大的提高了并发能力，因为单线程仅有一个`io_context`服务用来监听读写事件，就绪后回调函数在一个线程里串行调用, 如果一个回调函数的调用时间较长肯定会影响后续的函数调用，毕竟是穿行调用。而采用多线程方式，可以在一定程度上减少前一个逻辑调用影响下一个调用的情况，比如两个`socket`被部署到不同的`iocontext`上，但是当两个`socket`部署到同一个`iocontext`上时仍然存在调用时间影响的问题。不过我们已经通过逻辑队列的方式将网络线程和逻辑线程解耦合了，不会出现前一个调用时间影响下一个回调触发的问题。

## IOServicePool实现

IOServicePool本质上是一个线程池，基本功能就是根据构造函数传入的数量创建n个线程和iocontext，然后每个线程跑一个iocontext，这样就可以并发处理不同iocontext读写事件了。  
声明如下：

```c++
#pragma once
// NOTE:
// 如果要使用这个线程池的话，
// 将CServer的io_context替换为AsioIOServicePool::GetInstance()->get_io_service()
// main函数需要给服务器创建一个io_context

#include <thread>
#include <boost/asio.hpp>
#include <memory>
#include <vector>
#include "Singleton.hpp"
class AsioIOServicePool : public Singleton<AsioIOServicePool> {
    friend class Singleton<AsioIOServicePool>;

public:
    using IOService = boost::asio::io_context;
    using Work      = boost::asio::io_context::work;
    using WorkPtr   = std::unique_ptr<Work>;
    ~AsioIOServicePool();
    AsioIOServicePool( const AsioIOServicePool & )            = delete;
    AsioIOServicePool &operator=( const AsioIOServicePool & ) = delete;

    boost::asio::io_context &get_io_service();
    void Stop();

private:
    explicit AsioIOServicePool(
        size_t thread_size = std::thread::hardware_concurrency() );
    std::vector<IOService> _ioServices;
    std::vector<WorkPtr> _works;
    std::vector<std::thread> _threads;
    std::size_t _nextIOservice; // 轮询索引
};

```

实现如下：

```c++
AsioIOServicePool::AsioIOServicePool(size_t thread_size)
    : _ioServices(thread_size), _works(thread_size), _nextIOservice(0) {

    for (size_t i = 0; i < thread_size; ++i) {
        _works[i] = std::make_unique<Work>(Work(_ioServices[i]));
    }

    for (auto &IOService : _ioServices) {
        _threads.emplace_back([this, &IOService]() { IOService.run(); });
    }
}

boost::asio::io_context &AsioIOServicePool::get_io_service() {
    auto &service = _ioServices[_nextIOservice++ % _ioServices.size()];
    return service;
}

void AsioIOServicePool::Stop() {
    for (auto &work : _works) {
        work.reset();
    }

    for (auto &t : _threads) {
        t.join();
    }
}

AsioIOServicePool::~AsioIOServicePool() {
    std::cout << "AsioIOServicePool destroyed" << std::endl;
}

```

这段代码实现了一个基于 `boost::asio` 的 I/O 服务池（`AsioIOServicePool`），并使用了单例模式（通过 `Singleton<AsioIOServicePool>`）来确保全局只有一个实例。以下是对代码的详细解释：

#### 类定义

```c++
class AsioIOServicePool : public Singleton<AsioIOServicePool> {
    friend class Singleton<AsioIOServicePool>;

public:
    using IOService = boost::asio::io_context;
    using Work      = boost::asio::io_context::work;
    using WorkPtr   = std::unique_ptr<Work>;
    ~AsioIOServicePool();
    AsioIOServicePool( const AsioIOServicePool & )            = delete;
    AsioIOServicePool &operator=( const AsioIOServicePool & ) = delete;

    boost::asio::io_context &get_io_service();
    void Stop();

private:
    explicit AsioIOServicePool(
        size_t thread_size = std::thread::hardware_concurrency() );
    std::vector<IOService> _ioServices;
    std::vector<WorkPtr> _works;
    std::vector<std::thread> _threads;
    std::size_t _nextIOservice; // 轮询索引
};
```

- `AsioIOServicePool` 继承自 `Singleton<AsioIOServicePool>`，确保它是单例的。
- `friend class Singleton<AsioIOServicePool>;`：允许 `Singleton` 类访问 `AsioIOServicePool` 的私有成员。
- 使用 `using` 别名定义了 `IOService`、`Work` 和 `WorkPtr`。
- 删除了拷贝构造函数和赋值操作符，防止对象被复制。
- `get_io_service()` 方法用于获取一个 `io_context` 实例。
- `Stop()` 方法用于停止所有 `io_context` 并等待线程结束。
- 私有构造函数 `AsioIOServicePool` 接受一个线程数量参数，默认为硬件并发数。
- 成员变量包括 `_ioServices`（`io_context` 数组）、`_works`（工作对象数组）、`_threads`（线程数组）和 `_nextIOservice`（轮询索引）。

#### 实现文件

```c++
AsioIOServicePool::AsioIOServicePool(size_t thread_size)
    : _ioServices(thread_size), _works(thread_size), _nextIOservice(0) {

    for (size_t i = 0; i < thread_size; ++i) {
        _works[i] = std::make_unique<Work>(Work(_ioServices[i]));
    }

    for (auto &IOService : _ioServices) {
        _threads.emplace_back([this, &IOService]() { IOService.run(); });
    }
}
```

- 构造函数初始化 `_ioServices`、`_works` 和 `_nextIOservice`。
- 为每个 `io_context` 创建一个 `Work` 对象，防止 `io_context` 在没有工作时退出。
- 为每个 `io_context` 创建一个线程，并在该线程中运行 `io_context`。

```c++
boost::asio::io_context &AsioIOServicePool::get_io_service() {
    auto &service = _ioServices[_nextIOservice++ % _ioServices.size()];
    return service;
}
```

- `get_io_service()` 方法通过轮询方式返回一个 `io_context` 实例。

```c++
void AsioIOServicePool::Stop() {
    for (auto &work : _works) {
        work.reset();
    }

    for (auto &t : _threads) {
        t.join();
    }
}
```

- `Stop()` 方法重置所有 `Work` 对象，使 `io_context` 可以退出。
- 等待所有线程结束。

```c++
AsioIOServicePool::~AsioIOServicePool() {
    std::cout << "AsioIOServicePool destroyed" << std::endl;
}
```

- 析构函数输出一条消息，表示对象被销毁。

### 关于优雅退出

IOServicePool多线程服务器退出时，需要捕获退出信号如`SIGINT`、`SIGTERM`等，然后通知所有线程退出，然后等待线程结束，将退出信号和一个`io_context`绑定，当接收到退出信号时，我们将IOServicePool中的`io_context`退出，然后等待所有线程退出，最后销毁线程池。

```c++
int main() {
    try {
        auto pool = AsioIOServicePool::GetInstance();
        boost::asio::io_context  io_context;
        boost::asio::signal_set signals(io_context, SIGINT, SIGTERM);
        signals.async_wait([&io_context,pool](auto, auto) {
            io_context.stop();
            pool->Stop();
            });
        CServer s(io_context, 10086);
        io_context.run();
    }
    catch (std::exception& e) {
        std::cerr << "Exception: " << e.what() << endl;
    }
}
```
