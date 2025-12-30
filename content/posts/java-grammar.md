+++
title = "java语法"
tags = [ "java" ]
category = "编程语言"
date = "2024-03-16T15:47:30+08:00"
+++
# the grammar of java

## 数据类型

byte:

- 数据类型是8位

short:

- 数据类型是16位

int:

- 数据类型是32位

long:

- 数据类型是64位

float:

- 单精度浮点数

double:

- 双精度浮点数

boolean:

- 存储bool量

char:

- 存储任何字符

## number & math类

### number子类

> java为每一个数据类型都封装了对应的包装类

| 包装类    | 基本数据类型 |
| --------- | ------------ |
| booleans  | boolean      |
| byte      | byte         |
| short     | short        |
| integer   | int          |
| long      | long         |
| character | char         |
| float     | float        |
| double    | double       |

```java
public class main {
    public static void main(string[] args) {
        integer x = 5;
        system.out.println(x);
    }
}
```

一些基本方法

| 名称        | 描述                                 |
| ----------- | ------------------------------------ |
| xxxvalue()  | 将number对象转换为xxx数据类型并返回  |
| compareto() | 将number对象与参数比较               |
| equals()    | 判断number对象是否与参数相等         |
| valueof()   | 返回一个number对象指定的内置数据类型 |
| tostring()  | 以字符串形式返回值                   |
| paraseint() | 将字符串解析为int类型                |

用法

paraseint()

```java
public class main {
    public static void main(string[] args) {
        string re = "10.0";
        double x = double.parsedouble(re);
        system.out.println(x);
    }
}
```

### math类

> math类包含了基本的数学运算
> math类的方法都被定义为`static`形式，可以在主函数直接调用

```java
public class main {
    public static void main(string[] args) {
        system.out.println("the sin value of 90 degree is: " + math.sin(math.pi / 2.0));
        system.out.println("the cos value of 0 degree is: " + math.cos(0));
        system.out.println("the tan value of 60 degree is: " + math.tan(math.pi / 3.0));
        system.out.println("the degree of pi / 2 is: " + math.todegrees(math.pi / 2));
    }
}
```

基本方法

| 名称     | 描述                                     |
| -------- | ---------------------------------------- |
| abs()    | 返回参数的绝对值                         |
| ceil()   | 返回向上取整的给定参数的整数             |
| floor()  | 返回向下取整的给定参数的整数             |
| rint()   | 返回与参数最接近的整数，返回类型为double |
| round()  | 表示四舍五入 math.floor(x + 0.5)         |
| min()    | 返回两个参数中的最小值                   |
| max()    | 返回两个参数中的最大值                   |
| pow()    | 返回第一个参数的第二个参数的次方         |
| sqrt()   | 返回参数的算术平方根                     |
| random() | 返回一个随机数                           |

## character类

```java
public class main {
    public static void main(string[] args) {
        //note: 定义一个字符串数组
        char[] test = {'h' , 'e' , 'l' , 'l' , 'o'};
        for(char ch : test) {
            system.out.print(ch);
        }
        system.out.println();
        char ch = 'h';
        system.out.println(ch);
        //note: character字符串数组
        character[] test_2 = {'h' , 'e' , 'l' , 'l' , 'o'};
        system.out.println(test_2[1]);
    }
}
```

基本方法

| 方法           | 说明                                  |
| -------------- | ------------------------------------- |
| isletter()     | 是否是一个字母                        |
| isdigit()      | 是否是一个数字字符                    |
| iswhitespace() | 是否是一个空白字符                    |
| isuppercase()  | 是否是大写字母                        |
| islowercase()  | 是否是小写字母                        |
| tostring()     | 返回字符的字符串形式，字符串的长度为1 |

## string类

> java提供了string类来创建和操作字符串

字符串创建

```java
        //note: 创建字符串
        string tmp = "noob";
        string tmp2 = new string("awdawd");
        system.out.println(tmp);
        system.out.println(tmp2);
        //note: 用字符串数组创建字符串
        char[] hello = {'h' , 'e' , 'l' , 'l' , 'o'};
        string tmp3 = new string(hello);
        system.out.println(tmp3);
        string tmp4 = tmp2; // note: 这种创建方式是引用

```

求字符串长度

> length()

```java
        //note: 求字符串长度
        string world = "world";
        system.out.println(world.length());

```

连接字符串

> concat(string) 返回一个连接好字符串

```java
        //note: 连接字符串
        string s1 = "hello, " , s2 = "world!";
        string s3 = s1.concat(s2);
        system.out.println(s3);

```

创建格式化字符串

string类使用静态方法format()返回一个string对象而不是printstream对象
string类的静态方法format()能用来创建可复用的格式化字符串

```java
        //note: 创建格式化字符串
        string fs;
        fs = string.format("float is: " + "%f , int is: " + "%d , string is: " + "%s" , 10.1 , 12 , s3);
        system.out.println(fs);
```

