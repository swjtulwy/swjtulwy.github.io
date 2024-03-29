# Makefile与库的制作


## 静态库

库文件是计算机上的一类文件，可以简单的把库文件看成一种代码仓库，它提供给使用者一些可以直接拿来用的变量、函数或类。 

库是特殊的一种程序，编写库的程序和编写一般的程序区别不大，只是库不能单独运行。  

库文件有两种，静态库和动态库（共享库），区别是：静态库在程序的链接阶段被复制到了程序中；动态库在链接阶段没有被复制到程序中，而是程序在运行时由系统动态加载到内存中供程序调用。  

库的好处：1.代码保密 2.方便部署和分发

![](https://i0.hdslb.com/bfs/album/f0c218a3bd675d2e36dbb4f7244fd6a035c1c248.png)

### 静态库的使用

通过gcc的`-L`命令指定库的路径，通过`-l`命令指定库的名字，注意这里库的名字不加lib,例如库的文件名为`libxxx.lib`那么库名就是`xxx`。

## 动态库

程序启动之后，动态库会被动态加载到内存中，通过 `ldd `（list dynamic  dependencies）命令检查动态库依赖关系  

如何定位共享库文件呢？  

当系统加载可执行代码时候，能够知道其所依赖的库的名字，但是还需要知道绝对路径。此时就需要系统的动态载入器来获取该绝对路径。对于elf格式的可执行程序，是由ld-linux.so来完成的，它先后搜索elf文件的 **DT_RPATH段** ——> **环境变量LD_LIBRARY_PATH** ——> **/etc/ld.so.cache文件列表** ——> **/lib/，/usr/lib 目录**找到库文件后将其载入内存。

<img src="https://i0.hdslb.com/bfs/album/7becce9fdd2551f092508729af91976812e7a01e.png@1e_1c.webp" title="" alt="" data-align="center">

### 动态库的使用

程序在执行的时候是如何定位共享库文件的呢？

1) 当系统加载可执行代码时候，能够知道其所依赖的库的名字，但是还需要知道绝对路径。此时就需要系统动态载入器(dynamic linker/loader)。

2) 对于elf格式的可执行程序，是由ld-linux.so*来完成的，它先后搜索elf文件的 DT_RPATH段—>环境变量LD_LIBRARY_PATH—>/etc/ld.so.cache文件列表—>/lib/,/usr/lib 目录找到库文件后将其载入内存。

如何让系统能够找到它：

1. 如果安装在/lib或者/usr/lib下，那么ld默认能够找到，无需其他操作。

2. 如果安装在其他目录，需要将其添加到/etc/ld.so.cache文件中，步骤如下：
   
   1. 编辑/etc/ld.so.conf文件，加入库文件所在目录的路径
   
   2. 运行ldconfig ，该命令会重建/etc/ld.so.cache文件

## 静态库与动态库对比

#### 静态库特点总结：

1. 静态库对函数库的链接是放在编译时期完成的。

2. 程序在运行时与函数库再无瓜葛，移植方便。

3. 浪费空间和资源，因为所有相关的目标文件与牵涉到的函数库被链接合成一个可执行文件

#### 动态库特点总结：

1. 动态库把对一些库函数的链接载入推迟到程序运行的时期。

2. 可以实现进程之间的资源共享。（因此动态库也称为共享库）

3. 将一些程序升级变得简单。

4. 甚至可以真正做到链接载入完全由程序员在程序代码中控制（**显示调用**）。

静态库对程序的更新、部署和发布页会带来麻烦。如果静态库liba.lib更新了，所以使用它的应用程序都需要重新编译、发布给用户（对于玩家来说，可能是一个很小的改动，却导致整个程序重新下载，**全量更新**）。

动态库在程序编译时并不会被连接到目标代码中，而是在程序运行是才被载入。**不同的应用程序如果调用相同的库，那么在内存里只需要有一份该共享库的实例**，规避了空间浪费问题。动态库在程序运行是才被载入，也解决了静态库对程序的更新、部署和发布页会带来麻烦。用户只需要更新动态库即可，**增量更新**。

