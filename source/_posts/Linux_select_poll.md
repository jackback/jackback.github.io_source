---
title: Linux select pselect poll epoll用法
date: 2017-04-19 00:00:00
tags: [select, poll]
categories: Linux
---

# select & pselect #

    #include <sys/select.h>
    #include <sys/time.h>
    #include <sys/types.h>
    #include <unistd.h>
  
    void FD_CLR(int fd, fd_set *set);
    int  FD_ISSET(int fd, fd_set *set);
    void FD_SET(int fd, fd_set *set);
    void FD_ZERO(fd_set *set);

    int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
    监控fd状态（timeout 是到期前剩余时间）
    
    int pselect(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, const struct timespec *timeout, const sigset_t *sigmask);
    除了监测fd状态，pselect精度更准，且监测signal(另外pselect的timeout参数是const的)


monitor multiple file descriptors, waiting until one or more of the file descriptors become "ready" for some class of I/O operation

<!-- more -->

# poll & ppoll # 

    #include <poll.h>
 
    int poll(struct pollfd *fds, nfds_t nfds, int timeout);  

        fds:        监听事件的item数组     
        nfds:       数组成员数
        timeout:    超时时间(unit:milliseconds, -1 mean infinite, 0 mean immediately)
        return:     success: > 0 带有revents事件的数组个数;  = 0 time out 或者没有fd 事件
                    error:   -1, errno is set

    int ppoll(struct pollfd *fds, nfds_t nfds, const struct timespec *timeout_ts, const sigset_t *sigmask);

        fds:        监听事件的item数组     
        nfds:       数组成员数
        timeout_ts: 超时时间(unit:seconds+nanoseconds, -1:indefinitely, 0:immediately)
        sigmask:    监测的signal (遇到这些signal，ppoll也会返回) (NULL:indefinitely)
        return:     success: > 0 带有revents事件的数组个数;  = 0 time out 或者没有fd 事件
                    error:   -1, errno is set

performs a similar task to select(2): it waits for one of a set of file descriptors to become ready to perform I/O.


### 结构体
    struct pollfd {
       int   fd;         /* in,  file descriptor */
       short events;     /* in,  requested events 请求需要监测的事件类型 */
       short revents;    /* out, returned events 发生的事件类型 */
    };

    The bits that may be set/returned in events and revents are defined in <poll.h>:

    POLLIN      可读
    POLLPRI     紧急数据可读
    POLLOUT     可读(不会block)
    POLLRDHUP   对端关闭, 本段hangup, 只针对stream socket fd
    POLLERR     错误状态 (output only，即只会出现在revents中)
    POLLHUP     Hang up (output only，即只会出现在revents中)
    POLLNVAL    无效请求 (output only，即只会出现在revents中)



# epoll # 

The epoll API performs a similar task to poll(2): monitoring multiple file descriptors to see if I/O is possible on any of them.
The epoll API can be used either as an edge-triggered or a level-triggered interface and scales well to large numbers of watched file descriptors.

### 模式 and 
  LT(Level-triggered)   一般
  ET(edge-triggered)    事件发生时就触发

### epoll_create
  creates an epoll instance and returns a file descriptor referring to that instance, The more recent epoll_create1 extends the functionality.

    #include <sys/epoll.h>
    int epoll_create(int size);
    int epoll_create1(int flags);

### epoll_ctl
  register particular file descriptors interested. 

    #include <sys/epoll.h>
    int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

        epfd: return by epoll_create
        op:
            EPOLL_CTL_ADD Register the target file descriptor fd on the epoll instance
            EPOLL_CTL_MOD Change the event event associated with the target file descriptor fd.
            EPOLL_CTL_DEL Remove (deregister) the target file descriptor fd from the epoll
        fd: 需要监听的描述符fd
   
        event：需要监听的指定fd上的事件
            typedef union epoll_data {
                 void        *ptr;
                 int          fd;
                 uint32_t     u32;
                 uint64_t     u64;
             } epoll_data_t;
            
             struct epoll_event {
                 uint32_t     events; // EPOLLIN EPOLLOUT EPOLLRDHUP EPOLLPRI EPOLLERR 
                                         EPOLLHUP EPOLLET EPOLLONESHOT 
                 epoll_data_t data; // User data variable
             };
         
        return:
            success 0
            error   -1 errno


### epoll_wait
  waits for I/O events, blocking the calling thread if no events are currently available.

    #include <sys/epoll.h>
    int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
    int epoll_pwait(int epfd, struct epoll_event *events, int maxevents, int timeout, const sigset_t *sigmask);

        epfd: return by epoll_create
        events：out, 监听到的事件数组
        maxevents: in, 需要监听到事件个数，>0
        timeout: time out值， (-1 mean infinite, 0 mean immediately)
        sigmask: 监测的signal 
        return: the number of file descriptors ready for the requested I/O.



# 附（epoll 使用示例）

