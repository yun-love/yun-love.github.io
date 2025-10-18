---
title: C++指针详解及案例实现笔记
published: 2025-10-18
description: ''
image: ''
tags: [c++, 笔记]
category: '笔记'
draft: false 
lang: ''
---

# C++ 指针

## Part 1: 指针的本质 — “内存地址的门牌号”

想象一下，计算机的内存是一条无比巨大的街道，街道上有很多房子，每个房子都有一个唯一的 **门牌号**，这个门牌号就是 **内存地址**。

*   **变量**: 一个变量（比如 `int num = 10;`）就像是在某个房子里存放的物品（数字10）。这个房子本身也有一个门牌号。
*   **指针**: **指针本身也是一个变量**，但它的特殊之处在于，它里面存放的**不是普通物品，而是一个门牌号**。

> **指针的本质**: 一个专门用来存储其他变量 **内存地址** 的变量。

通过这个“门牌号”（地址），我们就可以间接地找到并操作那个房子里的物品。这就是指针的核心——**间接访问 (Indirection)**。

> **💡 为什么需要指针？**
> 1.  **`🚀` 性能与效率**: 直接传递大型对象的地址（几字节）远比复制整个对象（可能几兆字节）要快得多。
> 2.  **`🔧` 动态内存管理**: 程序在运行时，可以在需要时向操作系统“申请”一块内存（在堆上），并在用完后“归还”。这种操作必须通过指针来管理这块内存的地址。
> 3.  **`🔗` 实现高级数据结构**: 链表、树、图等数据结构，其节点之间的连接关系就是通过指针来维系的。
> 4.  **`💻` 底层交互**: 与硬件、操作系统API或C语言库交互时，指针是不可或缺的工具。

---

## Part 2: 指针的机械原理 — 语法与操作

掌握指针需要熟悉三个核心操作符：`*` (星号) 和 `&` (取地址符)。

### 2.1 声明指针
> **语法**: `type* pointer_name;`
> 星号 `*` 在这里表示“这是一个指针”。`type` 指明了这个指针将要指向的数据类型。

```cpp
int*   p_int;    // 一个将要指向 int 类型数据的指针
char*  p_char;   // 一个将要指向 char 类型数据的指针
Student* p_student; // 一个将要指向 Student 对象的指针
```
**类型安全**: C++是类型安全的。一个 `int*` 类型的指针只能指向 `int` 类型的变量，这可以防止许多潜在的错误。

### 2.2 获取地址 (`&`)
> `&` 操作符用于获取一个变量的内存地址。

```cpp
int score = 95;
// &score 的结果就是变量 score 在内存中的地址
// 将这个地址赋值给指针 p_score
int* p_score = &score;
```

### 2.3 解引用 (`*`) — 访问数据
> 当 `*` 用在**已经存在的指针**前面时，它就变成了 **解引用 (Dereference)** 或 **间接访问 (Indirection)** 操作符。它的意思是：“不要看指针本身的值（那个地址），而是去访问那个地址所指向的房子里的东西”。

```cpp
int score = 95;
int* p_score = &score;

// *p_score 的意思是 "访问 p_score 指向的地址上的数据"
std::cout << "Value via pointer: " << *p_score << std::endl; // 输出: 95

// 也可以通过指针修改原始数据
*p_score = 100;
std::cout << "Original value now: " << score << std::endl; // 输出: 100
```
> **⚠️ `*` 的双重含义**:
> *   在 **声明** 时 (`int* p;`)，`*` 表示这是一个指针。
> *   在 **使用** 时 (`*p = 10;`)，`*` 表示解引用。

### 2.4 `nullptr` — 安全的空指针 (C++11+)
一个未初始化的指针是“野指针”，它指向一个随机的内存地址，对其操作是极其危险的。为了表示一个指针当前“没有指向任何东西”，我们应该使用 `nullptr`。

```cpp
int* p = nullptr; // 明确地表示这是一个空指针，不指向任何有效地址

// 在使用指针前，检查它是否为空是一个非常好的习惯
if (p != nullptr) {
    // ... 安全地使用 p ...
}
```

---

## Part 3: 指针的核心应用场景

### 3.1 动态内存分配 (`new` 和 `delete`)

当程序需要在运行时才确定需要多少内存时（例如，用户输入的数组大小），就需要动态内存分配。这部分内存来自 **堆 (Heap)**。

*   **`new`**: 在堆上分配内存，并返回指向这块内存的指针。
*   **`delete`**: 释放由 `new` 分配的单个变量的内存。
*   **`delete[]`**: 释放由 `new[]` 分配的数组内存。

```cpp
// 分配一个 int
int* p_num = new int(42); // 在堆上创建一个int，值为42

// 分配一个大小为 N 的 int 数组
int N = 10;
int* p_array = new int[N];

// ... 使用 p_num 和 p_array ...
std::cout << *p_num << std::endl; // 输出 42
p_array[0] = 100;

// ！！！ 必须手动释放内存，否则会造成内存泄漏 ！！！
delete p_num;
delete[] p_array;
```
> **⭐ 现代C++实践**: 手动管理 `new`/`delete` 容易出错。在现代C++中，应优先使用 **智能指针 (`std::unique_ptr`, `std::shared_ptr`)** 来自动管理动态内存的生命周期。

### 3.2 指针与数组的亲密关系

在C++中，数组名在很多情况下会“退化”为指向其首元素的指针。

