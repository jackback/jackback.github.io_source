---
title: Linux inotify机制
date: 2017-04-19 00:00:00
tags: [inotify]
categories: Linux
---

Inotify 是一个 Linux 内核特性，它监控文件系统，并且及时向专门的应用程序发出相关的事件警告，比如删除、读、写和卸载操作等。您还可以跟踪活动的源头和目标等细节。
使用 inotify 很简单：创建一个文件描述符fd，附加一个或多个监视器（一个监视器 是一个路径和一组事件，对应一个描述符wd），然后使用 read() 方法从描述符fd获取事件信息。

inotify，linux内核版本 >= 2.6.13


<!-- more -->

# inotify C API

Inotify 提供几个系统调用，它们可以构建各种各样的文件系统监控器：

##### 1. fd = inotify_init();  / fd = inotify_init1(int flags);
    在内核中创建 inotify 子系统的一个实例instance，成功的话将返回一个文件描述符fd，失败则返回 -1。
    就像其他系统调用一样，如果 inotify_init() 失败，请检查 errno 以获得诊断信息。

##### 2. wd = inotify_add_watch(int fd, const char *pathname, uint32_t mask); 
    用于添加监视器。每个监视器必须提供一个路径名和相关事件的列表（每个事件由一个常量指定，比如 IN_MODIFY）。
    要监控多个事件，只需在事件之间使用逻辑操作符或'|'。
    如果 inotify_add_watch() 成功，该调用会为已注册的监视器返回一个惟一的标识符wd; 否则，返回 -1。
    使用这个标识符更改或删除监视器。

##### 3. inotify_rm_watch(int fd, int wd);
    删除一个inotify实例中的监视器wd。 success return 0 ； error return -1。
    删除操作将触发IN_IGNORED 事件给该wd

##### 4. 读事件、关闭监视器、关闭inotify实例 
    创建完inotify实例及添加监视器后，读inotify实例描述符fd，可以得到监控到的事件。
    可以使用select pselect poll epoll 等避免阻塞

    文件描述符上的由 inotify_init() 生成的通用 close() 删除所有活动监视器，并释放与 inotify 实例相关联的所有内存（这里也用到典型的引用计数警告。与实例相关联的所有文件描述符必须在监视器和 inotify 消耗的内存被释放之前关闭）。

##### 5. 使用技巧/注意事项
    如果监视中的文件或目录被删除，它的监视器也会被自动删除（在删除事件发出之后）。
    如果在已卸载的文件系统上监控文件或目录，监视器将在删除所有受影响的监视之前收到一个卸载事件。
    将 IN_ONESHOT 标志添加到监视器标记中，设置一个一次性警告。警告在发送之后将被删除。
    要修改一个事件，必须提供相同的路径名和不同的标记。新监视器将取代老监视器。
    考虑到实用性，不可能耗尽任何一个 inotify 实例的监视器。然而，您可能会耗尽事件队列的空间，这取决于处理事件的频率。
        队列溢出会引起 IN_Q_OVERFLOW 事件。
    close() 方法毁坏 inotify 实例和所有相关联的监视器，并清空队列中的所有等待事件。


# inotify 事件结构体及宏（详见inotify.h）

##### 1. inotify_event 结构体
    struct inotify_event {
       int      wd;       /* Watch descriptor */  属于哪个监视器的事件，说白了就是wd对应的pathname的事件
       uint32_t mask;     /* Mask of events */   //  IN_ACCESS  IN_MODIFY  IN_ATTRIB ... 中一个或多个的位或
       uint32_t cookie;   /* Unique cookie associating related events (for rename(2)) */ 用于关联两个事件
       uint32_t len;      /* Size of name field */  name长度
       char     name[];   /* Optional null-terminated name */  事件关联的文件名，不包括路径
    };

**cookie字段使用场景:** 当把一个文件从一个目录移动到另一个目录时，您可以使用 cookie 将两个事件绑在一起。
仅当您监视源和目标目录时，inotify 才生成两个移动事件 — 分别针对源和目标 —，并通过设置 cookie 将它们绑定在一起。
要监视一个移动操作，必须指定 IN_MOVED_FROM 或 IN_MOVED_TO，或使用简短的 IN_MOVE，它可以监视两个操作。
使用 IN_MOVED_FROM 和 IN_MOVED_TO 来测试事件类型。


##### 2. 事件类型/宏（详见inotify.h）
    IN_ACCESS         0x00000001 File was accessed (read) (*).
    IN_MODIFY         0x00000002 File was modified (*).
    IN_ATTRIB         0x00000004 Metadata changed, e.g., permissions, timestamps, extended attributes, 
                                 link count (since Linux 2.6.25), UID, GID, etc. (*).
    IN_CLOSE_WRITE    0x00000008 File opened for writing was closed (*).
    IN_CLOSE_NOWRITE  0x00000010 File not opened for writing was closed (*).
    IN_OPEN           0x00000020 File was opened (*).
    IN_MOVED_FROM     0x00000040 Generated for the directory containing the old filename when a file is renamed (*).
    IN_MOVED_TO       0x00000080 Generated for the directory containing the new filename when a file is renamed (*).
    IN_CREATE         0x00000100 File/directory created in watched directory (*).
    IN_DELETE         0x00000200 File/directory deleted from watched directory (*).
    IN_DELETE_SELF    0x00000400 Watched file/directory was itself deleted.
    IN_MOVE_SELF      0x00000800 Watched file/directory was itself moved.

    IN_UNMOUNT        0x00002000
    IN_Q_OVERFLOW     0x00004000
    IN_IGNORED        0x00008000

    复合型事件类型
    IN_CLOSE      (IN_CLOSE_WRITE | IN_CLOSE_NOWRITE)
    IN_MOVE       (IN_MOVED_FROM | IN_MOVED_TO)
    IN_ALL_EVENTS (IN_ACCESS | IN_MODIFY | IN_ATTRIB | IN_CLOSE_WRITE |   IN_CLOSE_NOWRITE | IN_OPEN 
                  | IN_MOVED_FROM |   IN_MOVED_TO | IN_DELETE | IN_CREATE | IN_DELETE_SELF |   IN_MOVE_SELF)

    IN_ONLYDIR        0x01000000
    IN_DONT_FOLLOW    0x02000000
    IN_MASK_ADD       0x20000000
    IN_ISDIR          0x40000000
    IN_ONESHOT        0x80000000


