+++
title = "socket的创建和连接"
tags = [ "C/C++", "网络编程", "socket", "计算机网络", "Linux",]
categories = [ "技术",]
date = "2024-06-30T00:51:53+08:00"
+++
# socket编程

## socket的概念

Socket（套接字）是计算机网络编程中的一个基本概念，它为应用程序提供了访问底层网络协议的接口，使得进程间能够通过网络进行通信。在更广泛的意义上，Socket 是操作系统提供的一个抽象层，用于简化网络编程并隐藏复杂的网络协议细节。

Socket 是两个进程间通信的端点，每个 Socket 都有唯一的地址，这个地址包括 IP 地址和端口号。IP 地址用于定位网络上的主机，而端口号则用于区分同一台主机上的不同服务。

## socket的类型

socket 可以分为几种类型，主要依据它们支持的协议和通信模式：

- **流式 Socket (SOCK_STREAM)：** 基于 TCP 协议，提供面向连接的服务，保证数据的顺序传输，且不会丢失数据。
- **数据报 Socket (SOCK_DGRAM)：** 基于 UDP 协议，提供无连接服务，数据报文独立传输，不保证数据的顺序和完整性，但传输效率较高。
- **原始 Socket (SOCK_RAW)：** 直接访问 IP 层，允许应用程序直接处理 IP 数据包，常用于网络分析或特殊用途的软件。

## socket提供的函数

