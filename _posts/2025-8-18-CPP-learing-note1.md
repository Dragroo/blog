---
layout: post
author: Dragroo
title: CPP学习笔记1
tag: CPP
catorgory: 技术
---

## 重定义

是指一个变量被多次定义

### 问题代码

```cpp
//global.h
#ifndef GLOBAL_H
#define GLOBAL_H
int a;
#endif

//global.cpp
#include global.h

//main.cpp
#include "global.h"
#include "iostream"

int main(){
    cout<<a;
    return 0;
}
```

由于预处理时会添加头文件，因此main.cpp和global.cpp都会定义一个变量a，在链接时就会出现重定义的错误

### 解决办法

在头文件中使用`extern`关键字声明变量（非定义），并在某个文件中定义，以便其他文件也可以使用

```cpp
//global.h
#ifndef GLOBAL_H
#define GLOBAL_H
extern int a;
#endif

//global.cpp
#include global.h
int a=10;

//main.cpp
#include "global.h"
#include "iostream"

int main(){
    cout<<a;
    return 0;
}
```

## 指针

1. 指针的递增是依据变量的类型来操作的，比如int* 类型，其地址会增加4，而char会增加1，这是编译器对指针类型的特殊处理

## const

### const引用与const指针

const引用/指针可以指向常量，也可指向变量，普通指针或引用不能指向常量，编译器会报错
```cpp
//常量
const int a = 5;//或者int const a = 5;
//变量
int b = 6;

const int& ra = a;
const int& rb = b;
const int& rc = 7;
//引用本身的特性决定它定义后无法指向其他变量

const int* pa = &a;
const int* pb = &b;
pa= &b;//可以修改指针指向的对象，但必须是常量

const int d = 9;
//错误
//int& rd = d;
//错误
//int* pd = &d;
```

### 常指针
```cpp
int a =1;
int b =2;
int * const pa = &a;
//正确，指向的值可以被修改
*pa = 2;
//错误，指针本身不可被修改
//*pa = &b;

```

### 顶层const和底层const

1. 顶层const是指对变量（包含指针）本身const属性的修饰
```cpp
const int a = 0; //对变量修饰
int * const pa = &a; //对指针修饰
```

2. 底层const是指对指针或引用指向内容不可修改
```cpp
int a = 0;
const int *pa = a;
int b;
const int &rb = b;
```

**总结**：
- 顶层const：对象是常量（值不能改），修饰“变量本身”或“指针本身”。
- 底层const：指向的内容是常量（内容不能改），修饰“指针指向的内容”或“引用”。
### 总结
const的位置决定了指针指向的变量是否可以被修改或者指针本身可以被修改。注意定义const相关的内容必须**当即赋值**，否则没有意义

## constexpr

### 一、constexpr函数

#### 1. 非constexpr函数的运行逻辑
```cpp
// inc.h（头文件：仅声明）
#ifndef INC_H
#define INC_H
int getSize(); // 普通函数声明
#endif

// inc.cpp（实现文件：定义）
#include "inc.h"
int getSize() { // 普通函数定义
    return 42; 
}

// main.cpp（调用）
#include "inc.h"
int main() {
    int size = getSize(); // 运行时调用，依赖链接器关联定义
    return 0;
}
```

非`constexpr`函数（普通函数）的编译和运行流程与`constexpr`完全不同，核心差异在于计算时机和链接方式：
1. 编译阶段

   仅需声明可见：普通函数的声明（返回类型、参数列表）放在头文件中，定义放在`.cpp`文件中。编译器在编译`main.cpp`时，只需知道函数的声明（如`int getSize();`）即可通过语法检查，无需知道具体实现。

2. 链接阶段

   符号解析：编译器将每个`.cpp`文件编译为目标文件（`.o`或`.obj`），普通函数的定义会生成一个“符号”（函数入口地址）。链接器会在所有目标文件中查找符号，将`main.cpp`中对`getSize()`的调用与`inc.cpp`中定义的符号关联，最终生成可执行文件。

3. 运行阶段

   动态调用：程序运行时，当执行到`getSize()`调用处，CPU会跳转到函数的内存地址执行代码，计算结果后返回。无法在编译时提前计算结果，必须在运行时执行函数体。

#### 2. constexpr的运行逻辑

```cpp
// inc.h
#ifndef INC_H
#define INC_H

constexpr int getSize() { // 直接在头文件中定义
    return 42;
}

#endif // INC_H

// main.cpp（调用）
#include "inc.h"
int main() {
    int size = getSize(); // 运行时调用，依赖链接器关联定义
    return 0;
}

```

**为什么constexpr函数需要头文件内定义？**

