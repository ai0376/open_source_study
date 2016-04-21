# libevent 的Reactor模型
libevent的设计是一个典型的Reactor模型，理解Reactor模型是理解libevent的基石，因此本节主要介绍典型的事件驱动设计模式---Reactor模式

> The Reactor design pattern handles service requests that are delivered concurrently to an application by one or more clients.

在[reactor.pdf](http://www.laputan.org/pub/sag/reactor.pdf)第一句话就直接说到Reactor 设计模式可以为一个应用同时处理一个或者多个客户端请求服务

## Reactor的事件处理机制
Reactor Bing 中文翻译为 "反应堆，反应器"，是一种事件驱动机制。与普通函数调用的不同之处在于：应用程序不是主动的调用某个API完成处理，而是恰恰相反，Reactor逆置了事件处理流程，应用程序需要提供相应的接口并注册到Reactor上,如果相应的事件发生，Reactor将主动调用应用程序注册的接口，这些接口又称为"callback function"（回调函数）.

在使用libevent时，需要向libevent框架注册相应的事件和会调函数，当注册的相应事件发生时，libevent会调用回调函数处理相应的时间(I/O读写，定时器，信号)

用"好莱坞原则"来形容Reactor再合适不过了： 不要打电话给我们，我们会打电话通知你。

***
*待续*

[link](http://blog.csdn.net/sparkliang/article/details/4957744)