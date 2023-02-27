# C++智能指针


<!--more-->

## C++智能指针原理

智能指针在C++11种被真正引入标准库，其实现依赖于以下原理：

### 面向对象封装

将指针包装为类，获取资源在构造函数种完成，释放资源在析构函数种完成，保证了资源获取即初始化（**RAII**）。在离开作用域时，智能指针对象的析构函数自动调用释放资源，无需手动释放。

### 引用计数

维护一个计数器用于追踪指向资源的被引用数，当资源被引用时，计数加一，资源被解引用时，计数减一。只在`shared_ptr`里使用

### 操作符重载

将指针的引用与解引用运算符重载，使得智能指针表现的行为与普通指针一致。`weak_ptr`没有重载，不能直接使用

## 为什么需要智能指针？

智能指针主要可以解决以下场景下的内存管理问题

### 内存泄漏

C++在堆上申请内存后，需要手动对内存进行释放。代码的初创者可能会注意内存的释放，但随着代码协作者加入，或者随着代码日趋复杂，很难保证内存都被正确释放。

尤其是一些代码分支在开发中没有被完全测试覆盖的时候，就算是内存泄漏检查工具也不一定能检查到内存泄漏。

```cpp
void test_memory_leak(bool open) {
    A *a = new A();

    if(open) {
        // 代码变复杂过程中，很可能漏了 delete(a);
        return;
    }

    delete(a);
    return;
}
```

### 多线程下对象的析构问题

多线程遇上对象析构，是一个很难的问题，稍有不慎就会导致程序崩溃。因此在对于 C++开发者而言，经常会使用**静态单例**来使得对象常驻内存，避免析构带来的问题。这势必会造成内存泄露，当单例对象比较大，或者程序对内存非常敏感的时候，就必须面对这个问题了。

先以一个常见的 C++多线程问题为例，介绍多线程下的对象析构问题。

比如我们在开发过程中，经常会在一个 Class 中创建一个线程，这个线程读取外部对象的成员变量。

```cpp
// 日志上报Class
class ReportClass {
private:
    ReportClass() {}
    ReportClass(const ReportClass&) = delete;
    ReportClass& operator=(const ReportClass&) = delete;
    ReportClass(const ReportClass&&) = delete;
    ReportClass& operator=(const ReportClass&&) = delete;

private:
    std::mutex mutex_;
    int count_ = 0;
    void addWorkThread();

public:
    void pushEvent(std::string event);

private:
    static void workThread(ReportClass *report);

private:
    static ReportClass* instance_;
    static std::mutex static_mutex_;

public:
    static ReportClass* GetInstance();
    static void ReleaseInstance();
};

std::mutex ReportClass::static_mutex_;
ReportClass* ReportClass::instance_;

ReportClass* ReportClass::GetInstance() {
    // 单例简单实现，非本文重点
    std::lock_guard<std::mutex> lock(static_mutex_);
    if (instance_ == nullptr) {
        instance_ = new ReportClass();
        instance_->addWorkThread();
    }
    return instance_;
}

void ReportClass::ReleaseInstance() {
    std::lock_guard<std::mutex> lock(static_mutex_);
    if(instance_ != nullptr) {
        delete instance_;
        instance_ = nullptr;
    }
}

// 轮询上报线程
void ReportClass::workThread(ReportClass *report) {
    while(true) {
        // 线程运行过程中，report可能已经被销毁了
        std::unique_lock<std::mutex> lock(report->mutex_);
        if(report->count_ > 0) {
            report->count_--;
        }
        usleep(1000*1000);
    }
}

// 创建任务线程
void ReportClass::addWorkThread() {
    std::thread new_thread(workThread, this);
    new_thread.detach();
}

// 外部调用
void ReportClass::pushEvent(std::string event) {
    std::unique_lock<std::mutex> lock(mutex_);
    this->count_++;
}
```

使用 ReportClass 的代码如下：

