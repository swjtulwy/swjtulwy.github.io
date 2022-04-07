# GDB调试基础


> GDB 是由 GNU 软件系统社区提供的调试工具，同 GCC 配套组成了一套完整的开发环境，GDB 是 Linux 和许多类 Unix 系统中的标准开发环境

GDB调试的三种方式：

1. 目标板直接使用GDB进行调试。

2. 目标板使用gdbserver，主机使用xxx-linux-gdb作为客户端。

3. 目标板使用`ulimit -c unlimited`，生成core文件；然后主机使用`xxx-linux-gdb ./test ./core`。

## GDB 调试

构造测试程序如下main.c和sum.c如下:

```c
// main.c:
#include <stdio.h>
#include <stdlib.h>
 
extern int sum(int value);
 
struct inout {
    int value;
    int result;
};

int main(int argc, char * argv[])
{
    struct inout * io = (struct inout * ) malloc(sizeof(struct inout));
    if (NULL == io) {
        printf("Malloc failed.\n");
        return -1;
    }

    if (argc != 2) {
        printf("Wrong para!\n");
        return -1;
    }

    io -> value = *argv[1] - '0';
    io -> result = sum(io -> value);
    printf("Your enter: %d, result:%d\n", io -> value, io -> result);
    return 0;
}
// sum.c:
int sum(int value) {
    int result = 0;
    int i = 0;
    for (i = 0; i < value; i++)
        result += (i + 1);
    return result;
}
```

然后`gcc main.c sum.c -o main -g`, 得到main可执行文件， 输入`gdb main`可进入调试.

下面介绍了gdb大部分功能，1.1 设置断点以及 1.3显示栈帧是常用功能；调试过程中可以需要1.6 单步执行，并且1.4 显示变量、1.5显示寄存器、1.8 监视点、1.9 改变变量的值。

如果进程已经运行中，需要1.11 attach到进程，或者1.10 生成转储文件进行分析。当然为了提高效率可以自定义1.13 初始化文件。

### 设置断点*

设置断点可以通过b或者break设置断点，断点的设置可以通过函数名、行号、文件名+函数名、文件名+行号以及偏移量、地址等进行设置。

格式为：

> break 函数名
> 
> break 行号
> 
> break 文件名:函数名
> 
> break 文件名:行号
> 
> break +偏移量
> 
> break -偏移量
> 
> break *地址

查看断点，通过`info break`查看断点列表。

```bash
(gdb) b 13
Breakpoint 1 at 0x11aa: file main.c, line 13.
(gdb) b sum.c:2
Breakpoint 2 at 0x123d: file sum.c, line 2.
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000000011aa in main at main.c:13
2       breakpoint     keep y   0x000000000000123d in sum at sum.c:2
```

删除断点通过命令包括：

> delete <断点id>：删除指定断点
> 
> delete：删除所有断点
> 
> clear
> 
> clear 函数名
> 
> clear 行号
> 
> clear 文件名：行号
> 
> clear 文件名：函数名

断点还可以条件断住

> break 断点 if 条件；比如break sum if value==9，当输入的value为9的时候才会断住。
> 
> condition 断点编号：给指定断点删除触发条件
> 
> condition 断点编号 条件：给指定断点添加触发条件

断点还可以通过`disable/enable`临时停用启用。

> disable
> 
> disable 断点编号
> 
> disable display 显示编号
> 
> disable mem 内存区域
> 
> enable
> 
> enable 断点编号
> 
> enable once 断点编号：该断点只启用一次，程序运行到该断点并暂停后，该断点即被禁用。
> 
> enable delete 断点编号
> 
> enable display 显示编号
> 
> enable mem 内存区域

#### 断点commands高级功能

大多数时候需要在断点处执行一系列动作，gdb提供了在断点处执行命令的高级功能commands。

```c
#include <stdio.h>

int total = 0;

int square(int i)
{
    int result=0;

    result = i*i;

    return result;
}

int main(int argc, char **argv)
{
    int i;

    for(i=0; i<10; i++)
    {
        total += square(i);
    }
    return 0;
}
```

比如需要对如上程序square参数i为5的时候断点，并在此时打印栈、局部变量以及total的值

编写gdb.init如下：

```c
set logging on gdb.log


b square if i == 5
commands
  bt full
  i locals
  p total
  print "Hit break when i == 5"
end
```

在gdb shell中`source gdb.init`，然后r执行命令，结果如下：

