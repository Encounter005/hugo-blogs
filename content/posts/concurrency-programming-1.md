+++
title = "并发编程-1"
tags = [ "C/C++", "并发编程",]
categories = [ "技术",]
date = "2024-03-23T22:43:23+08:00"
+++
# 线程基础

## 线程的创建

```c++
#include <iostream>
#include <thread>


void func() {
    std::cout << "hello world\n";
}

int main() {

    std::thread t1(func);
    t1.join();

    return 0;
}

```

如果函数需要传参

```c++
#include <iostream>
#include <thread>


void func(std::string name) {
    std::cout << name << "\n";
}

int main() {

    std::thread t1(func , "127.0.0.1");
    t1.join();

    return 0;
}

```

仿函数作为参数

```c++
class Background_task {
public:
    void operator[](const std::string& str) {
        std::cout << str << "\n";
    };
};
```

注意：

如果采用如下的方式调用函数，编译器一定会报错

```c++
    std::thread t1(Background_task());
    t1.join();
```

因为编译器会将t1当作一个函数对象，返回一个`std::thread`类型的值，函数的参数为一个函数指针，该函数指针返回值为`Back_ground_task`，参数为void

```c++
std::thread (*)(Background_task (*)());
```

修改的方式如下

```c++
    std::thread t1((background_task()));
    t1.join();
    std::thread t2{background_task()};
    t2.join();
```

lambda表达式作为参数

```c++
    std::thread t1([](const std::string& str){
        std::cout << str << "\n";
    } , "hello world\n");
    t1.join();

```

注意在线程中参数是以拷贝的形式进行传递，因此对于拷贝耗时的对象可以选择用引用或者指针进行传递。但是需要考虑对象的生命周期。因为线程的运行长度可能会超过参数的生命周期，这个时候如果线程还在访问一个已经被销毁的对象就会出现问题

### detach 和 join

> 主要API

一旦启动线程之后，我们必须决定是要等待到这个线程结束(join)，或者直接和主线程独立开来(detach)，我们必须二者选其一。如果`std::thread`对象销毁时我们还没有触发任何操作，则`std::thread`对象在析构函数将调用`std::terminate()`从而导致进程异常退出

- `join`: 调用这个API时，当前线程就会停滞，直到目标线程执行完毕。

- `detach`: 这个API是让目标线程成为守护线程。一旦`detach`之后，目标线程将独立运行，即便其对应的`std::thread`对象销毁也不影响线程的执行。并且无法与之通信。

对于这两个接口，都必须是可执行的线程才有意义。可以通过`joinable()`接口查询是否可以对它们进行`join`或者`detach`

### 管理线程

- `yield`: 通常在自己的主要任务已经完成时，希望让出处理器给其他任务使用

- `get_id`: 返回当前线程的id，可以用来标识不同的线程

- `sleep_for`: 可以让当前线程停滞一段时间

- `sleep_until`: 和`sleep_for`类似，但是以具体时间点为参数，这两个API都是以`chrono`api为基础

```c++
#include <iomanip>
#include <iostream>
#include <thread>
#include <sstream>

void print_time() {
    auto now = std::chrono::system_clock::now();
    auto in_time_t = std::chrono::system_clock::to_time_t(now);

    std::stringstream ssin;
    ssin << std::put_time(std::localtime(&in_time_t) , "%Y-%m-%d %X");
    std::cout << "now is: " << ssin.str() << "\n";

}

void sleep_thread() {
    std::this_thread::sleep_for(std::chrono::seconds(3));
    std::cout << "[thread-" << std::this_thread::get_id() << "] is waking up\n";
}

void loop_thread() {
    for(int i = 0; i < 10; i++) {
        std::cout << "[thread-" << std::this_thread::get_id() << "] print: " << i << "\n";
    }
}

int main() {
    print_time();
    std::thread t1(sleep_thread) , t2(loop_thread);
    t1.join();
    t2.detach();
    print_time();

    return 0;
}

```

