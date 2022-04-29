# 构造与析构


<!--more-->

## 深拷贝与浅拷贝

<img src="https://i0.hdslb.com/bfs/album/571b0a8c704f97aefe803b72937d0dcefe1b6383.webp" title="" alt="" data-align="center">

基本数据类型的特点：直接存储在栈(stack)中的数据  

引用数据类型的特点：存储的是该对象在栈中引用，真实的数据存放在堆内存里

引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。

在C++中：

1  在未定义显示拷贝构造函数的情况下，系统会调用**默认的拷贝函数**——即**浅拷贝**，它能够完成成员的一一复制。当数据成员中**没有指针**时，浅拷贝是可行的；但当数据成员中有指针时，如果采用简单的浅拷贝，则两类中的两个指针将指向同一个地址，当对象快结束时，会调用**两次析构函数**，而导致**指针悬挂**现象，所以，此时，**必须**采用深拷贝。

2 深拷贝与浅拷贝的区别就在于深拷贝会在堆内存中**另外申请空间**来储存数据，从而也就解决了指针悬挂的问题。简而言之，当数据成员中有指针时，**必须要用深拷贝**。

## 构造函数

首先明确以下一个事实：

```cpp
Person p = Person(); // 这是定义，不会进入赋值运算符的成员函数
p = Person(); // 这时赋值, 会进入赋值运算符的成员函数
```

**C++的构造函数的作用：初始化类对象的数据成员。**

即类的对象被创建的时候，编译系统对该对象**分配内存空间**，并自动调用构造函数，**完成类成员的初始化**，可以理解为构造函数的作用就是给类的各个非静态成员变量申请内存并赋初值。

当直接调用类的构造函数时，其生成的对象内存分配在**栈**上，例如 `A a = A(); 或 A();`

当使用new运算符生成指向对象的指针时，其生成的对象内存分配在**堆**上，例如`A a = new A();`

注意，对象的成员变量的初始化动作发生在**进入构造函数本体之前**，在构造函数内，都不算是被初始化，而是被赋值

如下：

```cpp
class Student {
public:
    Student(int id, const std::string& name, const std::vector<int>& score) {
        // 在花括号里面的都是赋值不是初始化
        m_Id = id;
        m_Name = name;
        m_Score = score;
    }
private:
    int m_Id;
    std::string m_Name;
    std::vector<int> m_Score;
};
```

下面的方法才是正确的初始化方式

```cpp
class Student {
public:
    Student(int id, const std::string& name, const std::vector<int>& score) :
        m_Id(id), m_Name(name), m_Score(score) {
    }
private:
    int m_Id;
    std::string m_Name;
    std::vector<int> m_Score;
};
```

**构造函数的特点：以类名作为函数名，无返回类型。**

我们常说的构造函数大致分为三种，即一般的构造函数、拷贝构造函数和移动构造函数。

### 一般构造函数

- 无参数构造函数/默认构造函数
  
  - 当调用构造函数不提供参数时，就会调用上述构造函数
  
  - 若没有定义任何构造函数时，编译器会自动创建一个没有参数的构造函数，称为**合成构造函数**

- 带参数的构造函数
  
  - 带参数的构造函数中，新的标准是可以用类中其余的构造函数在本构造函数中初始化部分成员变量，称其为**委托构造函数**

### 拷贝构造函数

当一个构造函数的参数为类的引用时，其就是一个拷贝构造函数。

若我们没有给一个类定义一个拷贝构造函数，编译器会自动给我们生成一个构造函数，称为**合成拷贝构造函数** ，除非我们在该拷贝构造函数的声明中指定为`=delete`。

合成拷贝构造函数执行的拷贝动作**都是浅拷贝**，即只对当前栈上内容进行拷贝，不会对栈中指针指向的内容进行拷贝。

拷贝构造函数的参数必须为本类的引用，否则会引发死循环

考虑以下情况：

```cpp
Person a = Person();
Person b = Person(a);
```

我们直到在函数中传参数的话，若不加引用就是值传递，那么每个实参都会复制一份，在栈中生成临时对象，然后该临时对象作用域在函数内部。

构造函数同理，若其参数不是本对象的引用，那么当调用该构造函数时，需要复制一份实参生成临时对象，而此时实参也是该类的一个对象，那么这个临时对象的构造又命中了拷贝构造函数，等于是套娃了，形成了死循环。所以拷贝构造函数的参数必须为本类的引用，另外最佳实践是，形参加上`cosnt`修饰，因为一般也不会用于修改被用于构造的对象。

