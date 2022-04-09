# Linux下的多线程


<!--more-->

## 线程概述

与进程（`process`）类似，线程（`thread`）是允许应用程序并发执行多个任务的一种机制。一个进程可以包含多个线程。同一个程序中的所有线程均会独立执行相同程序，且共享同一份全局内存区域，其中包括初始化数据段、未初始化数据段，以及堆内存段。（传统意义上的 UNIX 进程只是多线程程序的一个特例，该进程只包含一个线程）

 进程是 CPU 分配资源的最小单位，线程是操作系统调度执行的最小单位。  

线程是轻量级的进程（`LWP：Light Weight Process`），在 Linux 环境下线程的本质仍是进程。  

查看指定进程的 LWP 号：`ps –Lf pi`

◼ 当 Linux 最初开发时，在内核中并不能真正支持线程。但是它的确可以通过 clone()  系统调用将进程作为可调度的实体。这个调用创建了调用进程（calling process）的 一个拷贝，这个拷贝与调用进程共享相同的地址空间。LinuxThreads 项目使用这个调用来完成在用户空间模拟对线程的支持。不幸的是，这种方法有一些缺点，尤其是在信号处理、调度和进程间同步等方面都存在问题。另外，这个线程模型也不符合 POSIX 的要求。  

◼ 要改进 LinuxThreads，需要内核的支持，并且重写线程库。有两个相互竞争的项目开始 
来满足这些要求。一个包括 IBM 的开发人员的团队开展了 NGPT（Next-Generation POSIX Threads）项目。同时，Red Hat 的一些开发人员开展了 NPTL 项目。NGPT 在 2003 年中期被放弃了，把这个领域完全留给了 **NPTL**。 

◼ NPTL，或称为 **Native POSIX Thread Library**，是 Linux 线程的一个新实现，它克服了 LinuxThreads 的缺点，同时也符合 POSIX 的需求。与 LinuxThreads 相比，它在性能和稳定性方面都提供了重大的改进。  

◼ 查看当前 pthread 库版本：`getconf GNU_LIBPTHREAD_VERSION`

### 进程与线程之间的区别

进程间的信息难以共享。由于除去只读代码段外，父子进程并未共享内存，因此必须采用
一些进程间通信方式，在进程间进行信息交换。  

调用`fork()`来创建进程的代价相对较高，即便利用写时复制技术，仍然需要复制诸如内存页表和文件描述符表之类的多种进程属性，这意味着 `fork()` 调用在时间上的开销依然不菲。  

线程之间能够方便、快速地共享信息。只需将数据复制到共享（全局或堆）变量中即可。 

创建线程比创建进程通常要快 10 倍甚至更多。线程间是共享虚拟地址空间的，无需采用写时复制来复制内存，也无需复制页表

<img src="https://i0.hdslb.com/bfs/album/9c205712fa159a31956020d9892976988a945541.png" title="" alt="" data-align="center">

## 线程基本操作

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, 
void *(*start_routine) (void *), void *arg);
pthread_t pthread_self(void);
int pthread_equal(pthread_t t1, pthread_t t2);
void pthread_exit(void *retval);
int pthread_join(pthread_t thread, void **retval);
int pthread_detach(pthread_t thread);
int pthread_cancel(pthread_t thread);
```

### 线程创建（create）

```c
/*
    一般情况下,main函数所在的线程我们称之为主线程（main线程），其余创建的线程
    称之为子线程。
    程序中默认只有一个进程，fork()函数调用，2进行
    程序中默认只有一个线程，pthread_create()函数调用，2个线程。

    #include <pthread.h>
    int pthread_create(pthread_t *thread, const pthread_attr_t *attr, 
    void *(*start_routine) (void *), void *arg);

        - 功能：创建一个子线程
        - 参数：
            - thread：传出参数，线程创建成功后，子线程的线程ID被写到该变量中。
            - attr : 设置线程的属性，一般使用默认值，NULL
            - start_routine : 函数指针，这个函数是子线程需要处理的逻辑代码
            - arg : 给第三个参数使用，传参
        - 返回值：
            成功：0
            失败：返回错误号。这个错误号和之前errno不太一样。
            获取错误号的信息：  char * strerror(int errnum);

*/
#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>

