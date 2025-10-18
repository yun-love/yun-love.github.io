---
title: C++循环语句笔记整理
published: 2025-10-18
description: ''
image: ''
tags: [c++, 笔记]
category: '笔记'
draft: false 
lang: ''
---

# C++ 循环语句

循环是程序控制流的支柱，赋予我们高效处理重复性任务的能力。本笔记将从基础语法到高级应用，全面剖析C++的循环机制，助您写出更优雅、高效且健壮的代码。

## Part 1: 四大核心循环结构

这是C++循环的基础，每一种都有其最适用的场景。

### 1.1 `for` 循环

最经典、最通用的循环结构，尤其适用于 **迭代次数已知** 的情况。

> **语法结构**
> ```cpp
> for (initialization; condition; update) {
>     // 循环体代码...
> }
> ```
> *   `initialization`: **初始化**。仅在循环开始前执行一次。
> *   `condition`: **条件判断**。在每次迭代开始前检查。
> *   `update`: **更新**。在每次迭代结束后执行。

#### 多样化示例
```cpp
// 示例 1: 基本计数 (0 to 4)
for (int i = 0; i < 5; ++i) {
    std::cout << i << " ";
}

// 示例 2: 遍历数组
int arr[] = {10, 20, 30};
for (int i = 0; i < 3; ++i) {
    std::cout << "arr[" << i << "] = " << arr[i] << std::endl;
}
```

> **最佳实践**
> *   在循环的初始化部分声明计数器变量，以限制其作用域。
> *   优先使用前缀递增 `++i` 而不是后缀 `i++`。对于内置类型性能无差，但对于复杂类型（如迭代器），`++i` 效率更高。

### 1.2 `while` 循环

适用于 **迭代次数未知**，循环的持续依赖于一个动态变化的条件。

> **语法结构**
> ```cpp
> while (condition) {
>     // 循环体代码...
> }
> ```
> > **执行流程**: 先判断条件，若为`true`，则执行循环体，如此往复。

#### 示例
```cpp
// 示例: 用户输入验证
int number = 0;
while (number <= 0) {
    std::cout << "Enter a positive number: ";
    std::cin >> number;
}
```

> **注意**
> 必须确保循环体内有能够最终改变 `condition` 的逻辑，否则极易导致 **无限循环**。

### 1.3 `do-while` 循环

`while` 循环的变体，其核心特点是 **循环体至少会被执行一次**。

> **语法结构**
> ```cpp
> do {
>     // 循环体代码...
> } while (condition); // 注意：这里的分号必不可少！
> ```
> > **执行流程**: 先执行一次循环体，然后再判断条件。

#### 示例
```cpp
// 示例: 菜单驱动程序
char choice;
do {
    std::cout << "--- MENU ---\n";
    std::cout << "P) Play\n";
    std::cout << "Q) Quit\n";
    std::cout << "Enter your choice: ";
    std::cin >> choice;
} while (choice != 'Q' && choice != 'q');
```

### 1.4 范围 `for` 循环 (Range-based `for`)

自C++11起，这是遍历 **整个序列** (如数组、`std::vector`等) 的首选方式。它更简洁、可读性更高，并能从根本上杜绝索引越界错误。

> **语法结构**
> ```cpp
> for (declaration : range) {
>     // 循环体代码...
> }
> ```

#### 示例与最佳实践
```cpp
#include <vector>
#include <string>

std::vector<std::string> names = {"Alice", "Bob", "Charlie"};

// 实践 1: 只读访问 (最高效、最安全)
// 使用 const auto& 避免不必要的拷贝，并防止意外修改
std::cout << "Names: ";
for (const auto& name : names) {
    std::cout << name << " ";
}
std::cout << std::endl;


// 实践 2: 修改容器内元素
// 使用 auto& 获取元素的引用
std::vector<int> scores = {88, 95, 76};
for (auto& score : scores) {
    score += 5; // 为每个分数加5分
}
```

---

## Part 2: 精通循环控制

### 2.1 `break`: 紧急出口
立即终止其所在的 **最内层** 循环或 `switch` 语句。

```cpp
// 示例: 查找第一个能被7整除的数
for (int i = 1; i <= 100; ++i) {
    if (i % 7 == 0) {
        std::cout << "Found it: " << i << std::endl;
        break; // 任务完成，立即跳出循环
    }
}
```

### 2.2 `continue`: 跳过本轮
立即结束 **当前迭代**，直接进入循环的下一次迭代判断。

```cpp
// 示例: 计算所有正数的和
int numbers[] = {10, -5, 20, 0, -15, 30};
int sum = 0;
for (int n : numbers) {
    if (n <= 0) {
        continue; // 如果不是正数，则跳过本次累加
    }
    sum += n;
}
```

### 2.3 `goto` 的困境
`goto` 提供到程序中标签位置的无条件跳转。

> **强烈不推荐**
> 使用 `goto` 会使程序流程变得混乱，难以阅读、调试和维护，形成所谓的“意大利面条式代码”。在现代C++中，几乎所有场景都可以被 `break`, `continue`, `return` 或更好的程序结构（如函数）替代。

---

## Part 3: 高级技术与现代C++实践

### 3.1 嵌套循环：处理多维数据

当循环内部包含另一个循环时，用于处理矩阵、打印图形等场景。