- `socket()` 创建一个新的确定类型的套接字，类型用一个整型数值标识，并为它分配系统资源。
- `bind()` 一般用于服务器端，将一个套接字与一个套接字地址结构相关联，比如，一个指定的本地端口和IP地址。
- `listen()` 用于服务器端，使一个绑定的TCP套接字进入监听状态。
- `connect()`用于客户端，为一个套接字分配一个自由的本地端口号。 如果是TCP套接字的话，它会试图获得一个新的TCP连接。
- `accept()`用于服务器端。 它接受一个从远端客户端发出的创建一个新的TCP连接的接入请求，创建一个新的套接字，与该连接相应的套接字地址相关联。
- `send()`和`recv()`,或者`write()`和`read()`,或者`recvfrom()`和`sendto()`,用于往/从远程套接字发送和接受数据。
- `close()`用于系统释放分配给一个套接字的资源。 如果是TCP，连接会被中断。
- `gethostbyname()`和`gethostbyaddr()
- `select()`用于修整有如下情况的套接字列表： 准备读，准备写或者是有错误。
- `poll()`用于检查套接字的状态。 套接字可以被测试，看是否可以写入、读取或是有错误。
- `getsockopt()`用于查询指定的套接字一个特定的套接字选项的当前值。
- `setsockopt()`用于为指定的套接字设定一个特定的套接字选项。

### 1. `socket()`

- **作用**：创建一个新的 Socket。
- **参数**：
  - `int domain`：地址族，如 `AF_INET`（IPv4）或 `AF_INET6`（IPv6）。
  - `int type`：Socket 类型，如 `SOCK_STREAM`（面向连接，流式）或 `SOCK_DGRAM`（无连接，数据报）。
  - `int protocol`：协议，通常是 `0`，表示使用与 `type` 关联的默认协议。

### 2. `bind()`

- **作用**：将 Socket 与本地地址（IP 地址和端口号）相关联。
- **参数**：
  - `int sockfd`：Socket 文件描述符。
  - `const struct sockaddr *addr`：指向包含地址信息的 `sockaddr` 结构体的指针。
  - `socklen_t addrlen`：`sockaddr` 结构体的长度。

### 3. `listen()`

- **作用**：使 Socket 准备接收连接，通常用于服务器端。
- **参数**：
  - `int sockfd`：Socket 文件描述符。
  - `int backlog`：待处理连接请求的最大队列长度。

### 4. `accept()`

- **作用**：接受传入的连接请求，并返回一个新的 Socket 文件描述符用于通信。
- **参数**：
  - `int sockfd`：监听的 Socket 文件描述符。
  - `struct sockaddr *addr`：可选参数，用于存储客户端的地址信息。
  - `socklen_t *addrlen`：可选参数，用于存储地址信息的长度。

### 5. `connect()`

- **作用**：初始化与远程主机的连接，通常用于客户端。
- **参数**：
  - `int sockfd`：Socket 文件描述符。
  - `const struct sockaddr *addr`：指向包含远程主机地址信息的 `sockaddr` 结构体的指针。
  - `socklen_t addrlen`：`sockaddr` 结构体的长度。

### 6. `send()`

- **作用**：发送数据到已连接的 Socket。
- **参数**：
  - `int sockfd`：Socket 文件描述符。
  - `const void *buf`：指向要发送数据的缓冲区的指针。
  - `size_t len`：要发送的数据长度。
  - `int flags`：发送标志，如 `MSG_DONTROUTE`。

### 7. `recv()`

- **作用**：从 Socket 接收数据。
- **参数**：
  - `int sockfd`：Socket 文件描述符。
  - `void *buf`：接收数据的缓冲区。
  - `size_t len`：缓冲区的大小。
  - `int flags`：接收标志，如 `MSG_PEEK`。

### 8. `sendto()`

- **作用**：向特定地址发送数据，通常用于无连接的 Socket（如 UDP）。
- **参数**：与 `send()` 类似，额外包括目标地址和地址长度。

### 9. `recvfrom()`

- **作用**：接收数据并返回源地址信息，通常用于无连接的 Socket（如 UDP）。
- **参数**：与 `recv()` 类似，额外包括源地址信息和地址长度。

### 10. `close()`

- **作用**：关闭 Socket 文件描述符，释放资源。
- **参数**：
  - `int sockfd`：要关闭的 Socket 文件描述符。

### 11. `setsockopt()`

- **作用**：设置 Socket 的选项，如超时、重用地址等。
- **参数**：
  - `int sockfd`：Socket 文件描述符。
  - `int level`：设置选项的级别，如 `SOL_SOCKET` 或 `IPPROTO_TCP`。
  - `int optname`：选项名称。
  - `const void *optval`：指向选项值的指针。
  - `socklen_t optlen`：选项值的长度。

### 12. `getsockopt()`

- **作用**：获取 Socket 的选项。
- **参数**：与 `setsockopt()` 类似。

### 13. `shutdown()`

- **作用**：关闭 Socket 的读取或写入方向。
- **参数**：
  - `int sockfd`：Socket 文件描述符。
  - `int how`：关闭的方向，如 `SHUT_RD`（读取方向）或 `SHUT_WR`（写入方向）。

### 注意事项

- 使用 Socket 编程时，应确保正确处理错误情况，检查每个函数的返回值，并适当地处理任何错误代码。
- 对于某些函数，如 `send()` 和 `recv()`，如果在非阻塞模式下使用，可能需要处理 EAGAIN 或 EWOULDBLOCK 错误码，这表明操作无法立即完成。
- 在实际编程中，根据具体需求，可能还需要使用到其他函数，例如 `select()` 或 `poll()` 用于多路复用，或者 `fork()` 和 `exec()` 用于创建子进程处理连接。

## TCP socket通信流程

![tcp-socket.png](https://s2.loli.net/2024/06/30/SPcvUQlrHe3F9ZI.png)

服务器端流程如下：

1. 创建服务器的`socket`
2. 初始化`sever_addr`(服务器地址)
3. 将`socket`和`server_addr`绑定 bind
4. 开始监听`listen`
5. 开一个循环保证服务器不会结束，不断的`accept`接入的客户端请求，进行读写操作`write`和`read` (send()和recv()也行)
6. 关闭`socket`

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>
#include <arpa/inet.h>
#include <unistd.h>

#define SERVER_PORT 10005
#define BUFFLEN 1024
#define BACLOG 10

// 创建新的socket，返回文件描述符
int create_socket() {
    int res = socket(AF_INET, SOCK_STREAM, 0);
    // NOTE: 如果想要使用UDP协议，只需要将SOCK_STREAM改为SOCK_DGRAM即可
    if (res < 0) {
        perror("create socket failed");
        exit(EXIT_FAILURE);
    }
    printf("create socket successfully\n");
    return res;
}

// 初始化服务器地址信息
void initialize_address(struct sockaddr_in *server_addr) {
    memset(server_addr, 0, sizeof(*server_addr));
    server_addr->sin_family = AF_INET;
    // 监听所有可用地址
    server_addr->sin_addr.s_addr = htons(INADDR_ANY);
    server_addr->sin_port = htons(SERVER_PORT);
}

// 绑定socket到指定的地址
void try_bind(int socket_fd, const struct sockaddr *server_addr) {
    if (bind(socket_fd, server_addr, sizeof(*server_addr)) < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }
    puts("bind successfully");
}

// 处理客户端请求
void process_client_request(int socket_client) {
    char send_buf[BUFFLEN] = {0};
    char recv_buf[BUFFLEN] = {0};

    // 发送的消息，以方便日后添加或修改
    snprintf(send_buf, BUFFLEN, "Hello, this is the server");

    while (1) {
        ssize_t num_read = read(socket_client, recv_buf, BUFFLEN);
        // 客户端断开连接，或读取失败
        if (num_read <= 0) {
            if (num_read == 0)
                puts("Client closed connection");
            else
                perror("read failed");
            return;
        }
        recv_buf[num_read] = '\0'; // 为接收的字符串添加结束字符

        printf("From Client: %s\n", recv_buf);

        if (strncmp(recv_buf, "exit", 4) == 0) { // 客户端请求退出
            // 修改发送的消息
            strcpy(send_buf, "bye!!!");
            // 返回消息给客户端
            write(socket_client, send_buf, strlen(send_buf));
            printf("Me(Server)：%s\n", send_buf);
            return;
        }

        // 向客户端写入发送缓冲区的内容
        if (write(socket_client, send_buf, strlen(send_buf)) < 0) {
            perror("write failed");
            return;
        }
        printf("Me(Server): %s\n", send_buf);

        // 清空接收缓冲区，准备下一次读取
        memset(recv_buf, 0, BUFFLEN);
    }
}

int main() {
    int socket_server = create_socket(); // 创建socket
    struct sockaddr_in server_addr; // 声明地址结构

    initialize_address(&server_addr); // 初始化地址结构
    try_bind(socket_server, (struct sockaddr*)&server_addr); // 绑定地址到socket

    // 开始监听连接请求，设定最大等待连接数量为BACLOG
    if (listen(socket_server, BACLOG) == -1) {
        perror("listen failed");
        exit(EXIT_FAILURE);
    }
    puts("start listening");

    struct sockaddr_in client_addr;
    socklen_t client_addr_size = sizeof(client_addr);

    while (1) {
        // 初始化客户端地址结构
        memset(&client_addr, 0, sizeof(client_addr));
        // 接受新的连接请求
        int socket_client = accept(socket_server, (struct sockaddr*)&client_addr, &client_addr_size);
        printf("Client %s:%d connected\n",
            inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));
        if (socket_client < 0) {
            perror("receive failed");
            continue; // 尝试接受另一个连接
        }
        // 满足要求后开始处理客户端请求
        process_client_request(socket_client);
        close(socket_client); // 关闭客户端连接
    }

    // 正常情况下不会执行到这一步
    close(socket_server);
    return 0;
}


```