void * callback(void * arg) {
    printf("child thread...\n");
    printf("arg value: %d\n", *(int *)arg);
    return NULL;
}

int main() {
    pthread_t tid;
    int num = 10;
    // 创建一个子线程
    int ret = pthread_create(&tid, NULL, callback, (void *)&num);
    if(ret != 0) {
        char * errstr = strerror(ret);
        printf("error : %s\n", errstr);
    } 
    for(int i = 0; i < 5; i++) {
        printf("%d\n", i);
    }
    sleep(1);
    return 0;   // exit(0);
} 
```

### 线程终止（exit）

```c
/*

    #include <pthread.h>
    void pthread_exit(void *retval);
        功能：终止一个线程，在哪个线程中调用，就表示终止哪个线程
        参数：
            retval:需要传递一个指针，作为一个返回值，可以在pthread_join()中获取到。

    pthread_t pthread_self(void);
        功能：获取当前的线程的线程ID

    int pthread_equal(pthread_t t1, pthread_t t2);
        功能：比较两个线程ID是否相等
        不同的操作系统，pthread_t类型的实现不一样，有的是无符号的长整型，有的
        是使用结构体去实现的。
*/
#include <stdio.h>
#include <pthread.h>
#include <string.h>

void * callback(void * arg) {
    printf("child thread id : %ld\n", pthread_self());
    return NULL;    // pthread_exit(NULL);
} 

int main() {
    // 创建一个子线程
    pthread_t tid;
    int ret = pthread_create(&tid, NULL, callback, NULL);
    if(ret != 0) {
        char * errstr = strerror(ret);
        printf("error : %s\n", errstr);
    }
    // 主线程
    for(int i = 0; i < 5; i++) {
        printf("%d\n", i);
    }
    printf("tid : %ld, main thread id : %ld\n", tid ,pthread_self());
    // 让主线程退出,当主线程退出时，不会影响其他正常运行的线程。
    pthread_exit(NULL);
    printf("main thread exit\n");
    return 0;   // exit(0);
}
```

### 线程汇合（join）

```c
/*
    #include <pthread.h>
    int pthread_join(pthread_t thread, void **retval);
        - 功能：和一个已经终止的线程进行连接
                回收子线程的资源
                这个函数是阻塞函数，调用一次只能回收一个子线程
                一般在主线程中使用
        - 参数：
            - thread：需要回收的子线程的ID
            - retval: 接收子线程退出时的返回值
        - 返回值：
            0 : 成功
            非0 : 失败，返回的错误号
*/

#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>

int value = 10;

void * callback(void * arg) {
    printf("child thread id : %ld\n", pthread_self());
    // sleep(3);
    // return NULL; 
    // int value = 10; // 局部变量
    pthread_exit((void *)&value);   // return (void *)&value;
} 

int main() {

    // 创建一个子线程
    pthread_t tid;
    int ret = pthread_create(&tid, NULL, callback, NULL);

    if(ret != 0) {
        char * errstr = strerror(ret);
        printf("error : %s\n", errstr);
    }

    // 主线程
    for(int i = 0; i < 5; i++) {
        printf("%d\n", i);
    }

    printf("tid : %ld, main thread id : %ld\n", tid ,pthread_self());

    // 主线程调用pthread_join()回收子线程的资源
    int * thread_retval;
    ret = pthread_join(tid, (void **)&thread_retval);

    if(ret != 0) {
        char * errstr = strerror(ret);
        printf("error : %s\n", errstr);
    }

    printf("exit data : %d\n", *thread_retval);

    printf("回收子线程资源成功！\n");

    // 让主线程退出,当主线程退出时，不会影响其他正常运行的线程。
    pthread_exit(NULL);

    return 0; 
}
```

### 线程分离（detach）

```c
/*
    #include <pthread.h>
    int pthread_detach(pthread_t thread);
        - 功能：分离一个线程。被分离的线程在终止的时候，会自动释放资源返回给系统。
          1.不能多次分离，会产生不可预料的行为。
          2.不能去连接一个已经分离的线程，会报错。
        - 参数：需要分离的线程的ID
        - 返回值：
            成功：0
            失败：返回错误号
*/
#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>

