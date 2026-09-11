---
tags: cpp
---

## 1. 核心认知：多维数组是「数组的数组」

- **数组的数组** C++ 严格来说没有多维数组；`int ia[3][4]` 是「大小为 3 的数组，每个元素是含 4 个 int 的数组」
- **由内而外读** 先看名字 ia（含 3 个元素的数组），再看元素自身维度（int[4]），最后看左边：真正存的是 int
- **行 / 列** 二维习惯把第一维叫行 row，第二维叫列 column
- **维度不限** 下标可以继续嵌套下去，如 `int arr[10][20][30]`

> 「数组的数组」是全课地基：后面的指针退化、传参时「行可省略」的规则，全部由这一句推出来。

## 2. 定义与初始化

来源：`d7-14-multi-dimension_array/src/main.cpp`「initialization」段

```cpp
int ia[3][4]  = {{1,2,3,4},{5,6,7,8},{9,10,11,12}}; // 完整初始化
int ia2[3][4] = {1,2,3,4, 5,6,7,8, 9,10,11,12};     // 内层花括号可省，等价
int ia3[3][4] = {{1},{4},{8}};  // 每行只给首元素 1/4/8，其余补 0
int ia4[3][4] = {1,2,3,4};      // 省内层花括号含义变了：只初始化第一行
```

- **内层花括号** 加了按行分组，不加按顺序铺开；元素全列出时两种写法完全等价
- **部分初始化** 未列出的元素默认置 0；但省掉内层花括号后结果就大不一样（ia3 ≠ ia4）

## 3. 下标访问与行引用

来源：`main.cpp`「reference」段

```cpp
int(&ref_row_2)[4] = ia[1];    // 行引用：绑定 ia 的第二行（int[4]）
for (auto temp : ref_row_2)
    std::cout << temp << " ";  // 实测输出：5 6 7 8
std::cout << std::endl;
```

- **下标个数 = 维度** 表达式结果是元素（int）；**下标个数 < 维度** 结果是该索引处的内层数组
- `ia[2][3]` 是 int；`ia[1]` 是 int[4]，所以能声明 `int(&)[4]` 的引用绑上去

## 4. 嵌套 for 遍历

来源：`main.cpp`「C++ 11」段，range-for 按行填值

```cpp
constexpr size_t row_cnt = 3, col_cnt = 4;
int ia[row_cnt][col_cnt];
size_t cnt = 0;
for (auto &row : ia)
    for (auto &col : row) { col = cnt; ++cnt; } // 0..11 依次按行填入
```

- **外层必须用引用** `auto &row`，按值拷贝会让 row 退化，内层直接编不过：

```cpp
// 错误示范
for (auto row : ia)        // row 按值拷贝，退化为 int(*)[4] // X
    for (auto col : row)   // 对指针做 range-for：编译错误
```

## 5. 数组名、指针与三种遍历写法

来源：`main.cpp`「multi-dimension array name」段

```cpp
int array_md[3][4];
int (*ptr_row)[4] = array_md; // 数组名退化为「指向第一行」的指针
```

写法一：auto 推导 + sizeof 算边界（「type deduction style」段）

```cpp
for (auto p = ia; p < ia + sizeof(ia)/sizeof(ia[0]); ++p) // 行指针
    for (auto q = *p; q != *p + sizeof(ia[0])/sizeof(ia[0][0]); ++q)
        std::cout << *q << " "; // 输出：1 2 3 4 / 5 6 7 8 / 9 10 11 12
```

写法二：std::begin / std::end（「C++ 11 begin end」段）

```cpp
for (auto p = std::begin(ia); p != std::end(ia); ++p)
    for (auto q = *p; q != std::end(*p); ++q) // *p 是内层数组，可再 begin/end
        std::cout << *q << " ";               // 此段 ia 未初始化，输出乱值
```

写法三：类型别名（「using alias」段）

```cpp
using int_array = int[4];    // 新标准别名
typedef int int_array_t[4];  // 旧写法，等价
for (int_array *ptr_row = ia; ptr_row != std::end(ia); ++ptr_row)
    for (int *ptr_col = *ptr_row; ptr_col != *ptr_row + 4; ++ptr_col)
        std::cout << *ptr_col << " "; // 同样逐行逐元素输出
```

- **共同骨架** 外层持「行指针」，`*p` 解引用得到整行，内层再遍历 int

## 6. 传参退化：本课四条核心洞察

实际传参，来源：`matrix_calc.h` / `src/matrix_calc.cpp`

```cpp
// 计算两个矩阵的和，结果写入 sum
// m1、m2 传参后已经退化为指针，外层大小丢失，所以用 rows 显式传入行数
void calc_matrix_add(const int m1[][matrix_col], const int m2[][matrix_col],
                     int sum[][matrix_col], int rows);
```

- **退化的本质** 数组作为函数参数，无论写几维，都退化成指针。一维退化成「指向元素的指针」，二维退化成「指向第一行（内层数组）的指针」
- **「行可省略」更准确的说法** 第一维写了也被忽略，所以干脆省略。`int a[2][3]` 和 `int a[][3]` 作为参数完全等价，编译器都当成 `int (*a)[3]`。真正生效、必须写的是第二维起的内层尺寸——因为指针要知道「每行几个元素」才能正确步进到下一行
- **推广到更高维** 规则是「只有最外层的那个维度可省略」，其余全写。比如三维 `int a[][3][4]`，[3][4] 都得写
- **begin/end 判断法** 判断 begin/end 会不会吃瘪，看的是「此刻它是数组还是指针」，不是「在不在函数里」。std::end 需要知道长度，指针没有长度信息。所以：在 main 里对数组名用没问题，传进函数（已是指针）就编译报错

> 为什么列数必须写进类型？`int (*a)[3]` 的步长 = sizeof(int[3])，指针加减和 `a[i]` 定位下一行全靠它；行数只决定「走几步」，像 rows 那样当普通参数传即可。

## 7. 练习项目：matrix_calc（矩阵加法）

`d7-14-test1-matrix_calc`，读代码概括：
- **常量放头文件** `inc/matrix_calc.h` 里 `constexpr int matrix_row = 2, matrix_col = 3`
- **main.cpp** 两层 range-for（`auto &row` / `auto &val`）读入两个 2x3 矩阵，调 `calc_matrix_add` 求和，再以 `const auto &row` 输出
- **加法本体** `src/matrix_calc.cpp` 双重下标循环：`sum[i][j] = m1[i][j] + m2[i][j]`

```text
输入 1 2 3 4 5 6 与 6 5 4 3 2 1
两个矩阵的和为:
7 7 7
7 7 7
```

## 8. 课件有、代码与原笔记都没有的

- 三维示例：`int arr[10][20][30] = {0}`；用 `arr[0][0][0]` 给 `ia[2][3]` 赋值
- 行指针走到尾部：`p = &ia[2]`
- 练习 2：3x3 矩阵转置，核心一句 `transpose[j][i] = matrix[i][j]`，本次未做可补

## 9. 关联

- 一维数组退化前置：[[C14 - cstring and array]]
- 指针基础回顾：[[CPP/cpp_redraft/C6 - pointer]]
