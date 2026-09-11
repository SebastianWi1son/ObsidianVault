本课为理论课，无单独练习代码。

## 1. 变量计算
```cpp
int a = 10, b = 20;
std::cout << a / b << std::endl;   // 整型支持 + - * / %
float c = 10.5, d = 20.3;
std::cout << c + d << std::endl;   // 浮点支持 + - * /
```
-  **整型** int：支持 `+ - * / %` 等算术运算
-  **浮点** float / double：支持 `+ - * /`
-  **字符** char：也能参与计算 → 原因见 [[C2 - memory and encoding#2. ASCII 码表（编码原理）|ASCII]]

## 2. ASCII 码表（编码原理）
-  **ASCII**：给 128 个字符（0–127）各分配唯一数字编码，覆盖英文字母（大小写）、数字、标点、控制字符（换行、回车等）
-  例：`'A'` 对应十进制 65，`'a'` 对应 97
```cpp
char g = 'a', h = 'b';   // 内存里存的是数字 97 和 98
std::cout << (int)(g + h) << std::endl;   // 195：字符按整数参与运算
```
> 字符能做算术的根本原因：计算机中字符是用 ASCII 数字编码存储的，char 本质是一块存小整数的存储单元。

## 3. 类型划分与变量大小
-  **内置类型**（built-in）：C++ 提供的基本类型，如 double / float / int / char，可相互转换
-  **复合类型**（compound）：struct、class 等自定义类型；**引用和指针也属于复合类型**
-  **sizeof**：计算类型 / 变量占用的字节数

| 类型 | sizeof |
|---|---|
| char | 1 |
| int | 4 |
| float | 4 |
| double | 8 |
| long long | 8 |
-  佐证（d2-6-pointer 实测）：`sizeof(void*)`、`sizeof(int*)`、`sizeof(double*)` 全输出 **8**，指针大小固定、不随指向类型变 → [[C6 - pointer]]

## 4. 类型转换
-  内置类型 C++ 会**自动推导转换**，复合类型之后再介绍；**1B = 8bit**，1 字节最大能表示 127
```cpp
char a = 100;           // int(4B) → char(1B)：数字过大会损失精度
int b = a;              // char → int：不丢数据
unsigned int d = -1;    // 改符号 → 数据丢失，输出一个很大的数
```
-  **小 → 大**安全（char → int → double）；**大 → 小**或**改符号**可能丢数据

## 5. 变量作用域（Scope）
-  **作用域**：变量能被访问的代码区域，决定变量的**生命周期和可见性**

| 作用域 | 声明位置 | 可见范围 |
|---|---|---|
| 全局 | 函数外部 | 程序任何地方 |
| 局部 | 函数内部 | 仅该函数内，离开后不可见 |
| 命名空间 | namespace 内 | 外部用 `MyNamespace::` 前缀访问 |
| 类 | class 内 | 通过对象访问成员；静态成员可用类名访问 |
| 块 | `{}` 包围的代码块（局部的特例） | 仅该块内 |

> 课件提醒：全局变量哪里都能访问，但建议需要时才用——滥用会让代码难以理解和维护。

## 6. 存储区域划分
| 区域 | 存什么 | 特点 |
|---|---|---|
| 代码区 Code/Text Segment | 机器指令（函数定义、控制结构） | 共享、只读，执行期间不变 |
| 全局/静态存储区 Global/Static | 全局变量、static 变量 | 整个运行期都存在 |
| 栈区 Stack | 局部变量、函数参数、返回地址 | LIFO；函数返回时弹出、内存随之释放 |
| 堆区 Heap | `new` / `malloc` 动态分配的内存 | 手动分配释放，管理不当 → 内存泄漏、野指针 |
| 常量区 Constant Area | 字符串常量、编译期定值的 const 全局变量 | 只读，执行期间不变 |

```cpp
int globalVar = 10;             // 全局/静态存储区
static int staticVar = 20;      // 同上
void func() {
    int localVar = 30;                 // 栈区
    static int staticLocalVar = 40;    // 全局/静态区：只在第一次调用时初始化
    int* heapVar = new int(50);        // 堆区
    delete heapVar;                    // 堆内存必须手动释放
}
const char* strConst = "Hello, World!";   // 常量区：只读，别改指向的内容
```
> staticLocalVar 声明在函数里却"活得久"：它不存栈上，而在全局/静态存储区。作用域决定"哪里可见"，存储区域决定"存在哪、活多久"，两者是独立的概念。

## 7. 程序编译过程
| 阶段 | 输入 → 输出 | 做什么 |
|---|---|---|
| 预处理 Preprocessing | .cpp → .i | 宏展开、条件编译、`#include` 文件包含 |
| 编译 Compilation | .i → .s | 词法/语法/语义分析、优化，生成汇编代码 |
| 汇编 Assembly | .s → .o | 汇编器把汇编转成机器指令（目标代码） |
| 链接 Linking | .o + 库 → 可执行文件 | 解决外部符号引用，合并成可执行文件 |

-  **-save-temps=obj**：加进 CMake 编译选项 `set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -save-temps=obj")`，build 后能看到 .i / .s / .o 中间文件
> 和存储区域的联系：编译产物是磁盘上的可执行文件；运行时操作系统把其中的指令加载进内存的**代码区**，CPU 从代码区读取指令并执行。