基本方法

| 名称                                           | 描述                                                                           |
| ---------------------------------------------- | ------------------------------------------------------------------------------ |
| char charat(int index)                         | 返回指定索引处的char值                                                         |
| int compareto(string anotherstring)            | 按字典序比较两个字符串                                                         |
| int comparetoignorecase(string str)            | 按字典序比较两个字符串(忽略大小写)                                             |
| boolean contentequals(stringbuffer sb)         | 当且仅当字符串与指定的stringbuffer有相同顺序的字符时候返回真。                 |
| int indexof(string str)                        | 返回指定子字符串在此字符串中第一次出现处的索引。                               |
| int indexof(string str , int from index)       | 返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始。             |
| int lastindexof(int ch)                        | 返回指定字符在此字符串中最后一次出现处的索引。                                 |
| int lastindexof(int ch, int fromindex)         | 返回指定字符在此字符串中最后一次出现处的索引，从指定的索引处开始进行反向搜索。 |
| int lastindexof(string str)                    | 返回指定子字符串在此字符串中最右边出现处的索引。                               |
| int lastindexof(string str, int fromindex)     | 返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索。   |
| string touppercase()                           | 使用默认语言环境的规则将此 string 中的所有字符都转换为大写。                   |
| string tolowercase()                           | 使用默认语言环境的规则将此 string 中的所有字符都转换为小写。                   |
| char[] tochararray()                           | 将此字符串转换为一个新的字符数组。                                             |
| string substring(int beginindex, int endindex) | 返回一个新字符串，它是此字符串的一个子字符串。                                 |
| string substring(int beginindex)               | 返回一个新的字符串，它是此字符串的一个子字符串。                               |

## stringbuffer & string builder类

> 在使用 stringbuffer 类时，每次都会对 stringbuffer 对象本身进行操作，而不是生成新的对象，所以如果需要对字符串进行修改推荐使用 stringbuffer。
> 在不要求线程安全的情况下，使用string builder类，string builder会比stringbuffer快。

```java
        // note: 先创建一个大小为10的空字符串
        stringbuilder sb = new stringbuilder(10);
        system.out.println(sb);
        // note: 从尾部填充字符串
        sb.append("runoob..");
        system.out.println(sb);
        sb.append("!");
        system.out.println(sb);
        // note: 从指定索引处开始填充字符串，不足部分会扩容
        sb.insert(8 , "java");
        system.out.println(sb);
        // note: 删除指定区间的字符串
        sb.delete(5 , 8);
        system.out.println(sb);
```

## 数组

### 创建数组

```java
        /*
        *   note: 创建数组
        *   datatype[] array
        *   datatype[] array = new datatype[size]
        * */
        integer[] arr = {1 , 2 , 3 , 4 , 5};
        integer[] arr2 = new integer[100];

```

### 遍历数组及访问数组

```java
        // note: 遍历数组
        for(int i = 0; i < arr.length; i++) {
            system.out.println(arr[i]);
        }
        // note: 范围遍历 for_each
        for(integer value : arr) {
            system.out.println(value);
        }
```

### 数组作为函数参数

```java
    public static void printarray(integer[] arr) {
        for(integer value : arr) {
            system.out.println(value);
        }
    }
    printarray(new integer[]{10 , 20 , 30 , 40});
```

### 数组作为返回值

```java
    public static integer[] reverse(integer[] arr) {
        integer[] res = new integer[arr.length];
        for(int i = 0, j = arr.length - 1; i < arr.length; i++ , j--) {
            res[i] = arr[j];
        }
        return res;
    }
    printarray(reverse(arr));
```

### 多维数组

```java
        /*
        *  note: type[][] typename = new type[length][length]
        * */

        string[][] arr = new string[20][30];

```

### arrays类

> 包含了排序、查找、打印等内容

要先`import java.util.arrays`

1. 打印数组

```java
        int [] a = {1 , 2 , 3,  4,  5};
        system.out.println(a);
        system.out.println(arrays.tostring(a)); // 打印数组元素的值

```

2. 升序排序

```java
        int [] b = {2 , 56  ,12893 ,12 , 23 ,3  , 5 , 67123};
        system.out.println(arrays.tostring(b)); // 打印数组元素的值
        arrays.sort(b);
        system.out.println(arrays.tostring(b)); // 打印数组元素的值

```

3. 数组元素是引用类型的排序

```java
import java.util.arrays;

class man implements comparable{
    int age , id;
    string name;
    public man(int age , string name) {
        super(); // 父类型特征，先初始化父类型特征，再初始化子类
        this.age = age;
        this.name = name;
    }

    public int compareto(object o) {
        man man = (man) o;
        if(this.age < man.age) {
            return -1;
        } else if(this.age > man.age) {
            return 1;
        } else {
            return 0;
        }
    }
}

public class main {
    public static void main(string[] args) {
        man[] msmans = {new man(3, "a"), new man(60, "b"),new man(2, "c")};
        arrays.sort(msmans);
        system.out.println(arrays.tostring(msmans));
    }
}

```

