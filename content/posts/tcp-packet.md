+++
title = "tcp粘包问题"
tags = [ "C/C++", "网络编程", "socket", "计算机网络", "Linux"]
categories = [ "技术"]
date = "2024-07-26T21:11:21+08:00"
+++
# 什么是粘包

TCP粘包问题是指在使用TCP协议进行网络通信时，客户端和服务器之间发送的数据包可能会被TCP协议栈在底层进行合并或者拆分，导致客户端接收到的数据不再是单独、完整的数据包，而是多个数据包的内容被粘在一起或者多个数据包的内容被拆分到不同的接收缓冲区中。

# 粘包原因

## 1. 因为TCP是面向字节流的协议

传输的数据是以流的形式,而流数据是没有明确的开始结尾边界,所以 TCP 也没办法判断哪一段流属于一个消息;TCP协议是流式协议;所谓流式协议,即协议的内容是像流水一样的字节流,内容与内容之间没有明确的分界标志,需要认为手动地去给这些协议划分边界。

例如客户端每次发送N个字节给服务端，N取决于当前客户端的发送缓冲区是否有数据，比如发送缓冲区总大小为10个字节，当前有5个字节数据（例如上次要发送的数据‘loveu’）未发送完，那么此时只有5个字节的空闲空间，客户端调用发送接口发送“hello world!”其实就是只能发送“hello”给服务器，那么服务器一次性得到的数据就是“loveuhello”，而剩余的“world！”只能留给下一次发送，下一次服务器收到的就是“world！”

