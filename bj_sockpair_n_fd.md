我们知道父进程在子进程被fork出来之前打开的文件描述符是能被子进程继承下来的，但是一旦子进程已经创建后，父进程打开的文件描述符要怎样才能传递给子进程呢？Unix提供相应的技术来满足这一需求，这就是同一台主机上进程间的文件描述符传递，很美妙而且强大的技术。





想象一下我们试图实现一个服务器，接收多个客户端的连接，我们欲采用多个子进程并发的形式来处理多客户端的同时连接，这时候我们可能有两种想法：
1、客户端每建立一条连接，我们fork出一个子进程负责处理该连接；
2、预先创建一个进程池，客户端每建立一条链接，服务器就从该池中选出一个空闲(Idle)子进程来处理该连接。
后者显然更高效，因为减少了子进程创建的性能损耗，反应的及时性大大增强。这里恰恰就出现了我们前面提到的问题，所有子进程都是在服务器Listen到一条连接以前就已经fork出来了，也就是说新的连接描述符子进程是不知道的，需要父进程传递给它，它接收到相应的连接描述符后，才能与相应的客户端进行通信处理。这里我们就可以使用'传递文件描述符'的方式来实现。





关于sendmsg/recvmsg
通过socket发送数据，主要有三组系统调用，分别是：

- send/recv(与write/read类似，面向连接的)
  
- sendto/recvfrom(sendto与send的差别在于，sendto可以面向无连接,recvfrom与recv的区别是,recvfrom可以获取sender方的地址)
  
- sendmsg/recvmsg. 通过sendmsg,可以用msghdr参数，来指定多个缓冲区来发送数据，与writev系统调用类似。
  
    
  
    sendmsg函数原型如下:

```c
#include <sys/socket.h>
ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
```



其中，根据POSIX.1 msghdr的定义至少应该包含下面几个成员：

```c
struct msghdr {
    void *msg_name; /* optional address */
    socklen_t msg_namelen; /* address size in bytes */
    struct iovec *msg_iov; /* array of I/O buffers */
    int msg_iovlen; /* number of elements in array */
    void *msg_control; /* ancillary data */
    socklen_t msg_controllen; /* number of ancillary bytes */
    int msg_flags; /* flags for received message */
};
```


在Linux的manual page中，msghdr的定义为:

```c
struct msghdr {
    void         *msg_name;       /* optional address */
    socklen_t     msg_namelen;    /* size of address */
    struct iovec *msg_iov;        /* scatter/gather array */
    size_t        msg_iovlen;     /* # elements in msg_iov */
    void         *msg_control;    /* ancillary data, see below */
    socklen_t     msg_controllen; /* ancillary data buffer len */
    int           msg_flags;      /* flags on received message */
};
```


查看Linux内核源代码(3.18.1)，可知msghdr的准确定义为：

```c
struct msghdr {
    void        *msg_name;  /* ptr to socket address structure */
    int     msg_namelen;    /* size of socket address structure */
    struct iovec    *msg_iov;   /* scatter/gather array */
    __kernel_size_t msg_iovlen; /* # elements in msg_iov */
    void        *msg_control;   /* ancillary data */
    __kernel_size_t msg_controllen; /* ancillary data buffer length */
    unsigned int    msg_flags;  /* flags on received message */
};
```

可见，与Manual paga中的描述一致

![img](E:\workLinux\homeworks\image\20130612182041437)

其中，前两个成员`msg_name`和`msg_namelen`是用来在发送datagram时，指定目的地址的。如果是面向连接的，这两个成员变量可以不用。

接下来的两个成员,`msg_iov`和`msg_iovlen`，则是用来指定发送缓冲区数组的。其中，`msg_iovlen`是iovec类型的元素的个数。每一个缓冲区的起始地址和大小由iovec类型自包含，iovec的定义为：

```c
struct iovec {
    void *iov_base;   /* Starting address */
    size_t iov_len;   /* Number of bytes */
};
```


成员`msg_flags`用来描述接受到的消息的性质,由调用recvmsg时传入的flags参数设置。`recvmsg`的函数原型为:

```c
#include <sys/socket.h>
ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
```

与`sendmsg`相对应，<span style="color: rgba(255, 0, 0, 1)">recvmsg用msghdr结构指定多个缓冲区来存放读取到的结果</span>。flags参数用来修改recvmsg的默认行为。传入的flags参数在调用完recvmsg后，会被设置到msg所指向的msghdr类型的msg_flags变量中。

回来继续讲sendmsg和msghdr结构。

msghdr结构中剩下的两个成员,msg_control和msg_contorllen,是用来发送或接收控制信息的。其中,msg_control指向一个cmsghdr的结构体,msg_controllen成员是控制信息所占用的字节数。

注意,msg_controllen与前面的msg_iovlen不同,msg_iovlen是指的由成员msg_iov所指向的iovec型的数组的元素个数,而msg_controllen,则是所有控制信息所占用的总的字节数。

其实,msg_control也可能是个数组,但msg_controllen并不是该cmsghdr类型的数组的元素的个数。在Manual page中,关于msg_controllen有这么一段描述:



> To create ancillary data, first initialize the msg_controllen member of the msghdr with the length of the control message buffer. Use CMSG_FIRSTHDR() on the msghdr to get the first control message and CMSG_NEXTHDR to get all subsequent ones. In each control message, initialize cmsg_len (with CMSG_LEN), the other cmsghdr header fields, and the data portion using CMSG_DATA. Finally, the msg_controllen field of the msghdr should be set to the sum of the CMSG_SPACE() of the length of all control messages in the buffer.

在Linux 的Manual page(man cmsg)中,cmsghdr的定义为:

