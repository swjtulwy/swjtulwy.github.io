# C++中的虚函数


<!--more-->

c++作为面向对象的语言，主要有三大特性：继承、封装、多态。关于多态，简而言之就是用父类型的指针指向其子类的实例，然后通过父类的指针调用实际子类的成员函数。这种技术可以让父类的指针有“多种形态”，这是一种泛型技术。所谓泛型技术，说白了就是试图使用不变的代码来实现可变的算法。比如：模板技术，RTTI技术，虚函数技术，要么是试图做到在编译时绑定，要么试图做到运行时绑定。因此C++的多态分为静态多态（编译时多态）和动态多态（运行时多态）两大类。静态多态通过重载、模板来实现；动态多态就是通过本文的主角虚函数来体现的。

## 虚函数的内存分布

对C++ 了解的人都应该知道虚函数（Virtual Function）是通过一张虚函数表（Virtual Table）来实现的。简称为V-Table。在这个表中，主要是一个类的虚函数的地址表，这张表解决了继承、覆盖的问题，保证其内容真实反映实际的函数。这样，在有虚函数的类的实例中这个表被分配在了这个实例的内存中，所以，当我们用父类的指针来操作一个子类的时候，这张虚函数表就显得尤为重要了，它就像一个地图一样，指明了实际所应该调用的函数。因此有必要知道虚函数在内存中的分布。  

```cpp
class A {
  public:
    virtual void v_a(){}
    virtual ~A(){}
    int64_t _m_a;
};

int main(){
    A* a = new A();
    return 0;
}
```

如以上代码所示，在C++中定义一个对象 A，那么在内存中的分布大概是如下图这个样子。

- 首先在主函数的栈帧上有一个 A 类型的指针指向堆里面分配好的对象 A 实例。
- 对象 A 实例的**头部**是一个 vtable 指针，紧接着是 **A 对象按照声明顺序排列的成员变量**。（当我们创建一个对象时，便可以通过实例对象的地址，得到该实例的虚函数表，从而获取其函数指针。）
- vtable 指针指向的是代码段中的 A 类型的**虚函数表中的第一个虚函数起始地址**。
- 虚函数表的结构其实是有一个头部的，叫做 vtable_prefix ，紧接着是按照声明顺序排列的虚函数。
- 注意到这里有两个虚析构函数，因为对象有两种构造方式，栈构造和堆构造，所以对应的，对象会有两种析构方式，其中堆上对象的析构和栈上对象的析构不同之处在于，栈内存的析构不需要执行 delete 函数，会自动被回收。
- typeinfo 存储着 A 的类基础信息，包括父类与类名称，C++关键字 typeid 返回的就是这个对象。
- typeinfo 也是一个类，对于没有父类的 A 来说，当前 typeinfo 是 class_type_info 类型的，从虚函数指针指向的vtable 起始位置可以看出。

<img src="https://article.biliimg.com/bfs/article/2feb13cb2aa9b6015d781e1f837f0649b61bd05c.png@1e_1c.webp" title="" alt="" data-align="center">

通过之前的分析可以知道其实传统认为的虚函数表并不是单独存在而是虚表的一部分，如下图所示    

<img src="https://article.biliimg.com/bfs/article/637f41d1fa6f628aba9c4f60afec07dd62e4fa63.png" title="" alt="" data-align="center">

- 紫色线框中的内容仅限于虚拟继承的情形（若无虚拟继承，则无此内容）
- “offset to top”是指到对象起始地址的偏移值，只有多重继承的情形才有可能不为0，单继承或无继承的情形都为0。
- “RTTI information”是一个对象指针，它用于唯一地标识该类型。
- “virtual function pointers”也就是我们之前理解的虚函数表，其中存放着虚函数指针列表。

Virtual table（虚表）只实现了虚拟函数的一半机制,如果只有这些是没有用的。只有用某种方法指出每个对象对应的 `vtbl `时,它们才能使用。这是 virtual table pointer 的工作,它来建立这种联系。

