# 套接字 Socket

## 套接字描述符

是用文件描述符实现的，许多文件描述符的操作函数都可以使用。

创建套接字

```c
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
/* 成功则返回文件描述符，失败返回-1 */
```

- `domain`确定通信的特性，用`AF_`开头，意为address family

| 域          | 描述         |
| ----------- | ------------ |
| `AF_INET`   | IPv4因特网域 |
| `AF_INET6`  | IPv6因特网域 |
| `AF_UNIX`   | UNIX域       |
| `AF_UNSPEC` | 未指定       |

- `type`确定套接字的类型

| 类型             | 描述                                     |
| ---------------- | ---------------------------------------- |
| `SOCK_DGRAM`     | 长度固定、无连接的不可靠报文传输         |
| `SOCK_RAW`       | IP协议的数据报接口                       |
| `SOCK_SEQPACKET` | 长度固定、有序、可靠的面向连接的报文传输 |
| `SOCK_STREAM`    | 有序、可靠、双向的面向连接字节流         |

- 参数`protocol`，通常是0，表示按`domain`和`type`选择默认协议。在`AF_INET`域中，`SOCK_DGRAM`的默认协议是UDP，`SOCK_STREAM`的默认协议是TCP

对于数据报接口`SOCK_DGRAM`，不需要进行逻辑连接，只需要送出一个 报文，地址是对方进程所使用的套接字，而`SOCK_STREAM`在交换数据之前，要求本地套接字和与之通信的远程套接字之间建立一个逻辑连接，对于`SOCK_SEQPACKET`，与套接字类似，是面向连接的，但是发送的不是字节流，而是基于报文的服务（SCTP）。`SOCK_RAW`需要用户程序构造协议首部。

调用`socket()`和调用`open()`是类似的，都是获得输入输出描述符，当不再使用描述符的时候，利用`close()`进行关闭，尽管socket的描述符本质上是文件描述符，但并不是所有的文件描述符的函数都可以使用。

| 函数            | 处理套接字时的行为                                           |
| --------------- | ------------------------------------------------------------ |
| close           | 释放套接字                                                   |
| dup dup2        | 和一般文件描述符一样复制                                     |
| fchdir          | 失败，并将errno设置为ENOTDIR                                 |
| fchmod          | 未规定                                                       |
| fchown          | 由实现定义                                                   |
| fcntl           | 支持一些命令，F_DUPFD，F_GETFD，F_GETFL，F_GETOWN，F_SETFD，F_SETFL，F_SETOWN |
| fdatasync fsync | 由实现定义                                                   |
| fstat           | 不想写了，看书吧                                             |
|                 |                                                              |
|                 |                                                              |
|                 |                                                              |
|                 |                                                              |
|                 |                                                              |
|                 |                                                              |
|                 |                                                              |
|                 |                                                              |
|                 |                                                              |
|                 |                                                              |

禁止套接字的输出shutdown

```c
#include <sys/socket.h>
int shutdown(int sockfd, int how);
```

`how`有`SHUT_RD`、`SHUT_WR`和`SHUT_RDER`，对`close`，是所有的套接字都释放了才可以真正关闭

## 寻址

即如何确定一个目标通信进程，进程的标识有两部分，计算机的网络地址可以标识网络上想与之通信的计算机，而服务则帮助标识目标计算机上特定的进程。

### 字节序

TCP/IP协议栈指定大端字节序

```c
#include <arpa/inet.h>
uint32_t htonl(uint32_t hostint32); //返回网络字节序表示的32位整数
uint16_t htons(uint32_t hostint32); //返回网络字节序表示的16位整数
uint32_t ntohl(uint32_t hostint32); //返回主机字节序表示的32位整数
uint16_t ntohs(uint32_t hostint32); //返回主机字节序表示的16位整数
```

### 地址格式

```c
#include <arpa/inet.h>
const char *inet_ntop(int domain, const void *restrict addr, char *restrict str, socklen_t size);
// 成功则返回地址字符串指针，失败返回NULL
int inet_pton(int domain, const char *restrict str, void *restrict addr);
// 成功返回1，格式无效返回0，出错返回-1
```