功能: 使用epoll 实现tcp server

    #include <iostream>
    #include <sys/socket.h>
    #include <sys/epoll.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    #include <fcntl.h>
    #include <unistd.h>
    #include <stdio.h>
    #include <errno.h>
    
    using namespace std;
    
    #define MAXLINE 5
    #define OPEN_MAX 100
    #define LISTENQ 20
    #define SERV_PORT 5000
    #define INFTIM 1000
    
    void setnonblocking(int sock)
    {
        int opts;
        opts=fcntl(sock,F_GETFL);
        if(opts<0)
        {
            perror("fcntl(sock,GETFL)");
            exit(1);
        }
        opts = opts|O_NONBLOCK;
        if(fcntl(sock,F_SETFL,opts)<0)
        {
            perror("fcntl(sock,SETFL,opts)");
            exit(1);
        }
    }
    
    int main(int argc, char* argv[])
    {
        int i, maxi, listenfd, connfd, sockfd, epfd, nfds, portnumber;
        ssize_t n;
        char line[MAXLINE];
        socklen_t clilen;
    
        if (2 == argc ) {
            if( (portnumber = atoi(argv[1])) < 0 ) {
                fprintf(stderr,"Usage:%s portnumber/a/n", argv[0]);
                return 1;
            }
        }
        else {
            fprintf(stderr,"Usage:%s portnumber/a/n",argv[0]);
            return 1;
        }
    
        //声明epoll_event结构体的变量,ev用于注册事件,数组用于回传要处理的事件
        struct epoll_event ev, events[20];
        
        epfd = epoll_create(256); //生成用于处理accept的epoll专用的文件描述符
        if (epollfd == -1) {
            perror("epoll_create");
            exit(EXIT_FAILURE);
        }

        // set up server listen socket
        listenfd = socket(AF_INET, SOCK_STREAM, 0);
        
        //setnonblocking(listenfd);     //把socket设置为非阻塞方式
    
        ev.events=EPOLLIN | EPOLLET;    //设置要处理的事件类型及事件触发模式
        ev.data.fd=listenfd;            //设置与要处理的事件相关的文件描述符

        //注册epoll事件
        if (epoll_ctl(epfd, EPOLL_CTL_ADD, listenfd, &ev) == -1) {
            perror("epoll_ctl: listen_sock");
            exit(EXIT_FAILURE);
        }

        // bind server listen socket
        struct sockaddr_in serveraddr;
        bzero(&serveraddr, sizeof(serveraddr));
        serveraddr.sin_family = AF_INET;
        char *local_addr="127.0.0.1";
        inet_aton(local_addr, &(serveraddr.sin_addr));

        serveraddr.sin_port = htons(portnumber);
        bind(listenfd, (sockaddr *)&serveraddr, sizeof(serveraddr));
        listen(listenfd, LISTENQ);
        maxi = 0;


        struct sockaddr_in clientaddr;
        for ( ; ; ) {
            nfds = epoll_wait(epfd,events,20,500); //等待epoll事件的发生

            //处理所发生的所有事件
            for(i=0; i<nfds; ++i)
            {
                if(events[i].data.fd==listenfd)//如果新监测到一个SOCKET用户连接到了绑定的SOCKET端口，建立新的连接。
                {
                    connfd = accept(listenfd,(sockaddr *)&clientaddr, &clilen);
                    if(connfd<0){
                        perror("connfd<0");
                        exit(1);
                    }
                    //setnonblocking(connfd);
    
                    char *str = inet_ntoa(clientaddr.sin_addr);
                    cout << "accapt a connection from " << str << endl;
                    //设置用于读操作的文件描述符
    
                    ev.data.fd=connfd;
                    //设置用于注测的读操作事件
    
                    ev.events=EPOLLIN|EPOLLET;
                    //ev.events=EPOLLIN;
    
                    //注册ev
                    epoll_ctl(epfd,EPOLL_CTL_ADD,connfd,&ev);
                }
                else if(events[i].events&EPOLLIN)//如果是已经连接的用户，并且收到数据，那么进行读入。
                {
                    cout << "EPOLLIN" << endl;
                    if ( (sockfd = events[i].data.fd) < 0)
                        continue;
                    if ( (n = read(sockfd, line, MAXLINE)) < 0) {
                        if (errno == ECONNRESET) {
                            close(sockfd);
                            events[i].data.fd = -1;
                        } else
                            std::cout<<"readline error"<<std::endl;
                    } else if (n == 0) {
                        close(sockfd);
                        events[i].data.fd = -1;
                    }
                    line[n] = '/0';
                    cout << "read " << line << endl;
                    //设置用于写操作的文件描述符
    
                    ev.data.fd=sockfd;
                    //设置用于注测的写操作事件
    
                    ev.events=EPOLLOUT|EPOLLET;
                    //修改sockfd上要处理的事件为EPOLLOUT
    
                    //epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev);
                }
                else if(events[i].events&EPOLLOUT) // 如果有数据发送
                {
                    sockfd = events[i].data.fd;
                    write(sockfd, line, n);
                    //设置用于读操作的文件描述符
    
                    ev.data.fd=sockfd;
                    //设置用于注测的读操作事件
    
                    ev.events=EPOLLIN|EPOLLET;
                    //修改sockfd上要处理的事件为EPOLIN
    
                    epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev);
                }
            }
        }
        return 0;
    }
