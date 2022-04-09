# Unix进程模型


<!--more-->

## 进程的概念

进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础。在早期面向进程设计的计算机结构中，进程是程序的基本执行实体；在当代面向线程设计的计算机结构中，进程是线程的容器。程序是指令、数据及其组织形式的描述，进程是程序的实体。

  一个进程在CPU上运行可以有两种运行模式（进程状态）：用户模式和内核模式。如果当前运行的是用户程序（用户代码），那么对应进程就处于用户模式（用户态），如果出现系统调用或者发生中断，那么对应进程就处于内核模式（核心态）

## 进程的状态模型

下图为进程的7状态模型，常用的5状态模型在该图的基础上去掉了挂起状态。其中包括如下状态：

- **New State**：New state is the state when the process is under creation

- **Ready State**: When the process is created new state comes, which is called *ready state.*  After creation process comes under ready state. In ready state more than one process can also come.  For example: One process is created at the same time second process is created then both the process will come under ready state.

- **Running State**: From ready state we have to select a process, and then have to allot CPU to that process for run.  When CPU is allotted to process in ready state that process comes in *running state.*  In running state only one process can stay at a time. Because CPU can be allotted to single process at a time.

- **Wait State**: When a process request for input/output than that process will left the running state, and will join new state known as wait state.  In wait state more than one process can stay. After completion of I/O request process will go to ready state.

- **Termination State**: When process comes in running state, there is no more input output request by the process, because it’s already get completed.  So process will go to *termination state.*

- **Suspend Ready State**: When ready state is not able to occupy more states in it, than some states are suspended in *suspended state.* Suspend ready state will be in *secondary memory* not in primary memory. When ready state get space for new processes than, processes from suspended ready state gets switch back to ready state.  Such transaction is known as *resume.*

- **Suspend Wait State**: Similarly suspend wait state is also reside in process state diagram.

<img src="https://i0.hdslb.com/bfs/album/5d684d7f561832c46614cabf50fb18ac819bc078.png" title="" alt="" data-align="center">

## Linux进程组织

Linux进程通过一个`task_struct`结构体描述，在`linux/sched.h`中定义，通过理解该结构，可更清楚的理解linux进程模型。包含进程所有信息的`task_struct`数据结构是比较庞大的，但是该数据结构本身并不复杂，我们将它的所有域按其功能可做如下划分：

-    进程状态（State）
-    进程调度信息（Scheduling Information）
-    各种标识符（Identifiers）
-    进程通信有关信息（IPC：Inter_Process Communication）
-    时间和定时器信息（Times and Timers）
-    进程链接信息（Links）
-    文件系统信息（File System）
-    虚拟内存信息（Virtual Memory）
-    页面管理信息（page）
-    对称多处理器（SMP）信息
-    和处理器相关的环境（上下文）信息（Processor Specific Context）
-    其他信息

### 进程状态

为了对进程从产生到消亡的整个过程进行跟踪和描述，就需要定义各种进程的各种状态并制定相应的状态转换策略，以此来控制进程的运行。

Linux系统中，进程状态在 `task_struct` 中定义如下：

```c
volatile long state;    /* -1 unrunnable, 0 runnable, >0 stopped */
```

其状态取值如下：

```c
#define TASK_RUNNING        0
#define TASK_INTERRUPTIBLE  1
#define TASK_UNINTERRUPTIBLE    2
#define __TASK_STOPPED      4
#define __TASK_TRACED       8
/* in tsk->exit_state */
#define EXIT_ZOMBIE     16
#define EXIT_DEAD       32
/* in tsk->state again */
#define TASK_DEAD       64
#define TASK_WAKEKILL       128
#define TASK_WAKING     256
#define TASK_STATE_MAX      512
```

对进程每个状态简析如下：

- `TASK_RUNNING `（可运行状态）：处于这种状态的进程，要么正在运行、要么正准备运行。正在运行的进程就是当前进程（由`current`所指向的进程），而准备运行的进程只要得到CPU就可以立即投入运行，CPU是这些进程唯一等待的系统资源。

- `TASK_INTERRUPTIBLE`（可中断的等待状态）：表示进程被阻塞（睡眠），直到某个条件达成，进程的状态就被设置为`TASK_RUNNING`。处于该状态的进程正在等待某个事件（`event`）或某个资源，而被挂起。对应的`task_struct`结构被放入对应事件的等待队列中。处于可中断等待态的进程可以被信号（外部中断触发或者其他进程触发）唤醒，如果收到信号，该进程就从等待状态进入可运行状态，并且加入到运行队列中，等待被调度。

