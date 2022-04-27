# Unix套接字编程


<!--more-->

## Socket介绍

所谓 `socket`（套接字），就是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。一个套接字就是网络上进程通信的一端，提供了应用层进程利用网络协议交换数据的机制。从所处的地位来讲，套接字上联应用进程，下联网络协议栈，是应用程序通过网络协议进行通信的接口，是应用程序与网络协议根进行交互的接口。

 socket 可以看成是两个网络应用程序进行通信时，各自通信连接中的端点，这是一个逻辑上的概念。它是网络环境中进程间通信的 API，也是可以被命名和寻址的通信端点，使用中的每一个套接字都有其类型和一个与之相连进程。通信时其中一个网络应用程序将要传输的一段信息写入它所在主机的 socket 中，该 socket 通过与网络接口卡（NIC）相连的传输介质将这段信息送到另外一台主机的 socket 中，使对方能够接收到这段信息。socket 是由 IP 地址和端口结合的，提供向应用层进程传送数据包的机制。

 socket 本身有“插座”的意思，在 Linux 环境下，用于表示**进程间网络通信的特殊文件类型**。本质为内核借助缓冲区形成的伪文件。既然是文件，那么理所当然的，我们可以使用文件描述符引用套接字。与管道类似的，Linux 系统将其封装成文件的目的是为了统一接口，使得读写套接字和读写文件的操作一致。区别是管道主要应用于本地进程间通信，而套接字多应用于网络进程间数据的传递。  

## 字节序

现代 CPU 的累加器一次都能装载（至少）4 字节（这里考虑 32 位机），即一个整数。那么这 4  字节在内存中排列的顺序将影响它被累加器装载成的整数的值，这就是字节序问题。在各种计算机体系结构中，对于字节、字等的存储机制有所不同，因而引发了计算机通信领域中一个很重要的问题，即通信双方交流的信息单元（比特、字节、字、双字等等）应该以什么样的顺序进行传送。如果不达成一致的规则，通信双方将无法进行正确的编码/译码从而导致通信失败。  

**字节序，顾名思义字节的顺序，就是大于一个字节类型的数据在内存中的存放顺序(一个字节的数据当然就无需谈顺序的问题了)**。  

字节序分为**大端字节序**（Big-Endian） 和**小端字节序**（Little-Endian）。大端字节序是指一个整数的最高位字节（23 ~ 31 bit）存储在内存的低地址处，低位字节（0 ~ 7 bit）存储在内存的高地址处；小端字节序则是指整数的高位字节存储在内存的高地址处，而低位字节则存储在内存的低地址处。

### 字节序举例

<img src="https://i0.hdslb.com/bfs/album/01105be36c5929338ea0abd63f996be07cdc2aa2.png" title="" alt="" data-align="center">

### 字节序转换

当格式化的数据在两台使用不同字节序的主机之间直接传递时，接收端必然错误的解释之。解决问题的方法是：**发送端总是把要发送的数据转换成大端字节序数据后再发送，而接收端知道对方传送过来的数据总是采用大端字节序，所以接收端可以根据自身采用的字节序决定是否对接收到的数据进行转换**（小端机转换，大端机不转换）

网络字节顺序是 TCP/IP 中规定好的一种数据表示格式，它与具体的 CPU 类型、操作系统等无关，从而可以保证数据在不同主机之间传输时能够被正确解释，网络字节顺序采用大端排序方式。  

BSD Socket提供了封装好的转换接口，方便程序员使用。包括从主机字节序到网络字节序的转换函数：`htons`、`htonl`；从网络字节序到主机字节序的转换函数：`ntohs`、`ntohl`。

