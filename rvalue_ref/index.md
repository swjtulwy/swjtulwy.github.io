# 右值引用与移动语义


<!--more-->

[从4行代码看右值引用 - qicosmos(江南) - 博客园](https://www.cnblogs.com/qicosmos/p/4283455.html)

[Value categories - cppreference.com](https://en.cppreference.com/w/cpp/language/value_category)

[https://www.stroustrup.com/terminology.pdf](https://www.stroustrup.com/terminology.pdf)

## 左值与右值

### 标准定义

每个 C++ 表达式（一个操作符和它的操作数，一个字面值，一个变量名等等）都代表着两个独立属性：类型+属性分类。，属性分类也叫**值类别**。 值类别是编译器在表达式计算期间创建、复制和移动临时对象时必须遵循的规则的基础。

C++17 标准定义表达式值类别，如下所示：

- *glvalue* (general lvalue)： **泛左值，由(lvalue)和(xvalue)构成。**
- *rvalue* 右值，**由(xvalue)和(prvalue)构成。右值具有潜在的可移动性**
- *prvalue*(pure rvaluue) ： **纯右值，即传统意义上的右值**
- xvalue : **中间值？将亡值？，指通过“右值引用”产生的对象。 这里x可以理解为即将消失(expiring)，也可理解为中间（横跨左值和右值）**
- *lvalue* ：**传统意义上的左值**

<img src="https://i0.hdslb.com/bfs/album/5e059d54a34e5b1bc26e4d5026aea8366d529f89.png" title="" alt="" data-align="center">

详细的定义可以参考：[C++ lvalue，prvalue，xvalue，glvalue和rvalue详解（from cppreference） - PhiliAI - 博客园 (cnblogs.com)](https://www.cnblogs.com/Philip-Tell-Truth/p/6370019.html)

### 通俗解释

C/C++ 语言中可以放在赋值符号左边的变量，左值表示存储在计算机内存的对象，左值相当于地址值。右值：当一个符号或者常量放在操作符右边的时候，计算机就读取他们的“右值”，也就是其代表的真实值，右值相当于数据值。

**左值是可以取地址的，这也是区分左值和右值的唯一正确的标志**

左值(lvalue)：可以出现在 = 左边
右值(rvalue)：只能出现在 = 右边
看一些左值和右值的例子，下面是整型的

```cpp
int a = 9;
int b = 4;
// 左值为表达式
(a = 4) += 28; // 左值是一个表达式 a=4，这句话结果为 a=32
// 左值为变量
a = b;        // a 和 b 都是左值
a + b = 42;    // Error, a + b整体只是右值，出现右边，这句话出错
```

看一个关于 string 类型的

**特别注意，C++规定字符串是左值**

```cpp
string s1("Hello");
string s2("World");
s1 + s2 = s2;    // s1 + s2 是右值，但是这里编译成功了，后面看一下
string() = "World"; // 编译通过，左边是临时对象， 竟然也可以赋值
```

在函数中也会用到

```cpp
int foo() { return 5; }  // 函数返回的是一个右值
int x = foo();
int* p = &foo();  //Error，对函数的返回值取地址，但是函数返回值为右值，取地址不行
foo() = 7;        // Error，函数返回值为右值，只能出现在右边
```

> 简单的来说，能取地址的变量一定是左值，有名字的变量也一定是左值，最经典的void fun(p&& shit)，其中shit也是左值，因为右值引用是左值（所以才会有move，forward这些函数的产生，其中move出来一定是右值，forward保持变量形式和之前的不变，就是为了解决右值引用是左值的问题）。
> 
> 至于为什么不能把等号左边看成左值，因为在C++中，等号是可以运算符重载的，等号完全可以重载成为等号左边为右值的形式

> 纯右值是传统右值的一部分，纯右值是表达式产生的中间值，不能取地址

> 本质上，消亡值就是通过右值引用产生的值。右值一定会在表达式结束后被销毁，比如return x（x被copy以后会被销毁）, 1+2（3这个中间值会被销毁）。

## 左值引用与右值引用

 当右值出现在operater = (copy assignment) 的右侧，我们认为对其资源进行偷取/搬移（move）而非拷贝是可以的，是合理的。

那么就得有如下机制：

- 必须有语法让我们在调用端告诉编译器，这是一个”Rvalue“

- 必须有语法让我们在被调用端写出一个专门处理”Rvalue“的所谓move assignment函数

### 左值引用

左值引用根据其修饰符 `const `的不同，可以区分为**常量左值引用**和**非常量左值引用**。左值引用实际上就是指针。

非常量左值引用只能绑定到非常量左值，不能绑定到常量左值和常量右值，（因为非常左值可以改变其值，但常量不可改变，性质相矛盾）

非常量右值。而如果绑定到非常量右值，就有可能指向一个已经被销毁的对象。

常量左值引用能绑定到**非常量左值，常量左值，非常量右值，常量右值**。

```cpp
int a = 9;
int &aRef1 = a;                    // 非常量左值引用 <----> 非常量左值
const int &aRef2 = a;              // 常量左值引用   <----> 非常量左值

const int b = 4;   
int &bRef1 = b;                    // Error，非常量左值不能绑定常量右值
const int &bRef2 = b;              // 常量左值引用 <----> 常量左值

const int &ref1 = 2;               // 常量左值引用 <----> 右值（具体某个值）
const int &ref2 = a==2;                // 常量左值引用 <----> 右值（表达式）
```

### 右值引用

为什么要用到右值引用？

从实践角度讲，它能够完美解决 C++ 中长久以来为人所诟病的临时对象效率问题。从语言本身讲，它健全了 C++ 中的引用类型在左值右值方面的缺陷。从库设计者的角度讲，它给库设计者又带来了一把利器。从库使用者的角度讲，不动一兵一卒便可以获得“免费的”效率提升…

右值引用 (Rvalue Referene) 是 C++ 新标准 (C++11, 11 代表 2011 年 ) 中引入的新特性 , 它实现了转移语义 (Move Sementics) 和完美转发 (Perfect Forwarding)。它的主要目的有两个方面：

- 消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率。

- 能够更简洁明确地定义泛型函数。

为了区别，C++ 把 `&` 作为左值引用的声明符，把 `&&` 作为右值引用的声明符。例如，

```cpp
int GetValue() { return 3; }

int main() {
    int a = 0;
    int &alRef = a;                  // 左值引用
    int &&rRef1 = 1;                 // 临时对象是右值
    int &&rRef2 = GetValue();        // 调用的函数返回值为右值
}
```

```cpp
void ProcessValue(int &i) {                         // 左值引用
    cout << "lValue processed: " << i << endl;
}
void ProcessValue(int &&i) {                        // 右值引用
    cout << "rValue processed: " << i << endl;
}

int GetValue() { return 3; }

int main() {
    int a = 0;
    int b = 1;
    ProcessValue(a);                // 左值
    ProcessValue(1);                // 临时对象是右值
    ProcessValue(GetValue());        // 调用的函数为右值
    ProcessValue(a + b);                // 左值
}
```

从内存角度来看，我们给变量 a 分配好了内存，所以它是正统的内存使用者；而像函数的返回值、临时对象这样的右值，我们找不到它的内存，所以是盗版的内存使用者，这就是左值和右值的区别。

关于右值引用：

- 具名右值引用被视为左值。
- 无名右值引用被视为x值（将亡值）。
- 对函数的右值引用无论具名与否都将被视为左值。

```cpp
struct A {
  int m;
};
A&& operator+(A, A);
A&& f();
A a;
A&& ar = static_cast<A&&>(a);
```

这里表达式 `f(), f().m, static_cast<A&&>(a)`, 以及 `a + a `都是 x值。

而表达式 `ar` 为 A 类型的左值。

### move 与 forward

#### move

`std::move`可以获取一个表达式的右值引用在C++11中，标准库在`<utility>`中提供了一个有用的函数`std::move`，`std::move`并不能移动任何东西，它唯一的功能是将一个左值强制转化为右值引用，继而可以通过右值引用使用该值，以用于移动语义。从实现上讲，`std::move`基本等同于一个类型转换：`static_cast<T&&>(lvalue);`

`std::move`是将对象的状态或者所有权从一个对象转移到另一个对象，只是转移，没有内存的搬迁或者内存拷贝所以可以提高利用效率,改善性能.。

#### forward

可参考：[聊聊C++中的完美转发 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/161039484)

`std::forward`被称为**完美转发**，它的作用是保持原来的**值属性**不变。啥意思呢？通俗的讲就是，如果原来的值是左值，经`std::forward`处理后该值还是左值；如果原来的值是右值，经`std::forward`处理后它还是右值。

