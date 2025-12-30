+++
title = "函数式编程"
tags = [ "C/C++", "函数式编程", "并发编程",]
categories = [ "技术",]
date = "2024-04-08T22:18:55+08:00"
mathjax = true
+++
# 介绍

函数式编程是一种编程范式，它将计算视为数学函数的求值，并避免使用程序状态以及易变对象。其核心概念是使用函数来抽象作用在数据上的操作，而这些函数相互之间几乎没有或没有任何副作用。以下是一些函数式编程的关键特点：

1. **不可变性**：在函数式编程中，状态是不可变的。这意味着一旦创建，数据就不能改变。所有的变化都通过返回新的数据副本来完成，而不是直接修改现有数据。

2. **纯函数**：这些是没有副作用的函数。这意味着给定相同的输入，一个纯函数总是会产生相同的输出，并且在执行过程中不会对系统的其他部分产生影响（例如，不会修改全局变量或状态）。

3. **高阶函数**：函数在函数式编程中可以作为参数传递给其他函数，也可以作为结果返回。这允许创建抽象和组合函数。

4. **函数组合**：这是一种将两个或更多函数结合起来形成一个新函数的技术。组合的结果是一个包装了原来函数行为的新函数。

5. **递归**：由于不使用循环语句，函数式编程通常依赖于递归来执行重复或循环任务。

6. **延迟计算（惰性求值）**：这是一种技术，其中表达式不会立即计算，而是在需要结果之前延迟计算。

7. **模式匹配**：这常用于代数数据类型的解构和分析，允许直接根据数据的结构来处理数据。

在C++中，函数式编程风格可能不像在一些其他语言中那样自然和直观，因为C++本身是一种多范式编程语言，它同时支持面向对象和过程式编程。然而，C++11及其之后的标准引入了一些特性，可以支持更函数式的编程风格。例如，C++提供了标准库中的算法和函数对象，使得函数式编程风格的应用成为可能。

以下是一个使用C++标准库实现`map`和`filter`的例子

```c++
#include <iostream>
#include <algorithm>
#include <vector>
#include <iterator>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    std::vector<int> evenNums, doubleNums;

    std::transform(nums.begin(), nums.end(), std::back_inserter(doubleNums), [](int x){
        return x * 2;
    });

    std::copy_if(nums.begin(), nums.end(), std::back_inserter(evenNums), [](int x){
        return x % 2;
    });

    std::cout << "The even numbers are: ";
    for(auto& x : evenNums) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    std::cout << "The double numbers are: ";
    for(auto& x : doubleNums) {
        std::cout << x << " ";
    }

    return 0;
}
```

