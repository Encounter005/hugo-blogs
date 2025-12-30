+++
title = "并发编程-2"
tags = [ "C/C++", "并发编程",]
categories = [ "技术",]
date = "2024-03-23T22:49:37+08:00"
+++
# 互斥体与锁

## mutex

开发并发系统的目的主要是为了提升性能：将任务分散到多个线程，然后在不同的处理器上同时执行。这些分散开来的线程通常会包含两类任务：

1. 独立的对于划分给自己的数据进行处理
2. 对于结果的汇总

其中第一项任务由于每个线程都是独立的，不存在竞争条件的问题。而第二个任务，由于所有的线程都可能往总结果汇总，这就需要做保护了。
在某一个具体的时刻，只应当有一个线程更新结果，即：保证每个线程对于共享数据的访问是"互斥的"，`mutex`就提供了这样的功能。

| 方法     | 说明                                      |
| -------- | ----------------------------------------- |
| lock     | 加锁，如果不可用，则阻塞                  |
| try_lock | 尝试加锁，如果mutex不可以用直接返回(bool) |
| unlock   | 解开互斥锁                                |

这三个方法提供了基础的锁定和解除锁定的功能。使用`lock`意味着你有很强的意愿一定要获取到互斥体，而使用`try_lock`则是进行一次尝试。这意味着如果失败了，你通常还有其他的路径可以走。

在这些基础功能之上，其他的类分别在下面三个方面进行了扩展

- 超时： `timed_mutex`, `recursive_timed_mutex`, `shared_timed_mutex`的名称都带有`timed`,这意味着它们都支持超时的功能。它们都提供了`try_lock_for`和`try_lock_until`方法，这两个方法分别可以制定超时的时间长度和时间点。如果在超时的时间范围内没有能获取到锁，则直接返回，不再继续等待。

- 可重入：`rescursive_mutex`和`recursive_timed_mutex`的名称都带有`rescursive`。可重入或者叫做可递归，是指在同一个线程中，同一把锁可以锁定多次。这就避免了一些不必要的死锁。