### 地址查询

通过调用`gethostent()`，可以找到给定计算机的主机信息。

```c
#include <netdb.h>
struct hostent *gethostent(void); //成功则返回指针，失败则返回NULL
void sethostent(int stayopen);
void endhostent(void);
```

服务是由地址的端口号部分表示的，每一个服务由一个唯一的、熟知的端口号来提供。调用函数`getservbyname()`可以将一个服务名字映射到一个端口号，函数`getservbyport()`讲一个端口号映射到一个服务名，或者采用`getservent()`顺序扫描服务数据库。

```c
#include <netdb.h>
struct servent *getservbyname(const char *name, const char *proto);
struct servent *getservbyport(int port, const char *proto);
struct servent *getservent(void);
// 以上函数，成功返回指针，出错返回NULL
void setservent(int stayopen);
void endservent(void);
```

`getaddrinfo`允许将一个主机名字和服务名字映射到一个地址

```c
#include <sys/socket.h>
#include <netdb.h>
int getaddrinfo(const char *restrict host,
                const char *restrict sevice,
                const struct addrinfo *restrict hint,
                struct addrinfo **restrict res);
void freeaddrinfo(struct addrinfo *ai);
```

> `restrict`只能用来修饰指针，表示是唯一且初始的访问方式

`getnameinfo`将地址转换成主机名或者服务名

```c
#include <sys/socket.h>
#include <netdb.h>
int getnameinfo(const struct sockaddr *restrict addr,
                socklen_t alen,
                char *restrict host,
                socklen_t hostlen,
                char *restrict service,
                socklen_t servlen,
                unsigned int flags);
```

*这块有个好长的程序，后面可以考虑看看*

### 将套接字与地址绑定

与客户端关联的地址没有太大意义，可以让系统选择一个默认地址，对于服务器则需要给接收客户端请求的套接字绑定一个众所周知的地址，客户端应该通过某种方法发现这个地址，最简单的方法是为服务器保留一个地址并在/etc/services或者某个名字服务中注册（是什么？）

通过`bind()`函数将地址绑定到一个套接字

```c
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr *addr, socklen_t len);
//成功则返回0，失败则返回-1
```

- 地址有效，不能是其他机器的地址；
- 必须与创建套接字的时候的地址族所支持的格式相匹配；
- 端口号不小于1024，除非是root；
- 一般只有套接字端点可以与地址绑定（？）

对于因特网域，将IP指定为`INADDR_ANY`，套接字端点可以被绑定到所有的网络接口，可以接收到网卡的所有数据包。如果调用了`listen()`或者`connect()`，但没有绑定到地址，系统会选一个地址进行绑定。

调用`getsockname()`来发现绑定到一个套接字的地址

```c
#include <sys/socket.h>
int getsockname(int sockfd, struct sockaddr *restrict addr, socklen_t *restrict alenp);
```

套接字已经与对方连接时，调用`getpeername()`来获取对方的地址。

```c
#include <sys/socket.h>
int getpeername(int sockfd, struct sockaddr *trstrict addr, socklen_t *restrict alenp);
```

### 建立连接

如果是面向连接的网络服务（SOCK_STREAM或者SOCK_SEQPACKET），需要在开始交换数据之前，在请求服务的进程套接字（客户端）和提供服务的进程套接字（服务器）之间建立连接，利用`connect()`

```c
#include <sys/socket.h>
int connect(int sockfd, const struct sockaddr *addr, socklen_t len);
// 成功返回0，失败返回-1
```

`sockfd`是本地的描述符，如果没有绑定到某个地址，将由`connect`绑定一个默认地址，`addr`是想与之连接的服务器地址。应用程序可以使用`poll`或者`select`来判断文件描述符何时可写，可写的时候，连接完成。

`connect()`还可以用于无连接的网络服务（SOCK_DGRAM），所有发送报文的目标地址都为`connect()`中所指定的地址，这样每次传送报文的时候就不用再指定地址，仅能接收来自指定地址的报文