### 线程的归属权

> 每个线程都有其归属权，也就是说归属给每个变量管理

```c++

void func() {

}

std::thread t1(func);

```

`t1`是一个线程变量，管理一个线程，该线程执行`func()`对于`std::thread`C++不允许拷贝构造和拷贝赋值，所以只能通过移动和局部变量返回的方式将线程变量管理权转移给其他变量管理。

例如下面的例子

```c++
#include <iostream>
#include <thread>
#include <chrono>

namespace {
void func1() {
    while(true) {
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
}
void func2() {
    while(true) {
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
}

}; // namespace
int main() {
    //1. t1绑定函数func1()
    std::thread t1(func1);
    //2. 转移t1的管理的线程给t2，转移后t1无效
    std::thread t2 = std::move(t1);
    //3. t1绑定函数func2()
    t1 = std::thread(func2);
    //4. 新建一个线程t3
    std::thread t3;
    //5. 转移t2管理的线程给t3，，
    t3 = std::move(t2);
    //6. 转移t3管理的线程给t1
    t1 = std::move(t3);
    // BUG:不可以将一个线程的管理权交给一个已经绑定线程的变量，
    //否则会触发线程的terminate函数引发崩溃
    // t1 = std::move(t3);
    std::this_thread::sleep_for(std::chrono::seconds(2000));

    return 0;
}

```

### 异常处理

当我们启动一个线程之后，如果主线程崩溃，会导致子线程也会异常退出，调用`std::terminate()`，如果自线程在进行一些重要的操作比如将充值信息入库等，丢失这些信息是很为危险的。所以常用的做法是捕获异常，并且在异常情况下保证子线程稳定运行结束后，主线程抛出异常结束运行

```c++
void catch_exception() {
    int some_local_state = 0;
    std::function<void()> myfunc = [](){
        // doing something
    };

    std::thread functhread(myfunc);
    try{
        // 本线程做的一些事情，可能引发崩溃
        std::this_thread::sleep_for(std::chrono::seconds(1));
    } catch (std::exception& e) {
        functhread.join();
        throw std::runtime_error("ababababa\n");
    }

    functhread.join();
}
```

但是用这种方式编码会显得臃肿，可以使用`RAII`技术，保证线程对象析构的时候等待线程运行结束，回收资源。

```c++
class thread_guard {
private:
    std::thread& _t;
public:
    explicit thread_guard(std::thread& t) : _t(t) {}
    ~thread_guard() {
        if(_t.joinable()) {
            _t.join();
        }
    }
    thread_guard(const thread_guard&) = delete;
    thread_guard& operator=(const thread_guard&) = delete;
};

void auto_guard() {
    int some_local_state = 0;
    std::function<void()> myfunc = [](){
        // doing something
    };

    std::thread t1(myfunc);
    thread_guard guard(t1);
    std::cout << "auto guard finish\n";
}

```

### 慎用隐式转换

C++中会有一些隐式转换，例如`char*`转换为`std::string`等，这些隐式转换在线程的调用上可能会造成崩溃问题

```c++
    void danger_oops(int som_param) {
        char buffer[1024];
        sprintf(buffer, "%i", som_param);
        // NOTE:在线程内部将char const* 转化为std::string
        // 指针常量  char const*  指针本身不能变
        // 常量指针  const char * 指向的内容不能变
        std::thread t(print_str, 3, buffer);
        t.detach();
        std::cout << "danger oops finished " << std::endl;
    }
```

在以上代码中，当我们定义一个线程变量`thread t`时，传递给这个线程参数`buffer`会被保存到`thread`的成员变量中。而在线程对象t内部启动并运行线程时，参数才会被传递给调用函数`print_str`。而此时`buffer`可能到`}`就被销毁了。改进方式很简单，我们将参数传递给`thread`时显示转换成`string`就可以了，其实相当于拷贝一份过去。