```c
// h   - host 主机，主机字节序
// to  - 转换成什么
// n   - network  网络字节序
// s   - short  unsigned short
// l   - long   unsigned in

#include <arpa/inet.h>
// 转换端口
uint16_t htons(uint16_t hostshort);     // 主机字节序 - 网络字节序
uint16_t ntohs(uint16_t netshort);      // 主机字节序 - 网络字节序
// 转IP
uint32_t htonl(uint32_t hostlong);      // 主机字节序 - 网络字节序
uint32_t ntohl(uint32_t netlong);       // 主机字节序 - 网络字节序
```

## Socket地址

### 通用Socket地址

`socket `网络编程接口中表示 `socket `地址的是结构体 `sockaddr`，其定义如下： 

```c
#include <bits/socket.h>
struct sockaddr {
    sa_family_t sa_family;
    char        sa_data[14];
};
typedef unsigned short int sa_family_t;
```

`sa_family `成员是地址族类型（`sa_family_t`）的变量。地址族类型通常与协议族类型对应。常见的协议族（`protocol family`，也称 `domain`）和对应的地址族入下所示:

| 协议族      | 地址族      | 描述          |
| -------- | -------- | ----------- |
| PF_UNIX  | AF_UNIX  | UNIX本地域协议族  |
| PF_INET  | AF_INET  | TCP/IPv4协议族 |
| PF_INET6 | AF_INET6 | TCP/IPv6协议族 |

宏 `PF_* `和 `AF_*` 都定义在 `bits/socket.h `头文件中，且后者与前者有完全相同的值，所以二者通常混用

`sa_data `成员用于存放 `socket `地址值。但是，不同的协议族的地址值具有不同的含义和长度，如下所示

| 协议族      | 地址值的含义和长度                                          |
| -------- | -------------------------------------------------- |
| PF_UNIX  | 文件的路径名，长度可以达到108字节                                 |
| PF_INET  | 16bit 端口号和32bit IPv4 地址，共6字节                       |
| PF_INET6 | 16bit 端口号，32bit 流标识，128bit IPv6 地址，32bit范围ID,共26字节 |

由上表可知，14 字节的 `sa_data `根本无法容纳多数协议族的地址值。因此，Linux 定义了下面这个新的通用的 socket 地址结构体，这个结构体不仅提供了足够大的空间用于存放地址值，而且是内存对齐的：

```c
#include <bits/socket.h>
struct sockaddr_storage
{
    sa_family_t sa_family;
    unsigned long int __ss_align;
    char __ss_padding[ 128 - sizeof(__ss_align) ];
};
typedef unsigned short int sa_family_t;
```

### 专用Socket地址

很多网络编程函数诞生早于 IPv4 协议，那时候都使用的是 `struct sockaddr `结构体，为了向前兼容，现在sockaddr 退化成了`（void *）`的作用，传递一个地址给函数，至于这个函数是 `sockaddr_in` 还是`sockaddr_in6`，由地址族确定，然后函数内部再强制类型转化为所需的地址类型

<img src="https://i0.hdslb.com/bfs/album/96c89b02d535bca9ad4e2f0903da9b264d6dfb6d.png" title="" alt="" data-align="center">

UNIX 本地域协议族使用如下专用的 socket 地址结构体：

```c
#include <sys/un.h>
struct sockaddr_un
{
    sa_family_t sin_family;
    char sun_path[108];
};
```

TCP/IP 协议族有 `sockaddr_in `和 `sockaddr_in6 `两个专用的 socket 地址结构体，它们分别用于 IPv4 和 IPv6:

```c
#include <netinet/in.h>
struct sockaddr_in
{
    sa_family_t sin_family;     /* __SOCKADDR_COMMON(sin_) */
    in_port_t sin_port;         /* Port number.  */
    struct in_addr sin_addr;    /* Internet address.  */
    /* Pad to size of `struct sockaddr'. */
    unsigned char sin_zero[sizeof (struct sockaddr) - __SOCKADDR_COMMON_SIZE -
               sizeof (in_port_t) - sizeof (struct in_addr)];
};  
struct in_addr
{
    in_addr_t s_addr;
};
struct sockaddr_in6
{
    sa_family_t sin6_family;
    in_port_t sin6_port;    /* Transport layer port # */
    uint32_t sin6_flowinfo; /* IPv6 flow information */
    struct in6_addr sin6_addr;  /* IPv6 address */
    uint32_t sin6_scope_id; /* IPv6 scope-id */
 };
