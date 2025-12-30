+++
title = "单例模式"
tags = [ "设计模式",]
categories = [ "技术",]
date = "2024-07-13T00:57:35+08:00"
+++
## 介绍

单例对象的类必须保证只有一个实例存在。许多时候整个系统只需要拥有一个全局对象，这样有利于我们调整系统行为

## 实现思路

- 一个类能返回一个对象的引用(永远都是同一个)和一个获得该实例的方法(必须是静态方法) `getInstance`
- 调用这个方法时，如果类持有的引用不为空就返回这个引用。如果类保持的引用为空，就创建该类的实例返回这个实例的引用
- 将该类的构造函数定义为私有方法，这样其他处的代码就无法通过调用该类的构造函数来实例化该对象，只有通过该类的静态方法来实现唯一实例。

## 实现方式

1. 懒汉式

> 这种lazyloading很明显，不要求线程安全，在多线程不能正常工作

| 是否lazy初始化 | 是否多线程安全 | 实现难度 |
| -------------- | -------------- | -------- |
| 是             | 否             | 易       |


实现：局部静态变量方式

```c++
class SingleLazy{
private:
    SingleLazy() {

    }
    SingleLazy(const SingleLazy&) = delete;
    SingleLazy& operator=(const SingleLazy&) = delete;
public:
    static SingleLazy& getInstance() {
        static SingleLazy single;
        return single;
    }
};

void test_SingleLazy() {
    // 多线程可能出现问题
    std::cout << "s1 addr is " << &SingleLazy::getInstance() << "\n";
    std::cout << "s2 addr is " << &SingleLazy::getInstance() << "\n";

}
```

输出如下
```
s1 addr is 0x557ab517b151
s2 addr is 0x557ab517b151
```

确实生成了唯一实例，但是存在隐患，多线程可能生成多个实例

2. 饿汉式

| 是否lazy初始化 | 是否多线程安全 | 实现难度 |
| -------------- | -------------- | -------- |
| 否             | 是             | 易       |


- 优点：没有加锁，执行效率会提高
- 缺点：类加载时就初始化，浪费内存

实现：静态成员变量指针方式
> 定义一个类的静态成员变量，用来控制实现单例

```c++
class SingleHungry{
private:
    SingleHungry() {

    }
    SingleHungry(const SingleHungry&) = delete;
    SingleHungry& operator=(const SingleHungry&) = delete;
public:
    static SingleHungry* getInstance() {
        if(single == nullptr) {
            single = new SingleHungry();
        }
        return single;
    }
private:
    static SingleHungry *single;
};
// 饿汉式初始化
SingleHungry *SingleHungry::single = SingleHungry::getInstance();

void thread_func(int num) {
    std::cout << "this is thread " << num << "\n";
    std::cout << "Instance is " << SingleHungry::getInstance() << "\n";
}

void test_SingleHungry() {
    std::cout << "s1 addr is " << SingleHungry::getInstance() << "\n";
    std::cout << "s2 addr is " << SingleHungry::getInstance() << "\n";

    for(int i = 0; i < 3; i++) {
        std::thread ti(thread_func , i);
        ti.join();
    }
}
```

输出结果如下
```
s1 addr is 0x56334e3ae2b0
s2 addr is 0x56334e3ae2b0
this is thread 0
Instance is 0x56334e3ae2b0
this is thread 1
Instance is 0x56334e3ae2b0
this is thread 2
Instance is 0x56334e3ae2b0
```
可见无论是单线程还是多线程模式下，通过静态成员变量的指针实现的单例类都是唯一的。  

但是无论是饿汉式还是懒汉式都存在一个问题，那就是什么时候释放内存？多线程情况下，释放内存就很难了，还有二次释放内存的风险。

下面我们定义一个单例类并用懒汉式调用

```c++
class SinglePointer {
private:
    SinglePointer() {
    }
    SinglePointer(const SinglePointer&) = delete;
    SinglePointer& operator=(const SinglePointer&) = delete;
public:
    static SinglePointer* getInstance() {
        if(single != nullptr) {
            return single;
        }
        Lock.lock();
        if(single != nullptr) {
            Lock.unlock();
            return single;
        }

        single = new SinglePointer();
        Lock.unlock();
        return single;

    }
private:
    static SinglePointer *single;
    static std::mutex Lock;
};

std::mutex SinglePointer::Lock;
SinglePointer *SinglePointer::single = nullptr;
void thread_func_ptr(int num) {
    std::cout << "this thread is " << num << "\n";
    std::cout << "Instance is " << SinglePointer::getInstance() << "\n";
}

void test_SinglePointer() {
    std::cout << "s1 addr is " << SinglePointer::getInstance() << "\n";
    std::cout << "s2 addr is " << SinglePointer::getInstance() << "\n";

    for(int i = 0; i < 3; i++) {
        std::thread ti(thread_func_ptr , i);
        ti.join();
    }

    // 何时释放new的对象?造成内存泄漏

}
```