每个声明了虚函数的对象都带有它,它是一个看不见的数据成员,指向对应类的virtual table。这个看不见的数据成员也称为 `vptr`,被编译器加在对象里,位置只有才编译器知道。

## 虚函数实现原理

### 虚函数的调用过程

当调用一个虚函数时，首先通过对象内存中的vptr找到虚函数表vtbl，接着通过vtbl找到对应虚函数的实现区域并进行调用。其中**被执行的代码必须和调用函数的对象的动态类型相一致**。编译器需要做的就是如何高效的实现提供这种特性。不同编译器实现细节也不相同。大多数编译器通过虚表vtbl（virtual table）和虚表指针vptr（virtual table pointer）来实现的。 当一个类声明了虚函数或者继承了虚函数，这个类就会有自己的vtbl。vtbl核心就是一个函数指针数组，有的编译器用的是链表，不过方法都是差不多。vtbl数组中的每一个元素对应一个函数指针指向该类的一个虚函数，同时该类的每一个对象都会包含一个vptr，vptr指向该vtbl的地址。 在有继承关系时(子类相对于其直接父类)

- 一般继承时，子类的虚函数表中先将父类虚函数放在前，再放自己的虚函数指针。
- 如果子类覆盖了父类的虚函数，将被放到了虚表中**原来父类虚函数**的位置。
- 在多继承的情况下，**每个父类都有自己的虚表，子类的成员函数被放到了第一个父类的表中。**，也就是说当类在多重继承中时，其实例对象的内存结构并不只记录一个虚函数表指针。基类中有几个存在虚函数，则子类就会保存几个虚函数表指针

```cpp
class A{
private:
    uint64_t a;
public:
    virtual void A_a(){std::cout << __func__;}
};
class C{
private:
    uint64_t c;
public:
    virtual void C_a(){std::cout << __func__;}
};

class D:public A,public C{
private:
    uint64_t d;
public:
    virtual void D_a(){std::cout << __func__;}
};
```

<img src="https://article.biliimg.com/bfs/article/4e939d3122030034b298a724b868b9b240825dcd.jpg" title="" alt="" data-align="center">

### 性能分析

#### 调用性能

从前面虚函数的调用过程可知。当调用虚函数时过程如下（引自More Effective C++）:

1. **通过对象的 vptr 找到类的 vtbl**。  这是一个简单的操作,因为编译器知道在对象内 哪里能找到 vptr(毕竟是由编译器放置的它们)。因此这个代价只是一个偏移调整(以得到 vptr)和一个指针的间接寻址(以得到 vtbl)。

2. **找到对应 vtbl 内的指向被调用函数的指针**。  这也是很简单的, 因为编译器为每个虚函数在 vtbl 内分配了一个唯一的索引。这步的代价只是在 vtbl 数组内的一个偏移。

3. **调用第二步找到的的指针所指向的函数**。  
- 在单继承的情况下  
  调用虚函数所需的代价基本上和非虚函数效率一样，在大多数计算机上它多执行了很少的一些指令，**所以有很多人一概而论说虚函数性能不行是不太科学的。**
- 在多继承的情况  
  由于会根据多个父类生成多个vptr，在对象里为寻找 vptr 而进行的偏移量计算会变得复杂一些，但这些并不是虚函数的性能瓶颈。**虚函数运行时所需的代价主要是虚函数不是内联函数**。这也是非常好理解的，是因为内联函数是指在编译期间用被调用的函数体本身来代替函数调用的指令，但是虚函数的“虚”是指“直到运行时才能知道要调用的是哪一个函数。”但虚函数的运行时多态特性就是要在运行时才知道具体调用哪个虚函数，所以没法在编译时进行内联函数展开。当然如果通过对象直接调用虚函数它是可以被内联，但是大多数虚函数是通过对象的指针或引用被调用的，这种调用不能被内联。 因为这种调用是标准的调用方式，所以虚函数实际上不能被内联。

#### 空间占用