```
(gdb) source gdb.init 
Breakpoint 1 at 0x1129: file commands.c, line 6.
(gdb) r
Starting program: /home/lwy/workspace/gdbtest/commands 

Breakpoint 1, square (i=5) at commands.c:6
6       {
#0  square (i=5) at commands.c:6
        result = 25
#1  0x000055555555516f in main (argc=1, argv=0x7fffffffe048) at commands.c:20
        i = 6
result = 25
$1 = 55
$2 = "Hit break when i == 5"
```

可以看出断点在i==5的时候断住了，并且此时打印了正确的值。

### 运行

“`gdb 命令`”之后，run可以在gdb下运行命令；如果命令需要参数则跟在run之后。

```gdb
(gdb) run 9
Starting program: /home/lwy/workspace/gdbtest/main 9
Your enter: 9, result:45
[Inferior 1 (process 12362) exited normally]
```

如果需要断点在main()处，直接执行`start`就可以。

```
(gdb) start
Temporary breakpoint 1 at 0x555555555189: file main.c, line 11.
Starting program: /home/lwy/workspace/gdbtest/main 9

Temporary breakpoint 1, main (argc=21845, argv=0x0) at main.c:11
```

### 显示栈帧*

如果遇到断点而暂停执行，或者`coredump`可以显示栈帧。

通过bt可以显示栈帧，`bt full`可以显示**局部变量**。

```
(gdb) r 9
Starting program: /home/lwy/workspace/gdbtest/main 9

Breakpoint 2, sum (value=9) at sum.c:3
3           int i = 0;
(gdb) bt
#0  sum (value=9) at sum.c:3
#1  0x0000555555555204 in main (argc=2, argv=0x7fffffffe048) at main.c:24
(gdb) bt full
#0  sum (value=9) at sum.c:3
        result = 0
        i = 32767
#1  0x0000555555555204 in main (argc=2, argv=0x7fffffffe048) at main.c:24
        io = 0x5555555592a0
```

命令格式如下：

> bt
> 
> bt full：不仅显示backtrace，还显示局部变量
> 
> bt N：显示开头N个栈帧
> 
> bt full N

### 显示变量*

`print 变量名`可以显示变量内容。

`ptype 变量名` 可以打印变量类型

如果需要一行监控多个变量，可以通过`p {var1, var2, var3}`。

如果要跟踪自动显示，可以使用`display {var1, var2, var3}`

取消跟踪用 ` undisplay 编号`

查看文件代码：`list/l [文件名:][行号/函数名]`

设置显示的行数：`show list/listsize` , `set list/listsize 行数`

### 显示寄存器*

`info reg`可以显示寄存器内容。

```
(gdb) i r
rax            0x9                 9
rbx            0x555555555270      93824992236144
rcx            0x5555555592b0      93824992252592
rdx            0x9                 9
rsi            0x0                 0
rdi            0x9                 9
rbp            0x7fffffffdf20      0x7fffffffdf20
rsp            0x7fffffffdf20      0x7fffffffdf20
r8             0x5555555592a0      93824992252576
r9             0x7ffff7dd1070      140737351848048
r10            0x7ffff7fb9be0      140737353849824
r11            0x7ffff7fb9be0      140737353849824
r12            0x5555555550a0      93824992235680
r13            0x7fffffffe040      140737488347200
r14            0x0                 0
r15            0x0                 0
rip            0x55555555524b      0x55555555524b <sum+25>
eflags         0x216               [ PF AF IF ]
cs             0x33                51
ss             0x2b                43
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0
```

在寄存器名之前加$可以显示寄存器内容，

> p $寄存器：显示寄存器内容
> 
> p/x $寄存器：十六进制显示寄存器内容。

```
(gdb) p $ss
$2 = 43
(gdb) p/x $pc
$3 = 0x55555555524b
```

用x命令可以显示内容内容，`x/格式 地址`。

> x $pc：显示程序指针内容
> 
> x/i $pc：显示程序指针汇编。
> 
> x/10i $pc：显示程序指针之后10条指令。
> 
> x/128wx 0xfc207000：从0xfc20700开始以16进制打印128个word。

```
(gdb) x $pc
0x55555555524b <sum+25>:        0x00fc45c7
(gdb) x/i $pc
=> 0x55555555524b <sum+25>:     movl   $0x0,-0x4(%rbp)
(gdb) x/10i $pc
=> 0x55555555524b <sum+25>:     movl   $0x0,-0x4(%rbp)
   0x555555555252 <sum+32>:     jmp    0x555555555261 <sum+47>
   0x555555555254 <sum+34>:     mov    -0x4(%rbp),%eax
   0x555555555257 <sum+37>:     add    $0x1,%eax
   0x55555555525a <sum+40>:     add    %eax,-0x8(%rbp)
   0x55555555525d <sum+43>:     addl   $0x1,-0x4(%rbp)
   0x555555555261 <sum+47>:     mov    -0x4(%rbp),%eax
   0x555555555264 <sum+50>:     cmp    -0x14(%rbp),%eax
   0x555555555267 <sum+53>:     jl     0x555555555254 <sum+34>
   0x555555555269 <sum+55>:     mov    -0x8(%rbp),%eax
```

