# 条件编译与保护符


## IFNDEF

ifndef的含义是If not define, 其搭配使用如下

```c
// xxx.h
#ifndef __INCxxx.h
#define __INCxxx.h

#ifndef __cpluscplus
extern "C" {
#endif
    /*...*/
#ifndef
}
#endif

#endif
```

### 1.多次包含的情况

include xxx 就是将xxx的内容原地展开  

假设有：  a.h， 内容是

```c
A  
```

b.h， 内容是： 

```c
#include "a.h"  
B  
```

c.h， 内容是： 

```c
#include "a.h"  
C  
```

如果有一个文件x.c， 内容是：  

```c
#include "b.h"  
#include "c.h"  
X  
```

b.h和c.h的内容就会被插入到X之前， 也就是这个样子：  

```c
A  
B  
A  
C  
X  
```

A的内容就出现了2次。  

在更复杂的环境中， A的内容还可能出现多次。  

### 2.多次出现是有问题的

一般来说， 重复声明没什么问题。  
所以， 如果A.h中止包含一些声明， 那重复了也没什么关系。  

比如：

```c
int f(int);  
int f(int);  
int f(int);  
extern int i;  
extern int i;  
extern int i;  
struct x;  
struct x;  
struct x;
```

重复写N次也没关系。  

但头文件中会出现一类"定义"， 在**同一翻译单元**中是不能重复的。  

比如：

```c
struct x { ... };  
struct x { ... }; // 重复定义  
#define M ...  
#define M ...  // 重复定义
```

### 3.头文件保护符

有时候必须将这些定义放在头文件中， 所以就要用头文件保护符。  另外还有一类"定义"， 会产生<mark>外部符号</mark>。  这类"定义"在一个链接过程中只能有唯一一份。  是不可以加入到头文件中的。  这种定义依然有例外……  就是inline、模板和匿名名字空间， 就不扯远了……  

假设A的内容是：

```c
#ifndef A_H  
#define A_H  
AA  
#endif
```

如果A被展开多次，例如上面的X， 就会变成这个样子

```
// A_H是a.h的保护符， 必须是一个不冲突的名字。 那么，这里就不会有A_H的定义  
// 然后紧接这下一行中的条件编译就会选中#ifndef 和#endif之间的部分， 
// 也就是#define A_H 和AA  

#ifndef A_H  
#define A_H  
AA  
#endif  

B  
// 在a.h被第一次包含后， A_H就获得定义  
// 所以下一行的条件编译部分就被取消， AA就不会重复出现多次  
#ifndef A_H  
#define A_H  
AA  
#endif  
C  

X
```

最终交给编译器看到的代码就是：

```c
AA  
B  
C  
X
```

只要A_H是唯一的， AA就不会重复出现。  

就解决了这个问题， 一般情况就是这么用的， 是为**惯例**。  

### 4.外部头文件保护符

上面的用法是"内部头文件保护符"。 a.h的保护符是使用在a.h里。  

另外一种用法是"外部头文件保护符"， 如：  

```c
------ a.h ------  
AA  

------ b.h ------  
#ifndef A_H  
#define A_H  
#include "a.h"  
#endif  
B  

------ c.h ------  
#ifndef A_H  
#define A_H  
#include "a.h"  
#endif  
C  
```

当X同时包含b.h和c.h时， 最终效果和内部头文件保护符差不多。  

两者对比， 外部的优势是可以减少打开a.h的次数。  

而内部保护符可以降低a.h和b.h, c.h之间的耦合。  

### 5.定义保护符

马上就要到主题了……  

将头文件保护符的用法扩展一下， 就变成了定义保护符（这个名字是我捏造的）。  

保护的不是某个"头文件" 而是某个"定义"， 如：  

```c
------ a.h ------  

#ifndef A_X  
#define A_X  
struct x { ... };  
#endif  

#ifndef A_M  
#define A_M  
#define M ...  
#endif  

...  
```

b.h和c.h直接包含a.h， 最终效果也是一样。  

### 6.重复的定义保护符

到主题了……  同样是一个捏造的词。  

假设：  b.h包含a.h是为了获得struct x的定义。  

而c.h包含a.h是为了获得宏M的定义。  

除了上面作法， 还有另一种做法：  

a.h和上面差不多

```c
#ifndef X  
#define X  
struct x { ... };  
#endif  

#ifndef M  
#define M ...  
#endif
```

而b.h和c.h并不包含a.h， 而是直接将需要的定义写在b.h和c.h中  

```c
------ b.h ------
#ifndef X  
#define X  
struct x { ... }; 
#endif  

B

------ c.h ------
#ifndef M  
#define M ...  
#endif  

C
```

这样做其实耦合比外部头文件保护符还要高， 所以一般是不会采用的。  

但C的标准头文件必须这样做。  因为C89有一个要求， 具体我不记得了。  要么是要求标准头文件不能包含其他标准头文件。  要么是要求标准头文件不能包含任何其他文件。  （C++和C99取消了这个要求）  

stdio.h是C89的标准头文件。  例如， 它需要定义一个size_t， 作为一些函数的参数类型。  而另外有一些标准头文件也会有size_t。  所以这些头文件中的size_t都是这样提供的：

```c
#ifndef _SIZE_T_DEFINED  
#define _SIZE_T_DEFINED   
typedef unsigned xxx size_t;  
#endif
```

或者也可能将若干定义分组， 共用一个保护符。  

## Pragma once

维基百科定义如下

In the the C and C++ programming languages, `# pragma once` is a non-standard but widely supported preprocessor directive designed to cause the current source file to be included only once in a single compilation. Thus, `#pragma once` serves the same purpose as include guards, but with several advantages, including: less code, avoidance of name clashes, and sometimes improvement in compilation speed. On the other hand, `#pragma once` is not necessarily available in all compilers and its implementation is tricky and might not always be reliable.

意思就是可以用来代替`#IFNDEF`的的头文件保护符的预编译指令，非标准，看编译器支持情况，目前大多都支持了。

`pragma`还有更多有用的指令，可以参考：

[#pragma编译指令大全（上） - 简书 (jianshu.com)](https://www.jianshu.com/p/b019a3ca1c71)