在上面的虚函数实现原理部分，可以看到为了实现运行时多态机制，**编译器会给每一个包含虚函数或继承了虚函数的类自动建立一个虚函数表**，所以虚函数的一个代价就是会增加类的体积。在虚函数接口较少的类中这个代价并不明显，虚函数表vtbl的体积相当于几个函数指针的体积，如果你有大量的类或者在每个类中有大量的虚函数,你会发现 vtbl 会占用大量的地址空间。但这并不是最主要的代价，**主要的代价是发生在类的继承过程**中，在上面的分析中，可以看到，当子类继承父类的虚函数时，子类会有自己的vtbl，如果子类只覆盖父类的一两个虚函数接口，子类vtbl的其余部分内容会与父类重复。



**如果存在大量的子类继承，且重写父类的虚函数接口只占总数的一小部分的情况下，会造成大量地址空间浪费。** 在一些GUI库上这种大量子类继承自同一父类且只覆盖其中一两个虚函数的情况是经常有的，这样就导致UI库的占用内存明显变大。 由于虚函数指针`vptr`的存在，虚函数也会增加该类的每个对象的体积。在单继承或没有继承的情况下，类的每个对象会多一个`vptr`指针的体积，也就是4个字节；在多继承的情况下，类的每个对象会多N个（N＝包含虚函数的父类个数）`vptr`的体积，也就是**4N**个字节。当一个类的对象体积较大时，这个代价不是很明显，但当一个类的对象很轻量的时候，如成员变量只有4个字节，那么再加上4（或4N）个字节的`vptr`，对象的体积相当于翻了1（或N）倍，这个代价是非常大的。

## 虚函数应用的注意事项

### 内联函数 (inline)

虚函数用于实现运行时的多态，或者称为晚绑定或动态绑定。而内联函数用于提高效率。内联函数的原理是，在编译期间，对调用内联函数的地方的代码替换成函数代码。内联函数对于程序中需要频繁使用和调用的小函数非常有用。默认地，类中定义的所有函数，除了虚函数之外，会隐式地或自动地当成内联函数(注意：内联只是对于编译器的一个建议，编译器可以自己决定是否进行内联).  

无论何时，**使用基类指针或引用来调用虚函数，它都不能为内联函数(因为调用发生在运行时)**。但是，无论何时，使用类的对象(不是指针或引用)来调用时，可以当做是内联，因为编译器在编译时确切知道对象是哪个类的。

### 静态成员函数 (static)

static成员不属于任何类对象或类实例，所以即使给此函数加上virutal也是没有任何意义的。此外静态与非静态成员函数之间有一个主要的区别，那就是**静态成员函数没有this指针**，从而导致两者调用方式不同。虚函数依靠vptr和vtable来处理。vptr是一个指针，在类的构造函数中创建生成，并且只能用this指针来访问它，因为它是类的一个成员，并且vptr指向保存虚函数地址的vtable。**虚函数的调用关系：this -> vptr -> vtable ->virtual function**，对于静态成员函数，它没有this指针，所以无法访问vptr. 这就是为何**static函数不能为virtual**。

### 构造函数 (constructor)

虚函数基于虚表`vtable`（内存空间），构造函数 (`constructor`) 如果是`virtual`的，调用时也需要根据`vtable`寻找，但是`constructor`是`virtual`的情况下是找不到的，因为`constructor`自己本身都不存在了，创建不到`class`的实例，没有实例`class`的成员（除了`public static/protected static for friend class/functions`，其余无论是否`virtual`）都不能被访问了。此外构造函数不仅不能是虚函数。而且在构造函数中调用虚函数，实际执行的是父类的对应函数，因为自己还没有构造好,多态是被`disable`的。

### 析构函数 (deconstructor)

**对于可能作为基类的类的析构函数要求就是`virtual`的**。因为如果不是`virtual`的，派生类析构的时候调用的是基类的析构函数，而基类的析构函数只要对基类部分进行析构，从而可能导致派生类部分出现内存泄漏问题。

### 纯虚函数

析构函数可以是纯虚的，但**纯虚析构函数必须有定义体**，因为析构函数的调用是在子类中隐含的。
