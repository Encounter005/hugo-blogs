+++
title = "asio socket的创建和连接"
tags = [ "C/C++", "网络编程", "socket", "计算机网络", "Linux",]
categories = [ "技术",]
mathjax = true
date = "2024-07-03T16:22:19+08:00"
+++

## 终端节点的创建

所谓终端节点就是用来通信的端对端的节点，可以通过ip地址和端口构造，其的节点可以连接这个终端节点做通信

如果是客户端，可以通过对端的ip和端口构造一个`endpoint`对象

```c++
int client_endpoint() {
    // 定义需要解析的IP地址字符串和端口号
    std::string raw_ip_address = "127.0.0.1";
    unsigned short port_num   = 8080;

    boost::system::error_code error; // 错误代码容器

    // 使用Boost.Asio库中的ip::address_from_string()函数解析IP地址字符串
    boost::asio::ip::address ip_address =
        boost::asio::ip::address::from_string( raw_ip_address, error );

    // 检查解析过程是否有错误发生，并输出相应的错误信息或返回错误代码
    if (error.value() != 0) {
        std::cout << "Failed to parse IP address. Error code is: "
                  << error.value() << ", Message: "
                  << error.message() << std::endl;
        return error.value(); // 返回错误代码
    }

    // 使用解析的IP地址和端口号构建TCP endpoint对象
    boost::asio::ip::tcp::endpoint ep( ip_address, port_num );

    // 函数返回值，表示操作是否成功执行（如果未抛出异常则假设为0）
    return 0;
}
```

如果是服务端，则只需要本地地址绑定就可以生成`endpoint`

```c++
int server_endpoint() {
    unsigned short port_num = 8080; // 设置服务器监听端口号为8080

    // 使用IPv4地址类型设置一个任意的本地IP地址。这意味着服务器将侦听来自任何合法的IPv4地址。
    boost::asio::ip::address ip_address = boost::asio::ip::address_v4::any();

    // 创建一个TCP端点（endpoint），它将绑定到指定的本地IP地址和端口号上
    boost::asio::ip::tcp::endpoint ep(ip_address, port_num);

    return 0; // 表示函数执行成功，返回值为0
}

```

## 创建socket

1. 创建上下文`io_context`
2. 选择协议
3. 生成`socket`
4. 打开`socket`

```c++
int create_tcp_socket() {
    // 使用boost::asio中的ip模块和tcp类定义协议版本v4的套接字
    using boost::asio::ip::tcp;
    boost::asio::io_context ios; // 创建IO上下文，用于处理异步操作

    tcp protocol = tcp::v4(); // 初始化并配置为IPv4协议

    tcp::socket sock( ios,
        protocol ); // 使用给定的io_context创建一个新套接字，并将其初始化为指定协议版本
    boost::system::error_code ec; // 创建错误代码对象，用于捕获可能发生的异常

    sock.open( protocol, ec ); // 尝试打开一个与指定协议兼容的新套接字

    if ( ec.value() != 0 ) { // 检查是否发生任何错误
        std::cout << "Failed to open the socket! Error code: " << ec.value()
                  << " Message: " << ec.message() << std::endl;
        return ec.value(); // 如果打开套接字失败，返回错误代码作为结果值
    }

    return 0; // 成功的情况下，函数返回0表示成功创建了TCP套接字
}

```

上面的`socket`只是通信的`socket`,如果是服务端，我们还需要生成一个`acceptor`的`socket`,用来接受新的连接

```c++
int create_acceptor_socket() {
    // 使用boost::asio中的ip模块和tcp类定义协议版本v6的套接字接受器（acceptor）
    using boost::asio::ip::tcp;
    boost::asio::io_context ios; // 创建IO上下文，用于处理异步操作

    tcp protocol = tcp::v6(); // 初始化并配置为IPv6协议

    tcp::acceptor acceptor(
        ios ); // 使用给定的io_context创建一个新接受器（acceptor），用于监听连接请求

    boost::system::error_code ec; // 创建错误代码对象，用于捕获可能发生的异常

    // 尝试打开一个与指定协议兼容的新接受器
    acceptor.open( protocol, ec );

    if ( ec.value() != 0 ) { // 检查是否发生任何错误
        std::cout << "Failed to open the socket! Error code: " << ec.value()
                  << " Message: " << ec.message() << std::endl;
        return ec.value(); // 如果打开接受器失败，返回错误代码作为结果值
    }

    return 0; // 成功的情况下，函数返回0表示成功创建了IPv6协议的接受器（acceptor）
}
```

