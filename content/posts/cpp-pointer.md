+++
title = "智能指针"
tags = [ "C/C++",]
categories = [ "编程语言",]
mathjax = false
date = "2024-07-10T21:38:33+08:00"
+++
# 智能指针概述

1. 为什么要有智能指针：直接用`new`和`delete`运算符极其容易导致内存泄露，而且非常难以避免，于是人们发明了智能指针这种可以自动回收内存的的工具

2. 智能指针一共就三种：普通的指针可以单独一个指针占用一块内存，也可以多个指针共享一块内存

   1. 共享型智能指针：`shared_ptr`
      > 同一块堆内存可以被多个`shared_ptr`共享
   2. 独享型智能指针：`unique_ptr`
      > 同一块堆内存只能被一个`unique_ptr`拥有
   3. 弱引用智能指针：`weak_ptr`
      > 也是一种共享型智能指针，可以视为对共享型智能指针的一种补充

```c++
#include <iostream>

int main() {
  int *pi = new int(10);
  int *pi2(pi); //NOTE: pi和pi2共享同一块内存
  return 0;
}
```

3. 智能指针的注意事项

   智能指针和裸指针不要混用

## shared_ptr

### 工作原理

1. 我们在动态内存分配时，堆上的内存必须通过栈上的内存来寻址，也就是说栈上的指针(堆上的指针也可以指向堆内存，但终究是要通过栈来寻址)是寻找堆内存的唯一方式

2. 所以我们可以给堆内存添加一个引用计数，有几个指针指向它，它的引用计数就是几。当引用计数为 0 时，操作系统就会释放掉这块内存

### 常用操作

#### 1. 初始化

使用`new`运算符初始化

> 一般来说不推荐使用`new`进行初始化，因为 C++标准提供了专门创建`shared_ptr`的函数`make_shared()`，该函数是经过优化的，效率更高

使用`make_shared()`初始化

`注意` ： 千万不要用裸指针初始化`shared_ptr`，容易出现内存泄露的问题

使用复制构造函数初始化也行

```c++
#include <iostream>
#include <memory>

int main() {

  std::shared_ptr<int> shared1(new int (100));
  std::shared_ptr<int> shared2 = std::make_shared<int>(100);
  std::shared_ptr<int> shared3(shared2);// 使用复制构造函数初始化也行
  int *pi = new int(100);

  std::shared_ptr<int> shared4(pi);
  // delete pi; // NOTE: 会造成二次释放(堆内存的重复释放)

  return 0;
}

```

#### 2. shared_ptr 的引用计数

智能指针就是通过引用计数来判断释放内存的时机的  
`use_count()`函数可以得到`shared_ptr`对象的引用计数

```c++
#include <iostream>
#include <memory>

int main() {

  std::shared_ptr<int> shared1 = std::make_shared<int>(100);
  std::cout << shared1.use_count() << std::endl;

  std::shared_ptr<int> shared2(shared1);
  std::cout << shared1.use_count() << std::endl;

  shared2.reset(); // NOTE: 释放掉该指针对对象的控制权
  std::cout << shared1.use_count() << std::endl;

  return 0;
}

```

#### 3. 把 shared_ptr 当成普通指针使用

智能指针可以像普通指针那样使用，`shared_ptr`早已对各种操作进行了重载，就当它是普通指针就可以了

```c++
#include <iostream>
#include <memory>

int main() {


  std::shared_ptr<int> shared1 = std::make_shared<int>(100);
  std::cout << *shared1 << std::endl;

  return 0;
}

```

#### 4. 常用函数

1. `unique`函数
   > 判断该`shared_ptr`对象是否独占，若独占，返回`true`，否则返回`false`

```c++
#include <iostream>
#include <memory>

int main() {

  std::shared_ptr<int> shared1 = std::make_shared<int>(100);
  std::cout << shared1.unique() << std::endl;

  std::shared_ptr<int> shared2(shared1);
  std::cout << shared1.unique() << std::endl;


  shared2.reset();
  std::cout << shared1.unique() << std::endl;

  return 0;
}

```

