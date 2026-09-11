## V1. only single file.cpp
```cpp
// main.cpp
#include <iostream>
// int function_10000() { ... }
int add(int a, int b) { return a + b; }  // definition
int main() { std::cout << add(3, 4); }
```
-  **Problem:** 项目太长没法维护

## V2. function lib（把 add 拆成函数库，还没有 .h）
### 1. no declaration
```cpp
// mathx.cpp
int add(int a, int b) { return a + b; }  // definition
```
```cpp
// main.cpp
#include <iostream>
int main() { std::cout << add(3, 4); }   // X
```
-  **ERROR:** 'add' was not declared in this scope (main.cpp compiling)

> 每个 .cpp 是独立编译单元：编译 main.cpp 时根本没见过 mathx.cpp 的内容，编译器只认当前文件里出现过的名字。

### 2. with declaration
```cpp
// main.cpp
#include <iostream>
int add(int a, int b);    // declaration
int main() { std::cout << add(3, 4); }
```
-  **Problem:** 每个add函数消费端都要声明，麻烦且容易出事

> declaration 先让编译器放心"add 存在、长这样"；definition 在别的 .cpp 里，等链接阶段再对上。

## V3. xx.h advent（declaration 搬进 .h，.h 成为合同）
### 1. no self-inc-included（供应端自己不包含 .h）
```cpp
// mathx.h
int add(int a, int b);  // declaration
```
```cpp
// mathx.cpp
int add(int a, int b) { return a + b; }
```
```cpp
// consume.cpp
#include "mathx.h"
add(4, 3);
```
-  **好处:** 一处.h关联全项目
-  **Problem:** 消费端如果没和供应端(mathx.cpp)统一类型，不会报错且结果不可知
```cpp
// mathx.cpp  供应端
long add(int a, int b) { return (long)a + b; }

// mathx.h    合同
int  add(int a, int b);

// consumer   消费端
int int_num = add();
```
> 供应端交付和合同不一致，消费端却仍按旧合同调用：编译期查不出来，坑留给运行期。

### 2. with self-inc-included（供应端自包含）
```cpp
// mathx.h
int add(int a, int b);  // declaration
```
```cpp
// mathx.cpp
#include "mathx.h"
int add(int a, int b) { return a + b; }
```
```cpp
// consume.cpp
#include "mathx.h"
add(4, 3);
```
-  **declaration suit the definition, no seam**
-  **.h connect all the consumption side, 统一修改**
-  **好处:** 供应端应该按合同交付实现，消费端也应该按合同要求调用

> 供应端 include 自己的 .h，等于先按合同自查：definition 一旦和 declaration 对不上，在 mathx.cpp 这里就报错，不用等消费端踩坑。

### 3. 真实例子：MyClass（d2-2-hcpp，类走同一套套路）
```cpp
// inc/example.h
class MyClass {
public:
    MyClass();          // 构造函数声明
    void myFunction();  // 成员函数声明
};
```
```cpp
// src/example.cpp
#include "example.h"
MyClass::MyClass() { }
void MyClass::myFunction() { std::cout << "Hello from MyClass::myFunction!"; }
```
```cpp
// src/main.cpp
#include "example.h"
MyClass class_01;
class_01.myFunction();
```
-  **declaration in .h file, definition in separate .cpp file**
-  .cpp 里用 `MyClass::` 前缀写出成员函数的 definition，同样自包含自己的 .h

## 1. xx.h不是必需品，声明才是
-  让编译过关的从来是 declaration，不是 .h 本身
-  .h 只是把 declaration 集中一处、供全项目复用的载体

## 2. difference：<> 与 ""
```cpp
#include <iostream>   // 只从系统目录里找
#include "mathx.h"    // 优先从当前目录找，再进系统目录找
```
-  标准库用 <>，自己写的头文件用 ""
-  头文件放在单独目录（如 inc/）时，编译命令加 `-I` 指定搜索目录，如 `g++ -Iinc ...`（课件补充）

## 3. .h 里放什么（课件补充）
-  类的声明、函数原型 prototypes、extern 变量声明、宏定义、inline 函数、模板声明
-  原则：只放 declaration，实现（函数体）放 .cpp

## 4. 下节预告：重复包含与重定义
-  重复的声明不报错，但重复的class/struct会（作用域在同一个cpp）→ `#pragma once` 避免 redefinition，详见 [[C4 - redefinition]]
