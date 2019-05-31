---
title: UNIX之文件IO
date: 2019-05-31 22:47:15
tags: [Unix, c语言, 操作系统]
categories: UNIX环境高级编程
---

# 文件I/O

## 文件描述符

文件描述符是一个非负整数，对于UNIX内核而言，所有打开的文件都通过文件描述符引用，当打开一个现有文件或新建一个文件时候（比如使用`open`函数），**内核**会向进程返回 一个文件描述符。

按照惯例，UNIX系统shell将进程的标准输入输出等与对应的文件描述符相关联,定义在`<unistd.h>` 中。

| I/O           | 文件描述符 |
| ------------- | ---------- |
| STDIN_FILENO  | 0          |
| STDOUT_FILENO | 1          |
| STDERR_FILENO | 2          |

## 函数`open`和`openat`

```c
#include <fcntl.h> //file control
int open(const char* path, int oflag, ... /* mode_t mode */); 
int openat(int fd, const char* path, int oflag, ... /* mode_t mode */);
// 两函数的返回值：若成功返回文件描述符，出错则返回-1.
```

参数`path`为要打开的文件名字，`oflag`用来说明此函数的多个选项，一般用一个或多个常量进行"或"运算构成。常用的常量有`O_RDONLY,O_WRONLY,O_RDWR`,参数`mode`用于指定新文件的访问权限位

由`open`和`openat` 返回的文件描述符一定是最小的未用描述符数值。

`fd`参数将两者区分开来，共有三种可能性：

1. `path` 参数指定的是绝对路径名，在这种情况下，`fd`参数被忽略，`openat`函数相当于`open`函数。
2. `path`参数指定的是相对路径名，`fd`参数指出了相对路径名在文件系统中的开始地址。`fd`参数是通过打开相对路径名所在的目录来获取。
3. `path`参数指定 了相对路径名，`fd`参数具有特殊值`AT_FDCWD`。在这种情况下，路径名在当前的工作目录中获取，`openat` 函数在操作上与`open`函数类似。

`openat`函数希望解决两个问题，一是让线程可以使用相对路径打开目录中的文件，而不再只能打开当前工作目录。二是可以避免time-of-check-to-time-of-use(TOCTTOU)错误。



## 函数`creat`

```c
#include <fcntl.h>
int create(const char* path, mode_t mode);
// 若成功返回为只写打开的文件描述符；若出错，则返回-1.
// 此函数等效于:
open(path, O_WRONLY|O_CREAT|O_TRUNC, mode);
```

`create`函数的不足是它只以写的方式打开所创建的文件。现在可以用`open`实现写和读方式的打开：

```c
open(path, O_RDWR|O_CREAT|OTRUNC, mode);
```

## 函数`close`

```c
#include <unistd.h>
int close(int fd);
// 若成功返回0，出错返回-1.
```

关闭一个文件时还会释放该进程加在该文件上的所有记录锁，当一个进程终止时，内核自动关闭它打开的所有文件

## 函数`lseek`

可以使用`lseek`显示地为一个打开文件设置偏移量(current file offset)。

```c
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);
// 若成功返回新的文件偏移量，出错返回-1
```

对参数`offset`的解释与参数`whence`有关：

- 若`whence`是`SEEK_SET`,则将该文件的偏移量设置为距离文件开始处`offset`个字节。
- 若`whence`是`SEEK_CUR`,则将该文件的偏移量设置为距离当前值加`offset`个字节。
- 若`whence`是`SEEK_END`,则将该文件的偏移量设置为文件长度加`offset`个字节，`offset`可正可负。

如果文件描述符指向的是一个管道、FIFO或网络套接字，则该函数返回-1,并将errno设置为ESPIPE。

文件偏移量可以大于当前文件长度，这种情况下，对该文件的下一次写将加长该文件，并在文件中形成一个空洞，处于文件中，但没有写过的文件读为0，文件中的空洞并不要求在磁盘上占用存储区。

该函数只记录当前文件偏移量，不会进行I/O操作。

## 函数`read` 

调用`read`函数从打开文件中读数据

```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t nbytes);
//返回值：读到的字节数，若已到文件尾返回0，出错返回-1.
```

第二个参数表示通用指针，第三个参数中`size_t`是不带符号的整型，用于表示要读取的字节数。返回值为带符号的整型。

## 函数`write` 

调用`write`函数向打开文件写数据

```c
#include <unistd.h>
ssize_t write(int fd, const void* buf, size_t nbytes);
// 返回值：若成功返回已经写的字节数，若出错返回-1.
```

对于普通文件，写操作从文件的当前偏移处开始，如果在文件打开时指定了`O_APPEND`选项，则在每次写操作之前，将文件偏移量设置为文件结尾处，在一次成功写之后，文件偏移量增加实际写的字节数。

## Ｉ/O的效率

可以写一个程序来选择不同的缓冲区长度来测试其I/O的速度。

首先要获得一个随机文件：

```shell
dd if=/dev/zero of=test.file count=50000 bs=4096
```

程序如下：

