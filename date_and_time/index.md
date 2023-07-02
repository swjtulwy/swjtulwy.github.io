# C++中的日期与时间


<!--more-->

## 时间的几种表示方法

### UTC时间

UTC又称作协调世界时(Coordinated Universal Time)，是最主要的时间标准，其以原子时秒长为基础，在时刻上尽量接近于格林威治天文台的标准时间

协调世界时是世界上调节时钟和时间的主要时间标准，它与0度经线的平太阳时差不超过1秒。因此UTC时间加8就是北京标准时间(UTC+8)

### 本地时间

本地时间包含了与当地时区相关的信息

### 纪元时间

纪元时间(Epoch time)又被称为Unix时间，它表示1970年1月1日00:00UTC以来所经理的秒数(不考虑闰秒)。

这个是一个遗留问题，如果使用32位有符号数来存储的话，在2038年该值就会溢出。

### 时钟时间

从进程开始到结束，时钟所走过的时间，包括进程阻塞和等待的时间。

### CPU时间

进程获得CPU资源执行的时间包括用户态CPU时间和内核态CPU时间。

## C风格API

在C语言中，时间相关的结构体和API都在time.h头文件中，C++为了兼容C语言，这个头文件以及相关的接口都保留了，只不过头文件变成了ctime。

首先介绍下与时间相关的类型：

| 类型名           | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| clock_t          | CPU时间，一般用于统计进程执行时间，开启多线程的时候会比实际时间要多 |
| size_t           | sizeof运算符返回的无符号整数类型                             |
| time_t           | 从纪元起的时间类型                                           |
| tm               | 日历时间类型，一个结构体，包含年月日等信息                   |
| timespec (C++17) | 以秒和纳秒表示的时间（C++17）                                |



| 函数名                                                       | 说明                                    |
| ------------------------------------------------------------ | --------------------------------------- |
| std::clock_t clock()                                         | 返回自程序启动时起的原始处理器时钟时间  |
| std::time_t time(std::time_t* arg)                           | 返回自纪元起计的系统当前时间            |
| double difftime(std::time_t time_end, std::time_t time_beg)  | 计算时间之间的差                        |
| int timespec_get(std::timespec* ts, int base) (C++17)        | 返回基于给定时间基底的日历时间（C++17） |
| char* ctime(const std::time_t* time)                         | 转换 time_t 对象为文本表示              |
| char* asctime(const std::tm* time_ptr)                       | 转换 tm 对象为文本表示                  |
| std::size_t strftime(char* str, std::size_t count, const char* format, const std::tm* time) | 转换 tm 对象到自定义的文本表示          |
| std::size_t wcsftime( wchar_t* str, std::size_t count, const wchar_t* format, const std::tm* time) | 转换 tm 对象为定制的宽字符串文本表示    |
| std::tm* gmtime(const std::time_t* time)                     | 将time_t转换成UTC表示的时间             |
| std::tm* localtime(const std::time_t *time)                  | 将time_t转换成本地时间                  |
| std::time_t mktime(std::tm* time)                            | 将tm格式的时间转换成time_t表示的时间    |

上述的类型与函数之间的基本关系如下图所示