还可以通过`disassemble`指令来反汇编。

> disassemble
> 
> disassemble 程序计数器 ：反汇编pc所在函数的整个函数。
> 
> disassemble addr-0x40,addr+0x40：反汇编addr前后0x40大小。

### 单步执行*

单步执行有两个命令`next`和`step`，缩写为`n`和`s`，两者的区别是`next`遇到函数不会进入函数内部，`step`会执行到函数内部。

`finish `（跳出函数体）

如果需要逐条汇编指令执行，可以分别使用`nexti`和`stepi`。

### 继续执行*

 调试时，使用`continue`命令(缩写：`c`)继续执行程序。程序遇到断电后再次暂停执行；如果没有断点，就会一直执行到结束。

> continue：继续执行
> 
> continue 次数：继续执行一定次数。

### 监视点*

要想找到变量在何处被改变，可以使用`watch`命令设置监视点watchpoint。

> watch <表达式>：表达式发生变化时暂停运行
> 
> awatch <表达式>：表达式被访问、改变是暂停执行
> 
> rwatch <表达式>：表达式被访问时暂停执行

```
(gdb) watch i
Hardware watchpoint 3: i
(gdb) i b
Num     Type           Disp Enb Address            What
2       breakpoint     keep y   0x0000555555555244 in sum at sum.c:3
        stop only if value==9
        breakpoint already hit 1 time
3       hw watchpoint  keep y  
```

### 改变变量的值*

通过`set variable <变量>=<表达式>`来修改变量的值。

简写`set var 变量名=变量值` （循环中用的多）

`until `(跳出循环)

```
(gdb) b main
Breakpoint 4 at 0x555555555189: file main.c, line 11.
(gdb) r 9
Starting program: /home/lwy/workspace/gdbtest/main 9

Breakpoint 4, main (argc=21845, argv=0x0) at main.c:11
11      {
(gdb) n
12          struct inout * io = (struct inout * ) malloc(sizeof(struct inout));
(gdb) n
13          if (NULL == io) {
(gdb) n
18          if (argc != 2) {
(gdb) n
23          io -> value = *argv[1] - '0';
(gdb) n
24          io -> result = sum(io -> value);
(gdb) print io->value
$4 = 9
(gdb) set variable io->value=10
(gdb) n
25          printf("Your enter: %d, result:%d\n", io -> value, io -> result);
(gdb) n
Your enter: 10, result:55
26          return 0;
```

 `set $r0=xxx`：设置r0寄存器的值为xxx。

### 生成内核转储文件*

通过`generate-core-file`生成core.xxxx转储文件。

然后`gdb ./main ./core.xxxx`查看恢复的现场。

```
(gdb) generate-core-file
warning: target file /proc/14188/cmdline contained unexpected null characters
Saved corefile core.14188 


lwy@lwysLaptop:~/workspace/gdbtest$ gdb ./main ./core.14188 
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./main...
[New LWP 14188]
Core was generated by `/home/lwy/workspace/gdbtest/main 9'.
Program terminated with signal SIGTRAP, Trace/breakpoint trap.
#0  main (argc=2, argv=0x7fffffffe048) at main.c:26
26          return 0;
```

另一命令`gcore`可以从命令行直接生成内核转储文件。

> gcore `pidof 命令`：无需停止正在执行的程序以获得转储文件。

### attach到进程*

如果程序已经运行，或者是调试陷入死循环而无法返回控制台进程，可以使用`attach`命令。

> attach pid

通过ps aux可以查看进程的pid，然后使用bt查看栈帧。

以top为例操作步骤为：

> 1. ps -aux查看进程pid，为16974.
> 
> 2. sudo gdb attach 16974，使用gdb 附着到top命令。
> 
> 3. 使用bt full查看，当前栈帧。此时使用print等查看信息。
> 
> 4. 还可以通过info proc查看进程信息。

### 反复执行

continue、step、stepi、next、nexti都可以指定重复执行的次数。

`ignore 断点编号 次数`：可以忽略指定次数断点。

### dump内存到指定文件

在gdb调试中可能需要将一段内存导出到文件中，可以借助`dump`命令。

命令格式：

> dump binary memory FILE START STOP

比如`dump binary memory ./dump.bin 0x0 0x008000000`，将内存区间从0x0到0x00800000导出到dump.bin中。