```cpp
int arr[5] = {10, 20, 30, 40, 50};
int* p = arr; // arr 退化为 &arr[0]

// 这两种访问方式等价
std::cout << arr[2] << std::endl;      // 输出 30
std::cout << *(p + 2) << std::endl;    // 输出 30
```
**指针算术**: 当你对一个指针 `p` 加 `n` (`p + n`)，它实际移动的内存距离是 `n * sizeof(*p)`。这就是为什么 `p+2` 能准确地指向第三个元素。

### 3.3 指针与函数

这是指针最强大的应用之一。
*   **高效传递大对象** (虽然`const&`更常用，但指针也能做到)。
*   **允许函数修改调用方的数据** (按引用传递的另一种方式)。
*   **返回动态分配的内存**。

```cpp
// 这个函数在堆上创建一个数组并返回它
int* create_random_array(int size) {
    int* arr = new int[size];
    for (int i = 0; i < size; ++i) {
        arr[i] = rand() % 100;
    }
    return arr;
}
```

---

## Part 4: 高级指针

*   **指针的指针 (`**`)**: 一个指针，它存储的是另一个指针的地址。
    ```cpp
    int x = 10;
    int* p1 = &x;
    int** p2 = &p1; // p2 指向 p1
    std::cout << **p2 << std::endl; // 两次解引用，得到 x 的值 10
    ```
*   **函数指针**: 一个指针，它存储的是一个函数的入口地址，允许你像调用变量一样调用函数。
    ```cpp
    int add(int a, int b) { return a + b; }
    int (*func_ptr)(int, int) = &add; // 声明并初始化函数指针
    int result = func_ptr(5, 3); // 通过指针调用函数
    ```

---

## Part 5: 综合案例实现 — 字符串原地逆序

这个案例将完美地展示指针的威力：直接、高效地在内存层面操作数据。我们将编写一个函数，它接受一个C风格字符串 (`char*`)，并在不分配额外内存的情况下将其原地逆序。

**核心思想**: 使用两个指针，一个指向字符串的开头 (`start`)，一个指向结尾 (`end`)。不断交换它们指向的字符，然后 `start` 向右移动，`end` 向左移动，直到它们相遇。

### 完整代码

```cpp
#include <iostream>
#include <cstring> // For strlen

// --- 1. 辅助函数：交换两个字符 ---
// 使用指针接收地址，直接修改原始字符
void swapChars(char* a, char* b) {
    if (a != nullptr && b != nullptr) {
        char temp = *a; // 解引用a，读取值
        *a = *b;      // 解引用a和b，进行赋值
        *b = temp;
    }
}

// --- 2. 核心函数：原地逆序字符串 ---
// 接受一个指向字符的指针（C风格字符串）
void reverseStringInPlace(char* str) {
    if (str == nullptr) {
        return; // 处理空指针，保证安全
    }

    int len = std::strlen(str);
    if (len < 2) {
        return; // 长度为0或1的字符串无需逆序
    }

    // [指针的应用] -> 初始化两个指针
    char* start = str;               // start 指向字符串的第一个字符
    char* end = str + len - 1;       // end 指向字符串的最后一个字符（\0之前）

    // [指针的应用] -> 循环条件是指针的比较
    while (start < end) {
        // [指针的应用] -> 将指针传递给另一个函数
        swapChars(start, end);

        // [指针的应用] -> 指针算术
        start++; // 指针向右移动一个char的位置
        end--;   // 指针向左移动一个char的位置
    }
}

// --- 3. main 函数：测试我们的实现 ---
int main() {
    // [指针的应用] -> 动态分配内存来存储字符串
    char* my_string = new char[50];
    std::strcpy(my_string, "Hello, Pointer!");

    std::cout << "Original string: " << my_string << std::endl;

    // 调用逆序函数
    reverseStringInPlace(my_string);

    std::cout << "Reversed string: " << my_string << std::endl;

    // [指针的应用] -> 释放动态分配的内存
    delete[] my_string;
    my_string = nullptr; // 好习惯：释放后置为nullptr

    // 测试空指针和短字符串
    char* empty_str = nullptr;
    reverseStringInPlace(empty_str); // 应该能安全处理
    std::cout << "Reversing a nullptr is safe." << std::endl;

    return 0;
}
```

### 案例分析

这个小小的程序几乎囊括了指针的所有核心操作：
1.  **声明与初始化**: `char* start = str;` 和 `char* end = ...` 初始化了两个指针。
2.  **指针算术**: `end = str + len - 1`，以及循环中的 `start++` 和 `end--`，完美展示了指针如何根据其类型大小进行移动。
3.  **指针比较**: `while (start < end)` 条件直接比较两个内存地址的高低来控制循环。
4.  **解引用**: `swapChars` 函数中的 `*a` 和 `*b` 通过解引用来实际交换内存中的字符值。
5.  **指针作为函数参数**: `reverseStringInPlace(char* str)` 和 `swapChars(char* a, char* b)` 都展示了如何通过传递指针来让函数操作外部数据。
6.  **动态内存管理**: `main` 函数中使用 `new[]` 创建字符串，并在最后使用 `delete[]` 释放，这是一个完整的动态内存生命周期管理示例。
7.  **空指针安全**: `reverseStringInPlace` 函数开始时的 `if (str == nullptr)` 检查，是编写健壮指针代码的典范。

通过这个例子，我们可以清晰地看到，指针不是一个抽象的理论，而是一个可以直接“触摸”和“修改”内存的强大、具体的工具。