void * callback(void * arg) {
    printf("chid thread id : %ld\n", pthread_self());
    return NULL;
}

int main() {

    // 创建一个子线程
    pthread_t tid;

    int ret = pthread_create(&tid, NULL, callback, NULL);
    if(ret != 0) {
        char * errstr = strerror(ret);
        printf("error1 : %s\n", errstr);
    }

    // 输出主线程和子线程的id
    printf("tid : %ld, main thread id : %ld\n", tid, pthread_self());

    // 设置子线程分离,子线程分离后，子线程结束时对应的资源就不需要主线程释放
    ret = pthread_detach(tid);
    if(ret != 0) {
        char * errstr = strerror(ret);
        printf("error2 : %s\n", errstr);
    }

    // 设置分离后，对分离的子线程进行连接 pthread_join()
    // ret = pthread_join(tid, NULL);
    // if(ret != 0) {
    //     char * errstr = strerror(ret);
    //     printf("error3 : %s\n", errstr);
    // }

    pthread_exit(NULL);

    return 0;
}
```

### 线程取消（cancel）

```c
/*
    #include <pthread.h>
    int pthread_cancel(pthread_t thread);
        - 功能：取消线程（让线程终止）
            取消某个线程，可以终止某个线程的运行，
            但是并不是立马终止，而是当子线程执行到一个取消点，线程才会终止。
            取消点：系统规定好的一些系统调用，我们可以粗略的理解为从用户区到内核区的切换，这个位置称之为取消点。
*/

#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>

void * callback(void * arg) {
    printf("chid thread id : %ld\n", pthread_self());
    for(int i = 0; i < 5; i++) {
        printf("child : %d\n", i);
    }
    return NULL;
}

int main() {

    // 创建一个子线程
    pthread_t tid;

    int ret = pthread_create(&tid, NULL, callback, NULL);
    if(ret != 0) {
        char * errstr = strerror(ret);
        printf("error1 : %s\n", errstr);
    }

    // 取消线程
    pthread_cancel(tid);

    for(int i = 0; i < 5; i++) {
        printf("%d\n", i);
    }

    // 输出主线程和子线程的id
    printf("tid : %ld, main thread id : %ld\n", tid, pthread_self());


    pthread_exit(NULL);

    return 0;
}
```

### 线程属性（attr）

```c
/*
    int pthread_attr_init(pthread_attr_t *attr);
        - 初始化线程属性变量

    int pthread_attr_destroy(pthread_attr_t *attr);
        - 释放线程属性的资源

    int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);
        - 获取线程分离的状态属性

    int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
        - 设置线程分离的状态属性
*/     

#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>

void * callback(void * arg) {
    printf("chid thread id : %ld\n", pthread_self());
    return NULL;
}