- 共享：`shared_timed_mutex`和`shared_mutex`提供了共享功能。对于这类互斥体，实际上是提供了两把锁：一把是共享锁、一把是互斥锁。一旦某个线程获取了互斥锁，任何其他线程都无法再获取互斥锁和共享锁;但是如果有某个线程获取到了共享锁，其他线程无法再获取到互斥锁，但是还有获取到共享锁。这里互斥锁的使用和其他互斥体接口功能一样。而共享锁可以被同时多个线程获取到。共享锁通常用在[读者写者模型](https://zhuanlan.zhihu.com/p/189993251)上

使用共享锁的接口如下：

| 方法            | 说明                                   |
| --------------- | -------------------------------------- |
| lock_shared     | 获取互斥体的共享锁，如果无法获取则阻塞 |
| try_lock_shared | 尝试获取共享锁，如果不可用，直接返回   |
| unlock_shared   | 解锁共享锁                             |

接下里对之前的代码进行改造

```c++
// 初始化一个互斥锁，开全局
static std::mutex lock1;
void concurrent_func(int min , int max) {
    double tmp_sum = 0;
    for(int i = min ; i <= max; i++) {
        tmp_sum += sqrt(i);
    }
    lock1.lock();
    sum += tmp_sum;
    lock1.unlock();
}

void concurrent_task(int min , int max) {
    auto start_time = std::chrono::steady_clock::now();
    // 得到线程数量
    unsigned int concurrent_count = std::thread::hardware_concurrency();
    std::cout << "hardware_concurrency: " << concurrent_count << "\n";
    std::vector<std::thread> threads;
    min = 0;
    sum = 0;
    for(int t = 0; t < concurrent_count; t++) {
        int range = max / concurrent_count * (t + 1);
        threads.push_back(std::thread(concurrent_func , min , range));
        min = range + 1;
    }

    for(auto &i : threads) {
        i.join();
    }
    auto end_time = std::chrono::steady_clock::now();
    auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(end_time - start_time).count();
    std::cout << "Task finish, " << ms << " ms consumed , Result: " << sum << "\n";
}

```

这里有两个地方需要关注：

1. 在访问共享数据之前加锁
2. 在访问完之后解锁

输出如下：

```
hardware_concurrency: 20
Task finish, 201 ms consumed , Result: 2.10819e+13
```

通过多线程实现并行求容器和

```c++
template<typename Iterator , typename T>
void get_sum(Iterator fst , Iterator lst , T& res) {
    static std::mutex g1;
    auto tmp_res = std::accumulate(fst , lst , T{});
    std::lock_guard<std::mutex> g(g1);
    res += tmp_res;
}

template <typename Iterator, typename T>
T parallel_accumulate( Iterator fst, Iterator lst, T init ) {
    auto res = init;
    unsigned long const  min_thread = 2;
    unsigned long const  hardware_threads = std::thread::hardware_concurrency();
    unsigned long const  length = std::distance(fst , lst);
    unsigned long const  max_thread = (length + min_thread - 1) / min_thread;
    unsigned long const num_threads = std::min(hardware_threads ? hardware_threads : min_thread , max_thread );
    unsigned long const dis = length / num_threads;

    std::vector<std::thread> threads(num_threads);

    Iterator st = fst;

    for(auto& th : threads) {
        Iterator ed = st;
        std::advance(ed , dis);
        th = std::thread(get_sum<Iterator , T> , st , ed , std::ref(res));
        th.join();
        st = ed;
    }

    return res;
}

void use_parallel_acc() {
    std::vector<int> vec;
    for ( int i = 0; i < 100000; i++ ) {
        vec.push_back( i );
    }

    int sum = 0;
    sum     = parallel_accumulate<std::vector<int>::iterator, int>(
        vec.begin(), vec.end(), sum );
    std::cout << "sum is " << sum << "\n";
}

```

我们通常用锁的粒度来描述锁的范围。**细粒度**是指锁保护较小的范围，**粗粒度**是指保护较大的范围。出于性能的考虑，我们应该保证锁的粒度尽可能的细。并且，不应该在获取锁的范围内执行耗时的操作，例如执行IO。如果是耗时的运算也应该尽可能的移动到锁的外边。

## 锁的使用

1. mutex

我们可以通过`mutex`对共享数据进行加锁，防止多线程访问共享区造成数据不一致问题。如下，我们初始化一个共享变量`shared_data`,然后定义了一个互斥量`std::mutex`，接下来启动了两个线程，分别执行`use_lock`增加数据，和一个lambda表达式减少数据。结果可以看到两个线程对于共享数据的访问是独占的，单位时间片只有一个线程访问并输出日志

```c++
std::mutex mtx1;
int shared_data = 100;

void use_lock() {
    while(true) {
        mtx1.lock();
        shared_data++;
        std::cout << "current thread is " << std::this_thread::get_id() << "\n";
        std::cout << "shared_data is " << shared_data << "\n";
        mtx1.unlock();
        std::this_thread::sleep_for(std::chrono::microseconds(10));
    }
}

void test_lock() {
    std::thread t1(use_lock);
    std::thread t2([](){
        mtx1.lock();
        shared_data--;
        std::cout << "current thread is " << std::this_thread::get_id() << "\n";
        std::cout << "shared_data is " << shared_data << "\n";
        mtx1.unlock();
        std::this_thread::sleep_for(std::chrono::microseconds(10));
    });

    t1.join();
    t2.join();
}
```

2. lock_guard
   > lock_guard可以自动加锁和解锁

```c++
void use_lock() {
    while(true) {
        std::lock_guard<std::mutex> g(mtx1);
        shared_data++;
        std::cout << "current thread is " << std::this_thread::get_id() << "\n";
        std::cout << "shared_data is " << shared_data << "\n";
        std::this_thread::sleep_for(std::chrono::microseconds(10));
    }
}
```

`lock_guard`在作用域结束时自动调用其析构函数解锁，这么做的一个好处是简化了一些特殊情况从函数返回的写法，比如异常或者条件不满足时，函数内部直接return,锁也会自动解开。

3. 如何保证数据安全

有时候我们可以对共享数据的访问和修改聚合到一个函数，在函数内部加锁保证数据的安全性。但是对于读取类型的操作，即使读取函数是线程安全的，但是返回值抛给外边使用，存在不安全性。比如一个栈对象，我们要保证其在多线程访问的时候是安全的，可以在判断栈是否为空，判断操作内部我们可以加锁，但是判断结束后返回值就不加锁了，就会存在线程安全问题

比如定义了如下栈，对于多线程的访问时判断栈是否为空，此后两个线程同时出栈，可能会造成崩溃

```c++
template<typename T>
class threadsafe_stack1 {
private:
    std::stack<T> data;
    mutable std::mutex m;
public:
    threadsafe_stack1() {}
    threadsafe_stack1(const threadsafe_stack1& other) {
        std::lock_guard<std::mutex> Lock(m);
        data = other.data;
    }
    threadsafe_stack1& operator=(const threadsafe_stack1& other) = delete;
    void push(T new_value) {
        std::lock_guard<std::mutex> Lock(m);
        data.push(std::move(new_value));
    }

    // 问题代码
    T pop() {
        std::lock_guard<std::mutex> Lock(m);
        auto element = data.top();
        data.pop();
        return element;
    }

    bool empty() const {
        return data.empty();
    }
};
```

如下，线程1和线程2先后判断都不为空，之后执行出栈，会造成崩溃

```c++
void test_threadsafe_stack1() {
    threadsafe_stack1<int> safe_stack;
    safe_stack.push(1);
    std::thread t1([&safe_stack](){
        std::this_thread::sleep_for(std::chrono::seconds(1));
        safe_stack.pop();
    });

    std::thread t2([&safe_stack](){
        if(!safe_stack.empty()) {
            std::this_thread::sleep_for(std::chrono::seconds(1));
            safe_stack.pop();
        }
    });
    t1.join();
    t2.join();
}
```

解决这个问题我们可以用抛出异常函数，例如定义一个空栈的异常

```c++
struct empty_stack : std::exception {
    const char *what() const throw();
};
```

然后修改出栈函数

```c++
    T pop() {
        std::lock_guard<std::mutex> Lock(m);
        if(data.empty()) {
            throw empty_stack();
        }
        auto element = data.top();
        data.pop();
        return element;
    }
```

这么做就需要在外层使用的时候捕获异常。这是C++ 并发编程中提及的建议。但是现在这个`pop`函数仍存在问题，比如`T`是一个`vector<int>`类型时，那么在`pop`函数内部`element`就是`vector<int>`类型，开始`element`存储了一些int值，程序没问题，函数执行了`pop`操作，假设此时程序内存暴增，导致当前程序使用的内存足够大时，可用的有效空间不够，函数返回`element`时，就会存在`vector`做拷贝赋值时造成失败。即使我们捕获异常，释放部分空间但也会导致栈元素已经出栈，数据丢失了。这其实是内存管理不当造成的，但书中给了优化方案

```c++
template<typename T>
class threadsafe_stack1 {
private:
    std::stack<T> data;
    mutable std::mutex m;
public:
    threadsafe_stack1() {}
    threadsafe_stack1(const threadsafe_stack1& other) {
        std::lock_guard<std::mutex> Lock(m);
        data = other.data;
    }
    threadsafe_stack1& operator=(const threadsafe_stack1& other) = delete;
    void push(T new_value) {
        std::lock_guard<std::mutex> Lock(m);
        data.push(std::move(new_value));
    }

    std::shared_ptr<T> pop() {
        std::lock_guard<std::mutex> Lock(m);
        // NOTE:1.试图弹出前检查栈是否为空
        if(data.empty()) return nullptr;
        // NOTE:2.改动栈容器前设置返回值
        std::shared_ptr<T> const res(std::make_shared<T>(data.top()));
        data.pop();
        return res;

    }

    void pop(T& value) {
        std::lock_guard<std::mutex> Lock(m);
        if(data.empty()) {
            throw empty_stack();
        }
        value = data.top();
        data.pop();
    }

    bool empty() const {
        return data.empty();
    }
};
```

我们提供了两个版本的pop操作，一个是带引用类型参数的，一个是直接pop出智能指针类型，这样在`pop`函数内部减少了数据的拷贝，防止内存溢出，其实这两种做法确实是相比之前直接`pop`固定类修能够的值更节省内存，运行效率也好很多。我们也完全可以基于之前的思想，在pop时如果栈为空返回空指针，这样比抛出异常好些

## 死锁

### 死锁是如何造成的

死锁一般是调用顺序不一致而导致的。例如两个线程循环调用。当线程1先加锁A,在加锁B,而线程而先加锁B,在加锁A。那么在某一时刻就可能造成一种情况，线程1先加锁了A,线程2先加锁了B,那么他们都希望彼此占有对方的锁，又不释放自己占有的锁

```c++
std::mutex lock1 , lock2;
int m_1 = 0 , m_2 = 1;
void dead_lock1() {
    while(true) {
        std::cout << "DeadLock1 Begin\n";
        lock1.lock();
        m_1 = 1024;
        lock2.lock();
        m_2 = 2048;
        lock2.unlock();
        lock1.unlock();
        std::this_thread::sleep_for(std::chrono::milliseconds(5));
        std::cout << "DeadLock1 End\n";
    }
}

void dead_lock2() {
    while(true) {
        std::cout << "DeadLock2 Begin\n";
        lock2.lock();
        m_2 = 2222;
        lock1.lock();
        m_1 = 1111;
        lock1.unlock();
        lock2.unlock();
        std::this_thread::sleep_for(std::chrono::milliseconds(5));
        std::cout << "DeadLock2 End\n";
    }
}

void call_dead_lock() {
    std::thread t1(dead_lock1) , t2(dead_lock2);
    t1.join() , t2.join();
}
```

这样运行之后在某一个时刻一定会导致死锁。实际工作中避免死锁的一个方式就是将加锁和解锁的功能封装为独立函数，这样能保证独立的函数里执行完操作之后就解锁，不会导致一个函数里面使用多个锁的情况。

```c++

void atomic_lock1() {
    std::cout << "AtomicLock1 Begin\n";
    lock1.lock();
    m_1 = 1024;
    lock1.unlock();
    std::cout << "AtomicLock1 End\n";
}

void atomic_lock2() {
    std::cout << "AtomicLock2 Begin\n";
    lock2.lock();
    m_2 = 2048;
    lock2.unlock();
    std::cout << "AtomicLock2 End\n";
}

void safe_lock1() {
    while(true) {
        atomic_lock1();
        atomic_lock2();
        std::this_thread::sleep_for(std::chrono::milliseconds(5));
    }
}
void safe_lock2() {
    while(true) {
        atomic_lock2();
        atomic_lock1();
        std::this_thread::sleep_for(std::chrono::milliseconds(5));
    }
}

void call_safe_lock() {
    std::thread t1(safe_lock1) , t2(safe_lock2);
    t1.join() , t2.join();
}
```

### 同时加锁

当我们无法避免在一个函数内部使用两个互斥量，并且都要解锁的情况下，闹我们可以采取同时加锁的方式，我们先定义一个类，假设这个类不推荐拷贝构造，但我们也提供了这个类的拷贝构造和移动构造。

```c++
class some_big_object {
public:
    some_big_object() = default;
    some_big_object(int item) : _data(item) {}
    some_big_object(const some_big_object& other) : _data(other._data) {}
    some_big_object(some_big_object&& other) : _data(std::move(other._data)) {}
    some_big_object& operator=(const some_big_object& other) {
        if(_data == other._data) {
            return *this;
        }
        _data = other._data;
        return *this;
    }
    friend std::ostream& operator<<(std::ostream& os , const some_big_object& soj) {
        os << soj._data;
        return os;
    }

    friend void swap(some_big_object& o1 , some_big_object& o2) {
        some_big_object temp = std::move(o1);
        o1 = std::move(o2);
        o2 = std::move(temp);

    }

private:
    int _data;
};
```

接下来再定义一个类对上面的类进行管理，为防止多线程情况下数据混乱，包含了一个互斥量。

```c++
class big_obj_manager {
public:
    big_obj_manager(int data = 0) : sbo(data) {}
    void printinfo() {
        std::cout << "The current data is: " << sbo << "\n";
    }
    friend void danger_swap(big_obj_manager& bo1 , big_obj_manager& bo2);
    friend void safe_swap(big_obj_manager& bo1 , big_obj_manager& bo2);
    friend void safe_swap_scope(big_obj_manager& bo1 , big_obj_manager& bo2);

private:
    some_big_object sbo;
    std::mutex mtx;
};
```

为了演示哪些交换是安全的，哪些交换是危险的，所以写了三个函数

```c++
void danger_swap(big_obj_manager& bo1 , big_obj_manager& bo2) {
    std::cout << "thread[" << std::this_thread::get_id() << "] begin \n";
    if(&bo1 == &bo2) {
        return;
    }
    std::lock_guard<std::mutex> g1(bo1.mtx);
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::lock_guard<std::mutex> g2(bo2.mtx);
    swap(bo1.sbo , bo2.sbo);
    std::cout << "thread[" << std::this_thread::get_id() << "] end \n";
}

```

`danger_swap`是危险的交换方式，例如如下调用

```c++
void test_lock() {
    big_obj_manager o1(1) , o2(2);
    std::thread t1(danger_swap , std::ref(o1) , std::ref(o2)) , t2(danger_swap , std::ref(o1) , std::ref(o2));
    t1.join();
    t2.join();

    o1.printinfo();
    o2.printinfo();
}

```

这种调用方式存在隐患，因为`danger_swap`函数在两个线程中使用会造成竞争互相加锁的情况。那就需要用锁同时锁住两个锁。

```c++

void safe_swap(big_obj_manager& bo1 , big_obj_manager& bo2) {
    std::cout << "thread[" << std::this_thread::get_id() << "] begin \n";
    if(&bo1 == &bo2) {
        return;
    }
    // NOTE: 同时锁住两个以上的锁（最少两个）
    std::lock(bo1.mtx , bo2.mtx);
    std::lock_guard<std::mutex> g1(bo1.mtx , std::adopt_lock);
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::lock_guard<std::mutex> g2(bo2.mtx , std::adopt_lock);

    swap(bo1.sbo , bo2.sbo);

    std::cout << "thread[" << std::this_thread::get_id() << "] end \n";
}
void test_lock() {
    big_obj_manager o1(1) , o2(2);
    std::thread t1(safe_swap , std::ref(o1) , std::ref(o2)) , t2(danger_swap , std::ref(o1) , std::ref(o2));
    t1.join();
    t2.join();

    o1.printinfo();
    o2.printinfo();
}
```

上面的加锁方式可以简化，C++17`scoped_lock`可以对多个互斥两同时加锁，同时释放

```c++
void safe_swap_scope(big_obj_manager& bo1 , big_obj_manager& bo2) {
    std::cout << "thread[" << std::this_thread::get_id() << "] begin \n";
    if(&bo1 == &bo2) {
        return;
    }
    std::scoped_lock guard(bo1.mtx , bo2.mtx);
    // 等价于
    // std::scoped_lock<std::mutex , std::mutex> guard(bo1.mtx , bo2.mtx);

    swap(bo1.sbo , bo2.sbo);
    std::cout << "thread[" << std::this_thread::get_id() << "] end \n";
}
```

### 层级锁

现实开发中常常很难规避同一个函数内部多个加锁的情况，我们要尽可能避免循环加锁，所以可以自定义一个层级锁，保证实际项目中对多个互斥量加锁时是有序的。

```c++
#include <thread>
#include <climits>
#include <mutex>
#include <thread>
#include <exception>
#include <stdexcept>

class hierarchial_mutex{
public:
    explicit hierarchial_mutex(unsigned long value) : _hierarchy_value(value) , _previous_hierarchy_value(0) {}
    hierarchial_mutex(const hierarchial_mutex&) = delete;
    hierarchial_mutex& operator=(const hierarchial_mutex&) = delete;

    // @brief: 加锁函数，先检查是否违反当前层级值，如果没有，可以加锁并且更新线程的层级值
    void lock() {
        check_for_hierarchy_violation();
        _internal_mutex.lock();
        update_hierarchy_value();
    }

    // @brief: 解锁函数, 如果线程的层级值和当前层级值不一致，就抛出异常，否则先更新线程的层级为上一层的层级值，再解锁
    void unlock() {
        if(_this_thread_hierarchy_value != _hierarchy_value) {
            throw std::logic_error("mutex hierarchy violated");
        }
        _this_thread_hierarchy_value = _previous_hierarchy_value;
        _internal_mutex.unlock();
    }

    // @brief: 尝试加锁函数，先检查是否违反当前层级值，再检查锁是否可以加，最后在更新线程的层级值
    bool trylock() {
        check_for_hierarchy_violation();
        if(!_internal_mutex.try_lock()) {
            return false;
        }
        update_hierarchy_value();
        return true;
    }

private:
    std::mutex _internal_mutex;
    // NOTE: 当前层级值
    unsigned long const _hierarchy_value;
    // NOTE: 上一次层级值
    unsigned long _previous_hierarchy_value;
    // NOTE: 本线程记录的层级值
    static thread_local unsigned long _this_thread_hierarchy_value;
    // NOTE: thread_local 关键字修饰的变量具有线程周期，这些变量在线程开始时被生成，在线程结束时被销毁，并且每一个线程都拥有一个独立的变量实例

    // @brief: 检查当前线程是否违反当前层级
    void check_for_hierarchy_violation() {
        if(_this_thread_hierarchy_value <= _hierarchy_value) {
            throw std::logic_error("mutex hierarchy violated");
        }
    }

    // @brief: 更新线程的层级值
    void update_hierarchy_value() {
        _previous_hierarchy_value = _this_thread_hierarchy_value;
        _this_thread_hierarchy_value = _hierarchy_value;
    }

};

// 初始化静态成员变量
thread_local unsigned long hierarchial_mutex::_this_thread_hierarchy_value = ULONG_MAX;
```

层级锁能保证我们每个线程加锁时，一定是先加权重最高的锁(不然会抛出异常)，并且释放时也保证了顺序。  
主要原理就是将当前锁的权重保存在变量中，这样该线程再次加锁时判断线程变量的权重和锁的权重是否大于，如果满足条件就继续加锁。

### C++ unique_lock,共享锁和递归锁

> 介绍了 `unique_lock`、`shared_lock`、`recursive_lock`，其中`shared_lock`和`unique_lock`比较常用，而`recursive_lock`用得不多，尽可能规避使用这个锁

#### unique_lock

`unique_lock`和`lock_guard`用法基本一致，构造时默认加锁，析构时默认解锁，但`unique_lock`有个好处就是可以手动解锁，这一点尤为重要，方便我们控制锁区域的粒度，也能支持和条件变量配套使用。

```c++
// unique_lock的基本用法
std::mutex mtx;
int shared_data = 0;

void use_unique() {
    std::unique_lock<std::mutex> lock(mtx);
    std::cout << "Lock success \n";
    shared_data++;
    lock.unlock();
}
```

我们可以通过`unique_lock`的`owns_lock`判断是否持有锁

```c++
void owns_lock() {
    std::unique_lock<std::mutex> lock(mtx);
    shared_data++;
    if(lock.owns_lock()) {
        std::cout << "owns_lock\n";
    } else {
        std::cout << "doesn't own lock\n";
    }

    lock.unlock();
    if(lock.owns_lock()) {
        std::cout << "owns_lock\n";
    } else {
        std::cout << "doesn't own lock\n";
    }
}
```

`unqiue_lock`可以延时加锁

```c++
void defer_lock() {
    // 延迟加锁
    std::unique_lock<std::mutex> lock(mtx , std::defer_lock);
    // 可以加锁
    lock.lock();
    // 可以自动析构解锁，也可以手动解锁
    lock.unlock();
}
```

综合运用`owns_lock`和`defer_lock`

```c++
void use_own_defer() {
    std::unique_lock<std::mutex> lock( mtx );

    if ( lock.owns_lock() ) {
        std::cout << "Main thread has the lock\n";
    } else {
        std::cout << "Main thread doesn't have the lock";
    }

    std::thread t1( []() {
        std::unique_lock<std::mutex> lock( mtx );

        if ( lock.owns_lock() ) {
            std::cout << "The thread has the lock\n";
        } else {
            std::cout << "The thread doesn't have the lock";
        }
        lock.lock();
        if ( lock.owns_lock() ) {
            std::cout << "The thread has the lock\n";
        } else {
            std::cout << "The thread doesn't have the lock";
        }

        lock.unlock();
    } );

    t1.join();
}
```

上述代码会一次输出，但是程序会阻塞，因为子线程会卡在加锁的逻辑上，因为主线程为释放锁，而主线程又等待子线程推出，导致整个程序卡住

和`lock_guard`一样，`unique_lock`也支持领养锁

```c++
void use_own_adopt() {
    mtx.lock();
    std::unique_lock<std::mutex> lock(mtx , std::adopt_lock);
    if ( lock.owns_lock() ) {
        std::cout << "owns_lock\n";
    } else {
        std::cout << "doesn't own lock\n";
    }
}
```

经管是领养的，但是打印还是会出现`owns_lock`,因为不管如何锁被加上，就会输出`owns_lock`  
既然`unique_lock`支持领养操作也支持延迟加锁，那么可以用两种方式实现`lock_guard`实现的`swap`操作

```c++
int a = 10 , b = 99;

std::mutex mtx1 , mtx2;

void safe_swap() {
    std::lock(mtx1 , mtx2);
    std::unique_lock<std::mutex> lock1(mtx1 , std::adopt_lock) , lock2(mtx2 , std::adopt_lock);
    std::swap(a , b);

    // FIX: 错误用法
    // lock1.unlock();
    // lock2.unlock();
}

void safe_swap2() {
    std::unique_lock<std::mutex> lock1(mtx1 , std::defer_lock) , lock2(mtx2 , std::defer_lock);
    // lock1 , lock2加锁
    std::lock(lock1 , lock2);
    // FIX: 错误用法
    // std::lock(mtx1 , mtx2);
}
```

**注意**， 一旦`mutex`被`unique_lock`管理，加锁和释放操作就交给`unique_lock`，不能调用`mutex`加锁和解锁，因为锁的使用权已经交给了`unique_lock`

我们知道`mutex`是不支持拷贝和构造的，但是`unique_lock`支持移动，当一个`mutex`被转移给`unique_lock`后，可以通过`unique_ptr`转移其归属权

```c++
std::mutex mtx1 , mtx2 , mtx;
int shared_data = 0;

// 转移互斥量所有权
// 互斥量本身不支持move操作，但是unique_lock支持

std::unique_lock<std::mutex> get_lock() {
    std::unique_lock<std::mutex> lock(mtx);
    shared_data++;
    return lock;
}

void use_return() {
    std::unique_lock<std::mutex> lock(get_lock());
    shared_data++;
}
```

锁的粒度表示加锁的精细程度，一个锁的粒度要足够大，保证可以锁住要访问的共享数据。同时一个锁的粒度要足够小，保证非共享数据不被锁住影响性能，而`unique_ptr`则很好支持手动解锁

```c++
void precision_lock() {
    std::unique_lock<std::mutex> lock(mtx);
    shared_data++;
    lock.unlock();
    // NOTE:不涉及共享数据的耗时操作不要放在锁内执行
    std::this_thread::sleep_for(std::chrono::seconds(1));
    lock.lock();
    shared_data++;
}
```

#### 共享锁

试想这样一个场景，对于一个DNS服务，我们可以根据域名查询服务对应的ip地址，它很久才更新一次，比如新增记录，删除记录或者更新记录等。平时大部分时间都是提供给外部查询，对于查询操作，即使多个线程并发查询不加锁也不会有问题，但是当有线程修改DNS服务的ip记录或者增减记录时，其他线程不能查询，需等待修改完再查询。或者等待查询完，线程才能修改。也就是说读操作并不是互斥的，同一时间可以有多个线程同时读，但是写和读是互斥的，写与写是互斥的，简而言之，写操作需要独占锁。而读操作需要共享锁。

要想使用共享锁，需要使用互斥量`std::shared_mutex` , `std::shared_mutex`是C++17标准提出的， C++14标准可以使用`std::shared_timed_mutex`

`std::shared_mutex`和`std::shared_timed_mutex`都是用于实现多线程并发访问共享数据的互斥锁，但它们之间存在一些区别

1. `std::shared_mutex`

```
1.* 提供了lock()和try_lock_for()以及`try_lock_until`函数，这些函数都可以用于获取互斥锁。
2.* 提供了try_lock_shared()和lock_shared()函数，这些函数可以用于获取共享锁
3.* 当std::shared_mutex被锁定之后，其他尝试获取该锁的线程将会被阻塞，直到该锁被解锁。


```

2. `std::shared_timed_mutex`

```
1.* 与std::shared_mutex类似，也提供了lock() , try_lock_for() , try_lock_until() 函数用于获取互斥锁。
2.* 与std::shared_mutex不同的是，它还提供了try_lock_shared()和lock_shared()函数用于获取共享锁，这些函数在尝试获取共享锁时具有超时机制。
3.* 当std::shared_timed_mutex被锁定之后，其他尝试获取该锁的线程将会被阻塞，直到该锁被解锁，这与std::shared_mutex相同。然而，当尝试获取共享锁时，如果不能立即获得锁，std::shared_timed_mutex会设置一个超时，超时过后如果仍然没有获取到锁，则操作将返回失败
```

C++11标准没有共享互斥量，可以使用boost提供的`boost::shared_mutex`

如果我们想构造共享锁，可以使用`std::shared_lock`，如果我们想构造独占锁，可以使用`std::lock_guard`

我们可以用一个类`DNService`表示DNS服务，查询操作使用共享锁，写操作使用独占锁

```c++
class DNService {
public:
    DNService() = default;
    //读操作使用共享锁
    std::string QueryDNS(const std::string& dnsname) {
        std::shared_lock<std::shared_mutex> shared_locks(_shared_mutex);
        auto iter = _dns_info.find(dnsname);
        if(iter != _dns_info.end()) {
            return iter->second;
        }
        return "";
    }

    //写操作使用独占锁
    void AddDNSInfo(const std::string& dnsname , const std::string& dnsentry) {
        std::lock_guard<std::shared_mutex> guard_locks(_shared_mutex);
        _dns_info.insert(std::make_pair(dnsname , dnsentry));
    }

private:
    std::map<std::string , std::string> _dns_info;
    mutable std::shared_mutex _shared_mutex;
};

```

`QueryDNS`用来查询dns信息，多个线程可以同时访问
`AddDNSInfo`用来写入dns信息，属于独占锁，同一时刻只有一个线程在修改

#### 递归锁

有时候我们在实现接口的时候内部加锁，接口内部调用结束自动解锁。会出现一个接口调用另一个接口的情况。如果用普通的`std::mutex`就会出现卡死，因为嵌套加锁导致卡死，我们可以使用递归锁。

但可以从设计源头规避嵌套加锁的情况，我们可以将接口相同的功能抽象出来，统一加锁。

```c++
class RecursiveDemo{
public:
    RecursiveDemo() = default;
    bool QueryStudent(const std::string &name) {
        std::lock_guard<std::recursive_mutex> recursive_lock(_recursive_mutex);
        auto iter = _student_info.find(name);
        if(iter == _student_info.end()) {
            return false;
        }
        return true;
    }

    void AddScore(const std::string &name , const int score) {
        std::lock_guard<std::recursive_mutex> recursive_lock(_recursive_mutex);
        if(!QueryStudent(name)) {
            _student_info[name] = score;
        } else {
            _student_info[name] = _student_info[name] + score;
        }
    }

    // NOTE: 不推荐采用递归锁，使用递归锁说明设计思路并不理想，需要优化设计
    // NOTE: 推荐拆分逻辑，将共有逻辑拆分为同一接口

    void AddScoreAtomic(const std::string& name , const int score) {
        std::lock_guard<std::recursive_mutex> recursive_lock(_recursive_mutex);
        auto iter = _student_info.find(name);
        if(iter == _student_info.end()) {
            _student_info.insert(std::make_pair(name , score));
        } else {
            _student_info[name] = _student_info[name] + score;
        }
    }
private:
    std::map<std::string , int> _student_info;
    std::recursive_mutex _recursive_mutex;
    // NOTE: recursive_mutex可以在同一个线程被锁多次
};
```

我们可以看到`AddScore`函数内部调用了`QueryStudent`，所以采用了递归锁，但是我们同样可以改变设计，将两者共有的部分抽离出来生成一个新的接口`AddScoreAtomic`， `AddScoreAtomic`可以不适用于递归锁，照样能完成线程安全的操作。