```c++
        void danger_oops(int som_param) {
        char buffer[1024];
        sprintf(buffer, "%i", som_param);
        //在线程内部将char const* 转化为std::string
        //指针常量  char const*  指针本身不能变
        //常量指针  const char * 指向的内容不能变
        std::thread t(print_str, 3, std::string(buffer));
        t.detach();
        std::cout << "danger oops finished " << std::endl;
    }

```

### 引用参数

在线程要调用的回调函数参数作为引用类型时，需要将参数显式转换为引用对象传递给构造函数，如果采用如下调用会编译失败

```c++
void change_param(int& param) {
    ++param;
}

void ref_oops(int some_param) {
    std::cout << "before change , param is " << some_param << std::endl;
    // 需要引用显式转换
    std::thread t2(change_param , some_param);
    t2.join();
    std::cout << "after change , param is " << some_param << std::endl;
}
```

即使函数`change_param`的参数为`int&`类型，我们传递给t2的构造函数为`some_param`，也不会达到在`change_param`函数内部修改关联到外部`some_param`的效果。因为`some_param`在传递给`thread`的构造函数后会转变为右值保存，右值传递给一个左值引用会出问题，所以编译也出了问题。


只需要使用`std::ref()`就可以解决

```c++

void change_param(int& param) {
    ++param;
}

void ref_oops(int some_param) {
    std::cout << "before change , param is " << some_param << std::endl;
    // 需要引用显式转换
    std::thread t2(change_param , std::ref(some_param));
    t2.join();
    std::cout << "after change , param is " << some_param << std::endl;
}

```

### 绑定类成员函数

有时候我们需要绑定一个类的成员函数。

```c++
class X {
public:
    void do_length_work() {
        std::cout << "do_length_work" << std::endl;
    }
};

void bind_class_oops() {
    X my_x;
    std::thread t(&X::do_length_work , &my_x);
    t.join();
}

```

注意，如果`thread`绑定的是普通函数，可以在函数前加`&`或者不加`&`，因为编译器默认将普通函数名作为函数地址，但是如果是绑定类的成员函数，必须加`&`

```c++
    void thead_work1(std::string str) {
        std::cout << "str is " << str << std::endl;
    }
    std::string hellostr = "hello world!";
    //两种方式都正确
    std::thread t1(thead_work1, hellostr);
    std::thread t2(&thead_work1, hellostr);
```

### 使用move操作

有时候传递给线程的参数是独占的，所谓独占就是不支持拷贝赋值和构造，但是我们可以通过`std::move()`的方式将参数的所有权转移给线程。

```c++
void deal_unqiue(std::unique_ptr<int> p) {
    std::cout << "unique ptr data is " << *p << std::endl;
    (*p)++;
    std::cout << "after unqiue ptr data is " << *p << std::endl;
}

void move_oops() {
    auto p = std::make_unique<int>(100);
    std::thread t(deal_unqiue , std::move(p));
    t.join();

}
```

### 一次调用

在一些情况下，我们有些任务需要执行一次，并且我们只希望它执行一次，例如资源的初始化人物。这个时候就可以用到`std::call_once`和`std::once_flag`。这两个接口保证即便在多线程的环境下，相应的函数也只会调用一次。

例如下面的代码，有四个线程会执行`init()`函数，但是只有一个线程会执行。

```c++
#include <iostream>
#include <thread>
#include <mutex>

void init() {
    std::cout << "Initializing...." << std::endl;
}

void worker(std::once_flag& flag) {
    std::call_once(flag, init);
}

int main() {
    std::once_flag flag;
    std::thread t1( worker, std::ref(flag) );
    std::thread t2( worker, std::ref(flag) );
    std::thread t3( worker, std::ref(flag) );
    std::thread t4( worker, std::ref(flag) );
    std::thread t5( worker, std::ref(flag) );
    t1.join();
    t2.join();
    t3.join();
    t4.join();
    t5.join();
    return 0;
}
```

