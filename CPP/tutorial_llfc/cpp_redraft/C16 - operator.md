第 16 课实操主线：给 Complex 类重载 `+ - <<` → 三目运算符 → sizeof 求元素个数 → 逗号运算符 → new；test 项目用 `^=` 三步交换两数。代码来自 d7-15-operator 与 d7-15-test-operator 两个工程（src/main.cpp）。

## 1. 运算符重载 Operator Overloading

- **目的**：让自定义类型（类/结构体）像内置类型一样用运算符表达操作
- **硬性规则**：至少一个操作数是用户定义类型；`::`、`?:`、`sizeof` 不能重载；重载不改变优先级、结合性、操作数个数

### a. 成员函数形式：operator+

```cpp
// d7-15-operator/src/main.cpp —— class Complex { public: double real, imag; ... }
    // 重载单目
    Complex operator + (const Complex &ex_complex) {
        Complex result;
        result.real = this->real + ex_complex.real;
        result.imag = this->imag + ex_complex.imag;
        return result;
    }
```

> 为什么注释记"单目"：成员形式里左操作数就是 `this`，参数表只剩右操作数一个——按定义时的显式参数个数记的。`+` 本身仍是双目运算符，`c1 + c2` 完整读法是 `c1.operator+(c2)`。

### b. 友元形式：operator- 与 operator<<

```cpp
// d7-15-operator/src/main.cpp
    // 重载双目
    friend Complex operator - (const Complex &ex_complex_01, const Complex &ex_complex_02);
    // 友元重载
    friend std::ostream & operator << (std::ostream & out, const Complex& c);
```

```cpp
Complex operator - (const Complex &ex_complex_01, const Complex &ex_complex_02) {
    Complex result;
    result.real = ex_complex_01.real - ex_complex_02.real;
    result.imag = ex_complex_01.imag - ex_complex_02.imag;
    return result;
}
```

- **非成员形式**：没有 this，左右操作数都写进参数表（所以记"双目"）
- **返回 out 引用**：`cout << a << b` 链式调用才接得上

> 为什么 operator<< 做成友元而 operator+ 可以是成员：成员函数的 this 永远代表左操作数；`cout << c` 的左边是 ostream，Complex 的成员函数当不了这个左操作数，只能写成非成员，再用 friend 换取访问类成员的权限。

### c. main 里实测

```cpp
// d7-15-operator/src/main.cpp
Complex c1, c2;
c1.real = 1;  c2.real = 2;    // 1、2 是 int，赋给 double 成员：隐式转换
c1.imag = 3;  c2.imag = 4;
auto result_01 = c1 + c2;                   // auto 推导为 Complex，[[C8 - type deduction]]
std::cout << result_01.real << std::endl;   // 3
std::cout << result_01.imag << std::endl;   // 7
```

```cpp
auto result_02 = c1 - c2;               // .real / .imag 单独打印均为 -1
std::cout << result_02 << std::endl;    // real: -1 imag: -1（走友元 operator<<）
```

## 2. 三目运算符（条件运算符 ? :）

- **唯一的三目运算符**：`expr ? a : b`，expr 为真取 a，否则取 b
- **优先级很低**：只高于赋值和逗号；`score > 1000` 的括号是习惯（`>` 本来就更高）

```cpp
// d7-15-operator/src/main.cpp
// expr ?
// a(true) : b(false)
int score = 1200;
const char* rank = (score > 1000) ? "predator" : "master";
std::cout << rank << std::endl;    // predator
```

> 为什么 rank 是 const char*：两个字面值分别是 const char[9] / const char[7]，都要退化（decay）成 const char* 才有公共类型——三目的两个分支会向公共类型隐式转换，见 [[C8 - type deduction]]。

## 3. sizeof：返回所占字节数

- **sizeof(数组) / sizeof(首元素)**：总字节 ÷ 单元素字节 = 元素个数

