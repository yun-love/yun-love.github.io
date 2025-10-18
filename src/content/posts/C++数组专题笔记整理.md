---
title: C++数组(专题)笔记整理
published: 2025-10-18
description: ''
image: ''
tags: [c++, 笔记]
category: '笔记'
draft: false 
lang: ''
---

# C++ 数组

数组是一种将 **相同类型** 的元素存储在 **一块连续内存** 中的数据结构。这种内存布局使得通过索引进行快速的随机访问成为可能，是其核心优势。

## Part 1: C风格数组 (C-Style Arrays) — 基础与原理

这是C++从C语言继承而来的最原始的数组形式。

### 1.1 定义与语法

> **语法结构**
> ```cpp
> type array_name[array_size];
> ```
> *   `type`: 数组中存储的元素的类型。
> *   `array_name`: 数组的名称。
> *   `array_size`: 数组的大小，**必须是一个编译时常量**。

#### 声明与初始化
```cpp
// 1. 声明一个未初始化的数组
int scores[5]; // 在内存中分配了5个int的空间，值是未定义的

// 2. 声明并完整初始化
double prices[4] = {9.99, 15.0, 7.50, 4.99};

// 3. 声明并部分初始化
// 未指定的值会被自动初始化为0（或对应类型的零值）
int values[10] = {100, 200}; // values[0]=100, values[1]=200, 其余8个元素为0

// 4. 自动推断大小
char message[] = "Hello"; // 编译器会自动计算大小为6 (包括末尾的'\0'空字符)
```

### 1.2 内存原理：连续的力量

当你声明 `int arr[3];` 时，计算机会在内存中寻找一块足以容纳3个 `int` 的 **连续** 空间。
```
  Memory Address:  | 0x1000 | 0x1004 | 0x1008 | ...
  Content:         | arr[0] | arr[1] | arr[2] | ...
```
*   **`arr` 本身**: 代表整个数组，也常常被视为指向数组第一个元素 `arr[0]` 的地址。
*   **快速访问**: 访问 `arr[2]` 时，计算机会直接计算地址 `(arr的起始地址) + 2 * sizeof(int)`，一步到位，这就是为什么数组的随机访问时间复杂度是 O(1)。

### 1.3 C风格数组的致命缺陷

尽管基础，但C风格数组存在许多问题，是现代C++程序中bug的主要来源之一。

*   ** 没有边界检查 (No Bounds Checking)**
    ```cpp
    int arr[3] = {1, 2, 3};
    arr[5] = 99; // 严重错误！这会写入数组边界之外的内存
                 // 编译器不会报错，但会在运行时导致未定义行为（破坏数据、程序崩溃等）
    ```
*   ** 数组到指针的“退化” (Pointer Decay)**
    这是最核心也是最令人困惑的特性。当数组名被用于大多数表达式中时（例如传递给函数），它会“退化”为一个指向其首元素的指针。
    ```cpp
    void print_size(int arr[]) { // 实际上传入的是 int*
        // 在函数内部，sizeof(arr) 计算的是指针的大小（通常是4或8字节），
        // 而不是整个数组的大小！
        std::cout << "Size inside function: " << sizeof(arr) << std::endl;
    }

    int main() {
        int my_array[10];
        std::cout << "Size in main: " << sizeof(my_array) << std::endl; // 输出 40 (10 * 4)
        print_size(my_array); // 输出 8 (在64位系统上)
    }
    ```
*   ** 大小固定**
    数组的大小必须在编译时确定，无法在运行时动态改变。

---

## Part 2: 现代C++的救赎 — `std::array`

C++11引入了 `std::array`，它是对C风格数组的 **轻量、安全的封装**。

> **头文件**: `#include <array>`

### 2.1 定义与语法
```cpp
#include <array>

// 语法: std::array<Type, Size> name;
std::array<int, 5> my_std_array = {10, 20, 30, 40, 50};
```

### 2.2 `std::array` 的核心优势