`constexpr`（常量表达式）函数的核心特性是支持编译时计算，即编译器可以在编译阶段直接将函数调用替换为结果值（类似宏替换，但类型安全）。这要求：

编译时可见完整定义：编译器在处理`constexpr int size = getSize();`时，必须知道`getSize()`的具体实现才能计算结果。若定义放在`.cpp`文件中，其他文件（如`main.cpp`）仅能看到头文件中的声明，无法获取定义，导致编译失败。
隐式内联特性：`constexpr`函数默认隐式`inline`，允许在多个编译单元中存在定义（只要内容一致），因此适合放在头文件中供多文件共享，避免链接冲突。

#### 3. 对比总结


|特性|constexpr函数|非constexpr函数（普通函数）|
|--------------|------------|----------|
|计算时机|编译时（优先）或运行时（当参数为变量时）|仅运行时|
|定义位置|必须在头文件中（需编译时可见）|通常在.cpp文件中（仅声明在头文件）
|链接方式|隐式inline，无链接冲突|依赖链接器解析符号，重复定义会冲突|
|典型用途|编译时常量计算（如数组大小、模板参数）|运行时动态逻辑（如IO操作、复杂计算）|

### 二、constexpr指针

#### （一）constexpr指针的特殊性与修改必要性
constexpr指针的核心要求是编译时确定地址值，因此需满足以下约束（也是前面对constexpr函数修改的根本原因）：

##### 1. 指针本身必须是“编译时常量”（顶层const）
constexpr修饰指针时，指针变量本身的值（即指向的地址）必须在编译时确定，且后续不可修改（类似`int* const p`的“顶层const”效果）。例如：

```cpp
int global = 42;               // 全局变量（静态存储期，地址编译时可知）
constexpr int* p1 = &global;   // 正确：指向全局变量，地址编译时确定
constexpr int* p2 = nullptr;   // 正确：nullptr是编译时常量

void func() {
    int local = 10;
    constexpr int* p3 = &local; // 错误：局部变量地址编译时未知（运行时动态分配）
}
```

##### 2. 指向的对象必须满足“编译时可访问”
constexpr指针指向的对象必须具有静态存储期（如全局变量、static变量）或本身是constexpr对象（编译时初始化），否则地址无法在编译时确定：

```cpp
constexpr int ce = 42;         // constexpr对象（编译时初始化，地址固定）
constexpr const int* p = &ce;  // 正确：指向constexpr对象，地址编译时可知
```

##### 3. 与const的组合：区分“指向常量”与“常量指针”
constexpr指针需明确“指向的对象是否为常量”（底层const），语法上有两种常见形式：

| 写法 | 含义 | 等价于 |
|------|------|--------|
| `constexpr int* p` | 指针本身是常量（顶层const），指向int | `int* const p`（编译时初始化） |
| `constexpr const int* p` | 指针本身是常量，指向const int | `const int* const p`（编译时初始化） |


#### （二）非constexpr指针（普通指针）的运行逻辑
非constexpr指针（普通指针）的核心是运行时动态寻址，完全不受编译时约束，具体流程如下：

##### 1. 编译阶段：仅检查语法，不关心地址值
编译器只需确保指针的类型匹配（如`int*`不能指向`double`），无需知道指针的具体地址值。例如：

```cpp
// main.cpp
#include "inc.h" // 仅包含普通函数声明：int* getPtr();
int main() {
    int* p = getPtr(); // 编译通过：仅需知道getPtr()返回int*，无需地址值
}

// inc.cpp
int global = 100;
int* getPtr() { return &global; } // 定义在.cpp中，编译时对main.cpp不可见
```

##### 2. 链接阶段：符号解析地址
编译器将.cpp编译为目标文件（.o），普通指针的地址在链接时由链接器从其他目标文件中查找并绑定。例如：
- inc.cpp编译后生成包含global地址的符号；
- main.cpp中p的地址在链接时被替换为global的实际内存地址。

##### 3. 运行阶段：动态访问与修改
程序运行时，指针可以：
- 指向任意可访问对象：包括局部变量（栈内存）、动态分配内存（堆内存）等；
- 动态修改指向：指针本身的值（地址）可在运行时改变；
- 运行时计算地址：通过表达式（如`&arr[i]`、`p+1`）动态获取地址。

```cpp
void func() {
    int local = 20;
    int* p = &local;       // 指向栈内存（运行时地址）
    p = new int(30);       // 指向堆内存（运行时动态分配）
    *p = 40;               // 修改指向的对象值
    delete p;
}
```


#### （三）核心差异总结