```java
        integer[] arr = {10 , 20 , 30 , 40 , 50 , 60 , 70};
        int x = arrays.binarysearch(arr , 30);
        system.out.println(x);

```

5. 填充数组

```java
        int[] a = {1,2,323,23,543,12,59};
        system.out.println(arrays.tostring(a));
        arrays.fill(a , 10);
        system.out.println(arrays.tostring(a));
```

## 方法

1. 定义

```java
修饰符 返回值类型 方法名(参数类型 参数名){
    ...
    方法体
    ...
    return 返回值;
}
```

2. 可变参数

```java
    public static void print(double... args) {
        if(args.length == 0) {
            system.out.println("no argument passed");
        }

        for(double val : args) {
            system.out.print(val + ' ');
        }
        system.out.println();
    }
    public static void main(string[] args) {
        print(10.1 , 2.1 , 3.2);
    }
```

3. finalize()方法 (弃用)

> 这种方法在对象被垃圾收集器析构前调用，用来清除回收对象

在finalize()方法里，必须指定在对象销毁时候要执行的操作  
一般格式是

```java
protected void finalize() {
    // note: terminate your code
}
```

实例

```java
    class cake extends object{
        private int id;
        public cake(int id) {
            this.id = id;
            system.out.println("cake object " + id + "is created");
        }

        protected void finalize() throws java.lang.throwable {
            super.finalize();
            system.out.println("cake object " + id + "is disposed");
        }
    }
    public static void main(string[] args) {
        cake c1 = new cake(1);
        cake c2 = new cake(2);
        cake c3 = new cake(3);

        c2 = c3 = null;
        system.gc();
    }
```

## stream, file, io

`java.io`包含了所有操作输入、输出需要的类

使用这些类之前要先`import java.io.*`

### 读取控制台输入

java的控制台输入由`system.in`完成。  
为了获得一个绑定到控制台的字符串，你可以把`system.in`包装在一个`buffererreader`对象中来创建一个字符串流。

```java
    bufferedreader br = new bufferedreader(new inputstreamreader(system.in));
```

创建完`bufferedreader`对象之后，可以使用其中的`read()`方法来获取控制台的一个字符，或者使用`readline()`方法来获取一个字符串

### 从控制台读取多字符输入

```java
    public static void main(string[] args) throws ioexception {
        bufferedreader br = new bufferedreader(new inputstreamreader(system.in));
        char c;
        system.out.println("input something, press 'q' to quit");
        // read
        do {
            c = (char) br.read();
            system.out.println(c);
        } while(c != 'q');
    }
```

### 从控制台读取字符串

```java
    public static void main(string[] args) throws ioexception {
        bufferedreader br = new bufferedreader(new inputstreamreader(system.in));
        string read;
        system.out.println("enter lines of text");
        system.out.println("enter \"end\" to quit");
        // read
        do {
           read = br.readline();
           system.out.println(read);
        } while(!read.equals("end"));
    }

```

### 读写文件

![io_stream](/image_post/iostream.png)

#### fileinputstream

该流用于从文件读取数据，它的对象可以用关键字 new 来创建。

创建方法:

```java
inputstream f = new fileinputstream("c:/java/hello");
file f = new file("c:/java/hello");
inputstream in = new fileinputstream(f);
```

基本方法

| 名称                                         | 描述                                                                            |
| -------------------------------------------- | ------------------------------------------------------------------------------- |
| public void close() throws ioexception       | 关闭文件输入流并释放与此流相关的的所有系统资源。抛出ioexception异常             |
| public int read(int r) throws ioexception    | 从inputstream对象读取指定字节的数据。返回下一字节的数据，如果已经到结尾则返回-1 |
| public int read(byte[] r) throws ioexception | 从输入流读取r.length长度的字节。返回读取的字节数，如果是文件结尾则返回-1        |
| public int available() throws ioexception    | 返回下一次对此输入流调用的方法可以不接受阻塞地从此输入流读取的字节数。          |

#### fileoutputstream

该类用来创建一个文件并向文件中写数据。
如果该文件不存在，则会自动创建。

创建方法

```java
outputstream f = new fileoutputstream("c:/java/hello")
file f = new file("c:/java/hello");
outputstream fout = new fileoutputstream(f);
```

基本方法

| 名称                                           | 描述                                                            |
| ---------------------------------------------- | --------------------------------------------------------------- |
| public void close() throws ioexception         | 关闭此文件的输出流并释放有关的所有系统资源。抛出ioexception异常 |
| public void write(int w) throws ioexception    | 把指定的字节写入到流中                                          |
| public void write(byte[] w) throws ioexception | 把指定的数组w.length长度的字节写到outputstream中                |