typedef unsigned short  uint16_t;
typedef unsigned int    uint32_t;
typedef uint16_t in_port_t;
typedef uint32_t in_addr_t;
#define __SOCKADDR_COMMON_SIZE (sizeof (unsigned short int))
```

所有专用 socket 地址（以及 `sockaddr_storage`）类型的变量在实际使用时都需要转化为通用 socket 地址类型 `sockaddr`（强制转化即可），因为所有 socket 编程接口使用的地址参数类型都是 `sockaddr`。

## IP地址转换

通常，人们习惯用可读性好的字符串来表示 IP 地址，比如用点分十进制字符串表示 IPv4 地址，以及用十六进制字符串表示 IPv6 地址。但编程中我们需要先把它们转化为整数（二进制数）方能使用。而记录日志时则相反，我们要把整数表示的 IP 地址转化为可读的字符串。下面 3 个函数可用于用点分十进制字符串表示的 IPv4 地址和用网络字节序整数表示的 IPv4 地址之间的转换：

```c
#include <arpa/inet.h>
in_addr_t inet_addr(const char *cp);
int inet_aton(const char *cp, struct in_addr *inp);
char *inet_ntoa(struct in_addr in);
```

下面这对更新的函数也能完成前面 3 个函数同样的功能，并且它们同时适用 IPv4 地址和 IPv6 地址：

```c
#include <arpa/inet.h>
// p:点分十进制的IP字符串，n:表示network，网络字节序的整数
int inet_pton(int af, const char *src, void *dst);
    af:地址族： AF_INET  AF_INET6
    src:需要转换的点分十进制的IP字符串
    dst:转换后的结果保存在这个里面
// 将网络字节序的整数，转换成点分十进制的IP地址字符串
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
    af:地址族： AF_INET  AF_INET6
    src: 要转换的ip的整数的地址
    dst: 转换成IP地址字符串保存的地方
    size：第三个参数的大小（数组的大小）
    返回值：返回转换后的数据的地址（字符串），和 dst 是一样的
```

## 套接字函数

```c
#include <sys/types.h>      
#include <sys/socket.h>
#include <arpa/inet.h>  // 包含了这个头文件，上面两个就可以省略
int socket(int domain, int type, int protocol);
    - 功能：创建一个套接字
    - 参数：
        - domain: 协议族
            AF_INET : ipv4
            AF_INET6 : ipv6
            AF_UNIX, AF_LOCAL : 本地套接字通信（进程间通信）
        - type: 通信过程中使用的协议类型
            SOCK_STREAM : 流式协议
            SOCK_DGRAM  : 报式协议
        - protocol : 具体的一个协议。一般写0
            - SOCK_STREAM : 流式协议默认使用 TCP
            - SOCK_DGRAM  : 报式协议默认使用 UDP
        - 返回值：
            - 成功：返回文件描述符，操作的就是内核缓冲区。
            - 失败：-1         
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen); // socket命
名
    - 功能：绑定，将fd 和本地的IP + 端口进行绑定
    - 参数：
            - sockfd : 通过socket函数得到的文件描述符
            - addr : 需要绑定的socket地址，这个地址封装了ip和端口号的信息
            - addrlen : 第二个参数结构体占的内存大小

