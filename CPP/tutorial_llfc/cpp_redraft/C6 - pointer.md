## 1. 基础basic

```cpp
int var = 10;
int *ptr_01 = &var;    // ptr_01 point to var
int *ptr_02 = ptr_01;  // 指针也能拷贝赋值，两个指针指向同一个 var
```
-  **'point to' is equal to 'store'**：指向一个变量 = 肚里存着它的地址
-  declaration：**指向的类型 + `*ptr`**，例 `int *ptr_01`
-  指针**大小 size 固定**（64 位系统**默认 8 bytes**），与指向的类型无关
-  实测 `sizeof(int*) == sizeof(double*) == sizeof(void*) == 8`

> 指针存的是地址，地址宽度只由系统位数决定；指向的类型再大，指针本身也不会变胖

## 2. 空指针nullptr

```cpp
int *ptr = nullptr;
ptr = ptr_int;    // 空指针可再赋值，重新指向一个对象
// *ptr = 5; X    空指针没指向任何对象，解引用非法
```
-  **空指针**：暂时没指向任何对象，**可再赋值**
-  **无效指针**：值来路不明（野值/越界），**不可 [[../cpp/C6 - pointer#3. 解引用dereference|解引用]]**

> 空指针是"暂时没有"还能救；无效指针一旦使用后果无法预计，编译器不负责检查

## 3. 解引用dereference

```cpp
int *ptr = &var_01;
int var_02 = *ptr;  // dereference
// var_02 == var_01
```
-  **只有指针可以解引用**（只是肚里有地址的普通变量 X）
-  `*ptr`：指针**解引用的结果是一个对象**，既能读也能改（`*ptr = 20;` → var_01 变 20）

> 解引用 = 拿着肚里的地址把对象本身取出来；取到的是对象不是拷贝，所以能读也能改

## 4. 万能指针generic pointer

-  接受**任何类型**的**指针赋值**
-  [[../cpp/C6 - pointer#3. 解引用dereference|解引用]]之前必须类型转换

```cpp
double obj = 3.14;
double *ptr_obj = &obj;
void *generic_ptr = ptr_obj;          // double* 也收、int* 也收
std::cout << *((double*)generic_ptr); // 先转回 double* 再解引用 → 3.14
```

> 万能指针只记地址、不记类型，编译器不知道按什么类型取值；必须先转回原类型才能解引用

## 5. pointer to Array

```cpp
int array[5] = { 1, 2, 3, 4, 5 };
int *ptr_array = array;  // 数组名当首元素地址用
ptr_array += 1;          // offset by 1 element
// array head addr: 0x7ffd9c4aa070
// array sec  addr: 0x7ffd9c4aa074   <- 实测 addr + 4 (bytes)
```
-  **ptr += 1**：偏移**一个元素**；int 元素 4 bytes，addr + 4 (bytes)
-  指针算术**仅适用于指向数组元素的指针**

> 步长由指向的类型决定：指针 +1 不是地址 +1，而是"跳到下一个同类型元素"

## 6. 指针本身有地址

-  `&ptr_01`：指针**自己的地址**——指针也是对象，也占一块内存
-  `ptr_01`：肚里存的值 = var 的地址
-  `*ptr_01`：解引用 = var 的值

实测输出（三者区分）：

```cpp
// var address: 0x7fffecdb9ff0
// ptr_01 addr: 0x7fffecdba000 point to:0x7fffecdb9ff0 value:10
// ptr_02 addr: 0x7fffecdba008 point to:0x7fffecdb9ff0 value:10
```
-  两指针的 `point to` 与 `var address` **相同**；各自的 `addr` **互不相同**

> 指针是"存地址的对象"，自己也有门牌号；所以 `&ptr_01` 和 `ptr_01` 永远是两个地址

## 7. 指针的指针 pointer to pointer

```cpp
int var_07 = 321;
int *ptr_07 = &var_07;
int **ptr_08 = &ptr_07;  // 存指针的地址，多一级 *
```
-  `*` **的个数 = 指针的级别**：`int**` 是指向指针的指针
-  同一个 var_07 有三种取法，实测输出：

```cpp
// direct value         : 321   <- var_07 直接输出
// indirect value       : 321   <- *ptr_07 解引用一次
// doubly indirect value: 321   <- **ptr_08 解引用两次
```

> 指针自己有地址（§6），这个地址当然也能被另一个指针存起来，套几层都行

## 8. 指针的引用 reference to pointer

### a. ref 交叉验证

```cpp
int var_05 = 123;
int &ref_05 = var_05;
int *ptr_05 = &ref_05;  // 对引用取地址 = 取被绑变量的地址
int &ref_06 = *ptr_05;  // 对指针解引用，再绑一个引用
```
-  `ptr_05 == &var_05`：引用没有自己的地址，`&ref_05` 就是 `&var_05`
-  `*ptr_05 == var_05`：实测 ref_06 的地址、值都与 var_05 相同
-  引用与指针可以互相得到对方 → 回顾 [[C5 - reference]]

### b. 对指针的引用 int *&

```cpp
int var_09 = 2006;
int *ptr_09 = nullptr;
int *&ptr_10 = ptr_09;  // ptr_10 是指针 ptr_09 的别名
ptr_09 = &var_09;
// ptr_09 point to: 0x7fffecdb9ffc
// ptr_10 point to: 0x7fffecdb9ffc   <- 完全同步
```
-  `int *&`：对 `int*` 指针的引用；给 ptr_10 赋值 = 给 ptr_09 赋值
-  引用不是对象 → 不存在指向引用的指针；指针是对象 → 存在对指针的引用

> 从右往左读：`*&` 里的 `&` 说明 ptr_10 是引用，被引用的是 `int*`（一个指针）

## 9. 课件补充（没敲代码的部分）

-  **类型匹配**：指针类型必须与指向对象的类型一致，`int *pi = &dval; // X`（dval 是 double）
-  **空指针另两种写法**：`int *p2 = 0;`、`int *p3 = NULL;`（NULL 需 `#include <cstdlib>`），最推荐 nullptr
-  **int 变量不能赋给指针**：`pi = zero; // X` 即使 zero 恰好等于 0 也不行
-  **指针值的 4 种状态**：指向对象 / 指向紧邻对象所占空间的下一位置 / 空指针 / 无效指针
-  **符号的多重含义**：跟在类型名后是声明（`int *p` 指针、`int &r` 引用）；出现在表达式里是运算符（`*p` 解引用、`&var` 取地址）
-  **指针判空**：if 判断空/非空——if 还没学，到控制流课再回头看