客户端流程如下：

1. 创建客户端`socket`
2. 初始化`server_addr`
3. 连接到服务器`connect`
4. 利用`write`和`read`进行读写操作 (send()和recv()也行)
5. 关闭`socket`

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>
#include <arpa/inet.h>
#include <unistd.h>

#define SERVER_PORT 10005
#define BUFFLEN 1024

/*
 * 创建套接字并返回其文件描述符
 * 如果创建失败，程序将以状态1退出
 */
int create_socket() {
    int sock_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (sock_fd < 0) {
        perror("socket创建失败"); // 使用perror提供更多的错误信息
        exit(EXIT_FAILURE);
    }
    printf("socket create successfully\n");
    return sock_fd;
}

/*
 * 初始化服务器地址结构的函数
 */
void initialize_address(struct sockaddr_in *server_addr) {
    memset(server_addr, 0, sizeof(*server_addr));
    server_addr->sin_family = AF_INET;
    server_addr->sin_port = htons(SERVER_PORT);
    server_addr->sin_addr.s_addr = inet_addr("192.168.5.120"); // 服务器IP
}

/*
 * 建立到服务器的连接
 * 如果连接失败，程序将以状态1退出
 */
void connect_to_server(int socket_fd, const struct sockaddr *server_addr) {
    if (connect(socket_fd, server_addr, sizeof(*server_addr)) < 0) {
        perror("connection failed");
        exit(EXIT_FAILURE);
    }
    printf("connection successful\n");
}

/*
 * 客户端程序开始的主函数
 */
int main() {
    int socket_fd = create_socket(); // 创建socket

    struct sockaddr_in server_addr;
    initialize_address(&server_addr); // 初始化服务器详细信息

    connect_to_server(socket_fd, (struct sockaddr*)&server_addr); // 连接到服务器

    char send_buf[BUFFLEN] = {0}; // 用于发送数据的缓冲区
    char recv_buf[BUFFLEN] = {0}; // 用于接收数据的缓冲区

    // 持续从用户获取输入并发送到服务器，直到从服务器接收到"bye!!!"
    while (fgets(send_buf, BUFFLEN, stdin)) {
        // 向服务器写入用户输入
        if (write(socket_fd, send_buf, strnlen(send_buf, BUFFLEN)) == -1) {
            perror("write failed");
            break;
        }
        printf("Me(Client):%s", send_buf); // 打印用户输入
        memset(send_buf, 0, BUFFLEN); // 清除发送缓冲区

        // 读取服务器的响应
        if (read(socket_fd, recv_buf, BUFFLEN) == -1) {
            perror("receive failed");
            break;
        }
        printf("Server:%s", recv_buf); // 打印服务器的响应
        if (strcmp(recv_buf, "bye!!!") == 0) {
            break;
        }
        memset(recv_buf, 0, BUFFLEN); // 清除接收缓冲区
    }

    close(socket_fd); // 关闭套接字
    return 0;
}


```