服务器调用`listen()`来宣告可以接受连接请求。

```c
#include <sys/socket.h>
int listen(int sockfd, int backlog);
// 成功返回0，失败返回-1
```

参数`backlog`提供了一个提示，用于表示该进程所要入队的连接请求数量，一旦服务器调用了`listen()`，套接字就可以接收连接请求，使用函数`accept()`来获得连接请求并建立连接。

```c
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *restrict addr, socklen_t *restrict len);
//成功返回套接字描述符，失败返回-1
```

返回的是一个连接到调用了`connect()`的客户端，这个套接字与之前的具有相同的地址族和套接字类型，但是原始套接字并没有被连接到客户端，而是保持可连接的状态，并接受其他请求。

如果服务器调用`accept()`并且没有连接请求，服务器会阻塞到一个请求到来。服务器可以使用`poll()`或者`select()`来等待一个请求的到来，在这种情况下，一个等待处理的连接请求套接字会以可读的方式出现。

---

本来是想照着书上的程序实现一下，但是发现并不好，写了一部分结果有些东西都没有定义，我觉得可能是在那个apue.h里面，书上的东西写得很详细也很深，但是我一直觉得，对于这些东西，对我来说一口气了解得那么多其实意义不大，我还是秉持着在coding中理解，不会的再去查书，会更有意思，接受的效率也会更高一些。

---

