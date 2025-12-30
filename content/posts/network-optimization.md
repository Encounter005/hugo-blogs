+++
title = "OpenWrt TCP UDP 网络性能优化"
tags = [ "软路由", "Linux", "计算机网络",]
categories = [ "技术",]
date = "2024-03-24T22:34:34+08:00"
+++
写这篇文章的原因是今天在打2042的时候丢包严重，一开始以为是Dice的服务器又爆了，后面跟同学对比了之后发现完全是路由器的问题。  
刚好拨号和主路由用的是R2S，可以学习一下linux内核的一些跟`TCP, UDP`有关的参数修改

> 下面的参数可以在`/etc/sysctl.conf`这个文件中修改

另外再多说一句，`Turbo acc`这个插件对这种500M一下的带宽基本没什么作用

# 最大队列大小优化

在通过`TCP\UDP`层处理数据之前，系统会将数据放入内核队列中。`net.core.netdev_max_backlog`值指定在传递到上层之前要放入队列的最大数据包数。对于高负载的网络，默认值是不够的.因此，我们可以适当地增加该值可以解决内核导致的性能损失。默认值为1000，将其增加到3000以上足以阻止数据包在10Gbps网络中被丢弃

```bash
# 默认值
$ sysctl net.core.netdev_max_backlog
sysctl net.core.netdev_max_backlog = 1000

net.core.netdev_max_backlog = 4096
```

还有一个类似的设置是`net.ipv4.tcp_max_syn_backlog`，这个指定的是记住的连接请求的最大数量，但仍未受到来自客户端的确认。对于内存超过128MB的系统，默认值为1024，对于低内存计算机，默认值为128。如果服务器过载，可以尝试增加该值

```bash
# 默认值
$ sysctl net.ipv4.tcp_max_syn_backlog
net.ipv4.tcp_max_syn_backlog = 128

net.ipv4.tcp_max_syn_backlog = 4096
```

# 最大挂起连接数优化

应用程序可以在处理一个连接之前指定要放入队列的最大待处理请求数，当此值达到最大值时，进一步的连接开始退出。对于发布大量连接的Web服务器等应用程序，此值必须很高才能使这些连接正常工作

```
# 默认值
$ sysctl net.core.somaxconn
net.core.somaxconn = 128

net.core.somaxconn = 4096
```

# TCP FIN超时优化

在TCP连接中，双方必须独立关闭连接。在Linux下，发送FIN数据包以关闭连接并等待`FINACK`知道定义超时值。这玩意的默认值是60，可以减少到20或者30使TCP关闭连接并释放资源以进行另一个连接

```bash
$ sysctl net.ipv4.tcp_fin_timeout
sysctl net.ipv4.tcp_fin_timeout = 60

net.ipv4.tcp_fin_timeout = 30
```

# 重用TIME_WAIT状态的套接字进行新连接

在处理必须处理`TIME_WAIT`状态下的许多短`TCP`连接的Web服务器时，或许会有用

```
$ sysctl net.ipv4.tcp_tw_reuse
net.ipv4.tcp_tw_reuse = 0

net.ipv4.tcp_tw_reuse = 1
```

# tcp_keepalive_time优化

TCP连接是由两个socket组成，每个socket在连接的两端。当一方想要终止连接时，它会发送另一方确认的`RST`数据包并关闭其socket

然而，在此之前，双方将无期限地sochi其socket开放。这使得一方可能有意或者由于某些错误而关闭其插座，而无需通过`RST`通知另一端。为了检测此场景并关闭过时连接，可以使用`TCP Keep Alive`处理

Linux中有三个可配置属性来确定`Keep-Alives`的属性:

- tcp_keepalive_time 默认是7200
- tcp_keepalive_probes 默认是9
- tcp_keepalive_intvl 默认是75

具体过程如下:

