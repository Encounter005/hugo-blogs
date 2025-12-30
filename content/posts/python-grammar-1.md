+++
title = "python-grammar-1"
tags = [ "python",]
categories = [ "编程语言",]
date = "2024-05-02T16:47:48+08:00"
+++
# Python 数据类型

## 数字

在python中数字有三种类型：整数(int)、浮点数(float)、复数(complex)  
我们可以使用`type()`函数来判断一个变量或值属于哪个类，还可以通过`instance()`函数来检查对象是否属于特定的类

```python
def checkType(a):
    print(a, type(a))

a = 1
b = 2.5
c = 1+2j
print(c, '是复数吗？', isinstance(c, complex))


checkType(a)
checkType(b)
checkType(c)

```

## 输入

> input([prompt])

```python
    num = input('Enter a number')
    print(num)

    # 由于默认是字符串，需要做类型转换
    a = (int)(input('Enter an integer'))

```

## 列表

列表中的项目允许不是同一类型。  
我们可以使用`[]`运算符从列表中提取一个项目 或 一系列项目。注意，在Python中，索引从`0`开始。

```python
# NOTE: 声明

a = [1, 2.2, 'python']
b = [5, 10, 14, 20, 30 , 50 , 4]

print('a[2]=', a[2])
print('b[4]=', b[4])

```

## 元组

元组和列表相同，但是元组中的元素是不可变的，元组一旦创建就不可修改

它在括号内 () 定义，其中各项之间用逗号分隔

```python
# NOTE: 声明

tup = (1 , 'awdaw', 12);
print(tup)
# BUG: tup[1] = 1

```

## 字符串

字符串和元组一样，其中的元素是不可变的

```python
# NOTE: 声明

s = "awdawdwada"
print("s[4]", s[4])

# BUG: s[4] = 1
```

## 集合

Set 是唯一项的无序集合。Set 由用大括号 { } 括起来，并由逗号分隔的值的集合。集合中的项目是无序的

```python
# NOTE: 声明
s = {5, 5, 3, 3 ,1, 1, 2, 2, 4, 4, 6}
print('s =',s, type(s))
# BUG: s[2] = 2;
```

## 字典

类似哈希表，需要一个key和一个value，key是唯一的，value可以是任何类型。字典中的项目是无序的

```python
# NOTE: 声明
d = {1:'value', 'key':2}
print(d, type(d))

# NOTE: 访问
print("d['key'] = ", d['key'])

```

## 类

### 对象与类

```python
class Parrot:

    # 类属性
    species = 'bird'

    # Constructor
    def __init__(self, name, age):
        self.name = name;
        self.age = age;

    # Method
    def string(self, song):
        return "{} song is {}".format(self.name, song)
    def dance(self):
        return "{} is now dancing".format(self.name)


blu = Parrot("麻雀", 10)
woo = Parrot("鹦鹉", 14)

print("麻雀是 {}".format(blu.__class__.species))
print("鹦鹉是 {}".format(woo.__class__.species))

print("{} is {} old".format(blu.name, blu.age))
print("{} is {} old".format(woo.name, woo.age))

print( blu.string("极乐净土") )
print( blu.dance() )
```

### 继承

> 可以使用`isinstance` 和 `issubclass` 来检查一个对象是该类的实例和是否是一个特定的类或者子类

```python
class Bird:
    def __init__(self):
        print("Bird is ready")

    def whoisThis(self):
        print("Bird")

    def swim(self):
        print("Swimming faster")

class Penguin(Bird):
    def __init__(self):
        # call super() function
        super().__init__()
        print("Penguin is ready")

    def whoisThis(self):
        print("Penguin")

    def run(self):
        print("Run faster")

peggy = Penguin()
peggy.whoisThis()
peggy.run()
peggy.swim()

```

### 可封装性

```python
class Computer:

    def __init__(self):
        self.__maxprice = 900

    def sell(self):
        print("Price is: {}".format(self.__maxprice))

    def setMaxPrice(self, price):
        self.__maxprice = price

c = Computer()
c.sell()

print("Change price")
c.setMaxPrice(1000)
c.sell()

```

### 多态

```python
class Parrot:

    def fly(self):
        print("Parrot can fly")

    def swim(self):
        print("Parrot can't swim")

class Penguin:

    def fly(self):
        print("Penguin can't fly")

    def swim(self):
        print("Penguin can swim")


# universal method

def flying_test(bird):
    bird.fly()

blu = Parrot()
peggy = Penguin()

flying_test(blu)
flying_test(peggy)
```

### 运算符重载

> 通过在中实现特殊函数(\_\_function_name\_\_)

| 运算符    | 表达       | 在内部                  |
| --------- | ---------- | ----------------------- |
| +         | p1 + p2    | p1.\_\_add\_\_(p2)      |
| -         | p1 - p2    | p1.\_\_sub\_\_(p2)      |
| \*        | p1 \* p2   | p1.\_\_mul\_\_(p2)      |
| 求幂 \*\* | p1 \*\* p2 | p1.\_\_pow\_\_(p2)      |
| 相除 /    | p1 / p2    | p1.\_\_truediv\_\_(p2)  |
| 整除 //   | p1 // p2   | p1.\_\_floordiv\_\_(p2) |
| %         | p1 % p2    | p1.\_\_mode\_\_(p2)     |
| <<        | p1 << p2   | p1.\_\_lshift\_\_(p2)   |
| >>        | p1 >> p2   | p1.\_\_rshift\_\_(p2)   |
| and       | p1 and p2  | p1.\_\_and\_\_(p2)      |
| or        | p1 or p2   | p1.\_\_or\_\_(p2)       |
| ^         | p1 ^ p2    | p1.\_\_xor\_\_(p2)      |
| ~         | ~p1        | p1.\_\_invert\_\_()     |
| <         | p1 < p2    | p1.\_\_lt\_\_(p2)       |
| >         | p1 > p2    | p1.\_\_gt\_\_(p2)       |
| <=        | p1 <= p2   | p1.\_\_le\_\_(p2)       |
| >=        | p1 >= p2   | p1.\_\_ge\_\_(p2)       |
| ==        | p1 == p2   | p1.\_\_eq\_\_(p2)       |
| !=        | p1 != p2   | p1.\_\_ne\_\_(p2)       |


## 迭代器

Python中的Iterator只是一个可以迭代的对象。一个将返回数据的对象，一次返回一个元素。


自定类实现迭代器

```python
class PowTwo:
    def __init__(self, max = 0):
        self.max = max

    def __iter__(self):
        self.n = 0
        return self
    
    def __next__(self):
        if self.n <= self.max:
            result = 2 ** self.n
            self.n += 1
            return result
        else:
            raise StopIteration


a = PowTwo(4)
i = iter(a)
print(next(i))

for i in PowTwo(5):
    print(i)

```
