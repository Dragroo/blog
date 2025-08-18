---
layout: post
author: Dragroo
title: decltype类型推导
tag: CPP
catorgory: 技术
---

# decltype 类型推导规则详解

`decltype` 的类型推导规则核心是**完全依据表达式的原始类型和值类别（lvalue/rvalue）** 进行推导，不会像 `auto` 那样忽略顶层 `const`。以下是几种典型情况及推导规则：


## 一、情况1：表达式为普通变量/函数名（非括号包裹）
**规则**：直接推导为变量/函数的原始类型（保留顶层 `const`、引用、`volatile` 等所有修饰符）。

```cpp
const int x = 42;          // x的类型：const int（顶层const）
decltype(x) a = x;         // a的类型：const int（保留顶层const）
a = 100;                   // 错误：a是const int，不可修改

int& func();               // 函数返回类型：int&
decltype(func()) b = x;    // b的类型：int&（保留引用）
b = 200;                   // 正确：通过引用修改x的值（x变为200）
```


## 二、情况2：表达式为函数调用
**规则**：推导为函数的返回类型（不执行函数，仅看声明），保留引用和 `const`。

```cpp
int foo();                 // 返回类型：int
decltype(foo()) c = 10;    // c的类型：int

const int& bar();          // 返回类型：const int&
decltype(bar()) d = x;     // d的类型：const int&（保留const和引用）
```


## 三、情况3：表达式为左值（lvalue）且非变量名
**规则**：推导为该类型的左值引用（`T&`）。常见场景：赋值表达式、解引用指针、数组元素访问等。

```cpp
int a = 3, b = 4;
int arr[5] = {1,2,3,4,5};

// 1. 赋值表达式是左值
decltype(a = b) e = a;     // a = b 返回a的左值，推导为 int&
e = 100;                   // a变为100

// 2. 解引用指针是左值
int* p = &a;
decltype(*p) f = a;        // *p是左值，推导为 int&

// 3. 数组元素访问是左值
decltype(arr[0]) g = arr[0]; // arr[0]是左值，推导为 int&
```


## 四、情况4：表达式为右值（rvalue）
**规则**：推导为该类型本身（非引用）。常见场景：字面量、算术表达式、临时对象等。

```cpp
// 1. 字面量是右值
decltype(42) h = 10;       // 42是右值，推导为 int

// 2. 算术表达式是右值
decltype(a + b) i = 20;    // a + b返回右值，推导为 int
```


## 五、情况5：表达式带双层括号 `(expr)`
**规则**：无论 `expr` 原本是否为变量，均视为表达式处理（左值表达式推导为 `T&`，右值推导为 `T`）。

```cpp
int x = 5;
decltype(x) j = 10;        // 情况1：x是变量，推导为 int
decltype((x)) k = x;       // 情况5：(x)视为表达式（左值），推导为 int&
k = 20;                    // x变为20
```


## 六、情况6：表达式为指针/成员访问
**规则**：根据运算符特性判断值类别：
- `*p`（解引用）：左值 → 推导为 `T&`
- `p->member` 或 `obj.member`：若成员是变量，则同情况1（保留原始类型）

```cpp
struct S { int num; };
S s;
S* ptr = &s;

decltype(*ptr) l = s;      // *ptr是左值（指向对象），推导为 S&
decltype(ptr->num) m = 3;  // num是int变量，推导为 int（同情况1）
```


## 七、情况7：表达式为数组
**规则**：推导为数组类型（而非指针），保留数组大小。

```cpp
int arr[5];
decltype(arr) n;           // n的类型：int[5]（数组类型）
n[0] = 10;                 // 正确：可通过数组下标访问

// 对比auto：auto会推导为指针（int*）
auto arr_ptr = arr;        // arr_ptr的类型：int*
```


## 八、情况8：与 `decltype(auto)` 结合（C++14）
**规则**：用于函数返回类型推导，自动根据表达式值类别选择推导规则（等价于 `decltype(expr)`），避免显式写复杂类型。

```cpp
// 例1：返回左值引用
decltype(auto) func1(int& x) {
    return x;              // 返回x（左值），推导为 int&
}

// 例2：返回右值
decltype(auto) func2(int x) {
    return x + 1;          // 返回临时值（右值），推导为 int
}
```


## 核心对比：decltype vs auto

| 特性         | decltype(expr)                  | auto                            |
|--------------|---------------------------------|---------------------------------|
| 顶层 `const` | 保留（如 `const int → const int`） | 去除（如 `const int → int`）    |
| 引用         | 保留（如左值表达式 → `T&`）       | 去除（如 `int& → int`）         |
| 表达式值类别 | 敏感（左值→`T&`，右值→`T`）       | 不敏感（统一按值推导）          |
| 括号影响     | 有（`(x)` 视为表达式，可能推导为 `T&`） | 无（`auto (x) = y` 等价于 `auto x`） |


## 记忆口诀
decltype 推导三原则：
- 变量名直接给类型，函数调用看返回；
- 左值表达式加引用，右值表达式原样来；
- 括号包裹变表达式，数组指针要分开。

通过具体场景的代码示例，可更清晰地理解 `decltype` 的推导逻辑。