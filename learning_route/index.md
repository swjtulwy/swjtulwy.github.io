# 学习路线（持续更新）


<!--more-->

## C++语言

### C++基础语法

- [x] 头文件以及宏定义

- [x] const 与 constexpr

- [x] auto 与 decltype

- [x] using 与 命名空间

- [x] enum 枚举类

- [x] 分离式编译

- [x] 指针与引用

- [x] C++结构体与C结构体

- [x] 传值方式

- [x] 静态变量static

- [x] volatile关键字

- [x] 隐式类型转换与 explicit 关键字 

- [x] 类型转换 static_cast<>, dynamic_cast<>

- [x] Lambda表达式

### 面向对象

- [x]  面向对象理解：封装、继承、多态

- [x] struct 与 class区别

- [ ] 构造函数
  
  - [x] 初始化列表：顺序
  
  - [x] 默认构造函数
  
  - [x] 拷贝构造函数
  
  - [x] 移动构造函数

- [ ] 重载和重写
  
  - [x] 运算符重载
  
  - [x] 拷贝赋值运算符

- [ ] 多态与虚函数
  
  - [x] 多态：编译器多态（模板），运行时多态（重写）

### C++标准库

#### IO库

- [x] IO类： iostream

- [x] 文件输入输出： fstream

- [x] string流：sstream

#### 标准模板库（STL）

- [ ] 序列容器(vector,list,deque)
  
  - [x] vector: 扩容
  
  - [x] list: 双向列表
  
  - [x] deque: 多node组合

- [ ] 关联容器
  
  - [x] (multi) unordered_map/set : Hash Table + 链表/红黑树
  
  - [x] (multi) map/set : 红黑树，有序

- [ ] STL6大组成部分
  
  - [x] 容器(Container)、算法(Algorithm)、迭代器(Iterator)
  
  - [x] 适配器(Adapter)、分配器(Alloctor)、仿函数(Functor)

#### string类

### 动态内存与智能指针

- [x] shared_ptr

- [x] unique_ptr

- [x] new 与 delete

- [x] 右值引用 (&&) 与 对象移动(std::move)

### 异常处理

- [x] 抛出异常

- [x] 捕获异常

- [x] 异常类层次

## 数据结构与算法

### 数据结构

- [x] 链表
  
  - [x] 单向链表
  
  - [x] 双向链表

- [x] 队列
  
  - [ ] 环形队列

- [x] 栈
  
  - [ ] 双栈

- [ ] 堆
  
  - [ ] 建堆、调整堆
  
  - [ ] 大顶堆、小顶堆

- [ ] 二叉树
  
  - [x] 完全二叉树
  
  - [x] 先序遍历、中序遍历、后序遍历
  
  - [x] 二分查找树BST
  
  - [ ] 红黑树
  
  - [ ] b树、b+树

- [ ] 图
  
  - [ ] 有向图、无向图
  
  - [ ] 邻接表、邻接矩阵、稀疏矩阵

- [x] 散列表（Hash Table）

- [ ] 跳表

- [ ] 布隆过滤器

### 算法

- [x] 排序: 快排、归并、堆排序 、希尔  

- [x] 分治法、减治法，二分查找

- [ ] 图算法：dijkstra算法，最小生成树Kruskal

- [x] DFS，BFS

- [x] 贪心算法

- [ ] 动态规划

## 操作系统

### 基本概念

- [x] 用户态/内核态

- [x] 系统调用  

### 内存管理

- [ ] 物理内存/虚拟内存

- [ ] 虚拟地址映射

- [ ] 分页，页表

- [ ] 页面置换算法

- [ ] 缺页中断

- [ ] 内存池

### 进程与线程

- [x] 进程
  
  - [x] 孤儿进程
  
  - [x] 僵尸进程
  
  - [x] 守护进程

- [ ] 线程

- [ ] IPC
  
  - [x] 管道、命名管道
  
  - [x] 信号Signal
  
  - [x] 信号量Semaphore
  
  - [x] Socket
  
  - [x] 共享内存shared memory
  
  - [x] 内存映射mman

### 死锁

- [ ] 互斥
  
  - [x] 互斥锁
  
  - [x] 读写锁

- [ ] 同步
  
  - [x] 信号量
  
  - [x] 条件变量+互斥锁

### 文件系统

- [ ] 磁盘文件系统

- [ ] 虚拟文件系统

- [ ] 文件系统实现原理

### IO

### Linux系统实操

- [ ] shell / vi的使用

- [ ] Linux系统性能监控参数 ps / netstat / df

## 数据库

- 关系型数据库(MySQL)
  
  - [ ] MySQ安装与配置
  
  - [ ] sql建表，索引，存储过程
  
  - [ ] 存储引擎，myissam/innodb
  
  - [ ] 数据库连接池
  
  - [ ] 异步数据库请求
  
  - [ ] 数据库集群， 分库分表，读写分离

- NoSQL(Redis)
  
  - [ ] Redis编译安全，配置
  
  - [ ] Redis命令使用
  
  - [ ] Redis连接池/异步Redis的做法
  
  - [ ] Redis集群，数据备份
  
  - [ ] 缓存雪崩，缓存击穿

## 网络原理

- [x] 网络体系模型

- [x] TCP三次握手，四次挥手，滑动窗口，状态机

- [x] UDP的原理

- [ ] http/https/http2.0/http3.0

- [ ] Session Cookie application

- [ ] 网络安全，加密，数字签名

- [ ] wireshark, tcpdump

- [ ] iperf

## 网络编程

- [ ] ping, telnet, ifconfig 使用

- [x] socket编程， tcp/udp

- [ ] 网络IO模型，阻塞/非阻塞，同步异步

- [x] io多路复用 select/poll/epoll

- [x] epoll reactor, proactor

- [ ] time_wait/close_wait大量

- [ ] C10K/C1000K/C10M

- [ ] 网络框架 libevent/libv 协程 ntyco, libco

## 设计模式

- 设计原则

- 创建型模式
  
  - [ ] 单例模式
  
  - [ ] 工厂模式

- 结构型模式
  
  - [ ] 适配器模式
  
  - [ ] 代理模式
  
  - [ ] 组合模式
  
  - [ ] 装饰模式

- 行为型模式
  
  - [ ] 策略模式
  
  - [ ] 观察者模式
  
  - [ ] 迭代器模式
  
  - [ ] 模板方法模式
  
  - [ ] 命令模式

## 并行与分布式系统

   