int listen(int sockfd, int backlog);    // /proc/sys/net/core/somaxconn
    - 功能：监听这个socket上的连接
    - 参数：
        - sockfd : 通过socket()函数得到的文件描述符
        - backlog : 未连接的和已经连接的和的最大值， 5
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
    - 功能：接收客户端连接，默认是一个阻塞的函数，阻塞等待客户端连接 
    - 参数：
            - sockfd : 用于监听的文件描述符
            - addr : 传出参数，记录了连接成功后客户端的地址信息（ip，port）
            - addrlen : 指定第二个参数的对应的内存大小
    - 返回值：
            - 成功 ：用于通信的文件描述符 
            - -1 ： 失败

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
    - 功能： 客户端连接服务器
    - 参数：
            - sockfd : 用于通信的文件描述符
            - addr : 客户端要连接的服务器的地址信息
            - addrlen : 第二个参数的内存大小
    - 返回值：成功 0， 失败 -1
ssize_t write(int fd, const void *buf, size_t count);   // 写数据
ssize_t read(int fd, void *buf, size_t count);          // 读数
```

示例如下：

#### 服务器端

```c
#include <arpa/inet.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main()
{

    // 1 创建socket， 用于监听
    int lfd = socket(AF_INET, SOCK_STREAM, 0);
    if (lfd == -1)
    {
        perror("socket");
        exit(-1);
    }
    // 2 绑定端口
    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;
    // inet_pton(AF_INET, "172.25.93.158", &sockaddr.sin_addr.s_addr);
    saddr.sin_addr.s_addr = INADDR_ANY;
    saddr.sin_port = htons(9999);

    int ret = bind(lfd, (struct sockaddr *)&saddr, sizeof(saddr));
    if (ret == -1)
    {
        perror("bind");
        exit(-1);
    }

    // 3 监听
    ret = listen(lfd, 8);
    if (ret == -1)
    {
        perror("listen");
        exit(-1);
    }

    // 4 接受连接
    struct sockaddr_in clientaddr;
    socklen_t len = sizeof(clientaddr);
    int cfd = accept(lfd, (struct sockaddr *)&clientaddr, &len);
    if (cfd == -1)
    {
        perror("accept");
        exit(-1);
    }

    // 输出客户端信息
    char clientIP[16];
    inet_ntop(AF_INET, &clientaddr.sin_addr.s_addr, clientIP, sizeof(clientIP));
    unsigned short clientPort = ntohs(clientaddr.sin_port);
    printf("client Ip is %s, client port is %d\n", clientIP, clientPort);

    // 5 通信
    // 获取客户端数据
    char recvBuf[1024] = {0};
    while (1)
    {
        ret = read(cfd, recvBuf, sizeof(recvBuf));
        if (ret == -1)
        {
            perror("read");
            exit(-1);
        }
        else if (ret > 0)
        {
            printf("receive client data : %s\n", recvBuf);
        }
        else if (ret == 0)
        {
            printf("client closed...\n");
            break;
        }

        // 给客户端发送数据
        char sendBuf[] = "hello I am server";
        ret = write(cfd, sendBuf, sizeof(sendBuf) - 1);
        if (ret == -1)
        {
            perror("write");
            exit(-1);
        }
    }

    // 6 关闭连接
    close(cfd);
    close(lfd);
    return 0;
}
```

#### 客户端

```c
#include <arpa/inet.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main()
{

    // 1 创建套接字
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd == -1)
    {
        perror("socket");
        exit(-1);
    }
    // 2 连接服务器
    struct sockaddr_in serverAddr;
    serverAddr.sin_family = PF_INET;
    inet_pton(AF_INET, "172.25.93.158", &serverAddr.sin_addr.s_addr);
    serverAddr.sin_port = htons(9999);

    int ret = connect(fd, (struct sockaddr *)&serverAddr, sizeof(serverAddr));
    if (ret == -1)
    {
        perror("connect");
        exit(-1);
    }

    // 3 通信

    char recvBuf[1024] = {0};
    while (1)
    {

        char sendBuf[] = "hello I am client";
        ret = write(fd, sendBuf, sizeof(sendBuf) - 1);
        if (ret == -1)
        {
            perror("write");
            exit(-1);
        }
        sleep(1);   
        ret = read(fd, recvBuf, sizeof(recvBuf));
        if (ret == -1)
        {
            perror("read");
            exit(-1);
        }
        else if (ret > 0)
        {
            printf("receive server data : %s\n", recvBuf);
        }
        else if (ret == 0)
        {
            printf("server closed...\n");
            break;
        }
    }

    // 4 关闭连接
    close(fd);

    return 0;
} 
```

## TCP通信流程

<img src="https://i0.hdslb.com/bfs/album/2aa381a09efab3e9551f4d46adc07782dff6f658.png" title="" alt="" data-align="center">

### 服务器端 （被动接受连接的角色）

1. **创建**一个用于监听的套接字  （监听：监听有客户端的连接；套接字：这个套接字其实就是一个文件描述符  ）

2. 将这个**监听文件描述符**和本地的IP和端口**绑定**（IP和端口就是服务器的地址信息）客户端连接服务器的时候使用的就是这个IP和端口

3. **设置监听**，监听的fd开始工作 

4. 阻塞等待，当有客户端发起连接，解除阻塞，**接受**客户端的连接，会得到一个和客户端通信的套接字（fd）  

5. **通信**  接收数据、发送数据  

6. 通信结束，**断开**

### 客户端

1. **创建**一个用于通信的套接字（fd）

2. **连接**服务器，需要指定连接的服务器的 IP 和 端口

3. 连接成功了，客户端可以直接和服务器**通信**
   
   - 接收数据 
   
   - 发送数据

4. 通信结束，**断开**

### 多进程TCP通信

#### 服务端

```c
#include <stdio.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/types.h>
#include <unistd.h> 
#include <signal.h>
#include <wait.h>
#include <errno.h>