> 最佳实践：当本类不包含指针类型的成员变量时，可以使用合成的拷贝构造，即全都是浅拷贝，当本类包含指针类型的成员变量时，应使用自定义的拷贝构造并实现深拷贝。最好给参数加上`const`修饰符



看如下代码：

```cpp
#include <iostream>
#include <vector>

using namespace std;
class Buffer {
private:
    unsigned char *buf;
    int capacity;
    int length;
public:
    explicit Buffer(int capacity) : capacity(capacity), length(0), buf(new unsigned char[capacity]{ 0 }) {

    }

    int GetLength() {
        return length;
    }

    int GetCapacity() {
        return capacity;
    }

    bool write(unsigned char value) {
        if (length == capacity) return false;
        buf[length++] = value;
        return true;
    }

    ~Buffer() {
        delete[] buf;
    }
    friend ostream& operator<<(ostream &os, Buffer &buffer);
};

ostream& operator<<(ostream &os, Buffer &buffer) {
    os << "Buffer(" << buffer.length << "/" << buffer.capacity << ")[";
    for (int i = 0; i < buffer.capacity; ++i) {
        os << (int)buffer.buf[i] << ", ";
    }
    os << "]";
    return os;
}

int main() {
    auto buffer = Buffer(10);
    auto buffer2 = buffer;
    buffer.write(65);
    cout << buffer << endl;
    cout << buffer2 << endl;
    return 0;
}
```

```bssh
Buffer(1/10)[65, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
Buffer(0/10)[65, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
进程已结束,退出代码-1073740940 (0xC0000374)
```

看到没，由于没有定义拷贝构造函数，所以`buffer2`没有写却出现了65这个值。另外进程突出代码为一个负数，说明出现了异常，是因为，进行了两次析构。

在上述代码中加入深复制操作后，就有了正确的结果了，如下：

```cpp
Buffer(const Buffer& buffer) {
    if (&buffer == this) return; // 判断是否自赋值，避免后续操作开销
    this->capacity = buffer.capacity;
    this->length = buffer.length;
    this->buf = new unsigned char[buffer.capacity];
    std::copy(buffer.buf, buffer.buf + buffer.capacity, this->buf);
}
```

```bash
Buffer(1/10)[65, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
Buffer(0/10)[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
进程已结束,退出代码0
```

此外赋值运算符重载函数 的功能和拷贝构造函数类似，

> 最佳实践：当定义了拷贝构造函数时，也应一起定义赋值运算符重载函数

```cpp
Buffer& operator=(const Buffer& buffer) {
    if (&buffer == this) return *this;
    this->capacity = buffer.capacity;
    this->length = buffer.length;
    delete[] this->buf;
    this->buf = new unsigned char[buffer.capacity];
    std::copy(buf, buf + buffer.capacity, this->buf);
}
```

注意的是这里开头也要做自赋值判断，目的是为了避免多余的开销

### 移动构造函数

在拷贝构造函数中，若有如下代码

```cpp
auto buffer = Buffer(10); // 第一次构造
buffer = Buffer(16); // 这里构造了一个临时对象， 并将临时对象拷贝进行了一次拷贝
cout << buffer << endl;
return 0;
```

执行结果为

```bash
Constructor: Buffer(0/10)[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
Constructor: Buffer(0/16)[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
Copy: Buffer(0/10)[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
Destructor: Buffer(0/16)[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
Buffer(0/16)[0, 25, 151, 226, 40, 2, 0, 0, 80, 1, 151, 226, 40, 2, 0, 0, ]
Destructor: Buffer(0/16)[0, 25, 151, 226, 40, 2, 0, 0, 80, 1, 151, 226, 40, 2, 0, 0, ]
```

可以看到首先构造了一个`buffer`，调用了第一次构造函数，然后在赋值运算符的右侧，构造了一个临时对象，然后将这临时对象进行了一次拷贝赋值。然后临时对象销毁，接着打印`buffer`,最后`buffer`销毁

为了解决这种不必要的拷贝，C++11引入了一个新的概念也就是移动语义，同时带来了移动构造函数和移动赋值运算符

由此，我们加入以下代码