输出结果如下：

```
s1 addr is 0x55cede7582d0
s2 addr is 0x55cede7582d0
this thread is 0
Instance is 0x55cede7582d0
this thread is 1
Instance is 0x55cede7582d0
this thread is 2
Instance is 0x55cede7582d0
```
此时生成的单例对象的内存空间还没回收，这是个问题，另外如果多线程情况下多次delete也会造成崩溃。

所以需要一种自动回收内存的机制帮助我们回收内存，所以可以使用智能指针来做这个操作

实现方式：智能指针方式
> 可以利用智能指针自动回收内存的机制设计单例类

```c++
class SingleAuto{
private:
    SingleAuto() {

    }
    SingleAuto(const SingleAuto&) = delete;
    SingleAuto& operator=(const SingleAuto&) = delete;
public:
    ~SingleAuto() {
        std::cout << "single auto delete success\n";
    }
    static std::shared_ptr<SingleAuto> getInstance() {
        if(single != nullptr) {
            return single;
        }
        Lock.lock();
        if(single != nullptr) {
            Lock.unlock();
            return single;
        }
        single = std::shared_ptr<SingleAuto>(new SingleAuto());
        Lock.unlock();
        return single;
    }
private:
    static std::shared_ptr<SingleAuto> single;
    static std::mutex Lock;
};

std::shared_ptr<SingleAuto> SingleAuto:: single = nullptr;
std::mutex SingleAuto::Lock;

void test_SingleAuto() {
    auto sp1 = SingleAuto::getInstance();
    auto sp2 = SingleAuto::getInstance();

    std::cout << "sp1 is " << sp1 << "\n";
    std::cout << "sp2 is " << sp2 << "\n";
    // 存在隐患，可以手动删除裸指针，造成崩溃，因为会二次释放内存
    // delete sp1.get();

}
```

输出如下

```
sp1 is 0x56252c7252f0
sp2 is 0x56252c7252f0
```

智能指针方式不存在内存泄漏，但是有一个隐患就是单例类的析构函数是public,如果被人手动调用会存在崩溃问题，比如将上面的test_SingleAuto中的注释打开，程序会崩溃。


实现：辅助类智能指针单例模式
> 智能指针在构造的时候可以指定删除器，所以可以传递一个辅助类或者辅助函数帮助智能指针回收内存时调用我们指定的析构函数

```c++
// NOTE: safe deletorr
// 防止外界deletor
// 声明辅助类
// 该类定义放函数调用SingleAutoSafe析构函数
// 不可以提前声明辅助类，编译器会报 incomplete type
// class SafeDeletor
// 所以要提前定义辅助类
class SingleAutoSafe;
class SafeDeletor {
public:
    void operator()(SingleAutoSafe* s) {
        std::cout << "this is safe deletor operator()\n";
        delete s;
    }
};

void SafeDeletorFunc(SingleAutoSafe* s) {
    std::cout << "this is safe deletor func\n";
    delete s;
}

class SingleAutoSafe {
private:
    SingleAutoSafe() {

    }
    ~SingleAutoSafe() {
        std::cout << "this is sing auto safe deletor\n";
    }
    SingleAutoSafe(const SingleAutoSafe&) = delete;
    SingleAutoSafe& operator=(const SingleAutoSafe&) = delete;
    friend class SafeDeletor;
    friend void SafeDeletorFunc(SingleAutoSafe* s);
public:
    static std::shared_ptr<SingleAutoSafe> getInstance() {
        if(single != nullptr) {
            return single;
        }
        Lock.lock();
        if(single != nullptr) {
            Lock.unlock();
            return single;
        }
        // NOTE:额外指定删除器
        // single = std::shared_ptr<SingleAutoSafe>(new SingleAutoSafe() , SafeDeletor());
        // NOTE:额外指定删除函数
        single = std::shared_ptr<SingleAutoSafe>(new SingleAutoSafe() , SafeDeletorFunc);
        Lock.unlock();
        return single;
        
    }
private:
    static std::shared_ptr<SingleAutoSafe> single;
    static std::mutex Lock;
};

std::shared_ptr<SingleAutoSafe> SingleAutoSafe::single = nullptr;
std::mutex SingleAutoSafe::Lock;

void test_SingleAutoSafe() {
    auto sp1 = SingleAutoSafe::getInstance();
    auto sp2 = SingleAutoSafe::getInstance();
    std::cout << "sp1 is " << sp1 << "\n";
    std::cout << "sp2 is " << sp2 << "\n";
    // 此时无法访问析构函数，非常安全
    // delete sp1.get();
}
```
输出如下