- `TASK_UNINTERRUPTIBLE`（不可中断的等待状态）：该状态与 `TASK_INTERRUPTIBLE `状态类似，也表示进程被阻塞，处于睡眠状态。当进程等待的某些条件被满足了之后，内核也会将该进程的状态设置为 `TASK_RUNNING`。但是，处于这个状态下的进程不能在接收到某个信号之后立即被唤醒。这时该状态与 `TASK_INTERRUPTIBLE` 状态唯一的区别。

- `__TASK_STOPPED`（暂停状态）：此时的进程暂时停止运行来接受某种特殊处理。通常当进程接收到`SIGSTOP`、`SIGTSTP`、`SIGTTIN`或 `SIGTTOU`信号后就处于这种状态。例如，正接受调试的进程就处于这种状态。

- `__TASK_TRACED`（跟踪状态）：当前进程正在被另一个进程所监视。

- `EXIT_ZOMBIE`（僵死状态）：进程虽然已经终止，但由于某种原因，父进程还没有执行wait()系统调用，终止进程的信息也还没有回收。顾名思义，处于该状态的进程就是死进程，这种进程实际上是系统中的垃圾，必须进行相应处理以释放其占用的资源。

- `EXIT_DEAD`：一个进程的最终状态。

以下是LINUX进程间状态转换和内核调用图解

<img src="https://i0.hdslb.com/bfs/album/d15e830eabac7e7d47df0ad425732d19366f52a3.png@1e_1c.webp" title="" alt="" data-align="center">

### 进程内存布局

每个进程所分配的内存由很多部分组成，通常称之为“段（`segment`）”：

- **文本段**：包含了进程运行的程序机器语言指令。文本段具有只读属性，以防止进程通过错误指针意外修改自身指令。因为多个进程可同时运行同一程序，所以又将文本段设为可共享，这样，一份程序代码的拷贝可以映射到所有这些进程的虚拟地址空间中。
- **初始化数据段**：包含显示初始化的全局变量和静态变量。当程序加载到内存时，从可执行文件中读取这些变量的值。
- **未初始化数据段**：包含了未进行显示初始化的全局变量和静态变量。程序启动之前，系统将本段内所有内存初始化为0。出于历史原因，此段常被称为`BSS`段，这源于老版本的汇编语言助记符“block started by symbol”。将经过初始化的全局变量和静态变量与未初始化的全局变量和静态变量分开存放，其主要原因在于程序在磁盘上存储时，没有必要为未经初始化的变量分配存储空间。相反，可执行文件只需记录未初始化数据段的位置及所需大小，直到运行时再由程序加载器来分配空间。
- **栈**（`stack`）：是一个动态增长和收缩的段，有栈帧（`stack frames`）组成。系统会为每个当前调用的函数分配一个栈帧。栈帧中存储了函数的局部变量（所谓自动变量）、实参和返回值。
- **堆**（`heap`）：是可在运行时（为变量）动态进行内存分配的一块区域。堆顶端称为`program break`。

对于初始化和未初始化的数据段而言，不太常用、但表达更清晰的称为分别是用户初始化数据段（`user-initialized data segment`）和零初始化数据段（`zero-initialized data segment`）。

<img src="https://i0.hdslb.com/bfs/album/1bd649c75bd08d9fbae1dbb90374b5fe393e460a.png" title="" alt="" data-align="center">

在大多数Unix（包括Linux）中的C语言编程环境提供了3个全局符号（symbol）：`etext`、`edata`、`end`，可以在程序中使用这些符号以获取相应程序文本段、初始化数据段和非初始化数据段结尾处下一字节的地址。

使用这些符号，必须显式声明如下：

```c
// For example, &etext gives the address of the end of the program text / start of initialized data
extern char etext, edata, end;
```

图中标灰的区域表示这些范围在进程虚拟地址空间中不可用，也就是说，没有为这些区域创建页表（`page table`）。

### 孤儿进程（orphan）