```cpp
ReportClass::GetInstance()->pushEvent("test");
```

但当这个外部对象（即`ReportClass`）析构时，对象创建的线程还在执行。此时线程引用的对象指针为野指针，程序必然会发生异常。

解决这个问题的思路是在对象析构的时候，对线程进行`join`。

```cpp
// 日志上报Class
class ReportClass {
private:
    //...
    ~ReportClass();

private:
    //...
    bool stop_ = false;
    std::thread *work_thread_;
    //...
};

// 轮询上报线程
void ReportClass::workThread(ReportClass *report) {
    while(true)
    {
        std::unique_lock<std::mutex> lock(report->mutex_);

        // 如果上报停止，不再轮询上报
        if(report->stop_)
        {
            break;
        }

        if(report->count_ > 0) {
            report->count_--;
        }

        usleep(1000*1000);
    }
}

// 创建任务线程
void ReportClass::addWorkThread() {
    // 保存线程指针，不再使用分离线程
    work_thread_ = new std::thread(workThread, this);
}

ReportClass::~ReportClass() {
    // 通过join来停止内部线程
    stop_ = true;
    work_thread_->join();
    delete work_thread_;
    work_thread_ = nullptr;
}
```

这种方式看起来没问题了，但是由于这个对象一般是被多个线程使用。假如某个线程想要释放这个对象，但另外一个线程还在使用这个对象，可能会出现野指针问题。就算释放对象的线程将对象释放后将指针置为`nullptr`，但仍然可能在多线程下在指针置空前被另外一个线程取得地址并使用。

