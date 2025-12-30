+++
title = "并发编程-3"
tags = [ "C/C++", "并发编程",]
categories = [ "技术",]
date = "2024-03-26T17:24:09+08:00"
+++
# 利用条件变量实现线程安全队列
> 本文介绍如何使用条件变量控制并发的同步操作  
## 条件变量


试想有一个线程A一直输出1，另一个线程B一直输出2。我想让两个线程交替输出1，2，1，2…之类的效果，该如何实现？有的同学可能会说不是有互斥量mutex吗？可以用一个全局变量num表示应该哪个线程输出，比如num为1则线程A输出1，num为2则线程B输出2，mutex控制两个线程访问num，如果num和线程不匹配，就让该线程睡一会，这不就实现了吗？比如线程A加锁后发现当前num为2则表示它不能输出1，就解锁，将锁的使用权交给线程A，线程B就sleep一会。

```c++

int num = 1;
std::mutex mtx;
void PoorImplemention() {
    std::thread t1( []() {
        while ( true ) {
            {
                std::lock_guard<std::mutex> lock( mtx );
                if ( num == 1 ) {
                    std::cout << "This thread A print " << num << std::endl;
                    ;
                    num++;
                }
            }
            std::this_thread::sleep_for( std::chrono::seconds( 1 ) );
        }
    } );

    std::thread t2( []() {
        while ( true ) {
            {
                std::lock_guard<std::mutex> lock( mtx );
                if ( num == 2 ) {
                    std::cout << "This thread B print " << num << std::endl;
                    num--;
                }
            }
            std::this_thread::sleep_for( std::chrono::seconds( 1 ) );
        }
    } );

    t1.join(), t2.join();
}

```

`PoorImplement`虽然能实现我们交替打印的功能，会造成消息处理的不及时处理，因为线程A要循环检测`num`值，如果`num`不为1,则线程A就睡眠了，在线程A睡眠这段时间里面很可能线程B已经处理完了，此时A还在睡眠，是对资源的浪费，也错过了最佳的处理时机。所以我们提出了用条件变量来通知线程的机制，当线程A发现条件不满足时可以挂起，等待线程B通知，线程B通知线程A后，A被唤醒继续处理

```c++
int num = 1;
std::mutex mtx;
std::condition_variable cvA , cvB;
void ReasonableImplemention() {
    std::thread t1( []() {
        while ( true ) {
            std::unique_lock<std::mutex> lock(mtx);
            cvA.wait(lock , [](){
                return num == 1;
            });

            num++;
            std::cout << "Thread A print 1...." << std::endl;
            cvB.notify_one();
        }
    } );

    std::thread t2( []() {
        while ( true ) {
            std::unique_lock<std::mutex> lock(mtx);
            cvB.wait(lock , [](){
                return num == 2;
            });

            num--;
            std::cout << "Thread B print 2...." << std::endl;
            cvA.notify_one();
        }
    } );

    t1.join(), t2.join();
}

```

当条件不满足时(num != 1)`cvA.wait`就会挂起，等待线程B通知线程A唤醒，线程B采用`cvA.notify_one`。这么做的好处是线程交替处理十分及时，比起`sleep`的方式，我们可以从控制台看出差异效果，`sleep`的方式看出日志基本是每隔1秒才打印一次，效率不高

## 线程安全队列

之前我们实现过线程安全的栈，对于pop操作，我们如果在线程中调用empty判断是否为空，如果不为空，则pop，因为empty和pop内部分别加锁，是两个原子操作，导致pop时可能会因为其他线程提前pop导致队列为空，从而引发崩溃。我们当时的处理方式是实现了两个版本的pop，一种是返回智能指针类型，一种通过参数为引用的方式返回。对于智能指针版本我们发现队列为空则返回空指针，对于引用版本，
发现队列为空则抛出异常，这么做并不是很友好，所以我们可以通过条件变量完善之前的程序，不过这次我们重新实现一个线程安全队列。