我们知道在unix/linux中，正常情况下，子进程是通过父进程创建的，子进程在创建新的进程。子进程的结束和父进程的运行是一个异步过程,即父进程永远无法预测子进程 到底什么时候结束。 当一个 进程完成它的工作终止之后，它的父进程需要调用`wait()`或者`waitpid()`系统调用取得子进程的终止状态。

**孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被`init`进程(进程号为1)所收养，并由`init`进程对它们完成状态收集工作。**

```c
#include <unistd.h>
#include <stdio.h>

int main ()
{
    /*fpid表示fork函数返回的值,fork会返回两次，
    一次是父进程，返回值是子进程的Pid，在子进程会返回0*/
    pid_t fpid;
    fpid=fork();//fork后会出现两个分支执行下面的代码，一个父进程，一个新的子进程
    if (fpid < 0)
        printf("fork error!");
    else if (fpid == 0) { //
        printf("child id is %dn",getpid());
        sleep(100);
    }
    else { //父进程
        printf("parent id is %dn",getpid());
        sleep(30);//睡眠30s，在子进程之前退出
        printf("parend finally...");
    }
}
```

### 僵尸进程（zombie）

**僵尸进程：一个进程使用`fork`创建子进程，如果子进程退出，而父进程并没有调用`wait`或`waitpid`获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵尸进程。**

unix提供了一种机制可以保证只要父进程想知道子进程结束时的状态信息， 就可以得到。这种机制就是: 在每个进程退出的时候,内核释放该进程所有的资源,包括打开的文件,占用的内存等。 但是仍然为其保留一定的信息(包括进程号the process ID,退出状态the termination status of the process,运行时间the amount of CPU time taken by the process等)。直到父进程通过`wait / waitpid`来取时才释放。 

但这样就导致了问题，**如果进程不调用`wait / waitpid`的话，** **那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵死进程，将因为没有可用的进程号而导致系统不能产生新的进程. 此即为僵尸进程的危害，应当避免。**

**孤儿进程是没有父进程的进程，孤儿进程这个重任就落到了init进程身上**，init进程就好像是一个民政局，专门负责处理孤儿进程的善后工作。每当出现一个孤儿进程的时候，内核就把孤 儿进程的父进程设置为init，而init进程会循环地`wait()`它的已经退出的子进程。这样，当一个孤儿进程凄凉地结束了其生命周期的时候，init进程就会代表党和政府出面处理它的一切善后工作。**因此孤儿进程并不会有什么危害。**

任何一个子进程(init除外)在`exit()`之后，并非马上就消失掉，而是留下一个称为僵尸进程(Zombie)的数据结构，等待父进程处理。**这是每个 子进程在结束时都要经过的阶段。如果子进程在`exit()`之后，父进程没有来得及处理，这时用`ps`命令就能看到子进程的状态是“Z”。如果父进程能及时 处理，可能用`ps`命令就来不及看到子进程的僵尸状态，但这并不等于子进程不经过僵尸状态。  如果父进程在子进程结束之前退出，则子进程将由init接管。init将会以父进程的身份对僵尸状态的子进程进行处理。

僵尸进程危害场景：

例如有个进程，它定期的产 生一个子进程，这个子进程需要做的事情很少，做完它该做的事情之后就退出了，因此这个子进程的生命周期很短，但是，父进程只管生成新的子进程，至于子进程 退出之后的事情，则一概不闻不问，这样，系统运行上一段时间之后，系统中就会存在很多的僵死进程，倘若用ps命令查看的话，就会看到很多状态为Z的进程。 严格地来说，僵死进程并不是问题的根源，罪魁祸首是产生出大量僵死进程的那个父进程。因此，当我们寻求如何消灭系统中大量的僵死进程时，答案就是把产生大 量僵死进程的那个元凶枪毙掉（也就是通过kill发送SIGTERM或者SIGKILL信号啦）。枪毙了元凶进程之后，它产生的僵死进程就变成了孤儿进 程，这些孤儿进程会被init进程接管，init进程会wait()这些孤儿进程，释放它们占用的系统进程表中的资源，这样，这些已经僵死的孤儿进程 就能瞑目而去了。