## 绑定acceptor

对于acceptor类型的socket,服务器需要将其绑定到指定的断点，所有连接这个端点的连接都可以被接收到

```c++
int bind_acceptor_socket() {
    // 使用boost::asio中的ip模块和tcp类定义协议版本v4的套接字接受器（acceptor）
    using boost::asio::ip::tcp;

    // 指定要绑定的端口号，这里为8080
    unsigned short port_num = 8080;

    tcp::endpoint endpoint( boost::asio::ip::address_v4::any(),
        port_num ); // 创建一个网络端点对象用于指定IP地址和端口

    // 创建IO上下文，用于处理异步操作

    boost::asio::io_context ios;

    tcp::acceptor acceptor( ios,
        endpoint ); // 使用给定的io_context创建并初始化一个新接受器（acceptor），同时绑定到指定的端点

    boost::system::error_code ec; // 创建错误代码对象，用于捕获可能发生的异常

    // 尝试将接受器（acceptor）绑定到指定的网络端点
    acceptor.bind( endpoint, ec );

    if ( ec.value() != 0 ) { // 检查是否发生任何错误
        std::cout << "Failed to bind the socket! Error code: " << ec.value()
                  << " Message: " << ec.message() << std::endl;

        return ec.value(); // 如果绑定失败，返回错误代码作为结果值
    }

    return 0; // 成功的情况下，函数返回0表示成功绑定了接受器（acceptor）到指定端口
}
```

## 连接指定的端点

作为客户端可以连接服务器指定的端点进行连接

```c++
int connect_to_endpoint() {
    // IP地址字符串和端口号定义
    std::string raw_ip_address = "127.0.0.1";
    unsigned short port_num    = 8080;

    // 使用std::exception类来捕获可能抛出的异常
    using boost::asio::ip::tcp;
    try {
        tcp::endpoint endpoint(boost::asio::ip::address::from_string(raw_ip_address), port_num);

        // 创建IO上下文，用于处理异步操作

        boost::asio::io_context ios;

        // 创建socket并初始化为指定协议版本的套接字
        tcp::socket soc(ios, endpoint.protocol());

        // 尝试连接到指定端点（endpoint）
        soc.connect(endpoint);
    } catch (std::exception &e) {
        // 如果在尝试连接时抛出异常，捕获它并输出错误信息

        std::cerr << e.what() << std::endl;

        return -1;  // 返回一个非零值表示连接失败
    }

    return error.value(); // 函数返回0表示成功建立了与指定端点的连接
}
```

## 服务器接收连接

当有客户端连接时，服务器需要接收连接

```c++
int accept_new_connection() {
    // 指定要监听的端口号，这里为8080
    unsigned short port_num = 8080;

    using boost::asio::ip::
        tcp; // 使用ip模块和tcp类定义协议版本v4的套接字接受器（acceptor）

    tcp::endpoint endpoint( boost::asio::ip::address_v4::any(),
        port_num ); // 创建一个网络端点对象用于指定IP地址为任何可用地址和指定的端口

    // 创建IO上下文，用于处理异步操作

    boost::asio::io_context ios;

    try {
        tcp::acceptor acceptor( ios,
            endpoint.protocol() ); // 使用给定的io_context创建并初始化一个新接受器（acceptor），同时绑定到指定的端点

        // 设置最大连接队列长度为BACKLOG
        acceptor.listen( BACKLOG );

        // 创建socket并初始化为指定协议版本的套接字
        tcp::socket socket( ios, endpoint.protocol() );

        // 接受新的客户端连接请求，并将其绑定到新的socket对象上
        acceptor.accept( socket );
    } catch ( std::exception &e ) {
        // 如果在处理过程中抛出异常，捕获并输出错误信息

        std::cerr << e.what() << std::endl;

        return -1; // 返回一个非零值表示接受新连接失败
    }

    return 0; // 成功接收新的客户端连接后返回0作为结果值
}
```

## buffer

buffer就是用来接收和发送数据时缓存数据的接口

