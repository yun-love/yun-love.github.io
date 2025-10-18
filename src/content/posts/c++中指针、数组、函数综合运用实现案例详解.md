---
title: c++中指针、数组、函数综合运用实现案例详解
published: 2025-10-18
description: '本文通过一个完整的冒泡排序案例，详细介绍了在C++中如何结合使用指针、数组和函数，并使用CMake构建项目，涵盖了从项目结构、代码实现到编译执行的全过程。'
image: ''
tags: [c++, 案例, 详解]
category: '案例'
draft: false
lang: 'zh'
---

# C++中指针、数组与函数的协同应用：冒泡排序实例

在C++编程中，指针、数组和函数是三个核心概念，它们的结合使用构成了许多强大功能的基础。本文将通过一个具体的冒泡排序案例，详细解释如何将这三者有机地结合起来，并使用现代化的构建工具CMake来管理项目。

## 1. 项目构建与文件结构

一个良好的项目结构是软件开发的开端。我们使用CMake来管理项目，它可以自动化编译过程，处理库依赖，并支持跨平台构建。

### CMakeLists.txt 配置

首先，这是我们项目的根`CMakeLists.txt`文件，它定义了项目的基本配置。

```cmake
# 指定CMake的最低版本要求
cmake_minimum_required(VERSION 3.10)

# 定义项目名称
project(pointer_array_functions_example)

# 设置C++标准为C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 设置可执行文件的输出目录到项目根目录下的/bin文件夹
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/bin)

# 添加头文件目录，以便编译器能找到.hpp文件
include_directories(${CMAKE_SOURCE_DIR}/include)

# 自动查找src目录下的所有源文件，并存入SRC_LIST变量
aux_source_directory(${CMAKE_SOURCE_DIR}/src SRC_LIST)

# 根据源文件列表创建名为"app"的可执行文件
add_executable(app ${SRC_LIST})
```

### 项目目录结构

根据上述CMake配置，我们创建了清晰的、模块化的目录结构，将接口（头文件）、实现（源文件）和构建产物分离开。

```txt
.
├── CMakeLists.txt
├── bin/
│   └── app
├── build/
├── include/
│   └── head.hpp
└── src/
    ├── algorithms.cpp
    └── main.cpp
```
*   `include/`: 存放头文件 (`.hpp`)，用于函数和类的声明。
*   `src/`: 存放源文件 (`.cpp`)，用于功能的具体实现。
*   `bin/`: 存放最终生成的可执行文件。
*   `build/`: 用于执行CMake和编译过程的临时目录，避免污染源码区。

## 2. 代码实现详解

接下来，我们将分步实现代码，包括函数声明、函数定义以及主程序的调用。

### a. 头文件：函数声明 (`head.hpp`)

在`include/head.hpp`中，我们声明将在项目中使用的函数。这使得函数的定义和使用可以分离，提高了代码的可读性和可维护性。

```cpp
#ifndef HEAD_HPP
#define HEAD_HPP

// 声明冒泡排序函数
// 参数：一个整型指针（指向数组首地址），一个整型（数组长度）
void bubblesort(int* arr, int len);

// 声明打印数组函数
// 参数：一个整型指针（指向数组首地址），一个整型（数组长度）
void printArr(int* arr, int len);

#endif // HEAD_HPP
```

### b. 源文件：函数定义 (`algorithms.cpp`)

这个文件提供了`head.hpp`中声明的函数的具体实现。

```cpp
// src/algorithms.cpp
#include <iostream>
#include "head.hpp" // 包含我们自己的头文件

/**
 * @brief 使用冒泡排序算法对整型数组进行升序排序
 * @param arr 指向数组第一个元素的指针
 * @param len 数组的长度
 */
void bubblesort(int* arr, int len) {
    for (int i = 0; i < len - 1; i++) {
        for (int j = 0; j < len - 1 - i; j++) {
            // 如果前一个元素大于后一个元素，则交换它们
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}

/**
 * @brief 遍历并打印数组的所有元素
 * @param arr 指向数组第一个元素的指针
 * @param len 数组的长度
 */
void printArr(int* arr, int len) {
    for (int i = 0; i < len; i++) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;
}
```

