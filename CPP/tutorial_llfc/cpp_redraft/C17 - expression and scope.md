第 17 课主线：表达式 vs 语句 → 作用域种类（局部/全局/命名空间/块）→ 裸 `{}` 隔离同名变量 → 嵌套与遮蔽 → 生存期 vs 作用域。本课没有单独练习代码，例证取自 d4-8-type-deduction、d7-13-ctype_array_string 与 d7-16-test-c2cpp 综合练习的现成代码。

## 1. 表达式 Expression 与语句 Statement

- **表达式**：操作数 + 运算符的组合，**求值出一个值**（可能带副作用，如赋值、输出）
- **表达式语句**：表达式后面加 `;`，最常见的简单语句
- **声明语句**：引入名字（变量/函数/类型），目的是"让名字可用"而非求值
- **语句是执行单位**：到 `;` 或块尾 `}` 为止

```cpp
// d7-16-test-c2cpp/src/main.cpp
size_t arr_size;                     // 声明语句：引入名字，不求值
int* arr = create_array(arr_size);   // 表达式语句：函数调用 + 初始化
if (arr != nullptr) { ... }          // if 条件位放的是一个关系表达式
```

> 为什么要把两个词分开记：`j * i` 单独只是个值，配上 `;` 才成为一条可执行的语句；运算符如何组合出表达式见 [[C16 - operator]]。

## 2. 作用域 Scope 的种类（按课件）

| 种类 | 名字定义在哪 | 可见范围 | 例证 |
| --- | --- | --- | --- |
| **局部 local** | 函数或 `{}` 块内 | 声明处 → 块结束 | 练习代码里几乎所有变量 |
| **全局 global** | 所有函数之外 | 整个文件 | 课件 `globalVar`，本次代码未用 |
| **命名空间 namespace** | namespace 内 | 命名空间内 | 课件列出，本次未用 |
| **块 block** | 裸 `{}` 内 | 同局部，制造新作用域的手法 | d4-8 / d7-13 的裸块（第 3 节） |

- **补充：函数原型形参**：声明里的形参名只起自文档作用，可省略，定义处的名字才算数（d7-16-test-c2cpp/inc/func.h 里 `int* create_array(size_t &size);` 写成 `int* create_array(size_t&);` 也合法）

## 3. 块作用域实操：裸 `{}` 隔离同名变量

- d4-8 整个 main 就是一串裸块：每对 `{}` 是独立作用域，块内的类型别名/变量出块即不可见

```cpp
// d4-8-type-deduction/src/main.cpp
{
    typedef double wages;
    wages wage_01 = 10.5;         // wage_01 只活到这个 }
}
{
    using int64_t = long long;
    int64_t wage_01 = 20060321;   // 同名且不同类型：上个块的已消亡，不冲突
}
```

- d7-13 里 copy 和 cat 两个并列块各有一套 `dest` / `src`：另一个 scope，各自独立

```cpp
// d7-13-ctype_array_string/src/main.cpp —— 两个并列块
{
    char dest[20] = "Hello, ";
    const char * src = "CSting!";
    strcpy(dest + strlen(dest), src);   // 只改这个块的 dest
}
{
    char dest[20] = "Hello, ";          // 另一个 scope，各自独立
    strcat(dest, src);                  // 与上一块互不相干
}
```

- **同一块内不能重名**：d4-8 同块的第二个指针被迫叫 `ptr_const_int_02`，重定义是编译错误

> 为什么同名能并列不能同块：编译器按"离声明最近的包围块"给名字定归属；兄弟块互相看不见对方的名字，同一块里同名则无法唯一解析。

## 4. 嵌套作用域与名字遮蔽 Shadowing

- **嵌套可见**：内层作用域能看见外层名字，反方向不行；**遮蔽 shadowing**：内层同名名字暂时"挡住"外层的，出块后外层恢复
- 用户代码里目前只有"并列同名"（第 3 节），还没写过嵌套遮蔽——两者别混