```c
#include "apue.h"
#include <malloc.h>
int main(int argc, char* argv[]){
    int n;
    int buffsize = atoi(argv[1]);
    char* buf = (char*)malloc(buffsize*sizeof(char));
    while ((n=read(STDIN_FILENO,buf,buffsize))>0)
    {
        if(write(STDOUT_FILENO,buf,n)!=n){
            err_sys("write error");
        }
    }
    if(n<0){
        err_sys("read error");
    }
    exit(0);
}

```

然后输入下列命令：

```shell
time ./test_fileio.out 4096 <./test.file >/dev/null

```

就可以观察不同缓冲区长度对应的I/O时间了。

## 文件共享

内核使用三种数据结构表示打开文件，它们之间的关系决定了在文件共享方面一个进程对另一个进程可能产生的影响。

- 进程表(Process Table)

  每个进程表中都有一个记录项，记录项中包含一张打开的文件描述符表，可将其视为一个矢量，每个描述符占一项。与文件描述符关联的是：

  - ａ. 文件描述符标志(File descriptor flag)
  - b. 指向一个文件表项的指针(A pointer to a file table entry)

- 文件表(File Table)

  每个文件表项包括：

  - 文件状态标志(file status flags:read,write etc.))
  - 当前文件偏移量(current file offset)
  - 指向该文件v节点表项的指针(a pointer to a v-node table entry for the file)

- V节点表(V-node Table)

  - type of file
  - pointers to functions operating on file
  - i-node for the file: owner,size,device,disk location pointers.

![](UNIX之文件IO\20190531224936.png)

两个进程共享一个文件如下图所示：

![](UNIX之文件IO\20190531225011.png)

## 原子操作

场景：进程A和进程B在同一文件进行追加操作

1. Ａ的文件偏移量设置到1500(文件尾部EOF)
2. 内核切换到B
3. B的文件偏移量设置为1500(文件尾部EOF)
4. B写入文件，文件大小增加到1600
5. 内核切换到进程A
6. A写入文件，Ｂ的数据被覆盖！

解决方案：UNIX系统为这样的操作提供了一种原子操作方法，即在文件打开时设置O_APPEND标志。

## 函数`dup`和`dup2`

这两个函数都可以用来复制一个现有的文件描述符

```c
#include <unistd.h>
int dup(int fd);
int dup2(int fd, int fd2);
//返回值：成功返回新的文件描述符，出错返回-1

```

有`dup`返回的新文件描述符一定是当前可用文件描述符中的最小数值。对于`dup2`，可以用`fd2`参数指定新描述符的值。如果`fd2`已经打开，则先将其关闭。如若`fd`等于`fd2`，则`dup2`返回`fd2`，而不关闭它。

`dup(fd)`等效于`fcntl(fd, F_DUPFD, 0)` 

`dup2(fd, fd2)` 等效于`close(fd2); fcntl(fd, F_DUPFD, fd2)` 

## 函数`sync`　、`fsync` 和`fdatasync` 

通常，当内核需要重用缓冲区来存放其他磁盘块数据时，它会把所有延迟写数据块写入磁盘。为了保证磁盘上实际文件系统和缓冲区中内容的一致性，UNIX系统提供了以上三个函数。

```c
#include <unistd.h>
int fsync(int fd);
int fdatasync(int fd);
// 返回值:若成功返回0,错误则返回-1
void sync(void);

```

- `sync` 只是将所修改过的块缓冲区排入写队列，然后就返回，它不等带实际写磁盘操作结束。update守护进程周期性调用该函数，定期冲洗内核缓冲区。另外，命令`sync(1)` 也调用`sync`函数。
- `fsync` 函数只对有文件描述符`fd`指定的一个文件起作用，并且等待写磁盘操作结束才返回。
- `fdatasync`函数类似于`fsync`，但它只影响文件的数据部分。

## 函数`fcntl`

`fcntl`函数可以改变已经打开的文件的属性

```c 
#include <fcntl.h>
#include <sys/types.h>
#include <unistd.h>
int fcntl(int fd, int cmd, ... /* int arg */);
// 返回值：若成功则依赖cmd,出错返回-1

```

功能对应如下：

| cmd                           | fcntl() results         |
| ----------------------------- | ----------------------- |
| F_DUPFD                       | 复制一个已有的描述符    |
| F_GETFD or F_SETFD            | 获取/设置文件描述符标志 |
| F_GETFL or F_SETFL            | 获取/设置文件状态标志   |
| F_GETOWN or F_SETOWN          | 获取/设置异步I/O所有权  |
| F_GETLK, F_SETLK, or F_SETLKW | 获取/设置记录锁         |

## 函数`ioctl`

`ioctl`是I/O操作的杂物箱。终端I/O是使用该函数最多的地方。

```c
#include <unistd.h>
#include <sys/ioctl.h>
int ioctl(int fd, int request, ...);
//返回值：若出错，返回－１；若成功,返回其他值

```

比如磁带操作使得我们可以在磁带上写一个文件结束标志、倒带、越过指定个数的文件或记录等，用本章中的其他函数(`read、write、lseek`等)都难以表示这些操作，所以对这些设备进行操作的最容易的方法就是使用`ioctl`