| 特性 | 描述 | 示例 |
| :--- | :--- | :--- |
| **`✅` 知道自身大小** | 内置 `.size()` 成员函数，传递给函数时不会丢失大小信息。 | `my_std_array.size()` |
| **`✅` 安全的边界检查** | 提供 `.at(index)` 成员函数，它会进行边界检查，越界则抛出异常。 | `my_std_array.at(2)` |
| **`✅` 完整的容器接口** | 支持迭代器 (`.begin()`, `.end()`)，可以无缝与STL算法配合使用。 | `std::sort(arr.begin(), arr.end())` |
| **`✅` 不会退化为指针**| 作为一个对象，它在函数传递时可以被完整地复制或引用。 | `void func(const std::array<int,5>& arr)` |

```cpp
#include <array>
#include <iostream>
#include <algorithm>

void process_array(const std::array<int, 4>& arr) {
    std::cout << "Array size in function: " << arr.size() << std::endl;
}

int main() {
    std::array<int, 4> nums = {3, 1, 4, 2};
    process_array(nums); // 大小信息被完整保留

    // 安全访问
    try {
        nums.at(5) = 99;
    } catch (const std::out_of_range& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    }

    // 与STL算法结合
    std::sort(nums.begin(), nums.end());
    for(int n : nums) std::cout << n << " "; // 输出 1 2 3 4
}
```

> ** 何时使用 `std::array`?**
> 当你需要在栈上创建一个 **编译时大小固定** 的数组时，`std::array` 应该是你的 **默认选择**。它兼具C风格数组的性能和现代C++的安全性。

---

## Part 3: 终极武器 — `std::vector` (动态数组)

`std::vector` 是C++中最灵活、最常用的序列容器。虽然严格来说它不是一个“数组”，但在实践中，它是 **C风格动态数组的完美替代品**。

> **头文件**: `#include <vector>`

### 3.1 核心特性：动态
`std::vector` 的大小可以在运行时 **动态改变**，它会自动处理内存的分配和释放。

### 3.2 语法与常用操作
```cpp
#include <vector>
#include <iostream>

int main() {
    // 创建一个空的vector
    std::vector<int> dynamic_array;

    // 添加元素
    dynamic_array.push_back(10); // [10]
    dynamic_array.push_back(20); // [10, 20]
    dynamic_array.push_back(30); // [10, 20, 30]

    // 访问元素 (同样支持 .at() 和 [])
    std::cout << "Element at index 1: " << dynamic_array[1] << std::endl;

    // 获取当前大小
    std::cout << "Current size: " << dynamic_array.size() << std::endl;

    // 移除最后一个元素
    dynamic_array.pop_back(); // [10, 20]

    // 遍历
    for (const auto& val : dynamic_array) {
        std::cout << val << " ";
    }
}
```

> ** 何时使用 `std::vector`?**
> 当你需要一个数组，但其 **大小在编译时未知**，或者需要在程序运行期间 **动态增删元素** 时，`std::vector` 是不二之选。

---

## Part 4: 多维数组

### 4.1 C风格多维数组

```cpp
// 2行3列的矩阵
int matrix[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};

// 内存布局仍然是连续的： 1, 2, 3, 4, 5, 6
```
> ** 性能提示**: 遍历多维数组时，遵循其内存布局（按行遍历）可以最大化缓存命中率，带来巨大性能提升。

### 4.2 现代C++多维数组

*   **大小固定**: `std::array<std::array<int, 3>, 2> matrix;`
*   **大小动态**: `std::vector<std::vector<int>> matrix;`
    > 这种方式非常灵活，可以创建“锯齿数组”（每行长度不同），但缺点是数据不再保证是单一连续的内存块，可能带来性能开销。

---

## Part 5: 总结与决策指南

| 特性 | C-Style Array | `std::array` | `std::vector` |
| :--- | :--- | :--- | :--- |
| **大小** | **编译时**固定 | **编译时**固定 | **运行时**动态 |
| **内存位置** | 栈 / 静态区 | 栈 / 对象内 | 堆 |
| **边界检查** | ❌ **无** | ✅ **有** (`.at()`) | ✅ **有** (`.at()`) |
| **大小信息** | ❌ **会退化**为指针 | ✅ **保留** (`.size()`) | ✅ **保留** (`.size()`) |
| **STL兼容性**| 差 (需手动传递大小) | ✅ **完美** | ✅ **完美** |
| **性能** | 极高 (无开销) | 极高 (零成本抽象) | 极高 (但动态扩容有成本) |
| **何时使用?**| 遗留代码，或与C库交互 | **大小固定的数组首选** | **大小动态的数组首选** |