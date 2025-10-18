---
title: C++函数详解及案例实现笔记
published: 2025-10-18
description: ''
image: ''
tags: [c++, 笔记]
category: '笔记'
draft: false 
lang: ''
---
# C++ 函数

函数（Functions）是一段封装了特定任务、可被重复调用的代码块。它们是实现 **代码复用**、**模块化** 和 **抽象** 的核心工具，能让我们的程序结构更清晰、更易于维护。

## Part 1: 函数的核心理念 (The "Why")

想象一下，如果没有函数，你的 `main` 函数将包含成百上千行代码，逻辑混乱不堪。函数解决了以下关键问题：

*   **`🔄` 代码复用 (Reusability)**: 无需重复编写相同的代码。一次编写，随处调用。
*   **`📦` 模块化 (Modularity)**: 将复杂的程序分解成一个个独立的、易于管理的小模块。每个函数负责一项单一的任务。
*   **`🎭` 抽象 (Abstraction)**: 调用者只需知道函数 *做什么*（它的接口），而无需关心其内部 *怎么做*（它的实现细节）。

## Part 2: 函数的解剖学 (The "How")

### 2.1 声明与定义

这是C++中一个至关重要的区别。

*   **函数声明 (Declaration / Prototype)**: 告诉编译器一个函数“长什么样”。它指定了函数的返回类型、名称和参数列表，但不包含函数体。声明以分号结束。
    > **目的**: 让其他代码在不知道函数具体实现的情况下也能合法地调用它。通常放在头文件 (`.h`) 中。
*   **函数定义 (Definition / Implementation)**: 提供了函数的实际代码实现（函数体）。
    > **目的**: 真正实现函数的功能。通常放在源文件 (`.cpp`) 中。

```cpp
// 1. 函数声明 (Prototype)
// 告诉编译器：有一个名为 add 的函数，它接受两个int，返回一个int
int add(int a, int b);

// main 函数可以调用 add，因为它“认识”它了
int main() {
    int result = add(5, 3); // 合法调用
    return 0;
}

// 2. 函数定义 (Implementation)
// 在程序的其他地方，我们提供 add 的具体实现
int add(int a, int b) {
    return a + b;
}
```

### 2.2 函数的构成要素

```cpp
return_type function_name(parameter_list) {
    // ---- Function Body ----
    // statements...
    // return value;
    // -----------------------
}
```
*   **`return_type`**: 函数执行完毕后返回给调用者的值的类型。如果函数不返回任何值，则使用 `void`。
*   **`function_name`**: 函数的唯一标识符。
*   **`parameter_list`**: 传递给函数的数据，也称为“参数”。每个参数都像一个在函数内部声明的局部变量。
*   **`Function Body`**: 包含在大括号 `{}` 内的代码，是函数的具体实现。

## Part 3: 数据传递的艺术 — 参数传递

如何将数据传入函数是至关重要的，C++提供了多种方式：

### 3.1 按值传递 (Pass-by-Value)

> **原理**: 将实参的 **副本** 传递给函数。函数内部对参数的任何修改 **不会** 影响到原始的实参。

```cpp
void increment(int x) {
    x++; // 修改的是 x 的副本
    std::cout << "Inside function, x = " << x << std::endl;
}

int main() {
    int num = 10;
    increment(num);
    std::cout << "Outside function, num = " << num << std::endl;
}
// 输出:
// Inside function, x = 11
// Outside function, num = 10
```
*   **优点**: 安全，不会意外修改外部数据。
*   **缺点**: 对于大型对象（如 `struct`, `class`, `std::vector`），创建副本的开销可能很大。

### 3.2 按引用传递 (Pass-by-Reference)

> **原理**: 将实参的 **别名** (alias) 传递给函数。函数内部对参数的操作 **会直接** 影响到原始的实参。通过在类型后加 `&` 来实现。

```cpp
void increment_ref(int& x) { // x 是 num 的别名
    x++; // 直接修改原始的 num
}

int main() {
    int num = 10;
    increment_ref(num);
    std::cout << "Outside function, num = " << num << std::endl;
}
// 输出:
// Outside function, num = 11
```
*   **优点**: 高效（没有拷贝开销），且能让函数修改外部数据。
*   **缺点**: 可能会意外修改数据，降低代码安全性。

