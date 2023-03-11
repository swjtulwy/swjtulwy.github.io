# C++中的单例模式


<!--more-->

## 什么是单例模式(Singleton Pattern)？

单例模式又称为单件模式，是一种”对象性能“模式。面向对象很好地解决了”抽象“的问题，但是必不可免地要付出一定代价。通常情况下，面向对象的成本大都可以忽略不计。但是某些情况下，面向对象所带来的成本必须谨慎处理。

单例模式是使用最广泛的设计模式之一。其意图是保证一个类仅有一个实例，并提供一个访问它的全局访问点，该实例被所有程序模块共享。

定义一个单例类值得关注的几点如下：

1. 私有化构造函数（包括拷贝构造、赋值等），以防止外界创建单例类的对象。

2. 使用类的私有静态指针变量指向类的唯一实例。

3. 使用一个公有的静态方法获取该实例

<img title="" src="https://article.biliimg.com/bfs/article/55500165e3216432a1f0b47b091049f7448cb599.png" alt="" width="290" data-align="center">

## 懒汉模式

### 教学版本

单例实例在第一次被使用时才进行初始化，这叫做延迟初始化

```cpp
// version 0.1
class Singleton {
public:
    static Singleton* getInstance();
private:
    static Singleton* m_instance;
    Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};
// 类外初始化静态成员变量
Singleton* Singleton::m_instance = nullptr;
Singleton* Singleton::getInstance() {
    if (m_instance == nullptr) {
        m_instance = new Singleton();
    }
    return m_instance;
}
```

这种方式，看上去很好理解。但是其存在一些潜在的问题：

1. 内存泄漏

2. 非线程安全

上述代码的内存泄漏场景主要是在程序运行结束时静态的成员指针指向的内存没有释放。针对这种情况，可以有两种解决方案，**一种是使用智能指针，另一种是使用静态的嵌套类对象来辅助析构**。

### 解决内存泄漏的方案

使用智能指针的代码如下：智能指针在引用计数为0时候，会自动析构，智能指针本身是类，带有析构函数。

```cpp
// 智能指针解决内存泄漏方案
// version 0.2.1
class Singleton {
public:
    static std::shared_ptr<Singleton> getInstance();
private:
    static std::shared_ptr<Singleton> m_instance;
    Singleton() {}
//    ~Singleton() {}  // 智能指针方式析构函数不能私有
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

std::shared_ptr<Singleton> Singleton::m_instance = nullptr;
std::shared_ptr<Singleton> Singleton::getInstance() {
    if (m_instance == nullptr) {
        // make_shared要求构造函数要public，所以采用这种方式
        m_instance = std::shared_ptr<Singleton>(new Singleton());
    }
    return m_instance;
}
```

采用静态嵌套类对象的解决办法如下:

```cpp
// version 0.2.2
class Singleton {
public:
    static Singleton* getInstance();
private:
    class Deletor {
    public:
        ~Deletor() {
            if (Singleton::m_instance != nullptr) {
                delete Singleton::m_instance;
            }
        }
    };
    static Deletor deltetor;
private:
    static Singleton* m_instance;
    Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};
// 类外初始化静态成员变量
Singleton* Singleton::m_instance = nullptr;
Singleton* Singleton::getInstance() {
    if (m_instance == nullptr) {
        m_instance = new Singleton();
    }
    return m_instance;
}
```

这种方式在程序运行结束时，系统会调用静态成员`deletor`的析构函数，该析构函数会删除单例的唯一实例。使用这种方法释放单例对象有以下特征：

- 在单例类内部定义专有的嵌套类。
- 在单例类内定义私有的专门用于释放的静态成员。
- 利用程序在结束时析构全局变量的特性，选择最终的释放时机。

总的来说，还是很麻烦

### 解决线程安全问题

上述代码在单线程下是正确无误的，但在多线程下会出现竞态条件，为了避免出现这种情况，我们可以采取加锁来解决这一问题

```cpp
// version 0.3.1
class Singleton {
public:
    static std::shared_ptr<Singleton> getInstance();
private:
    static std::shared_ptr<Singleton> m_instance;
    static std::mutex m_mutex;
    Singleton() {}
//    ~Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

std::shared_ptr<Singleton> Singleton::m_instance = nullptr;
std::mutex Singleton::m_mutex;
std::shared_ptr<Singleton> Singleton::getInstance() {
    std::lock_guard<std::mutex> lock{m_mutex};
    if (m_instance == nullptr) {
        m_instance = std::shared_ptr<Singleton>(new Singleton());
    }
    return m_instance;
}nce;
}
```

上述代码解决了线程安全的问题，但是上述情况下锁的代价太高，每次不管存不存在实例都要加锁，可以引入DCL来解决这一问题，即采用双段锁校验的方式：