```c
#include <unistd.h>
#include <stdio.h>

int main ()
{
    /*fpid表示fork函数返回的值,fork会返回两次，
    一次是父进程，返回值是子进程的Pid，在子进程会返回0*/
    pid_t fpid;
    fpid=fork();//fork后会出现两个分支执行下面的代码，一个父进程，一个新的子进程
    if (fpid < 0)
        printf("fork error!");
    else if (fpid == 0) { //
        printf("child id is %dn",getpid());
        sleep(30);//睡眠30s，在父进程之前退出
        printf("child finally...");
    }
    else { //父进程
        printf("parent id is %dn",getpid());
        sleep(60);
        printf("parend finally...");
    }
}
```

#### 僵尸进程解决办法

##### 通过信号机制

子进程退出时向父进程发送`SIGCHILD`信号，父进程处理`SIGCHILD`信号。在信号处理函数中调用`wait`进行处理僵尸进程。测试程序如下所示：

```c
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
#include <stdlib.h>
#include <signal.h>

static void sig_child(int signo);

int main()
{
    pid_t pid;
    //创建捕捉子进程退出信号
    signal(SIGCHLD,sig_child);
    pid = fork();
    if (pid < 0)
    {
        perror("fork error:");
        exit(1);
    }
    else if (pid == 0)
    {
        printf("I am child process,pid id %d.I am exiting.\n",getpid());
        exit(0);
    }
    printf("I am father process.I will sleep two seconds\n");
    //等待子进程先退出
    sleep(2);
    //输出进程信息
    system("ps -o pid,ppid,state,tty,command");
    printf("father process is exiting.\n");
    return 0;
}

static void sig_child(int signo)
{
     pid_t        pid;
     int        stat;
     //处理僵尸进程
     while ((pid = waitpid(-1, &stat, WNOHANG)) >0)
            printf("child %d terminated.\n", pid);
}
```

##### fork两次

《Unix 环境高级编程》8.6节说的非常详细。原理是将<mark>子进程成为孤儿进程，从而其的父进程变为init进程</mark>，通过init进程可以处理僵尸进程。测试程序如下所示

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>