```
sp1 is 0x563f192162b0
sp2 is 0x563f192162b0
this is safe deletor func
```
SafeDeletor要写在SingleAutoSafe上边，并且SafeDeletor要声明为SingleAutoSafe类的友元类，这样就可以访问SingleAutoSafe的析构函数了。 我们在构造single时制定了SafeDeletor(),single在回收时，会调用SingleAutoSafe的仿函数，从而完成内存的销毁。 并且SingleAutoSafe的析构函数为私有的无法被外界手动调用了。

通过辅助类调用单例类的析构函数保证了内存释放的安全性和唯一性。这种方式是生产中常用的。如果将test_SingleAutoSafe的注释打开，手动删除在编译阶段就会报错，达到了代码安全的目的。

## 通过单例模板类

```c++
#pragma once
#include <iostream>
#include <memory>
#include <mutex>
#include <thread>

template<typename T>
class TD;


template<typename T>
class Single_T;

template<typename T>
class SafeDeletor {
public:
    void operator()(T* s) {
        std::cout << "this is safe deletor operator()\n";
        delete s;
    }
};

template<typename T>
void SafeDeletorFunc(T* s) {
    std::cout << "this is safe deletor func\n";
    delete s;
}

template<typename T>
class Single_T {
protected:
    Single_T() = default;
    ~Single_T() {
        std::cout << "this is single template deletor\n";
    }
    Single_T(const Single_T&) = delete;
    Single_T& operator=(const Single_T&) = delete;
    friend class SafeDeletor<T>;
    friend void SafeDeletorFunc<T>(T* s);
public:
    static std::shared_ptr<T> getInstance() {
        if(single != nullptr) {
            return single;
        }
        Lock.lock();
        if(single != nullptr) {
            Lock.unlock();
            return single;
        }
        // NOTE:额外指定删除器
        // single = std::shared_ptr<T>(new T, SafeDeletor<T>());
        // NOTE:额外指定删除函数
        single = std::shared_ptr<T>(new T , SafeDeletorFunc<T>);
        Lock.unlock();
        return single;
        
    }
private:
    static std::shared_ptr<T> single;
    static std::mutex Lock;
};
// NOTE: 模板类的static成员要放在hpp文件内初始化

template<typename T>
std::shared_ptr<T> Single_T<T>::single = nullptr;
template<typename T>
std::mutex Single_T<T>::Lock;

```


```cpp
// 使用std::call_once

#pragma once
#include <mutex>
#include <memory>
#include <iostream>

template<typename T>
class SingleTon {
protected:
    explicit SingleTon() = default;
    SingleTon(const SingleTon& ) = delete;
    SingleTon& operator=(const SingleTon&) = delete;
    ~SingleTon() {
        std::cout << "This is SingleTon deletor" << std::endl;
    }
    static std::shared_ptr<T> single;
public:

    static std::shared_ptr<T> getInstance() {
        static std::once_flag flag;
        std::call_once(flag, [&](){
            single = std::make_shared<T>(new T);
        });
        return single;
    }
};

template<typename T>
std::shared_ptr<T> SingleTon<T>::single = nullptr;

```

使用方式：我们只需要定一个单例类，继承这个模板，并将构造和析构都设置为私有，同时设置友元保证自己的析构和构造可以被友元调用即可。

```c++
class SingleNet : public Single_T<SingleNet> {
private:
    SingleNet() = default;
    ~SingleNet() = default;
    SingleNet(const SingleNet&) = delete;
    SingleNet& operator=(const SingleNet&) = delete;
    friend class SafeDeletor<SingleNet>;
    friend class Single_T<SingleNet>;
    friend void SafeDeletorFunc<SingleNet>(SingleNet* s);
};

void test_Single_T() {
    auto sp1 = SingleNet::getInstance();
    auto sp2 = SingleNet::getInstance();
    std::cout << "sp1 is " << sp1 << "\n";
    std::cout << "sp2 is " << sp2 << "\n";
}
```