### 3.3 按指针传递 (Pass-by-Pointer)

> **原理**: 将实参的 **内存地址** 传递给函数。函数通过解引用指针来访问和修改原始数据。

```cpp
void increment_ptr(int* p) { // p 存储了 num 的地址
    if (p != nullptr) {
        (*p)++; // 解引用指针，修改原始的 num
    }
}

int main() {
    int num = 10;
    increment_ptr(&num); // 传递 num 的地址
    std::cout << "Outside function, num = " << num << std::endl;
}
// 输出:
// Outside function, num = 11
```

### ⭐ 现代C++最佳实践：`const` 引用传递

> 结合了 **按值传递的安全性** 和 **按引用传递的高效性**。这是传递大型对象作为只读输入的 **首选方式**。

```cpp
// 传入一个大型对象 vector
void print_vector(const std::vector<int>& vec) {
    // & : 避免了昂贵的拷贝
    // const : 保证函数不会修改原始的 vector，非常安全
    // vec.push_back(99); // 编译错误！不能修改 const 引用
    for (int item : vec) {
        std::cout << item << " ";
    }
}
```

## Part 4: 获取函数结果 — 返回值

### 4.1 基本返回

使用 `return` 关键字将一个值返回给调用者。返回值的类型必须与函数的 `return_type` 兼容。

### 4.2 返回多个值

函数本身只能有一个返回类型，但有多种技巧可以实现返回多个值的效果：
1.  **返回 `struct` 或 `class`**: 将多个值封装在一个对象中返回。
2.  **返回 `std::pair` 或 `std::tuple`**: 用于返回2个或多个异构值。
3.  **使用输出参数**: 通过传递引用或指针来“返回”值。

```cpp
#include <tuple>

// 使用 std::tuple 返回多个值
std::tuple<int, double> get_stats(const std::vector<int>& data) {
    int sum = 0;
    for(int x : data) sum += x;
    double avg = static_cast<double>(sum) / data.size();
    return std::make_tuple(sum, avg);
}
```

### ⚠️ 陷阱：返回悬挂引用 (Dangling Reference)

**绝对不要** 返回对函数内部局部变量的引用或指针。当函数结束时，其局部变量的内存会被销毁，返回的引用/指针将指向无效的内存。

```cpp
// ❌ 严重错误！
int& get_dangling_reference() {
    int local_var = 10;
    return local_var; // local_var 在函数返回后立即销毁！
}
```

## Part 5: 高级函数特性

### 5.1 默认参数 (Default Arguments)

允许在函数声明中为参数指定默认值。调用时如果未提供该参数，则使用默认值。

```cpp
void show_message(const std::string& msg, const std::string& level = "INFO") {
    std::cout << "[" << level << "] " << msg << std::endl;
}

show_message("File not found.", "WARNING"); // [WARNING] File not found.
show_message("Operation successful.");   // [INFO] Operation successful.
```

### 5.2 函数重载 (Function Overloading)

允许在同一作用域内定义多个 **同名** 函数，只要它们的 **参数列表不同** (参数个数或类型不同)。

```cpp
void print(int i) { std::cout << "Printing an int: " << i << std::endl; }
void print(double d) { std::cout << "Printing a double: " << d << std::endl; }
void print(const std::string& s) { std::cout << "Printing a string: " << s << std::endl; }

print(10);
print(3.14);
print("hello");
```

### 5.3 Lambda 表达式 (Lambda Expressions, C++11+)

一种创建 **匿名函数** (没有名字的函数) 的便捷方式。在与STL算法结合使用时威力巨大。