在这个例子中，我们定义了一个数组`numbers`，使用`std::transform` 函数遍历它并创建一个新的数组`doubledNumbers`(其中包含 numbers 的每个元素乘以2的结果）。然后我们使用`std::copy_if` 函数过滤出偶数并创建了另一个数组 `evenNumbers`。

C++20标准加入了`ranges`库，对于支持这个标准的编译器，可以写出更接近函数式编程风格的代码。例如：

```c++
#include <iostream>
#include <algorithm>
#include <vector>
#include <iterator>
#include <ranges>

int main() {
    std::vector<int> nums = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    auto doubles =
        nums | std::views::transform( []( int i ) { return i * 2; } );
    auto evens =
        nums | std::views::filter( []( int i ) { return i % 2 == 0; } );

    std::cout << "The even numbers are: ";
    for ( auto x : evens ) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    std::cout << "The double numbers are: ";
    for ( auto x : doubles ) {
        std::cout << x << " ";
    }

    return 0;
}
```

接下来演示一下如果通过并行和函数式编程提高运行效率

# 快速排序

## 介绍

快速排序是一种高效的排序算法，它使用分而治之的策略来对一系列元素进行排序。其核心思想是选择一个“基准”（pivot）元素，然后将数组分为两部分，使得左边部分的所有元素都不大于基准元素，而右边部分的所有元素都不小于基准元素，然后对这两部分递归地进行快速排序。
以下是一个 C++ 实现的快速排序的示例代码：

```c++
    //c++ 版本的快速排序算法
    template<typename T>
    void quick_sort_recursive(T arr[], int start, int end) {
        if (start >= end) return;
        T key = arr[start];
        int left = start, right = end;
        while(left < right) {
            while (arr[right] >= key && left < right) right--;
            while (arr[left] <= key && left < right) left++;
            std::swap(arr[left], arr[right]);
        }
        if (arr[left] < key) {
            std::swap(arr[left], arr[start]);
        }
        quick_sort_recursive(arr, start, left - 1);
        quick_sort_recursive(arr, left + 1, end);
    }
    template<typename T>
    void quick_sort(T arr[], int len) {
        quick_sort_recursive(arr, 0, len - 1);
    }
```

排序演示

假设一开始序列$x_i$是：5，3，7，6，4，1，0，2，9，10，8。

此时，ref=5，i=1，j=11，从后往前找，第一个比5小的数是x8=2，因此序列为：2，3，7，6，4，1，0，5，9，10，8。

此时i=1，j=8，从前往后找，第一个比5大的数是x3=7，因此序列为：2，3，5，6，4，1，0，7，9，10，8。

此时，i=3，j=8，从第8位往前找，第一个比5小的数是x7=0，因此：2，3，0，6，4，1，5，7，9，10，8。

此时，i=3，j=7，从第3位往后找，第一个比5大的数是x4=6，因此：2，3，0，5，4，1，6，7，9，10，8。

此时，i=4，j=7，从第7位往前找，第一个比5小的数是x6=1，因此：2，3，0，1，4，5，6，7，9，10，8。

此时，i=4，j=6，从第4位往后找，直到第6位才有比5大的数，这时，i=j=6，ref成为一条分界线，它之前的数都比它小，之后的数都比它大，对于前后两部分数，可以采用同样的方法来排序。

## 改进

我们用函数式编程来修改上面的快速排序

改进分为了两种版本，如下：

### 迭代器版：

```c++

// @brief 快速排序
// @param first 起始迭代器
// @param last 结束迭代器
template <typename RandomAccessIterator>
void quick_sort(RandomAccessIterator first, RandomAccessIterator last) {
// NOTE: 4. 终止条件：
//          当递归到子数组长度为 0 或 1，即 first >= last 时，排序函数返回，因为长度为 0 或 1 的数组自然是有序的，不需要进一步排序。
    if (first >= last) return;

// NOTE: 1. 选择基准元素：
//          函数首先选择一个基准（pivot）。在这个例子中，基准被选择为要排序部分的中间元素。
    auto pivot = *std::next(first, std::distance(first, last) / 2);

// NOTE: 2. 划分操作：
//          使用两次 std::partition 函数，第一次将所有小于基准的元素移动到基准的左边，第二次将所有等于基准的元素移动到基准右边的起始位置。
//          这样一来，基准元素的正确位置就被找到了，即所有在基准左边的元素都不大于基准，所有在基准右边的元素都不小于基准。
    RandomAccessIterator middle1 = std::partition(first, last,
        [pivot](const auto& em) { return em < pivot; });
    RandomAccessIterator middle2 = std::partition(middle1, last,
        [pivot](const auto& em) { return !(pivot < em); });

// NOTE: 3. 递归排序：
//          对基准左边和右边的子数组递归地调用 quick_sort 函数。左子数组包含所有小于基准的元素，右子数组包含所有大于基准的元素。
    quick_sort(first, middle1);
    quick_sort(middle2, last);
}
```

通过这个过程，数组被分为越来越小的部分，并且每个部分通过递归调用都被排序，最终得到整个数组的有序排列。

### std::list版:
```c++
//@brief 快速排序
//@param input 待排序的list
template <typename T> std::list<T> sequential_quick_sort( std::list<T> input ) {
    if ( input.empty() ) {
        return input;
    }
    std::list<T> result;

    // NOTE: 1.
    // 将input中的第一个元素放入result中，并且将这第一个元素从input中删除
    result.splice( result.begin(), input, input.begin() );

    // NOTE: 2. 去result的第一个元素，将来用这个元素做切割，切割input中的列表
    T const &pivot = *result.begin();

    // NOTE: 3.
    // std::partition是一个标准库函数，用于将容器或数组中的元素按照指定的条件进行分区
    // 使得满足条件的元素排在不满足元素之前
    // 所以经过计算divide_point指向的是input中第一个大于等于pivot的元素
    auto divide_point = std::partition(
        input.begin(), input.end(), [&]( T const &t ) { return t < pivot; } );

    // NOTE: 4. 我们将小于pivot的元素放入lower_part中
    std::list<T> lower_part;
    lower_part.splice( lower_part.end(), input, input.begin(), divide_point );

    // NOTE: 5. 我们将lower_part传递给sequential_quick_sort返回一个新的有序的从小到大的序列
    // lower_part中都是小于divide_point的值
    auto new_lower( sequential_quick_sort( std::move( lower_part ) ) );
    // NOTE: 6. 我们剩余的input列表传递给sequential_quick_sort递归调用，input中都是大于divide_point的值
    auto new_higher( sequential_quick_sort( std::move( input ) ) );
    // NOTE: 7. 到此时new_hither和new_lower都是从小到大排序好的列表


    // NOTE: 8. 进行拼接
    //          比divide_point的值小的部分从当前result的起始位置开始插入，大的部分从当前result的终止位置开始插入
    result.splice( result.end(), new_higher );
    result.splice( result.begin(), new_lower );
    return result;
}
```

补充知识:

> 关于list.splice()

```
splice()函数是list中的一个剪贴函数，将另外一个list中的元素剪贴到本list中，共有三个重载

list1为要操作的list
list2为被剪去的list
position为list1中的某个位置的迭代器

list1调用splice()函数

1. list1.splice(position , list2)  将list2中的所有元素剪贴到list1中的position位置

2. list1.splice(position , list2 , iter) 将list2中某个位置的迭代器iter所指向的元素剪贴到list1中的position位置

3. list1.splice(position , list2 , iter1 , iter2) 将list2中的某个范围迭代器iter1到iter2中的所有元素剪贴到list1中的从position开始的位置

```

调用如下

```c++
void test_sequential_quick_sort() {
    std::list<int> nums = {6 , 1 , 0 , 7 , 5 , 2 , 9 , -1};
    auto sort_result = sequential_quick_sort(nums);
    std::cout << "The result is: ";
    for(auto& item : sort_result) {
        std::cout << " " << item;
    }
    std::cout << std::endl;
}
```

这个函数是一个使用快速排序对链表进行排序的实现。快速排序是一种常用的排序算法，它的基本思想是选择一个基准元素，然后将数组分为两部分，一部分是小于基准元素的元素，另一部分是大于基准元素的元素。然后对这两部分再分别进行快速排序。这个函数使用了C++模板，可以处理任何数据类型的链表。函数的主要步骤包括：

1. 将链表的第一个元素作为基准元素，并将其从链表中删除。

2. 使用`std::partition`函数将链表分为两部分，一部分是小于基准元素的元素，另一部分是大于或等于基准元素的元素。

3. 对这两部分分别进行递归排序。将排序后的两部分和基准元素合并，返回排序后的链表。

## 并行方式

我们提供并行方式的函数式编程，可以极大的利用cpu多核的优势，这在并行计算中很常见。

迭代器版本：

```c++
template <typename RandomAccessIterator>
void Sort(RandomAccessIterator first, RandomAccessIterator last) {
    if (first >= last) return;

    auto pivot = *std::next(first, std::distance(first, last) / 2);
    RandomAccessIterator middle1 = std::partition(first, last,
        [pivot](const auto& em) { return em < pivot; });
    RandomAccessIterator middle2 = std::partition(middle1, last,
        [pivot](const auto& em) { return !(pivot < em); });

    std::future<void> f1 = std::async(std::launch::async, &Sort<RandomAccessIterator>, first, middle1);
    std::future<void> f2 = std::async( std::launch::async, &Sort<RandomAccessIterator>, middle2, last );
    f1.get();
    f2.get();
}
```

std::list版本：

```c++
template <typename T> std::list<T> parrallel_quick_sort( std::list<T> input ) {
    if(input.empty()) {
        return input;
    }

    std::list<T> result;
    result.splice(result.begin() , input , input.begin());
    T const& pivot = *result.begin();

    auto divide_point = std::partition(input.begin() , input.end() , [&](T const &t){
        return t < pivot;
    });

    std::list<T> lower_part;
    lower_part.splice(lower_part.end() , input , input.begin() , divide_point);

    std::future<std::list<T>> new_lower(std::async(&parrallel_quick_sort<T> , std::move(lower_part)));
    // std::future<std::list<T>> new_higher(std::async(&parrallel_quick_sort<T> , std::move(input)));
    // result.splice(result.end() , new_higher.get());
    auto new_higher(parrallel_quick_sort(std::move(input)));
    result.splice(result.end() , new_higher);
    result.splice(result.begin() , new_lower.get());

    return result;
}

```

我们对`lower_part`的排序调用了`std::async`并行处理，而`higher_part`则是串行执行的。这么做提高了计算的并行能力，但有人会问，如果一个数组的大小是1024,那么就是2的10次方，则需要启动10个线程执行，这仅是对一个1024大小的数组排序，如果有多个数组排序，开辟线程会不会很多？其实不用担心这个，因为`std::async`的实现方式是通过`std::launch::deffered`或者`std::launch::async`完成的。编译器会计算当前能否开辟线程，如果能则是使用`std::launch::async`模式开辟线程，如果不能则采用`std::launch::deffered`串行执行。当然，也可以通过线程池来实现并行计算

```c++
template <typename T>
std::list<T> thread_pool_quicK_sort( std::list<T> input ) {
    ThreadPool &ins = ThreadPool::getInstance();
    if ( input.empty() ) {
        return input;
    }

    std::list<T> result;
    result.splice( result.begin(), input, input.begin() );
    T const &pivot = *result.begin();

    auto divide_point = std::partition(
        input.begin(), input.end(), [&]( T const &t ) { return t < pivot; } );

    std::list<T> lower_part;
    lower_part.splice( lower_part.end(), input, input.begin(), divide_point );

    auto new_lower =
        ins.commit( &parrallel_quick_sort<T>, std::move( lower_part ) );
    auto new_higher( parrallel_quick_sort( std::move( input ) ) );
    result.splice( result.end(), new_higher );
    result.splice( result.begin(), new_lower.get() );
    return result;
}

```