2. `reset`函数

   1. 当`reset`函数有参数时，改变此`shared_ptr`对象指向的内存
   2. 当`reset`函数无参数时，将此`shared_ptr`对象置空，也就是将对象内存的指针设置为`nullptr`

```c++
#include <iostream>
#include <memory>

int main() {

  std::shared_ptr<int> shared1 = std::make_shared<int>(100);
  std::cout << shared1.unique() << std::endl;

  std::shared_ptr<int> shared2 = std::make_shared<int>(100);

  shared1.reset(new int(100));

  shared2 = shared1;
  shared1.reset();
  return 0;
}

```

3. `get`函数， 强烈不推荐使用

如果一定要用，那么一定不能`delete`返回的指针

```c++
#include <iostream>
#include <memory>

int main() {

  std::shared_ptr<int> shared1 = std::make_shared<int>(100);
  std::cout << shared1.unique() << std::endl;

  delete shared1.get(); // NOTE: 堆内存重复释放

  return 0;
}

```

4. `swap`函数

   > 交换两个智能指针所指向的内存

1. `std`命名空间中全局的`swap`函数
1. `shared_ptr`类提供的`swap`函数

```c++
#include <iostream>
#include <memory>

int main() {
  std::shared_ptr<int> shared1 = std::make_shared<int>(100);
  std::shared_ptr<int> shared2 = std::make_shared<int>(1000);

  shared1.swap(shared2);
  std::cout << *shared1 << std::endl;
  std::cout << *shared2 << std::endl;

  std::swap(shared1, shared2);
  std::cout << *shared1 << std::endl;
  std::cout << *shared2 << std::endl;

  return 0;
}

```

#### 5. 关于智能指针创建数组的问题

#### 6. 用智能指针作为参数传递时直接值传递就行

`shared_ptr`的大小为固定的`8`或`16`字节

> 也就是两倍指针的大小，32 位系统指针为`4`个字节，64 位系统指针为`8`个字节，`shared_ptr`中就两个指针，所以直接按值传递就行了

#### 7. 总结

在现代程序中，当想要共享一块堆内存时，优先使用`shared_ptr`，可以极大的减少内存泄露的问题

## weak_ptr

### 1. 介绍

1. 这个智能指针是在 C++11 的时候引入标准库，它的出现完全是为了弥补`shared_ptr`的天生缺陷，其实`shared_ptr`可以说是几乎完美
2. 只是通过引用计数实现的方式也引来了引用成环的问题，这种问题靠他自己是没法解决的，所以在 C++11 的时候将`shared_ptr`和`weak_ptr`一起引入了标准库，依次来解决循环引用的问题

### 2. shared_ptr 循环引用的问题

```c++
#include <iostream>
#include <memory>

class B;

class A {

public:
  std::shared_ptr<B> sharedB;
};

class B {

public:
  std::weak_ptr<A> sharedA; // NOTE: 只有把其中一个堆内存用weak_ptr来控制，这两块堆内存才会被释放
};

int main() {
  // std::shared_ptr<int> shared1 = std::make_shared<int>(100);
  // std::cout << shared1.use_count() << std::endl;
  //
  // std::weak_ptr<int> weak1(shared1);
  // std::cout << shared1.use_count() << std::endl;


  std::shared_ptr<A> sharedA1 = std::make_shared<A>();
  std::shared_ptr<B> sharedB1 = std::make_shared<B>();

  sharedA1->sharedB = sharedB1; // NOTE: 两个堆内存，你指我，我指你，双方都在等着对方释放
  引用计数都为1，当引用计数为0的时候，堆内存才会被释放，所以这就造成了内存泄露
  sharedB1->sharedA = sharedA1;

  return 0;
}
```

### 3. weak_ptr 的作用原理

