---
title: select poll epoll 初探
date: 2018-5-11 11:11:11 +0800
categories: epoll
tags: epoll
---

# select poll epoll 初探
这三个都是所谓的synchronous I/O multiplexing（同步I/O多路复用），在学习这个之前，有必要了解一下常见的I/O模型，
而在了解I/O模型前又又必要了解一下内核空间及用户空间。

## 1 内核空间/用户空间
所谓的内核空间就是操作系统使用的内存空间，用户空间就是用户进程使用的内存空间，为了操作系统的安全，用户进程不能直接
操作操作系统内存空间，所以，内核空间和用户空间的数据交换需要通过内核提供的一组接口进行，也就是常说的系统调用。

## 2 I/O模型
I/O模型就是I/O操作的方式，I/O操作就是用户进程与外界数据交换的操作，而与外界进行数据交换需要操作系统内核提供支持，
常见的I/O模型又四种，阻塞I/O、非阻塞I/O、多路复用I/O、信号驱动I/O、异步I/O。

### 2.1 阻塞I/O（blocking I/O）
阻塞I/O就是进程请求I/O操作时会阻塞，具体一点就是用户进程向操作系统发起I/O请求，例如read操作中调用了系统函数recvfrom，
内核收到请求后就开始了I/O操作的第一个阶段————准备数据，如果是网络I/O请求，那么内核需要等待足够的数据到来，这时数据
保存在内核缓冲区，在这这个等待过程中，用户进程一直处于阻止塞状态（当然用户的I/O请求中也可以设置非阻塞，下面会提到），
当数据准备好了，内核就会将数据从内核缓冲区中拷贝到用户空间。用户感知到的就是调用recvfrom后，进程阻塞，直到数据准备
好了返回给用户进程。这就是最易理解的I/O模式。

### 2.2 非阻塞I/O（nonblocking I/O）
可以设置socket使之成为non-blocking，继续以读操作为例子，在非阻塞I/O模式中，当用户进程调用recvfrom时，内核立即返回，
在内核还未准备好数据时返回error，用户进程可以隔一段时间再次调用recvfrom，直到内核准备好数据，就会把数据考哦被到用户
空间，完成I/O操作。

### 2.3 I/O多路复用
I/O多路复用类似非阻塞I/O，不一样的是使用多路复用机制可以使一个进程同时处理多个网络连接的I/O，基本原理是通过select 
poll epoll会不断轮询多个socket，当其中某个I/O就绪就会通知用户进程。

### 2.3 异步I/O（Asynchronous I/O）
异步I/O是一种完全不会阻塞的I/O模型，之所以说完全不会阻塞是因为使用这种方式发起I/O请求后，内核立即返回，之后内核准备
数据，数据准备好后直接拷贝至用户空间，最后向用户进程发送一个signal，告诉进程I/O操作完成。而非阻塞I/O模型中，非阻塞
的是内核数据准备阶段，在数据拷贝阶段还是会阻塞用户进程。

### 2.4 信号驱动I/O（Signal drived I/O）
使用信号驱动I/O时，当网络套接字可读后，内核通过发送SIGIO信号通知应用进程，于是应用可以开始读取数据。有时也称此方式为
异步I/O。但是严格讲，该方式并不能算真正的异步I/O，因为实际读取数据到应用进程缓存的工作仍然是由应用自己负责的，即需要
用户主动调用系统函数去读取数据。

## 3 I/O多路复用之select poll epoll
select poll epoll都属于I/O多路复用模式的实现。

### 3.1 select
``` c
/*
 * @param n: readfds writefds exceptfds中的fd数量不能超过n+1
 * @param readfds: 需要监控的读操作fd
 * @param writefds: 需要监控的写操作fd
 * @param exceptfds: 需要监控的异常fd
 * @param timeout: 指定select可以等待的时间间隔，如果设成0秒和0毫秒，则select不等待，立即测试fd集中的所有fd，
 *                 如果设成NULL，等待无限久，可以被信号中断
 * @return 成功:返回三个集合中所有被设置的bit位数量|超时:返回0|失败:返回-1
 */
int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```
fd_set是以位图的形式来存储这些文件描述符，n为位图的长度。   
select优点：   
目前几乎在所有的平台上支持，其良好跨平台支持也是它的一个优点   
select缺点：   
1.每次调用 select()，都需要把 fd 集合从用户态拷贝到内核态，这个开销在 fd 很多时会很大，同时每次调用 select() 都需要在内核遍历传递进来的所有 fd，这个开销在 fd 很多时也很大。   
2.单个进程能够监视的文件描述符的数量存在最大限制，在 Linux 上一般为 1024，可以通过修改宏定义甚至重新编译内核的方式提升这一限制，但是这样也会造成效率的降低   

### 3.2 poll
``` c
/*
 * @param fds: 需要监控的文吉安描述符及需要监控的事件
 * @param nfds: fds中的文件描述符总数
 * @param timeout: 和select一样但是两个字段单位分别是秒和微秒
 *
 * @return 成功:返回正整数，返回又时间发生的fd数量|超时:返回0（也表示无事件发生）|失败:返回-1
 */
int poll(struct pollfd *fds, nfds_t nfds, int timeout);

struct pollfd {
    int   fd;         /* file descriptor */
    short events;     /* requested events */
    short revents;    /* returned events */
};
```
poll使用链表保存文件描述符，因此没有了监视文件数量的限制。

### 3.3 epoll
epoll实现机制和select/poll完全不一样，设想一下如下场景：有100万个客户端同时与一个服务器进程保持着TCP连接。而每一时刻，通常只有几百上千个TCP连接是活跃的(事实上大部分场景都是这种情况)。如何实现这样的高并发？     
在select/poll时代，服务器进程每次都把这100万个连接告诉操作系统(从用户态复制句柄数据结构到内核态)，让操作系统内核去查询这些套接字上是否有事件发生，轮询完后，再将句柄数据复制到用户态，让服务器应用程序轮询处理已发生的网络事件，这一过程资源消耗较大，因此，select/poll一般只能处理几千的并发连接。
``` c
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```
epoll的用法一句话描述就是三步曲：     
第一步：epoll_create()系统调用。此调用返回一个句柄，之后所有的使用都依靠这个句柄来标识。    
第二步：epoll_ctl()系统调用。通过此调用向epoll对象中添加、删除、修改感兴趣的事件，返回0标识成功，返回-1表示失败。     
第三部：epoll_wait()系统调用。通过此调用收集收集在epoll监控中已经发生的事件。    




 
