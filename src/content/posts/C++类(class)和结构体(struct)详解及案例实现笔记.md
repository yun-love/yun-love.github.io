---
title: C++类(class)和结构体(struct)详解及案例实现笔记
published: 2025-10-18
description: ''
image: ''
tags: [c++, 笔记]
category: '笔记'
draft: false 
lang: ''
---
`class` 和 `struct` 是C++面向对象编程（OOP）的基石。它们允许我们将数据和操作数据的函数封装在一起，创建出自定义的、有意义的数据类型。这篇笔记将深入探讨它们的本质、异同，以及如何在现代C++中优雅地使用它们。


# C++ 类(class)与结构体(struct) 

## Part 1: 核心理念 — 从数据到“对象”

在C语言中，我们使用 `struct` 将不同的数据打包在一起，但数据本身和操作数据的函数是分离的。
```c
// C-Style
struct Rectangle_C {
    int width;
    int height;
};

// 操作数据的函数是全局的、分离的
int getArea_C(Rectangle_C r) {
    return r.width * r.height;
}
```
这存在问题：任何人都可以随意修改 `width` 和 `height`，而且 `getArea_C` 和 `Rectangle_C` 之间没有强制的关联。

C++的 `class` 和 `struct` 引入了**面向对象**的思想，将**数据（成员变量）**和**行为（成员函数）**封装在一起，形成一个紧密相关的整体——**对象 (Object)**。

> **核心思想**: 创建一个“能自我管理”的实体。一个 `Rectangle` 对象不仅**拥有**宽度和高度，还**知道**如何计算自己的面积。

```cpp
// C++ Style
class Rectangle {
public: // 后面会详细解释
    int width;
    int height;

    // 行为（成员函数）被封装在类内部
    int getArea() {
        return width * height;
    }
};
```

## Part 2: `class` vs `struct` — 唯一的区别

在C++中，`class` 和 `struct` 的功能几乎完全相同。它们都可以包含：
*   成员变量 (Member Variables)
*   成员函数 (Member Functions / Methods)
*   构造函数和析构函数
*   继承、多态、模板等所有面向对象的特性

它们之间**唯一的区别**在于 **默认的成员访问权限**：

*   **`struct`**: 成员默认是 **`public` (公开的)**。
*   **`class`**: 成员默认是 **`private` (私有的)**。

```cpp
struct MyStruct {
    int data; // 这里默认是 public
};

class MyClass {
    int data; // 这里默认是 private
};

int main() {
    MyStruct s;
    s.data = 10; // ✅ 合法，因为 s.data 是 public 的

    MyClass c;
    // c.data = 20; // ❌ 编译错误！因为 c.data 是 private 的
}
```

> **⭐ 社区约定与最佳实践**:
> *   **使用 `struct`**: 当你想要创建一个主要用于**聚合数据**的简单对象，且其成员可以被自由访问时。可以把它看作一个“增强版的C结构体”。例如：`Point { double x, y; }`, `Color { int r, g, b; }`。
> *   **使用 `class`**: 当你想要创建一个具有复杂行为、需要**封装和保护其内部状态**的对象时。类强调数据和行为的结合，以及对外部隐藏实现细节。这是构建复杂系统的标准选择。

从现在开始，我们将主要使用 `class` 来讲解面向对象的通用概念，但请记住，这些概念同样适用于 `struct`。

---

## Part 3: `class` 的解剖学 — 构建一个完整的对象

一个设计良好的类通常包含以下几个关键部分：

### 3.1 成员变量 (Member Variables)
也称为属性 (Attributes)，是类所包含的数据。它们定义了对象的状态。

### 3.2 成员函数 (Member Functions)
也称为方法 (Methods)，是类所包含的行为。它们定义了对象能做什么，通常用于操作成员变量。

### 3.3 访问修饰符 (Access Specifiers)

这是实现 **封装 (Encapsulation)** 的关键。
*   **`public`**: 公开成员。可以被类的外部任何代码访问。这是类的**接口**。
*   **`private`**: 私有成员。只能被**类自身的成员函数**访问。外部代码无法直接访问。这是类的**实现细节**。
*   **`protected`**: 受保护成员。与 `private` 类似，但允许其**子类**访问。在继承中会用到。

> **封装的核心思想**: 将数据（通常是 `private`）和操作数据的函数（通常是 `public`）捆绑在一起，对外部隐藏不必要的实现细节。这可以防止数据被意外篡改，并允许你在未来修改内部实现而不影响外部调用者。

### 3.4 构造函数 (Constructor) & 析构函数 (Destructor)

*   **构造函数**: 一个特殊的成员函数，在**创建对象时自动调用**。它的主要任务是**初始化成员变量**，确保对象一出生就处于一个有效的状态。
    *   它没有返回类型。
    *   它的名字与类名完全相同。

*   **析构函数**: 另一个特殊的成员函数，在**对象被销毁时自动调用**。它的主要任务是**释放资源**（例如，释放在构造函数中用 `new` 分配的内存）。
    *   它没有返回类型，也没有参数。
    *   它的名字是 `~` 加上类名。

```cpp
class SmartArray {
private:
    int* data;
    int size;

public:
    // 构造函数
    SmartArray(int sz) {
        size = sz;
        data = new int[size]; // 分配资源
        std::cout << "SmartArray created.\n";
    }

    // 析构函数
    ~SmartArray() {
        delete[] data; // 释放资源
        data = nullptr;
        std::cout << "SmartArray destroyed.\n";
    }
};

void test() {
    SmartArray arr(10); // 在这里，构造函数被调用
} // test函数结束，arr离开作用域，析构函数在这里被自动调用
```