`weak_ptr`的作用对象需要绑定到`shared_ptr`对象上，作用原理是`weak_ptr`不会改变`shared_ptr`的引用计数，只要`shared_ptr`对象的引用计数为 0，就会释放内存，`weak_ptr`不会影响到释放内存的功能

### 4. 总结

`weak_ptr`是用比较少，就是为了处理`shared_ptr`循环引用问题设计的

## unique_ptr

### 1. 介绍

独占式智能指针，在是用智能指针时，我们一般优先考虑独占式智能指针，因为消耗更小，如果发现内存需要共享，那么再去是用`shared_ptr`

### 2. unique_ptr 的初始化

> 和 shared_ptr 完全类似

1. 使用`new`运算符初始化
2. 使用`make_unique`初始化

```c++
#include <iostream>
#include <memory>

int main() {

  std::unique_ptr<int> unique1(new int(100));
  std::unique_ptr<int> unique2 = std::make_unique<int>(1000);
  std::cout << *unique1 << std::endl;
  std::cout << *unique2 << std::endl;

  return 0;
}
```

### 3. unique_ptr 的常用操作

1. `unique_ptr`禁止复制构造函数，他禁止赋值运算符的重载运算。否则独占毫无意义
2. `unique_ptr`允许移动构造，移动赋值。移动语义代表之前的对象已经失去了意义，移动操作自然不影响独占的特性

```c++
#include <iostream>
#include <memory>

int main() {

  std::unique_ptr<int> unique1 = std::make_unique<int>(100);
  std::unique_ptr<int> unique2 = std::make_unique<int>(1000);

  unique2 = std::move(unique1);
  std::cout << *unique2 << std::endl;

  return 0;
}
```

4. `reset()`函数
   1. 不带参数的情况下，释放智能指针的对象，并将智能指针置空
   2. 带参数的情况下，释放智能指针的对象，并将智能指针指向新的对象

```c++
#include <iostream>
#include <memory>

int main() {

  std::unique_ptr<int> unique1 = std::make_unique<int>(100);
  std::unique_ptr<int> unique2 = std::make_unique<int>(1000);

  unique1.reset();
  unique2.reset(new int(12));

  std::cout << *unique2 << std::endl;

  return 0;
}
```

4. 将`unique_ptr`的对象转化为`shared_ptr`的对象，当`unique_ptr`的对象作为一个右值时，就可以将该对象转化为`shared_ptr`的对象
   > 这个使用的并不多，需要将独占式指针转化为共享式指针常常是因为先前设计失误
   > 注意：shared_ptr 对象无法将其转化为 unique_ptr 对象

```c++
#include <iostream>
#include <memory>

void myfunc(std::unique_ptr<int> unique3) {
  std::shared_ptr<int> shared1(std::move(unique3));
} // NOTES: 一旦将一个对象转化成右值时，必须保证以后不再单独是用这个对象

int main() {

  std::unique_ptr<int> unique1 = std::make_unique<int>(100);
  std::unique_ptr<int> unique2 = std::make_unique<int>(1000);

  return 0;
}
```

### 智能指针的适用范围

#### 1. 能使用智能指针就尽量是用智能指针 ， 但是有些情况下不能使用智能指针

有些函数必须使用 C 语言的指针，这些函数又没有替代，这种情况下，才使用普通的指针，其他情况一律是用智能指针

必须使用 C 语言的指针包括:

1. 网络传输函数：比如 windows 下的`send`,`recv`函数，智能使用 C 语言的指针，无法替代
2. C 语言文件操作部分：这方面 C++已经有了替代品，C++的文件部分完全支持智能指针，所以在做大型项目时，推荐使用 C++的文件操作功能

#### 2. 我们应该是用哪个智能指针呢？

1. 优先使用`unique_ptr`，内存需要共享时使用`shared_ptr`
2. 当使用`shared_ptr`时，如果出现循环引用的情况下，再去考虑`weak_ptr`


