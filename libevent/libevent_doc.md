#　libevent
## 介绍
libevent是一个用于开发可扩展性网络服务器的基于事件驱动模型的网络库．
libevent具有几个显著亮点：
* 事件驱动，高性能
* 轻量级，专注于网络
* 跨平台，支持Windows,　Linux,　Mac OS等
* 支持多种I/O多路复用技术，epoll,　poll,　dev/poll,　select和kqueue等
* 支持I/O，定时器和信号等事件
* 注册事件优先级

libevent是为了取代在事件驱动的网络服务器的事件循环．应用程序只需要调用`event_dispatch()`然后动态的添加和删除事件而不需要改变事件循环．
libevent已经被广泛的应用，作为底层网络库；比如Memcached, Vomit, NetChat，Chromium等

## 标准用法
每一个使用libevent的应用程序必须包含`<event.h>`头文件，并通过链接器连接`-levent`.在使用库里面的方法之前必须调用`event_init()`或`event_base_new`去初始化libevent库．

## 事件通知
为了监视每一个文件描述符，你必须声明一个event结构和调用`event_set()`去初始化成员结构．启动通知，你要通过`event_add()`添加结构加入到监控事件列表．事件列表必须一直保留在活动的情况下，并且它应该是在堆上分配．最后你要调用`event_dispatch()`去循环调度事件．

## I/O缓冲
libevent在上层提供一个定期事件回调的抽象，这个抽象被称作缓冲事件．一个缓冲事件提供输入和输出缓冲区，能够自动的填充和取出．一个使用的缓冲事件用户不再直接的进行I/O操作，而是通过从输入缓冲区中读取，写入到输出缓冲区．

一旦初始化通过`bufferevent_new()`,`bufferevent`结构的`bufferevent_enable()`和`bufferevent_disable()`会被反复使用．通过调用`bufferevent_read()`和`bufferevent_write()`而取代直接的读写socket.

当被允许读时，bufferevent将试图读取文件描述符和调用读的回调函数．写回调被执行时，输出缓冲区写到临界时，将默认返回0.
