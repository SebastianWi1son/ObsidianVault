## 1. main 函数：程序入口

```cpp
// --- d1-1-environment/src/main.cpp ---
#include "hello.h"

int main() {
    print_hello_cpp();   // 调用 hello.cpp 里的函数
    return 0;            // 0 = 正常结束
}
```
-  **main** 是程序**入口**：可执行文件从这里开始跑，每个工程只有一个
-  **return 0** 表示正常退出；返回非 0 一般表示出错
-  main 里只管流程，真正的打印在 hello.cpp 里 → 结构见 [[C1 - variable and output#3. 多文件工程雏形|多文件工程雏形]]

> 一个函数做一件事：main 负责调度，print_hello_cpp() 负责打印；这是后面把代码拆成多个文件的起点。

## 2. std::cout 链式输出

```cpp
// --- d1-1-environment/src/hello.cpp ---
#include <iostream>

void print_hello_cpp() {
    std::cout << "Hello,C plus plus!" << std::endl;
}
```
-  **std::cout** standard output：标准输出流对象，代表屏幕；用它要先 `#include <iostream>`
-  **<<** 插入运算符：把右边的数据塞进左边的流
-  **链式**：多个 `<<` 连着写，从左到右依次输出
-  **std::endl**：换行 + **刷新缓冲区**，保证内容立刻上屏

> cout 像传送带：每个 `<<` 往带上放一件东西（字符串、数字都行），endl 放一个换行并把整条带子发出去。

## 3. 多文件工程雏形

```
d1-1-environment/
├── CMakeLists.txt
├── inc/hello.h        // declaration
├── src/hello.cpp      // definition
└── src/main.cpp       // 入口
```
```cmake
# --- CMakeLists.txt ---
add_executable(app src/main.cpp src/hello.cpp)   # 两个 .cpp 都要编进来
target_include_directories(app PRIVATE inc)      # 去 inc/ 找 .h
set(CMAKE_CXX_STANDARD 17)                       # 标准定为 C++17
```
```cpp
// --- inc/hello.h ---
#pragma once
void print_hello_cpp();
```
-  **src** 放源文件、**inc** 放头文件，目录骨架从第一个工程就定型
-  `#include <iostream>` 系统头用 **<>**；`#include "hello.h"` 自己写的用 **""**
-  CLion 新建工程自动生成 CMakeLists.txt，构建产物在 cmake-build-debug/
-  d2-3-variable 同款结构：main.cpp + sum.cpp + inc/sum.h
-  declaration / .h / 链接的细节到头文件课展开 → [[C3 - header and source]]

## 4. 变量定义与初始化

```cpp
// --- d2-3-variable/src/main.cpp ---
int num_01 = 2;               // 定义 + 初始化
int num_02 = 3;
int sum = num_01 + num_02;   // 右边算完，再赋给左边

std::cout << "The sum of " << num_01 << " and " << num_02
          << " is " << sum << std::endl;
// The sum of 2 and 3 is 5
```
-  **变量**：有类型的存储单元；类型决定能存什么数据、占多大内存
-  定义格式：`类型 名字 = 初始值;`，如 `int num_01 = 2;`
-  cout 一条链上可混着**字符串字面量和变量**，依次拼接输出
-  命名（课件）：字母/数字/下划线，**不能数字开头**；大小写敏感
```cpp
int 1num;   // X 数字开头
```
-  常见类型还有 double / char 等（课件示例，未逐一敲）

> 变量是收纳柜：int 的柜子只装整数，装好什么类型就定了；柜子（存储单元）本身还有地址，以后讲指针就是拿地址做文章。

## 5. bool 输出与 std::boolalpha

```cpp
// --- d2-3-variable/src/main.cpp ---
bool is_right = false;
std::cout << std::boolalpha;                  // 打开开关
std::cout << "is_right: " << is_right << std::endl;
// is_right: false
```
-  默认 cout 输出 bool 是 **1 / 0**
-  **std::boolalpha** 操纵符：打开后按 **true / false** 输出
-  课件练习：bool 赋 -100 再输出 → true（非 0 即 true），未敲

## 6. 接住别的 .cpp 里函数的返回值

```cpp
// --- d2-3-variable/inc/sum.h ---
#pragma once
int use_variable();

// --- d2-3-variable/src/sum.cpp ---
int use_variable() {
    return 12;      // 返回值交给调用方
}

// --- d2-3-variable/src/main.cpp ---
int num_03 = use_variable();    // 用变量接住返回值
std::cout << "Variable value is " << num_03 << std::endl;
// Variable value is 12
```
-  函数的 **return** 值就是一个普通值，可以直接拿来初始化变量
-  实现在另一个 .cpp 里，main 照样能调 → 靠 [[C1 - variable and output#3. 多文件工程雏形|多文件工程]] + .h 里的 declaration
-  机制细节 → [[C3 - header and source]]