```java
    public static void main(string[] args) {
        try{
            byte bwrite[] = {11 , 23 , 3 , 40 , 5};
            outputstream os = new fileoutputstream("output.txt");
            for(byte value : bwrite) {
                os.write(value);
            }
            os.close();

            inputstream is = new fileinputstream("input.txt");
            int size = is.available();

            for(int i = 0; i < size; i++) {
                system.out.print((char) is.read() + " ");
            }
            is.close();
        } catch(ioexception e) {
            system.out.println("exception");
        }
    }
```

#### scanner类

可以通过scanner类来获取用户输入

创建方法

```java
    import java.util.scanner;
    scanner sc = new scanner(system.in);
```

在读取字符串的时候，通常需要使用`hasnext()`与`hasnextline`判断是否还有输入的数据

```java
    public static void main(string[] args) {
        scanner sc = new scanner(system.in);
        // read
        system.out.println("input something");
        if (sc.hasnext()) {
            string str1 = sc.next();
            system.out.println(str1);
        }
        sc.close();
    }
```

```java
    public static void main(string[] args) {
        scanner sc = new scanner(system.in);
        // read
        system.out.println("input something");
        if (sc.hasnextline()) {
            string str1 = sc.nextline();
            system.out.println(str1);
        }
        sc.close();
    }
```

##### next与nextline的区别

next():

1. 一定要读取到有效字符后才可以结束输入
2. 对输入有效字符之前遇到的空白，next()方法会自动将其去掉
3. 只有输入有效字符之后才将其后面输入的空白作为分隔符或者结束符
4. next()不能的得到带有空格的字符串

nextline():

1. 以`enter`为结束符，也就是说，nextline()方法返回的是输入回撤之前的所有字符
2. 可以获得空白

如果要输入int或float类型的数据，在scanner类中也有支持，但是在输入之前最好先使用hasnextxxx()方法进行验证，再使用nextxxx()来读取

```java
    public static void main(string[] args) {
        scanner sc = new scanner(system.in);
        // read
        int i = 0;
        float f = 0.0f;
        system.out.println("input integer");
        if(sc.hasnextint()) {
            i = sc.nextint();
        }
        system.out.println("the integer is: " + i);

        system.out.println("input float");
        if(sc.hasnextfloat()) {
            f = sc.nextfloat();
        }
        system.out.println("the float is: " + f);
        sc.close();
    }
```

例如计算输入的数的总和和平均值

```java
    public static void main(string[] args) {
        system.out.println("input some value");
        scanner sc = new scanner(system.in);

        double sum = 0;
        int cnt = 0;
        while(sc.hasnextdouble()) {
            double m = sc.nextdouble();
            sum += m;
            cnt++;
        }
        // note: 如果需要退出循环，只需要输入一个非数字字符即可

        system.out.println("the sum is: " + sum);
        system.out.println("the avg is: " + sum / cnt);
    }
```

## 异常处理

有三种异常类型：

- 检查型异常: 最具代表的检查性异常是用户错误或问题引起的异常，这是程序员无法预见的。例如要打开一个不存在文件时，一个异常就发生了，这些异常在编译时不能被简单地忽略。
- 运行时异常：运行时异常是可能被程序员避免的异常。与检查性异常相反，运行时异常可以在编译时被忽略。
- 错误： 错误不是异常，而是脱离程序员控制的问题。错误在代码中通常被忽略。例如，当栈溢出时，一个错误就发生了，它们在编译也检查不到的。

### exception类的层次

所有的异常类是从`java.lang.exception`类继承的子类

![exception_hierarchy](/image_post/exception-hierarchy.png)

### 捕获异常

```java
try {
    // your code
} catch(exceptionname e) {
    // catch 块
}
```

实例：
检测数组索引是否合法

```java
    public static void main(string[] args) {
       try{
           int a[] = new int[2];
           scanner sc = new scanner(system.in);
           system.out.println("please input a index");
           int index = 0;
           if(sc.hasnextint()) {
               index = sc.nextint();
           }
           // note: 如果索引不合法，数组就会抛出arrayindexoutofboundsexception异常
           system.out.println("access element successfully: " + a[index]);
       }  catch(arrayindexoutofboundsexception e) {
           system.out.println("exception thrown: " + e);
       }
       system.out.println("out of the block");
    }
```

#### 多重捕获块

```java
try{

} catch(exceptiontype e1) {

} catch(exceptionname e2) {

} catch(exceptiontype e3) {

}
```

```java
try {
    file = new fileinputstream(filename);
    x = (byte) file.read();
} catch(filenotfoundexception f) { // not valid!
    f.printstacktrace();
    return -1;
} catch(ioexception i) {
    i.printstacktrace();
    return -1;
}
```

上述代码中，抛出的异常会去匹配对应的异常类型的catch块

