# libevent代码结构
libevent的源代码虽然都在一层文件夹下，但是其代码分类相当清晰．只要分为头文件，内部使用头文件，辅助功能函数，日志，libevent框架，对系统I/O多路复用机制的封装，信号管理，定时事件管理，缓冲区管理，基本数据结构和libevent的两个实用库等几个部分，有些部分可能就是一个源文件．

## 头文件[*.h]
libevent公用头文件都安装在event2目录中，分为三类：
* API头文件：定义libevent公用接口．这类头文件没有特定后缀．
* 兼容头文件：为已放弃的函数提供兼容的头部包含定义
* 结构头文件：这类头文件以相对不稳定的布局定义各种结构体。这些结构体重的一些事为了提供快速访问而暴露。直接依赖这类头文件中的任何结构体都会破坏程序对其他版本libevent的二进制兼容性，有时候是以非常难以调试的方式出现。这类头文件具有后缀"_struct.h"

其中compat/sys/queue.h 中一系列宏定义了5个数据结构：单向链表，双向链表，简单队列，Tail 队列，环形队列
# 实现[*.c]
event.c ： event主要方法实现

epoll.c : 对epoll的封装

select.c : 对select的封装

devpull.c : 对dev/poll的封装

kqueue.c : 对kueue的封装

signal.c ： 对信号事件的处理

evutil.c : 一些辅助功能函数的实现，包含创建socket pair和一些时间操作函数

log.c : log日志实现

buffer*.c ： 对缓冲区封装

____
[link](http://www.cnblogs.com/hustcat/archive/2010/08/31/1814022.html)