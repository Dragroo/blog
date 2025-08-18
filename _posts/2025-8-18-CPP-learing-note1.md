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
