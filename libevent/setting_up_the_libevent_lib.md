# libevent 库设置

libevent 有一些全局设置在整个进程中被共享。这些设置在整个库中起作用。

你必须在调用libevent其他部分之前做这些改变，否则，libevent可能处于不一致的状态。

## libevent中的日志消息

libevnet可以记录错误和警告信息。如果你编译支持日志，它也可以记录调试信息。默认的，这些消息被写到标准错误输出(stderr)，你可以通过定制log函数覆盖这些默认行为。

**接口**
```
#define EVENT_LOG_DEBUG 0
#define EVENT_LOG_MSG   1
#define EVENT_LOG_WARN  2
#define EVENT_LOG_ERR   3

/* Deprecated; see note at the end of this section */
#define _EVENT_LOG_DEBUG EVENT_LOG_DEBUG
#define _EVENT_LOG_MSG   EVENT_LOG_MSG
#define _EVENT_LOG_WARN  EVENT_LOG_WARN
#define _EVENT_LOG_ERR   EVENT_LOG_ERR

typedef void (*event_log_cb)(int severity, const char *msg);

void event_set_log_callback(event_log_cb cb);
```
编写自己的函数匹配关联`event_log_cb`函数，作为一个参数传递给`event_set_log_callback()`函数来覆盖libevent的日志行为。当libevent要记录一个日志的消息的时候，它会把消息传递给你提供的函数。你可以再次调用`event_set_log_callback()`函数传递NULL参数，返回默认的记录日志的行为。

**例子**
```
#include <event2/event.h>
#include <stdio.h>

static void discard_cb(int severity, const char *msg)
{
    /* This callback does nothing. */
}

static FILE *logfile = NULL;
static void write_to_file_cb(int severity, const char *msg)
{
    const char *s;
    if (!logfile)
        return;
    switch (severity) {
        case _EVENT_LOG_DEBUG: s = "debug"; break;
        case _EVENT_LOG_MSG:   s = "msg";   break;
        case _EVENT_LOG_WARN:  s = "warn";  break;
        case _EVENT_LOG_ERR:   s = "error"; break;
        default:               s = "?";     break; /* never reached */
    }
    fprintf(logfile, "[%s] %s\n", s, msg);
}

/* Turn off all logging from Libevent. */
void suppress_logging(void)
{
    event_set_log_callback(discard_cb);
}

/* Redirect all Libevent log messages to the C stdio file 'f'. */
void set_logfile(FILE *f)
{
    logfile = f;
    event_set_log_callback(write_to_file_cb);
}
```
**注意**
在用户提供`event_log_cb`回调函数中调用libevent函数是不安全的！例如，你试图编写一个使用bufferevent将警告信息发送给某个套接字的日志回调函数，可能会遇到难以诊断的bug。未来的libevent版中的某些函数可能会移除这个限制。
____
待续
[翻译](http://www.wangafu.net/~nickm/libevent-book/Ref1_libsetup.html)