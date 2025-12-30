+++
title = "CPP-类"
tags = [ "C/C++", "面向对象",]
categories = [ "编程语言",]
date = "2024-03-17T22:46:26+08:00"
+++
# 类

## 简介

类就是对一种类型的抽象，例如纸箱、木箱都是箱子，箱子就是类，而木箱、纸箱就是类实例化的对象

## 构造函数

类相当于定义了一个新的类型，该类型生成在堆或栈上的对象时内存排布与C语言相同。C++规定有在类对象创建时就在对应内存将数据初始化的能力，这就是构造函数

例子:

```c++
#include <iostream>

class Box {
public:

    // 默认构造函数
    Box() = default;
    // 普通构造函数
    Box(const std::string& name, const int cap) : id_(name) , capacity_(cap) {
        std::cout << "Box Constructor" << std::endl;
    }
    // 复制构造函数
    Box(const Box& other) {
        std::cout << "Box copy constructor" << std::endl;
        id_ = other.id_;
        capacity_ = other.capacity_;
    }
    // 移动构造函数
    Box(Box&& other) noexcept {
        std::cout << "Box move constructor" << std::endl;
        id_ = std::move(other.id_);
        capacity_ = other.capacity_;
        capacity_ = 0;
    }

    // 析构函数
    ~Box() {}


private:
    std::string id_;
    std::size_t capacity_;
};

int main() {
    Box b1("Paper Box", 100); // 创建一个对象
    Box b2(b1); // 复制一个对象
    Box b3 = std::move(b2); // 移动构造
    return 0;
}
```

输出如下

```
Box Constructor
Box copy constructor
Box move constructor
```

上面的例子演示了构造函数的几种类型:

- 普通构造函数: 如代码所示
- 复制构造函数：用另一个对象来初始化对象
- 移动构造函数：用另一个对象来初始化对象, 可以避免复制带来的开销
- 默认构造函数：当没有定义任何构造函数时，编译器会为该类生成一个默认构造函数，默认构造函数什么都没做，内存没被初始化 例如`Box() {}`

## 析构函数

当类对象被销毁时，就会调用析构函数。栈上对象的销毁就是函数栈的销毁时

```c++
#include <iostream>

class Box {
public:

    // 默认构造函数
    Box() = default;
    // 普通构造函数
    Box(const std::string& name, const int cap) : id_(name) , capacity_(cap) {
        std::cout << "Box Constructor" << std::endl;
    }
    // 复制构造函数
    Box(const Box& other) {
        std::cout << "Box copy constructor" << std::endl;
        id_ = other.id_;
        capacity_ = other.capacity_;
    }
    // 移动构造函数
    Box(Box&& other) noexcept {
        std::cout << "Box move constructor" << std::endl;
        id_ = std::move(other.id_);
        capacity_ = other.capacity_;
        capacity_ = 0;
    }

    // 析构函数
    ~Box() {
        std::cout << "Box destructor" << std::endl;
    }


private:
    std::string id_;
    std::size_t capacity_;
};

int main() {
    Box b1("Paper Box", 100); // 创建一个对象
    Box b2(b1); // 复制一个对象
    Box b3 = std::move(b2); // 移动构造
    return 0;
}

```

输出如下

```
Box Constructor
Box copy constructor
Box move constructor
Box destructor
Box destructor
Box destructor
```

## 总结

1. 构造函数就是C++提供的**必须有的**在对象创建时初始化对象的方法(默认什么都不做也是一种初始化的方式)

2. 当类对象销毁时有一些我们必须手动操作的步骤时，析构函数就派上了用场（例如手动释放内存

所以，几乎所有的类都需要写一个构造函数，但是析构函数却未必需要