```c++
#pragma once

#include <memory>
#include <mutex>
#include <condition_variable>
#include <queue>

template <typename T> class Queue_Thread_Safe {
private:
    std::queue<T> data_;
    std::mutex mtx_;
    std::condition_variable cvq_;

public:
    Queue_Thread_Safe() = default;
    Queue_Thread_Safe( const Queue_Thread_Safe & );
    void push( T && );
    void wait_and_pop( T && );
    std::shared_ptr<T> wait_and_pop();
    bool try_pop( T& );
    std::shared_ptr<T> try_pop();
    bool empty();
};

template <typename T>
Queue_Thread_Safe<T>::Queue_Thread_Safe( const Queue_Thread_Safe &other ) {
    std::lock_guard<std::mutex> lock( other.mtx_ );
    this->data_ = other.data_;
}

template <typename T> void Queue_Thread_Safe<T>::push( T &&param ) {
    std::lock_guard<std::mutex> lock( mtx_ );
    data_.emplace( param );
    // NOTE: 这里通知线程是因为如果别的线程有pop操作，由于队列可能是空的会被挂起，所以要通知一个线程
    cvq_.notify_one();
}

template <typename T> void Queue_Thread_Safe<T>::wait_and_pop( T &&value ) {
    std::unique_lock<std::mutex> lock( mtx_ );
    cvq_.wait( lock, [this]() { return !data_.empty(); } );
    value = data_.front();
    data_.pop();
}

template <typename T> std::shared_ptr<T> Queue_Thread_Safe<T>::wait_and_pop() {
    std::unique_lock<std::mutex> lock( mtx_ );
    cvq_.wait( lock, [this]() { return !data_.empty(); } );
    std::shared_ptr<T> res( std::make_shared<T>( data_.front() ) );
    data_.pop();
    return res;
}

template<typename T> bool Queue_Thread_Safe<T>::try_pop(T& value) {
    std::unique_lock<std::mutex> lock(mtx_);
    if(empty()) {
        return false;
    }
    value = data_.front();
    return true;
}

template<typename T>
std::shared_ptr<T> Queue_Thread_Safe<T>::try_pop() {
    std::unique_lock<std::mutex> lock(mtx_);
    cvq_.wait(lock, [this](){
        return !data_.empty();
    });
    std::shared_ptr<T> res( std::make_shared<T>(data_.front()) );
    data_.pop();
    return res;
}


template<typename T> bool Queue_Thread_Safe<T>::empty() {
    // WARN:这里记得要加个锁，因为在判断队列是否为空的时候，要保证状态一致
    std::unique_lock<std::mutex> lock(mtx_);
    return data_.empty();
}

// 测试函数
void test_thread_safe_queue() {
    thread_safe_queue<int> safe_queue;
    std::mutex mtx_print;
    std::thread producer( [&]() {
        for ( int i = 0;; i++ ) {
            safe_queue.push( std::forward<decltype(i)>(i) );
            {
                std::lock_guard<std::mutex> lock_print( mtx_print );
                std::cout << "producer push data is " << i << std::endl;
            }
            std::this_thread::sleep_for( std::chrono::milliseconds( 200 ) );
        }
    } );

    std::thread consumer1( [&]() {
        while ( true ) {
            auto data = safe_queue.wait_and_pop();
            {
                std::lock_guard<std::mutex> lock_print( mtx_print );
                std::cout << "consumer1 wait and pop data is " << *data
                          << std::endl;
            }
            std::this_thread::sleep_for( std::chrono::milliseconds( 500 ) );
        }
    } );

    std::thread consumer2( [&]() {
        while ( true ) {
            auto data = safe_queue.try_pop();
            if ( data != nullptr ) {
                {
                    std::lock_guard<std::mutex> lock_print( mtx_print );
                    std::cout << "consumer2 try pop data is " << *data
                              << std::endl;
                }
            }
            std::this_thread::sleep_for( std::chrono::milliseconds( 500 ) );
        }
    } );

    producer.join();
    consumer1.join();
    consumer2.join();
}

```

输出如下:

```
producer push data is 0
consumer1 wait and pop data is 0
producer push data is 1
producer push data is 2
consumer2 try pop data is 1
consumer1 wait and pop data is 2
producer push data is 3
producer push data is 4
consumer1 wait and pop data is 3
consumer2 try pop data is 4
producer push data is 5
producer push data is 6
producer push data is 7
consumer2 try pop data is 5
consumer1 wait and pop data is 6
producer push data is 8
producer push data is 9
consumer2 try pop data is 7
consumer1 wait and pop data is 8
producer push data is 10
producer push data is 11
producer push data is 12
consumer2 try pop data is 9
consumer1 wait and pop data is 10
producer push data is 13
producer push data is 14
consumer2 try pop data is 11
consumer1 wait and pop data is 12
producer push data is 15
producer push data is 16
```

# C++异步

