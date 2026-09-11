## V1.  only single file.cpp
```cpp
#include <iostream>
// int function_10000() { ... }
int add(int a, int b) { return a + b; }  // definition
int main() { std::cout << add(3, 4); }
```
- **Problem:** 项目太长没法维护

## V2. function lib
### 1. no declaration
- mathx.cpp
```cpp
int add(int a, int b) { return a + b; }
```
- main.cpp
```cpp
#include <iostream>
int main() { std::cout << add(3, 4); }
```
**ERROR:** 'add' was not declared in this scope (main.cpp compiling)

### 2. with declaration
- main.cpp
```cpp
#include <iostream>
int add(int a, int b);    // declaration
int main() { std::cout << add(3, 4); }
```
- **Problem:** 每个add函数消费端都要声明，麻烦且容易出事
## V3. xx.h advent
### 1. no self-inc-included
- mathx.cpp
```cpp
int add(int a, int b) { return a + b; }
```

- mathx.h
```cpp
int add(int a, int b);  // declaration
```

- consume.cpp
```cpp
#include "mathx.h"
add(4, 3);
...
```
- **好处:** 一处.h关联全项目
- **Problem:** 消费端如果没和供应端(mathx.cpp)统一类型，不会报错且结果不可知
```cpp
// mathx.cpp 供应端
long add(int a, int b) { return (long)a + b; }

// mathx.h   合同
int  add(int a, int b);

// consumer  消费端
int int_num = add();
```

### 2. with self-inc-included
- mathx.cpp
```cpp
#include "mathx.h"
int add(int a, int b) { return a + b; }
```
**declaration suit the definition, no seam**
- mathx.h
```cpp
int add(int a, int b);  // declaration
```
**.h connect all the cosumption side, 统一修改**
- consume.cpp
```cpp
#include "mathx.h"
add(4, 3);
...
```
- **好处:** 供应端应该按合同交付实现，消费端也应该按合同要求调用
## V4. now
### pragma once
**重复的声明不报错，但重复的class/struct会** -> "#pragma once" to advoid redefinition
**作用域在同一个cpp**

### declaration in .h file
### definition in separate .cpp file


## 1. xx.h不是必需品，声明才是

## 2. diffrence
1. <>
```cpp
#include <iostream>
```
只从系统目录里找
2. ""
```cpp
#include "mathx.h"
```
优先从当前目录找，再进系统目录找