![tlv1.png](https://s2.loli.net/2024/07/30/FLGRpuP6DyoK4vQ.png)

## 2. 数据发送和接收速率不匹配

如果发送方发送数据的速度比接收方处理数据的速度快，就可能导致多个消息被一次性读取。比如客户端1s内发送了两次“hello world！”，服务器过了2s才接收到数据，那一次性就会读出两个“hello world”

## 3. tcp底层的安全和效率机制不允许字节数特别少的小包发送频率过高

tcp会在底层累计数据长度到一定大小才一起发送，比如连续发送1字节的数据要累计到多个字节才发送，可以了解下tcp底层的[Nagle算法](https://blog.csdn.net/m0_61567378/article/details/130886149)。

# 处理粘包

处理粘包的方式主要采用应用层定义收发包格式的方式，这个过程俗称切包处理，常用的协议被称为`tlv`协议(消息id+消息长度+消息内容)

![tlv1.png](https://s2.loli.net/2024/07/26/3q8MRKOo9YD7z4S.png)

为了方便理解，这里先简化发送格式，改成“消息长度+消息内容”的方式

![tlvsimple.png](https://s2.loli.net/2024/07/26/blhFoOsUPANJQa2.png)

## 消息节点

```c++
#define MAX_LENGTH 1024 * 2
#define HEAD_LENGTH 2

class MsgNode {
public:
    friend class Session;
    MsgNode( char *msg, short max_len )
        : _cur_len( 0 ), _total_len( max_len + HEAD_LENGTH ) {
        _msg = new char[_total_len + 1]();     // 这里➕1是为了存放'\0'
        memcpy( _msg, &max_len, HEAD_LENGTH ); // 留出两个字节存储消息头
        mempcpy( _msg + HEAD_LENGTH, msg, max_len ); // 存储消息体
        _msg[_total_len] = '\0';
    }
    MsgNode( short max_len ) : _cur_len( 0 ), _total_len( max_len ) {
        _msg = new char[_total_len + 1]();
    }

    void clear() {
        memset( _msg, 0, _total_len );
        _cur_len = 0;
    }

    ~MsgNode() { delete[] _msg; }

private:
    short _cur_len;   // 当前已处理的数据长度
    short _total_len; // 数据的总长度
    char *_msg;       // 存储的数据指针
};

```

## Session的改进

为了能够对收到的数据进行切包处理，需要定义一个消息接收节点、一个bool变量表示头部信息是否处理完成，以及将处理好的头部先缓存起来的结构

```c++
    std::shared_ptr<MsgNode> _recv_msg_node;  //收到消息结构
    bool _b_head_parse;                       //是否处理完头部信息
    std::shared_ptr<MsgNode> _recv_head_node; //收到头部结构
```

## 完善接收逻辑

```c++
void Session::HandleRead( const boost::system::error_code &error,
    size_t bytes_transferred, std::shared_ptr<Session> _self_shared ) {
    if ( !error ) {

        PrintRecvData(_data, bytes_transferred);
        std::chrono::seconds duration(2);
        std::this_thread::sleep_for(duration);
        // 已经移动的字符串
        int copy_len = 0;
        while ( bytes_transferred > 0 ) {
            // 判断头部是否处理
            if ( !_b_head_parse ) {
                // NOTE: step 1
                // 如果数据小于头部大小，先将数据放入_recv_head_node
                if ( bytes_transferred + _recv_head_node->_cur_len <
                     HEAD_LENGTH ) {
                    memcpy( _recv_head_node->_msg + _recv_head_node->_cur_len,
                        _data + copy_len, bytes_transferred );
                    _recv_head_node->_cur_len += bytes_transferred;
                    memset( _data, 0, MAX_LENGTH );
                    _socket.async_read_some(
                        boost::asio::buffer( _data, MAX_LENGTH ),
                        std::bind( &Session::HandleRead, this,
                            std::placeholders::_1, std::placeholders::_2,
                            _self_shared ) );
                    return;
                }

                // NOTE: step 2
                // 收到的数据比头部多，可能是多个逻辑包，要做切包处理
                // 头部剩余未复制的长度
                int head_remain = HEAD_LENGTH - _recv_head_node->_cur_len;
                memcpy( _recv_head_node->_msg + _recv_head_node->_cur_len,
                    _data + copy_len, head_remain );

                // 更新已处理的data长度和剩余未处理长度
                copy_len += head_remain;
                bytes_transferred -= head_remain;
                // 获取头部数据
                short data_len = 0;
                memcpy( &data_len, _recv_head_node->_msg, HEAD_LENGTH );
                std::cout << "data_len is: " << data_len << std::endl;

                // 头部非法长度
                if ( data_len > HEAD_LENGTH ) {
                    std::cout << "Invalid data length is: " << data_len
                              << std::endl;
                    _server->ClearSession( _uuid );
                    return;
                }

                _recv_msg_node = std::make_shared<MsgNode>( data_len );

                // NOTE: step 3
                // 消息的长度小于头部规定的长度，说明数据未收集全，则先将部分消息放到接收节点里
                if ( (int)bytes_transferred < data_len ) {
                    memcpy( _recv_msg_node->_msg + _recv_msg_node->_cur_len,
                        _data + copy_len, bytes_transferred );
                    _recv_msg_node->_cur_len += bytes_transferred;
                    memset( _data, 0, MAX_LENGTH );
                    _socket.async_read_some(
                        boost::asio::buffer( _data, MAX_LENGTH ),
                        std::bind( &Session::HandleRead, this,
                            std::placeholders::_1, std::placeholders::_2,
                            _self_shared ) );
                    // 头部处理完成
                    _b_head_parse = true;
                    return;
                }

                memcpy( _recv_msg_node->_msg + _recv_msg_node->_cur_len,
                    _data + copy_len, data_len );

                _recv_msg_node->_cur_len += data_len;
                copy_len += data_len;
                bytes_transferred -= data_len;
                _recv_msg_node->_msg[_recv_msg_node->_total_len] = '\0';
                std::cout << "Recv msg is: " << _recv_msg_node->_msg
                          << std::endl;
                // use Send for testing
                Send( _recv_msg_node->_msg, _recv_msg_node->_total_len );
                // 继续轮询未处理的数据
                _b_head_parse = false;
                _recv_head_node->clear();
                if ( bytes_transferred <= 0 ) {
                    memset( _data, 0, MAX_LENGTH );
                    _socket.async_read_some(
                        boost::asio::buffer( _data, MAX_LENGTH ),
                        std::bind( &Session::HandleRead, this,
                            std::placeholders::_1, std::placeholders::_2,
                            _self_shared ) );
                    return;
                }
                continue;
            }

            // NOTE: step 4
            // 已经处理完头部，处理上次未接收完的消息数据
            // 接收的数据仍不足剩余未处理的
            int remain_msg =
                _recv_msg_node->_total_len - _recv_msg_node->_cur_len;
            if ( (int)bytes_transferred < remain_msg ) {
                memcpy( _recv_msg_node->_msg + _recv_msg_node->_cur_len,
                    _data + copy_len, bytes_transferred );
                _recv_msg_node->_cur_len += bytes_transferred;
                memset( _data, 0, MAX_LENGTH );
                _socket.async_read_some(
                    boost::asio::buffer( _data, MAX_LENGTH ),
                    std::bind( &Session::HandleRead, this,
                        std::placeholders::_1, std::placeholders::_2,
                        _self_shared ) );
                return;
            }

            memcpy( _recv_msg_node->_msg + _recv_msg_node->_cur_len,
                _data + copy_len, remain_msg );
            bytes_transferred -= remain_msg;
            copy_len += remain_msg;
            _recv_msg_node->_msg[_recv_msg_node->_total_len] = '\0';
            std::cout << "Recv msg is: " << _recv_msg_node->_msg << std::endl;
            // use Send for testing
            Send( _recv_msg_node->_msg, _recv_msg_node->_total_len );
            // 继续轮询未处理的数据
            _b_head_parse = false;
            _recv_head_node->clear();
            if ( bytes_transferred <= 0 ) {
                _socket.async_read_some(
                    boost::asio::buffer( _data, MAX_LENGTH ),
                    std::bind( &Session::HandleRead, this,
                        std::placeholders::_1, std::placeholders::_2,
                        _self_shared ) );
                return;
            }
            continue;
        }
    } else {
        std::cout << "handle read failed, code is: " << error.value()
                  << " message is: " << error.message() << std::endl;
        _server->ClearSession( _uuid );
    }
}

```


1. `copy_len`：已经处理的数据长度，因为存在一次接收多个包的情况，所以copy_len的意义是在于记录已经处理的数据的长度

2. 首先判断`_b_head_parse`是否为`false`，如果为`false`，则表示头部未处理，需要先处理头部。先判断接收的数据是否小于`HEAD_LENGTH`，如果小于则需要拷贝数据到`_recv_head_node`中，然后再读取剩余的数据。

3. 如果受到的数据比头部数据多，可能是多个数据包，需要做切包处理。根据之前保留在`_recv_head_node`中的数据长度，计算出剩余未读取的头部长度，然后取出剩余头部长度保存在`_recv_head_node`中。然后通过`memcpy`从节点拷贝出数据写入short类型的`data_len`，并更新`copy_len`，进而得到消息长度， 然后再读取剩余的消息体。先判断接收到数据未处理部分的长度和总共要接收的数据长度大小，如果小于总共要接收的长度，说明消息体还没接收完，则将未处理的部分写入到`_recv_msg_node`里，回调读事件。否则说明消息体接收完全

4. 将消息体数据接收到`_recv_msg_node`中，接收完全后返回给对端。当然存在多个逻辑包粘连，此时要判断`bytes_transferred`是否<=0，如果是则说明只有一个逻辑包，我们处理完了，继续监听读事件，就直接返回即可，否则说明有多个数据包粘连，就继续执行上述操作

5. 因为存在`_b_head_parse`为`true`，就是包头接收并处理完的情况，但是包体未接收完，则再次出发读事件，此时就要继续进行上述操作

总体流程如下

![dealHandRead.png](https://s2.loli.net/2024/07/26/mr2BgQoTFJe4EYK.png)

## 粘包测试

为了测试粘包，需要制造粘包产生的现象，可以让客户端发送的频率高一些，服务器接收的频率低一些，这样造成前后端收发数据不一致导致多个数据包在服务器tcp缓冲区滞留产生粘包现象。


测试粘包之前，在服务器的`Session`中添加打印二进制函数


```c++
void Session::PrintRecvData(char *data, int length) {
    std::stringstream ss;
    std::string result = "0x";
    for(int i = 0; i < length; i++) {
        std::string hexstr;
        ss << std::hex << std::setw(2) << std::setfill('0') << int(data[i]) << std::endl;
        ss >> hexstr;
        result += hexstr;
    }

    std::cout << "Recv raw data is: " << result << std::endl;
}
```
然后将这个函数放到HandleRead里，每次收到数据就调用这个函数打印接收到的最原始的数据，然后睡眠2秒再进行收发操作，用来延迟接收对端数据制造粘包，之后的逻辑不变


```c++
void Session::HandleRead( const boost::system::error_code &error,
    size_t bytes_transferred, std::shared_ptr<Session> _self_shared ) {
    if ( !error ) {
        
        PrintRecvData(_data, bytes_transferred);
        std::chrono::seconds duration(2);
        std::this_thread::sleep_for(duration);

```


客户端代码实现收发分离


```c++
#include <boost/asio.hpp>
#include <chrono>
#include <iostream>
#include <thread>
using namespace std;
using boost::asio::ip::tcp;
constexpr int MAX_LENGTH  = 1024 * 2;
constexpr int HEAD_LENGTH = 2;

int main() {
    try {
        boost::asio::io_context ioc;
        tcp::endpoint remote_ep(
            boost::asio::ip::address::from_string( "127.0.0.1" ), 10086 );
        tcp::socket sock( ioc );
        boost::system::error_code ec = boost::asio::error::host_not_found;
        sock.connect( remote_ep, ec );
        if ( ec ) {
            cout << "connect failed: " << ec.message() << endl;
            return 0;
        }

        std::thread send_thread( [&]() {
            while ( true ) {
                this_thread::sleep_for( std::chrono::milliseconds( 1 ) );
                const char *request_msg = "Hello World!";

                size_t request_len         = strlen( request_msg );
                char send_data[MAX_LENGTH] = { 0 };
                memcpy( send_data, &request_len, 2 );
                memcpy( send_data + 2, request_msg, request_len );

                boost::asio::write(
                    sock, boost::asio::buffer( send_data, request_len + 2 ) );
            }
        } );

        std::thread recv_thread( [&]() {
            while ( true ) {
                this_thread::sleep_for(std::chrono::milliseconds(1));
                std::cout << "Begin to receive" << std::endl;
                char reply_head[HEAD_LENGTH];
                size_t reply_length = boost::asio::read(
                    sock, boost::asio::buffer( reply_head, HEAD_LENGTH ) );
                short msglen = 0;
                memcpy( &msglen, reply_head, HEAD_LENGTH );
                char msg[MAX_LENGTH] = { 0 };
                size_t msg_length    = boost::asio::read(
                    sock, boost::asio::buffer( msg, msglen ) );
                cout << "Reply is: ";
                cout.write( msg, msg_length ) << endl;
                cout << "Reply length is: " << msg_length << endl;
            }
        } );

        send_thread.join();
        recv_thread.join();

    } catch ( std::exception &e ) {
        cerr << e.what() << endl;
    }

    return 0;
}

```


# 总结

该服务虽然实现了粘包处理，但是服务器仍存在不足，比如当客户端和服务器处于不同平台时收发数据会出现异常，根本原因是未处理大小端模式的问题。