[Socket](https://www.cnblogs.com/jiangzhaowei/p/8261174.html)

网络间进程的识别是通过端口进行的，也就是协议、ip、端口标识了目标主机中的唯一程序

**文件描述符**，打开一个文件就会得到一个文件描述符，是一个很小的正整数，每个进程在PCB（Process Control Block）中保存了一份文件描述符，文件描述符就是这个表的索引，每个表项都指向了一个打开了的文件的指针

**文件指针**，C中使用文件指针作为IO的句柄，文件指针指向的是进程用户区中的一个被称为FILE结构的数据结构，包括一个缓冲区和一个文件描述符，文件描述符是文件描述符表的一个索引

也就是文件指针里面有一个文件描述符

---

[Linux系统结构](https://blog.csdn.net/hguisu/article/details/6122513#t7)

硬盘分区的标识，一般使用/dev/hd\[a~z\]\[1~\]或者/dev/sd\[a~z\]\[1~\]来表示，h是IDE接口的，s是SCSI接口的

分区包括主分区和扩展分区，总数不超过4个（MBR？不确定），扩展分区又可以分为多个逻辑分区，主分区与扩展分区是1~4，逻辑分区从5开始

hda1、hda2、hda5、hda6，那么这几个分区都是来自第一块硬盘的，前面两个是主分区或扩展分区，后面两个一定是逻辑分区，及时硬盘只有一个主分区，逻辑分区的编号也是从5开始的。

分区与目录

- 任何一个分区都必须挂载到某个目录上才能进行读写等操作；
- 目录是逻辑上的区分，分区是物理上的区分；
- 根目录是所有LINUX的文件和目录所在的地方，需要挂载上一个磁盘分区（*这条没看懂*）

主要目录的作用

| 目录        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| /bin        | 二进制可执行命令                                             |
| /dev        | 设备特殊文件                                                 |
| /etc        | 系统管理和配置文件                                           |
| /etc/rc.d   | 启动的配置文件和脚本                                         |
| /home       | 用户主目录的基点，                                           |
| /lib        | 标准程序设计库，又叫动态链接库                               |
| /sbin       | 系统管理命令，存放系统管理员使用的管理程序（比如？           |
| /tmp        | 公用的临时文件存储点                                         |
| /root       | 系统管理员的主目录（呵呵，特权阶级）←这个吐槽笑死我          |
| /mnt        | 让用户临时挂载其他的文件系统                                 |
| /lost+found | 系统非正常关机留下的无家可归的文件                           |
| /proc       | 虚拟目录，是系统内存的映射，可以访问这个目录获取系统信息     |
| /var        | 某些大文件的溢出区                                           |
| /usr        | 最庞大的目录，要用到的应用程序和文件几乎都在这里<br>/usr/X11R6 X window<br>/usr/bin 众多应用程序<br>/usr/sbin 超级用户的一些管理程序<br>/usr/doc linux文档<br>/usr/include 一些头文件<br>/usr/lib 常用的动态链接库和软件包的配置文件<br>/usr/man 帮助文档<br>/usr/src 源代码，内核源码在/usr/src/linux<br>/usr/local/bin 本地增加的命令<br>/usr/local/lib 本地增加的库 |

现在一个分区可以出现多个文件系统（例如LVM），而多个分区也可以合成一个文件系统（例如LVM，RAID）

Linux操作系统的文件权限等参数与数据不在同一个区块，权限放在inode中，而数据放在data block中，还有一个超级区块superblock会记录文件系统的整体信息，包括inode和block的总量、余量等

对于一个磁盘，在指定文件系统后，整个分区被分为1024/2048/4096个字节的块，根据块使用的不同，分为

- **超级块**（superblock）：是磁盘的第一块空间，记录整个文件系统的基本信息，块大小、inode/data block的总量、余量等

**到这里才看了一半，剩下的下次再看= =**

---

基本的socket过程

![1](./pics_socket/1.bmp)

服务器首先初始化socket，然后与端口绑定bind，对端口进行监听listen，调用accept阻塞等待客户端连接，客户端首先初始化一个socket，然后连接服务器，如果连接成功，这时客户端与服务器的连接就建立了。客户端发送数据请求，服务器端接收请求后处理请求，然后把数据发送给客户端，客户端读取数据，然后关闭连接，一次交互结束。

```c
int socket(int protofamily, int type, int protocol); // 返回sockfd
```

sockfd唯一标识一个socket，后续操作都用到这个东西，三个参数分别为

- `protofamily`：协议域，常用的有AF_INET，AF_INET6，AF_LOCAL（或AF_UNIX），AF_ROUTE等等，协议族决定了socket的地址类型，在通信中必须使用相应的地址
- `type`：指定socket的类型，`SOCK_STREAM`，`SOCK_DGRAM`，`SOCK_RAW`，`SOCK_PACKET`，`SOCK_SEQPACKET`等
- `protocol`：常用的有`IPPROTO_TCP`、`IPPROTO_UDP`、`IPPROTO_SCTP`、`IPPROTO_TIPC`等，protocol为0时，会自动选择默认的对应协议

当调用`socket()`创建一个socket的时候，返回的描述字存在于它的协议族空间中，但没有具体的地址，如果想给它一个地址，需要调用`bind()`函数，否则在`connect()`、`listen()`的时候会自动分配一个端口

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen); //
```

把一个地址族（ipv4或者ipv6这样）的特定地址给socket，三个参数

- `sockfd`：socket描述字，通过socket()唯一创建的标识一个socket，bind()就是给这个描述字绑定一个名字
- `addr`：指向要给sockfd绑定的协议地址，这个地址根据创建时地址协议族的不同而不同
- `addrlen`：地址的长度

ipv4

```c
struct sockaddr_in
{
    sa_family_t sin_family;  /* addredd family: AF_INET */
    in_port_t sin_port;      /* port in network byte order */
    struct in_addr sin_addr; /* internet address */
}
struct in_addr
{
    uint32_t s_addr; /* address in network byte order */
}
```

ipv6

```c
structsockaddr_in6 { 
    sa_family_t sin6_family;   /* AF_INET6 */ 
    in_port_t sin6_port;       /* port number */ 
    uint32_t sin6_flowinfo;    /* IPv6 flow information */ 
    struct in6_addr sin6_addr; /* IPv6 address */ 
    uint32_t sin6_scope_id;    /* Scope ID (new in 2.4) */ 
};
struct in6_addr { 
    unsigned char   s6_addr[16]; /* IPv6 address */ 
};
```

Unix

```c
#define UNIX_PATH_MAX    108

struct sockaddr_un { 
    sa_family_t sun_family;               /* AF_UNIX */ 
    char        sun_path[UNIX_PATH_MAX];  /* pathname */ 
};
```

通常服务器在启动的时候会绑定一个众所周知的地址（ip+端口号），用于提供服务，客户端可以连接；而客户端就不需要，只要到时候`connect()`系统分配就行了。这就是为什么服务器在`listen()`之前会调用`bind()`，而客户端就不需要。

**网络字节序和主机字节序**

- Little-Endian：低位字节放在内存低地址，高位字节放在内存的高地址
- Big-Endian：高位字节放在内存的低地址，低位字节放在内存的高地址

网络字节序，4个字节的32位数据首先传送0~7bit，然后8~15位，等等，网络字节序是大端字节序

```c
int listen(int sockfd, int backlog);
```

- `sockfd`，要监听的socket描述字
- `backlog`，可以排队的最大连接个数

`socket()`函数创建的socket默认是主动类型的，`listen()`会将socket变成被动类型的，等待客户的连接。

```c
int connect(int sockfd, const struct socksddr *addr, socklen_t addrlen);
```

- `sockfd`，客户端的socket描述字
- `addr`，服务器的socket地址
- `addrlen`，地址长度

客户端调用`connect()`来与服务器建立TCP连接

TCP服务器在调用`socket()`、`bind()`、`listen()`之后，就会开始监听指定的socket。客户端在调用`socket()`、`connect()`后，就向服务器发送了一个连接请求，TCP服务器监听到这个请求后，就会调用`accept()`接收请求，这样连接就建立了

```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen); //返回连接connect_fd
```

- `sockfd`，监听套接字，当有一个客户端与服务器连接时，它使用一个端口号，这个端口号与这个套接字描述字关联
- `addr`，用来接收一个返回值，是客户端的地址
- `len`，上述`addr`结构的大小，指明`addr`结构占据的字节个数

`accept()`成功返回则成功建立连接了

监听套接字会被复制而返回一个已经与客户端连接的连接套接字，而之前的监听套接字保持存在的状态，方便后续连接

一般有这么些组函数

`read()`/`write()`

`recv()`/`send()`

`readv()`/`writev()`

`recvmsg()`/`sendmsg()`

`recvfrom()`/`sendto()`

```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
```

```c
#include <sys/types.h>
#include <sys/socket.h>
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
ssize_t recv(int sockfd, void *buf, size_t len, int flags);