注意，我们无法确定哪个线程执行了`init()`函数，但是我们也不需要关心，因为只要有一个线程执行了`init()`函数就可以了

### 并发任务

实例：需要计算某个范围内所有数的平方根总和

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <cmath>
#include <chrono>

static int MAX = 10e8;
static double sum = 0;

void func(int min , int max) {
    for(int i = min ; i <= max; i++) {
        sum += sqrt(i);
    }
}

void task(int min , int max) {
    // 得到调用func之前的时间
    auto start_time = std::chrono::steady_clock::now();
    sum = 0;
    func(min , max);
    // 得到调用后func之后的时间
    auto end_time = std::chrono::steady_clock::now();
    // 得到持续的时间
    auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(end_time - start_time).count();
    std::cout << "Task finish, " << ms << " ms consumed , Result: " << sum << "\n";
}

int main() {
    task(1 , MAX);

    return 0;
}

```

输出为

```
Task finish, 1821 ms consumed , Result: 2.10819e+13
```

很明显上面的计算使用单线程性能太差，可以使用并发进行。

1. 先获取当前硬件支持多少个线程并行执行
2. 根据处理器情况决定线程数量
3. 对于每一个线程都通过`func()`函数来完成任务，并划分一部分数据给它处理
4. 等待每一个线程结束。

```c++
void concurrent_task(int min , int max) {
    auto start_time = std::chrono::steady_clock::nowxainshi();
    // 得到线程数量
    unsigned int concurrent_count = std::thread::hardware_concurrency();
    std::cout << "hardware_concurrency: " << concurrent_count << "\n";
    std::vector<std::thread> threads;
    min = 0;
    sum = 0;
    for(int t = 0; t < concurrent_count; t++) {
        int range = max / concurrent_count * (t + 1);
        threads.push_back(std::thread(func , min , range));
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

输出如下

```
hardware_concurrency: 20
Task finish, 2086 ms consumed , Result: 1.53287e+12
```

但是我们会发现性能并没有多少提升，并且结果还是错的。

要搞清楚为什么，我们需要一点背景知识

对于现代处理器来说，为了加速处理的速度，每个处理器都会有自己的高速缓存，这个高速缓存是与每个处理相对应的

> 现在的处理器起码有三级缓存

![OgtpNY.png](https://ooo.0x0.ooo/2024/03/23/OgtpNY.png)

处理器在进行计算的时候，高速缓存会参与其中，例如数据的读和写。而高速缓存和系统主存（Memory）是有可能存在不一致的。即：某个结果计算后保存在处理器的高速缓存中了，但是没有同步到主存中，此时这个值对于其他处理器就是不可见的。

事情还没有这么简单。我们对于全局变量值的修改`sum += sqrt(i);`这条语句，它并非是原子的。它其实很多条指令的组合才完成的。假设在某个设备上，这条语句通过下面这几个步骤来完成的，时序可能如下：

![OgtJEv.png](https://ooo.0x0.ooo/2024/03/23/OgtJEv.png)

如图所示，在时间点a的时候，所有线程对于`sum`变量的值是一致的。

但是在时间点b之后，thread3上已经对`sum`进行了赋值，而这个时候其他几个线程也同时在其他处理器上使用了这个值，那么这个时候它们所使用的值是旧的，也就是错误的。最后得到的结果也就是错误的。

### 竞争条件与临界区

当多个进程或者线程同时访问共享数据时，只要有一个任务会修改数据，那么就可能会发生问题。此时结果依赖于这些任务执行的相对时间，这种场景称为[竞争条件](https://zhuanlan.zhihu.com/p/426072739)

访问共享数据的代码片段称之为**临界区**。例如上面的实例，临界区就是读写`sum`变量的地方

要避免竞争条件，就需要对临界区进行数据保护。

那么对于上面的实例的解决方法就是一次只让一个线程访问共享数据，访问完了再让其他线程接着访问，这样就可以避免问题发生了。

