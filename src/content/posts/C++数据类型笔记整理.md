---
title: C++数据类型笔记整理
published: 2025-10-18
description: ''
image: ''
tags: [c++, 笔记]
category: '笔记'
draft: false 
lang: ''
---

## C++ 数据类型综合深度解析

在C++中，数据类型是构建程序的基本骨架。它不仅告知编译器如何解释内存中的数据，还定义了可以对这些数据执行哪些操作。深刻理解C++的数据类型体系是编写高效、安全且可维护代码的基石。

### C++ 数据类型家族

C++ 的数据类型可分为三大核心类别：
1.  **基本数据类型 (Fundamental Data Types)**: C++语言内置的基础类型。
2.  **派生数据类型 (Derived Data Types)**: 基于基本类型构建的更复杂类型。
3.  **用户定义数据类型 (User-Defined Data Types)**: 由程序员根据需求自行创建的类型。

---

### 1. 基本数据类型 (Fundamental Data Types)

这些是C++的原子类型，是所有其他类型的基础。

#### 1.1 核心基本类型

| 类型 | 关键字 | 典型大小 | 典型范围/值 | 示例 |
| :--- | :--- | :--- | :--- | :--- |
| **整型** | `int`, `short`, `long`, `long long` | 4字节 (int) | -2,147,483,648 到 2,147,483,647 | `int userCount = 1000;` |
| **浮点型** | `float`, `double`, `long double` | 4字节 (float), 8字节 (double) | `float`约6-7位小数精度 | `float price = 19.99f;`<br>`double pi = 3.1415926535;` |
| **字符型** | `char` | 1字节 | -128 到 127 或 0 到 255 | `char grade = 'A';` |
| **布尔型** | `bool` | 1字节 | `true` 或 `false` | `bool isAvailable = true;` |
| **无类型**| `void` | N/A | 无 | `void logMessage();` |

**原理**: 当你声明一个变量时（如 `int a;`），编译器会在内存中预留一块与该类型大小相匹配的空间（`int`通常是4字节）。变量名`a`就成了这块内存空间的别名，你可以通过它来读写这块内存。

#### 1.2 类型修饰符与高级主题

修饰符可以改变基本类型的特性。

*   **`signed` / `unsigned`**:
    *   **介绍**: `unsigned` 只能表示非负整数，但其正数表示范围是对应`signed`类型的两倍。整型默认为`signed`。
    *   **示例 (常见陷阱)**:
        ```cpp
        unsigned int a = 5;
        int b = -10;

        // 警告：当有符号和无符号整数混合运算时，有符号数会被隐式转换为无符号数。
        // -10 会被转换成一个非常大的正整数，导致比较结果与直觉相反。
        if (a + b > 0) {
            // 这个分支会被执行！
            std::cout << "5 + (-10) > 0 is true because of implicit conversion." << std::endl;
        }
        ```
    *   **注意事项**: 强烈建议避免 `signed` 和 `unsigned` 类型的混合运算。应使用 `unsigned` 类型（如 `size_t`）来表示数量、大小或索引等永远不会为负的值。

*   **`const` (常量)**:
    *   **介绍**: `const` 关键字用于声明一个其值在初始化后不能被修改的实体。
    *   **多样化示例**:
        ```cpp
        const double PI = 3.14159; // 常量，不能修改

        int value = 10;
        // 指向常量的指针：不能通过指针修改值
        const int* ptr_to_const = &value;
        // *ptr_to_const = 20; // 编译错误!

        // 常量指针：指针自身的指向不能改变
        int* const const_ptr = &value;
        *const_ptr = 25; // 合法，可以修改指向的值
        // const_ptr = &another_value; // 编译错误!

        // 指向常量的常量指针：指针的指向和指向的值都不能改变
        const int* const const_ptr_to_const = &value;
        ```

*   **`volatile` (易变)**:
    *   **介绍**: 告知编译器，变量的值可能在程序代码的控制之外被意外更改（例如由硬件或另一线程）。这会阻止编译器对该变量的访问进行优化。
    *   **示例 (概念)**:
        ```cpp
        // 假设这是一个由硬件实时更新的状态寄存器地址
        volatile unsigned int* hardware_status = (unsigned int*)0x12345678;
        while (*hardware_status == 0) {
            // 等待硬件准备就绪。若无 volatile，编译器可能优化掉这个循环。
        }
        ```