1. 客户端打开TCP连接
2. 如果`tcp_keepalive_time`秒的连接是静默的，则发送一个空的`ACK`数据包
3. 服务器是否使用自己的相应的`ACK`进行响应
   - 没有
     - 等待tcp_keepalive_intvl秒，然后发送另一个`ACK`
     - 重复，直到已发送的`ACk`探测数等于`tcp_keepalive_probes`
     - 如果此时未收到响应，请发送`RST`并终止连接
   - 有
     - 返回第二步

默认情况下，在大多数操作系统上启动了此过程的处理，因此一旦另一端无响应2小时11分钟( 7200s + 75 \* 9 ), 则会定期移除死TCP连接

```bash
$ sysctl net.ipv4.tcp_keepalive_time
net.ipv4.tcp_keepalive_time = 7200

net.ipv4.tcp_keepalive_time = 1200
```

# 启用智能MTU黑同检测优化

一旦启用，系统将尝试使用路径MTU发现机制在客户端和服务器之间找到MTU，可以通过`ip a`检查接口上的MTU:

```bash
$ ip a | grep 'mtu'
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN group default qlen 1000
4: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master br-lan state UP group default qlen 1000
5: br-lan: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
6: pppoe-wan: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1492 qdisc fq_codel state UNKNOWN group default qlen 3
```

