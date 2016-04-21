# libevent 的Reactor模型
libevent的设计是一个典型的Reactor模型，理解Reactor模型是理解libevent的基石，因此本节主要介绍典型的事件驱动设计模式---Reactor模式

> The Reactor design pattern handles service requests that are delivered concurrently to an application by one or more clients.

在[reactor.pdf](http://www.laputan.org/pub/sag/reactor.pdf)第一句话就直接说到Reactor 设计模式可以为一个应用同时处理一个或者多个客户端请求服务

## Reactor的事件处理机制
Reactor Bing 中文翻译为 "反应堆，反应器"，是一种事件驱动机制。与普通函数调用的不同之处在于：应用程序不是主动的调用某个API完成处理，而是恰恰相反，Reactor逆置了事件处理流程，应用程序需要提供相应的接口并注册到Reactor上,如果相应的事件发生，Reactor将主动调用应用程序注册的接口，这些接口又称为"callback function"（回调函数）.

在使用libevent时，需要向libevent框架注册相应的事件和会调函数，当注册的相应事件发生时，libevent会调用回调函数处理相应的时间(I/O读写，定时器，信号)

用"好莱坞原则"来形容Reactor再合适不过了： 不要打电话给我们，我们会打电话通知你。
## Reactor模式结构
在Reactor模式中，有5个关键参与者。
* 描述符（handle）：由操作系统提供，用于识别每个事件，如Socket描述符、文件描述符等。在Linux中，它用一个整数来表示。事件可以来自外部，如来自客户端的连接请求、数据等。事件也可以来自内部，如定时器事件。
* 同步事件分离器（demultiplexer）：是一个函数，用于等待一个或多个事件的发生。调用者会被阻塞，直到分离器分离的描述符集上有事件发生。Linux的select函数是一个经常被使用的分离器。
* 事件处理器接口（event handler）：是有一个或多个模板函数组成的接口。这些模板函数描述了和应用程序相关的对于某个事件的操作。
* 具体的事件处理器：是事件处理器接口的实现，它实现了应用程序提供的某个服务。每个具体的事件处理器总和一个描述符相关。它使用描述符来识别事件、识别应用程序提供的服务。
* Reactor管理器：定义了一些接口，用于应用程序控制事件调度，以及应用程序注册、删除事件处理器和相关的描述符。它是事件处理器的调度核心。Reactor管理器使用同步事件分离器来等待事件的发生。一旦事件发生，Reactor管理器先是分离每个事件，然后调度事件处理器，最后调用相关模板函数来处理这个事件。

![Reactor模型整体框架](http://7xoqng.com1.z0.glb.clouddn.com/20160421.png "Reactor模型整体框架")

## Reactor模式的优点
Reactor模式是编写高性能网络服务器的必备技术之一，它具有如下优点：
* 响应快，不必为单个同步时间所阻塞，虽然Reactor本身依然是同步的
* 编程相对简单，可以最大程度的避免复杂的多线程及同步问题，并且避免多线程或者多进程的切换开销
* 可扩展性，可以方便的通过增加Reactor实例个数来充分利用CPU资源
* 可复用性，Reactor框架本身与具体事件处理逻辑无关，具有很高的复用性

***
*待续*

[link](http://blog.csdn.net/sparkliang/article/details/4957744)