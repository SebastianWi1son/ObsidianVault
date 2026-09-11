## 1. 函数：把一段逻辑打包成名字
-  **函数（function）**执行特定任务的代码块：`return_type name(params) { body }`，定义一次、多处调用
-  主项目 d8-17-function：全部写在 main.cpp 里、用 `{}` 块分段；inc/func.h 和 src/func.cpp 建了但基本空着
-  真正的声明/定义分离在 test 项目里完成，见 [[#8. 练习记录：d8-17-test-function|第 8 节]]

> 函数是"打包 + 复用"：对外只暴露签名（返回类型 + 参数列表），实现藏在函数体里，调用者不需要知道内部怎么算

## 2. 返回引用：先踩两个错误示范
```cpp
// --- main.cpp (d8-17-function) ---
int &getReference() { int local_var = 100; return local_var; }   // 错误示范，不要返回引用局部变量
int * getPointer() { int local_var = 100; return &local_var; }   // 错误示范不要返回指向局部变量的指针
```
-  两个调用我都注释掉了（`// std::cout << getReference() << std::endl;`）：结果未定义，不敢真调
-  **悬垂引用 / 悬垂指针（dangling）**：局部变量随函数返回销毁，指向它的引用/指针悬空
```cpp
// --- main.cpp (d8-17-function) --- 正确姿势：返回"活得比自己久"的参数本身
int &getMaxRef(int &num_01, int &num_02) {
    if (num_01 < num_02) { return num_02; }
    else { return num_01; }    // 调用方传入的变量，函数返回后依然有效
}
int &max = getMaxRef(num_01, num_02);    // main.cpp：num_01 = 9, num_02 = 10
std::cout << "max = " << max << std::endl;   // max = 10
```
> 规则一句话：引用/指针想跨函数返回，指向的东西必须在函数返回后仍然存活——参数、堆内存可以，局部变量不行（[[C5 - reference]]）

## 3. 返回堆数组指针：allocateArray
```cpp
// --- main.cpp (d8-17-function) ---
int * allocateArray(size_t size) {
    int* array = new int[size];
    for (size_t i = 0; i < size; i++) { array[i] = i * 2; }
    return array;               // new 出来的在堆上，返回后依然有效
}
int* arr = allocateArray(10);   // main.cpp
for (size_t i = 0; i < 10; i++) { std::cout << arr[i] << "  "; }   // 0 2 4 6 8 10 12 14 16 18
delete [] arr;                 // 谁调用谁负责释放
```
-  与第 2 节错误示范对照：**堆内存**不随函数结束回收，所以能返回；内存责任跟着指针走（[[CPP/cpp_redraft/C6 - pointer]]）

## 4. 函数重载：同名不同参
```cpp
// --- main.cpp (d8-17-function) ---
int add(int a, int b) { return a + b; }         // 重载函数
int add(int a, int b, int c) { return a + b + c; }
double add(double a, double b) { return a + b; }
int add(int &a, int &b) { return a + b; }       // 与第一个只差引用
// main.cpp
std::cout << "add(2, 3)      : " << add(2, 3) << std::endl;        // 5
std::cout << "add(2, 3, 4)   : " << add(2, 3, 4) << std::endl;     // 9
std::cout << "add(3.14, 5.20): " << add(3.14, 5.20) << std::endl;  // 8.34
```
-  **重载（overload）**：函数名相同 + 参数列表不同（个数/类型），编译器按实参挑版本；返回类型不参与区分
-  `add(2, 3)` 走的是值版本：字面量是右值，绑不上 `int &`
-  取地址也按目标类型选版本：`int (*ptr_add)(int a, int b) = &add;` 精确匹配 `add(int, int)`

## 5. 默认参数与 inline
-  **默认参数（default argument）**从右往左连续给；声明与定义分离时只写在**声明**处
-  声明示例：`void registerInfo(std::string name, int age, std::string city = "Unknown");`——只写了声明，没写定义也没调用；默认参数真正用上是在 test 项目（第 8 节）
-  **inline**：关键字只是"'建议'编译器在调用处展开"（我的原注释），这课没敲函数体

## 6. Lambda：匿名函数 + 捕获列表
```cpp
// --- main.cpp (d8-17-function) ---
void printNum(int num) { std::cout << num << std::endl; }   // 具名函数先来一发
std::for_each(numbers.begin(), numbers.end(), printNum);   // 1~5 各一行
std::for_each(numbers.begin(), numbers.end(), [](int num) -> void {  // 匿名版，效果相同
    std::cout << num << std::endl;
});
```
-  **Lambda** `[捕获](参数) -> 返回类型 { 函数体 }`：就地定义的小函数，不用先起名字
```cpp
// capture sum —— 按引用捕获
int sum = 0;
std::for_each(numbers.begin(), numbers.end(), [&sum](int num) -> void {
    sum += num;
});   // 输出 sum = 15
// capture sum_addr —— 按值捕获一个指针，结果同样是 sum = 15
sum = 0; int *sum_addr = &sum;
std::for_each(numbers.begin(), numbers.end(), [sum_addr](int num) -> void {
    *sum_addr += num;
});
```
> `[sum_addr]` 按值捕获的是**指针的副本**，但副本和原指针指向同一个 sum，解引用后照样改得动；换成 `[sum]` 按值捕获 int 副本就改不到外界了

## 7. 函数指针与回调
```cpp
// --- main.cpp (d8-17-function) ---
int array[5];
int (*parray)[5] = &array;             // 指向int[5]数组的指针
int (*ptr_add)(int a, int b) = &add;   // 指向函数的指针，写法同构：(*名字) 先结合
int add_result = (*ptr_add)(100, 100);
std::cout << "add_result = " << add_result << std::endl;   // add_result = 200
```
-  `execute(void (*ptr_func)())`：函数接受函数指针参数，`(*ptr_func)();` 解引用调用与 `ptr_func();` 不解引用调用等价；`execute(sayHello)` 打印两遍 Hello Function!
```cpp
// --- main.cpp (d8-17-function) ---
typedef void (*Callback)();     //  定义回调类型
void registerCallback(Callback cb) {
    std::cout << "Before Callback" << std::endl;
    cb();   // 执行回调
    std::cout << "After Callback" << std::endl;
}
```
-  `registerCallback(myCallback)` 输出：Before Callback / Callback Executed! / After Callback——**回调（callback）**把函数当参数传进去，让对方在合适的时机替你调用

## 8. 练习记录：d8-17-test-function
这个项目补上了声明/定义分离（[[C3 - header and source]]），main 只 `#include "func.h"`：
```cpp
// --- func.h ---
typedef void (*Callback)();
void event_routine(Callback cb);
unsigned int get_fibonacci_term(int num_term, unsigned int num_add_01 = 0, unsigned int num_add_02 = 1);
// --- func.cpp ---
unsigned int get_fibonacci_term(int num_term, unsigned int num_add_01, unsigned int num_add_02) {
    if (num_term < 0) { return -1; }
    if (num_term == 0) { return num_add_01; }        // 基准情形
    return get_fibonacci_term(num_term - 1, num_add_02, (num_add_01 + num_add_02));
}
```
-  **默认参数写在声明**、定义处不重复写——第 5 节的规则在这里落地；`get_fibonacci_term(10)` 只传一项，输出 55
-  **尾递归（tail recursion）**：递归调用是函数最后一步，`(a, b) → (b, a+b)` 滚动往前推
-  Lambda 排序：`std::sort(..., [](const std::string& a, const std::string& b) -> bool { return a.length() < b.length(); })` 按长度升序，迭代器遍历输出 kiwi grape apple banana pineapple strawberry（等长元素顺序不保证，std::sort 不稳定）
-  **事件回调**：event_routine 打印 Event #01/#02 → `cb()` → Event #03/#04；传入 get_gyro_z_dps（打印 "Gyro Z dps: xxx dps"）——事件发生时干什么由调用方决定

> 尾递归与普通递归的差别：`fib(n-1) + fib(n-2)` 返回后还要做加法（栈帧得留着等结果），尾递归的返回值直接就是递归调用的返回值，编译器可以优化成循环

## 9. 综合练习：函数操作数组的三个模式（d7-16-test-c2cpp）
数组课与函数课的交汇点；func.h 里 `#include <cstddef>` 用 size_t
### a. `size_t &size` 出参引用：一个函数带回两样东西
```cpp
// --- func.cpp ---
int* create_array(size_t &size) {
    std::cin >> size;               // 函数里读长度，改的是 main 里的 arr_size 本体
    int *arr = new int[size];
    for (size_t i = 0; i < size; i++) { std::cin >> arr[i]; }   // filling array...
    return arr;                     // 数组指针走 return，长度走引用出参
}
```
-  main.cpp：`size_t arr_size;` 未初始化，`int* arr = create_array(arr_size);` 长度由函数带回来
> return 只能带回一个值；数组指针占掉了 return，长度就用引用出参带出来——改的就是 arr_size 本体（[[C5 - reference]]）
### b. 返回堆数组指针 `int*`：失败返回 nullptr
```cpp
// --- func.cpp ---
int* create_fibonacci(size_t &size) {
    if (size < 2) { std::cout << "Fibonacci array is empty." << std::endl; return nullptr; }   // size 同样先 cin 读入（见 a）；失败路径
    int *arr = new int[size];       // 堆数组，跨函数返回仍有效
    arr[0] = 0; arr[1] = 1;
    for (size_t i = 2; i < size; i++) { arr[i] = arr[i - 1] + arr[i - 2]; }
    return arr;
}
```
-  main.cpp：`if (fibonacci != nullptr)` 先判空再用，最后 `delete []arr; delete []fibonacci;` 统一释放——与第 3 节同理，**堆内存**能跨函数返回（[[CPP/cpp_redraft/C6 - pointer]]）
### c. `const int arr[]` 只读参数：态度写在签名里
```cpp
// --- func.cpp ---
void print_array(const int arr[], size_t size) {   // 只看不改，承诺不动内容
    for (size_t i = 0; i < size; i++) { std::cout << arr[i] << "\t"; }   // \t align
    std::cout << std::endl;
}
```
-  对照 `bubble_sort(int arr[], size_t size)` 不带 const：就地 `std::swap(arr[j], arr[j+1])`，明确"会动你的数据"；提前有序则 `if (!is_swapped) break;`
> 数组形参本质是指针（只传首地址），函数里拿不到长度，所以 size 必须单独传/带出——这是模式 a 存在的根本原因（[[CPP/cpp_redraft/C6 - pointer]]）

## 10. 课件有、我没敲的
-  **changeValue 三连对照**：同一个函数分别用 `int num` / `int &num` / `int *num`，观察函数外变量变不变（值传递为什么改不了外界的对照实验）
-  **返回对象**：`Person createPerson(...)` 按值返回对象 + RVO 优化（类是下一课 C19 的内容）
-  **普通递归与 factorial**：`n * factorial(n - 1)` 常规写法、双递归 fibonacci、栈溢出风险——我只敲了尾递归版
-  **inline 函数体**：`inline int square(int x)`，我只留了注释
-  **Lambda 捕获全家桶**：`[=]` 全按值、`[x, &y]` 混合捕获；`std::function<void(int)>` 事件系统（练习 3）