#### 1.3 现代C++中的字符类型

为了更好地支持Unicode，C++11及以后版本引入了新的字符类型。

*   `char8_t` (C++20), `char16_t`, `char32_t`:
    *   **介绍**: 分别用于存储UTF-8, UTF-16, 和 UTF-32 编码的字符。
    *   **示例**:
        ```cpp
        const char8_t* utf8_str = u8"你好，世界！"; // u8前缀
        const char16_t* utf16_str = u"你好";       // u前缀
        const char32_t* utf32_str = U"世界";       // U前缀
        ```

---

### 2. 派生数据类型 (Derived Data Types)

这些类型是基于基本类型的组合或引用。

#### 2.1 数组 (Array)

*   **介绍**: 存储在连续内存位置的相同类型元素的集合。
*   **多样化示例**:
    ```cpp
    // 一维C风格数组
    int scores[] = {98, 87, 92, 79, 85};
    scores[0] = 100; // 修改第一个元素

    // 二维数组 (矩阵)
    int matrix[2][3] = { {1, 2, 3}, {4, 5, 6} };
    std::cout << matrix[1][2] << std::endl; // 输出 6
    ```
*   **注意事项**: C风格数组在传递给函数时会“退化”为指针，丢失大小信息，且容易发生越界访问。
*   **现代C++替代方案**:
    *   **`std::array`**: 固定大小的数组封装，更安全，接口更友好。
    *   **`std::vector`**: 动态数组，大小可在运行时改变，是C++中最常用的容器之一。
    ```cpp
    #include <array>
    #include <vector>

    std::array<int, 3> std_array = {1, 2, 3};
    std::cout << "std::array size: " << std_array.size() << std::endl;

    std::vector<int> std_vector = {4, 5, 6};
    std_vector.push_back(7); // 动态添加元素
    std::cout << "std::vector element at(1): " << std_vector.at(1) << std::endl; // at()会进行边界检查
    ```

#### 2.2 指针 (Pointer)

*   **介绍**: 指针是一个变量，其值为另一个变量的内存地址。
*   **原理**: 指针存储一个地址。通过解引用操作符 `*`，可以访问该地址处存储的值。
*   **多样化示例**:
    ```cpp
    // 基本用法
    int var = 20;
    int* ptr = &var;
    *ptr = 30; // 通过指针修改var的值
    std::cout << var << std::endl; // 输出 30

    // 函数指针：指向函数的指针
    void say_hello() { std::cout << "Hello!" << std::endl; }
    void (*func_ptr)() = &say_hello;
    func_ptr(); // 调用函数，输出 "Hello!"

    // 指针的指针：用于在函数内修改一个指针
    void allocate_memory(int** p) { *p = new int(100); }
    int* my_ptr = nullptr;
    allocate_memory(&my_ptr); // 传递指针的地址
    std::cout << *my_ptr << std::endl; // 输出 100
    delete my_ptr;
    ```
*   **注意事项**: 务必处理**空指针**和**悬挂指针**。使用 `new`/`new[]` 分配的内存必须用 `delete`/`delete[]` 释放，否则会导致**内存泄漏**。在现代C++中，应优先使用**智能指针** (`std::unique_ptr`, `std::shared_ptr`)来自动管理内存。

#### 2.3 引用 (Reference)

*   **介绍**: 引用是一个已存在变量的别名。它必须在声明时初始化，且一旦绑定不能再更改。
*   **实战对比 (函数参数)**:
    ```cpp
    void by_value(int val) { val++; } // 1. 值传递：不改变原始变量
    void by_pointer(int* ptr) { (*ptr)++; } // 2. 指针传递：改变原始变量
    void by_reference(int& ref) { ref++; } // 3. 引用传递：改变原始变量，语法更简洁

    int x = 10, y = 10, z = 10;
    by_value(x);
    by_pointer(&y);
    by_reference(z);
    // 结果: x=10, y=11, z=11
    ```
*   **注意事项**:
    *   引用不能为空 (`nullptr`)。
    *   函数不应返回局部变量的引用。
    *   对于大型对象，使用**常量引用 (`const T&`)**作为函数参数可以避免昂贵的复制开销，同时保证函数不会修改该对象。

