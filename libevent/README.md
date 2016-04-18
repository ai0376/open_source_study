#　libevent 2.0
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
每一个使用libevent的应用程序必须包含`<event2/event.h>`头文件，并通过链接器连接`-levent`.(相反的如果你只想链接主要的event或者基础的I/O缓冲区管理代码，不想链接其他的代码协议，可以使用`-levent_code`)

## 库设置
在你调用libevent库中其他函数之前，你首先需要设置库。如果你想在多线程应用程序中使用libevent,你需要初始化线程支持，通过使用`evthread_use_pthreads()`或者`evthread_use_windows_threads()`.你可以查看`<event2/thread.h>`了解更多的信息.
另一点，你可以使用`event_set_mem_functions`取代libevent的内存管理,使用`event_enable_debug_mode()`函数打开调试模式.

## 创建一个event base
接下来，你需要创建一个`event_base`结构，使用`event_base_new()`或者`event_base_new_with_config()`.`event_base`的责任是保持跟踪哪一个event时间是即将发生的（换句话说，观察哪一个会成为活动的）和哪些事件是活动的。每一个事件都和单个`evnet_base`相关.

## 事件通知
你想监视每一个文件描述符，你必须创建一个event结构通过`event_new()`函数.(你也可以调用`event_assign()`函数声明一个event结构去初始化结构里面的成员.)启动通知，你要通过`event_add()`添加结构加入到监控事件列表．事件列表必须一直保留在活动的情况下，并且它应该是在堆上分配．

## 调度事件
最后，你要调用`event_dispatch()`去循环调度事件．你也可以使用`event_base_loop()`达到更精细的控制。

目前，只有一个线程可以被一个给定的`event_base`调度一次。如果你想同时运行事件在多线程中，你也可以有一个单一的event_base,把那些事件添加到一个工作队列，或者你可以创建多个event_base 对象.

## I/O缓冲区
libevent在上层提供一个定期事件回调的抽象，这个抽象被称作缓冲事件．一个缓冲事件提供输入和输出缓冲区，能够自动的填充和取出．一个使用的缓冲事件用户不再直接的进行I/O操作，而是通过从输入缓冲区中读取，写入到输出缓冲区．

一旦初始化通过`bufferevent_new()`,`bufferevent`结构的`bufferevent_enable()`和`bufferevent_disable()`会被反复使用．通过调用`bufferevent_read()`和`bufferevent_write()`而取代直接的读写socket.

当被允许读时，bufferevent将试图读取文件描述符和调用读的回调函数．写回调被执行时，输出缓冲区写到临界时，将默认返回0.