[关于MTU](https://zhuanlan.zhihu.com/p/360516704)

要在Linux上启用此功能，运行下面的命令：

```bash
$ sysctl net.ipv4.tcp_mtu_probing
net.ipv4.tcp_mtu_probing = 0
$ sysctl net.ipv4.tcp_base_mss
net.ipv4.tcp_base_mss = 1024

net.ipv4.tcp_mtu_probing = 1
```

`tcp_mtu_probing`，控制TCP分组化-层路径MTU发现，有三个可选值：

- 0 已禁用
- 1 默认情况下禁用，在检测到ICMP黑洞时起用
- 2 始终启用，使用`tcp_base_mss`的初始MSS

# 内核缓冲区优化

系统默认的socket缓冲区大小:

```bash
$ sysctl net.core.wmem_default
net.core.wmem_default = 212992
$ sysctl net.core.rmem_default
net.core.rmem_default = 212992
$ sysctl net.core.rmem_max
sysctl net.core.rmem_max = 212992
$ sysctl net.core.wmem_max
net.core.wmem_max = 212992
```

这些参数显示分配给任何类型连接的默认和最大写入、读取缓冲区大小。 由于分配的空间来自RAM，因此默认值设置总是有点低。增加这一点可能会提高运行NFS等服务器的系统的性能。 将它们增加到256k / 4MB将最有效，否则您必须对这些值进行基准测试，以找到系统配置的理想值

```bash
# 256 KB / 4 MB
net.core.rmem_default = 262144
net.core.wmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_max = 4194304


# Or 256 Kb / 64 MB
net.core.rmem_default = 262144
net.core.wmem_default = 262144
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
```

# TCP缓冲区大小优化

确认一下默认值:

```bash
$ sysctl net.ipv4.tcp_rmem
net.ipv4.tcp_rmem = 4096        131072   6291456
$ sysctl net.ipv4.tcp_wmem
net.ipv4.tcp_wmem = 4096        16384   4194304
```

这些值是三个整数的数组，分别指定TCP读取和发送缓冲区的最小，平均和最大值

>  [!WARNING]
>  值要以页为单位。如果需要查看页面大小，可以使用`getconf PAGE_SIZE`来查看

TCP缓冲区最大值修改：

```bash

# 64MB
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 16384 67108864

# 12MB:
net.ipv4.tcp_rmem = 4096 87380 12582912
net.ipv4.tcp_wmem = 4096 16384 12582912

# 4MB:
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 16384 4194304
```

# Time Wait优化

`TIME WAIT TCP`套接字状态是套接字关闭但等待处理仍在网络中的数据包的状态。 参数tcp_max_tw_buckets是 TIME_WAIT 状态下的最大套接字数。 达到此数字后，系统将开始在此状态下销毁套接字

此限制仅用于防止简单的DoS攻击，您不得人为地降低限制，而是增加它（可能在增加安装的内存之后），如果网络条件需要超过默认值

```bash
$ sysctl net.ipv4.tcp_max_tw_buckets
net.ipv4.tcp_max_tw_buckets = 4096

$ sudo vi /etc/sysctl.d/98-network-custom.conf
net.ipv4.tcp_max_tw_buckets = 5000
```

如果遇到大量的TCP 错误，如：

```bash
__ratelimit: 33491 callbacks suppressed
TCP: time wait bucket table overflow
```

可以增加`net.ipv4.tcp_max_tw_buckets`的值，比如 654320，前提是拥有足够的内存

请尝试以下命令来确定您是否有来自一个地址的大量连接，或者您是否受到分布式攻击

`netstat -nt | cut -c 40- | cut -d: -f1 | sort | uniq -c | sort -n netstat -nt | cut -d: -f2 | sort | uniq -c | sort -n`

如果您从几个IP地址获得高数字，则更容易限制连接。 然后，您可以向 iptables 添加拒绝规则或速率限制规则，以限制从这些地址访问

经测试，优化此项可能造成上传文件至某些网站超时或错误

# 开启TCP时间戳

关于[tcp_timestamps](https://zhuanlan.zhihu.com/p/612010050)

Linux 系统中的 tcp_timestamps 参数控制是否启用 TCP 时间戳。要启用该功能可以用下面的命令：

```bash
net.ipv4.tcp_timestamps=1
```

通常，出于性能考虑，默认情况下，在现代操作系统上该参数会启用，除非有特定安全上的考虑需要关闭它。在某些情况下，为了提高安全性，可能会选择关闭时间戳功能，因为时间戳可以被恶意用户用来估算服务上次重启的时间，或者进行更精细的网络流量分析。

# TCP Sack

`tcp_sack` 表示 TCP Selective Acknowledgment (选择性确认)，这是一个 TCP/IP 的性能优化特性。在 TCP 通信中，数据是按顺序发送的，如果包丢失了，底层 TCP 协议需要进行重传。传统的 TCP 重传机制是累积确认的，也就是说，只有当所有的包按顺序全部接收时，接收端才会发送一个 ACK (确认)。
选择性确认允许接收方告诉发送方哪些数据包已经成功接收，即使它们不是按顺序到达的。这样，如果一些数据包丢失，发送方可以只重传那些未被确认接收的数据包，而不用从丢失的数据包或者先前已经被确认接收的部分开始重传整个数据流。这可以大大提高网络通信的效率，尤其是在丢包率较高或者网络延迟较大的环境下。

要启用该功能:

```bash
net.ipv4.tcp.sack = 1
```


# TCP窗口大小

`tcp_window_scaling` 是 TCP 协议提供的一个特性，用于支持较大的窗口大小，从而使得在高延迟和高带宽的网络上可以有更好的性能。在TCP中，窗口大小决定了不需要等待确认应答就可以发送的数据的最大值。这个值也就是说，在任一时刻，可以无确认地发送在网络上未确认的数据的最大量。
正常情况下，TCP 窗口的大小由一个 16 位的字段控制，这限制了最大的窗口大小为 65,535 字节（64 KB）。随着网络的发展，这个大小变得不足以利用高速网络的潜力。通过启用窗口缩放选项，TCP 可以使用最多 14 位的移位计数来扩大这个值，从而扩大可能的窗口大小，最大可达 1GB。

要启用该功能:

```bash
net.ipv4.tcp_window_scaling = 1
```

禁用 tcp_window_scaling 会限制 TCP 窗口的最大值为 64 KB，这可能会降低网络性能，尤其是在高速宽带连接上。因此，在不需要兼容老旧网络设备或特定网络配置的情况下，建议保持该选项为启用状态，以便能够充分利用网络带宽。

