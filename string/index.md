# 字符串的一些问题


初学C语言会觉得字符串处理是一件比较繁琐的事情，后来学了C++发现字符串的处理可以这么方便。但是C++又为了兼容C，导致经常有混用的情况。本文就重点介绍C++中的字符串和C风格字符串的区别。

首先，明确一点，C语言中的字符串以及相关的标准库都能完全在C++中兼容使用。所以在介绍C++中的字符串特指C++中特有的。下面介绍几个概念

## 字符串字面值(string literal)

字符串字面值是一串常量字符，字符串字面值常量用双引号括起来的零个或多个字符表示，为兼容C语言，C++中所有的字符串字面值都由编译器自动在**末尾添加一个空字符**。
字符串没有变量名字，自身表示自身

```cpp
"Hello World!" //simple string literal
"" //empty string literal
"\nCC\toptions\tfile.[cC]\n" //string literal using newlines and tabs
```

```
字符字面值： 'A' //single quote:character literal
字符串字面值： "A" //double quote:character string literal.包含字母A和空字符的字符串
```

字符串字面值可以赋值给变量，但是与字符串字面值相关联的内存空间位于**只读部分**，因此它是**常量字符数组**。 

```c
char* ptr = "hello";
ptr[1] = 'a';//crash! attemps to write to read-only memory. //只读
```

因此，当引用字符串字面值的时候使用指向const的字符数组。

```c
const char* ptr = "hello";
ptr[1] = 'a';//bug! attempts to write to read-only memory.
```

当将字符串字面值赋值给**字符数组的初始值**的时候。由于**字符数组存放与栈中**，不允许引用其他地方的内存，因此编译器会将字符串字面值**复制**到栈的数组内存中。因此，**可以进行相应的修改**。

```c
char stackArray[] = "hello";
stackArray[1] = 'a';
```

## C风格字符串

字符串字面值的类型实质是`const char`类型的数组。C++从C语言继承下来的一种通用结构是C风格字符串，而字符串字面值就是该类型的实例。C风格字符串是以空字符null结束的字符数组。

```c
char ca1[]={'C', '+', '+'}; // no null, not C-style string
char ca2[]={'C', '+', '+', '\0'}; // explicit null
char ca3[]="C++"; // null terminator added automatically
const char *cp="C++"; // null terminator added automatically
char *cp1=ca1; // points to first element of a array, but not C-style string
char *cp2=ca2; // points to first element of a null-terminated char array
```

ca1和cp1都不是C风格字符串：ca1是一个不带结束符`null`的字符数组，而指针cp1指向ca1，因此，它指向的并不是以null结束的数组。

C++语言通过`(const) char *`类型的指针来操纵C风格字符串。

C 风格字符串在C语言中有C标准库函数来处理：`string.h`

C 语言中也有对字符进行处理的标准库函数: `ctype.h`

这些库函数在C++中都有对应的版本，只需要去掉`.h`前面加上c就行了，这样也能在C++区分出这是C风格的函数

即`cstring`和`cctype`

C风格字符串中带来困恼的原因是字符数组总要**以空字符结尾**，这给计算数组空间带来不便，也容易引发安全问题，处理字符串的相关复制，移动情形时，调用者必须保证目标字符串有足够的大小。

另一个苦恼是C风格的字符串的复制，截取等相关操作不能像对待基础数据类型一样方便，这些情况在C++风格的字符串中由重载运算符实现，方便了用户。

所以一般C++不推荐使用C风格字符串，即使要使用，也尽量在C标准库中使用诸如`strn`前缀的函数处理，以降低风险。

## C++风格字符串

C++使用标准库类型string表示可变长的字符序列，标准库中的名称符号都包含在命名空间呢`std`中。

在初始化`string`对象时可以使用多种方式比如：

```cpp
string s1;  // 默认初始化，s1是一个空字符串
string s2 = s1; // s2是s1的副本
string s3 = "hiya"; // s3是该字符串字面值的副本
string s4(10, 'c'); // s4的内容是cccccccccc
```

`string`对象可以用标准库中的流直接读写

比如`cin`，但是读时从非空白开始，即忽略开头的空白，遇到下一处空白结束。

当要读取一整行的时候可以使用`getline(cin,line)` ，(`line`是读取后存放的`string`对象)该函数从给定的输入流中读取内容，**直到遇到换行符为止**，换行符**也被读取进来**，然后把读取的内容存放到`line`中，不包含换行符。

`string`对象可以使用== 、<、>、>=等比较符号，这个很好理解，不多说。

**另外值得注意的是，string对象之间可以相加，字符串字面值和string对象也可以相加，但是字符串字面值之间不能相加！**

最后谈一下C风格的字符串和C++风格字符串怎么转换

C风格字符串转换到C++的string对象很简单，可以将C风格字符串(包括字面值)作为string初始化的参数即可，或者可以用`=`号。

更一般的情况是，任何出现字符串字面值的地方都可以用以空字符结束的字符数组来替代：

- **即允许使用以空字符结束的字符数组来初始化string对象或为string对象赋值。**
- **在string对象的加法运算中允许使用以空字符结束的字符数组作为其中一个运算对象；在string对象的复合赋值运算中允许使用以空字符结束的字符数组作为右侧的运算对象。**

`string`对象转换为C风格字符串则有一个方法`c_str()` ,直接调用即可。

```cpp
string s = "msl";
string s1("dad");
const char* p = "dsjka";
char p1[] = "das";
string w = s + "da";
string q = s + p;
string t = s + p1;
string u = p + p1;  // wrong
string k = p + "daas"; // wrong
```

如上面代码所示，最后两个有错，其余都是允许的。

## 字符串与整型的互相转换

先说`string`对象与int型转换

`int`转换为`string`对象有全局函数`std::to_string()`(C++11)。

当然还可以用其他的`stringstream`的方式。

`string`转为`int`可以利用标准库中的`atoi()`函数（ *ascii to integer*），但要先转为C风格串，从C标准库继承过来，举例

```cpp
std::string str = "123";
int n = atoi(str.c_str()); // 先转为C风格
cout<<n; //123
```

同时除了`atoi()`还要`atof()`等一系列函数。

另外用一些C语言中的格式化输入输出函数也可以实现,比如`sprintf,sscanf`等。

```cpp
string s1 = "1234";
int a = atoi(s1.c_str());
string s2 = to_string(a);
```

