1. [C+++的编译过程](https://zhuanlan.zhihu.com/p/618037867)
分为预编译、编译、汇编、链接
2. [inline那些事](https://light-city.github.io/stories_things/basic_content/inline/)
- 类中直接定义的是隐式内联函数，如果声明以后再定义，需要显示加上内联
- 内联函数实际上就是将调用函数的地方直接替换成函数体内部的语句，而不是执行时再去替换
- 和宏定义的区别？
主要体现在类型检查上，宏定义只是单纯的文本替换

### 宏定义示例

假设我们想要创建一个宏来计算两个数的最大值：

```c
#include <stdio.h>

#define MAX(a, b) (((a) > (b)) ? (a) : (b))

int main() {
    int x = 10;
    int y = 20;
    int z = MAX(x++, y);  // 注意这里的陷阱
    printf("The max value is: %d\n", z);
    return 0;
}
```

在这个例子中，`MAX`宏被定义为返回`a`和`b`中的较大值。然而，当我们用`x++`作为参数时，由于宏只是简单的文本替换，`x++`会被替换两次（一次在`(a) > (b)`中，一次在`(a) : (b)`中），导致结果不是我们预期的。实际上，这段代码会导致`x`被递增两次，这是宏的一个常见陷阱。

### 内联函数示例

现在，让我们使用内联函数来实现相同的功能：

```c
#include <stdio.h>

inline int max(int a, int b) {
    return (a > b) ? a : b;
}

int main() {
    int x = 10;
    int y = 20;
    int z = max(x++, y);  // 没有陷阱，x只递增一次
    printf("The max value is: %d\n", z);
    return 0;
}
```

在这个例子中，`max`是一个内联函数。当我们在`main`函数中调用`max(x++, y)`时，`x`只会递增一次，因为我们传递的是`x++`的值，而不是重复使用`x`。这是因为内联函数的行为类似于常规函数，但在调用点上进行了内联扩展。

### 总结

- **宏定义**：宏定义在预处理阶段进行文本替换，没有类型检查，容易出现意外行为，如上面例子中的`x++`被计算了两次。
- **内联函数**：内联函数在编译阶段进行内联展开，具有类型检查，行为更加可预测，避免了宏定义中的陷阱。

这些例子展示了为什么在现代编程实践中，通常推荐使用内联函数而非宏定义，特别是在需要类型安全的情况下。
3. [结构体的浅拷贝和深拷贝](https://blog.csdn.net/Ivan804638781/article/details/110913901)
主要针对成员有指针的情况，如果浅拷贝，复制的是指针的值，而并不是开辟新的内存空间，这样当另外一个结构体所占用的内存释放时，这个复制的指针指向的内存空间就是非法的，因此需要深拷贝，即开辟一块新的内存，再将对应的值复制过来
在C/C++中，当你将一个结构体赋值给另一个结构体时，使用“=”操作符，这通常是**浅拷贝**¹²。

**浅拷贝**：浅拷贝会逐个复制结构体的每个成员。如果结构体中包含指针，浅拷贝只会复制指针的值（即地址），而不会复制指针所指向的实际数据。这意味着两个结构体中的指针将指向同一块内存区域¹²。

**深拷贝**：深拷贝不仅复制结构体的每个成员，还会复制指针所指向的实际数据，从而在内存中创建独立的副本¹²。

例如：
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int id;
    char *name;
} Person;

int main() {
    Person p1;
    p1.id = 1;
    p1.name = malloc(20 * sizeof(char));
    strcpy(p1.name, "Alice");

    Person p2 = p1; // 浅拷贝
    printf("p2.id: %d, p2.name: %s\n", p2.id, p2.name);

    // 修改p1.name
    strcpy(p1.name, "Bob");
    printf("p1.name: %s, p2.name: %s\n", p1.name, p2.name); // p2.name也会变成"Bob"

    free(p1.name);
    return 0;
}
```
在这个例子中，`p2`是`p1`的浅拷贝，因此修改`p1.name`也会影响`p2.name`。

如果你有更多问题或需要进一步的解释，请告诉我！
¹: [GeeksforGeeks](https://www.geeksforgeeks.org/shallow-copy-and-deep-copy-in-c/)
²: [LearnCpp](https://www.learncpp.com/cpp-tutorial/shallow-vs-deep-copying/)

源: 与 Copilot 的对话， 2024/9/2
(1) Shallow Copy and Deep Copy in C++ - GeeksforGeeks. https://www.geeksforgeeks.org/shallow-copy-and-deep-copy-in-c/.
(2) 21.13 — Shallow vs. deep copying – Learn C++ - LearnCpp.com. https://www.learncpp.com/cpp-tutorial/shallow-vs-deep-copying/.
(3) Difference between Shallow and Deep copy of a class. https://www.geeksforgeeks.org/difference-between-shallow-and-deep-copy-of-a-class/.
(4) Shallow vs Deep Copying of Python Objects – Real Python. https://realpython.com/copying-python-objects/.
(5) Copy struct in C [Shallow & Deep Copy] - OpenGenus IQ. https://iq.opengenus.org/copy-struct-in-c/.

----------------------------------------------
在C/C++中，实现深拷贝通常需要手动编写代码来复制对象的所有成员，包括动态分配的内存。以下是一个示例，展示如何为包含指针的结构体实现深拷贝：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int id;
    char *name;
} Person;

// 深拷贝函数
Person deep_copy(const Person *src) {
    Person dest;
    dest.id = src->id;
    dest.name = malloc(strlen(src->name) + 1); // 分配内存
    strcpy(dest.name, src->name); // 复制字符串
    return dest;
}

int main() {
    Person p1;
    p1.id = 1;
    p1.name = malloc(20 * sizeof(char));
    strcpy(p1.name, "Alice");

    Person p2 = deep_copy(&p1); // 深拷贝
    printf("p2.id: %d, p2.name: %s\n", p2.id, p2.name);

    // 修改p1.name
    strcpy(p1.name, "Bob");
    printf("p1.name: %s, p2.name: %s\n", p1.name, p2.name); // p2.name 不会改变

    // 释放内存
    free(p1.name);
    free(p2.name);
    return 0;
}
```

在这个示例中，`deep_copy`函数为目标对象分配新的内存，并复制源对象的内容，从而实现深拷贝。这样，修改源对象的内容不会影响目标对象。

如果你有更多问题或需要进一步的解释，请告诉我！

源: 与 Copilot 的对话， 2024/9/2
(1) How to Make a Deep Copy of an Object in Java | Baeldung. https://www.baeldung.com/java-deep-copy.
(2) Shallow Copy and Deep Copy in C++ - GeeksforGeeks. https://www.geeksforgeeks.org/shallow-copy-and-deep-copy-in-c/.
(3) Deep Copy of an Object in C# - Code Maze. https://code-maze.com/csharp-deep-copy-of-object/.
(4) copy — Shallow and deep copy operations — Python 3.12.5 documentation. https://docs.python.org/3/library/copy.html.
(5) How to implement deep copy in golang-Golang-php.cn. https://www.php.cn/faq/526370.html.
4. [联合体union详解](https://www.cnblogs.com/wft1990/p/6700434.html)
所有成员都共享起始地址，并且联合体的大小要大于等于最大成员的大小，同时联合体的大小还要是每个成员大小的倍数。
5. [手写shared_ptr](https://csguide.cn/cpp/memory/shared_ptr.html#%E6%89%8B%E5%86%99-shared-ptr)
- 定义模版：template \<typename T\>
- 成员包含一个普通指针ptr，使用模版定义(T* ptr)；
- 一个指向记录引用计数的指针count；
- 一个构造函数，传一个普通指针进去（一般是通过new创建的类对象返回的指针），同时将引用计数的值设置为1
- 拷贝构造函数，传一个share_ptr(即自己创建的这个类)，将ptr和count复制过去，并对count指向的值加1
- 重载赋值符号，如果等号两边不是同一个shared_ptr，需要：
    * 执行release(),即对原有的count指向的值减1；如果减1后值为0，则使用delete，释放两个指针成员对应的内存
    * 复制新的ptr和count，count指向的值加1
- 析构函数，执行release()操作
- 还有下面几个函数：
    * T& operator*() const { return *ptr_; }
    * T* operator->() const { return ptr_; }
    * T* get() const { return ptr_; }
    * size_t use_count() const { return count_ ? *count_ : 0; }