> **基本语法**: `[capture_clause](parameters) -> return_type { body }`

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};
int count = std::count_if(numbers.begin(), numbers.end(),
    // 这就是一个 Lambda 表达式
    [](int n) { return n > 2; }
);
// count 的结果是 3
```

---

## Part 6: 综合案例实现 — 文本分析器

这个案例将把前面所有的知识点融会贯通，创建一个简单的文本分析工具，它将被分解成多个功能独立的函数。

**目标**: 分析一段文本，并报告其字符数、单词数以及每个字符的出现频率。

### 完整代码

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <map>      // 用于存储字符频率
#include <cctype>   // 用于 isspace()

// --- 1. 函数声明 (Prototypes) ---
// 将所有功能拆分成独立的函数

// 计算单词数量 (按引用传递，高效且可修改)
int countWords(const std::string& text);

// 获取每个字符的出现频率 (返回一个复杂的数据结构)
std::map<char, int> getCharacterFrequencies(const std::string& text);

// 打印最终的分析报告 (void返回类型，const&传递)
void printAnalysisReport(const std::string& text,
                         int wordCount,
                         const std::map<char, int>& frequencies);

// --- 2. main 函数: 程序的主入口和流程控制器 ---
int main() {
    // 我们的主函数现在非常干净，只负责协调和调用
    std::string document = "Hello C++ world! C++ is a powerful language.";
    std::cout << "Analyzing document:\n\"" << document << "\"\n\n";

    // 调用功能函数来完成具体工作
    int words = countWords(document);
    std::map<char, int> freqs = getCharacterFrequencies(document);

    // 调用打印函数来显示结果
    printAnalysisReport(document, words, freqs);

    return 0;
}

// --- 3. 函数定义 (Implementations) ---

/**
 * @brief 计算字符串中的单词数。
 * @param text 用于分析的文本 (const& 传递，高效且安全).
 * @return 单词的数量.
 */
int countWords(const std::string& text) {
    if (text.empty()) {
        return 0;
    }

    int count = 0;
    bool inWord = false;
    for (char c : text) {
        if (std::isspace(c)) {
            inWord = false;
        } else if (!inWord) {
            inWord = true;
            count++;
        }
    }
    return count;
}

/**
 * @brief 计算每个字符在字符串中出现的次数.
 * @param text 用于分析的文本.
 * @return 一个 map，键是字符，值是出现次数.
 */
std::map<char, int> getCharacterFrequencies(const std::string& text) {
    std::map<char, int> frequencies;
    for (char c : text) {
        // 忽略空格
        if (!std::isspace(c)) {
            frequencies[c]++;
        }
    }
    return frequencies;
}

/**
 * @brief 打印格式化的分析报告.
 * @param text 原始文本.
 * @param wordCount 已计算的单词数 (按值传递，因为int很小).
 * @param frequencies 字符频率的 map (const& 传递，避免拷贝).
 */
void printAnalysisReport(const std::string& text,
                         int wordCount,
                         const std::map<char, int>& frequencies) {
    std::cout << "--- Text Analysis Report ---\n";
    std::cout << "Total Characters (excluding spaces): " << frequencies.size() << std::endl;
    std::cout << "Total Words: " << wordCount << std::endl;

    std::cout << "\nCharacter Frequencies:\n";
    for (const auto& pair : frequencies) {
        std::cout << "'" << pair.first << "': " << pair.second << "\n";
    }
    std::cout << "--- End of Report ---\n";
}
```

### 案例分析

1.  **模块化**: 我们将复杂的“文本分析”任务分解为 `countWords`, `getCharacterFrequencies`, `printAnalysisReport` 三个独立的、职责单一的函数。
2.  **抽象**: `main` 函数无需关心如何计算单词或频率，它只需调用相应的函数并传递数据即可。
3.  **参数传递**:
    *   `const std::string& text`: `const` 引用被广泛用于传递文本，因为它既高效（避免了字符串的深拷贝）又安全（保证函数不会修改原始文本）。
    *   `int wordCount`: `int` 是一个小类型，直接 **按值传递** 既简单又高效。
    *   `const std::map<char, int>& frequencies`: `map` 是一个复杂的数据结构，使用 **`const` 引用** 传递可以避免整个 `map` 的昂贵拷贝。
4.  **返回值**:
    *   `countWords` 返回一个简单的 `int`。
    *   `getCharacterFrequencies` 展示了如何返回一个复杂的STL容器 `std::map`。
    *   `printAnalysisReport` 是一个 `void` 函数，它的任务是执行一个动作（打印），而不是计算一个值。
5.  **声明与定义**: 我们在 `main` 函数之前提供了所有函数的 **声明**，使得 `main` 可以合法地调用它们，而函数的 **定义** 则放在了后面，代码结构非常清晰。