int main() {

    // 创建一个线程属性变量
    pthread_attr_t attr;
    // 初始化属性变量
    pthread_attr_init(&attr);

    // 设置属性
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

    // 创建一个子线程
    pthread_t tid;

    int ret = pthread_create(&tid, &attr, callback, NULL);
    if(ret != 0) {
        char * errstr = strerror(ret);
        printf("error1 : %s\n", errstr);
    }

    // 获取线程的栈的大小
    size_t size;
    pthread_attr_getstacksize(&attr, &size);
    printf("thread stack size : %ld\n", size);

    // 输出主线程和子线程的id
    printf("tid : %ld, main thread id : %ld\n", tid, pthread_self());

    // 释放线程属性资源
    pthread_attr_destroy(&attr);

    pthread_exit(NULL);

    return 0;
}
```

## 线程同步（synchronization）

◼ 线程的主要优势在于，能够通过全局变量来共享信息。不过，这种便捷的共享是有代价的：必须确保多个线程不会同时修改同一变量，或者某一线程不会读取正在由其他线程修改的变量。  

◼ <mark>临界区</mark>是指访问某一共享资源的代码片段，并且这段代码的执行应为原子操作，也就是同时访问同一共享资源的其他线程不应终端该片段的执行。  

◼ 线程同步：即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作，其他线程才能对该内存地址进行操作，而其他线程则处于等待状态。

### 互斥量（锁）

#### 互斥量基本原理

为避免线程更新共享变量时出现问题，可以使用互斥量（mutex 是 **mutual exclusion** 的缩写）来确保同时仅有一个线程可以访问某项共享资源。可以使用互斥量来保证对任意共享资源的原子访问。 

◼ 互斥量有两种状态：已锁定（locked）和未锁定（unlocked）。任何时候，至多只有一个线程可以锁定该互斥量。试图对已经锁定的某一互斥量再次加锁，将可能阻塞线程或者报错失败，具体取决于加锁时使用的方法。  

◼ 一旦线程锁定互斥量，随即成为该互斥量的所有者，只有所有者才能给互斥量解锁。一般情况下，对每一共享资源（可能由多个相关变量组成）会使用**不同的互斥量**，每一线程在访问同一资源时将采用如下协议：  

1. 针对共享资源锁定互斥量  

2. 访问共享资源  

3. 对互斥量解锁

◼ 如果多个线程试图执行这一块代码（一个临界区），事实上只有一个线程能够持有该互斥量（其他线程将遭到阻塞），即同时只有一个线程能够进入这段代码区域，如下图所示：

<img title="" src="https://i0.hdslb.com/bfs/album/7e02471c53c2b07e42322c73de653fa9c52b2746.png" alt="" data-align="center">

#### 互斥量示例

```c
/*
    互斥量的类型 pthread_mutex_t
    int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
        - 初始化互斥量
        - 参数 ：
            - mutex ： 需要初始化的互斥量变量
            - attr ： 互斥量相关的属性，NULL
        - restrict : C语言的修饰符，被修饰的指针，不能由另外的一个指针进行操作。
            pthread_mutex_t *restrict mutex = xxx;
            pthread_mutex_t * mutex1 = mutex;

    int pthread_mutex_destroy(pthread_mutex_t *mutex);
        - 释放互斥量的资源

    int pthread_mutex_lock(pthread_mutex_t *mutex);
        - 加锁，阻塞的，如果有一个线程加锁了，那么其他的线程只能阻塞等待

    int pthread_mutex_trylock(pthread_mutex_t *mutex);
        - 尝试加锁，如果加锁失败，不会阻塞，会直接返回。

    int pthread_mutex_unlock(pthread_mutex_t *mutex);
        - 解锁
*/
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

// 全局变量，所有的线程都共享这一份资源。
int tickets = 1000;

// 创建一个互斥量
pthread_mutex_t mutex;

void * sellticket(void * arg) {

    // 卖票
    while(1) {

        // 加锁
        pthread_mutex_lock(&mutex);

        if(tickets > 0) {
            usleep(6000);
            printf("%ld 正在卖第 %d 张门票\n", pthread_self(), tickets);
            tickets--;
        }else {
            // 解锁
            pthread_mutex_unlock(&mutex);
            break;
        }

        // 解锁
        pthread_mutex_unlock(&mutex);
    }



    return NULL;
}

