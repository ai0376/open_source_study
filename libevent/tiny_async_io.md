# A Tiny Introduction Asynchronous IO
大部初学程序员都是从阻塞IO调用开始的。如果一个IO调用是同步的，当你调用它，它不会返回，直到这个操作完成或者过去足够多的时间使你的网络栈自动放弃。 当你在一个TCP连接上调用`connect()`，例如，你的操作系统队列一个SYN包到达主机上TCP连接的另一边。它不会把控制权交给你，直到你接收到对面的一个SYN ACK包或者直到过去了足够多的事件，它决定放弃。

这有一个非常简单的例子使用阻塞网络调用。它去打开一个www.google.com的连接，发送一个简单的HTTP请求，然后打印响应到标准输出.(google大陆被墙，主机可以换成www.baidu.com)

**例子：一个简单的阻塞HTTP 客户端**
```
/* For sockaddr_in */
#include <netinet/in.h>
/* For socket functions */
#include <sys/socket.h>
/* For gethostbyname */
#include <netdb.h>

#include <unistd.h>
#include <string.h>
#include <stdio.h>

int main(int c, char **v)
{
    const char query[] =
        "GET / HTTP/1.0\r\n"
        "Host: www.google.com\r\n"
        "\r\n";
    const char hostname[] = "www.google.com";
    struct sockaddr_in sin;
    struct hostent *h;
    const char *cp;
    int fd;
    ssize_t n_written, remaining;
    char buf[1024];

    /* Look up the IP address for the hostname.   Watch out; this isn't
       threadsafe on most platforms. */
    h = gethostbyname(hostname);
    if (!h) {
        fprintf(stderr, "Couldn't lookup %s: %s", hostname, hstrerror(h_errno));
        return 1;
    }
    if (h->h_addrtype != AF_INET) {
        fprintf(stderr, "No ipv6 support, sorry.");
        return 1;
    }

    /* Allocate a new socket */
    fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        perror("socket");
        return 1;
    }

    /* Connect to the remote host. */
    sin.sin_family = AF_INET;
    sin.sin_port = htons(80);
    sin.sin_addr = *(struct in_addr*)h->h_addr;
    if (connect(fd, (struct sockaddr*) &sin, sizeof(sin))) {
        perror("connect");
        close(fd);
        return 1;
    }

    /* Write the query. */
    /* XXX Can send succeed partially? */
    cp = query;
    remaining = strlen(query);
    while (remaining) {
      n_written = send(fd, cp, remaining, 0);
      if (n_written <= 0) {
        perror("send");
        return 1;
      }
      remaining -= n_written;
      cp += n_written;
    }

    /* Get an answer back. */
    while (1) {
        ssize_t result = recv(fd, buf, sizeof(buf), 0);
        if (result == 0) {
            break;
        } else if (result < 0) {
            perror("recv");
            close(fd);
            return 1;
        }
        fwrite(buf, 1, result, stdout);
    }

    close(fd);
    return 0;
}
```
上述的代码所有的网络调用都是阻塞的:`gethostbyname`函数直到www.google.com解析成功或者失败后才会返回；`connect`函数直到连接成功才返回；`recv`函数直到接收到数据或者一个关闭才会返回；`send`函数直到最后刷新它的输出到内核写缓冲区。

现在，IO阻塞并不是不幸的。在此期间如果你的程序不去做其他事情，那么对你来说阻塞IO将工作的很好。 但是，假设你需要写一个程序去处理同时处理多个连接。让我们来具体的举一个例子：加入你想从两个连接中读取输入，但是你不知道那个连接将第一个输入。你不能说

**坏例子**
```
/*这些代码不能工作*/
char buf[1024];
int i, n;
while (i_still_want_to_read()) {
    for (i=0; i<n_sockets; ++i) {
        n = recv(fd[i], buf, sizeof(buf), 0);
        if (n==0)
            handle_close(fd[i]);
        else if (n<0)
            handle_error(fd[i], errno);
        else
            handle_input(fd[i], buf, n);
    }
}
```
当有数据在fd[2]上到来时，你的程序不能读取fd[2]上的数据，在fd[0]和fd[1]上的数据读完之前。

有时候人们为了解决这个问题，采用多线程，或者多进程服务。其中一个最简单的方法就是用多线程，每一个线程去处理一个连接。这样每一个连接都有一个自己的进程，一个连接的IO阻塞调用等待不会影响其他连接的进程阻塞。

这还有另一个例子程序。这是一个微不足道的服务程序，监听TCP连接端口为40713，从输入一行，读取数据，经过ROT13处理后的数据写出。这里为每一个到来的连接调用Unix的`fork()`来创建一个新的进程。

**例子：ROT13分支出来的server**
```
/* For sockaddr_in */
#include <netinet/in.h>
/* For socket functions */
#include <sys/socket.h>

#include <unistd.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

#define MAX_LINE 16384

char rot13_char(char c)
{
    /* We don't want to use isalpha here; setting the locale would change
     * which characters are considered alphabetical. */
    if ((c >= 'a' && c <= 'm') || (c >= 'A' && c <= 'M'))
        return c + 13;
    else if ((c >= 'n' && c <= 'z') || (c >= 'N' && c <= 'Z'))
        return c - 13;
    else
        return c;
}

void child(int fd)
{
    char outbuf[MAX_LINE+1];
    size_t outbuf_used = 0;
    ssize_t result;

    while (1) {
        char ch;
        result = recv(fd, &ch, 1, 0);
        if (result == 0) {
            break;
        } else if (result == -1) {
            perror("read");
            break;
        }

        /* We do this test to keep the user from overflowing the buffer. */
        if (outbuf_used < sizeof(outbuf)) {
            outbuf[outbuf_used++] = rot13_char(ch);
        }

        if (ch == '\n') {
            send(fd, outbuf, outbuf_used, 0);
            outbuf_used = 0;
            continue;
        }
    }
}

void
run(void)
{
    int listener;
    struct sockaddr_in sin;

    sin.sin_family = AF_INET;
    sin.sin_addr.s_addr = 0;
    sin.sin_port = htons(40713);

    listener = socket(AF_INET, SOCK_STREAM, 0);

#ifndef WIN32
    {
        int one = 1;
        setsockopt(listener, SOL_SOCKET, SO_REUSEADDR, &one, sizeof(one));
    }
#endif

    if (bind(listener, (struct sockaddr*)&sin, sizeof(sin)) < 0) {
        perror("bind");
        return;
    }

    if (listen(listener, 16)<0) {
        perror("listen");
        return;
    }



    while (1) {
        struct sockaddr_storage ss;
        socklen_t slen = sizeof(ss);
        int fd = accept(listener, (struct sockaddr*)&ss, &slen);
        if (fd < 0) {
            perror("accept");
        } else {
            if (fork() == 0) {
                child(fd);
                exit(0);
            }
        }
    }
}

int main(int c, char **v)
{
    run();
    return 0;
}
```


所以，我们有一个完美的解决方案去同时处理多连接？那我们现在可以停止写这本书，然后去干其他事情了吗？其实并不是.首先，进程的创建（或者线程的创建）在某些平台上是相当昂贵的。在现实生活中，你想用一个线程池，取代去创建新进程。
*待续*
[link](http://www.wangafu.net/~nickm/libevent-book/01_intro.html)