+++
title = "protobuf的配置和使用"
tags = [ "C/C++", "网络编程", "socket", "计算机网络", "Linux", "protobuf",]
categories = [ "技术",]
mathjax = false
date = "2024-08-01T10:59:34+08:00"
+++
# protobuf简介

Protocol Buffers（简称 protobuf）是 Google 开发的一种数据交换格式。它是一种灵活、高效且自动化的结构化数据序列化方法，类似于 XML、JSON 和其他配置文件格式，但更小、更快、更简单。我们的逻辑是有类等抽象数据构成的，而tcp是面向字节流的，我们需要将类结构序列化为字符串来传输。

主要特点：

1. 语言无关：protobuf 支持多种编程语言，包括 C++、Java、Python 等，并且可以轻松地在不同语言之间进行通信。
2. 平台无关：可以跨多个平台使用，无论是在 32 位还是 64 位系统上。
3. 效率高：相比于 XML 或 JSON，protobuf 在序列化和反序列化时的性能更好，生成的数据也更紧凑。

# 编译protobuf

如果是linux系统，可以直接从自己的包管理器下载protobuf

例如`archlinux`

```
yay -S protobuf
```

如果是windows系统，我们需要从[官网](https://github.com/protocolbuffers/protobuf/releases)下载源代码进行编译

[具体教程](https://blog.csdn.net/weixin_42968757/article/details/120033598)

当protobuf编译完成后，我们可以通过`protobuf --version`来检查是否安装成功

# 使用流程

- 定义消息格式：首先需要定义消息的结构，这通常是通过 .proto 文件完成的。这些文件描述了你想要交换的数据的结构。

我们先创建一个`msg.proto`文件，并且写入如下内容

```proto
    syntax = "proto3";
    message Book
    {
       string name = 1;
       int32 pages = 2;
       float price = 3;
    }
```

这个文件用来定义我们需要发送的信息

- 编译 .proto 文件：使用 Protocol Buffers 编译器（protoc），根据 .proto 文件生成特定语言的源代码

```
protoc --cpp_out=. ./msg.proto
```

./msg.proto 表示msg.proto所在的位置，因为我们是在msg.proto所在文件夹中执行的protoc命令,所以是当前路径即可。
执行后，会看到当前目录生成了msg.pb.h和msg.pb.cc两个文件，这两个文件就是我们要用到的头文件和cpp文件。

- 序列化与反序列化：使用生成的类来创建消息对象，并将这些对象序列化为字节流，或者从字节流中反序列化为对象。

这里我们写一个测试函数

```c++
void test_protobuf() {
    Book book;
    book.set_name("CPP programing");
    book.set_pages(100);
    book.set_price(200);
    std::string bookstr;
    book.SerializeToString(&bookstr);
    std::cout << "serialize str is " << bookstr << std::endl;
    Book book2;
    book2.ParseFromString(bookstr);
    std::cout << "book2 name is " << book2.name() << " price is "
        << book2.price() << " pages is " << book2.pages() << std::endl;
    getchar();

}

```

输出如下：

```
serialize str is
CPP programingdHC
book2 name is CPP programing price is 200 pages is 100
```

上面的demo中将book对象先序列化为字符串，再将字符串反序列化为book2对象。

# 在网络中的应用

先为服务器定义一个用来通信的protobuf

```proto
syntax = "proto3";
message MsgData
{
    int32 id = 1;
    string data = 2;
}
```

id代表消息的编号，data代表消息的内容

接着修改服务器接收和发送数据的逻辑  
当服务器收到数据并完成切包处理，将信息反序列化为具体要使用的结构，打印相关信息，然后再发送给客户端

```c++
Data msgdata;
std::string receive_data;
msgdata.ParseFromString( std::string(
    _recv_msg_node->_msg, _recv_msg_node->_total_len ) );
std::cout << "Recv msg is: " << msgdata.data() << std::endl;
std::string return_str =
    "server has receive msg, msg data is  " + msgdata.data();
Data msgreturn;
msgreturn.set_id( msgdata.id() );
msgreturn.set_data( msgdata.data() );
msgreturn.SerializeToString( &return_str );
Send( return_str );

```

同样，客户端在发送的时候也利用protobu进行消息序列化，然后发送给服务器

```c++
Data msgdata;
msgdata.set_id( 1001 );
msgdata.set_data( "Hello world!" );
std::string request;
msgdata.SerializeToString( &request );
std::cout << "message id: " << msgdata.id()
          << " content: " << msgdata.data() << std::endl;
boost::asio::write(
    sock, boost::asio::buffer( request, request.length() ) );

```

# 关于protbuf在CMake中的配置

> 注意一定要在编译参数中加入`-Wl,--copy-dt-needed-entries`，否则会报[DSO missing](https://linuxpip.org/how-to-fix-dso-missing-from-command-line)

```cmake
set(CMAKE_CXX_FLAGS
    "${CMAKE_CXX_FLAGS} -g -O2  -Wall -Wextra -Weffc++ -Werror=uninitialized  -Werror=return-type -Wconversion -Wsign-compare -Werror=unused-result -Werror=suggest-override -Wzero-as-null-pointer-constant -Wmissing-declarations -Wold-style-cast -Wnon-virtual-dtor -Wl,--copy-dt-needed-entries"
)

```

关于对proto文件的配置，可以参考这个

```cmake
# 获取编译器
find_program(
    PROTOC_CXX
    protoc
    DOC "Protobuf Compiler (protoc)"
    REQUIRED
)

# 需要编译的 proto 文件
file (GLOB PROTO_SOURCE_FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/*.proto"
)

set(PROTO_PATH    "${CMAKE_CURRENT_SOURCE_DIR}")
set(PROTO_CXX_OUT "${CMAKE_CURRENT_BINARY_DIR}/gen_cxx")

file(MAKE_DIRECTORY ${PROTO_CXX_OUT})

# 使用 protoc 处理 proto 文件
foreach(input_proto ${PROTO_SOURCE_FILES})
    get_filename_component(DIR ${input_proto} DIRECTORY)
    get_filename_component(FILE_NAME ${input_proto} NAME_WE)


    set(OUTPUT_CXX_HEADER   "${PROTO_CXX_OUT}/${FILE_NAME}.pb.h")
    set(OUTPUT_CXX_SOURCE   "${PROTO_CXX_OUT}/${FILE_NAME}.pb.cc")
    list(APPEND OUTPUT_SOURCES_CXX
        ${OUTPUT_CXX_HEADER} ${OUTPUT_CXX_SOURCE})
endforeach()


add_custom_command(
    OUTPUT  ${OUTPUT_SOURCES_CXX}
    COMMAND ${PROTOC_CXX} --cpp_out=${PROTO_CXX_OUT} --proto_path=${PROTO_PATH} ${PROTO_SOURCE_FILES}
    DEPENDS ${PROTO_SOURCE_FILES}
    WORKING_DIRECTORY ${PROTO_PATH}
    COMMENT "Generate Cpp Protobuf Source Files"
)

add_custom_target(
    compile_cxx_protos
    DEPENDS ${OUTPUT_SOURCES_CXX}
)

# 设置生成源文件包含目录变量供上层引用
set(PROTO_GEN_CXX_INCLUDE_DIRS ${PROTO_CXX_OUT} PARENT_SCOPE)

# 将生成的文件打包为库 proto_gen_cxx
# 程序可以链接到该库

add_library(proto_gen_cxx ${OUTPUT_SOURCES_CXX})
target_link_libraries(proto_gen_cxx protobuf)
add_dependencies(proto_gen_cxx compile_cxx_protos)

```
