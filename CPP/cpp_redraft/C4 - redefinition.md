> 上一环：[[C3 - header and source]] —— .h 成了全项目共享的合同之后，新问题：**同一个 .h 被多次展开**，里面的定义就出现多份

## 1. 为什么会重定义
-  **重复的声明不报错，但重复的class/struct会** → 所以需要防重定义
-  同一个 cpp 里 `#include "example.h"` 两次 → 预处理原样展开两次 → `class MyClass` 定义出现两份 → `redefinition of 'MyClass'` // X
-  间接重复更常见：a.h 和 b.h 都包含了 example.h，main.cpp 同时包含 a.h 和 b.h // X
-  另一种触发（课件 04）：**.h 里定义变量**，被多个 .cpp 各包含一次 → 链接期 `multiple definition of 'global_age'` // X

> 声明只是"承诺"，可以重复；定义是"实物"，只能有一份。防重定义就是保证每份实物只被看到一次。

## 2. 防重复包含的两种写法
### a. include guard
```cpp
// --- example.h（d2-2-hcpp/inc）---
#ifndef D2_2_HCPP_EXAMPLE_H   // 第一次包含：宏未定义，往下走
#define D2_2_HCPP_EXAMPLE_H   // 立刻定义宏
class MyClass {
public:
    MyClass();                // 构造函数声明
    void myFunction();        // 成员函数声明
};
#endif                        // 第二次包含：宏已定义，整段跳过
```
-  宏名惯例：`项目_路径_文件名` 全大写，尽量唯一避免撞名
### b. #pragma once
```cpp
// --- hello.h（d1-1-environment/inc）---
#pragma once              // 作用域在同一个cpp：此文件只展开一次
void print_hello_cpp();   // declaration
```
-  **作用域在同一个cpp**：编译器保证该文件在同一编译单元内不重复包含
-  一行搞定，不用起宏名

## 3. 两种写法差异（课件补充）
-  guard 靠**宏**判断，标准 C++ 写法；once 靠编译器记录"文件已包含"，非标准但主流编译器全支持
-  我的代码里两种都在用：example.h / global.h 用 guard，hello.h 用 once；两者也可同时写，不冲突

## 4. declaration in .h file / definition in separate .cpp file
-  **现行惯例**：.h 只放 declaration，definition 放单独的 .cpp
-  变量用 `extern` 声明（课件 04 的解决方案）：声明可重复，不会撞

```cpp
// --- global.h（d2-4-externs/inc）---
#ifndef DAY2_4_EXTERNS_GLOBAL_H
#define DAY2_4_EXTERNS_GLOBAL_H
extern int global_age;           // declaration
extern std::string global_name;  // declaration
#endif
```
```cpp
// --- global.cpp ---
int global_age = 30;                   // definition，全项目唯一
std::string global_name = "John Doe";  // definition
```
-  函数/成员函数同理：example.h 放声明，函数体在 example.cpp（见 [[C3 - header and source]]）
-  class/struct 本体可以整个放 .h：guard/once 保证同一个 cpp 里只展开一份，不同 cpp 各有一份不算冲突

## 5. 总结
-  头文件只做 declaration，不做变量 definition
-  同一个 cpp 内防重复包含：guard 或 once 二选一
-  跨 .cpp 防多重定义：extern 声明 + 定义收进单独 .cpp