#### throw / throws 关键字

`throw`关键字用于在代码中抛出异常，`throws`关键字用于在方法声明中指定可能会抛出的异常类型

##### throw 关键字

通常情况下，当代码执行到某个条件下无法继续正常执行时，可以使用`throw`关键字抛出异常，已告知调用者当前代码的执行状态

例如下面的代码，用来判断参数是否小于0,如果是，则抛出一个illegalargumentexception异常

```java
    public static void test(int num) {
        if(num < 0) {
            throw new illegalargumentexception("number must be greater than 0");
        }
    }
```

##### throws 关键字

`throws`关键字用于在方法声明中指定该方法可能抛出异常。当方法内部抛出指定类型的异常时，该异常会被传递给调用该方法的代码，并在该代码中处理异常

例如下面的代码，当readfile方法内部发生异常时，会将该异常传递给调用该方法的代码。在调用该方法的代码中，必须捕获或声明处理ioexception

```java
    public static void readfile(string filepath) throws ioexception {
        bufferedreader bf = new bufferedreader(new filereader(filepath));
        string line = bf.readline();
        while(line != null) {
            system.out.println(line);
            line = bf.readline();
        }
        bf.close();
    }
```

一个方法可以声明抛出多个异常，多个异常之间用逗号隔开

```java
    public static void withdraw(double amount) throws remoteexception, insufficientresourcesexception {
        // do something
    }

```

#### finally 关键字

`finally` 关键字用来创建在 `try` 代码块后面执行的代码块。

无论是否发生异常，`finally` 代码块中的代码总会被执行。

在 `finally` 代码块中，可以运行清理类型等收尾善后性质的语句。

```java
try {

} catch(exceptiontype e) {

} finally {

}
```

实例：

```java
    public static void main(string[] args) {
        int[] a = new int[2];
        try {
            system.out.println("access element three: " + a[3]);
        } catch (arrayindexoutofboundsexception e) {
            system.out.println("exception thrown: " + e);
        } finally {
            a[0] = 6;
            system.out.println("first element value: " + a[0]);
            system.out.println("the finally statement is excuted");
        }
    }
```

注意：

- catch块不能独立于try块
- try块后面不能没有catch块也没finally块
- try, catch, finally块之间不能添加任何代码

#### try-with-source

我们可以使用`try-with-source`语法糖来打开资源，并且可以在语句执行完毕后确保每个资源都被自动关闭

```java
    public static void main(string[] args) throws ioexception {
        try (scanner sc = new scanner(system.in)){
            printwriter writer = new printwriter(new file("testwrite.txt"));
            while(sc.hasnext()) {
                writer.print(sc.next());
            }
        }
    }
```

多个资源回收时，`try-with-source`语句以相反的顺序关闭这些资源

### 声明自定义异常

在java编写自定义异常时，要记住以下几点：

- 所有异常都必须是`throwable`的子类
- 如果希望写一个检查性异常类，则需要继承`exception`类。
- 如果希望写一个运行时异常类，那么则需要继承`runtimeexception`类

创建方式:

```java
class myexception extends exception {

}
```

例如我们要做一个检查银行账户类，用户在取款时，有可能余额小于取款额，这时我们就可以设计一个资金不足异常类，用于在这种情况下抛出异常

```java
class insufficientfundsexception extends exception {
    private double amount;
    public insufficientfundsexception(double amount) {
        this.amount = amount;
    }
    public double getamount() {
        return amount;
    }
}

```

然后再设计一个检查账户类

```java
class checkingaccount {

    // balance是余额，number是卡号
    private double balance;
    private int number;

    public checkingaccount(int num) {
        this.number = num;
    }

    // 存钱
    public void deposit(double amount) {
        this.balance = amount;
    }

    // 取款
    public void withdraw(double amount) throws insufficientfundsexception {
        if(balance >= amount) {
            balance -= amount;
        } else {
            double needs = amount - balance;
            throw new insufficientfundsexception(needs);
        }
    }

    // 返回余额
    public double getbalance() {
        return balance;
    }

    // 返回卡号
    public int getnumber() {
        return number;
    }
}

```

测试如下：

```java
    public static void main(string[] args) {
        checkingaccount c = new checkingaccount(123);
        system.out.println("deposit 5000.0...");
        c.deposit(5000.0);
        try {
            system.out.println("\nwithdrawing $1000...");
            c.withdraw(100);
            system.out.println("\nwithdrawing $6000...");
            c.withdraw(6000);
        } catch(insufficientfundsexception e) {
            system.out.println("sorry, but you are shot $" + e.getamount());
            e.printstacktrace();
        }
    }

```

结果如下：

```
insufficientfundsexception
	at checkingaccount.withdraw(main.java:40)
	at main.main(main.java:63)
deposit 5000.0...

withdrawing $1000...

withdrawing $6000...
sorry, but you are shot $1100.0


```

