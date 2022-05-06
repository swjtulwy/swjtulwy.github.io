# C和C++中的输入输出


<!--more-->

## C语言IO

C语言定义了一套跨平台的带缓冲的输入输出函数，相对于Unix系统I/O，我们称之为标准I/O。标准**I/O** 库在用户空间维护了自己的 stdio 缓冲区，所以标准**I/O** 是带有缓存的，而**文件 I/O** 在用户空间是不带有缓存的，所以在性能、效率上，标准**I/O** 要优于 **文件I/O**。

理解上述概念后，就知道了由于标准I/O带有缓存区，所以标准I/O相关函数都不可避免的要显示提供一个缓存区buffer，这个`buffer` 一般是一个指针，指向一块连续的内存，其最常见的形式是字节数组，因为字节是表示存储的最小单元，但也可以是其类型的数组。另外，为了知道缓冲区内部数据的多少，我们往往还需要一个参数表示缓冲区中要写入或读取的数据的数量。由于我们只是传入了缓冲区的指针，对于缓冲区内部单个数据的大小也要指定，通过单个数据大小和数据数量我们就间接指定了缓冲区内进行操作的数据大小了。总结一下，主要的参数有如下：

- `void *buffer`: 指向缓冲区的指针，可以是任意类型

- `size_t size` ：缓冲区内每个对象的大小（字节为单位）

- `size_t count`: 需要进行操作的对象数量

- `FILE *stream`: 需要进行操作的文件流

C语言中，头文件 `<stdio.h>` 提供了通用的文件操作函数以支持窄字符的的输入输出 ，而 `<wchar.h>` 头文件则提供了支持宽字符的输入输出。

I/O流操作有两个重要的数据结构，一个是 `FILE` 类型的结构体表示，我们能通过`FILE*` 类型的指针访问和操作其关联的流。每个流都与一个外部物理设备（文件、标准输入流、打印机、串行端口等）相对应。另一个就是`fpos_t`，表示文件中位置和状态的结构类型，能够在文件中唯一指定一个位置和多字节解析器状态。

C标准库中有三个预定义标准流：

- `stdin`:标准输入流

- `stdout`:标准输出流

- `stderr`标准错误输出流

### 文件访问

主要有如下几个函数：

- `FILE *fopen( const char *filename, const char *mode );` : 用于打开文件，返回指向`FILE*`结构的对应的文件指针

- `int fclose(FILE *stream );`：用于关闭打开的文件，成功返回0否则返回EOF

- `int fflush(FILE *stream );` : 对于输出流（以及输出最后一个操作的更新流），将任何未写入的数据从流的缓冲区写入关联的输出设备。对于输入流（以及输入最后一个操作的更新流），行为是未定义的。如果 stream 是空指针，则所有打开的输出流都将被刷新，包括在库包中操作或程序无法直接访问的输出流。
  
  - `setbuf`：手动设置缓冲区大小

### 直接输入输出

- `size_t fread(void *buffer, size_t size, size_t count, FILE *stream );` 用于从指定的流中读取`count`个`size`大小的对象到`buffer`中，返回读取成功的字节数

- `size_t fwrite(const void *buffer, size_t size, size_t count, FILE *stream );` 用于将`buffer`中`count`个`size`大小的对象写入到指定的流中，返回写入成功的字节数

### 未格式化输入输出

#### 输入输出单个字符

- `int fgetc( FILE *stream );`/`int getc( FILE *stream );` : 成功返回字符对应的int值，否则返回EOF

- `int getchar(void);` 等价于`getc(stdin)`

- `int fputc(int ch, FILE *stream)`/`int putc(int ch, FILE *stream)`:成功返回字符对应的int值，否则返回EOF并在流中设置error

- `int putchar(int ch);` 等价于`putc(ch, stdout)`;

#### 输入输出字符串

- `char *fgets(char *str, int count, FILE* stream);` 从给定的文件流中读取最多count - 1个字符并将其存储到str指向的字符数组中。当解析到换行符时就会停止，这时str会包含该换行符，或者当检测到EOF标记时也会停止。如果读取中没有发生错误，那么会在结尾写入一个空字符。 

- `int fputs(const char *str, FILE *stream);` 将`str`中每个非空字符写入输出流`stream`,就像一致重复地在执行`fgetc`. 结尾的空字符不会别写入

- `char *gets_s( char *str, rsize_t n ); / char *gets( char *str );`从`stdin`中读取最多`count - 1`个字符并将其存储到str指向的字符数组中 . 当解析到换行符或者EOF标记时停止，如果读取中没有发生错误，那么会在结尾写入一个空字符。 如果碰到换行符，则会直接丢弃，且不会被计算到写入的字符数量中

- `int puts( const char *str );` 将`str`中每个非空字符写入`stdout`,然后追加一个换行符，就像一致重复地在执行`fgetc`. 结尾的空字符不会别写入

### 格式化输入输出

- `int scanf( const char*format, ... );`从`stdin`中读取输入

- `int fscanf(FILE *stream, const char *format, ... );` 从`stream`中读取输入

- `int sscanf( const char *buffer, const char *format, ... );`: 从`buffer`中按照格式输入

- `int printf( const char *format, ... );` 输出到`stdout`

- `int fprintf(FILE *stream, const char *format, ... );` 输出到`stream`

- `int sprintf( char *buffer, const char *format, ... );` 输出到`buffer`

- `int snprintf( char *restrict buffer, size_t bufsz, const char *restrict format, ... );`  按照格式`format`写入`buffer` 指定长度`bufsz`防止缓冲区溢出，更推荐这个

### 文件定位

- `long ftell(FILE *stream );` 返回流中的位置指示器， `stream` 如果流以二进制模式打开，那么这个函数返回文件中的字节数，从头开始计数

- `int fseek(FILE *stream, long offset, int origin );` 设置文件的位置指示器，`origin`表示起始位置，`offert`表示偏移量。 `origin`有`SEEK_SET`、`SEEK_CUR`和`SEEK_END`

### 错误处理

- `void clearerr(FILE *stream );` 重置错误标志和EOF指示器

- `int feof(FILE *stream );` 检查是否到达文件末尾，非零就是到了，0就是没

- `int ferror( FILE *stream );` 检查流是否出现错误

- `void perror( const char *s );` 打印错误码`errno`对应的字符描述到`stdout`

### 若干宏常量

`EOF, FOPEN_MAX,FILENAME_MAX,BUFSIZ`

## C++IO流

