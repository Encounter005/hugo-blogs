+++
title = "C++中的静态类型、动态类型、RTTI"
tags = [ "C/C++",]
categories = [ "编程语言",]
date = "2024-04-22T21:10:39+08:00"
+++
# 简介

本文介绍了C++中的静态类型、动态类型、RTTI。

## 静态类型

C++ 是一种静态类型语言（static typing），这意味着类型检查是在编译时进行而非运行时。换句话说，每个变量、对象、函数返回值等在编译时都必须明确其类型。这种类型系统的主要优点是能够在程序运行之前发现类型不匹配等错误，从而提高代码的安全性和稳定性。

举一个例子

```cpp
#include <iostream>

int main() {
    int num = 42;        // 'num' 静态类型为int
    double pi = 3.14159; // 'pi' 静态类型为double

    // FIX: num = pi; 这种写法会造成编译错误，因为类型不匹配

    std::cout << "The value of num is: " << num << std::endl;
    std::cout << "The value of pi is: " << pi << std::endl;

    return 0;
}
```

## 动态类型

在C++这种静态类型语言中，通常所说的"dynamic typing"（动态类型）并不是内置的语言特性，因为C++要求变量在编译时就明确其类型。相对的，"dynamic typing"通常指的是在运行时能够处理多种不同类型的数据。在C++中实现类似动态类型的行为，通常是通过以下几种方式：

1. **void指针** ：void\* 类型，这种方式通常称为"smart pointer"，它允许程序员在不担心对象的生存周期的情况下，指向任意对象，从而实现动态类型的支持，但使用它们往往需要手动管理类型和内存，同时要在使用前将其转换（强制类型转换）回正确的类型。

2. **多态**：利用C++的多态特性，可以通过基类指针或引用来操作一组派生自同一基类的对象。这使得代码可以在运行时处理多种不同类型的对象，而不必在编译时知道具体的派生类型。

3. **任意类型**： 通过使用`std::any`类型，你可以存放任何类型的值，并且在需要的时候通过`std::any_cast`来提取原始类型，这在某种程度上提供了动态类型的能力。

举点例子：

```cpp
#include <iostream>
#include <any>

void test_void_pointer() {
    int x = 42;
    float y = 3.14f;
    std::string z = "hello world\n";

    void* void_ptr;

    void_ptr = &x;
    std::cout << "int_value: " << *(static_cast<int*>(void_ptr)) << std::endl;
    void_ptr = &y;
    std::cout << "double_value: " << *(static_cast<double*>(void_ptr)) << std::endl;
    void_ptr = &z;
    std::cout << "string_value: " << *(static_cast<std::string*>(void_ptr)) << std::endl;
}

void test_any() {
    std::any any_value;
    any_value = 42;
    std::cout << "int_value: " << std::any_cast<int>(any_value) << std::endl;
    any_value = 3.14;
    std::cout <<"double_value: " << std::any_cast<double>(any_value) << std::endl;
    any_value = std::string( "awdawda" );
    std::cout << "string_value: " << std::any_cast<std::string>(any_value) << std::endl;
}

int main() {
    test_void_pointer();
    test_any();
    return 0;
}

```

需要注意的是，上面实例中的`void*`和`std::any`由于额外的类型检查和转换，会在运行时导致额外的性能开销。

## RTTI

RTTI，即运行时类型信息（Run-Time Type Identification），是C++语言中的一个机制，它允许在程序运行时获取对象的类型信息。RTTI是面向对象编程中多态性的一个重要特性，它能够让我们在程序运行时确定对象的实际派生类型。

RTTI主要涉及以下两个操作符和一个类类型特性：

1. **dynamic_cast**: 用于安全地在继承体系中转换指针或引用。它是类型转换的一种，可以将基类的指针或引用转换为派生类的指针或引用，并在转换不安全的情况下提供错误检查。

2. **typeid**: 当你对表达式使用typeid时，它会返回一个std::type_info对象的引用，该对象代表了表达式的类型。如果该表达式是一个多态类型（即含有虚函数的类）的对象，typeid会返回该对象的动态类型，也就是最派生的类型。
3. **std::type_info**: 这是与`typeid`一起使用的标准库类，其对象包含了类型的信息，如类型的名称。它提供了比较两个类型是否相等的能力，即通过`type_info`的`operator==`来确定两个对象是否为同一类型。

例子：

typeid:

```c++
#include <iostream>
#include <typeinfo>

class Base { virtual void dummy() {} };
class Derived : public Base { /* ... */ };

int main() {
    Base* base_ptr = new Derived;

    // Using typeid to get the type of the object
    std::cout << "Type: " << typeid(*base_ptr).name() << '\n';

    delete base_ptr;
    return 0;
}
```

dynamic_cast:

```c++
#include <iostream>

class Base { virtual void dummy() {} };
class Derived1 : public Base { /* ... */ };
class Derived2 : public Base { /* ... */ };

int main() {
    Base* base_ptr = new Derived1;

    // Using dynamic_cast to safely downcast the pointer
    Derived1* derived1_ptr = dynamic_cast<Derived1*>(base_ptr);
    if (derived1_ptr) {
        std::cout << "Downcast to Derived1 successful\n";
    }
    else {
        std::cout << "Downcast to Derived1 failed\n";
    }

    Derived2* derived2_ptr = dynamic_cast<Derived2*>(base_ptr);
    if (derived2_ptr) {
        std::cout << "Downcast to Derived2 successful\n";
    }
    else {
        std::cout << "Downcast to Derived2 failed\n";
    }

    delete base_ptr;
    return 0;
}
```

RTTI通常用于实现某些需要类型检查的功能，尤其是那些与继承体系交互的场景。但是，也应当注意它可能会导致一些性能开销，因此程序设计时还应考虑是否有其他方式能够达到相同目的，如使用虚函数来实现多态，而不是依赖RTTI来进行类型判断。在使用RTTI时，请确保编译器的相应设置或选项已经开启，因为某些编译器可能允许关闭RTTI来减少程序的体积和提高性能。在这种情况下，RTTI相关的功能将不可用。

type_info:

```c++
#include <iostream>
#include <typeinfo>
#include <memory>

class Base{
public:
    virtual ~Base() {}
};

class Derived : public Base {};

int main() {
    std::unique_ptr<Base> b(new Base());
    std::unique_ptr<Derived> d(new Derived());

    const std::type_info& ti_b = typeid(*b);
    const std::type_info& ti_d = typeid(*d);

    std::cout << "Base class type: " << ti_b.name() << std::endl;
    std::cout << "Derived class type: " << ti_d.name() << std::endl;

    if(ti_b == ti_d) {
        std::cout << "The types are the same!" << std::endl;
    } else {
        std::cout << "The types are different!" << std::endl;
    }


    return 0;
}
```