# 面向对象

## 继承

### 特性

- 子类拥有父类非`private`属性、方法
- 子类拥有自己的属性和方法，即子类可以对父类进行扩展
- 子类可以用自己的方式实现父类的方法
- java的继承是单继承，但是可以多重继承，无法多继承

![inherit](/image_post/inherit.png)

继承格式:

```java
class 父类 {

}

class 子类 extends 父类 {

}
```

### 继承关键字

继承可以使用`extends`和`implements`关键字，而且所有的类都是继承于`java.lang.object`，当一个类没有继承的两个关键字，则默认继承`object`祖先类

**extends, super, this**关键字

- extends只能继承一个类
- super用来实现对父类成员的访问，用来引用当前的父类对象
- this，指向自己的引用

```java

class animal {
    private string name;
    private int id;

    public animal() {
        this.name = "none";
        this.id = 0;
    }
    public animal(string name , int id) {
        this.name = name;
        this.id = id;
    }

    public void eat() {
        system.out.println("eating");
    }

    public void sleep() {
        system.out.println("sleeping");
    }

    public void introduction() {
        system.out.println("my name is: " + name + " and my id is: " + id);
    }

    public void test() {
        system.out.println("this is animal");
    }
}

class penguin extends animal {
    public penguin(string name , int id) {
        super(name , id);
    }
}

class mouse extends animal {
    public mouse(string name , int id) {
        super(name , id);
    }
}

class dog extends animal {
    public void test() {
        system.out.println("this is dog");
    }
    public void test2() {
        super.test();
    }
}

public class main {
    public static void main(string[] args) {
        mouse mou = new mouse("mou" , 1);
        penguin pen = new penguin("pen" , 1);
        pen.eat();
        mou.introduction();
        dog dog = new dog();
        dog.test();
        dog.test2();
    }
}
```

**implements**关键字

> 可以是java变相具有多继承的特性，使用范围为类继承接口的情况，可以同时继承多个接口

```java
interface a {
    public void eat();
    public void sleep();
}

interface b {
    public void show();
}

abstract class c implements a , b {
}

```

**final**关键字

- 使用`final`关键字声明类，就是把类定义为最终类，不能被继承，或者用于修饰方法，该方法不能被子类重写

格式：

```java

// 声明类
final class a {}

// 声明方法

(public, private, default, protected) final return type functionname() {}
```

### 构造器

如果父类的构造器带有参数，则必须在子类的构造器中显式地通过`super`关键字调用父类的构造器并配以适当的参数列表，如果父类没有则不需要，系统会自动调用父类的无参构造器

```java
class superclass {
    private int n;
   public superclass() {
       system.out.println("call superclass()");
   }
   public superclass(int n) {
       this.n = n;
       system.out.println("call superclass(int n)");
   }
}

class subclass extends superclass {
   private int n;
   public subclass(int n) {
       super(n);
       system.out.println("subclass(n)");
       this.n = n;
   }
    public subclass() {
        system.out.println("subclass");
    }

}

public class main {
    public static void main(string[] args) {
        system.out.println("------subclass 类继承------");
        subclass sc1 = new subclass();
        subclass sc2 = new subclass(100);
        system.out.println("------subclass2 类继承------");
    }
}
```

## override & overload

### 重写(override)

重写是子类对父类的允许访问的方法的实现过程重写，返回值和形参都不改变。  
重写方法不能抛出新的检查异常或者比被重写的方法声明更加广泛的异常。例如：父类的一个方法声明了一个检查异常ioexception，但是在重写这个方法的时候不能抛出exception异常，因为exception是ioexception的父类，抛出ioexception或者ioexception的子类异常

```java
class animal {
    public void move() {
        system.out.println("moving....");
    }
}

class dog extends animal {
    public void move() {
        system.out.println("the dog is moving...");
    }
}

public class main {
    public static void main(string[] args) {
        animal a = new animal();
        animal b = new dog();
        a.move();
        b.move();
    }
}

```

在上面的代码中，dog类继承自animal类并重写了move方法，尽管b是属于animal类型，但它运行的是dog类的move方法。注意，如果dog类存在animal类中没有的方法，并且在调用了这个方法，编译不会通过。

方法的重写原则

- 参数列表和被重写的方法的参数列表必须完全相同
- 返回类型与被重写的方法的返回类型可以不相同，但是必须是父类返回值的派生类
- 访问权限不能比父类被重写的方法的访问权限更低。
- 父类的成员方法只能被它的子类重写
- 声明为 static 的方法不能被重写，但是能够被再次声明。
- 声明为 final 的方法不能被重写。
- 子类和父类在同一个包中，那么子类可以重写父类所有方法，除了声明为 private 和 final 的方法。
- 子类和父类不在同一个包中，那么子类只能够重写父类的声明为 public 和 protected 的非 final 方法。
- 重写的方法能够抛出任何非强制异常，无论被重写的方法是否抛出异常。但是，重写的方法不能抛出新的强制性异常，或者比被重写方法声明的更广泛的强制性异常，反之则可以。
- 构造方法不能被重写。