ssize_t sendto(int sockfd, const void *buf, size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);

ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
ssize_t revcmsg(int sockfd, struct msghdr *msg, int flags);
```

```c
#include <unistd.h>
int close(int fd);
```

调用`close()`之后这个socket描述字就不能再被使用了。`close()`是将socket描述字的引用计数置-1

**TCP的建立，三次握手**

第一次握手，client发送syn（syn = j）到server，进入SYN_SEND状态，等待server确认；SYN：同步序列编号（Synchronize Sequence Numbers）

第二次握手，server接收到syn包，确认client的syn（ack = j+1），同时自己也发送一个syn包（syn = k），即发送SYN+ACK包，此时server进入SYN_RECV状态

第三次握手，client收到server的SYN+ACK，向server发送确认包ACK（ack = k+1），发完客户和服务器进入ESTABLISHED状态，完成握手。

图不好画，不画图了

当client调用`connect()`的时候，发送syn j包，进入阻塞状态，server监听到syn j之后，调用`accept()`，发送syn k和ack j+1包，client收到后，`connect()`	返回，发送ack k+1对server的包进行确认，服务器收到ack k+1后，`accept()`返回，三次握手完毕，

**TCP的终止，四次握手**

TCP是全双工的，所以四个方向必须单独进行关闭，原则上，一方完成其数据发送任务，就可以发送一个FIN来终止这一方向的连接，发起方执行主动关闭，另一方是被动关闭

- 客户端A发送一个FIN，用来关闭A到B的数据传送
- 服务器B接收到FIN，发送一个ACK，确认序号为收到的序号+1，和SYN一样，一个FIN将占用一个序号
- 服务器B关闭与客户端A的连接，发送一个FIN给A
- 客户端A发送ACK，序号为收到的序号+1

其实两者的区别主要是因为建立连接的时候，服务器一下子发送两个包，一个确认包，一个是自己的syn包

实例

```c
/*************************************************************************
	> File Name: server.c
	> Author: ippfcox
	> Mail: ippfcox@163.com 
	> Created Time: Tue 25 Sep 2018 10:46:16 AM CST
 ************************************************************************/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#define PORT 17266