```cpp
Buffer(Buffer&& buffer) noexcept{
    cout << "Move Constructor: " << *this << endl;
    // 你的归我了
    this->capacity = buffer.capacity;
    this->length = buffer.length;
    this->buf = buffer.buf;
    // 你的删除
    buffer.capacity = 0;
    buffer.length = 0;
    buffer.buf = nullptr;
}

Buffer& operator=(Buffer&& buffer) noexcept{
    cout << "Move: " << *this << endl;
    if (&buffer == this) return *this;
    this->capacity = buffer.capacity;
    this->length = buffer.length;
    delete[] this->buf;
    this->buf = buffer.buf;

    buffer.capacity = 0;
    buffer.length = 0;
    buffer.buf = nullptr;
}
```

> 最佳实践，一要加上noexcept保证不发生异常，二移动赋值时要进行自赋值判断，三被移动的对象的指针要赋值为空指针

结果如下，可以看到Copy操作变为了Move操作

```cpp
Constructor: Buffer(0/10)[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
Constructor: Buffer(0/16)[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
Move: Buffer(0/10)[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
Destructor: Buffer(0/0)[]
Buffer(0/16)[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
Destructor: Buffer(0/16)[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ]
```

当加入如下代码或者在诸如`vector`的一些支持移动操作的标准库容器中，也会使用到移动构造函数

```cpp
auto buffer1 = std::move(buffer);
```

结果：

```cpp
Move Constructor: Buffer(2/0)[]
Destructor: Buffer(0/0)[]
```

#### 最佳实践

为了减少重复代码，可以在拷贝构造函数中调用拷贝赋值函数，移动构造函数同理

```cpp
// 初始化，指针要为空，因为赋值操作中有 先delete[] 操作。
Buffer(const Buffer& buffer) : capacity(0), length(0), buf(nullptr){
    cout << "Copy Constructor: " << *this << endl;
    *this = buffer;
}
// 要将buffer左值转为右值，这里buffer具名，是一个左值
Buffer(Buffer&& buffer) noexcept : capacity(0), length(0), buf(nullptr){
    *this = std::move(buffer);
}
```

更进一步，拷贝赋值和移动赋值还是有些重复代码，无非就是把一个对象的值转移到另一对象，为了解决这一问题，出现一个新的编程范式：Copy & Swap

首先在类中定义一个交换函数：

```cpp
static void Swap(Buffer& lhs, Buffer& rhs) noexcept{
    std::swap(lhs.buf, rhs.buf);
    std::swap(lhs.capacity, rhs.capacity);
    std::swap(lhs.length, rhs.length);
}
```

接着改进拷贝构造和拷贝赋值，其中，拷贝构造只需将将其他的值复制给自己的初始化列表即可，而拷贝赋值则是将形参由引用变成值传递，所以直接和临时对象交换即可，保证了异常安全

```cpp
// 再改进版
Buffer(Buffer& buffer) : capacity(buffer.capacity),
                        length(buffer.length),
                        buf(buffer.capacity ? new unsigned char[buffer.capacity] : nullptr) {
    cout << "Copy Constructor: " << *this << endl;
}

// 改进版 Copy & Swap, 异常安全，因为传进来的是拷贝的临时对象，
Buffer& operator=(Buffer buffer) {
    cout << "Copy & Swap: " << *this << endl;
    Swap(*this, buffer);
    return *this;
} 
```

接着改进移动构造和移动赋值，由于移动构造传进来的是一个`xvalue`,将亡值，所以可以毫不犹豫交换，自己可以初始化为0先然后调用交换函数即可

```cpp
Buffer(Buffer&& buffer) noexcept : Buffer(buffer.capacity){
    Swap(*this, buffer);
}
其实这时已经不需要移动赋值了，将亡值可以和拷贝赋值一起操作
```

## 析构函数

析构函数用于在销毁一个对象动作完成前执行一些操作

**当一个类具有子类时，或者一个类有可能被继承时，应该将其析构函数设置为虚析构函数。**

这时因为，当一个类中有指针成员时，在多态的场景中，例如有一个指向父类类型的指针被赋值了一个子类对象的地址时，若析构函数不是虚函数，那么该对象析构时调用的是父类的析构函数而不是子类对象的析构函数，那么会导致子类的一些特有成员变量未被销毁，从而造成**内存泄漏**。