**super**关键字

当需要在子类中调用父类的被重写方法时，要使用 super 关键字。

```java
class animal {
    public void move() {
        system.out.println("moving....");
    }
}

class dog extends animal {
    public void move() {
        super.move();
        system.out.println("the dog is moving...");
    }
}

public class main {
    public static void main(string[] args) {
        animal b = new dog();
        b.move();
    }
}
```

### 重载(overload)

## 接口

接口（英文：interface），在java编程语言中是一个抽象类型，是抽象方法的集合，接口通常以interface来声明。一个类通过继承接口的方式，从而来继承接口的抽象方法。

### 接口与类的区别:

- 接口不能用于实例化对象
- 接口没有构造方法
- 接口中所有的方法都必须是抽象方法
- 接口不能包含成员变量，除了`static`，`final`变量
- 接口不是被类继承了，而是要被类实现
- 接口支持多继承

### 接口特性

- 接口是隐式抽象的，当声明一个接口时，不必使用`abstract`关键字
- 接口中的每一个方法也是隐式抽象的，所有字段是隐式的`public static final`，因此没必要在接口内部指定访问说明符

### 接口实现

和抽象类一样，无法创建接口的对象。可以使用`implements`关键字来实现接口

```java
interface polygon {
    void getarea(int length, int breadth);
}

class rectangle implements polygon {
   public void getarea(int length, int breadth) {
       system.out.println("the area is: " + length * breadth);
   }
}

public class main {
    public static void main(string[] args) {
        rectangle r1 = new rectangle();
        r1.getarea(10, 20);
    }
}
```

### 接口中的私有方法和静态方法

与类相似，可以使用其引用访问接口的静态方法

```java
polygon.staticmethod();
```

### 接口中的默认方法

要在接口内使用默认方法，可以使用`default`关键字

```java
public default void getside() {
    // do something...
}
```

具体实现

```java
import java.lang.math;

interface polygon {
    void getarea();

    default void getperimeter(int... sides) {
        int perimeter = 0;
        for (int side : sides) {
            perimeter += side;
        }
        system.out.println("the total sides is: " + perimeter);
    }
}

class trangle implements polygon {
    private int a, b, c;
    private double s, area;

    trangle(int a, int b, int c) {
        this.a = a;
        this.b = b;
        this.c = c;
        s = 0;
    }
    public void getarea() {
        s = (double) (a + b + c) / 2;
        area = math.sqrt(s * (s - a) * (s - b) * (s - c));
        system.out.println("the area is: " + area);
    }
}

public class main {
    public static void main(string[] args) {
        trangle t1 = new trangle(2 , 3, 4);
        t1.getarea();
        t1.getperimeter(2 , 3 , 4);
    }
}
```

### 接口与extends关键字

接口可以继承其他接口

```java

interface line {

}

interface polygon extends line {

}
```

注意一个接口可以继承多个接口

```java

interface a {

}

interface b {

}

interface c extends a, b {

}

```

## java抽象类和抽象方法

### 抽象类

java抽象类是无法实例化的类，使用`abstract`声明

```java
abstract class animal {
    // doing something...
}
```

### 抽象方法

依然使用`abstract`关键字来创建抽象方法

```java
abstract void makesound();
```

### 抽象类的继承

```java
abstract class animal {
    public void displayinfo() {
        system.out.println("i am a animal");
    }
}

class dog extends animal {
}

public class main {
    public static void main(string[] args) {
        dog d1 = new dog();
        d1.displayinfo();
    }
}
```

输出如下

```
i am a animal
```

### 重写抽象方法

在java中，必须在子类重写父类的抽象方法。如果子类也被声明为抽象，则不必强制重写抽象方法

```java
abstract class animal {
    abstract void makesound();
    public void eat() {
        system.out.println("i can eat...");
    }
}

class dog extends animal {
    public void makesound() {
        system.out.println("barking...");
    }
}

public class main {
    public static void main(string[] args) {
        dog d1 = new dog();
        d1.eat();
        d1.makesound();
    }
}
```

### 访问抽象类的构造函数

依旧使用`super()`关键字访问父类

```java

abstract class animal {
    animal() {
        // doing something...
    }
}

class dog extends animal {
    dog() {
        super();
    }
}

```

### java抽象实例

```java
abstract class animal {
    abstract void makesound();
}

class dog extends animal {
    public void makesound() {
        system.out.println("barking...");
    }
}

class cat extends animal {
    public void makesound() {
        system.out.println("meows...");
    }
}

public class main {

    public static void test(animal... ani) {
        if(ani.length <= 0) {
            return;
        }
        for(animal val : ani) {
            val.makesound();
        }
    }

    public static void main(string[] args) {
        dog d1 = new dog();
        cat c1 = new cat();

        test(d1, c1);
    }
}

```