```cpp
// version 0.3.2
class Singleton {
public:
    static std::shared_ptr<Singleton> getInstance();
    void showMessage() {
        std::cout << m_instance << std::endl;
    }
private:
    static std::shared_ptr<Singleton> m_instance;
    static std::mutex m_mutex;
    Singleton() {}
//    ~Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

std::shared_ptr<Singleton> Singleton::m_instance = nullptr;
std::mutex Singleton::m_mutex;
std::shared_ptr<Singleton> Singleton::getInstance() {
    if (m_instance == nullptr) {
        std::lock_guard<std::mutex> lock{m_mutex};
        if (m_instance == nullptr) {
            m_instance = std::shared_ptr<Singleton>(new Singleton());
        }
    }
    return m_instance;
}
```

这种情况下, 还会存在一个 隐性的问题，在使用new 分配内存的过程种，包括三个过程，申请内存，构造对象，返回对象地址。这三个过程也许会被编译器给优化，从而导致执行的顺序发生变化，在多线程场景下，如果先返回了地址而构造对象没有完成，另一个线程就可能获得一个未构造的对象，从而进行了错误的后续操作。这种情况下，我们可以使用以下方式解决，主要借助`atomic_thread_fence` 构建内存屏障：

```cpp
// 解决线程安全问题, DCL double check lock 加上 内存屏障,避免指令乱序
// version 0.3.3
class Singleton {
public:
    static Singleton* getInstance();
private:
    class Deletor {
        ~Deletor() {
            if (Singleton::m_instance != nullptr) {
                delete m_instance;
            }
        }
    };
    static Deletor deletor;
private:
    static std::atomic<Singleton*> m_instance; 
    static std::mutex m_mutex;
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

std::atomic<Singleton*> Singleton::m_instance;
std::mutex Singleton::m_mutex;

Singleton* Singleton::getInstance() {
    // 使用tmp优化性能
    Singleton* tmp = m_instance.load(std::memory_order_relaxed);
    // 获取内存fence
    std::atomic_thread_fence(std::memory_order_acquire); 
    if (tmp == nullptr) {
        std::lock_guard<std::mutex> lock{m_mutex};
        tmp = m_instance.load(std::memory_order_relaxed);
        if (tmp == nullptr) {
            tmp = new Singleton();
            // 释放内存fence
            std::atomic_thread_fence(std::memory_order_release); 
            m_instance.store(tmp, std::memory_order_relaxed);
        }
    }
    return tmp;
}
```

### 最佳实践

C++11规定了local static在多线程条件下的初始化行为，要求编译器保证了内部静态变量的线程安全性。在C++11标准下，《Effective C++》提出了一种更优雅的单例模式实现，使用函数内的 local static 对象。这样，只有当第一次访问`getInstance()`方法时才创建实例。这种方法也被称为`Meyers' Singleton`。C++0x之后该实现是线程安全的，C++0x之前仍需加锁。

```cpp
// version 0.3.4
class Singleton {
public:
    static Singleton& getInstance();
private:
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

Singleton& Singleton::getInstance() {
    static Singleton instance;
    return instance;
}
```

## 饿汉模式

饿汉模式下，静态实例在定义时就创建了，因此不存在线程安全的问题。

```cpp
// version 1.1

class Singleton {
public:
    static Singleton& getInstance();
private:
    static Singleton m_instance;
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

Singleton Singleton::m_instance;  // 默认初始化
Singleton& Singleton::getInstance() {
    return m_instance;
}
```

由于在`main`函数之前初始化，所以没有线程安全的问题。但是潜在问题在于`no-local static`对象（函数外的`static`对象）在不同编译单元中的初始化顺序是未定义的。也即，`static Singleton instance;`和`static Singleton& getInstance()`二者的初始化顺序不确定，如果在初始化完成之前调用 `getInstance()` 方法会返回一个未定义的实例。

## 总结

- Eager Singleton 虽然是线程安全的，但存在潜在问题；
- Lazy Singleton通常需要加锁来保证线程安全，但局部静态变量版本在C++11后是线程安全的；
- 局部静态变量版本（Meyers Singleton）最优雅。

## 所有代码及简单测试

