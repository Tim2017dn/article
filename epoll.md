# epoll

# 一、前置知识

## 1.1 IO

IO就是输入输出，这很好理解。

IO设备就是我们用于和电脑交互的硬件，包含输入设备：键盘、鼠标，输出设备:显示器、打印机。还有电脑间交互的设备，例如网卡。

我们接触到的IO方法，一般有标准库中的 流式IO ，文件IO，还有网络IO。

## 1.2 输入

在正式介绍IO模型之前，我们需要了解，对于一个标准IO的输入，计算机干了哪些。

主要有两个基本操作：

​	1、等待数据就位

​	2、将数据从内核拷贝到进程

对于网络IO，输入基于socket，第一个步骤包含在网络上等待数据到达，当数据包到达，它会被内核拷贝到缓冲区。第二个步骤拷贝数据，数据从内核缓冲区拷贝到应用缓冲区。

## 1.3 IO模型

我们首先需要回顾一下IO模型。五个基本IO模型

- blocking I/O 阻塞
- nonblocking I/O 非阻塞IO
- I/O multiplexing (`select` and `poll`) IO多路复用
- signal driven I/O (`SIGIO`) // 信号驱动IO，暂不介绍
- asynchronous I/O (the POSIX `aio_`functions) // 异步IO ，暂不介绍

### 1.3.1Nonblocking I/O Model

figure 1：

![graphics/06fig01.gif](./pic/06fig01.gif)

如图1所示，应用端的进程调用 recvfrom ,这个调用需要等内核完成以下两个步骤。

1.完成 datagram ready，也就是之前提到的第一步等数据

2.完成 copy，也就是将数据从内核空间拷到用户空间。

在这个期间 ，应用进程一直是被阻塞的，它需要等函数返回，干不了其他事。



### 1.3.2 Nonblocking I/O Model

figure 2：

![graphics/06fig02.gif](./pic/06fig02.gif)

我们希望socket是不阻塞，我们需要让内核知道 “当收到IO操作，但数据就绪还没完成，放回一个error，而不是让用户进程sleep”。

如图2所示：应用进程发了3次recvfrom请求，但数据还没有ready，所以内核回复了 EWOULDBLOCK 这个error。应用进程第四次调用 recvfrom，kernel这边 数据准备好了，那么再把数据从内核空间拷到用户空间，数据拷好后返回。

应用进程一直处于一个循环内，在这个循环内不断调用 recvfrom，直到收到copy完成的信息。

### 1.3.3 I/O mutiplexing model

figure 3：

![graphics/06fig03.gif](./pic/06fig03.gif)

如图三所示，应用进程首先阻塞在select这个调用里，等待数据准备好。当select收到这个socket是可读的返回（也就是数据ready了），这时recvfrom才发出第二个调用 recvfrom，等待数据拷贝完成。



如果只是这样描述，好像IO多路复用也没啥优势，也是阻塞，同时只需要1条recvfrom系统调用，现在还需要两条系统调用。优势在于select可以等待不止一个描述符就绪。



**tips**：文件描述符

在[Linux](https://so.csdn.net/so/search?from=pc_blog_highlight&q=Linux)操作系统中，可以将一切都看作是文件，包括普通文件，目录文件，字符设备文件（如键盘，鼠标…），块设备文件（如硬盘，光驱…），套接字等等，所有一切均抽象成文件，提供了统一的接口，方便应用程序调用

文件描述符：简称fd，为了高效管理已被打开的文件所创建的索引。其fd本质上就是一个非负整数（通常是小整数）。

eg.

```
int fp = fopen ("xxx.txt", "w+");
```

```
int socket(int domain, int type, int protocol);
int listenfd = socket(AF_INET, SOCK_STREAM, 0);
```

# 二、select

select 允许进程指引内核等待多个时间









### 补充信息

同步和异步

posix 给出这两个概念的定义：

- 同步IO操作会导致请求线程被阻塞，直到该IO操作完成
- 异步IO操作不会导致请求线程阻塞。

按照这个定义，我们不难发现上述三种IO模型都是同步的，因为 recvfrom这个IO操作都把进程阻塞了。



# 九十九：参考资料

1.https://en.wikipedia.org/wiki/Input/output

2.https://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06lev1sec2.html



进程的状态



io:

https://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06lev1sec2.html

https://www.cnblogs.com/lojunren/p/3856290.html

https://docs.microsoft.com/en-us/windows/win32/fileio/differences-in-local-and-network-i-o



https://embetronicx.com/tutorials/linux/device-drivers/epoll-in-linux-device-driver/#Epoll_in_Linux_Device_Driver



https://programmer.help/blogs/introduction-to-select-poll-epoll-i-o-multiplexing.html

这篇很不错

https://chowdera.com/2021/07/20210726203242441j.html