void recyleChild(int arg) {
    // SIGCHILD信号处理函数
    // 用于回收资源
    while (1) {
        int ret = waitpid(-1, NULL, WNOHANG);
        if (ret == -1) {
            // 所有子进程都被回收
            break;
        } else if (ret == 0) {
            // 还有子进程
            break;
        } else if (ret > 0) {
            printf("子进程 %d 被回收了\n", ret);
        }
    }
    return;
}

int main() {
    // 注册信号，当子进程退出时，回收资源
    struct sigaction act;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    act.sa_handler = recyleChild;
    sigaction(SIGCHLD, &act, NULL);

    // 1 创建监听socket 
    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    if (lfd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2 为监听socket绑定端口 bind
    struct sockaddr_in server_addr; 
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9999);
    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr.s_addr);
    int ret = bind(lfd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    if (ret == -1) {
        perror("bind");
        exit(-1);
    }

    // 3 开始监听 listen
    ret = listen(lfd, 128);
    if (ret == -1) {
        perror("listen");
        exit(-1);
    }
    // 4 循环等待接受请求 accept
    while (1) {
        struct sockaddr_in client_add
r;
        socklen_t client_socket_len = sizeof(client_addr);
        // 这里的第三个参数要传长度的指针
        int cfd = accept(lfd, (struct sockaddr *)&client_addr, &client_socket_len);
        if (cfd == -1) {
            // 这里为了避免因为accept被回收资源函数中断而退出程序
            if(errno == EINTR) {
                continue;
            }
            perror("accept");
            exit(-1);
        }
        printf("client connected, clientSockedFd: %d\n", cfd);

        // 5 创建新进程
        pid_t pid = fork();
        if (pid == 0) {
            // 6 新进程与客户socket通信
            char buf[1024] = {0};
            while (1) {
                // 循环从客户端读取
                ret = read(cfd, buf, sizeof(buf));
                if (ret == -1) {
                    perror("read");
                    exit(-1);
                } else if (ret > 0) {
                    printf("server read data: %s \n", buf);
                } else if (ret == 0) {
                    // 读完了
                    printf("client closed...\n");
                    break;
                }
                // 向客户端写入
                ret = write(cfd, buf, sizeof(buf));
                if (ret == -1) {
                    perror("write");
                    exit(-1);
                }
            }
            close(cfd);
            exit(0); // 子进程退出
        } 

    }
    // 7 回收子进程资源

    // 8 关闭连接
    close(lfd);

    return 0;
}
```

#### 客户端

```c
#include <stdio.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>

