## 1. 引用 reference = 别名 alias

```cpp
int num_01 = 100;
int &num_01_alias = num_01;     // create alias for num_01
```
-  **引用**是已存在变量的**别名**：`int &` 读作"int 类型的引用"
-  对别名的操作**直接落在原变量**上，不需要像指针那样 dereference

> 引用不是新对象，只是同一个对象的另一个名字；变量是家，别名是同一个家的另一个称呼

## 2. 实测：同地址同值

```cpp
std::cout << "num_01 addr:       " << &num_01 << std::endl;
std::cout << "num_01_alias addr: " << &num_01_alias << std::endl;
// num_01 addr:       0x7ffd3f5c9b14
// num_01_alias addr: 0x7ffd3f5c9b14   <- 同一个地址

std::cout << "num_01 value:       " << num_01 << std::endl;
std::cout << "num_01_alias value: " << num_01_alias << std::endl;
// 都是 100
```
-  查地址方法：变量前加 `&` 即可输出地址
-  实测**地址相同、值相同** → num_01_alias 和 num_01 是**同一块内存**

> 地址可理解为门牌号：别名和本名对应的门牌号一样，进的是同一个家，取出的东西自然一样

## 3. 双向修改可见

```cpp
num_01 = 200;        // 改本名
// num_01: 200, num_01_alias: 200

num_01_alias = 300;  // 改别名
// num_01: 300, num_01_alias: 300
```
-  改 **num_01** → alias 跟着变；改 **num_01_alias** → 本名跟着变
-  两个名字不存在"谁是主"的问题，权重完全相同

## 4. 拷贝 copy ≠ 别名 alias

```cpp
int num_02 = num_01;  // 拷贝：另开一块内存，存一份相同的值
num_02 = 400;         // 只影响 num_02
// num_01 仍是 300
```
-  赋值 = **副本**：新空间、新地址，之后互不相干
-  别名 = **同一块内存**：一处改，处处可见

## 5. 引用的三个特性

```cpp
int &ref_01;           // X 引用必须初始化
int num_01 = 100;
int &ref_01 = num_01;  // 定义即绑定

int num_02 = 300;
ref_01 = num_02;       // 不是换绑！是把 num_02 的值赋给 num_01
```
-  **必须初始化**：创建时必须绑定一个已存在的对象
-  **定义即绑定，不可重新绑定**：一生只认一个对象
-  **没有空引用**：不像指针有 nullptr

> `ref_01 = num_02` 会被编译器当成对 ref_01 赋值（也就是对 num_01 赋值）；引用永远无法通过赋值"改嫁"别的变量

## 6. 引用 vs 指针

-  **指针**：自身是一个对象，存别人的地址；可重定向、可为 nullptr、取值要 dereference
-  **引用**：不是对象，只是别名；定义即绑定、无空引用、拿来直接用
-  底层实现上引用通常靠指针，但语法上不用 `*`，更简洁安全 → 对照 [[CPP/cpp_redraft/C6 - pointer]]

## 7. 延伸：const 引用与右值引用

```cpp
int var_01 = 150;
const int &ref_01 = var_01;   // ref_01's read-only windows of var_01

const int &ref_04 = 100;      // const 引用可以绑右值(字面量)
int &&ref_05 = 100;           // 右值引用 rvalue reference (C++11)
// int &ref_05 = 100; X 普通引用(左值引用)不能绑右值
```
-  **const 引用**：原变量的**只可读窗口**，自己改不了原变量、外界仍可改
-  **右值引用** `T&&`：绑定临时值（右值）；主要用途是函数参数/返回值（移动语义），此处先认识写法
-  深挖见 [[CPP/cpp_redraft/C7 - const]]

## 8. 用途：函数参数与返回值

-  引用主要用作**函数参数/返回值**：直接访问原始数据、避免拷贝，提高效率和可读性
-  目前先记住别名语义，函数场景到函数课再展开
