## 1. '.h'文件中const声明
- const声明**必须初始化**
```cpp
// global.h
const int global_var = 100;
```

```cpp
// main.cpp
#include "global.h"

// global_var addr: 0x5abc0c4ad008
```

```cpp
// global.cpp
#include "global.h"

// global_var addr: 0x5abc0c4ad07c
```

- 两个global_var在预处理时占用不同内存空间，且不会造成重定义
> 'cpp'是**独立编译单元**, '.h'文件会展开到每个.cpp文件中
> 普通变量会产生多个**外部链接符号**，链接时冲突
> 而命名空间作用域下const**默认**具有**内部链接**，每个cpp得到自己的独立变量
> 编译器可能会进一步把变量提前替换，但这不是避免冲突的根本原因

- 使用extern修饰可以指定使用外部.cpp内的const变量
```cpp
// global.h
extern const int var_ex_const;
```

```cpp
// global.cpp
const int var_ex_const = 100;
```
## 2. reference & const
```cpp
int var_01 = 150;
const int &ref_01 = var_01;
const int &ref_03 = var_01 * 100; // 'var_01 * 100' is a constant expr
```
- ref_01是var_01的**只可读窗口:** ref_01不可改变var_01内值,但**外界仍可改变**var_01值
- **const变量**必须由**const引用** (变量不变是自相矛盾)

## 3.  pointer & const 
### a. pointer to const
```cpp
const double pi_const = 3.14159;
const double *ptr_pi_const = &pi_const;
// ptr_pi_const是一个指针，指向const double类型变量(一个值不可改的变量)
```

```cpp
double pi = 3.1415926;
const double *ptr_pi = &pi;
// ptr_pi是一个指针，指向一个double类型变量，外部可修改该值，但不能由该指针修改
// 该指针是沿目标路径的只可读窗口
```
> 通过**这个指针访问**的目标都**不可修改**，但**指针变量本身**都可以**重新指向其他对象**
### b. const pointer
```cpp
double sqrt2 = 1.41;
double *const ptr_sqrt2 =  &sqrt2;
sqrt2 = 1.40;
// 一个const指针，指向double类型变量，不可重定向
```
> 通过该指针可以修改变量，但指针指向不可修改

## 4.  constexpr
```cpp
constexpr int const_var = 100;
constexpr int var_const = get_const; // 函数返回值编译时就能确定
// constexpr int *ptr_constexptr = &var; X 编译器会判断出不符合编译时确定
```
### constrast
- **const:** 运行时可以确定的
- **constexpr:** 编译时就可以确定的
> 是提前判断出变量是否在编译时确定的标注
> 也是强制约束编译器检查语句是否符合编译时确定的规则