```cpp
#include <bits/stdc++.h>

//#define LAZY_01 1
//#define LAZY_021 1
//#define LAZY_022 1
//#define LAZY_031 1
//#define LAZY_032 1
//#define LAZY_033 1
#define LAZY_034 1
//#define HUNGERY_01 1


#if LAZY_01
// version 0.1
class Singleton {
public:
    static Singleton* getInstance();
    void showMessage() {
        std::cout << m_instance << std::endl;
    }
private:
    static Singleton* m_instance;
    Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};
// 类外初始化静态成员变量
Singleton* Singleton::m_instance = nullptr;
Singleton* Singleton::getInstance() {
    if (m_instance == nullptr) {
        m_instance = new Singleton();
    }
    return m_instance;
}

#elif LAZY_021
// 智能指针解决内存泄漏方案
// version 0.2.1
class Singleton {
public:
    static std::shared_ptr<Singleton> getInstance();
    void showMessage() {
        std::cout << m_instance.get() << std::endl;
    }
private:
    static std::shared_ptr<Singleton> m_instance;
    Singleton() {}
//    ~Singleton() {}  // 智能指针方式析构函数不能私有
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

std::shared_ptr<Singleton> Singleton::m_instance = nullptr;
std::shared_ptr<Singleton> Singleton::getInstance() {
    if (m_instance == nullptr) {
        // make_shared要求构造函数要public，所以采用这种方式
        m_instance = std::shared_ptr<Singleton>(new Singleton());
    }
    return m_instance;
}

#elif LAZY_022
// 静态嵌套类解决内存泄漏方案
// version 0.2.2
class Singleton {
public:
    static Singleton* getInstance();
    void showMessage() {
        std::cout << m_instance << std::endl;
    }
private:
    class Deletor {
    public:
        ~Deletor() {
            if (Singleton::m_instance != nullptr) {
                delete Singleton::m_instance;
            }
        }
    };
    static Deletor m_deltetor;
private:
    static Singleton* m_instance;
    Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};
// 类外初始化静态成员变量
Singleton* Singleton::m_instance = nullptr;
Singleton* Singleton::getInstance() {
    if (m_instance == nullptr) {
        m_instance = new Singleton();
    }
    return m_instance;
}


#elif LAZY_031
// 解决线程安全问题
// version 0.3.1
class Singleton {
public:
    static std::shared_ptr<Singleton> getInstance();
    void showMessage() {
        std::cout << m_instance.get() << std::endl;
    }
private:
    static std::shared_ptr<Singleton> m_instance;
    static std::mutex m_mutex;
    Singleton() {}
//    ~Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

std::shared_ptr<Singleton> Singleton::m_instance = nullptr;
std::mutex Singleton::m_mutex;
std::shared_ptr<Singleton> Singleton::getInstance() {
    std::lock_guard<std::mutex> lock{m_mutex};
    if (m_instance == nullptr) {
        m_instance = std::shared_ptr<Singleton>(new Singleton());
    }
    return m_instance;
}

#elif LAZY_032
// 解决线程安全问题, DCL double check lock
// version 0.3.2
class Singleton {
public:
    static std::shared_ptr<Singleton> getInstance();
    void showMessage() {
        std::cout << m_instance.get() << std::endl;
    }
private:
    static std::shared_ptr<Singleton> m_instance;
    static std::mutex m_mutex;
    Singleton() {}
//    ~Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

std::shared_ptr<Singleton> Singleton::m_instance = nullptr;
std::mutex Singleton::m_mutex;
std::shared_ptr<Singleton> Singleton::getInstance() {
    if (m_instance == nullptr) {
        std::lock_guard<std::mutex> lock{m_mutex};
        if (m_instance == nullptr) {
            m_instance = std::shared_ptr<Singleton>(new Singleton());
        }
    }
    return m_instance;
}
#elif LAZY_033
// 解决线程安全问题, DCL double check lock 加上 内存屏障,避免指令乱序
// version 0.3.3
class Singleton {
public:
    static Singleton* getInstance();
    void showMessage() {
        std::cout << m_instance << std::endl;
    }

private:
    class Deletor {
        ~Deletor() {
            if (Singleton::m_instance != nullptr) {
                delete m_instance;
            }
        }
    };
    static Deletor deletor;
private:
    static std::atomic<Singleton*> m_instance;
    static std::mutex m_mutex;
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

std::atomic<Singleton*> Singleton::m_instance;
std::mutex Singleton::m_mutex;
Singleton* Singleton::getInstance() {
    Singleton* tmp = m_instance.load(std::memory_order_relaxed);
    std::atomic_thread_fence(std::memory_order_acquire); // 获取内存fence
    if (tmp == nullptr) {
        std::lock_guard<std::mutex> lock{m_mutex};
        tmp = m_instance.load(std::memory_order_relaxed);
        if (tmp == nullptr) {
            tmp = new Singleton();
            std::atomic_thread_fence(std::memory_order_release); // 释放内存fence
            m_instance.store(tmp, std::memory_order_relaxed);
        }
    }
    return tmp;
}

#elif LAZY_034
// version 0.3.4
class Singleton {
public:
    static Singleton& getInstance();
    void showMessage() {
        std::cout << this << std::endl;
    }
private:
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

Singleton& Singleton::getInstance() {
    static Singleton instance;
    return instance;
}

#elif HUNGERY_01
// version 1.1
class Singleton {
public:
    static Singleton& getInstance();
    void showMessage() {
        std::cout << this << std::endl;
    }
private:
    static Singleton m_instance;
    Singleton() {}
    ~Singleton() {}
    Singleton(const Singleton& obj);
    Singleton& operator=(const Singleton&);
};

Singleton Singleton::m_instance;  // 默认初始化
Singleton& Singleton::getInstance() {
    return m_instance;
}
#endif


int main() {

#if LAZY_01 || LAZY_022 || LAZY_033
    Singleton* a = Singleton::getInstance();
    Singleton* b = Singleton::getInstance();
#elif LAZY_021 || LAZY_031 || LAZY_032
    std::shared_ptr<Singleton> a = Singleton::getInstance();
    std::shared_ptr<Singleton> b = Singleton::getInstance();
#elif LAZY_034 || HUNGERY_01
    Singleton* a = &Singleton::getInstance();
    Singleton* b = &Singleton::getInstance();
#endif
    a->showMessage();
    b->showMessage();
    return 0;
}
```