```c
struct cmsghdr {
    socklen_t   cmsg_len;   /* data byte count, including header */
    int         cmsg_level; /* originating protocol */
    int         cmsg_type;  /* protocol-specific type */
    /* followed by  unsigned char   cmsg_data[]; */
};
```


注意,<span color style="color: rgba(255, 0, 0, 1)">控制信息的数据部分,是直接存储在cmsg_type之后的</span>。但中间可能有一些由于对齐产生的填充字节,由于这些填充数据的存在，对于这些控制数据的访问,必须使用Linux提供的一些专用宏来完成。这些宏包括如下几个:

```c
#include <sys/socket.h>

struct cmsghdr *CMSG_FIRSTHDR(struct msghdr *msgh);
struct cmsghdr *CMSG_NXTHDR(struct msghdr *msgh, struct cmsghdr *cmsg);
size_t CMSG_ALIGN(size_t length);
size_t CMSG_SPACE(size_t length);
size_t CMSG_LEN(size_t length);
unsigned char *CMSG_DATA(struct cmsghdr *cmsg);
```


其中:

`CMSG_FIRSTHDR()`返回msgh所指向的msghdr类型的缓冲区中的第一个cmsghdr结构体的指针。

`CMSG_NXTHDR()`返回传入的cmsghdr类型的指针的下一个cmsghdr结构体的指针。

`CMSG_ALIGN()`根据传入的length大小,返回一个包含了添加对齐作用的填充数据后的大小。

`CMSG_SPACE()`中传入的参数length指的是一个控制信息元素(即一个cmsghdr结构体)后面数据部分的字节数,返回的是这个控制信息的总的字节数,即包含了头部(即cmsghdr各成员)、数据部分和填充数据的总和。

`CMSG_DATA`根据传入的cmsghdr指针参数,返回其**后面数据部分**的指针。

`CMSG_LEN`传入的参数是一个<span color style="color: rgba(250,0,0,1)">控制信息中的数据部分的大小</span>,返回的是这个根据这个数据部分大小,需要配置的cmsghdr结构体中cmsg_len成员的值。这个大小将为对齐添加的填充数据也包含在内。

用一张图来表示这几个变量和宏的关系为:
￼

如前所述,msghdr结构中,msg_controllen成员的大小为所有cmsghdr控制元素调用CMSG_SPACE()后相加的和。

讲了这么多msghdr,cmsghdr,还是没有讲到如何传递文件描述符。其实很简单,本来sendmsg是和send一样,是用来传送数据的,只不过其数据部分的buffer由参数msg_iov来指定,至此,其行为和send可以说是类似的。

<span color style="color: rgba(255, 0, 0, 1)">但是sendmsg提供了可以传递控制信息的功能,我们要实现的传递描述符这一功能,就必须要用到这个控制信息。</span>在msghdr变量的cmsghdr成员中,<span color style="color: rgba(255, 0, 0, 1)">由控制头cmsg_level和cmsg_type来设置"传递文件描述符"</span>这一属性,并将要传递的文件描述符作为数据部分,保存在cmsghdr变量的后面。这样就可以实现传递文件描述符这一功能,在此时,是不需要使用msg_iov来传递数据的。

具体的说,为msghdr的成员msg_control分配一个cmsghdr的空间,将该cmsghdr结构的cmsg_level设置为SOL_SOCKET,cmsg_type设置为SCM_RIGHTS,并将要传递的文件描述符作为数据部分,调用sendmsg即可。其中,

SCM表示`socket-level control message`,

SCM_RIGHTS表示我们要**传递访问权限**。



弄清楚了发送部分,文件描述符的接收部分就好说了。跟发送部分一样,为控制信息配置好属性,并在其后分配一个文件描述符的数据部分后，在成功调用recvmsg后,控制信息的数据部分就是在接收进程中的新的文件描述符了,接收进程可直接对该文件描述符进行操作。

```c
//实现
int write_fd(int fd, void *ptr, int nbytes, int sendfd)
{
    struct msghdr msg;
    struct iovec iov[1];
    // 有些系统使用的是旧的msg_accrights域来传递描述符，Linux下是新的msg_control字段
#ifdef HAVE_MSGHDR_MSG_CONTROL
    union{ // 前面说过，保证cmsghdr和msg_control的对齐
        struct cmsghdr cm;
        char control[CMSG_SPACE(sizeof(int))];
    }control_un;
    struct cmsghdr *cmptr;
    // 设置辅助缓冲区和长度
    msg.msg_control = control_un.control;
    msg.msg_controllen = sizeof(control_un.control);
    // 只需要一组附属数据就够了，直接通过CMSG_FIRSTHDR取得
    cmptr = CMSG_FIRSTHDR(&msg);
    // 设置必要的字段，数据和长度
    cmptr->cmsg_len = CMSG_LEN(sizeof(int)); // fd类型是int，设置长度
    cmptr->cmsg_level = SOL_SOCKET;
    cmptr->cmsg_type = SCM_RIGHTS;  // 指明发送的是描述符
    *((int*)CMSG_DATA(cmptr)) = sendfd; // 把fd写入辅助数据中
#else
    msg.msg_accrights = (caddr_t)&sendfd; // 这个旧的更方便啊
    msg.msg_accrightslen = sizeof(int);
#endif
    // UDP才需要，无视
    msg.msg_name = NULL;
    msg.msg_namelen = 0;
    // 别忘了设置数据缓冲区，实际上1个字节就够了
    iov[0].iov_base = ptr;
    iov[0].iov_len = nbytes;
    msg.msg_iov = iov;
    msg.msg_iovlen = 1;
    return sendmsg(fd, &msg, 0);
} 
```