### 3.5 `this` 指针
在每个非静态成员函数内部，都有一个隐藏的指针，名为 `this`。`this` 指针**指向调用该成员函数的对象本身**。

```cpp
class Window {
private:
    int width, height;
public:
    void setDimensions(int width, int height) {
        // 使用 this-> 来区分成员变量和同名的参数
        this->width = width;
        this->height = height;
    }
};
```

---

## Part 4: 综合案例实现 — `BankAccount`

这个案例将展示如何从头设计一个 `BankAccount` 类，它封装了账户的核心数据和行为，并利用构造函数和访问控制来保证其健壮性。

### 完整代码

```cpp
#include <iostream>
#include <string>
#include <stdexcept> // 用于异常处理

// --- 1. class 的定义 ---
// 我们设计一个银行账户类
class BankAccount {
private: // [封装] 数据成员是私有的，保护它们不被外部随意修改
    std::string owner_name;
    double balance;
    
public: // [接口] 成员函数是公有的，提供与对象交互的唯一途径
    
    // --- 构造函数 ---
    // 创建对象时必须提供户主名和初始余额
    BankAccount(const std::string& owner, double initial_balance) {
        owner_name = owner;

        // 保证初始余额不能为负
        if (initial_balance < 0) {
            balance = 0;
            std::cout << "Warning: Initial balance cannot be negative. Set to 0.\n";
        } else {
            balance = initial_balance;
        }
        
        std::cout << "Account for " << owner_name << " created with balance: "
                  << balance << std::endl;
    }

    // --- 成员函数 (Methods) ---
    
    // 存款
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            std::cout << "Deposited: " << amount << ". New balance: " << balance << std::endl;
        } else {
            std::cout << "Error: Deposit amount must be positive." << std::endl;
        }
    }

    // 取款
    void withdraw(double amount) {
        if (amount <= 0) {
            std::cout << "Error: Withdraw amount must be positive." << std::endl;
            return;
        }

        if (amount > balance) {
            std::cout << "Error: Insufficient funds." << std::endl;
        } else {
            balance -= amount;
            std::cout << "Withdrew: " << amount << ". New balance: " << balance << std::endl;
        }
    }

    // 查询余额 (const 成员函数)
    // 'const' 关键字保证这个函数不会修改任何成员变量
    double getBalance() const {
        return balance;
    }

    // 获取户主名
    std::string getOwnerName() const {
        return owner_name;
    }
};

// --- 2. 使用 struct 定义一个简单的数据聚合类型 ---
struct Transaction {
    std::string type;
    double amount;
};

// --- 3. main 函数: 使用我们创建的类和结构体 ---
int main() {
    // 创建一个 BankAccount 对象，构造函数被自动调用
    std::cout << "--- Creating accounts ---\n";
    BankAccount alice_account("Alice", 500.0);
    BankAccount bob_account("Bob", -100.0); // 测试构造函数中的验证逻辑
    
    std::cout << "\n--- Performing transactions ---\n";
    
    // 使用 public 接口与对象交互
    alice_account.deposit(200.0);
    alice_account.withdraw(150.0);
    alice_account.withdraw(1000.0); // 测试余额不足

    bob_account.deposit(300.0);

    // alice_account.balance = 1000000; // ❌ 编译错误！balance是private的，不能直接访问
    
    std::cout << "\n--- Final Balances ---\n";
    std::cout << alice_account.getOwnerName() << "'s final balance is: "
              << alice_account.getBalance() << std::endl;
              
    std::cout << bob_account.getOwnerName() << "'s final balance is: "
              << bob_account.getBalance() << std::endl;

    // 使用 struct
    Transaction t;
    t.type = "DEPOSIT"; // 合法，因为struct成员默认是public
    t.amount = 200.0;
    std::cout << "\nTransaction recorded: " << t.type << ", Amount: " << t.amount << std::endl;

    return 0;
}
```

### 案例分析

1.  **封装 (Encapsulation)**: `balance` 和 `owner_name` 是 `private` 的。这意味着我们不能从 `main` 函数中写出 `alice_account.balance = -999;` 这样的危险代码。所有对余额的修改都必须通过受控的 `public` 函数（`deposit`, `withdraw`）来进行，这些函数内部包含了验证逻辑（例如，不能取比余额还多的钱）。
2.  **构造函数**: `BankAccount(owner, initial_balance)` 确保了每个 `BankAccount` 对象在创建时都必须有一个户主和一个有效的初始余额。这避免了对象处于未初始化或无效状态的可能。
3.  **`class` vs `struct` 的选择**:
    *   `BankAccount` 被实现为 `class`，因为它有复杂的内部状态（`balance`）需要保护，有明确的行为（`deposit`, `withdraw`），并且需要通过构造函数来保证其初始化的正确性。这是一个典型的需要**封装**的例子。
    *   `Transaction` 被实现为 `struct`，因为它只是一个简单的数据包，用来聚合两个相关的值（类型和金额）。我们希望能够自由地读取和设置它的成员，不需要复杂的逻辑或保护。这是一个典型的**数据聚合**的例子。
4.  **`const` 成员函数**: `getBalance()` 和 `getOwnerName()` 被标记为 `const`。这向编译器和使用者承诺，调用这些函数不会改变对象的状态。这是一个非常好的编程实践，可以提高代码的安全性和可读性。