# inotify 程序示例
    #include <stdio.h>
    #include <stdlib.h>
    #include <errno.h>
    #include <sys/types.h>
    #include <sys/inotify.h>
    
    #define EVENT_SIZE  ( sizeof (struct inotify_event) )
    #define BUF_LEN     ( 1024 * ( EVENT_SIZE + 16 ) )
    
    int main( int argc, char **argv ) 
    {
      int length, i = 0;
      int fd;
      int wd;
      char buffer[BUF_LEN];
    
      fd = inotify_init();
    
      if ( fd < 0 ) {
        perror( "inotify_init" );
      }
    
      wd = inotify_add_watch( fd, "/home/jim/test_inotity", 
                             IN_MODIFY | IN_CREATE | IN_DELETE );
      length = read( fd, buffer, BUF_LEN );  
    
      if ( length < 0 ) {
        perror( "read" );
      }  
    
      while ( i < length ) {
        struct inotify_event *event = ( struct inotify_event * ) &buffer[ i ];
        if ( event->len ) {
          if ( event->mask & IN_CREATE ) {
            if ( event->mask & IN_ISDIR ) {
              printf( "The directory %s was created.\n", event->name );       
            }
            else {
              printf( "The file %s was created.\n", event->name );
            }
          }
          else if ( event->mask & IN_DELETE ) {
            if ( event->mask & IN_ISDIR ) {
              printf( "The directory %s was deleted.\n", event->name );       
            }
            else {
              printf( "The file %s was deleted.\n", event->name );
            }
          }
          else if ( event->mask & IN_MODIFY ) {
            if ( event->mask & IN_ISDIR ) {
              printf( "The directory %s was modified.\n", event->name );
            }
            else {
              printf( "The file %s was modified.\n", event->name );
            }
          }
        }
        i += EVENT_SIZE + event->len;
      }
    
      ( void ) inotify_rm_watch( fd, wd );
      ( void ) close( fd );
    
      exit( 0 );
    }

    运行测试
    $ ./watcher  & 
    $ cd $HOME/test_inotify
    $ touch a b c
    The file a was created.
    The file b was created.
    The file c was created.

    
# inotify 程序示例2
    #include <stdio.h>
    #include <string.h>
    #include <stdlib.h>
    #include <sys/inotify.h>
    
    struct wd_path
    {
        int wd;
        char *path;
    };
    char * event_array[] = {
    "File was accessed",
    "File was modified",
    "File attributes were changed",
    "writtable file closed",
    "Unwrittable file closed",
    "File was opened",
    "File was moved from X",
    "File was moved to Y",
    "Subfile was created",
    "Subfile was deleted",
    "Self was deleted",
    "Self was moved",
    "",
    "Backing fs was unmounted",
    "Event queued overflowed",
    "File was ignored"
    };
    #define EVENT_NUM 16
    
    int main(int argc,char *argv[])
    {
        int fd,wd,len,tmp_len,i;
        char buffer[1024],*offset,target[1024];
        struct wd_path *wd_array;
        struct inotify_event *event;
        wd_array=(struct wd_path*)malloc((argc-1)*sizeof(struct wd_path));
        fd=inotify_init();
        if(-1==fd) 
        {
            printf("failed to init inotifyn");
            return 0;
        }
        
        printf("argc = %d \n", argc);
        if (argc < 2) {
            return -1;
        }
        
        printf("argv[0] = %d \n", argv[0]);
        printf("argv[1] = %d \n", argv[1]);
        
        for(i=0;i<argc-1;i++) 
        {
            wd_array[i].path = argv[i+1];
            wd=inotify_add_watch(fd,wd_array[i].path,IN_ALL_EVENTS); 
            wd_array[i].wd=wd;
        }
        
        memset(buffer,0,1024);
        while(len=read(fd,buffer,1024))
        {
            offset=buffer;
            event=(struct inotify_event*)buffer;
            while(((char *)event-buffer)<len)
            {
                for(i=0;i<argc-1;i++)
                {
                    if(event->wd==wd_array[i].wd)
                    {
                        memset(target,0,1024);
                        strcpy(target,wd_array[i].path);
                        strcat(target,"/");
                        strcat(target,event->name);
                        printf("nnntarget:%s:n",target); 
                    }
                }
                for(i=0;i<EVENT_NUM;i++)
                {
                    if(event->mask&(1<<i))
                        printf("    event:%sn",event_array[i]);
                }
                tmp_len=sizeof(struct inotify_event)+event->len;
                event=(struct inotify_event*)(offset+tmp_len);
                offset+=tmp_len;
            }
        }
        return 1;
    }