int main() {

    // 初始化互斥量
    pthread_mutex_init(&mutex, NULL);

    // 创建3个子线程
    pthread_t tid1, tid2, tid3;
    pthread_create(&tid1, NULL, sellticket, NULL);
    pthread_create(&tid2, NULL, sellticket, NULL);
    pthread_create(&tid3, NULL, sellticket, NULL);

    // 回收子线程的资源,阻塞
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);
    pthread_join(tid3, NULL);

    pthread_exit(NULL); // 退出主线程

    // 释放互斥量资源
    pthread_mutex_destroy(&mutex);

    return 0;
}
```

### 死锁

◼ 有时，一个线程需要同时访问两个或更多不同的共享资源，而每个资源又都由不同的互斥量管理。当超过一个线程加锁同一组互斥量时，就有可能发生死锁。  

◼ 两个或两个以上的进程在执行过程中，因争夺共享资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁

#### 死锁的几种场景

- 忘记释放锁  

- 重复加锁  

- 多线程多锁，抢占锁资源

更多关于死锁的内容将在操作系统的相关理论文章中介绍，本文不再继续讨论。

#### 死锁产生的四个必要条件

虽然进程在运行过程中，可能发生死锁，但死锁的发生必须具备一定的条件，死锁的发生必须具有以下四个必要条件。

（1）**互斥条件**：指进程对所分配的资源进行排它性使用，即在一段时间内某资源只由一个进程占用。如果此时还有其它进程请求资源，则请求者只能等待，直至占有资源的进程用完释放。

（2）**请求和保持条件**：指进程已经保持至少一个资源，但又提出了新的资源请求，而该资源已被其它进程占有，此时请求进程阻塞，但又对自己已获得的其它资源保持不放。

（3）**不剥夺条件**：指进程已获得的资源，在未使用之前，不能被剥夺，只能在使用完时由自己释放。

（4）**循环等待条件**：指在发生死锁时，必然在一个进程 -- 资源的环形链，即进程集合{P0， P1， P2 ...， Pn} 中的 P0 正在等待一个 P1 占用的资源；P1 正在等待 P2 占用的资源，....，Pn正在等待已被 P0 占用的资源。

#### 死锁处理方法

在系统中已经出现死锁后，应该及时检测到死锁的发生，并采取适当的措施来解除死锁

（1）**预防死锁** ：这是一种较简单和直观的事先预防的方法。方法是通过设置某些限制条件，去破坏产生死锁的四个必要条件中的一个或者几个，来预防发生死锁。预防死锁是一种较易实现的方法，已被广泛使用。但是由于所施加的限制条件往往太严格，可能会导致系统资源利用率和系统吞吐量降低。

（2）**避免死锁**：该方法同样是属于事先预防的策略，但它并不须事先采取各种限制措施去破坏产生死锁的的四个必要条件，而是在资源的动态分配过程中，用某种方法去防止系统进入不安全状态，从而避免发生死锁。

（3）**检测和解除死锁**：先检测：这种方法并不须事先采取任何限制性措施，也不必检查系统是否已经进入不安全区，此方法允许系统在运行过程中发生死锁。但可通过系统所设置的检测机构，及时地检测出死锁的发生，并精确地确定与死锁有关的进程和资源。检测方法包括定时检测、效率低时检测、进程等待时检测等。然后解除死锁：采取适当措施，从系统中将已发生的死锁清除掉。

这是与检测死锁相配套的一种措施。当检测到系统中已发生死锁时，须将进程从死锁状态中解脱出来。常用的实施方法是撤销或挂起一些进程，以便回收一些资源，再将这些资源分配给已处于阻塞状态的进程，使之转为就绪状态，以继续运行。死锁的检测和解除措施，有可能使系统获得较好的资源利用率和吞吐量，但在实现上难度也最大。

#### 死锁示例

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

// 创建2个互斥量
pthread_mutex_t mutex1, mutex2;

void * workA(void * arg) {

    pthread_mutex_lock(&mutex1);
    sleep(1);
    pthread_mutex_lock(&mutex2);

    printf("workA....\n");

    pthread_mutex_unlock(&mutex2);
    pthread_mutex_unlock(&mutex1);
    return NULL;
}


void * workB(void * arg) {
    pthread_mutex_lock(&mutex2);
    sleep(1);
    pthread_mutex_lock(&mutex1);

    printf("workB....\n");

    pthread_mutex_unlock(&mutex1);
    pthread_mutex_unlock(&mutex2);

    return NULL;
}

int main() {

    // 初始化互斥量
    pthread_mutex_init(&mutex1, NULL);
    pthread_mutex_init(&mutex2, NULL);

    // 创建2个子线程
    pthread_t tid1, tid2;
    pthread_create(&tid1, NULL, workA, NULL);
    pthread_create(&tid2, NULL, workB, NULL);

    // 回收子线程资源
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);

    // 释放互斥量资源
    pthread_mutex_destroy(&mutex1);
    pthread_mutex_destroy(&mutex2);

    return 0;
}
```