### 总结

- 为了实现抽象类的功能，我们从其继承子类并创建该子类的对象。
- 类必须重写抽象类的所有抽象方法。但是，如果子类被声明为抽象的，则不必强制重写抽象方法。

## 嵌套静态类

与常规类一样，静态嵌套类可以同时包含静态和非静态方法

```java
class animal {
    static class mammal {

    }
}
```

例子

```java

class animal {
    // 内部类
    class reptile {
        public void displayinfo() {
            system.out.println("i am a reptile...");
        }
    }

    // 静态类
    static class mammal {
        public void displayinfo() {
            system.out.println("i am a mammal...");
        }
    }
}

public class main {
    public static void main(string[] args) {
        //创建外部类对象
        animal ani = new animal();
        //非静态类的对象创建
        animal.reptile rep = ani.new reptile();
        rep.displayinfo();
        //静态嵌套类的对象创建
        animal.mammal ma = new animal.mammal();
        ma.displayinfo();
    }
}
```

在上面的实例中  
关于非静态类reptile的创建，使用

```java
        animal.reptile rep = ani.new reptile();
```

关于静态类mammal的创建，使用

```java
        animal.mammal ma = new animal.mammal();
```

### 访问外部类的成员


静态嵌套类只能访问外部类的成员（静态字段和方法）


示例：访问非静态成员

```java
class animal {
    // 内部类
    class reptile {
        public void displayinfo() {
            system.out.println("i am a reptile...");
        }
    }

    // 静态类
    static class mammal {
        public void displayinfo() {
            system.out.println("i am a mammal...");
        }
    }

    public void eat() {
        system.out.println("eating...");
    }
}

public class main {
    public static void main(string[] args) {
        //创建外部类对象
        animal ani = new animal();
        //非静态类的对象创建
        animal.reptile rep = ani.new reptile();
        rep.displayinfo();
        //静态嵌套类的对象创建
        animal.mammal ma = new animal.mammal();
        ma.displayinfo();
        ma.eat();
    }
}
```

```输出
/home/encounter/code/java/learning/src/main.java:31:11
java: 找不到符号
  符号:   方法 eat()
  位置: 类型为animal.mammal的变量 ma

```
在上面的实例中，animal类中创建了一个非静态的方法eat()，由于ma是静态类的对象，因此无法从静态类访问非静态方法

### 静态顶级类

只有嵌套类可以是静态的，不能有静态的顶级类

```java
static class animal {
    public void displayinfo() {
        system.out.println("i am an animal");
    }
}
public class main {
    public static void main(string[] args) {
        animal.displayinfo();
    }
}

```

```输出
/home/encounter/code/java/learning/src/main.java
java: 此处不允许使用修饰符static
```

### 匿名类

在java中，一个类可以包含另一个称为嵌套类的类。可以在不提供任何名称的情况下创建嵌套类。

***没有任何名称的嵌套类成为匿名类***

必须在另一个类定一个匿名类，因此也被成为匿名内部类

```java
class outerclass {
    // define
    object1 = new type(paramenterlist) {
        // doing something...
    };
}
```
匿名类通常继承子类或实现接口  
在这里, **类型(type)**可以是  

1. 匿名类继承的父类
2. 匿名实现的接口

实例：匿名内部类继承类  

```java
class polygon {
    public void display() {
        system.out.println("in class of polygon");
    }
}

class anonymousdemo {
    public void createclass() {
        // 创建匿名类，继承类polygon
        polygon p1 = new polygon() {
            public void display() {
                system.out.println("in class of anonymousdemo");
            }
        };
        p1.display();
    }
}

public class main {
    public static void main(string[] args) {
        anonymousdemo any = new anonymousdemo();
        any.createclass();
    }
}

```

输出

```
in class of anonymousdemo
```
在上面的实例中，创建了一个类`polygon`，只有一个方法`display()`，然后创建了一个匿名类，该类继承类`polygon`并重写了`display()`方法，在运行时，将创建一个类p1,然后调用p1的`display()`方法


实例：实现接口的匿名类

```java
interface polygon {
    public void display();
}

class anonymousdemo {
    public void createclass() {
        polygon p1 = new polygon() {
            public void display() {
                system.out.println("in class of anonymousdemo");
            }
        };
        p1.display();
    }
}

public class main {
    public static void main(string[] args) {
        anonymousdemo any = new anonymousdemo();
        any.createclass();
    }
}
```

输出

```
in class of anonymousdemo

```

### 匿名类的优点

在匿名类中，只需要创建对象。即创建对象执行某些任务

```java
object = new example() {
    public void display() {
        system.out.println("overwrite display()");
    }
}
```