int main()
{
    pid_t  pid;
    //创建第一个子进程
    pid = fork();
    if (pid < 0)
    {
        perror("fork error:");
        exit(1);
    }
    //第一个子进程
    else if (pid == 0)
    {
        //子进程再创建子进程
        printf("I am the first child process.pid:%d\tppid:%d\n",getpid(),getppid());
        pid = fork();
        if (pid < 0)
        {
            perror("fork error:");
            exit(1);
        }
        //第一个子进程退出
        else if (pid >0)
        {
            printf("first procee is exited.\n");
            exit(0);
        }
        //第二个子进程
        //睡眠3s保证第一个子进程退出，这样第二个子进程的父亲就是init进程里
        sleep(3);
        printf("I am the second child process.pid: %d\tppid:%d\n",getpid(),getppid());
        exit(0);
    }
    //父进程处理第一个子进程退出
    if (waitpid(pid, NULL, 0) != pid)
    {
        perror("waitepid error:");
        exit(1);
    }
    exit(0);
    return 0;
}
```

### 守护进程

#### 控制终端

在 UNIX 系统中，用户通过终端登录系统后得到一个 shell 进程，这个终端成为 shell 进程的**控制终端**（Controlling Terminal），进程中，控制终端是保存在 `PCB` 中的信息，而 `fork() `会复制 PCB 中的信息，因此由 shell 进程启动的其它进程的控制终端也是这个终端。 

默认情况下（没有重定向），每个进程的标准输入、标准输出和标准错误输出都指向控制终端，进程从标准输入读也就是读用户的键盘输入，进程往标准输出或标准错误输出写也就是输出到显示器上。  

在控制终端输入一些特殊的控制键可以给前台进程发信号，例如` Ctrl + C `会产生 `SIGINT `信号，`Ctrl + \ `会产生 `SIGQUIT `信号

**进程组**和**会话**在进程之间形成了一种两级层次关系：**进程组是一组相关进程的集合，会话是一组相关进程组的集合**。进程组和会话是为支持 **shell 作业控制**而定义的抽象概念，用户通过 shell 能够交互式地在前台或后台运行命令。  

#### 进程组

进程组由一个或多个共享同一进程组标识符（`PGID`）的进程组成。一个进程组拥有一个进程组首进程，该进程是创建该组的进程，其进程 ID 为该进程组的 ID，新进程会继承其父进程所属的进程组 ID。  

进程组拥有一个生命周期，其**开始时间为首进程创建组的时刻，结束时间为最后一个成员进程退出组的时刻**。一个进程可能会因为终止而退出进程组，也可能会因为加入了另外一个进程组而退出进程组。**进程组首进程无需是最后一个离开进程组的成员**

#### 会话

**会话是一组进程组的集合。会话首进程是创建该新会话的进程**，其进程 ID 会成为会话 ID。新进程会继承其父进程的会话 ID。  

**一个会话中的所有进程共享单个控制终端**。控制终端会在会话首进程首次打开一个终端设备时被建立。一个终端最多可能会成为一个会话的控制终端。  

**在任一时刻，会话中的其中一个进程组会成为终端的前台进程组，其他进程组会成为后台进程组**。只有前台进程组中的进程才能从控制终端中读取输入。当用户在控制终端中输入终端字符生成信号后，该信号会被发送到前台进程组中的所有成员。  

当控制终端的连接建立起来之后，**会话首进程会成为该终端的控制进程**

<img src="https://i0.hdslb.com/bfs/album/72b29d2f169c0222c15c3cadacae62607bf1c6af.png" title="" alt="" data-align="center">

#### 守护进程

**守护进程**（Daemon Process），也就是通常说的 `Daemon `进程（精灵进程），是Linux 中的后台服务进程。它是一个生存期较长的进程，通常独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。一般采用以 d 结尾的名字。  

##### 守护进程特征

生命周期很长，守护进程会在系统启动的时候被创建并一直运行直至系统被关闭。  

它在后台运行并且不拥有控制终端。没有控制终端确保了内核永远不会为守护进程自动生成任何控制信号以及终端相关的信号（如 `SIGINT`、`SIGQUIT`）。  

进程调用 `setsid `函数建立一个新会话，如果调用此函数的进程不是一个进程组的组长，则此函数创建一个新会话。具体会发生以下3件事：

- 该进程变成新会话的会话首进程（session leader，会话首进程是创建该会话的进程）。此时，该进程是新会话的唯一进程。
- 该进程成为一个新进程组的组长进程。新进程组ID是该调用进程的进程ID
- 该进程没有控制终端。如果调用`setsid`之前该进程有一个控制终端，那么这种联系也被切断

**如果该调用进程已经是一个进程组的组长，则此函数返回出错**。为了保证不处于这种情况，通常先调用`fork`，然后使其父进程终止，而子进程则继续。因为子进程继承了父进程的进程组ID，而其进程ID是重新分配的，两者不可能相等，这就保证了子进程不是一个进程组的组长。我认为创建新会话的进程不能是组长进程的原因：在新创建的会话中，创建会话的进程成为了会话首进程，同时，该进程成为一个**新进程组的组长进程**。新进程组ID是该调用进程的进程ID，如果其已经是一个组长进程，那么就会产生矛盾。

##### 一般守护进程创建步骤

Linux 的大多数服务器就是用守护进程实现的。比如，Internet 服务器 inetd，Web 服务器 httpd 等。

1. 执行一个 `fork()`，之后父进程退出，子进程继续执行。 

2. 子进程调用 `setsid()` 开启一个新会话。 

3.  清除进程的 `umask` 以确保当守护进程创建文件和目录时拥有所需的权限。  

4. 修改进程的当前工作目录，通常会改为根目录（`/`）。  

5. 关闭守护进程从其父进程继承而来的所有打开着的文件描述符。  

6. 在关闭了文件描述符0、1、2之后，守护进程通常会打开`/dev/null` 并使用`dup2() `使所有这些描述符指向这个设备。  

7. 核心业务逻辑

#### 示例

```c
 /*
    写一个守护进程，每隔2s获取一下系统时间，将这个时间写入到磁盘文件中。
*/

#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/time.h>
#include <signal.h>
#include <time.h>
#include <stdlib.h>
#include <string.h>

void work(int num) {
    // 捕捉到信号之后，获取系统时间，写入磁盘文件
    time_t tm = time(NULL);
    struct tm * loc = localtime(&tm);
    // char buf[1024];

    // sprintf(buf, "%d-%d-%d %d:%d:%d\n",loc->tm_year,loc->tm_mon
    // ,loc->tm_mday, loc->tm_hour, loc->tm_min, loc->tm_sec);

    // printf("%s\n", buf);

    char * str = asctime(loc);
    int fd = open("time.txt", O_RDWR | O_CREAT | O_APPEND, 0664);
    write(fd ,str, strlen(str));
    close(fd);
}

