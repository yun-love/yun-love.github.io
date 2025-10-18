---
title: C++数据类型、循环语句、数组的综合实现案例
published: 2025-10-18
description: ''
image: ''
tags: [c++, 笔记, 案例]
category: '案例'
draft: false 
lang: ''
---
## 综合示例：学生分数管理系统

**目标**: 创建一个程序，该程序可以：
1.  存储一组学生的名字和他们的多门课程分数。
2.  使用循环来输入和处理这些数据。
3.  计算每个学生的平均分和总分。
4.  找出班级的最高分。
5.  打印格式化的成绩报告。

这个例子将集中展示以下知识点：
*   **数据类型**: `int`, `double`, `std::string`, `bool`。
*   **用户定义数据类型**: 使用 `struct` 来封装学生信息。
*   **数组**: 使用 `std::vector` (现代C++首选的动态数组) 来存储多个学生的数据。
*   **循环**: 使用范围`for`循环和嵌套`for`循环来处理数据。

---

### Step 1: 包含头文件和定义数据结构

我们首先需要包含必要的库，并定义一个结构体 `Student` 来表示每个学生的信息。这体现了 **用户定义数据类型** 的强大之处。

```cpp
#include <iostream>  // 用于输入输出 (cin, cout)
#include <vector>    // 使用 std::vector (动态数组)
#include <string>    // 使用 std::string (字符串)
#include <numeric>   // 使用 std::accumulate (用于求和)
#include <limits>    // 用于获取数值类型的最大值

// [数据类型] -> 用户定义数据类型 (struct)
// 封装了一个学生的所有相关数据
struct Student {
    std::string name;             // [数据类型] -> std::string
    std::vector<int> scores;      // [数组] -> vector作为动态数组存储分数
    double average_score = 0.0; // [数据类型] -> double
};
```

### Step 2: 主程序框架和数据输入

在 `main` 函数中，我们创建一个 `std::vector<Student>` 来存储全班学生。然后使用循环来录入学生信息。

```cpp
int main() {
    // [数组] -> 使用 vector 作为学生列表的容器，大小可以动态变化
    std::vector<Student> students;
    int num_students;
    int num_scores_per_student;

    std::cout << "Enter the number of students: ";
    std::cin >> num_students;
    std::cout << "Enter the number of scores per student: ";
    std::cin >> num_scores_per_student;

    // [循环] -> 使用 for 循环来录入每个学生的信息
    for (int i = 0; i < num_students; ++i) {
        Student new_student;
        std::cout << "\nEnter name for student #" << i + 1 << ": ";
        std::cin >> new_student.name;

        std::cout << "Enter " << num_scores_per_student << " scores for " << new_student.name << ": ";
        // [循环] -> 嵌套循环，为当前学生录入多门成绩
        for (int j = 0; j < num_scores_per_student; ++j) {
            int score;
            std::cin >> score;
            new_student.scores.push_back(score);
        }
        students.push_back(new_student);
    }
```

### Step 3: 数据处理与计算

录入数据后，我们需要遍历学生列表，计算每个人的平均分，并找出班级的最高分。

```cpp
    // --- Data Processing Section ---
    
    // [数据类型] -> bool 和 int
    int class_highest_score = 0;
    std::string top_student_name;
    bool first_student = true;

    // [循环] -> 使用范围 for 循环处理每个学生的数据
    // 这是现代C++处理容器的首选方式，更安全、更简洁
    for (auto& student : students) {
        // 计算总分
        // std::accumulate 是一个STL算法，可以替代手动循环求和
        int sum_of_scores = std::accumulate(student.scores.begin(), student.scores.end(), 0);

        // 计算平均分，注意类型转换以避免整数除法
        if (!student.scores.empty()) {
            student.average_score = static_cast<double>(sum_of_scores) / student.scores.size();
        }

        // 找出班级最高分
        // [循环] -> 嵌套的范围 for 循环，查找当前学生的最高分
        for (int score : student.scores) {
            if (score > class_highest_score) {
                class_highest_score = score;
            }
        }
    }
```

### Step 4: 输出结果报告

最后，我们再次使用循环来打印格式化的报告。

```cpp
    // --- Report Generation Section ---

    std::cout << "\n\n--- Student Grade Report ---" << std::endl;
    std::cout << "============================" << std::endl;

    // [循环] -> 使用 const auto& 进行只读遍历，效率高且安全
    for (const auto& student : students) {
        std::cout << "Student: " << student.name << std::endl;
        std::cout << "  Scores: ";
        for (int score : student.scores) {
            std::cout << score << " ";
        }
        std::cout << "\n  Average: " << student.average_score << std::endl;
        std::cout << "----------------------------" << std::endl;
    }

    std::cout << "============================" << std::endl;
    std::cout << "Class Highest Score: " << class_highest_score << std::endl;
    std::cout << "============================" << std::endl;

    return 0;
}
```

