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

#### 按行读取

- `ssize_t getline(char **lineptr, size_t *n, FILE *stream);` 等价于`getdelim(lineptr, n, '\n', stream)` 

- `ssize_t getdelim(char ** restrict lineptr, size_t* restrict n, int delimiter, FILE* stream);`  : 像fgetc一样从流stream中读取字符，直到遇到分隔符delimiter, 将读取到的字符串存储到`lineptr`指向的地址，自动增加其大小，就好像对输入使用了`realloc`一样，存储的字符包含了分隔符，并且添加了一个空字符在结尾。成功返回存储在`buffer`中的字符数量，包含分隔符，但是不包括空字符。错误则返回-1并在流中设置`feof`和`ferror`

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

### 示例

#### 文件复制

```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

int main() {
    FILE* fpr = fopen("file.txt", "r+");
    if (fpr == NULL) {
        printf("Error of opening a file : file.txt\n");
        exit(1);
    }
    FILE* fpw = fopen("newfile.txt", "w+");
    if (fpw == NULL) {
        printf("Error of opening a file : newfile.txt\n");
        exit(1);
    }
    size_t len = 0;
    char buf[BUFSIZ];
    memset(buf, 0, BUFSIZ);
    while ((len = fread(buf, sizeof(char), BUFSIZ, fpr)) > 0) {
        fwrite(buf, sizeof(char), len, fpw);
    }
    printf("copy file success!");
    fclose(fpw);
    fclose(fpr);
    return 0;
}
```

#### 测试

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
void test_getchar() {
    char c = getchar();
    printf(c == '\n' || c == ' ' || c == '\t' ? "输入的字符是换行或空格" : "输入的字符是：%c", c);
}

void test_fgetc() {
    FILE* f = fopen("file.txt", "r");
    if (f == NULL) {
        printf("open file error\n");
    }
    char buf[BUFSIZ];
    memset(buf, 0, BUFSIZ);
    int i = 0;
    int val = 0;
    while ((val = fgetc(f)) != '\n') {
        buf[i++] = val;
    }
    buf[i++] = '\0';
    printf("从文件file.txt中读取到的第一行字符串为: %s", buf);
}

void test_fgets() {
    FILE* f = fopen("file.txt", "r");
    if (f == NULL) {
        printf("open file error\n");
    }
    char buf[BUFSIZ];
    memset(buf, 0, BUFSIZ);

    while (fgets(buf, BUFSIZ, f) != NULL) {
        printf("%s", buf);
    }
    if (feof(f)) {
        puts("End of File!");
    }
}

