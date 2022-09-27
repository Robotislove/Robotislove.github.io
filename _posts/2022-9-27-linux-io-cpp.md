---
layout: post
title: Linux 网络IO总结学习笔记
date: 2022-09-27
author: lau
tags: [Linux, Blog]
comments: true
toc: false
pinned: false


---

Linux 网络IO总结学习笔记。

<!-- more -->

## 1 简介

Linux的设计哲学是“一切皆为文件”，比如磁盘被抽象为块设备，键盘被抽象为字符设备等等。因此对不同设备读取和写数据时可以使用系统调用write和read两个

操作进行数据交互。通常读取数据的过程是使用open打开一个文件描述符–>使用write或者read读写数据–>使用close关闭文件。
![](https://img-blog.csdnimg.cn/20200603122431601.png)

然而对于计算机系统，请求数据本身是一个过程，现代操作系统大多采用段页式虚拟内存进行内存管理。在Linux中如果目标数据不在内存中则需要出发缺页中断

将数据缓存到页缓存中供用户端使用，即数据会仙贝拷贝到操作系统内核的缓冲区中，然后才会从操作系统的内核缓冲区拷贝到应用程序的地址空间进行访问。这

个过程可以明显的看到分为两个阶段：请求数据(waiting for the data to be ready)和数据从内核到进程的拷贝(copying the data from the kernel to the process)。

上面提到的通过缓存进行IO的缺点也很明显：数据在传输过程中需要在应用程序地址空间和内核进行多次数据拷贝操作，数据拷贝操作所带来的 CPU 以及内存开

销是非常大的。特别实在后端开发中面对大量socket链接时。

因此出现了五种针对上述数据请求过程存在问题的网络模式方案：

1. 阻塞I/O(blocking IO);
2. 非阻塞I/O(nonblocking IO);
3. I/O多路复用(IO multiplexing);
4. 信号驱动I/O(signal driven IO);
5. 异步I/O(asynchronouse IO)。

## 2 不同的IO模式

  下面将上面提到的数据等待阶段和数据拷贝完成阶段分别指定为阶段1和阶段2。

### 2.1 阻塞I/O(blocking IO)

![](https://img-blog.csdnimg.cn/20200603122431613.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyYXlPbkRyZWFt,size_16,color_FFFFFF,t_70)

当进程试图访问一个IO设备的数据，而该IO设备并未准备好数据时，设备的驱动程序会通过内核让该进程进入sleep状态。

对于阻塞IO阶段1和阶段2都是处于阻塞状态。阻塞IO的优缺点：

- 优点：CPU友好。当进程请求数据时进程的状态会转换成阻塞，直到IO完成转换成就绪状态供CPU调度，不会占用太多的CPU；
- 缺点：当进程中的IO操作并不是必须的时候，任务不友好。比如进程只是尝试性的访问下IO或者设备同时访问多个IO，只要其中一个IO存在数据即可的情况，显然会在第一个IO处阻塞其他IO的状态反而无法得知。

通常发生阻塞的情况：

- read,标准输入输出流，socket，管道设备等默认阻塞;
- write,一般write不发生阻塞，除非写入的数据大小比缓冲区的空间大小大而导致无法全部写入。

假设进程希望从三个管道中任意一个读取数据：

```c++
read(pipe0, buffer, sizeof(buffer));
read(pipe1, buffer, sizeof(buffer));
read(pipe2, buffer, sizeof(buffer));
```

由于管道本身是阻塞IO，如果pipe0没有数据输入就会导致进程阻塞，pipe1和pipe2的数据都不会被读取。

```c++
int flag = fcntl(pipe_id, F_GETFL);
fcntl(pipe_id, F_SETFL, f1 | O_NONBLOCK);
```

通过以上代码设置管道为非阻塞，如果三个管道都没有数据则进程无法完成数据请求。

阻塞IO要么获取全部数据，要么什么也得不到，而进程可能希望监听多个IO,只要有一个IO拥有数据即可，这边是IO多路复用需要做的。

### 2.2 非阻塞I/O(nonblocking IO)

当进程访问IO设备，而IO设备未准备好数据时，过程立马返回一个error，告诉进程没有可获取的数据。

通常情况下对于非阻塞IO会通过while(buffer){recvfrom();}轮询的方式获取数据。非阻塞IO阶段1非阻塞，阶段2阻塞。非阻塞IO的优点是控制权在进程端，如

何组织完全由进程端决定，缺点是如果使用轮询会导致大量的cpu时间浪费。
![](https://img-blog.csdnimg.cn/20200603122431591.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyYXlPbkRyZWFt,size_16,color_FFFFFF,t_70)

### 2.3 I/O多路复用(IO multiplexing)
  IO多路复用也称事件驱动IO，用户进程负责需要从多个IO中的一个读取数据，进程通过select, poll, epoll等几个操作对多个IO进行轮询，当其中一个IO获得数

据时则通知用户进程数据到达，用户进程使用recvfrom进行数据读取。

  可以看到IO多路复用阶段1和阶段2都被阻塞，需要调用两次系统调用。

  仔细想想IO多路复用的过程很像多线程+阻塞IO的情况（使用多线程，每个线程监听一个IO），只是select,poll层面上不同。当服务器处理的连接数比较少

时，多线程+阻塞IO的效率相比于多路IO复用高；当服务器处理的链接比较多时，IO多路复用比较占优势。


![](https://img-blog.csdnimg.cn/20200603122706764.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyYXlPbkRyZWFt,size_16,color_FFFFFF,t_70)

在IO multiplexing Model中，实际中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。只不

过process是被select这个函数block，而不是被socket IO给block。

### 2.4 信号驱动I/O(signal driven IO)

信号驱动式I/O是指进程预先告知内核，使得当某个描述符上发生某事是，内核使用信号通知相关进程。

基本步骤如下：

- 建立sigio信号处理函数:

```c++
signal(SIGIO, sig_io);
```

- 设置对应IO属主进程：

```c++
fcntl(socket_fd, F_SETOWN, getpid());
```

- 开启该IO的信号驱动式IO：

```c++
const int on = 1;
ioctl(socket_fd, O_ASYNC, &on);
```

对于socket链接信号驱动IO产生的信号过于频繁无法准确判定是哪个事件，一般不常用。

TCP产生的信号事件：

1. 监听套接字上某个连接请求请求已经完成；
2. 某个断连接请求已经发起；
3. 某个连接之半已经关闭；
4. 数据到达套接字；
5. 数据已经从套接字发送走(即输出缓冲区有空闲空间)；
6. 发生某个异步错误。

UDP产生的信号事件：

1. 数据包到达套接字；
2. 套接字上产生异步错误。

### 2.5 异步I/O(asynchronouse IO)

当进程发起read操作只够立即返回继续进程操作，kernel收到asynchronouse read后也立即返回；当数据准备完成后，kernel将数据拷贝到用户内存，给用户发

送signal，用户读取数据。

理论上，速度可以但是实际使用中不尽人意。
![](https://img-blog.csdnimg.cn/20200603122431630.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyYXlPbkRyZWFt,size_16,color_FFFFFF,t_70)

 详情见：[Linux AIO（异步IO）那点事儿](http://www.yeolar.com/note/2012/12/16/linux-aio/)

### 2.6 不同IO方式的对比

![](https://img-blog.csdnimg.cn/20200603122431605.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyYXlPbkRyZWFt,size_16,color_FFFFFF,t_70)

## 3 IO多路复用API

select，poll，epoll都是IO多路复用的机制。I/O多路复用就是通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。

### 3.1 select

和select相关的API:

```c++
void FD_CLR(int fd, fd_set *set);       //从fd_set中删除文件描述符fd
int  FD_ISSET(int fd, fd_set *set);     //测试文件描述符fd是否输入fd_set
void FD_SET(int fd, fd_set *set);       //将一个文件描述符加入到fd_set中
void FD_ZERO(fd_set *set);              //清空fd_set对象
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);    //监听fd_set中的文件描述符
int pselect(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, const struct timespec *timeout, const sigset_t *sigmask);   //先设定信号屏蔽，在进行监听
```

select目前几乎在所有的平台上支持，其良好跨平台支持也是它的一个优点。select的一个缺点在于单个进程能够监视的[文件描述符](https://so.csdn.net/so/search?q=文件描述符&spm=1001.2101.3001.7020)的数量存在最大限制，在Linux

上一般为1024，可以通过修改宏定义甚至重新编译内核的方式提升这一限制，但 是这样也会造成效率的降低。

```c++
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/select.h>

#define BUFF_SIZE 100
#define MAX_FD 1024

//程序本身并不严谨，为了演示只演示过程
int main()
{
    //创建socket
    int sock_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (-1 == sock_fd)
    {
        perror("socket create failed!\n");
        exit(-1);
    }

    //绑定服务器地址和端口
    struct sockaddr_in addr = {0};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(8000);
    addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    int ret = bind(sock_fd, (struct sockaddr *)(&addr), sizeof(addr));
    if (ret == -1)
    {
        perror("bind server address failed!\n");
        exit(-1);
    }

    //监听socket
    ret = listen(sock_fd, 100);
    if (ret == -1)
    {
        perror("bind server address failed!\n");
        exit(-1);
    }

    //IO多路复用
    fd_set readfds;
    fd_set writefds;
    FD_ZERO(&readfds);
    FD_ZERO(&writefds);
    FD_SET(sock_fd, &readfds);

    fd_set temprfds = readfds;
    fd_set tempwfds = writefds;
    int maxfd = sock_fd;

    int n_ready = 0;
    char buffer[MAX_FD][BUFF_SIZE] = {0};

    while (1)
    {
        temprfds = readfds;
        tempwfds = writefds;

        n_ready = select(maxfd + 1, &temprfds, &tempwfds, NULL, NULL);
        if (FD_ISSET(sock_fd, &temprfds))
        {
            struct sockaddr_in recv_addr;
            socklen_t len = sizeof(recv_addr);
            int client_fd = accept(sock_fd, (struct sockaddr *)&recv_addr, &len);
            if (client_fd == -1)
            {
                perror("client connect server failed!\n");
                exit(-1);
            }

            //将新accept的scokfd加入监听集合，并保持maxfd为最大fd
            FD_SET(client_fd, &readfds);
            maxfd = maxfd > client_fd ? maxfd : client_fd;
            //如果意见检查了nready个fd，就没有必要再等了，直接下一个循环
            if (--n_ready == 0)
                continue;
        }

        //遍历文件描述符表，处理接收到的消息
        int fd = 0;
        for (fd = 0; fd <= maxfd; fd++)
        {
            if (fd == sock_fd)
                continue;

            if (FD_ISSET(fd, &temprfds))
            {
                int ret = read(fd, buffer[fd], sizeof buffer[0]);
                if (0 == ret)
                { //客户端链接已经断开
                    close(fd);
                    FD_CLR(fd, &readfds);
                    if (maxfd == fd)
                        --maxfd;
                    continue;
                }
            }
            //将fd加入监听可写的集合
            FD_SET(fd, &writefds);
        }
        //找到了接收消息的socket的fd，接下来将其加入到监视写的fd_set中
        //将在下一次while()循环开始监视
        if (FD_ISSET(fd, &tempwfds))
        {
            int ret = write(fd, buffer[fd], sizeof buffer[0]);
            printf("ret %d: %d\n", fd, ret);
            FD_CLR(fd, &writefds);
        }
    }

    close(sock_fd);
    return 0;
}
```

### 3.2 poll

  poll是一种基于select的改良机制，其针对select的一些缺陷进行了重新设计，包括不需要备份fd_set等等，但是依然是遍历整个文件描述符表，效率较低。

```c++
int ppoll(struct pollfd *fds, nfds_t nfds, const struct timespec *tmo_p, const sigset_t *sigmask);
struct pollfd {
               int   fd;         /* file descriptor */
               short events;     /* requested events */
               short revents;    /* returned events */
              };

POLLIN  //There is data to read.
POLLPRI //There is urgent data to read (e.g., out-of-band data on TCP socket; pseudoterminal master in packet mode has seen state change in slave).
POLLOUT //Writing is now possible, though a write larger that the available space in a socket or pipe will still block (unless O_NONBLOCK is set).
POLLRDHUP (since Linux 2.6.17)  //Stream  socket peer closed connection, or shut down writing half of connection.  The _GNU_SOURCE feature test macro must be defined (before including any header files) in order to obtain this definition.
POLLERR //Error condition (only returned in revents; ignored in events).
POLLHUP //Hang up (only returned in revents; ignored in events).  Note that when reading from a channel such as a pipe or a stream socket, this event merely indicates that the peer closed its  end of the channel.  Subsequent reads from the channel will return 0 (end of file) only after all outstanding data in the channel has been consumed.
POLLNVAL    //Invalid request: fd not open (only returned in revents; ignored in events).When compiling with _XOPEN_SOURCE defined, one also has the following, which convey no further information beyond the bits listed above:
POLLRDNORM  //Equivalent to POLLIN.
POLLRDBAND  //Priority band data can be read (generally unused on Linux).
POLLWRNORM  //Equivalent to POLLOUT.
POLLWRBAND  //Priority data may be written.
```

select和poll的异同：
  不同点：

1. select通过位图表示三个fdset,poll使用pollfd实现；
2. select有最大数量限制，poll没有；

  相同点：

1. 都需要轮询响应的描述符来获得就绪的描述符；
2. 当连接比较多时，只有少量就绪连接性能线性下降。

```c++
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/select.h>
#include <poll.h>

#define BUFF_SIZE 100
#define MAX_FD 1024

//程序本身并不严谨，为了演示只演示过程
int main()
{
    //创建socket
    int sock_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (-1 == sock_fd)
    {
        perror("socket create failed!\n");
        exit(-1);
    }

    //绑定服务器地址和端口
    struct sockaddr_in addr = {0};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(8000);
    addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    int ret = bind(sock_fd, (struct sockaddr *)(&addr), sizeof(addr));
    if (ret == -1)
    {
        perror("bind server address failed!\n");
        exit(-1);
    }

    //监听socket
    ret = listen(sock_fd, 100);
    if (ret == -1)
    {
        perror("bind server address failed!\n");
        exit(-1);
    }

    //IO多路复用

    struct pollfd myfds[MAX_FD] = {0};
    myfds[0].fd = sock_fd;
    myfds[0].events = POLLIN;
    int maxnum = 1;

    int nready;
    //准备二维数组buf，每个fd使用buf的一行，数据干扰
    char buf[MAX_FD][BUFF_SIZE] = {0};

    while (1)
    {
        //poll直接返回event被触发的fd的个数
        nready = poll(myfds, maxnum, -1);
        int i = 0;
        for (i = 0; i < maxnum; i++)
        {
            if (myfds[i].revents & POLLIN)
            {
                if (myfds[i].fd == sock_fd)
                {
                    struct sockaddr_in recv_addr;
                    socklen_t len = sizeof(recv_addr);
                    int client_fd = accept(sock_fd, (struct sockaddr *)&recv_addr, &len);
                    myfds[maxnum].fd = client_fd;
                    myfds[maxnum].events = POLLIN;
                    maxnum++;
                    if (--nready == 0)
                        continue;
                }
                else
                {
                    int ret = read(myfds[i].fd, buf[myfds[i].fd], sizeof(buf[0]));
                    if (0 == ret) //断开连接
                    {
                        close(myfds[i].fd);

                        //初始化将文件描述符表所有的文件描述符标记为-1
                        //close的文件描述符也标记为-1
                        //打开新的描述符时从表中搜索第一个-1
                        //open()就是这样实现始终使用最小的fd
                        //这里为了演示并没有使用这种机制
                        myfds[i].fd = -1;
                        continue;
                    }

                    myfds[i].events = POLLOUT;
                }
            }
            else if (myfds[i].revents & POLLOUT)
            {
                int ret = write(myfds[i].fd, buf[myfds[i].fd], sizeof buf[0]);
                myfds[i].events = POLLIN;
            }
        }
    }

    close(sock_fd);
    return 0;
}
```

### 3.3 epoll
epoll在poll基础上实现的更为健壮的接口，它每次只会遍历我们关心的文件描述符，也是现在主流的web服务器使用的多路复用技术，epoll一大特色就是支持EPOLLET(边沿触发)和EPOLLLT (水平触发)，前者表示如果读取之后缓冲区还有数据，那么只要读取结束，剩余的数据也会丢弃，而后者表示里面的数据不会丢弃，下次读的时候还在，默认是EPOLLLT。

```c++
int epoll_create(int size)；//创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；    //函数是对指定描述符fd执行op操作
struct epoll_event {
  __uint32_t events;  /* Epoll events */
  epoll_data_t data;  /* User data variable */
};

//epoll_event事件类型，具体内容info epoll_ctl
//EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
//EPOLLOUT：表示对应的文件描述符可以写；
//EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
//EPOLLERR：表示对应的文件描述符发生错误；
//EPOLLHUP：表示对应的文件描述符被挂断；
//EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
//EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);  //等待epfd上的io事件，最多返回maxevents个事件

```

epoll支持两种工作模式：

- LT模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件;
- ET模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。
  

```c++
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/select.h>
#include <poll.h>
#include <sys/epoll.h>

#define BUFF_SIZE 100
#define MAX_FD 1024

//程序本身并不严谨，为了演示只演示过程
int main()
{
    //创建socket
    int sock_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (-1 == sock_fd)
    {
        perror("socket create failed!\n");
        exit(-1);
    }

    //绑定服务器地址和端口
    struct sockaddr_in addr = {0};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(8000);
    addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    int ret = bind(sock_fd, (struct sockaddr *)(&addr), sizeof(addr));
    if (ret == -1)
    {
        perror("bind server address failed!\n");
        exit(-1);
    }

    //监听socket
    ret = listen(sock_fd, 100);
    if (ret == -1)
    {
        perror("bind server address failed!\n");
        exit(-1);
    }

    //IO多路复用
    /* 创建epoll对象 */
    int epoll_fd = epoll_create(1024);

    //准备一个事件结构体
    struct epoll_event event = {0};
    event.events = EPOLLIN;
    event.data.fd = sock_fd; //data是一个共用体，除了fd还可以返回其他数据

    //ctl是监控listenfd是否有event被触发
    //如果发生了就把event通过wait带出。
    //所以，如果event里不标明fd，我们将来获取就不知道哪个fd
    epoll_ctl(epoll_fd, EPOLL_CTL_ADD, sock_fd, &event);

    struct epoll_event revents[MAX_FD] = {0};

    int nready;
    //准备二维数组buf，每个fd使用buf的一行，数据干扰
    char buf[MAX_FD][BUFF_SIZE] = {0};

    while (1)
    {
        //wait返回等待的event发生的数目
        //并把相应的event放到event类型的数组中
        nready = epoll_wait(epoll_fd, revents, MAX_FD, -1);
        int i = 0;
        for (; i < nready; i++)
        {
            //wait通过在events中设置相应的位来表示相应事件的发生
            //如果输入可用，那么下面的这个结果应该为真
            if (revents[i].events & EPOLLIN)
            {
                //如果是listenfd有数据输入
                if (revents[i].events == sock_fd)
                {
                    struct sockaddr_in recv_addr;
                    socklen_t len = sizeof(recv_addr);
                    int client_fd = accept(sock_fd, (struct sockaddr *)&recv_addr, &len);
                    struct epoll_event event = {0};
                    event.events = EPOLLIN;
                    event.data.fd = client_fd;
                    epoll_ctl(epoll_fd, EPOLL_CTL_ADD, client_fd, &event);
                }
                else    //读取数据
                {
                    int ret = read(revents[i].data.fd, buf[revents[i].data.fd], sizeof buf[0]);
					if(0 == ret){
						close(revents[i].data.fd);
						epoll_ctl(epoll_fd, EPOLL_CTL_DEL, revents[i].data.fd, &revents[i]);
					}

					revents[i].events = EPOLLOUT;
					epoll_ctl(epoll_fd, EPOLL_CTL_MOD, revents[i].data.fd, &revents[i]);
                }
            }
            else if (revents[i].events & POLLOUT)           //写入
            {
                int ret = write(revents[i].data.fd, buf[revents[i].data.fd], sizeof buf[0]);
				revents[i].events = EPOLLIN;
				epoll_ctl(epoll_fd, EPOLL_CTL_MOD, revents[i].data.fd, &revents[i]);
            }
        }
    }

    close(sock_fd);
    return 0;
}

```

epoll相比于select和poll高效的原因是基于一个事实：同时连接的大量客户端在同一时刻可能只有很少处于就绪状态。epoll对就绪的描述符进行遍历，而select,poll需要遍历所有的描述符。

## 4 参考

- [Linux AIO（异步IO）那点事儿](http://www.yeolar.com/note/2012/12/16/linux-aio/)
- [信号驱动式I/O](https://www.jianshu.com/p/8ed3e677ad7b)
- [Linux I/O多路复用](https://www.cnblogs.com/xiaojiang1025/p/6032316.html)
- [Linux IPC tcp/ip socket 编程](https://www.cnblogs.com/xiaojiang1025/p/5950458.html)
- [linux IO模式和IO多路复用](https://www.jianshu.com/p/7fbda1696789)
- [Linux IO模式及 select、poll、epoll详解](https://segmentfault.com/a/1190000003063859)
- [Linux IO 多路复用理解](https://blog.csdn.net/zhouguoqionghai/article/details/82531523)