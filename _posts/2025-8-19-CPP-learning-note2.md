---
layout: post
author: Dragroo
title: CPP学习笔记2
tag: CPP
catergory: 技术
---

## struct
### struct初始化

```cpp
struct Person {
    std::string name;
    int age;
    double height;
    Person(std::string m_name, int m_age, double m_height):name(m_name),age(m_age),height(m_height){}
};
Person p1 = {"Alice", 25, 165.5}; // 按顺序初始化所有成员
Person p2{"Jack", 21, 167.0};
Person p3("Amy",24,178.0);
```
### 别名的使用

```cpp

struct Person {
    std::string name;
    int age;
    double height;
    Person(std::string m_name, int m_age, double m_height):name(m_name),age(m_age),height(m_height){}
};
//方式1
typedef Person anotherPerson;

//方式2
typedef struct {
    std::string name;
    int age;
    double height;
    Person(std::string m_name, int m_age, double m_height):name(m_name),age(m_age),height(m_height){}
} person1;

//方式3
using person2 = struct {
    std::string name;
    int age;
    double height;
    Person(std::string m_name, int m_age, double m_height):name(m_name),age(m_age),height(m_height){}
};

```
### 一些与类的区别

在C++中，结构体与有自己的构造函数，析构函数，成员函数，也可以设置访问属性（`private`/`protected`/`public`），主要的区别就是struct默认`public`属性，而类默认`private`属性

## using namespace

### 问题
如果两个命名空间有相同的变量、函数等，当调用相关的内容时，会引发冲突

### 解决办法
1. 尽量不使用using namespace
```cpp
#include <iostream>
int main(){
    std::cout<<"Hello World"<<std::endl;
}
```

2. 可以将需要使用的内容用`using`单独列出来，直接指出来使用的是哪个`namespace`的内容
```cpp
#include <iostream>
using std::cout;
using std::endl;

int main(){
    cout<<"Hello World"<<endl;
}
```

### 头文件不应包含using namespace 声明
因为某个文件可能包含不同的头文件，如果使用using可能产生冲突