### 读写锁

当有一个线程已经持有互斥锁时，互斥锁将所有试图进入临界区的线程都阻塞住。但是考虑一种情形，当前持有互斥锁的线程只是要读访问共享资源，而同时有其它几个线程也想读取这个共享资源，但是由于互斥锁的排它性，所有其它线程都无法获取锁，也就无法读访问共享资源了，但是实际上多个线程同时读访问共享资源并不会导致问题。  

◼ 在对数据的读写操作中，更多的是读操作，写操作较少，例如对数据库数据的读写应用。为了满足当前能够允许多个读出，但只允许一个写入的需求，线程提供了读写锁来实现。

◼ 读写锁的特点：

- 如果有其它线程读数据，则允许其它线程执行读操作，但不允许写操作。  

- 如果有其它线程写数据，则其它线程都不允许读、写操作。  

- 写是独占的，写的**优先级高**。

**示例**

```c
/*
    读写锁的类型 pthread_rwlock_t
    int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);
    int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
    int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
    int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
    int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
    int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
    int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);

    案例：8个线程操作同一个全局变量。
    3个线程不定时写这个全局变量，5个线程不定时的读这个全局变量
*/

#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

// 创建一个共享数据
int num = 1;
// pthread_mutex_t mutex;
pthread_rwlock_t rwlock;

void * writeNum(void * arg) {

    while(1) {
        pthread_rwlock_wrlock(&rwlock);
        num++;
        printf("++write, tid : %ld, num : %d\n", pthread_self(), num);
        pthread_rwlock_unlock(&rwlock);
        usleep(100);
    }

    return NULL;
}

void * readNum(void * arg) {

    while(1) {
        pthread_rwlock_rdlock(&rwlock);
        printf("===read, tid : %ld, num : %d\n", pthread_self(), num);
        pthread_rwlock_unlock(&rwlock);
        usleep(100);
    }

    return NULL;
}

int main() {

   pthread_rwlock_init(&rwlock, NULL);

    // 创建3个写线程，5个读线程
    pthread_t wtids[3], rtids[5];
    for(int i = 0; i < 3; i++) {
        pthread_create(&wtids[i], NULL, writeNum, NULL);
    }

    for(int i = 0; i < 5; i++) {
        pthread_create(&rtids[i], NULL, readNum, NULL);
    }

    // 设置线程分离
    for(int i = 0; i < 3; i++) {
       pthread_detach(wtids[i]);
    }

    for(int i = 0; i < 5; i++) {
         pthread_detach(rtids[i]);
    }

    pthread_exit(NULL);

    pthread_rwlock_destroy(&rwlock);

    return 0;
}
```

### 条件变量

互斥锁一个明显的缺点是它只有两种状态：锁定和非锁定。而条件变量通过**允许线程阻塞和等待另一个线程发送信号**的方法弥补了互斥锁的不足，它常和互斥锁一起配合使用。使用时，条件变量被用来阻塞一个线程，当条件不满足时，线程往往解开相应的互斥锁并等待条件发生变化。一旦其他的某个线程改变了条件变量，他将通知相应的条件变量唤醒一个或多个正被此条件变量阻塞的线程。这些线程将重新锁定互斥锁并重新测试条件是否满足。一般说来，条件变量被用来进行**线程间的同步**。

**条件本身是由互斥量保护的。线程在改变条件状态之前必须首先锁住互斥量。**