#### 核心概念解析：
1.  **指针作为函数参数**：`bubblesort`和`printArr`函数都接受一个`int* arr`类型的参数。当我们将一个数组（例如`main`函数中的`myArr`）传递给这个参数时，实际上传递的是该数组第一个元素的内存地址。
2.  **数组退化 (Array Decay)**：在C++中，当数组名作为函数参数传递时，它会**“退化” (decay) 成一个指向数组第一个元素的指针**。函数内部无法通过这个指针直接获取原始数组的大小。因此，我们必须显式地传递数组的长度`len`作为另一个参数。
3.  **指针与数组操作**：在函数内部，虽然`arr`是一个指针，但我们仍然可以使用数组下标语法`arr[j]`来访问元素。这是因为C++将`arr[j]`解释为`*(arr + j)`，即从指针`arr`指向的地址开始，偏移`j`个元素位置，然后解引用得到该位置的值。

### c. 主程序：调用与执行 (`main.cpp`)

`main.cpp`是程序的入口点。在这里，我们定义一个数组，计算其长度，然后调用排序和打印函数来处理它。

```cpp
// src/main.cpp
#include <iostream>
#include "head.hpp" // 包含函数声明

int main() {
    // 1. 初始化数组
    int myArr[] = {3, 8, 1, 4, 2, 9, 6, 5, 7, 10};

    // 2. 计算数组长度
    // sizeof(myArr) 计算整个数组占用的总字节数 (例如 10 * 4 = 40 bytes)
    // sizeof(myArr[0]) 计算单个元素占用的字节数 (例如 4 bytes)
    // 两者相除得到数组的元素个数。
    int len = sizeof(myArr) / sizeof(myArr[0]);

    // 3. 打印排序前的数组
    std::cout << "Original array: ";
    printArr(myArr, len);

    // 4. 调用冒泡排序函数
    bubblesort(myArr, len);

    // 5. 打印排序后的数组
    std::cout << "Sorted array:   ";
    printArr(myArr, len);

    return 0;
}
```

#### 核心概念解析：
*   **`sizeof` 操作符**：`sizeof`是C++中的一个**编译时运算符**，用于计算变量或类型在内存中占用的字节数。
    *   **重要提示**：`sizeof(arr) / sizeof(arr[0])`这种计算数组长度的方法**仅在数组被定义的同一个作用域内有效**。一旦数组名作为参数传递给函数（发生退化），在函数内部使用`sizeof(arr)`将只能得到指针本身的大小（在64位系统上通常是8字节），而不是整个数组的大小。这正是为什么我们必须将`len`作为独立参数传递的原因。

## 3. 编译与运行

使用CMake，编译和运行项目非常简单：

1.  **进入`build`目录**：
    ```bash
    cd build
    ```

2.  **生成构建系统（例如Makefile）**：
    ```bash
    cmake ..
    ```

3.  **编译项目**：
    ```bash
    make
    ```

4.  **运行可执行文件**：
    ```bash
    ../bin/app
    ```

### 预期输出

执行程序后，你将看到以下输出：
```
Original array: 3 8 1 4 2 9 6 5 7 10 
Sorted array:   1 2 3 4 5 6 7 8 9 10 
```

## 总结

本案例全面展示了C++中指针、数组和函数如何协同工作：
-   我们通过**指针**将**数组**高效地传递给**函数**进行处理。
-   理解了**数组退化**的概念，并学会了通过传递额外的大小参数来解决这个问题。
-   掌握了`sizeof`运算符在定义域内计算数组长度的技巧。
-   通过将声明（`.hpp`）和定义（`.cpp`）分离，并使用CMake进行构建，我们实践了模块化和现代化的C++项目管理方法。

这个简单的冒泡排序例子，不仅是一个算法的实现，更是理解C++内存管理和程序结构的重要实践。