在C++中`future`, `promise`和`async`是C++标准库中的一些重要概念，它们可以用于实现异步编程。它们的具体用法可以参考[官方文档](https://en.cppreference.com/w/cpp/thread/future)。

## async用法

`std::async`是用于异步执行函数的函数模板，它返回一个`future`对象，该对象用于获取函数的返回值

例子:

```c++
#include <iostream>
#include <chrono>
#include <future>
#include <thread>

// 定义一个异步任务
std::string FetchFromDB(const std::string& query) {
    //模拟一个异步任务，比如从数据库中获取数据
    std::this_thread::sleep_for(std::chrono::milliseconds(200));
    return "Data: " + query;
}

void async_demo() {
    // NOTE: 使用std::async异步调用FetchFromDB
    std::future<std::string> resultFromDB = std::async(std::launch::async , FetchFromDB , "Data");
    // 主线程中做其他事情
    std::cout << "Doing something else" << std::endl;
    // 从future对象中获取数据
    std::string dbData = resultFromDB.get();
    std::cout << dbData << std::endl;
}

int main() {
    async_demo();

    return 0;
}
```

在这个事例中，`std::async`创建了一个新的线程(或从内部线程池中挑选了一个线程)并自动与一个`std::promise`对象相关联。`std::promise`对象被传递给`FetchFromDB`函数，函数返回值被存储在`std::future`对象中，在主线程中，我们可以使用`std::future::get`方法从`std::future`对象中获取数据。注意，在使用`std::async`的情况下，我们必须使用`std::launch::async`标志来明确表示我们希望函数异步执行。

输出如下：

```
Doing something else
Data: Data
```

## async的启动策略

`std::async`函数可以接受几个不同的启动策略，这些策略在`std::launch`枚举中定义。除了`std::launch::async`之外，还有以下启动策略

1. `std::launch::deferred`:这种策略意味着任务将在调用`std::future::get()`或`std::future::wait()`函数时延迟执行。换句话说，任务将在需要结果时同步执行。
2. `std::launch::async | std::launch::deferred`: 这种策略是上面两个策略的组合。任务可以在一个单独的线程上异步执行，也可以延迟执行，具体取决于实现。

默认情况下，`std::async`使用`std::launch::async|std::launch::deferred`策略。这意味着任务可能异步执行，也可能延迟执行，具体取决于实现。需要注意的是，不同的编译器和操作系统可能会有不同的默认行为。

## future的wait和get

`std::future::get()`和`std::future::wait()`是c++中用于处理异步人物的两个方法，它们的功能和用法有一些重要区别。

1. std::future::get()

`std::future::get()`是一个阻塞调用，用于获取`std::future`对象表示的值或异常。如果异步任务还没有完成，`get()`会阻塞当前线程，直到任务完成。如果任务已经完成，`get()`会立即返回结果。重要的是，`get()`只能调用一次，因为它会移动或消耗掉`std::future`对象的状态。一旦`get()`被调用，`std::future`对象就不能再被用来获取结果。

2. std::future::wait()

`std::future::wait`也是一个阻塞调用，但它与`get()`的主要区别在与`wait()`不会返回任务的结果。它只是等待异步任务完成。如果任务已经完成，`wait()`会立即返回。如果任务没有完成，`wait()`会阻塞当前线程，直到任务完成。与`get()`不同，`wait()`可以被多次调用，它不会消耗掉`std::future`对象的状态。

总结：

- `std::future::get()`用于获取并返回任务的结果，而`std::future::wait()`只是等待任务完成。
- `get()`只能被调用一次，而`wait()`可以被多次调用。
- 如果任务还没有完成，`get()`和`wait()`都会阻塞当前线程，但`get()`会一直阻塞知道任务完成并返回结果，而`wait()`只是在等待任务完成。

你可以使用`std::future`的`wait_for()`或`wait_until()`方法来检查异步操作是否完成。这些方法返回一个表示操作状态的`std::future_status`的值

```c++
if(fut.wait_for(std::chrono::seconds(0)) == std::future_status::ready) {
// 操作完成
} else {
// 操作尚未完成
}
```

### 将任务和future关联

`std::packaged_task`和`std::future`是C++11中引入的两个类，它们用于处理异步任务的结果。

`std::packaged_task`是一个可调用目标，它包装了一个任务，该任务可以在另一个线程上运行。它可以捕获任务的返回值或异常，并将其存储在`std::future`对象中，一边以后使用。

以下是使用`std::packaged_task`和`std::future`对象的基本步骤：

1. 创建一个`std::packaged_task`对象，该对象包装了要执行的任务。
2. 调用`std::packaged_task`对象的`get_future()`方法，该方法返回一个与任务关联的`std::future`对象
3. 在另一个线程上调用`std::packaged_task`对象的`operator()`，用于执行任务
4. 在需要任务结果的地方，调用与任务关联的`std::future`对象的`get()`方法，以获取任务的返回值或异常

例子:

```c++
int mytask() {
    std::this_thread::sleep_for(std::chrono::seconds(5));
    std::cout << "my task run 5s" << std::endl;
    return 52;
}

void use_package() {
    // 创建了一个包装了任务的std::packaged_task对象
    std::packaged_task<int()> task(mytask);

    // 获取与任务关联的std::future对象
    std::future<int> result = task.get_future();

    // 在另一个线程上执行任务
    std::thread t(std::move(task)); // NOTE: std::packaged_task对象不能被复制，只能移动。因为std::packaged_task内部保存了一个对应的执行任务，这个任务应该被唯一执行，并且任务的结果也应该唯一保存，因此不允许复制，只能移动。
    t.detach(); // 将线程与主线程分离，以便主线程可以等待任务完成

    // 等待任务完成并获取结果
    int value = result.get(); // get在获取结果之前会阻塞当前线程
    std::cout << "The result is: " << value << std::endl;
}
```

在上面的实例中，我们创建了一个包装了任务的`std::packaged_task`对象，并获取了与任务关联的`std::future`对象，然后，我们在另一个线程上执行任务，并等待任务完成并获取结果。最后，我们输出结果。

我们可以使用`std::function`和`std::packaged_task`来包装带参数的函数。`std::packaged_task`是一个模板类，他包装了一个可调用对象，并允许我们将其作为异步任务传递。

## promise的用法

C++11引入了`std::promise`和`std::future`两个类，用于实现异步编程。`std::promise`用于在某一线程中设置某个值或异常，而`std::future`则用于在另一线程中获取这个值或异常。

例子：

```c++
void set_value(std::promise<int> prom) {
    prom.set_value(10);
    std::cout << "promise set value successfully by the thread\n";
}

void promise_demo() {
    std::promise<int> prom;

    std::future<int> fut = prom.get_future();

    std::thread t1(set_value , std::move(prom));

    std::cout << "Waiting for the thread to set value ...\n";
    std::cout << "Value set by the thread: " << fut.get() << "\n";
    t1.join();
}
```

输出:

```
Waiting for the thread to set value ...
Value set by the thread: promise set value successfully by the thread
10
```

在上面的代码中，我们首先创建了一个`std::promise<int>`对象，然后通过调用`get_future()`方法获取与之相关联的`std::future<int>`对象。然后，我们在新线程中通过调用`set_value()`方法设置`promise`的值，并在主线程中通过调用`fut.get()`方法获取这个值。注意，在调用`fut.get()`方法时，如果`promise`的值还没有被设置，则该方法会阻塞当前线程，直到值被设置为止。

除了`set_value`方法外，`std::promise`还有一个`set_exception()`方法，用于设置异常。该方法接受一个`std::exception_ptr`参数，该参数可以通过调用`std::current_exception()`方法获取。

例子如下：

```c++
void set_exception(std::promise<void> prom) {
    try {
        // 抛出一个异常
        throw std::runtime_error("An error occurred\n");

    } catch(...) { // NOTE: ... 表示捕获任意类型的异常
        // 设置promise的异常
        prom.set_exception(std::current_exception());

    }
}

void promise_exception_demo() {
    std::promise<void> prom;
    // 获取与promise相关联的对象
    std::future<void> fut = prom.get_future();
    // 创建一个线程
    std::thread t1(set_exception , std::move(prom));
    // 在主线程获取future的异常
    try{
        std::cout << "Waiting for the thread to set exception....\n";
        fut.get();
    } catch(const std::exception& e) {
        std::cout << "Exception set by the thread: " << e.what() << "\n";
    }

    t1.join();
}

```

输出如下

```
Waiting for the thread to set exception....
Exception set by the thread: An error occurred
```

当然我们在使用`std::promise`时要注意一点，如果`std::promise`被释放了，而其他线程还未使用与`std::promise`关联的future,当其使用这个`std::future`时会报错。

例子：

```c++

void use_promise_destruct() {
    std::thread t;
    std::future<int> fut;

    {
        std::promise<int> prom;
        fut = prom.get_future();
        t = std::thread(set_value , std::move(prom));
    }

    std::cout << "Waiting for the thread to set value ...\n";
    std::cout << "Value set by the thread: " << fut.get() << "\n";
    t.join();
}
```

随着局部`}`的结束，`prom`可能被释放也可能会被延迟释放，如果立即释放则`fut.get()`获取的值会报`error_value`的错误

## 共享型的future

当我们需要多个线程等待同一个执行结果时，需要使用`std::shared_future`

以下是一个适合使用`std::shared_future`的场景，多个线程等待一个异步操作的结果

假设你有一个异步任务，需要多个线程等待其完成，然后这些线程需要访问任务的结果。在这种情况下，你可以使用`std::shared_future`来共享异步结果

```c++

void myFunction(std::promise<int>&& prom) {
    // 模拟一些工作

    std::this_thread::sleep_for(std::chrono::seconds(1));
    prom.set_value(42);

}

void threadFunction(std::shared_future<int> fut) {
    try{
        int result = fut.get();
        std::cout << "Result: " << result << "\n";
    } catch(const std::future_error& e) {
        std::cout << "Future error: " << e.what() << std::endl;
    }
}

void shared_future_demo() {
    std::promise<int> prom;
    std::shared_future<int> shared_fut = prom.get_future();

    // NOTE: 第一个线程先去执行任务，后面两个线程等待std::shared_future的值
    std::thread myThread1(myFunction , std::move(prom));
    std::thread myThread2(threadFunction, shared_fut);
    std::thread myThread3(threadFunction , shared_fut);
    myThread1.join() , myThread2.join() , myThread3.join();
}

```

我们创建了一个`std::promise<int>`对象`prom`和一个与之关联的`std::shared_future<int>`对象`shared_fut`。然后我们将`promise`对象移动到另一个线程`myThread1`中，该线程将执行`myFunction`函数，并在完成后设置`prom`的值，那么`shared_fut.get()`将返回该值。这些线程可以同时访问和等待`future`对象的结果，而不会相互干扰

注意，如果一个`future`被移动给两个`shared_future`是错误的

```c++
void threadFunction(std::shared_future<int> fut) {
    try{
        int result = fut.get();
        std::cout << "Result: " << result << "\n";
    } catch(const std::future_error& e) {
        std::cout << "Future error: " << e.what() << std::endl;
    }
}

void shared_future_demo() {
    std::promise<int> prom;
    std::shared_future<int> shared_fut = prom.get_future();

    std::thread myThread1(myFunction , std::move(prom));
    std::thread myThread2(threadFunction, std::move(shared_fut));
    std::thread myThread3(threadFunction , std::move(shared_fut));
    myThread1.join() , myThread2.join() , myThread3.join();
}

```

这种用法是错误的，一个`future`通过隐式构造传递给`shared_future`之后，这个`shared_future`被移动传递给两个线程是不合理的，因为第一次移动之后`shared_future`的生命周期被转移了，接下俩`myThread3`构造时用的`std::move(future)`future已经失效了，会报错，一般都是`no state`之类的错误。

### 异常处理

`std::future`是一个模板类，它用于表示一个可能还没有准备好的异步操作的结果。你可以通过调用`std::future::get()`方法来获取这个结果。如果在获取结果是发生了异常，那么`std::future::get()`会重新抛出这个异常

例子：

```c++
void may_throw() {
    throw std::runtime_error("Oops, something went wrong!");
}
void get_future_error() {
    // 创建一个异步任务
    std::future<void> result(std::async(std::launch::async , may_throw));
    try {
        result.get();
    } catch(const std::exception& e) {
        std::cerr << "Caught exception: " << e.what() << std::endl;
    }
}
```

在这个例子中，我们创建了一个异步任务`may_throw`，这个任务会抛出一个异常。然后，我们创建一个`std::future`对象`result`来表示这个任务的结果。在`get_future_error`函数中，我们调用`result.get()`来获取任务的结果。如果在获取结果时发生了异常，那么`result.get()`会重新抛出这个异常，然后我们在`catch`块中捕获了并打印这个异常。

输出：

```
Caught exception: Oops, something went wrong!
```

## 线程池

我们可以利用上面提到的`std::packaged_task`和`std::promise`构建线程池，提高程序的并发能力

线程池的知识:

线程池是一种多线程处理形式，它处理过程中将任务添加到队列，然后在创建线程后自动启动这些任务。线程池线程都是后台线程。每个线程都使用默认的堆栈大小，以默认的优先级运行，并处于多线程单元中。如果某个线程在托管代码中空闲（如正在等待某个事件）,则线程池将插入另一个辅助线程来使所有处理器保持繁忙。如果所有线程池线程都始终保持繁忙，但队列中包含挂起的工作，则线程池将在一段时间后创建另一个辅助线程但线程的数目永远不会超过最大值。超过最大值的线程可以排队，但他们要等到其他线程完成后才启动。

```c++
#pragma once

#include <iostream>
#include <future>
#include <atomic>
#include <thread>
#include <condition_variable>
#include <vector>
#include <mutex>
#include <queue>
#include <functional>

template <typename T> class TD;

class ThreadPool {
public:
    ThreadPool( const ThreadPool & )            = delete;
    ThreadPool &operator=( const ThreadPool & ) = delete;

    static ThreadPool &getInstance() {
        static ThreadPool ins;
        return ins;
    }

    using Task = std::packaged_task<void()>;

    ~ThreadPool() { stop(); }

    // NOTE: 用于将任务队列中的任务提交到pool_里面
    template <typename F, class... Args>
    auto commit( F &&func, Args &&...args )
        -> std::future<decltype( func( args... ) )> {
        using RetType = decltype( func( args... ) );

        if ( stop_.load() ) {
            // NOTE: 这里如果触发异常处理，可以通过异常处理来了解状态
            return std::future<RetType>{};
        }

        // NOTE:关于这里为什么要使用std::shared_ptr
        // 1.
        // 避免对象过早的销毁，我们通常需要在另一个线程中执行task,可能会在创建的作用域之外。
        // 2.
        // 允许对象的共享，比如说，你可以在一个线程中安排一个任务，并在另一个线程中等待该任务完成并获取其结果。
        // 这样的话，任务对象就需要在多个线程中共享，而
        // std::shared_ptr 正好可以满足这个要求。
        auto task = std::make_shared<std::packaged_task<RetType()>>(
            std::bind( std::forward<F>( func ), std::forward<Args>( args )... ) );

        std::future<RetType> ret = task->get_future();
        {
            std::lock_guard<std::mutex> cv_mt( cv_mt_ );
            tasks_.emplace( [task] { ( *task )(); } );
        }
        cv_lock_.notify_one();
        return ret;
    }

    int idleThreadCount() { return thread_num_; }

private:
    ThreadPool( unsigned int num = 5 ) : stop_( false ) {
        if ( num < 1 ) {
            thread_num_ = 1;
        } else {
            thread_num_ = num;
        }
        start();
    }

    void start() {
        for ( int i = 0; i < thread_num_; i++ ) {
            pool_.emplace_back( [this]() {
                while ( !this->stop_.load() ) {
                    Task task;
                    std::unique_lock<std::mutex> cv_mt( cv_mt_ );
                    this->cv_lock_.wait( cv_mt, [this]() {
                        // NOTE: 当stop_为true,或者任务队列不空时，线程都会醒来。
                        return this->stop_.load() || !this->tasks_.empty();
                    } );

                    if ( this->tasks_.empty() ) {
                        return;
                    }

                    task = std::move( this->tasks_.front() );
                    this->tasks_.pop();
                    // 拿到一个任务，空闲线程-1
                    this->thread_num_--;
                    task(); // NOTE: 拿到一个新任务，进行异步调用，执行任务。
                    // 任务执行完之后，空闲线程又回来了，空闲线程+1
                    this->thread_num_++;
                }
            } );
        }
    }

    void stop() {
        stop_.store( true );
        cv_lock_.notify_all();
        for ( auto &td : pool_ ) {
            if ( td.joinable() ) {
                std::cout << "join thread " << td.get_id() << std::endl;
                td.join();
            }
        }
    }

private:
    std::mutex cv_mt_;
    std::condition_variable cv_lock_;
    std::atomic_bool stop_;
    std::atomic_int thread_num_;
    std::queue<Task> tasks_;
    std::vector<std::thread> pool_;
};

```

注意：

1. 线程池做的任务是并发的、无序的，无法保证有序性
2. 如果执行的任务是强关联或者互斥性很大，建议使用单线程，线程池的意义不大

[源码链接](https://github.com/Encounter005/Notes/tree/main/C%2B%2B/src)