两个线程操作同一临界区时，通过互斥锁保护，若A线程已经加锁，B线程再加锁时候会被阻塞，直到A释放锁，B再获得锁运行，进程B必须不停的主动获得锁、检查条件、释放锁、再获得锁、再检查、再释放，一直到满足运行的条件的时候才可以（而此过程中其他线程一直在等待该线程的结束），这种方式是比较消耗系统的资源的。而条件变量同样是阻塞，还需要通知才能唤醒，线程被唤醒后，它将重新检查判断条件是否满足，如果还不满足，该线程就休眠了，应该仍阻塞在这里，等待条件满足后被唤醒，节省了线程不断运行浪费的资源。这个过程一般用`while`语句实现。当线程B发现被锁定的变量不满足条件时会自动的释放锁并把自身置于等待状态，让出CPU的控制权给其它线程。其它线程 此时就有机会去进行操作，当修改完成后再通知那些由于条件不满足而陷入等待状态的线程。这是一种通知模型的同步方式，大大的节省了CPU的计算资源，减少了线程之间的竞争，而且提高了线程之间的系统工作的效率。这种同步方式就是条件变量。

**示例（生产者-消费者模型）**

```c
/*
    条件变量的类型 pthread_cond_t
    int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);
    int pthread_cond_destroy(pthread_cond_t *cond);
    int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
        - 等待，调用了该函数，线程会阻塞。
    int pthread_cond_timedwait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime);
        - 等待多长时间，调用了这个函数，线程会阻塞，直到指定的时间结束。
    int pthread_cond_signal(pthread_cond_t *cond);
        - 唤醒一个或者多个等待的线程
    int pthread_cond_broadcast(pthread_cond_t *cond);
        - 唤醒所有的等待的线程
*/
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

// 创建一个互斥量
pthread_mutex_t mutex;
// 创建条件变量
pthread_cond_t cond;

struct Node{
    int num;
    struct Node *next;
};

// 头结点
struct Node * head = NULL;

void * producer(void * arg) {

    // 不断的创建新的节点，添加到链表中
    while(1) {
        pthread_mutex_lock(&mutex);
        struct Node * newNode = (struct Node *)malloc(sizeof(struct Node));
        newNode->next = head;
        head = newNode;
        newNode->num = rand() % 1000;
        printf("add node, num : %d, tid : %ld\n", newNode->num, pthread_self());

        // 只要生产了一个，就通知消费者消费
        pthread_cond_signal(&cond);

        pthread_mutex_unlock(&mutex);
        usleep(100);
    }

    return NULL;
}

void * customer(void * arg) {

    while(1) {
        pthread_mutex_lock(&mutex);
        // 保存头结点的指针
        struct Node * tmp = head;
        // 判断是否有数据
        if(head != NULL) {
            // 有数据
            head = head->next;
            printf("del node, num : %d, tid : %ld\n", tmp->num, pthread_self());
            free(tmp);
            pthread_mutex_unlock(&mutex);
            usleep(100);
        } else {
            // 没有数据，需要等待
            // 当这个函数调用阻塞的时候，会对互斥锁进行解锁，当不阻塞的，继续向下执行，会重新加锁。
            pthread_cond_wait(&cond, &mutex);
            pthread_mutex_unlock(&mutex);
        }
    }
    return  NULL;
}

int main() {

    pthread_mutex_init(&mutex, NULL);
    pthread_cond_init(&cond, NULL);

    // 创建5个生产者线程，和5个消费者线程
    pthread_t ptids[5], ctids[5];

    for(int i = 0; i < 5; i++) {
        pthread_create(&ptids[i], NULL, producer, NULL);
        pthread_create(&ctids[i], NULL, customer, NULL);
    }

    for(int i = 0; i < 5; i++) {
        pthread_detach(ptids[i]);
        pthread_detach(ctids[i]);
    }

    while(1) {
        sleep(10);
    }

    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&cond);

    pthread_exit(NULL);

    return 0;
}
```

### 信号量