### 完整代码与运行示例

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <numeric> // For std::accumulate
#include <limits>  // For std::numeric_limits

// [数据类型] -> 用户定义数据类型 (struct)
struct Student {
    std::string name;             // [数据类型] -> std::string
    std::vector<int> scores;      // [数组] -> vector作为动态数组存储分数
    double average_score = 0.0; // [数据类型] -> double
};

int main() {
    // --- Data Entry Section ---
    std::vector<Student> students;
    int num_students;
    int num_scores_per_student;

    std::cout << "Enter the number of students: ";
    std::cin >> num_students;
    std::cout << "Enter the number of scores per student: ";
    std::cin >> num_scores_per_student;

    // [循环] -> 使用 for 循环来录入每个学生的信息
    for (int i = 0; i < num_students; ++i) {
        Student new_student;
        std::cout << "\nEnter name for student #" << i + 1 << ": ";
        std::cin >> new_student.name;

        std::cout << "Enter " << num_scores_per_student << " scores for " << new_student.name << ": ";
        // [循环] -> 嵌套循环，为当前学生录入多门成绩
        for (int j = 0; j < num_scores_per_student; ++j) {
            int score;
            std::cin >> score;
            new_student.scores.push_back(score);
        }
        students.push_back(new_student);
    }

    // --- Data Processing Section ---
    int class_highest_score = 0;

    // [循环] -> 使用范围 for 循环处理每个学生的数据
    for (auto& student : students) {
        int sum_of_scores = std::accumulate(student.scores.begin(), student.scores.end(), 0);

        if (!student.scores.empty()) {
            student.average_score = static_cast<double>(sum_of_scores) / student.scores.size();
        }
        
        // [循环] -> 嵌套的范围 for 循环，查找当前学生的最高分
        for (int score : student.scores) {
            if (score > class_highest_score) {
                class_highest_score = score;
            }
        }
    }

    // --- Report Generation Section ---
    std::cout << "\n\n--- Student Grade Report ---" << std::endl;
    std::cout << "============================" << std::endl;

    for (const auto& student : students) {
        std::cout << "Student: " << student.name << std::endl;
        std::cout << "  Scores: ";
        for (int score : student.scores) {
            std::cout << score << " ";
        }
        std::cout << "\n  Average: " << student.average_score << std::endl;
        std::cout << "----------------------------" << std::endl;
    }

    std::cout << "============================" << std::endl;
    std::cout << "Class Highest Score: " << class_highest_score << std::endl;
    std::cout << "============================" << std::endl;

    return 0;
}
```

#### 运行示例
```
Enter the number of students: 2
Enter the number of scores per student: 3

Enter name for student #1: Alice
Enter 3 scores for Alice: 95 88 92

Enter name for student #2: Bob
Enter 3 scores for Bob: 78 85 80


--- Student Grade Report ---
============================
Student: Alice
  Scores: 95 88 92 
  Average: 91.6667
----------------------------
Student: Bob
  Scores: 78 85 80 
  Average: 81
----------------------------
============================
Class Highest Score: 95
============================
```

### 知识点直观体现

*   **数据类型**: 我们清晰地看到了 `int` 用于存储整数分数，`double` 用于存储可能带小数的平均分，`std::string` 用于存储名字。最重要的是，`struct Student` 将这些不同类型的数据 **聚合** 在一起，形成了一个有意义的逻辑单元。
*   **循环语句**:
    *   外层的 `for (int i = 0; ...)` 完美地控制了 **已知次数** (学生人数) 的数据录入。
    *   内层的 `for (int j = 0; ...)` 展示了 **嵌套循环** 在处理二维数据（每个学生的多门成绩）时的应用。
    *   处理和打印部分的 **范围`for`循环** (`for (auto& student : students)`) 则体现了现代C++的简洁与安全，我们无需关心索引，直接操作每个学生对象。
*   **数组**:
    *   `std::vector<Student> students` 作为 **动态数组**，让我们可以在程序开始时才决定要录入多少学生，非常灵活。
    *   每个 `Student` 对象内部的 `std::vector<int> scores` 同样是动态数组，适应了不同课程数量的需求。
    *   `push_back()` 操作直观地展示了向动态数组末尾添加元素的过程。