| 维度 | constexpr指针 | 非constexpr指针（普通指针） |
|------|--------------|------------------------------|
| 地址确定时机 | 编译时（必须是常量表达式） | 运行时（动态计算或符号解析） |
| 指向对象 | 仅限静态存储期对象或constexpr对象 | 任意可访问对象（栈、堆、全局等） |
| 可修改性 | 指针本身不可修改（顶层const） | 可修改指向（除非显式const修饰） |
| 典型用途 | 编译时地址计算（如数组大小、模板参数） | 运行时动态内存管理、对象引用 |

**一句话总结**：constexpr指针是“编译时确定地址的常量指针”，需严格绑定静态对象；普通指针是“运行时动态寻址的变量”，灵活但依赖运行时内存布局。两者的设计目标分别对应编译时优化与运行时灵活性。

### 模板特化与非类型模板参数

模板特化与非类型模板参数是C++泛型编程中两个不同维度的概念，核心区别如下：

#### 一、定义与核心目的

1. 模板特化（Template Specialization）

   定义：为已有的通用模板（类模板/函数模板）针对特定类型或场景提供定制化实现，覆盖通用模板的默认行为。
   核心目的：解决通用模板对某些特殊类型/值处理不高效或逻辑不适用的问题（如为int*指针类型特化类模板，避免通用版本的错误逻辑）。

2. 非类型模板参数（Non-type Template Parameters）

   定义：模板参数列表中传入的具体数值、指针、引用或枚举值（而非类型名），编译时确定其值，用于配置模板的行为。
   核心目的：在编译期传递常量信息，实现模板的参数化配置（如固定数组大小、缓冲区容量、策略标记等）。

#### 二、语法与使用场景

**模板特化**

- 语法：

  - 全特化：`template<> class 类模板名<特化类型> { ... }`
  - 偏特化（仅类模板）：`template<typename T> class 类模板名<T*> { ... }`（针对指针类型偏特化）

- 场景：

  为`std::vector<bool>`特化以压缩存储空间（每个元素占1bit而非1字节）。
  为`const char*`类型特化字符串比较函数，避免按地址比较而非内容比较。


**非类型模板参数**

- 语法：
`template<类型 标识符> class 类模板名 { ... }`
（类型必须是整数/枚举、指针、引用、std::nullptr_t，C++20后支持浮点数）
- 场景：

  - 固定大小数组：`template<int N> class Array { int data[N]; }`;（编译时确定数组大小）。
  - 策略标记：`template<bool ThreadSafe> class Logger { ... };`（通过ThreadSafe=true/false控制是否加锁）。

#### 三、本质区别

|维度|模板特化|非类型模板参数|
|---|--------|------------ |
|作用对象|针对已有模板的特定类型/值定制实现|为模板传入编译期常量配置模板行为|
|语法地位|是对通用模板的“补充”或“覆盖”|是模板参数的一种类型（与类型参数并列）|
|实例化逻辑|特化版本优先于通用版本被实例化|参数值参与模板实例化，不同值对应不同实例|
|核心价值|解决特殊类型的适配问题|实现模板的编译期参数化配置|

#### 四、代码示例

```cpp
// 通用模板：返回类型大小  
template <typename T>  
class SizeOf { public: static constexpr int value = sizeof(T); };  

// 特化：针对指针类型返回指向的元素大小（假设已定义指针成员）  
template <typename T>  
class SizeOf<T*> { public: static constexpr int value = sizeof(*T()); };  

// 使用：  
SizeOf<int>::value;    // 通用版本：4（int大小）  
SizeOf<int*>::value;   // 特化版本：4（int*指向的int大小）
```

```cpp
// 非类型参数N：编译期确定数组大小  
template <int N>  
class FixedArray {  
    int data[N];  
public:  
    static constexpr int size() { return N; }  
};  

// 使用：  
FixedArray<10> arr;  // 数组大小为10，编译期固定  
arr.size();          // 返回10
```

### 五、一句话理解

模板实例化传的参数是类型，而非类型模板参数实例化传的是具体的值

## 类型别名与类型推导

### 类型别名

```cpp
#include <iostream>
//下面的三种方式都可以定义类型别名
typedef int num;
// using num = int;
// #define num int
int main(){
    num a = 5;
    num b = 10;
    num c = a + b;
    std::cout << "The sum is: " << c << std::endl;
    return 0;
}
```

### auto类型推导
```cpp
int a = 0;
auto b=0; //b为int类型
auto c=a; //int
auto d=0, *pd = &d;//d-int, pd-int*
```
`auto`关键字会去除顶层`const`（**`auto&` 例外**）

```cpp
const int a = 0;
auto b = a;//int
const int * pa =&a;
auto pb = pa; //const int *
auto& ra = a; //const int &
```

### decltype类型推导
规则较为复杂