#define MAXLINE 4096

int main(int argc, char **argv)
{
	int sockfd;
	int connfd;
	struct sockaddr_in servaddr;
	char buff[4096];
	int n;

	/* init socket */
	if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1) //注意括号
	{
		printf("create socket error: %s(errno: %d)\n", strerror(errno), errno);
		exit(0);
	}

	memset(&servaddr, 0, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY); //设置IN_ADDRANY，自动分配本机地址
	servaddr.sin_port = htons(PORT);

	/* bind localaddr to the created socket */
	if(bind(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) == -1)
	{
		printf("bind socket error: %s(errno: %d)\n", strerror(errno), errno);
		exit(0);
	}

	/* start to listen */
	if(listen(sockfd, 5) == -1) // 最大连接数5
	{
		printf("listen socket error: %s(errno: %d)\n", strerror(errno), errno);
		exit(0);
	}
	printf("waiting for request...\n");
	while(1) // 阻塞等待
	{
		if((connfd = accept(sockfd, (struct sockaddr *)NULL, NULL)) == -1)
		{
			printf("accept socket error: %s(errno: %d)\n", strerror(errno), errno);
			continue;
		}
		n = recv(connfd, buff, MAXLINE, 0);
		buff[n] = '\0';
		printf("recv msg from client: %s\n", buff); // 不在前面写的话会导致输出两次，子进程还有多的一次
		if(!fork()) //建立了一个发送子进程，靠返回值判断，子进程中返回0，父进程中不为0
		{
			if(send(connfd, "Hi, Andie!\n", 12, 0) < 0)
			{
				perror("send error\n");
				close(connfd);
				exit(1);
			}
		}
		close(connfd);
	}
	close(sockfd);
}

```

```c
/*************************************************************************
	> File Name: client.c
	> Author: ma6174
	> Mail: ma6174@163.com 
	> Created Time: Tue 25 Sep 2018 11:29:28 AM CST
 ************************************************************************/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 17266
#define MAXLINE 4096

int main(int argc, char ** argv)
{
	int sockfd;
	int n;
	int rec_len;
	char recvline[MAXLINE];
	char sendline[MAXLINE];
	struct sockaddr_in servaddr;

    if(argc != 2)
	{
		printf("./client <ipaddress>\n");
		exit(0);
	}

	if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
	{
		printf("create socket error: %s(errno: %d)\n", strerror(errno), errno);
		exit(0);
	}

	memset(&servaddr, 0, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port = htons(PORT);
	if(inet_pton(AF_INET, argv[1], &servaddr.sin_addr) <= 0)
	{
		printf("inet_pton error for %s\n", argv[1]);
		exit(0);
	}

	if((connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr))) < 0)
	{
		printf("connect error: %s(errno: %d)\n", strerror(errno), errno);
		exit(0);
	}

	printf("send msg to server...\n");
	fgets(sendline, 4096, stdin);
	if(send(sockfd, sendline, strlen(sendline), 0) < 0)
	{
		printf("send msg error: %s(errno: %d)\n", strerror(errno), errno);
		exit(0);
	}
	
	if((rec_len = recv(sockfd, recvline, MAXLINE, 0)) == -1)
	{
		perror("recv error\n");
		exit(1);
	}
	recvline[rec_len] = '\0';
	printf("recv: %s\n", recvline);
	close(sockfd);
}
```

以上是TCP的过程

---

UDP：

https://www.cnblogs.com/Pan-Z/p/6628521.html

---

如何获取本地的地址，可以利用`getifaddrs`