int main() {
    // 1 创建socket
    int cfd = socket(PF_INET, SOCK_STREAM, 0);
    if (cfd == -1) {
        perror("socket");
        exit(-1);
    }
    // 2 建立连接 connect
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9999);
    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr.s_addr);
    // 注意要类型转换
    int ret = connect(cfd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret == -1) {
        perror("connect");
        exit(-1);
    }

    // 3 通信
    char buf[1024] = {0};
    while (1) {
        scanf("%s", buf);
        write(cfd, buf, sizeof(buf));
        int len = read(cfd, buf, sizeof(buf));
        if (len == -1) {
            perror("read");
            exit(-1);
        } else if (len > 0) {
            if (strcmp(buf, "quit") == 0) {
                printf("client quit... \n");
                break;
            } else {
                printf("receive server data: %s\n", buf);
            }
        } else if (len == 0){
            // 读取完了服务器断开连接
            printf("server closed");
            break;
        }
    }

    // 4 关闭连接
    close(cfd);

    return 0;
}
```

### 多线程TCP通信

#### 服务端

```c
#include <stdio.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <pthread.h>
#include <string.h>

struct sockInfo
{
    // 线程号
    pthread_t tid;
    // 通信的文件描述符
    int fd;
    struct sockaddr_in addr;

};

struct sockInfo sockinfos[128];

void* work(void * arg) 
{
    // 6 在新线程中通信
    struct sockInfo* pinfo = (struct sockInfo*) arg;
    int cfd = pinfo->fd;
    // 打印客户端的IP和端口
    char clientIp[16];
    inet_ntop(AF_INET, &pinfo->addr.sin_addr.s_addr, clientIp, sizeof(clientIp));
    unsigned short clientPort = ntohs(pinfo->addr.sin_port);
    printf("client ip is : %s, port is : %d\n", clientIp, clientPort);

    char buf[1024];
    while (1) {
        // 循环从客户端读取
        int len = read(cfd, buf, sizeof(buf));
        if (len == -1) {
            perror("read");
            exit(-1);
        } else if (len > 0) {
            // 接收到了数据
            printf("server reveiced data: %s\n", buf);
        } else if (len == 0) {
            // 读完了，关闭连接
            printf("client closed\n");
            break;
        }
        // 往客户端写数据
        write(pinfo->fd, buf, sizeof(buf));
    }
    close(pinfo->fd);
    return NULL;

}