![](https://article.biliimg.com/bfs/article/831e73e96b2f022aec51a97cc313a6fcdfb6e117.png@1e_1c.webp)

### C风格计算进程时间

```cpp
#include<iostream>
#include<ctime>
#include<cstdlib>
#include<threads.h>
#include<iomanip>
#include<sys/time.h>

using namespace std;


int f(void* thr_data) {
    volatile double d = 0;
    for (int i = 0; i < 10000; ++i) {
        for (int j = 0; j < 10000; ++j) {
            d += d * i * j;
        }
    }
    return 0;
}

int main() {
    clock_t pre = std::clock();
    std::time_t start = std::time(nullptr); // very low precision
    struct timeval tv_begin;
    gettimeofday(&tv_begin, nullptr);
    timespec ts_begin;
    std::timespec_get(&ts_begin, TIME_UTC);
    thrd_t thr1, thr2;
    thrd_create(&thr1, f, nullptr);
    thrd_create(&thr2, f, nullptr);
    thrd_join(thr1, nullptr);
    thrd_join(thr1, nullptr);
    timespec ts_end;
    std::timespec_get(&ts_end, TIME_UTC);
    struct timeval tv_end;
    gettimeofday(&tv_end, nullptr);
    std::time_t end = std::time(nullptr);
    clock_t post = std::clock();
    double duration = 1000.0 * (post - pre) / CLOCKS_PER_SEC;
    std::cout << "CPU time used " << duration << " ms" << std::endl;
    std::cout << "Wall time passed " << 1000.0 * std::difftime(end, start) << " ms" << std::endl;
    std::cout << "Wall time passed(POSIX API) " << (tv_end.tv_sec * 1000000 + tv_end.tv_usec) - 
                                        (tv_begin.tv_sec * 1000000 + tv_begin.tv_usec) << " us" << std::endl;
    std::cout << "Wall time passed(high precision) " << (ts_end.tv_sec * 1000000000 + ts_end.tv_nsec) 
                                        - (ts_begin.tv_sec * 1000000000 + ts_begin.tv_nsec) << " ns" << std::endl;
    
 
    return 0;
}
```

结果如下：

```
CPU time used 861.249 ms
Wall time passed 1000 ms
Wall time passed(POSIX API) 431405 us
Wall time passed(high precision) 431404002 ns
```

可以看到用time函数获取墙上时间是很不精确的，用timespec结构体可以获得纳秒级的精度，而timeval 结构体结合gettimeofday可以获得微秒级别精度，但是后者只适用于POSIX 接口。

### UTC时间与本地时间

C风格的日期时间库中，gmtime将std::time_t类型的纪元时间转换为UTC标准时间，使用localtime则将纪元时间转换为本地时区的日历时间。日历时间用tm结构表示，结构体如下：

```cpp
struct tm {
  int tm_sec;
  int tm_min
  int tm_hour;
  int tm_mday;
  int tm_mon;
  int tm_year;
  int tm_wday;
  int tm_yday;
  int tm_isdst;
};
```

```cpp
#include<iostream>
#include<ctime>
#include<iomanip>
#include<locale>

int main() {
    std::time_t cur_time = time(nullptr);
    std::tm* cur_gmtime = gmtime(&cur_time);
    std::tm* cur_localtime = localtime(&cur_time);
    std::cout << "put_time打印UTC时间: " << std::put_time(cur_gmtime, "%Y-%m-%d %H:%M:%S") << std::endl;
    std::cout << "asctime打印UTC时间: " << std::asctime(cur_gmtime) << std::endl;
    char timestr[100];
    std::strftime(timestr, sizeof(timestr), "%Y-%m-%d %H:%M:%S", cur_gmtime);
    std::cout << "strftime打印UTC时间: " << timestr << std::endl;
    std::cout << "put_time打印local时间: " << std::put_time(cur_localtime, "%Y-%m-%d %H:%M:%S") << std::endl;
    std::cout << "asctime打印local时间: " << std::asctime(cur_localtime) << std::endl;
    return 0;
}
```

打印结果：

```
put_time打印UTC时间: 2023-07-02 16:05:24
asctime打印UTC时间: Sun Jul  2 16:05:24 2023

strftime打印UTC时间: 2023-07-02 16:05:24
put_time打印local时间: 2023-07-02 16:05:24
asctime打印local时间: Sun Jul  2 16:05:24 2023
```

注意的是，如果我们手动打印tm结构体各个字段的话，tm_year是从1900年开始经历的年份，所以tm_year要加上1900，tm_mon的范围是[0, 11]，所以要加上1。

## C++风格API

C++11 引入了choro库，该库能够以各种精度处理时间。其定义了三种主要的时间概念：

- 时钟
- 时长
- 时间点

### 时钟

C++11的chrono库主要包含了三种类型的时钟：

| 名称                  | 说明                           |
| --------------------- | ------------------------------ |
| system_clock          | 来自系统范畴实时时钟的挂钟时间 |
| steady_clock          | 决不会调整的单调时钟           |
| high_resolution_clock | 拥有可用的最短嘀嗒周期的时钟   |

system_clock来源是系统时钟。然而在大多数系统上，系统时间是可以在任何时候被调节的。所以如果用来计算两个时间点的时间差，这并不是一个好的选择。

steady_clock是一个单调时钟。此时钟的时间点无法减少，因为物理时间向前移动。因而steady_clock是度量间隔的最适宜的选择。

high_resolution_clock表示实现提供的拥有最小计次周期的时钟。它可以是system_clock或steady_clock的别名，或者第三个独立时钟（尽量避免使用）。

对于这三个时钟类，有着以下共同的成员：

| 名称       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| now()      | 静态成员函数，返回当前时间，类型为clock::time_point          |
| time_point | 成员类型，当前时钟的时间点类型。                             |
| duration   | 成员类型，时钟的时长类型。                                   |
| rep        | 成员类型，时钟的tick类型，等同于clock::duration::rep         |
| period     | 成员类型，时钟的单位，等同于clock::duration::period          |
| is_steady  | 静态成员类型：是否是稳定时钟，对于steady_clock来说该值一定是true |

每个时钟类都有着一个静态成员函数`now()`来获取当前时间。该函数的返回类型则是由该时钟类的time_point描述，例如`std::chrono::time_point<std::chrono::system_clock>`或者`std::chrono::time_point<std::chrono::steady_clock>`。我们可以使用auto关键字来简写。

阅读文档，我们不难发现system_clock有着与另外两个clock所不具有的特性：它是唯一有能力映射其时间点到C-Style时间的C++时钟。system_clock提供了两个静态成员函数来与std::time_t进行互相转换：

| 名称        | 说明                              |
| ----------- | --------------------------------- |
| to_time_t   | 转换系统时钟时间点为 std::time_t  |
| from_time_t | 转换 std::time_t 到系统时钟时间点 |

![](https://article.biliimg.com/bfs/article/4ad7d17274c96f8dac90a5d7b15e6baa89475356.png@1e_1c.webp)

### 时长

人类对精度的要求永无止境。C-Style日期时间库为了提供对纳秒的精度，增加了timespec类型及相关的函数。那如果以后对更高精度的需求越来越高，C++标准库还要不断增加更多的类型和配套函数吗？这明显是一个不合理的设计。因而C++标准提出了一个新的解决思路，而这个思路涉及到了C++11引入的一个新的头文件和类型：ratio。

#### ratio

`std::ratio`定义在`<ratio>`文件中，提供了编译期的比例计算功能。

`std::ratio`的定义如下：

```cpp
template<std::intmax_t Num, std::intmax_t Denom = 1> class ratio;
```

类成员Num即为分子，类成员Denom即为分母。我们可以直接通过调用类成员来获取相关值。

`<ratio>`头文件还包含了：ratio_add，ratio_subtract，ratio_multiply，ratio_divide来完成分数的加减乘除四则运算。

例如，想要计算 5 / 7 + 59 / 1023  ，我们可以这样写：

```cpp
ratio_add<ratio<5, 7>, ratio<59, 1023>> result;
double value = ((double) result.num) / result.den;
cout << result.num << "/" << result.den << " = " << value << endl;
```

对于编译期有理数算数的相关内容，可以在这篇文档中找到更多信息：

[编译时有理数算术 - cppreference.comzh.cppreference.com/w/cpp/numeric/ratio](https://link.zhihu.com/?target=https%3A//zh.cppreference.com/w/cpp/numeric/ratio)

有了ratio之后，结合`std::chrono::duration`，我们便可以表示任意精度的值了。

例如，相对于秒来说，毫秒是 1 / 1,000 ，微秒是 1 / 1,000,000 ，纳秒是 1 / 1,000,000,000 。通过ratio就可以这样表达：

```cpp
std::ratio<1, 1000>       milliseconds;
std::ratio<1, 1000000>    microseconds;
std::ratio<1, 1000000000> nanoseconds;
```

#### 时长类型

类模板 std::chrono::duration 表示时间间隔。有了ratio之后，表达时长就很方便了，下面是chrono库中提供的很常用的几个时长单位：

| 类型                      | 定义                                                   |
| ------------------------- | ------------------------------------------------------ |
| std::chrono::nanoseconds  | duration<至少 64 位的有符号整数类型, std::nano>        |
| std::chrono::microseconds | duration<至少 55 位的有符号整数类型, std::micro>       |
| std::chrono::milliseconds | duration<至少 45 位的有符号整数类型, std::milli>       |
| std::chrono::seconds      | duration<至少 35 位的有符号整数类型>                   |
| std::chrono::minutes      | duration<至少 29 位的有符号整数类型, std::ratio<60>>   |
| std::chrono::hours        | duration<至少 23 位的有符号整数类型, std::ratio<3600>> |

我们可以调用duration类的`count()`成员函数来获取具体数值。

#### 时长运算

时长运算可以直接使用“+”或“-”相加相减。chrono库也提供了几个常用的函数：

| 函数          | 说明                                         |
| ------------- | -------------------------------------------- |
| duration_cast | 进行时长的转换                               |
| floor (C++17) | 以向下取整的方式，将一个时长转换为另一个时长 |
| ceil (C++17)  | 以向上取整的方式，将一个时长转换为另一个时长 |
| round (C++17) | 转换时长到另一个时长，就近取整，偶数优先     |
| abs (C++17)   | 获取时长的绝对值                             |

例如：想要知道2个小时零5分钟一共是多少秒，可以这样写：

```cpp
chrono::hours two_hours(2);
chrono::minutes five_minutes(5);

auto duration = two_hours + five_minutes;
auto seconds = chrono::duration_cast<chrono::seconds>(duration);
cout << "02:05 is " << seconds.count() << " seconds" << endl;
```

我们可以得到：

```text
02:05 is 7500 seconds
```

从C++14开始，你甚至可以用字面值来描述常见的时长。这包括：

- `h`表示小时
- `min`表示分钟
- `s`表示秒
- `ms`表示毫秒
- `us`表示微妙
- `ns`表示纳秒

这些字面值位于`std::chrono_literals`命名空间下。于是，可以这样表达2个小时以及5分钟：

```cpp
using namespace std::chrono_literals;
auto two_hours = 2h;
auto five_minutes = 5min;
```

### 时间点

时钟的now函数返回的值就是一个时间点，时间点包含了时钟和时长两个信息。

类模板`std::chrono::time_point`表示时间中的一个点，定义如下：

```cpp
template<class Clock,class Duration = typename Clock::duration> class time_point;
```

与我们的常识一致，时间点具有加法和减法操作：

```text
时间点 + 时长 = 时间点
时间点 - 时间点 = 时长
```

因而我们可以通过两个时间点相减来计算一个时间间隔，示例如下：

```cpp
auto start = chrono::steady_clock::now();
double sum = 0;
for(int i = 0; i < 100000000; i++) {
    sum += sqrt(i);
}
auto end = chrono::steady_clock::now();

auto time_diff = end - start;
auto duration = chrono::duration_cast<chrono::milliseconds>(time_diff);
cout << "Operation cost : " << duration.count() << "ms" << endl;
```

两个时间点也存在着比较操作，用于判断一个时间点在另外一个时间点之前还是之后，`std::chrono::time_point`重载了`==`，`!=`，`<`，`<=`，`>`，`>=`操作符来实现比较操作。