`boost::asio提`供了`asio::mutable_buffer`和`asio::const_buffer`两种类型的buffer，他们是一段连续的空间，首字节存储了后续数据的长度。`asio::mutable_buffer`用于写服务，`asio::const_buffer`用于读服务。但是着这两个结构都没有被asio的api直接使用

对于api的buffer参数，asio提出了`MutableBufferSequence`和`ConstBufferSequence`两种类型的buffer参数，他们是由多个`asio::mutable_buffer`和多个`asio::const_buffer`组成的容器。`MutableBufferSequence`和`ConstBufferSequence`的具体类型是`boost::asio::mutable_buffer`和`boost::asio::const_buffer`。也就是说`boost::asio`为了节省空间，将一部分连续空间组合起来，交给api使用。可以理解为`MutableBufferSequence`的数据结构为`std::vector<boost::asio::mutable_buffer>`，`ConstBufferSequence`的数据结构为`std::vector<boost::asio::const_buffer>`

`std::vector<boost::asio::mutable_buffer>`的结构如下

![mutable_buffer.png](https://s2.loli.net/2024/07/03/kqJpCB9e13wNYVx.png)

每个vector存储的都是`mutable_buffer`的地址，每个`mutable_buffer`的第一个字节表示数据的长度，后面跟着数据内容。

这么复杂的结构交给用户使用并不合适，所以asio提出了buffer()函数，该函数接收多种形式的字节流，该函数返回`asio::mutable_buffers_1`或者`asio::const_buffers_1`结构的对象。

如果传递给buffer()的参数是一个只读类型，则函数返回`asio::const_buffers_1 `类型对象。

如果传递给buffer()的参数是一个可写类型，则返回`asio::mutable_buffers_1 `类型对象。

`asio::const_buffers_1`和`asio::mutable_buffers_1`是`asio::mutable_buffer`和`asio::const_buffer`的适配器，提供了符合`MutableBufferSequence`和`ConstBufferSequence`概念的接口，所以他们可以作为boost::asio的api函数的参数使用。

总的来说，我们可以用buffer()函数来生成我们要使用的缓存存储数据  
比如boost的发送接口send要求的参数为`ConstBufferSequence`类型

```c++
template<typename ConstBufferSequence>
std::size_t send(const ConstBufferSequence &buffers);
```

将"Hello World"转换成这种类型

```c++
void use_const_buffer() {
    std::string buf = "Hello World";
    boost::asio::const_buffer asio_buf(buf.c_str(), buf.length());
    std::vector<boost::asio::const_buffer> buffers;
    buffers.push_back(asio_buf);
}
```

现在buffers就是可以传递给接口send的类型，但是这样太复杂了，可以直接使用buffer函数转为为send所需要的类型

```c++
void use_buffer_str() {
    asio::const_buffer asio_buf("Hello World");
}
```

asio_buf可以直接传递给send接口。也可以将数组转化为send接受的类型

```c++
void use_buffer_array() {
    const size_t BUF_SIZE = 20;
    std::unique_ptr<char[]> buf(new char[BUF_SIZE]);
    auto input_buf = boost::asio::buffer(static_cast<void*>(buf.get()), BUF_SIZE);
}
```

对于流式操作，可以用`streambuf`，将输入输出流和`streambuf`组合起来，可以实现流式输入和输出

```c++
void use_stream_buffer() {
    // 创建一个 asio::streambuf 对象来存储和处理流操作。
    asio::streambuf buf;
    
    // 将 std::ostream 与作为参数传递给 `operator<<` 的 streambuf 关联起来，用于写入数据。
    std::ostream output(&buf);
    
    // 向输出流中写入两条消息，并以换行符分隔。
    output << "Message1\n";
    output << "Message2";
    
    // 创建一个 std::istream 与 streambuf 相关联，以便从其中读取数据。
    // 注意：这里传递的 `&buf` 实际上是作为 `std::streambuf*` 的指针，而不是 `asio::streambuf` 类型。这是一个小错误示例代码中的注释可能未能正确地指出这一点。
    
    std::istream input(&buf);
    
    // 定义一个字符串变量用于存储读取的数据，并使用 getline() 函数从输入流中读取数据直到遇到换行符 '\n'。
    std::string message1;
    std::getline(input, message1);
    
    // 现在，`message1` 变量将包含读取的 "Message1" 字符串（因为 getline() 函数停止在遇到下一个换行符之前）。
}
```