<img src="https://i0.hdslb.com/bfs/album/cd1f80953668fafd7611f2a2f6843835d16cd1cc.png" title="" alt="" data-align="center">

<img src="https://i0.hdslb.com/bfs/album/bec8136ab4a21b6e79cd57db0b1ea80511f51dde.png" title="" alt="" data-align="center">

## Makefile

一个工程中的源文件不计其数，其按类型、功能、模块分别放在若干个目录中，Makefile 文件定义了一系列的规则来指定哪些文件需要先编译，哪些文件需要后编  
译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为 Makefile 文件就  
像一个 Shell 脚本一样，也可以执行操作系统的命令。  

Makefile 带来的好处就是“自动化编译” ，一旦写好，只需要一个 make 命令，整  
个工程完全自动编译，极大的提高了软件开发的效率。make 是一个命令工具，是一个  
解释 Makefile 文件中指令的命令工具，一般来说，大多数的 IDE 都有这个命令，  
比如 Delphi 的 make，Visual C++ 的 nmake，Linux 下 GNU 的 make。

### Makefile规则

一个 Makefile 文件中可以有一个或者多个规则  

```makefile
目标 ...: 依赖 ... 
    命令（Shell 命令）
    ...
```

目标：最终要生成的文件（伪目标除外）  

依赖：生成目标所需要的文件或是目标  

 命令：通过执行命令对依赖操作生成目标（命令前必须 Tab 缩进）  

Makefile 中的其它规则一般都是为第一条规则服务的

### 依赖检查与更新检测

命令在执行之前，需要先检查规则中的依赖是否存在 ，如果存在，执行命令  ，如果不存在，向下检查其它的规则，检查有没有一个规则是用来生成这个依赖的， 如果找到了，则执行该规则中的命令  

在执行规则中的命令时，会比较目标和依赖文件的时间 。如果依赖的时间比目标的时间晚，需要重新生成目标 ，如果依赖的时间比目标的时间早，目标不需要更新，对应规则中的命令不需要被执行

### 变量

#### 自定义变量

变量名=变量值  var=hello  

#### 预定义变量

AR : 归档维护程序的名称，默认值为 ar

CC : C 编译器的名称，默认值为 cc

CXX : C++ 编译器的名称，默认值为 g++

@ : 目标的完整名称

< : 第一个依赖文件的名称  

^ : 所有的依赖文件

#### 特殊命令

##### `$(wildcard PATTERN...)`

功能：获取指定目录下指定类型的文件列表  

参数：PATTERN 指的是某个或多个目录下的对应的某种类型的文件，如果有多个目录，一般使用空格间隔 

返回：得到的若干个文件的文件列表，文件名之间使用空格间隔  

 示例： `$(wildcard *.c ./sub/*.c) `

返回值格式: a.c b.c c.c d.c e.c f.c

##### $(patsubst <pattern>,<replacement>,<text>)

功能：查找<text>中的单词(单词以“空格”、“Tab”或“回车”“换行”分隔)是否符合模式<pattern>，如果匹配的话，则以<replacement>替换。  

<pattern>可以包括通配符`%`，表示任意长度的字串。如果<replacement>  中也包含`%`，那么，<replacement>中的这个`%`将是<pattern>中的那个%  所代表的字串。(可以用`\`来转义，以`\%`来表示真实含义的`%`字符)  

返回：函数返回被替换过后的字符串  

示例：`$(patsubst %.c, %.o, x.c bar.c)  `

返回值格式: x.o bar.o

### 简单示例

```makefile
#定义变量
src=sub.o add.o mult.o div.o main.o
target=app
$(target):$(src)
    $(CC) $(src) -o $(target)

%.o:%.c
    $(CC) -c $< -o $@
```