| 线程A                                           | 线程B                                                                                                                                                      |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ReportClass::GetInstance()->ReleaseInstance() | ReportClass* report = ReportClass::GetInstance();<br/>if (report) {<br/> &emsp;&emsp;&emsp;// 此时切换到线程A<br/>&emsp;&emsp;report->pushEcvent("test");<br/>} |

此种场景下，锁机制已经很难解决这个问题。对于多线程下的对象析构问题，智能指针可谓是神器。接下来我们先对智能指针的基本用法进行说明。

## 智能指针用法

智能指针设计的初衷就是可以帮助我们管理堆上申请的内存，可以理解为开发者只需要申请，而释放交给智能指针。

目前 C++11 主要支持的智能指针为以下几种

- `unique_ptr`
- `shared_ptr`
- `weak_ptr`

包含在头文件`<memory>`中

### unique_ptr

`unique_ptr`“唯一”拥有其所指对象，同一时刻只能有一个`unique_ptr`指向给定对象（通过**禁止拷贝语义、只有移动语义**来实现）。相比与原始指针`unique_ptr`由于其RAII的特性，使得在出现异常的情况下，动态资源能得到释放。`unique_ptr`指针本身的生命周期：从`unique_ptr`指针创建时开始，直到离开作用域。离开作用域时，若其指向对象，则将其所指对象销毁(默认使用`delete`操作符，用户可指定其他操作)。`unique_ptr`指针与其所指对象的关系：在智能指针生命周期内，可以改变智能指针所指对象，如创建智能指针时通过构造函数指定、通过`reset`方法重新指定、通过`release`方法释放所有权、通过移动语义转移所有权。

概括就是以下两点：

1、`unique_ptr`**不能被复制**到另外一个`unique_ptr`  
2、`unique_ptr`所持有的对象**只能通过转移语义将所有权转移**到另外一个`unique_ptr`

如下代码所示：

```cpp
std::unique_ptr<A> a1(new A());
std::unique_ptr<A> a2 = a1;v // 编译报错，不允许复制
std::unique_ptr<A> a3 = std::move(a1); // 可以转移所有权，所有权转义后a1不再拥有任何指针
```

使用场景如下：

```cpp
class A {
public:
    void do_something() {}
};

void test_unique_ptr(bool open) {
    std::unique_ptr<A> a(new A());
    a->do_something();

    if(open) {
        // 不再需要手动释放内存
        return;
    }

    // 不再需要手动释放内存
    return;
}
```

`unique_ptr`本身拥有的方法主要包括：

1、get() 获取其保存的原生指针，尽量不要使用  
2、bool() 判断是否拥有指针  
3、release() 释放所管理指针的所有权，返回原生指针。但并不销毁原生指针。  
4、reset() 释放并销毁原生指针。如果参数为一个新指针，将管理这个新指针

```cpp
std::unique_ptr<A> a1(new A());
A *origin_a = a1.get(); // 尽量不要暴露原生指针
if(a1) {
    // a1 拥有指针
}

std::unique_ptr<A> a2(a1.release()); // 常见用法，转义拥有权
a2.reset(new A()); // 释放并销毁原有对象，持有一个新对象
a2.reset(); // 释放并销毁原有对象，等同于下面的写法
a2 = nullptr; // 释放并销毁原有对象
```

### shared_ptr

`shared_ptr`多个指针指向相同的对象。`shared_ptr`使用引用计数，每一个`shared_ptr`的拷贝都指向相同的内存。每使用一次，内部的引用计数加1，每析构一次，内部的引用计数减1，减为0时，自动删除所指向的堆内存。`shared_ptr`内部的**引用计数是线程安全的**，**但是对象的读取需要加锁**。

- **初始化**。智能指针是个模板类，可以指定类型，传入指针通过构造函数初始化。也可以使用`make_shared`函数初始化。不能将指针直接赋值给一个智能指针，一个是类，一个是指针。例如`std::shared_ptr<int> p4 = new int(1);`的写法是错误的
- **拷贝和赋值**。拷贝使得对象的引用计数增加1，赋值使得原对象引用计数减1，当计数为0时，自动释放内存。后来指向的对象引用计数加1，指向后来的对象。
- **get函数获取原始指针**
- **注意不要用一个原始指针初始化多个shared_ptr**，否则会造成二次释放同一内存
- **注意避免循环引用**，`shared_ptr`的一个最大的陷阱是循环引用，循环，循环引用会导致堆内存无法正确释放，导致内存泄漏。循环引用在`weak_ptr`中介绍

`shared_ptr`本身拥有的方法主要包括：

1、`get()` 获取其保存的原生指针，尽量不要使用  
2、`bool()` 判断是否拥有指针  
3、`reset()` 释放并销毁原生指针。如果参数为一个新指针，将管理这个新指针  
4、`unique() `如果引用计数为 1，则返回 `true`，否则返回 `false  `
5、`use_count()` 返回引用计数的大小

```cpp
#include <iostream>
#include <memory>

int main() {
    {
        std::unique_ptr<int> uptr(new int(10));  // 绑定动态对象
        //std::unique_ptr<int> uptr2 = uptr;  // 不能赋值
        //std::unique_ptr<int> uptr2(uptr);  // 不能拷贝
        std::unique_ptr<int> uptr2 = std::move(uptr); // 转换所有权
        uptr2.release(); // 释放所有权
    }
    // 超出uptr的作用域，内存释放
}
```

### weak_ptr

`weak_ptr`是为了配合`shared_ptr`而引入的一种智能指针，因为它不具有普通指针的行为，**没有重载**`operator*`和`->`,它的最大作用在于协助`shared_ptr`工作，像旁观者那样观测资源的使用情况。`weak_ptr`可以从一个`shared_ptr`或者另一个`weak_ptr`对象构造，获得资源的观测权。但`weak_ptr`没有共享资源，它的构造**不会引起指针引用计数的增加**。使用`weak_ptr`的成员函数`use_count()`可以观测资源的引用计数，另一个成员函数`expired()`的功能等价于`use_count()==0`,但更快，表示被观测的资源(也就是`shared_ptr`的管理的资源)已经不复存在。**`weak_ptr`可以使用一个非常重要的成员函数`lock()`** 从被观测的`shared_ptr`获得一个可用的shared_ptr对象， 从而操作资源。但当`expired()==true`的时候，`lock()`函数将返回一个存储空指针的`shared_ptr`。

由于`shared_ptr`是通过引用计数来管理原生指针的，那么最大的问题就是**循环引用**（比如 a 对象持有 b 对象，b 对象持有 a 对象），这样必然会导致内存泄露。而`weak_ptr`不会增加引用计数，因此将循环引用的一方修改为弱引用，**可以避免内存泄露**。

`weak_ptr`可以通过一个`shared_ptr`创建。

```
std::shared_ptr<A> a1(new A());
std::weak_ptr<A> weak_a1 = a1; //不增加引用计数
```

`weak_ptr`本身拥有的方法主要包括：

1、`expired()` 判断所指向的原生指针是否被释放，如果被释放了返回 `true`，否则返回 `false`  
2、`use_count()` 返回原生指针的引用计数  
3、`lock()` 返回 `shared_ptr`，如果原生指针没有被释放，则返回一个非空的 `shared_ptr`，否则返回一个空的 `shared_ptr`  
4、`reset()` 将本身置空 

```cpp
std::shared_ptr<A> a1(new A());
std::weak_ptr<A> weak_a1 = a1;//不增加引用计数

if(weak_a1.expired()) {
    // 如果为true，weak_a1对应的原生指针已经被释放了
}

long a1_use_count = weak_a1.use_count(); // 引用计数数量

if(std::shared_ptr<A> shared_a = weak_a1.lock()) {
    // 此时可以通过shared_a进行原生指针的方法调用
}

weak_a1.reset();// 将weak_a1置空
```

## 智能指针最佳实践

### 智能指针如何选择

在介绍指针如何选择之前，我们先回顾一下这几个指针的特点

1、`unique_ptr`独占对象的所有权，由于没有引用计数，因此性能较好  
2、`shared_ptr`共享对象的所有权，但性能略差  
3、`weak_ptr`配合`shared_ptr`，解决循环引用的问题

由于性能问题，那么可以粗暴的理解：优先使用`unique_ptr`。但由于`unique_ptr`不能进行复制，因此部分场景下不能使用的。

#### unique_ptr 的使用场景

`unique_ptr`一般在不需要多个指向同一个对象的指针时使用。但这个条件本身就很难判断，在我看来可以简单的理解：这个对象在对象或方法内部使用时优先使用`unique_ptr`。

##### 1、对象内部使用

```cpp
class TestUnique {
private:
    std::unique_ptr<A> a_ = std::unique_ptr<A>(new A());
public:
    void process1() {
        a_->do_something();
    }

    void process2() {
        a_->do_something();
    }

    ~TestUnique() {
        // 此处不再需要手动删除a_
    }
};
```

##### 2、方法内部使用

```cpp
void test_unique_ptr() {
    std::unique_ptr<A> a(new A());
    a->do_something();
}
```

#### shared_ptr 的使用场景及最佳实践

`shared_ptr`一般在需要多个执行同一个对象的指针使用。在我看来可以简单的理解：这个对象需要被多个 Class 同时使用的时候。

```cpp
class B {
private:
    std::shared_ptr<A> a_;

public:
    B(std::shared_ptr<A>& a): a_(a) {}
};

class C {
private:
    std::shared_ptr<A> a_;

public:
    C(std::shared_ptr<A>& a): a_(a) {}
};

std::shared_ptr<B> b_;
std::shared_ptr<C> c_;

void test_A_B_C() {
    std::shared_ptr<A> a = std::make_shared<A>();
    b_ = std::make_shared<B>(a);
    c_ = std::make_shared<C>(a);
}
```

在上面的代码中需要注意，我们使用`std::make_shared`代替`new`的方式创建`shared_ptr`。

因为使用`new`的方式创建`shared_ptr`会导致出现两次内存申请，而`std::make_shared`在内部实现时只会申请一个内存。因此建议后续均使用`std::make_shared`。

如果`A`想要调用`B`和`C`的方法怎么办呢？可否在`A`中定义`B`和`C`的`shared_ptr`呢？答案是不可以，这样会产生循环引用，导致内存泄露。

此时就需要`weak_ptr`出场了。

```cpp
class A {
private:
    std::weak_ptr<B> b_;
    std::weak_ptr<C> c_;
public:
    void do_something() {}

    void set_B_C(const std::shared_ptr<B>& b, const std::shared_ptr<C>& c) {
        b_ = b;
        c_ = c;
    }
};
```

```cpp
a->set_B_C(b_, c_);
```

如果想要在`A`内部将当前对象的指针共享给其他对象，需要怎么处理呢？

```cpp
class D {
private:
    std::shared_ptr<A> a_;

public:
    std::shared_ptr<A>& a): a_(a) {}
};

class A {
// 上述代码省略

public:
    void new_D() {
        // 错误方式，用this指针重新构造shared_ptr，将导致二次释放当前对象
        std::shared_ptr<A> this_shared_ptr1(this);
        std::unique_ptr<D> d1(new D(this_shared_ptr1));
    }
};
```

如果采用`this`指针重新构造`shared_ptr`是肯定不行的，因为重新创建的`shared_ptr`与当前对象的`shared_ptr`没有关系，没有增加当前对象的引用计数。这将导致任何一个`shared_ptr`计数为 0 时提前释放了对象，后续操作这个释放的对象都会导致程序异常。

此时就需要引入`shared_from_this`。对象继承了`enable_shared_from_this`后，可以通过`shared_from_this()`获取当前对象的`shared_ptr`指针。

```cpp
class A: public std::enable_shared_from_this<A> {
// 上述代码省略

public:
    void new_D() {
        // 错误方式，用this指针重新构造shared_ptr，将导致二次释放当前对象
        std::shared_ptr<A> this_shared_ptr1(this);
        std::unique_ptr<D> d1(new D(this_shared_ptr1));
        // 正确方式
        std::shared_ptr<A> this_shared_ptr2 = shared_from_this();
        std::unique_ptr<D> d2(new D(this_shared_ptr2));
    }
};
```

### 智能指针的错误使用方法

#### 1、使用智能指针托管的对象，尽量不要再使用原生指针

很多开发同学在最开始使用智能指针的时候，对同一个对象会混用智能指针和原生指针，导致程序异常。

```cpp
void incorrect_smart_pointer1() {
    A*a = new A();
    std::unique_ptr<A> unique_ptr_a(a);

    // 此处将导致对象的二次释放
    delete a;
}
```

#### 2、不要把一个原生指针交给多个智能指针管理

如果将一个原生指针交个多个智能指针，这些智能指针释放对象时会产生对象的多次销毁

```cpp
void incorrect_smart_pointer2() {
    A* a = new A();
    std::unique_ptr<A> unique_ptr_a1(a);
    std::unique_ptr<A> unique_ptr_a2(a); // 此处将导致对象的二次释放
}
```

#### 3、尽量不要使用 get()获取原生指针

```cpp
void incorrect_smart_pointer3() {
    std::shared_ptr<A> shared_ptr_a1 = std::make_shared<A>();

    A *a= shared_ptr_a1.get();

    std::shared_ptr<A> shared_ptr_a2(a); // 此处将导致对象的二次释放

    delete a; // 此处也将导致对象的二次释放
}
```

#### 4、不要将 this 指针直接托管智能指针

```cpp
class E {
    void use_this() {
        //错误方式，用this指针重新构造shared_ptr，将导致二次释放当前对象
        std::shared_ptr<E> this_shared_ptr1(this);
    }
};
```

```cpp
std::shared_ptr<E> e = std::make_shared<E>();
```

#### 5、智能指针只能管理堆对象，不能管理栈上对象

栈上对象本身在出栈时就会被自动销毁，如果将其指针交给智能指针，会造成对象的二次销毁

```cpp
void incorrect_smart_pointer5() {
    int int_num = 3;
    std::unique_ptr<int> int_unique_ptr(&int_num);
}
```

### 解决多线程下对象析构的问题

有了智能指针之后，我们就可以使用智能指针解决多线程下的对象析构问题。

我们使用`shared_ptr`管理`ReportClass`。并将 `weak_ptr`传给子线程，子线程会判断外部的`ReportClass`是否已经被销毁，如果没有被销毁会通过`weak_ptr`换取`shared_ptr`，否则线程退出。解决了外部对象销毁，内部线程使用外部对象的野指针的问题。

```cpp
// 日志上报Class
class ReportClass: public std::enable_shared_from_this<ReportClass> {
    //...

private:
    static void workThread(std::weak_ptr<ReportClass> weak_report_ptr);

private:
    static std::shared_ptr<ReportClass> instance_;
    static std::mutex static_mutex_;

public:
    static std::shared_ptr<ReportClass> GetInstance();
    static void ReleaseInstance();
};

std::mutex ReportClass::static_mutex_;
std::shared_ptr<ReportClass> ReportClass::instance_;

std::shared_ptr<ReportClass> ReportClass::GetInstance() {
    // 单例简单实现，非本文重点
    std::lock_guard<std::mutex> lock(static_mutex_);
    if (!instance_) {
        instance_ = std::shared_ptr<ReportClass>(new ReportClass());
        instance_->addWorkThread();
    }
    return instance_;
}

void ReportClass::ReleaseInstance() {
    std::lock_guard<std::mutex> lock(static_mutex_);
    if(instance_) {
        instance_.reset();
    }
}

// 轮询上报线程
void ReportClass::workThread(std::weak_ptr<ReportClass> weak_report_ptr) {
    while(true) {
        std::shared_ptr<ReportClass> shared_report_ptr = weak_report_ptr.lock();
        if(!shared_report_ptr) {
            return;
        }

        std::unique_lock<std::mutex>(shared_report_ptr->mutex_);

        if(shared_report_ptr->count_ > 0) {
            shared_report_ptr->count_--;
        }

        usleep(1000*1000);
    }
}

// 创建任务线程
void ReportClass::addWorkThread() {
    std::weak_ptr<ReportClass> weak_report_ptr = shared_from_this();
    std::thread work_thread(workThread, weak_report_ptr);
    work_thread.detach();
}

// 外部调用
void ReportClass::pushEvent(std::string event) {
    std::unique_lock<std::mutex> lock(mutex_);
    this->count_++;
}
```

并且在多个线程使用的时候，由于采用`shared_ptr`管理，因此只要有`shared_ptr`持有对象，就不会销毁对象，因此不会出现多个线程使用时对象被析构的情况。只有该对象的所有`shared_ptr`都被销毁的时候，对象的内存才会被释放，保证的对象析构的安全。

## 智能指针设计与实现

　下面是一个简单智能指针的demo。智能指针类将一个计数器与类指向的对象相关联，引用计数跟踪该类有多少个对象共享同一指针。每次创建类的新对象时，初始化指针并将引用计数置为1；当对象作为另一对象的副本而创建时，拷贝构造函数拷贝指针并增加与之相应的引用计数；对一个对象进行赋值时，赋值操作符减少左操作数所指对象的引用计数（如果引用计数为减至0，则删除对象），并增加右操作数所指对象的引用计数；调用析构函数时，构造函数减少引用计数（如果引用计数减至0，则删除基础对象）。智能指针就是模拟指针动作的类。所有的智能指针都会重载 -> 和 * 操作符。智能指针还有许多其他功能，比较有用的是自动销毁。这主要是利用栈对象的有限作用域以及临时对象（有限作用域实现）析构函数释放内存。

```cpp
#include <iostream>
#include <memory>

template<typename T>
class SmartPointer {
private:
    T* _ptr;
    size_t* _count;
public:
    SmartPointer(T* ptr = nullptr) :
            _ptr(ptr) {
        if (_ptr) {
            _count = new size_t(1);
        } else {
            _count = new size_t(0);
        }
    }

    SmartPointer(const SmartPointer& ptr) {
        if (this != &ptr) {
            this->_ptr = ptr._ptr;
            this->_count = ptr._count;
            (*this->_count)++;
        }
    }

    SmartPointer& operator=(const SmartPointer& ptr) {
        if (this->_ptr == ptr._ptr) {
            return *this;
        }

        if (this->_ptr) {
            (*this->_count)--;
            if (this->_count == 0) {
                delete this->_ptr;
                delete this->_count;
            }
        }

        this->_ptr = ptr._ptr;
        this->_count = ptr._count;
        (*this->_count)++;
        return *this;
    }

    T& operator*() {
        assert(this->_ptr == nullptr);
        return *(this->_ptr);

    }

    T* operator->() {
        assert(this->_ptr == nullptr);
        return this->_ptr;
    }

    ~SmartPointer() {
        (*this->_count)--;
        if (*this->_count == 0) {
            delete this->_ptr;
            delete this->_count;
        }
    }

    size_t use_count(){
        return *this->_count;
    }
};

int main() {
    {
        SmartPointer<int> sp(new int(10));
        SmartPointer<int> sp2(sp);
        SmartPointer<int> sp3(new int(20));
        sp2 = sp3;
        std::cout << sp.use_count() << std::endl;
        std::cout << sp3.use_count() << std::endl;
    }
    //delete operator
}
```

## Reference

[C++ 智能指针最佳实践&源码分析 - 极术社区 - 连接开发者与智能计算生态 (aijishu.com)](https://aijishu.com/a/1060000000286819)

[C++11中智能指针的原理、使用、实现 - wxquare - 博客园 (cnblogs.com)](https://www.cnblogs.com/wxquare/p/4759020.html)

[【C++】智能指针的原理和实现_51CTO博客_c++智能指针](https://blog.51cto.com/liangchaoxi/4055018?u_atoken=6835d3d7-157f-40ee-8422-d086a60d9042&u_asession=01GKYAA755dbk0QchYnek7WhaWIRA60HFFz3gOsZm-3lQXBd_bophpGcbvNHPTEKtlX0KNBwm7Lovlpxjd_P_q4JsKWYrT3W_NKPr8w6oU7K86JeL1rXG6FuH3vMq9jSWDfn5mCeVMgxCWEzoJwRNmmWBkFo3NEHBv0PZUm6pbxQU&u_asig=05ieqWdNMTAm9Lsshuh83Rih4OB7BcydSoDwd9bNS8Q4sUJG0Wn9y164YkJCIH6gmRkxL5mhp3jVm8PEu3Zzibfld3nlzxMERFLJ1c_8S_lBIBejGPjJuCETsGu1P5QhSjtsgYRLh4MLB4W011elLw0rth3bpJEBtzHBPJDXi5nGX9JS7q8ZD7Xtz2Ly-b0kmuyAKRFSVJkkdwVUnyHAIJzQB9LMV0T6XLTUHPuigl_R_c-IeOTGaaamlNIqL1CvDjhJzOfaR9Z_iFF5VAMa753O3h9VXwMyh6PgyDIVSG1W9jhKq1iX7CrQjxL9wpXk1u2t06Kw4WXjL8pvgPxmcQymrP3J6vaUnVVXeTmljiLOtgxWCCv0Va-6dWdLZ0McT_mWspDxyAEEo4kbsryBKb9Q&u_aref=RYGJ9FtNdZA9a6A%2ByChQw14oMF8%3D)

