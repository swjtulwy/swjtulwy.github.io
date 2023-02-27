# Linux下常用命令


<!--more-->

Linux命令有很多，功能十分强大，掌握Linux的基础命令是一个程序员的基本要求。本文概括Linux下的常用命令，包括文件处理、用户权限、系统资源、网络等。

比较常见的如`man`、`ls`、`cd`等本文就不做介绍了，主要介绍一些功能强大的，参数又多的命令，后续将会陆续补充。

## 系统资源查看

### top

Top命令用于**实时**查看Linux下的各个进程占用的系统资源，包括CPU占用率、内存占用率和进程号等

![](https://article.biliimg.com/bfs/article/b4a3f5fe1e84e4512ad198971c54fe8d604b4814.png)

各个参数含义比较容易理解，就是英文缩写，不会的再查手册就好了

下面是主要的交互指令

- **Ctrl+L 擦除并且重写屏幕。** h或者? 显示帮助画面，给出一些简短的命令总结说明。 

- **k       终止一个进程**。 系统将提示用户输入需要终止的进程PID，以及需要发送给该进程什么样的信号。一般的终止进程可以使用15信号；如果不能正常结束那就使用信号9强制结束该进程。默认值是信号15。在安全模式中此命令被屏蔽。 i 忽略闲置和僵死进程。这是一个开关式命令。 

- **q 退出程序**。 

- **r 重新安排一个进程的优先级别**。系统提示用户输入需要改变的进程PID以及需要设置的进程优先级值。输入一个正值将使优先级降低，反之则可以使该进程拥有更高的优先权。默认值是10。 

- **S 切换到累计模式**。 

- **s 改变两次刷新之间的延迟时间**。系统将提示用户输入新的时间，单位为s。如果有小数，就换算成m s。输入0值则系统将不断刷新，默认值是5 s。需要注意的是如果设置太小的时间，很可能会引起不断刷新，从而根本来不及看清显示的情况，而且系统负载也会大大增加。 f或者F 从当前显示中添加或者删除项目。 

- **o或者O 改变显示项目的顺序**。 

- **l 切换显示平均负载和启动时间信息**。 

- **m 切换显示内存信息**。 

- **t 切换显示进程和CPU状态信息**。 

- **c 切换显示命令名称和完整命令行**。 

- **M 根据驻留内存大小进行排序**。

- **P 根据CPU使用百分比大小进行排序**。 

- **T 根据时间/累计时间进行排序**。 

- **W 将当前设置写入~/.toprc文件中**。这是写top配置文件的推荐方法。

### df

Linux df（英文全拼：disk free） 命令用于显示目前在 Linux 系统上的文件系统磁盘使用情况统计。

常用组合：`df -h `

### ps

常用组合：**ps aux** 或者 **ps -aux** 或 **ps -ef**

### pstree

查看进程树

### pgrep

`pgrep`命令用来查找进程的信息，通常会和kill命令来连用，在指定条件下`kill`问题进程。

pgrep 参数是进程的名字中的子串

### ipcs

用于报告 Linux 中进程间通信设施的状态，显示的信息包括消息列表、共享内存和信号量的信息。

### strace

用于诊断、调试 Linux 用户空间跟踪器。我们用它来监控用户空间进程和内核的交互，比如系统调用、信号传递、进程状态变更等。

### vmstat

虚拟内存统计。

### mpstat

显示各个可用 CPU 的状态统计。

### iostat

统计系统 IO。

## 网络命令

### netstat

Linux netstat 命令用于显示网络状态。

利用 netstat 指令可让你得知整个 Linux 系统的网络情况。

```bash
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
```

`netstat -a` : 列出所有连接 上述命令列出 tcp, udp 和 unix 协议下所有套接字的所有连接。然而这些信息还不够详细，管理员往往需要查看某个协议或端口的具体连接情况。

`netstat -at` : 使用 **-t** 选项列出 TCP 协议的连接

`netstat -au` : 使用 **-u** 选项列出 UDP 协议的连接

`netstat -an`:  使用 **-n** 选项拒绝显示别名，能显示数字的全部转化为数字``

`netstat -al` : 使用 **-l** 仅列出在Listen(监听)的服务状态

`netstat -ap`:  使用 **-p** 显示建立相关链接的程序名

因此  `netstat -tunlp | grep 端口号`  可以**查看端口情况**

### ifconfig

### route

### nslookup

`nslookup`用于查询DNS的记录，查询域名解析是否正常，在网络故障时用来诊断网络问题。`nslookup`是一个程序的名字，这个程序让因特网服务器管理员或任何的计算机用户输入一个主机名（举例来说，“www.baidu.com”）并发现相应的IP地址。它也会相反的名字查找为一个你指定的 IP 住址找出主机名。

### host

host也可以用来查看DNS解析结果

### lsof

list open files 列举打开的文件列表

`lsof -i : 端口号` 可以查看端口占用情况

## 文件处理

### tar

Linux tar（英文全拼：tape archive ）命令用于备份文件。

tar 是用来建立，还原备份文件的工具程序，它可以加入，解开备份文件内的文件。

参数含义: c 表示 create， z 表示使用 gzip， v 表示 view 查看过程， f 指定备份文件， -C 指定要解压的目录, x 表示 extract 解压。t 表示list 列出压缩文件内容

压缩文件 非打包:

```bash
# touch a.c       
# tar -czvf test.tar.gz a.c   //压缩 a.c文件为test.tar.gz
a.c
```

列出压缩文件内容

```bash
# tar -tzvf test.tar.gz 
-rw-r--r-- root/root     0 2010-05-24 16:51:59 a.c
```

解压文件

```bash
# tar -xzvf test.tar.gz 
a.c
```

### cat

cat 是 concatenate的意思

（用于查看文本文件的内容，后接要查看的文件名，通常可用管道与more和less一起使用）

- `cat file1` 从第一个字节开始正向查看文件的内容
- `tac file1` 从最后一行开始反向查看一个文件的内容
- `cat -n file1` 标示文件的行数
- `more file1` 查看一个长文件的内容
- `head -n 2 file1` 查看一个文件的前两行
- `tail -n 2 file1` 查看一个文件的最后两行
- `tail -n +1000 file1` 从1000行开始显示，显示1000行以后的
- `cat filename | head -n 3000 | tail -n +1000` 显示1000行到3000行
- `cat filename | tail -n +3000 | head -n 1000` 从第3000行开始，显示1000(即显示3000~3999行)

### find

- `find / -name file1` 从 '/' 开始进入根文件系统搜索文件和目录
- `find / -user user1` 搜索属于用户 'user1' 的文件和目录
- `find /usr/bin -type f -atime +100` 搜索在过去100天内未被使用过的执行文件
- `find /usr/bin -type f -mtime -10` 搜索在10天内被创建或者修改过的文件
- `whereis halt` 显示一个二进制文件、源码或man的位置
- `which halt` 显示一个二进制文件或可执行文件的完整路径

### grep

（分析一行的信息，若当中有我们所需要的信息，就将该行显示出来，该命令通常与管道命令一起使用，用于对一些命令的输出进行筛选加工等等）

- `grep Aug /var/log/messages` 在文件 '/var/log/messages'中查找关键词"Aug"
- `grep ^Aug /var/log/messages` 在文件 '/var/log/messages'中查找以"Aug"开始的词汇
- `grep [0-9] /var/log/messages` 选择 '/var/log/messages' 文件中所有包含数字的行
- `grep Aug -R /var/log/*` 在目录 '/var/log' 及随后的目录中搜索字符串"Aug"
- `sed 's/stringa1/stringa2/g' example.txt` 将example.txt文件中的 "string1" 替换成 "string2"
- `sed '/^$/d' example.txt` 从example.txt文件中删除所有空白行