int main() {

    // 1.创建子进程，退出父进程
    pid_t pid = fork();

    if(pid > 0) {
        exit(0);
    }

    // 2.将子进程重新创建一个会话
    setsid();

    // 3.设置掩码
    umask(022);

    // 4.更改工作目录
    chdir("/home/nowcoder/");

    // 5. 关闭、重定向文件描述符
    int fd = open("/dev/null", O_RDWR);
    dup2(fd, STDIN_FILENO);
    dup2(fd, STDOUT_FILENO);
    dup2(fd, STDERR_FILENO);

    // 6.业务逻辑

    // 捕捉定时信号
    struct sigaction act;
    act.sa_flags = 0;
    act.sa_handler = work;
    sigemptyset(&act.sa_mask);
    sigaction(SIGALRM, &act, NULL);

    struct itimerval val;
    val.it_value.tv_sec = 2;
    val.it_value.tv_usec = 0;
    val.it_interval.tv_sec = 2;
    val.it_interval.tv_usec = 0;

    // 创建定时器
    setitimer(ITIMER_REAL, &val, NULL);

    // 不让进程结束
    while(1) {
        sleep(10);
    }

    return 0;
}
```

### 进程管理命令

##### ps

ps命令（`Processes  Statistic`）时Linux系统中最为常用的进程查看工具，主要用于显示包含当前运行的各进程完整信息的静态快照。通过不同的命令选项，可以有选择性地查看进程信息。

■ a：显示当前终端下的所有进程信息，包括其他用户的进程。与“x”选项结合时将显示系统中所有的进程信息。

■ u：使用以用户为主的格式输出进程信息。

■ x：显示当前用户在所有终端下的进程信息。

■ –e：显示系统内的所有进程信息。

■ –l：使用长格式显示进程信息。

■ –f：使用完整的格式显示进程信息。

以上是ps命令中几个常用的选项，需要注意的是，有一部分选项时不带“-”前缀的。习惯上将上述选项组合在一起使用，如执行“`ps aux`”或“`ps -elf`”

<img src="https://i0.hdslb.com/bfs/album/7acd5951c3a44a4432cabc9d7a16d2ed3440f6c2.png" title="" alt="" data-align="center">

上述输出信息中，第一行为列表标题，其各字段的含义描述如下。

■ USER：启动该进程的户账号的名称。

■ PID：该进程在系统中的数字ID号，在当前系统中是唯一的。

■ TTY：表明该进程在哪个终端上运行。“？”表示未知或不需要终端。

■ STAT:显示了进程当前的状态，如S（休眠）、R（运行）、Z（僵死）、<（高优先级）、N（低优先级）、s（父进程）、+（前台进程）。

■ START：启动该进程的时间。

■ TIME：该进程占用的CPU时间。

■ COMMAND：启动该进程的命令的名称。

■ %CPU：CPU占用的百分比。

■ %MEM：内存占用的百分比。

■ VSZ：占用虚拟内存（swap空间）的大小。

■ RSS：占用常驻内存（物理内存）的大小。

#### top

top命令将会在当前终端以全屏交互式的界面显示进程排名，及时跟踪包括CPU、内存等系统资源占用情况，默认情况下每三秒刷新一次。作用类似Windows系统中的“任务管理器”。

<img src="https://i0.hdslb.com/bfs/album/bd6bce53bfcc21d7823cd16aabf1da5d883f5372.png" title="" alt="" data-align="center">

上述输出信息中，开头的部分显示了系统任务（Tasks）、CPU占用、内存占用（Mem）、交换空间（Swap）等汇总信息；汇总信息下方依次显示当前进程的排名情况。相关信息的含义表述如下。

**系统任务（Tasks）信息**：`total`，总进程数; `running`,正在运行的进程数；`sleeping`，休眠的进程数；`stopped`，终止的进程数；`zombie`，僵死无响应的进程数。

**CPU占用信息**：`us`，用户占用；`sy`，内核占用；`ni`，优先级调度占用；`id`，空闲CPU；`wa`，1/0等占用； `hi`，硬件中断占用，`si`，软件中断占用；`st`，虚拟化占用。

**内存占用（Mem）信息**：`total`，总内存空间；`used`，已用内存；`free`，空闲内存；`buffers`，缓冲区域。

**交换空间（Swap）占用**：`total`，总内存空间；`used`，已用内存；`free`，空闲内存；     `buffers`，缓冲区域。

在`top`命令的全屏操作界面中：可以按P键根据CPU占用情况对进程列表进行排序。

> 按M键根据内存占用情况进行排序。
> 
> 按N键根据启动时间进行排序。
> 
> 按h键可以获得top程序的在线帮助。
> 
> 按q键可以正常的退出top程序。
> 
> 按k键在列表上方会出现“PID to kill：”的提示信息，根据提示输入指定进程的PID并按Enter键确认即可终止对应的进程。

#### pgrep

使用`pgrep`命令可以根据进程的名称、运行该进程的用户、进程所在的终端等多种属性查询特定进程的PID号。

通过pgrep命令，可以只指定进程的一部分名称进行查询，结合“-l”选项可同时输出对应的进程名（否则只输PID号，不便于理解）。例如，查询进程中包含“log”的进程及其PID号，可以执行以下操作。

```bash
[root@lwy~]# pgrep -l “log”
2538  rsyslog
2113  mcelog
```

还可以结合“-U”选项查询特定用户的进程、“-t”选项查询在特定终端运行的进程。例如，若要查询用户zhangsan在tty3终端上运行的进程及PID号，可以执行以下操作。

```bash
[root@lwy~]# pgrep -l -U zhangsan-t tty3
2105 bash
2122 vim
```

#### pstree

`pstree`命令可以输出Linux系统中各进程的树形结构，以更加直观地判断出各进程之间的相互关系（父、子进程）。

`pstree`命令默认情况下只显示个进程的名称，结合“-p”选项使用时可以同时列出对应的PID号，结合“-u”选项可以列出对应的用户名，结合“-a”选项可以列出完整的命令信息。例如，执行“`pstree -aup`”命令可以查看当前系统的进程树，包括个进程对应的PID号、用户名、完整命令等信息。从出输出结果中可以看出，init进程确实是Linux系统中所有进程的“始祖”

#### 改变进程的运行方式

##### 挂起当前的进程

  当Linux系统中的命令正在前台执行时（运行尚未结束），按`Ctrl+Z`组合键可以将当前进程挂起（调入后台并停止执行），这种操作在需要暂停当前进程并进行其他操作时特别有用。

```bash
[root@lwy~]# cp /dev/cdrom/mnt/Redhat6.0.iso
[1]+ Stopped     cp -i/dev/cdrom /mnt/Redhat6.0.iso
```

##### 查看后台的进程

查看当前终端中在后台运行的进程任务时，可以使用`jobs`命令，结合“`-l`”选项可以同时显示出该进程对应的PID号。

```bash
[root@lwy~]# jobs -l
[1]+  2193 Stopped cp -i /dev/cdrom /mnt/Redhat6.0.iso
```

##### 将后台的进程恢复运行

■ 使用`bg`（BackGround，后台）命令，可以将后台中暂定执行（如按`Ctrl+Z`组合键挂起）的任务恢复运行，继续在后台执行操作、

■ 使用`fg`命令（ForeGround，前台）命令，可以将后台任务重新恢复到前台运行。

除非后台中的任务只有一个，否则`bg`和`fg`命令都需要指定后台进程的任务编号作为参数。

例如，执行“`fg 1`”命令可以将之前挂起至后台的cp进程重新调入前台执行。

```bash
[root@lwy/]# jobs
[1]+ Stopped         cp -i/dev/cdrom /mnt/Redhat6.0.iso
[root@lwy/]# fg 1
cp -i /dev/cdrom /mnt/Redhat6.0.iso
```

##### 用pkill命令终止进程

使用pkill命令可以根据进程的名称、运行该进程的用户、进程所在的终端等多种属性终止特定的进程，大部分选项与`pgrep`命令基本类似，如“`-U`”（指定用户）、“`-t`”（指定终端）等选项。

例如，若要终止由用户zhangsan启动的进程（包括登陆Shell），可以执行以下操作。

```bash
[root@lwy/]# pgrep -l -U"zhangsan"
2105 bash
[root@lwy/]# pkill -9 -U"zhangsan"
[root@lwy/]# pgrep -l -U"zhangsan"
```

