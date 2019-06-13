---
title: 标准IO
date: 2019-06-02 22:33:39
tags: [Unix, c语言, 操作系统]
categories: UNIX环境高级编程
---

# 标准I/O库

## 引言

标准I/O库是1975年由Dennis Ritchie编写的，已经被大多数操作系统实现，也是ANSI C标准，它可以处理很多细节，比如缓冲区分配、以优化的块长度执行I/O等。

![](标准IO\DeepinScreenshot_select-area_20190602151221.png)

## 流和FILE对象

对于标准I/O库，它们的操作是围绕流进行的，当用标准I/O库打开或创建一个文件时，我们使一个流与一个文件关联。对于不同的字符集，一个字符用几个字节表示也不统一。标准I/O文件流可以用于单字节或多(宽)字节字符集。流的定向(stream's orientation)决定了所读写的字符是单字节还是多字节的。当一个流被创建时，它并没有被定向。如若在为定向的流上使用一个多字节I/O函数，则会将该字节流定向为宽定向。只有两个函数可以改变流的定向：`freopen`清除一个流的定向，`fwide`函数可以用于设置流的定向。

```c
#include <stdio.h>
#include <wchar.h>
int fwide(FILE *stream, int mode);
// 宽定向返回正值，字节定向返回负值，为定向返回0
```

`mode`参数的不同使得`fwide`函数执行不同的工作,其值为负，则函数试图使指定的流是字节定向，其值为正，则函数试图使得指定的流是宽字节定向，为0则函数将不试图设置流的定向，但返回标识该流定向的值。

`fwide`并不改变已定向流的定向，且无出错返回

当打开一个流时，标准I/O函数`fopen`返回一个指向FILE对象的指针。该对象通常是一个结构，它包含了标准I/O库为管理该流需要的所有信息，包括用于实际I/O的文件描述符、指向用于该缓冲区的指针、缓冲区长度、当前在缓冲区中的字节数等。我们称指向FILE对象的指针为文件指针。

## 标准输入、标准输出和标准错误

对一个进程预定义了三个流，并且这三个流可以自动地被进程使用，即标准输入、标准输出和标准错误。这些流引用的文件与`STDIN_FILENO`、`STDOUT_FILENO`和`STDERR_FILENO`描述符所引用的文件相同。

这三个流通过预定义的文件指针`stdin`、`stdout`和`stderr`加以引用。这三个文件指针定义包含在`<stdio.h>`中

## 缓冲

标准I/O使用缓冲的目的是尽可能减少使用`read`和`write`调用的次数，它对每个I/O流自动地进行缓冲管理，从而避免了应用程序需要考虑这一点带来的麻烦。

标准I/O提供了三种缓冲方式：

- 全缓冲(Fully Buffered I/O)

  Actual I/O(实际I/O) take place when std I/O buffer is FILLED!

  DIsk file are fully buffered(驻留在磁盘上的文件通常使用全缓冲)

  A Buffer can be flushed(write out contents of buffer to disk, ... ,etc.)

  - fflush():write out buffer contents (std I/O)
  - tcflush():discard buffer data (terminal driver)

- 行缓冲(Line Buffered I/O)

  Actual I/O is performed only when a newline character is encountered(遇到换行符)

  Line buffering is used on a stream associated with a terminal.(涉及终端通常使用行缓冲)

  Fixed  buffer size->I/O takes place before newline when buffer is full.由于缓冲区大小固定，即使没有遇到换行符，也会进行I/O操作。

- 不带缓冲(Unbuffered I/O)

  Characters unbuffered(不对字符进行缓冲存储)

  stderr unbuffered(标准错误是不带缓冲的)

  hence, error messages are always output very quickly(因此错误返回才会很快)

可以用下列函数更改缓冲类型

```c
#include <stdio.h>
void setbuf(FILE *stream, char *buf);

int setvbuf(FILE *stream, char *buf, int mode, size_t size);
//若成功返回0，出错返回非0
```

这些函数一定要在流已经被打开后调用，而且也应在对该流执行任何一个其他操作之前调用

使用`setbuf`函数可以打开或关闭缓冲机制。为了带缓冲进行I/O，参数`buf`必须指向一个长度为`BUFSIZ`的缓冲区（该常量定义在`<stdio.h>`中）。为了关闭缓冲，将`buf`设置为NULL。

使用`setvbuf`，我们可以精确地说明所需的缓冲类型。这是用`mode`参数实现的：

| mode   | 缓冲方式 |
| ------ | -------- |
| _IOFBF | 全缓冲   |
| _IOLBF | 行缓冲   |
| _IONBF | 不带缓冲 |

如果指定一个不带缓冲的流，则忽略`buf`和`size`参数，如果指定全缓冲，则可以设定这两个参数。如果`buf`是`NULL`则，系统自动为该流分配适当长度的缓冲区。

任何时候都可以强制冲洗一个流：该函数将使该流所有未写的数据都传送至内核。

```c
#include <stdio.h>
int fflush(FILE* fp);
// 成功返回0，出错返回EOF，若fp=NULL，则所有流都被冲洗。
```

## 打开流

下列三个函数打开一个标准流

```c
#include <stdio.h>

FILE *fopen(const char *pathname, const char *mode);

FILE *fdopen(int fd, const char *mode);

FILE *freopen(const char *pathname, const char *mode, FILE *stream);
// 若成功返回文件指针，出错返回NULL
```

- `fopen`函数用于打开路径名为`pathname`的一个指定的文件。
- `fdopen`取一个已有的文件描述符，并使一个标准I/O流与该描述符结合。此函数通常用于由创建管道和网络通信通道函数返回的描述符。因为这些特殊类型的文件不能用`open`函数打开。
- `freopen`函数在一个指定的流上打开一个指定的文件，若该流已经打开，则先关闭该流。若该流已经定向，则用`freopen`清除该定向。此函数一般用于将一个指定的文件打开为一个预定义的流：标准输入、标准输出或标准错误。

`mode`指定对该I/O流的读、写方式。ISO C规定如下：

| mode             | 说明                                 | open(2)标志                 |
| ---------------- | ------------------------------------ | --------------------------- |
| r or rb          | 为读而打开                           | O_RDONLY                    |
| w or wb          | 将文件截断至0长，或为写而创建        | O_WRONLY\|O_CREAT\|O_TRUNC  |
| a or ab          | 追加：在文件尾写而打开，或为写而创建 | O_WRONLY\|O_CREAT\|O_APPEND |
| r+ or r+b or rb+ | 为读和写打开                         | O_RDWR                      |
| w+ or w+b or wb+ | 将文件截断至0长，或为读和写而打开    | O_RDWR\|O_CREAT\|O_TRUNC    |
| a+ or a+b or ab+ | 为在文件尾读和写而打开或创建         | O_RDWR\|O_CREAT\|O_APPEND   |

字符b可以区分文本文件和二进制文件，实际上UNIX并不区分，所以在UNIX系统下并无作用。

对于`fdopen` `mode`参数的意义有所区别，因为该文件描述符已经被打开，所以`fdopen`为写而打开并不截断该文件，即`fdopen`函数不能截断它为写而打开的任一文件。

打开一个标准i/O流的6中不同方式：

| 限制               | R    | w    | a    | r+   | w+   | a+   |
| ------------------ | ---- | ---- | ---- | ---- | ---- | ---- |
| 文件必须已经存在   | #    |      |      | #    |      |      |
| 放弃文件以前的内容 |      | #    |      |      | #    |      |
| 流可以读           | #    |      |      | #    | #    | #    |
| 流可以写           |      | #    | #    | #    | #    | #    |
| 流只在尾端处写     |      |      | #    |      |      | #    |

流被打开时系统默认是全缓冲，若流引用终端设备，则该流是行缓冲的，可以用`flose`函数关闭一个打开的流：

```c
#include <stdio.h>

int fclose(FILE *stream);
// 成功返回0 出错返回EOF
```

在该文件被关闭之前，冲洗缓冲区的输出数据，缓冲区中的任何输入数据都被丢弃，如果标准I/O库已经为该流分配一个缓冲区，则释放此缓冲区。

当一个进程正常终止时（exit函数，main函数返回），则所有带未写缓冲数据的标准I/O流都被冲洗，所有打开的标准I/O流都被关闭。

## 读和写流

一旦打开了流就可以在三种不同类型的非格式化I/O中进行选择，并对其进行读、写操作。

- 每次一个字符的I/O（Character-at-a-time）

  `getc(),fgetc(),getchar()`

- 每次一行的I/O（Line-at-a-time）

  `fgets(),fputs`

- 直接I/O（Direct I/O）

  read / write some number of objects（每次操作读写一定数量的某种对象）

```c
#include <stdio.h>

int fgetc(FILE *stream);

int getc(FILE *stream); // 一般被实现为宏

int getchar(void); // ==getc(stdin)
// 若成功返回下一个字符，若已到达文件尾端或出错，返回EOF
```

```c
#include <stdio.h>

int feof(FILE *stream);

int ferror(FILE *stream);
//条件为真返回非0，否则返回0
void clearerr(FILE* stream);
// 清除下面两个标志
```

在大多数实现中，为每个流在FILE对象中维护了两个标志：

- 出错标志（error flag）
- 文件结束标志（EOF flag）

从流中读取数据之后，可以调用`ungetc`将字符再压送回流中。

```c
#include <stdio.h>
int ungetc(int c, FILE *stream);
// 若成功返回c,出错返回EOF
```

压送回流中的字符以后又可以从流中读出，但读出字符的顺序与压送回的顺序相反。回送的字符不一定必须是上一次读到的字符，不能回送EOF，因为一次成功的调用会清除该流的文件结束标志。

对应与输入函数，有下列输出函数：

```c
#include <stdio.h>

int fputc(int c, FILE *stream);

int putc(int c, FILE *stream); //可被实现为宏

int putchar(int c); // ==putc(c,stdout)
// 成功返回c,出错返回EOF
```

## 每次一行I/O

输入：

对于`fgets`:必须是定缓冲区的长度n,此函数一直读到下一个换行符为止，但是不超过n-1个字符，读入的字符被送入缓冲区。该缓冲区以null字节结尾。如果该行包括最后一个换行符的字符数超过n-1，则返回一个不完整的行，但是缓冲区总是以null字符结尾，所以`fgets`函数是保留换行符的。

```c
#include <stdio.h>
// stores:line+ \n +NULL
char *fgets(char *s, int size, FILE *stream); //从指定流读
// stores: line only,without \n and NULL
char *gets(char *s); // 从标准输入读,不推荐使用,曾造成1998因特网蠕虫事件，因为不能指定缓冲区长度，就可能造成缓冲区溢出

// 成功返回s,若已经到达文件尾端或出错返回NULL

```

输出：

```c
#include <stdio.h>
// 将一个以null字节终止的字符串写到指定的流
// 通常，在null字节前是一个换行符，但不要求总是如此
int fputs(const char *s, FILE *stream); // 以null字节终止，但不写出终止符null
// 会写出一个额外的\n，不推荐使用
int puts(const char *s);
// 成功返回非负值，出错返回EOF

```

## 标准I/O的效率

- How does the three types of unformatted I/O compare in efficiency?

  一般一次一字符I/O<一次一行I/O<不带缓冲（最佳块）

- How does unformatted I/O compare with direct read() and write()?

  取决于直接写的BUFFSIZE，当BUFFSIZE为1时，用`fgetc`要快的多，因为read版本执行了很多次系统调用，而fgetc执行了很多此函数调用但系统调用要少得多。

- Are macros more efficient than functions?Why?

  因为该功能需要被多次调用，用函数实现开销更大，如果能用宏实现，要快得多。

使用`fgetc`和`fputc`从终端输入输出的程序如下：

```c
#include "apue.h"
int main(void){
    int c;
    while ((c=getc(stdin)!=EOF))
    {
        if(putc(c,stdout)==EOF){
            err_sys("output error");
        }
    }
    if(ferror(stdin)){
        err_sys("input error");
    }
    exit(0);
}

```

使用`fgets`和`fputs`从终端输入输出的程序如下：

```c
#include "apue.h"
int main(void){
    char buf[MAXLINE];
    while (fgets(buf,MAXLINE,stdin)!=NULL)
    {
        if(fputs(buf,stdout)==EOF){
            err_sys("putput error");
        }
    }
    if(ferror(stdin)){
        err_sys("input error");
    }
    exit(0);
}

```

两者测试效率结果如下：

![](标准IO\DeepinScreenshot_select-area_20190602205924.png)

## 二进制I/O

二进制I/O读或写一整个结构，在不能够使用`fgets`或`fputs`的场景，可能由于结构中有NULL或\n在一个结构里，因此就有了二进制I/O

```c
#include <stdio.h>

size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);//从流中读入

size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);// 写入流中
// 返回读或写的对象数

```

这两个函数有以下两种常见的用法：

- 读或写一个二进制数组，例如为了将一个浮点数组的第2-5个元素写至一文件上，可以编写以下程序：

  ```c
  float data[10];
  if(fwrite(&data[2], sizeof(float), 4, fp)!=4){
  	err_sys("fwrite error");
  }
  
  ```

  其中，指定`size`为每个数组元素的长度，`nmemb`为欲写的元素个数

- 读或写一个结构，例如：

  ```c
  struct{
  	short count;
      long total;
      char name[NAMESIZE];
  } item;
  if(fwrite(&item, sizeof(item), 1, fp)!=1){
      err_sys("fwrite error");
  }
  
  ```

二进制I/O在不同的系统上可能不兼容，因为：

- 在一个结构中，同一成员的偏移量可能随着编译程序和系统的不同而不同。
- 用来存储多字节整数和浮点值的二进制格式在不同的系统结构间也可能不同

## 定位流

有三种方法定位流：

文件位置指示器是从文件起始位置开始度量，以字节为度量单位

- `ftell`和`fseek`函数 

  ```c
  #include <stdio.h>
  // 若成功返回0 出错返回-1 whence值与lseek函数一样SEEK_SET、SEEK_CUR和SEEK_END
  int fseek(FILE *stream, long offset, int whence);
  // 若成功返回当前文件位置指示，若出错返回-1L
  long ftell(FILE *stream);
  // rewind可以将一个流设置到文件的起始位置
  void rewind(FILE *stream);
  
  ```

- `ftello`和`fseeko`函数

  ```c
  #include <stdio.h>
  // 若成功返回0 出错返回-1
  int fseeko(FILE *stream, off_t offset, int whence); // 处理类型off_t以外都相同于fseek
  // 若成功返回当前文件位置，若出错返回(off_t)-1
  off_t ftello(FILE *stream);
  
  ```

- `fgetpos`和`fsetpos`函数 （ISO C引入，使用一个抽象数据类型`fpos_t`记录文件位置）需要移植到非UNIX就应该用这个。

  ```c
  // 下面的函数将文件位置指示器的当前值存入由pos指向的对象中，在以后调用fsetpos可以使用此值来重新定位
  int fgetpos(FILE *stream, fpos_t *pos);
  
  int fsetpos(FILE *stream, const fpos_t *pos);
  
  ```

  

## 格式化I/O

### 格式化输出

```c
#include <stdio.h>
int printf(const char *format, ...);
int fprintf(FILE *stream, const char *format, ...);
int dprintf(int fd, const char *format, ...);
// 以上三个函数返回值：成功返回输出的字符数，若输出有错返回负值。
int sprintf(char *str, const char *format, ...);
//成功返回存入数组的字符数，编码出错返回负值
int snprintf(char *str, size_t size, const char *format, ...);
// 若缓冲区足够大，返回将要存入数组的字符数；若编码出错，返回负值

```

格式说明控制其余参数如何编写，以后又如何显示、每个参数都按照转换说明写，转换说明以%开始，形式如下：

```
%[flags][fldwidth][precision][lenmodifier]convtype

```

`flag`标志如下：

| flags | 说明                                             |
| ----- | ------------------------------------------------ |
| ‘     | （撇号）将整数按照千位分组字符                   |
| -     | 在字段内左对齐输出                               |
| +     | 总是显示带符号转换的正负号                       |
| 空格  | 如果第一个字符不是正负号，则在其前面加上一个空格 |
| #     | 指定另一种转换形式(如十六进制为0x)               |
| 0     | 添加前导0进行填充                                |

`fldwidth`说明最小字段宽度，转换后若小于宽度则用空格或*填充

`precision`说明整型转换后最少输出数字位数、浮点数转化后小数点后的最少位数、字符串转换后的最大字节数。

`lenmodifier`说明参数长度

### 格式化输入

```c
#include <stdio.h>

int scanf(const char *format, ...);
int fscanf(FILE *stream, const char *format, ...);
int sscanf(const char *str, const char *format, ...);
// 三个函数返回值：赋值的输入项数；若输入出错或在任一转换前已到达文件尾端，返回EOF

```

`scanf`族用于分析输入字符串，并将字符序列转换成指定类型的变量。

## 实现细节

每个标准I/O都有一个与其相关联的文件描述符，可以对一个流调用`fileno`函数以获得其描述符

```c
#include <stdio.h>
int fileno(FILE* fp);
//返回与该流相关联的文件描述符

```

如果你要使用`dup`或`fcntl`等函数，则需要此函数。

## 临时文件

ISO C标准I/O库提供了两个函数来创建临时文件

```c
#include <stdio.h>
char* tmpnam(char* ptr); //返回指向唯一路径名的指针
FILE* tmpfile(void); //成功返回文件指针，出错返回NULL

```

对于`tmpnam`,产生一个与现有文件名不同的一个有效路径名字符串。若`ptr`为NULL，则所产生的路径名存放在一个静态区区中

对于`tmpfile`，创建一个临时二进制文件(类型wb+)，在关闭该文件或程序结束时将自动删除这种文件。该函数经常使用的标准UNIX技术是先调用`tmpnam`产生一个唯一的路径名，然后，用该路径名创建一个文件，并立即`unlink`它。

可以用下列程序说明这两个函数：

```c
#include "apue.h"
int main(void){
    char name[L_tmpnam], line[MAXLINE];
    FILE* fp;
    printf("%s\n",tmpnam(NULL)); //first temp name
    tmpnam(name);   // second temp name
    printf("%s\n",name); 
    if((fp=tmpfile())==NULL) // create temp file
        err_sys("tmpfile error");
    fputs("one line of output\n",fp); //write to temp file
    rewind(fp); // then read it back
    if(fgets(line, sizeof(line),fp)==NULL)
        err_sys("fgets error");
    fputs(line,stdout); //print the line we wrote
    exit(0);
}

```

结果：

![](标准IO\DeepinScreenshot_select-area_20190602221241.png)

使用`tmpnam`和`tempnam`至少有一个缺点：在返回唯一路径名和用该名字创建文件之间存在一个时间窗口，在这个时间窗口中，另一个进程可以用相同名字创建文件。因此应该使用`tmpfile`和`mkstemp函数`。