信号量又称为信号灯（semaphore），其与自旋锁不同的地方是它可以引起调用者休眠，也就是信号量本质上是一种睡眠锁。如果有一个任务试图获得一个不可用（已经被占用）的信号量时，**信号量会将其推进一个等待队列，然后让其睡眠**。这时处理器能重获自由，从而去执行其他代码。当持有的信号量可用（被释放后），处于等待队列中的那个任务将被唤醒，并将获得该信号量。 信号量一个有用的特性是它**可以同时允许任意数量的锁持有者**，而自旋锁和互斥锁在一个时刻最多允许一个任务持有它。信号量同时允许的持有者数量可以在声明信号量时指定。这个值称为使用者数量（usage count）或简单的叫做数量（count）。通常情况下，信号量和自旋锁一样，在一个时刻仅允许有一个锁持有者。这时计数等于1，这样的信号量被称为二值信号量或者称为互斥信号量。另一方面，初始化时也可以把数量设置为大于1的非0值。这种情况，信号量被称为计数信号量（counting semaphore），它允许在同一时刻至多有count个锁持有者。

**示例（生产者-消费者模型）**

```c
/*
    信号量的类型 sem_t
    int sem_init(sem_t *sem, int pshared, unsigned int value);
        - 初始化信号量
        - 参数：
            - sem : 信号量变量的地址
            - pshared : 0 用在线程间 ，非0 用在进程间
            - value : 信号量中的值

    int sem_destroy(sem_t *sem);
        - 释放资源

    int sem_wait(sem_t *sem);
        - 对信号量加锁，调用一次对信号量的值-1，如果值为0，就阻塞

    int sem_trywait(sem_t *sem);

    int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
    int sem_post(sem_t *sem);
        - 对信号量解锁，调用一次对信号量的值+1

    int sem_getvalue(sem_t *sem, int *sval);

    sem_t psem;
    sem_t csem;
    init(psem, 0, 8);
    init(csem, 0, 0);

    producer() {
        sem_wait(&psem);
        sem_post(&csem)
    }

    customer() {
        sem_wait(&csem);
        sem_post(&psem)
    }

*/

#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>
#include <semaphore.h>

// 创建一个互斥量
pthread_mutex_t mutex;
// 创建两个信号量
sem_t psem;
sem_t csem;

struct Node{
    int num;
    struct Node *next;
};

// 头结点
struct Node * head = NULL;

void * producer(void * arg) {

    // 不断的创建新的节点，添加到链表中
    while(1) {
        sem_wait(&psem);
        pthread_mutex_lock(&mutex);
        struct Node * newNode = (struct Node *)malloc(sizeof(struct Node));
        newNode->next = head;
        head = newNode;
        newNode->num = rand() % 1000;
        printf("add node, num : %d, tid : %ld\n", newNode->num, pthread_self());
        pthread_mutex_unlock(&mutex);
        sem_post(&csem);
    }

    return NULL;
}

void * customer(void * arg) {

    while(1) {
        sem_wait(&csem);
        pthread_mutex_lock(&mutex);
        // 保存头结点的指针
        struct Node * tmp = head;
        head = head->next;
        printf("del node, num : %d, tid : %ld\n", tmp->num, pthread_self());
        free(tmp);
        pthread_mutex_unlock(&mutex);
        sem_post(&psem);

    }
    return  NULL;
}

int main() {

    pthread_mutex_init(&mutex, NULL);
    sem_init(&psem, 0, 8);
    sem_init(&csem, 0, 0);

    // 创建5个生产者线程，和5个消费者线程
    pthread_t ptids[5], ctids[5];

    for(int i = 0; i < 5; i++) {
        pthread_create(&ptids[i], NULL, producer, NULL);
        pthread_create(&ctids[i], NULL, customer, NULL);
    }

    for(int i = 0; i < 5; i++) {
        pthread_detach(ptids[i]);
        pthread_detach(ctids[i]);
    }

    while(1) {
        sleep(10);
    }

    pthread_mutex_destroy(&mutex);

    pthread_exit(NULL);

    return 0;
}
```