int main() {
    test_getchar();
    test_fgetc();
    test_fgets();
    return 0;
}  
```

## C++IO流

[C++ IO Streams and File Input/Output (ntu.edu.sg)](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/cp10_io.html)

C++把输入和输出看作字节流，输入时，程序从输入流中抽取字节；输出时，程序将字节插入到输出流中。对于面向文本的程序来说， 每个字节代表一个字符，更通俗地说，字节可以构成字符或数值数据的二进制表示。

IO操作的过程中，任何需要被传递的数据，在经过IO类库处理前后是不同的。这样，我们可以把数据的表示分为两种：内部表示和外部表示。

数据的内部表示便于程序进行数据处理。典型的内部表示有：整型数的二进制表示、浮点数的IEEE表示、字符的ASCII或Unicode编码表示。数据的外部表示则根据不同的外部设备的需要，有具体不同的表现形式。如果外部数据表示是可读的字符序列，则称为文本IO，否则为二进制IO。标准IO流的主要目的是支持文本IO，不直接支持二进制IO。

虽然IO流是以流的方式进行数据传递，但这并不表明传递的数据不能有任何结构，而是指IO流的概念是以流的方式进行输入输出，所传递数据的内部结构隐藏在对流数据的解释中

### IO的步骤

在IO流里，输入输出分为4步：格式化/解析，缓冲，编码转换和传递。

**格式化/解析**：在内部数据表示（以字节为单位）与外部数据表示（以字符为单位）之间进行双向转换。例如一个2字节的整数10002，就需要5个字符来表示。

**缓冲**：用于在格式/解析与传递只加缓存字符序列。对于输出，较短的字符序列格式化之后并不马上输出，而是保存在缓冲区里，待累积到一定规模之后再传递到外部设备。相反，从外部设备读入的大量数据也是先放在缓冲区，然后逐步取出完成输入。默认时，IO流的输入输出都是经过缓冲的，也可以让IO流工作在无缓冲模式下。

**编码转换**： 是将一种字符表达式转换成另一种字符表达式。如果格式化产生的字符表达式与外部字符表达式不同（输出时），或者外部表达式与IO流能解析的表达式不同（输入时），就必须进行编码转换。如多字节编码与宽字符编码之间的转换等。多数情况下并不需要进行编码转换。

**传递**：主要是与外部设备进行通信。输出时，传递负责将经过格式化、缓冲即编码转换后的字符序列发送到外部设备；输入时，则负责将外部设备抽取数据，为其后进行的编码转换、缓冲及解析提供字符序列。

### IO流类库组成结构

IO流类库在不同平台的具体实现上，可能会有所变化，但从总体设计上来看，C++流库主要由两个流类层次组成：

**（1）以streambuf类为父类的类层次**  

主要完成信息通过缓冲区的交换。派生层次如下

<img src="https://i0.hdslb.com/bfs/album/35937d9dfbbddaeec0a0f765e2077cf718b2ba03.png" title="" alt="" data-align="center">

缓冲区：是一个队列数据结构，由一字符序列和两个指针组成，这两个指针分别指向字符要被插入或被取出的位置。  

`streambuf`类为所有的`streambuf`类层次对象设置了一个固定的内存缓冲区，动态划分为两部分：  

用做输入的取区，用取指针指示当前取字符位置。  

用做输出的存区，用存指针指示当前存字符位置。

**（2）以ios类为父类的类层次**

ios类及其派生类是在`streambuf`类实现的通过缓冲区的信息交换的基础上，进一步增加了各种格式化的输入/输出控制方法。它们为用户提供使用流类的接口，它们均有一个指向`streambuf`的指针。

ios类有四个直接派生类：

1. `istream`
2. `ostream`
3. `fstreambase`
4. `strstreambase`

这四种流作为流库中的基本流类。ios类的派生层次如下：

<img src="https://i0.hdslb.com/bfs/album/0448f281752dc07fda3e89ba2ea29f27ca4daae0.png" title="" alt="" data-align="center">

### 标准输入流cin

不同版本的抽取运算符（>>）查看输入流的方法是相同的，它们**跳过**空白（空格、换行和制表符）直到遇到非空白字符。

一般情况下可以使用如下方式测试流状态：

```cpp
while(cin >> input) {
    sum += input;
}
cin >> input; // 这里流已经无效了，应该使用cin.clear() 重置流
if (cin.eof()) cout << "End of File" << endl;
```

#### 字符输入

`cin.get(ch) `读取单个字符，不会跳过空白，返回流对象cin

`cin.get()`，读取单个字符，不会跳过空白，返回读取的字符

考虑包含空包的输入时可以有如下代码：

```cpp
int ch;
while ((ch = cin.get()) != EOF) {
    // process input;
}
```

**使用哪种方式（>> , cin.get(ch), cin.get()）读取字符取决于是否希望跳过空白**

#### 字符串输入

cin提供了用于字符串输入的成员函数：

- `istream& get(char*, int, char)` 第一个参数时要放置的字符串地址，第二个是比要读取的字符数大1，第三个指定分界符

- `istream& get(char*, int)` 默认指定换行符号为分界符

- `istream& getline(char*, int, char)` 

- `istream& get(char*, int, char)` 

`get`和`getline`方法的区别是`get`不会丢弃换行符，也就是换行符依旧停留在输入流中，而`getline`会**抽取并丢弃**换行符

#### 直接输入

`istream& read(char*, int, char)` `read`方法不会在输入后加上空字符，因此不能将输入转换为字符串。其与`ostream.write()`搭配使用完成文件的输入输出

### 标准输出流cout

以下代码很好地展示了用法

```cpp
#include<iostream>
#include<iomanip>
#include<cmath>
using namespace std;

int main() {
    cout << fixed << right;
    cout << setw(6) << "N" << setw(14) << "square root"
         << setw(15) << "fourth root\n";
    double root;
    for (int n = 10; n <= 100; n += 10) {
        root = sqrt(double(n));
        cout << setw(6)  << setfill('.') << n << setfill(' ') 
             << setw(12) << setprecision(3) << root
             << setw(14) << setprecision(4) << sqrt(root)
             << endl;
    } 
    return 0;
}
```

```bash
     N   square root   fourth root
....10       3.162        1.7783
....20       4.472        2.1147
....30       5.477        2.3403
....40       6.325        2.5149
....50       7.071        2.6591
....60       7.746        2.7832
....70       8.367        2.8925
....80       8.944        2.9907
....90       9.487        3.0801
...100      10.000        3.1623
```

### 流状态

![](https://i0.hdslb.com/bfs/album/a65839112efeadc333780c62fa0cb2c5d43a1ba9.png)

### 文件输入和输出

#### 输出文件步骤

1. 创建一个`ofstream`对象来管理输出流

2. 将该对象与特定的文件关联起来

3. 以使用`cout`的方式使用该对象，唯一的区别是输出将进入文件而不是屏幕

```cpp
ofstream fout;
fout.open("file.txt");
fout << "data...";
// 或者
ofstream fout("file.txt");
fout << "data...";
```

> 警告：以默认模式打开文件进行输出，将直接将文件长度截短为0，相当于删除了文件之前的内容

#### 输入文件步骤

1. 创建一个`ifstream`对象来管理输出流

2. 将该对象与特定的文件关联起来

3. 以使用`cin`的方式使用该对象

```cpp
ifstream fin;
fin.open("file.txt");
char ch;
fin >> ch;
// 或者
ifstream fin("file.txt");
fin >> ch;
```

#### 流状态检查

<img src="https://i0.hdslb.com/bfs/album/855715d124b3b31be658855b2e01697d971a54b6.png" title="" alt="" data-align="center">检查文件是否打开：`in.is_open()` 相对于以前的方式更好，因为它能够检测出其他方式不能检测出的微妙问题，如上图所示

#### 文件模式

<img src="https://i0.hdslb.com/bfs/album/638a84a1f2149662103ce9d40528d86a4d78f781.png" title="" alt="" data-align="center">

<img src="https://i0.hdslb.com/bfs/album/b5e184ea78a043a6fa5610e4002fb6121e022269.png" title="" alt="" data-align="center">

