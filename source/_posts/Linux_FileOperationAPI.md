---
title: Linux 文件操作API
date: 2016-00-00 00:00:00
tags: [open, ioctl]
categories: Linux
---

# open

    #include <sys/types.h>
    #include <sys/stat.h>
    #include <fcntl.h>

    int open(const char *pathname, int flags); //打开一个现有的文件
    int open(const char *pathname, int flags, mode_t mode);  //打开文件不存在，则先创建
    
    【flags】:----------
    O_RDONLY, O_WRONLY, or O_RDWR.  这三个之一必备
    O_APPEND:  The file is opened in append mode.
    O_ASYNC:   This feature is available only for terminals, pseudoterminals, sockets, pipes and FIFOs 
    O_CLOEXEC: Enable the close-on-exec flag for the new file descriptor.
        了解这个选项需要明白fork后跟随exec和fork后不跟随exec的区别
        fork之后若跟随执行exec后，该进程执行的程序完全替换为新程序，而新程序则从其main函数开始执行。因为调用exec并不创建新进程，所以先后的进程ID并未改变。exec只是用一个全新的程序替换了当前进程的正文、数据、堆和栈段。
        也就是说之前的打开的文件描述符fd会被清理掉，导致打开的文件句柄无法释放，所以有了这个选项，在当前进程执行exec族函数时，会先做一个清理工作，close掉当前进程的文件描述符，再载入要执行的程序到当前进程环境。

    O_CREAT:-
        【mode】: （如果是新建文件，就设置初始访问权限）
            S_IRWXU  00700 user (file owner) has read, write and execute permission
            S_IRUSR  00400 user has read permission
            S_IWUSR  00200 user has write permission
            S_IXUSR  00100 user has execute permission
            S_IRWXG  00070 group has read, write and execute permission
            S_IRGRP  00040 group has read permission
            S_IWGRP  00020 group has write permission
            S_IXGRP  00010 group has execute permission
            S_IRWXO  00007 others have read, write and execute permission
            S_IROTH  00004 others have read permission
            S_IWOTH  00002 others have write permission
            S_IXOTH  00001 others have execute permission
            多个模式可位或 bitwise-or

    O_DIRECT:    Try to minimize cache effects of the I/O to and from this file.
    O_DIRECTORY: If  pathname is not a directory, cause the open to fail.
    O_EXCL:      如果同时指定了O_CREAT，而文件已经存在，则导致调用出错
    O_LARGEFILE: 
    O_NOATIME:   
    O_NOCTTY: 如果pathname指的是终端设备(tty)，则不将此设备分配作为此进程的控制终端
    O_NOFOLLOW
    O_NONBLOCK : When possible, the file is opened in nonblocking mode.
    O_PATH: 
    O_SYNC: The file is opened for synchronous I/O. Any write(2)s on the resulting file descriptor will block  
            the calling process until the data has been physically written to the underlying hardware.
            只在数据被写入外存或者其他设备之后操作才返回
    O_TRUNC: If  the  file already exists and is a regular file and the open mode allows writing
             (i.e., is O_RDWR or O_WRONLY) it will be truncated to length 0.
            如果文件存在，而且为读写或只写方式打开，则将其长度截断为0
    

# 文件类型
    ls -l
    drwxr-xr-x. 2 anxier anxier 4096 1月 15 00:29 desktop
    第一栏的信息包含10字符。第1个字符，表示文件的类型；第2～4位，代表文件所有者（User）的权限，分别为读、写、执行；第5～7位，代表文件所有者的同组用户（Group）的权限，分别是读、写、执行；第8～10位，代表其他用户（Other）的权限，分别为读、写、执行。


|       1       |   2   |   3   |   4   |   5   |   6   |   7   |   8   |   9   |   10  |
|---------------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
| -/l/c/s/d/b/p |   r/- |  w/-  |  x/-  |   r/- |  w/-  |  x/-  |   r/- |  w/-  |  x/-  |

    - 普通文件
    d 目录文件      文本文件和二进制文件。
    l 链接文件
    b 块设备文件
    c 字符设备文件
    p 管道文件      用于在进程间传递数据
    s 套接口文件


# ioctl

    #include <sys/ioctl.h>
    
    int ioctl(int d, int request, ...);

    request: ----
        TIOCSETD int *ldisc change the line discipline:
        TTYDISC termios interactive line discipline
        TABLDISC tablet line discipline
        SLIPDISC serial IP line discipline
        PPPDISC PPP line discipline
        TIOCGETD int *ldisc return the current line discipline
        TIOCSBRK set the terminal into BREAK condition
        TIOCCBRK clear the terminal BREAK condition
        TIOCSDTR assert data terminal ready
        TIOCCDTR clear data terminal ready
        TIOCGPGRP int *tpgrp return the terminal's process group
        TIOCSPGRP int *tpgrp associate the terminal's process group
        TIOCGETA struct termios *term get the terminal's termios attributes
        TIOCSETA struct termios *term set the terminal's termios attributes
        TIOCSETAW struct termios *term set the termios attrs after any output completes
        TIOCSETAF struct termios *term after any output completes, clear input and set termios attrs
        TIOCOUTQ int *num current number of characters in the output queue
        TIOCSTI char *cp manually send a character to the terminal
        TIOCSTOP stop output (like typing ^S)
        TIOCSTART start output (like typing ^Q)
        TIOCSCTTY make this the controlling terminal for the process
        TIOCDRAIN wait until all output is drained
        TIOCEXCL set exclusive use on the terminal
        TIOCNXCL clear exclusive use of the terminal
        TIOCFLUSH int *what clear input/output if `what' has FREAD/FWRITE set
        TIOCGWINSZ struct winsize *ws get the winsize information
        TIOCSWINSZ struct winsize *ws set the winsize information
        TIOCCONS int *on redirect kernel console messages
        TIOCMSET int *state set the modem state bit flags according to the following:
        TIOCM_LE Line Enable
        TIOCM_DTR Data Terminal Ready
        TIOCM_RTS Request To Send
        TIOCM_ST Secondary Transmit
        TIOCM_SR Secondary Receive
        TIOCM_CTS Clear To Send
        TIOCM_CAR Carrier Detect
        TIOCM_CD Carrier Detect (synonym)
        TIOCM_RNG Ring Indication
        TIOCM_RI Ring Indication (synonym)
        TIOCM_DSR Data Set Ready
        TIOCMGET int *state get the modem state bit flags
        TIOCMBIS int *state add modem state bit flags, OR-ing in the new states
        TIOCMBIC int *state clear modem state bit flags