int main() {
    // 1 创建socket
    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    if (lfd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2 为socket绑定IP端口 bind
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9999);
    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr.s_addr);
    int ret = bind(lfd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    if (ret == -1) {
        perror("bind");
        exit(-1);
    }

    // 3 开启监听 listen
    ret = listen(lfd, 128);
    if (ret == -1) {
        perror("listen");
        exit(-1);
    }

    // 初始化线程池
    int max = sizeof(sockinfos) / sizeof(sockinfos[0]);
    for (int i = 0; i < max; ++i) {
        bzero(&sockinfos[i], sizeof(sockinfos[i]));
        sockinfos[i].fd = -1;
        sockinfos[i].tid = -1;
    }
    while (1) {
        // 4 循环监听 接受连接 accept
        struct sockaddr_in client_addr;
        int len = sizeof(client_addr);
        int cfd = accept(lfd, (struct sockaddr*)&client_addr, &len);
        if (cfd == -1) {
            perror("accept");
            exit(-1);
        }
        // 5 开启新线程

        // 寻找可用线程
        struct sockInfo *pinfo;
        for (int i = 0; i < max; ++i) {
            if (sockinfos[i].fd == -1) {
                pinfo = &sockinfos[i];
                break;
            } else {
                // 线程池满
                if (i == max - 1) {
                    sleep(1);
                    --i;
                }
            }
        }
        // 记录客户端文件描述符
        pinfo->fd = cfd;
        // 记录客户端地址信息
        memcpy(&pinfo->addr, &client_addr, sizeof(client_addr));
        // 创建线程
        pthread_create(&pinfo->tid, NULL, work, pinfo);

        // 分离线程
        pthread_detach(pinfo->tid);

    }

    // 7 关闭
    close(lfd);
    return 0;
}
```

## UDP通信流程

<img src="https://i0.hdslb.com/bfs/album/92a4b797fd76a5bf3d65d976dfe969bb5e019e0e.png" title="" alt="" data-align="center">

```c
#include <sys/types.h>
#include <sys/socket.h>
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
                      const struct sockaddr *dest_addr, socklen_t addrlen);
        - 参数：
            - sockfd : 通信的fd
            - buf : 要发送的数据
            - len : 发送数据的长度
            - flags : 0
            - dest_addr : 通信的另外一端的地址信息
            - addrlen : 地址的内存大小

ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                        struct sockaddr *src_addr, socklen_t *addrlen);
        - 参数：
            - sockfd : 通信的fd
            - buf : 接收数据的数组
            - len : 数组的大小 
            - flags : 0
            - src_addr : 用来保存另外一端的地址信息，不需要可以指定为NULL
            - addrlen : 地址的内存大
```

### 服务端

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main() {

    // 1.创建一个通信的socket
    int fd = socket(PF_INET, SOCK_DGRAM, 0);

    if(fd == -1) {
        perror("socket");
        exit(-1);
    }   

    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);
    addr.sin_addr.s_addr = INADDR_ANY;

    // 2.绑定
    int ret = bind(fd, (struct sockaddr *)&addr, sizeof(addr));
    if(ret == -1) {
        perror("bind");
        exit(-1);
    }

    // 3.通信
    while(1) {
        char recvbuf[128];
        char ipbuf[16];

        struct sockaddr_in cliaddr;
        int len = sizeof(cliaddr);

        // 接收数据
        int num = recvfrom(fd, recvbuf, sizeof(recvbuf), 0, (struct sockaddr *)&cliaddr, &len);

        printf("client IP : %s, Port : %d\n", 
            inet_ntop(AF_INET, &cliaddr.sin_addr.s_addr, ipbuf, sizeof(ipbuf)),
            ntohs(cliaddr.sin_port));

        printf("client say : %s\n", recvbuf);

        // 发送数据
        sendto(fd, recvbuf, strlen(recvbuf) + 1, 0, (struct sockaddr *)&cliaddr, sizeof(cliaddr));

    }

    close(fd);
    return 0;
}
```

### 客户端

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

int main() {

    // 1.创建一个通信的socket
    int fd = socket(PF_INET, SOCK_DGRAM, 0);

    if(fd == -1) {
        perror("socket");
        exit(-1);
    }   

    // 服务器的地址信息
    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;
    saddr.sin_port = htons(9999);
    inet_pton(AF_INET, "127.0.0.1", &saddr.sin_addr.s_addr);

    int num = 0;
    // 3.通信
    while(1) {

        // 发送数据
        char sendBuf[128];
        sprintf(sendBuf, "hello , i am client %d \n", num++);
        sendto(fd, sendBuf, strlen(sendBuf) + 1, 0, (struct sockaddr *)&saddr, sizeof(saddr));

        // 接收数据
        int num = recvfrom(fd, sendBuf, sizeof(sendBuf), 0, NULL, NULL);
        printf("server say : %s\n", sendBuf);

        sleep(1);
    }

    close(fd);
    return 0;
}
```

## 参考

[Socket原理详解 - 编程宝库 (codebaoku.com)](http://www.codebaoku.com/it-c/it-c-230866.html)

