---
layout: post
author: Dragroo
title: 泛型编程-参数
tag: CPP
catergory: 技术
---

# 模板特化与非类型模板参数详解

## 类型模板参数 (Type Template Parameters)

类型模板参数是模板编程中最基础的形式，使用 `typename` 或 `class` 关键字声明。它们代表在模板实例化时需要提供的具体类型。

### 基本语法与特性

```cpp
template <typename T> // T 是类型模板参数
class Container {
    T element;
public:
    void set(const T& value) { element = value; }
    T get() const { return element; }
};

// 使用示例
Container<int> intContainer;   // T 被指定为 int
Container<std::string> strContainer; // T 被指定为 std::string
```

### 关键特点：
1. **表示类型**：在模板定义中代表某种类型
2. **实例化时指定**：使用时需要提供具体类型
3. **支持多种类型**：可同时有多个类型参数
   ```cpp
   template <typename Key, typename Value>
   class KeyValuePair {
       Key key;
       Value value;
   };
   ```
4. **类型推导**：函数模板可自动推导类型
   ```cpp
   template <typename T>
   void print(const T& value) {
       std::cout << value << std::endl;
   }
   print(42); // 自动推导 T 为 int
   ```

## 非类型模板参数 (Non-type Template Parameters)

非类型模板参数允许使用值（而非类型）作为模板参数，这些值必须是编译期常量。

### 基本语法与特性

```cpp
template <int Size> // Size 是非类型模板参数
class FixedArray {
    int data[Size];
public:
    int& operator[](int index) { return data[index]; }
};

// 使用示例
FixedArray<10> smallArray;   // Size = 10
FixedArray<100> largeArray;  // Size = 100
```

### 允许的类型：
1. **整型**：`int`, `long`, `char` 等
2. **指针/引用**：指向函数或对象的指针/引用
3. **枚举类型**
4. **C++20 新增**：浮点类型和字面量类类型

### 关键特点：
1. **表示值**：在模板定义中代表具体的值
2. **必须是编译期常量**：值在编译时确定
3. **类型必须明确指定**：
   ```cpp
   template <auto Value> // C++17 起支持 auto 推导
   class ConstantHolder {
       static constexpr auto value = Value;
   };
   ConstantHolder<42> intHolder;   // Value 类型为 int
   ConstantHolder<'A'> charHolder; // Value 类型为 char
   ```

## 模板特化 (Template Specialization)

模板特化为特定类型或值提供特殊实现，分为全特化和偏特化。

### 1. 全特化 (Full Specialization)

为所有模板参数提供具体类型/值：

```cpp
// 主模板
template <typename T>
class Printer {
public:
    void print() {
        std::cout << "Generic printer" << std::endl;
    }
};

// int 类型的全特化
template <>
class Printer<int> {
public:
    void print() {
        std::cout << "Integer printer" << std::endl;
    }
};

// 使用
Printer<double> d; d.print(); // 输出: Generic printer
Printer<int> i; i.print();    // 输出: Integer printer
```

### 2. 偏特化 (Partial Specialization)

为部分模板参数提供具体类型/值，或添加约束：

```cpp
// 主模板
template <typename T, typename U>
class Pair {
    T first;
    U second;
};

// 偏特化：当两个类型相同时
template <typename T>
class Pair<T, T> {
    T first;
    T second;
};

// 偏特化：当第二个类型是指针时
template <typename T, typename U>
class Pair<T, U*> {
    T first;
    U* second;
};

// 使用
Pair<int, double> p1; // 使用主模板
Pair<int, int> p2;    // 使用第一个偏特化
Pair<int, double*> p3; // 使用第二个偏特化
```

## 关键区别总结

| 特性               | 类型模板参数                     | 非类型模板参数                  | 模板特化                         |
|--------------------|----------------------------------|---------------------------------|----------------------------------|
| **本质**          | 表示类型                        | 表示值                          | 为特定参数提供定制实现           |
| **声明方式**      | `template <typename T>`         | `template <int N>`             | 基于主模板的扩展                |
| **实例化要求**    | 提供具体类型                    | 提供编译期常量值                | 自动匹配特定类型/值组合         |
| **主要用途**      | 创建通用类型容器/算法           | 固定大小容器/编译期计算         | 优化特定类型的性能/行为         |
| **值类型限制**    | 无                              | 必须是编译期常量                | 无                              |
| **C++标准支持**   | C++98 起支持                    | C++98 起支持                   | C++98 起支持                    |
| **典型示例**      | `std::vector<T>`               | `std::array<T, N>`             | `std::vector<bool>` 特化       |

## 综合应用示例

```cpp
#include <iostream>
#include <array>

// 主模板：类型参数 T 和非类型参数 Size
template <typename T, size_t Size>
class SmartArray {
    std::array<T, Size> data;
public:
    void fill(const T& value) {
        for(auto& item : data) item = value;
    }
    
    void print() const {
        std::cout << "Generic SmartArray: ";
        for(const auto& item : data) std::cout << item << " ";
        std::cout << "\n";
    }
};

// 偏特化：当 Size 为 0 时的特殊处理
template <typename T>
class SmartArray<T, 0> {
public:
    void fill(const T&) {
        std::cout << "Cannot fill zero-sized array\n";
    }
    
    void print() const {
        std::cout << "Zero-sized SmartArray\n";
    }
};

// 全特化：int 类型且大小为 5 的特殊实现
template <>
class SmartArray<int, 5> {
    std::array<int, 5> data;
public:
    void fill(int value) {
        std::cout << "Special int array filling...\n";
        for(auto& item : data) item = value;
    }
    
    void print() const {
        std::cout << "Special int array: ";
        for(int val : data) std::cout << val << " ";
        std::cout << "\n";
    }
};

int main() {
    SmartArray<double, 4> da; // 使用主模板
    da.fill(3.14);
    da.print();
    
    SmartArray<char, 0> ca; // 使用 Size=0 的偏特化
    ca.fill('A');
    ca.print();
    
    SmartArray<int, 5> ia; // 使用全特化
    ia.fill(42);
    ia.print();
}
```

输出：
```
Generic SmartArray: 3.14 3.14 3.14 3.14 
Cannot fill zero-sized array
Zero-sized SmartArray
Special int array filling...
Special int array: 42 42 42 42 42 
```

## 使用场景总结

1. **类型模板参数**：
   - 通用容器类（`vector`, `list`）
   - 通用算法（`sort`, `find`）
   - 类型擦除技术

2. **非类型模板参数**：
   - 固定大小容器（`std::array`）
   - 数学计算模板（矩阵大小）
   - 策略模式中的编译期策略选择

3. **模板特化**：
   - 优化特定类型的性能（如 `std::vector<bool>`）
   - 为特定类型提供特殊行为
   - 实现类型特征（type traits）

理解这些概念的区别和适用场景，能够帮助您更有效地使用 C++ 模板元编程技术，创建更灵活高效的代码。