```cpp
// 示例: 遍历二维矩阵
int matrix[2][3] = {{1, 2, 3}, {4, 5, 6}};

for (int i = 0; i < 2; ++i) { // 外层循环遍历行
    for (int j = 0; j < 3; ++j) { // 内层循环遍历列
        std::cout << matrix[i][j] << "\t";
    }
    std::cout << std::endl;
}
```
> **注意**: `break` 和 `continue` 只会影响其所在的 **最内层** 循环。

### 3.2 超越循环：STL算法的威力

手动编写循环有时是冗长且易错的。STL提供了大量算法，能以 **声明式** 的方式表达意图，使代码更清晰、更安全。

```cpp
#include <algorithm> // 关键头文件
#include <vector>

std::vector<int> vec = {1, 2, 3, 4, 5, 6};

// 传统循环查找
int target = -1;
for (int x : vec) {
    if (x > 3 && x % 2 == 0) {
        target = x;
        break;
    }
}

// STL算法替代
auto it = std::find_if(vec.begin(), vec.end(), [](int x) {
    return x > 3 && x % 2 == 0; // 查找第一个大于3的偶数
});

if (it != vec.end()) {
    target = *it;
}
```

> **优势**: 代码更关注 **“做什么”** 而非 **“怎么做”**，意图一目了然，且库的实现通常经过高度优化。

---

## Part 4: 性能、陷阱与专业级考量

### 4.1 性能调优：让循环快如闪电

####  **缓存局部性 (Cache Locality)**
CPU访问缓存的速度远超主存。以线性、连续的方式访问内存可以最大化缓存命中率，带来巨大的性能提升。

```cpp
const int SIZE = 2048;
static int matrix[SIZE][SIZE];

// ✅ 方式1: 缓存友好 (速度极快)
// 按行主序访问，内存地址连续
for (int i = 0; i < SIZE; ++i) {
    for (int j = 0; j < SIZE; ++j) {
        matrix[i][j]++;
    }
}

// ❌ 方式2: 缓存灾难 (速度极慢)
// 按列访问，内存地址大幅跳跃，导致缓存频繁失效
for (int j = 0; j < SIZE; ++j) {
    for (int i = 0; i < SIZE; ++i) {
        matrix[i][j]++;
    }
}
```

### 4.2 常见陷阱

####  **迭代器失效 (Iterator Invalidation)**
在遍历容器时，若执行了改变容器结构的操作（如在`vector`中`insert`或`erase`），可能会导致迭代器失效，引发 **未定义行为** (通常是程序崩溃)。

```cpp
std::vector<int> v = {1, 2, 3, 4, 5, 6};

// ❌ 错误做法
// for (auto it = v.begin(); it != v.end(); ++it) {
//     if (*it % 2 == 0) v.erase(it); // 严重错误！it 在此失效！
// }

// ✅ 正确的删除方式 (C++11+)
for (auto it = v.begin(); it != v.end(); /* 此处无递增 */) {
    if (*it % 2 == 0) {
        it = v.erase(it); // erase会返回下一个有效迭代器
    } else {
        ++it;
    }
}
```

####  **浮点数作为循环条件**
由于精度问题，浮点数的比较是不可靠的。

```cpp
// ❌ 危险: 可能因精度误差导致无限循环或次数不符
// for (float x = 0.0f; x != 1.0f; x += 0.1f) { ... }

// ✅ 安全: 使用整数计数
for (int i = 0; i < 10; ++i) {
    float x = i * 0.1f;
    // ...
}
```

#### 🌀 **无符号整数的反向循环**
这是一个经典的陷阱。当一个`unsigned`变量(如`size_t`)为0时，再`--`会使其“下溢”成一个极大的正数。

```cpp
std::vector<int> items = {1, 2, 3};
// ❌ 错误！无限循环！
// for (size_t i = items.size() - 1; i >= 0; --i) {
//     // 当 i=0 时，下一次 --i 会使 i 变成 unsigned 的最大值，i>=0 恒为真
// }

// ✅ 正确写法 1: 使用有符号整数
for (int i = items.size() - 1; i >= 0; --i) { /* ... */ }

// ✅ 正确写法 2: 使用反向迭代器 (推荐)
for (auto it = items.rbegin(); it != items.rend(); ++it) { /* ... */ }
```

---

## Part 5: 总结与决策指南

| 循环类型 | 何时选择？ | 优点 | 注意事项 |
| :--- | :--- | :--- | :--- |
| **范围 `for`** | **遍历容器或数组的全部元素** | 最简洁、最安全、意图最明确 | 不适用于需要索引或部分遍历的场景 |
| **`for`** | **迭代次数已知**或需要精细控制索引 | 结构清晰，控制灵活 | 容易出现“差一错误”，代码稍显冗长 |
| **`while`** | **迭代次数未知**，循环依赖于一个先决条件 | 逻辑简单，适用于不确定次数的场景 | 容易造成无限循环 |
| **`do-while`**| **循环体至少需要执行一次** | 保证至少执行一次 | 使用场景相对较少，注意结尾分号 |
| **STL 算法** | 当循环逻辑符合标准模式时（查找、转换等）| 代码更具表现力，潜在性能更好 | 需要熟悉STL库 |