```cpp
// 课件 17 §2.3 —— 嵌套块同名 = 遮蔽（先记课件例）
int x = 10;
{ int x = 20; }    // 内层 x 隐藏外层 x；出块后 x 仍是 10，外层从没被覆盖
```
```cpp
// d7-16-test-c2cpp/src/func.cpp —— print_multiplication_tab，嵌套 for
for (size_t i = 1; i < 10; i++) {         // i 的作用域 = 整条外层 for
    for (size_t j = 1; j < 10; j++) {     // j 的作用域 = 只在内层 for
        std::cout << j << "*" << i << "=" << j * i << "\t";   // 内层可读外层 i
    }
}
```

> 为什么 for 头部声明的变量出循环就不可见：`for(...)` 自带一个隐藏块包住条件、增量与循环体；循环结束名字消亡，所以两个循环都叫 `i` 不冲突。

## 5. 生存期 Lifetime vs 作用域 Scope

- **作用域**：编译期概念——名字在哪里能被查到；**生存期**：运行期概念——对象/内存何时回收
- **栈上局部变量**：两者同步，块结束名字和对象一起消亡；**heap 对象**：生存期由 new/delete 决定，与创建它的函数作用域无关

```cpp
// d7-16-test-c2cpp/src/func.cpp —— create_array 尾部
int *arr = new int[size];    // arr：函数块作用域；int[size] 本体：heap 生存期
return arr;                  // 名字 arr 随函数结束消亡，数组本体还活着
```
```cpp
// d7-16-test-c2cpp/src/main.cpp —— 在 main 的作用域里结束数组生存期
delete []arr;
delete []fibonacci;          // 不接住指针去 delete 就是泄漏
```

> 为什么函数都返回了、数组还没死：`return arr` 拷贝的只是地址值；之后局部名字 arr 消亡，但 heap 那块内存直到 main 里 `delete []` 才回收。栈/堆存储区域见 [[C2 - memory and encoding]]。

## 6. 练习记录：d7-16 综合练习（乘法表 / 冒泡排序 / 斐波那契）

- **for 头内声明 i / j**：循环变量作用域 = 该条 for（第 4 节乘法表）；**size_t 循环变量**：无符号，与数组下标、sizeof 同族
- **is_swapped 作用域 = 外层一轮**：声明在外层循环体内，每轮重新创建并置 false，"每轮重置"由作用域自动完成
- 其他点：`if (size < 2) return nullptr;`（提前 return 跳转）、`if (arr != nullptr)`（关系表达式做条件）

```cpp
// d7-16-test-c2cpp/src/func.cpp —— bubble_sort
for (size_t i = 0; i < size - 1; i++) {
    bool is_swapped = false;              // 本轮开始时重新诞生
    for (size_t j = 0; j < size - i - 1; j++) {
        if (arr[j] > arr[j + 1]) { std::swap(arr[j], arr[j+1]); is_swapped = true; }
    }
    if (!is_swapped) break;               // 跳转语句：提前终止排序
}
```
```cpp
// d7-16-test-c2cpp/src/func.cpp —— create_fibonacci 核心表达式
arr[0] = 0;
arr[1] = 1;
for (size_t i = 2; i < size; i++) {
    arr[i] = arr[i - 1] + arr[i - 2];     // 下标 + 加减 + 赋值，一条表达式语句
}
```

> 为什么 is_swapped 不声明在函数顶部：放进外层循环体，"每轮归零"由作用域自动保证；放函数顶就得手动每轮赋 false，忘一次就是 bug。

## 7. 课件有、本次没单独敲的要点

- **条件语句**：`if / else if / else` 天天在用；`switch` 每个分支记得 `break`，否则 fall-through 串到下一 case
- **迭代语句**：`while` 先判断、`do-while` 后判断（至少执行一次）；本次练习全用 for；空语句 `;` 也算一条语句
- **跳转语句**：`break`（冒泡用过）/ `continue` / `return` / `goto`（能不用就不用）
- **异常语句**：`try / catch / throw`，catch 里 `throw;` 可原样重抛
- **左值/右值**：课件本课未展开，先记"表达式求值出一个值"即可，后续再补
