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

### 总结
const的位置决定了指针指向的变量是否可以被修改或者指针本身可以被修改。注意定义const相关的内容必须**当即赋值**，否则没有意义

## constexpr

### constexpr函数

#### 一、非constexpr函数的写法
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

#### 二、constexpr的使用

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

#### 三、对比总结


|特性|constexpr函数|非constexpr函数（普通函数）|
|--------------|------------|----------
|计算时机|编译时（优先）或运行时（当参数为变量时）|仅运行时|
|定义位置|必须在头文件中（需编译时可见）|通常在.cpp文件中（仅声明在头文件）
|链接方式|隐式inline，无链接冲突|依赖链接器解析符号，重复定义会冲突|
|典型用途|编译时常量计算（如数组大小、模板参数）|运行时动态逻辑（如IO操作、复杂计算）|

## 模板

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