---

### 3. 用户定义数据类型 (User-Defined Data Types)

#### 3.1 结构体 (struct) 与 类 (class)

*   **介绍**: 两者都允许将不同类型的数据成员和操作这些数据的成员函数封装在一起。主要区别在于 `struct` 的成员默认是 `public`，而 `class` 的成员默认是 `private`。
*   **实践示例 (Class)**:
    ```cpp
    #include <string>
    #include <iostream>

    class Student {
    private: // 封装数据，保护其不被外部直接修改
        std::string name;
        int age;

    public:
        // 构造函数：在创建对象时初始化
        Student(const std::string& n, int a) : name(n), age(a) {
            std::cout << "Student " << name << " is created." << std::endl;
        }

        // 析构函数：在对象销毁时自动调用，用于清理资源
        ~Student() {
            std::cout << "Student " << name << " is destroyed." << std::endl;
        }

        // 公有方法：提供与对象交互的安全接口
        void print_info() const { // const 成员函数，保证不修改对象状态
            std::cout << "Name: " << name << ", Age: " << age << std::endl;
        }
    };

    int main() {
        Student s1("Alice", 20);
        s1.print_info();
    } // s1 在此处离开作用域，其析构函数被自动调用
    ```

#### 3.2 联合体 (Union)

*   **介绍**: 一种特殊的数据结构，所有成员共享同一块内存空间。任何时候只能有效地存储和访问其中一个成员。
*   **场景示例 (带类型标签)**:
    ```cpp
    struct Variant {
        enum Type { INT, FLOAT };
        Type type; // 标记当前存储的是哪种类型
        union {
            int i;
            float f;
        } data;
    };

    Variant v;
    v.type = Variant::FLOAT;
    v.data.f = 99.9f;
    ```
*   **注意事项**: 手动管理联合体类型容易出错。在C++17及以后版本，应优先使用类型安全的 **`std::variant`**。

#### 3.3 枚举 (Enumeration)

*   **介绍**: 一种由一组命名的整型常量组成的类型，可增强代码的可读性。
*   **现代C++用法 (`enum class`)**:
    ```cpp
    // 作用域枚举 (enum class) 更安全，避免了命名冲突和隐式转换
    enum class TrafficLight { RED, YELLOW, GREEN };

    void take_action(TrafficLight light) {
        switch (light) {
            case TrafficLight::RED: std::cout << "Stop!" << std::endl; break;
            case TrafficLight::YELLOW: std::cout << "Caution!" << std::endl; break;
            case TrafficLight::GREEN: std::cout << "Go!" << std::endl; break;
        }
    }

    int main() {
        TrafficLight current_light = TrafficLight::GREEN;
        take_action(current_light);
    }
    ```

---

### 4. 自动类型推导 (C++11+)

*   **`auto`**:
    *   **介绍**: 让编译器根据初始化表达式自动推断变量的类型。
    *   **示例**:
        ```cpp
        auto i = 42; // i 是 int
        auto message = std::string("C++"); // message 是 std::string

        // 在处理复杂类型名（如迭代器）时尤其有用
        std::vector<int> numbers = {1, 2, 3};
        for(const auto& num : numbers) { // 使用 const auto& 遍历，高效且安全
            std::cout << num << " ";
        }
        ```
*   **`decltype`**:
    *   **介绍**: 从一个表达式中推断出类型，但并不实际计算该表达式。
    *   **示例**:
        ```cpp
        int x = 0;
        decltype(x) y = 10; // y 的类型是 int
        ```

---

### 总结与最佳实践

1.  **始终初始化变量**: 使用未初始化的变量会导致未定义行为。
2.  **选择正确的类型**: 根据数据的性质选择最合适的类型（例如，用 `bool` 表示逻辑状态，用 `size_t` 表示大小）。
3.  **警惕类型转换**: 隐式类型转换可能导致数据丢失或意外行为。使用 `static_cast` 进行安全的、明确的类型转换。
4.  **拥抱现代C++**: 优先使用 `std::vector`, `std::array` 替代C风格数组，使用智能指针管理动态内存，使用 `enum class` 替代传统 `enum`。
5.  **善用 `const`**: 尽可能地使用 `const` 来保证数据不被意外修改，提高代码的健壮性。