```cpp
// d7-15-operator/src/main.cpp
int array[3] = {0};
auto len = sizeof(array) / sizeof(array[0]);   // 12 / 4
std::cout << len << std::endl;    // 3，len 推导为 size_t（无符号）
```

> 为什么这里数组没退化成指针：sizeof 不求值操作数，操作数是数组本身时给整个数组的字节数（12），decay 被抑制——与 [[C14 - cstring and array]] 的退化规则正好互补。

## 4. 逗号运算符

- **求值顺序有保证**：`(a, b)` 先算 a 丢结果，再算 b，整体取 b 的值
- 逗号和 `&& || ?:` 是少数**规定顺序**的运算符；`+ - * /` 都没规定

```cpp
// d7-15-operator/src/main.cpp
int num = 1;
int result = (num++, num + 2);    // num 先自增到 2，再取 num + 2
std::cout << result << std::endl;    // 4
// int r2 = num++ + num;          // X 未定义行为：一写一读无顺序保证
```

> 为什么同一表达式里改 num 又用 num 没事：逗号是序列点，左侧 `num++` 的副作用先落地，右侧才求值。换成 `num++ + num` 就没这层保证——读写撞车即 UB。

## 5. new / delete / typeid

- **new**：在 heap 上分配并构造，返回指针；**typeid / delete 本次只留了注释**没实际敲

```cpp
// d7-15-operator/src/main.cpp
// new
{
    // heap
    struct Student { int id; std::string name; int age; };
    Student *ptr_student = new Student;   // heap 上分配
    // ... free
}
// typeid
// delete
```

- `struct Student` 定义在函数块内：局部类型，只在这个块里可见
- 应补一句 `delete ptr_student;`；释放后再 `ptr_student->id` // X 悬空指针，UB

> 为什么用 new 而不是普通局部变量：`new Student` 活在 heap，不受块作用域结束回收；代价就是必须手动还——块结束指针消失而内存还在，即泄漏。

## 6. 练习记录：test 项目——^= 三步交换

- **题目**（课件练习 1）：不用第三个变量交换两个整数
- **手法**：三次 `^=` 复合赋值，核心恒等式 `a ^ b ^ b == a`；实测 15 10 → 10 15，y 中途暂存了旧 x

```cpp
// d7-15-test-operator/src/main.cpp
int x = 15;
int y = 10;
// core operation
x ^= y;    // x = 15 ^ 10 = 5   (1111 ^ 1010 = 0101)
y ^= x;    // y = 10 ^ 5  = 15  (1010 ^ 0101 = 1111)
x ^= y;    // x = 5 ^ 15  = 10  (0101 ^ 1111 = 1010)
std::cout << x << std::endl;    // 10
std::cout << y << std::endl;    // 15
```

> 值得记住的边界：拿它交换同一个变量（`x ^= x`）会直接清零，两值全毁——面试写法，工程上用 tmp 或 std::swap 更稳。

## 7. 课件有、本次没敲的要点

- **算术运算符**：整数 `/` 是整除截断（10 / 3 == 3）；`%` 只用于整型
- **关系/逻辑运算符**：返回 bool；`&& ||` 短路求值
- **位运算全家**：`& | ^ ~ << >>`，本次只用了 `^`；`~5 == -6`（补码）
- **自增自减**：前置 `++a` 先加后用、后置 `a++` 先用后加（`int b = ++a;` vs `int c = a--;`）
- **赋值家族**：`= += -= *= /= %=` 与 `&= |= ^= <<= >>=`
- **指针/成员运算符**：`* & -> .`（第 6 课已练）
- **优先级总表**：乘除 → 加减 → 移位 → 关系 → 相等 → `&` → `^` → `|` → `&&` → `||` → `?:` → 赋值 → 逗号；同级再看结合性
- **待补**：typeid / delete 实操；课件练习 2/3（指针解引用改平方、迭代器区间求和）
