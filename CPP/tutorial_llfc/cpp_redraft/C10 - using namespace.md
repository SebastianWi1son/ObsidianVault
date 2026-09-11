## 1. 我的约定：一律写全 std::
-  我的代码里所有库名字都写全 `std::xxx`（d2-3-variable 从头到尾：`std::cout`、`std::endl`、`std::boolalpha`），从没用过 using 声明，更没碰过 using namespace
-  所以本课没有专门练习代码，笔记以课件为主整理
-  好处：**无命名污染**——每个名字来源明确，永远不会和标准库里的同名名字打架

> 这一课讲的是"如何省写 std:: 前缀"，我选择保留前缀：每次多写五个字符，换来名字来源完全清晰

## 2. 前缀本身：作用域操作符
-  **`::` 作用域操作符（scope operator）**告诉编译器：到左边名字所示的作用域里去找右边的名字
-  `std::cin` = 使用命名空间 std 中的名字 cin；标准库的名字都住在 std 里

## 3. using 声明（using declaration）
-  形式 `using namespace::name;`：声明之后，这个名字可以直接用，不必再写前缀

```cpp
// 课件示例（变量名换成我的习惯）
using std::cin;

int main() {
    int num_01;
    cin >> num_01;          // OK，cin 与 std::cin 含义相同
    // cout << num_01;      // X cout 没做 using 声明，必须用完整名字
    std::cout << num_01;    // 显式写 std:: 永远正确
    return 0;
}
```

## 4. 每个名字都要独立的声明
-  一条 using 声明只引入命名空间的**一个成员**，想省写几个名字就写几条

```cpp
// 课件示例
using std::cin;
using std::endl;

int main() {
    int num_01;
    cin >> num_01;
    std::cout << num_01 << endl;   // cout 没声明，仍要写全名
    return 0;
}
```

## 5. 头文件不应包含 using 声明
-  头文件内容会被**拷贝**到每个包含它的文件里 → 头文件里的 using 声明会强加给所有包含者
-  风险：**名字冲突（name conflict）**——包含者不知不觉多出了名字，可能和自己代码里的名字撞上 → [[C3 - header and source]]

> using 声明本质是"在有限范围内省前缀"；放进头文件就变成强加给所有文件，名字来源失控

## 6. using 声明 vs using namespace
-  **using 声明** `using std::cout;`：一次只放一个名字进当前作用域，影响最小（课件讲的就是这种）
-  **using 指令（using directive）** `using namespace std;`：把 std 里的名字整个倒进当前作用域
-  后果：**命名污染**——日后自己定义了和标准库同名的函数/变量，编译器直接报歧义
-  第 5 节头文件禁令和我写全前缀的习惯是同一个理由：让名字的来源始终可控
