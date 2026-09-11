## 1. typedef 类型别名

```cpp
typedef double wages;            // wages 是 double 的同义词
typedef wages w, *p;             // w -> double, p -> double*
wages wage_01 = 10.5;
w wage_02 = wage_01;
p ptr_wage_03 = &wage_01;
```

- **typedef**：给已有类型起同义词，复杂类型名变简单、见名知意
- **连续声明**`typedef wages w, *p;`：一条语句出两个别名，`*`只从属于p
- 别名可以再作为新别名的基础（由 wages 造出 w 和 p）

## 2. using 别名声明

```cpp
using int64_t = long long;
int64_t wage_01 = 20060321;
```

- **alias declaration**（C++11）：等号左侧是别名，右侧是原类型
- 与typedef等价，是新标准引入的新写法

## 3. 别名陷阱: const pstring

```cpp
typedef char* pstring;              // pstring 是 'char*' 的别名
char char_01 = 'w';
const pstring cstr = &char_01;      // char* const, 不是 const char*
const pstring *pstr_on_declared;    // 指向 "char* const" 的指针
// cstr++; X 指针本身只读, 不可重定向
```

- `const pstring`是**常量指针**（指针本身只读），不是指向常量字符的指针
- 想当然地把别名替换成`const char*`是本节唯一的坑

> pstring已经是"指向char的指针"这个**整体类型**，const修饰的是这个整体，即top-level const
> 别名不是文本替换，要当成一个新类型名来理解；top/low-level 见 [[CPP/cpp_redraft/C7 - const]]

## 4. auto 基本用法

```cpp
int num_01, num_02;
auto num_03 = num_01 + num_02;             // int
auto num_int = 2, *ptr_int = &num_int;     // ok: 基本类型都是 int, ptr_int -> int*
// auto num_int_01 = 314, num_double = 3.14; X
```

- **auto**：编译器从初始值推算类型，变量**必须有初始值**
- 一条声明语句只有一个基本数据类型，各变量的初始基本类型必须**一致**（314是int，3.14是double，冲突）
- `*`、`&`只从属于各自的声明符，不算进基本类型

## 5. auto 识别 reference

```cpp
int num_int = 1, &ref_num_int = num_int;
auto num_auto = ref_num_int;   // int, 引用属性被抛弃
```

- 用引用初始化auto，推导结果是**被引用对象的类型**

> 使用引用其实就是使用所引用的对象，真正参与初始化的是对象的值，引用属性不参与推导

## 6. auto 穿透 const

```cpp
int num_int = 1;
const int num_const_int = num_int;
auto num_auto = num_const_int;              // int: top-level const 被穿透
auto ptr_const_int = &num_const_int;        // const int*: low-level const 保留
const auto num_const_auto = num_const_int;  // const int: 显式加回
auto &ref_num_int = num_const_int;          // const int&: 按绑定规则保留
```

```cpp
auto num_auto_02 = num_const_int, &ref_int = num_int;                     // int / int&
auto &ref_const_int = num_const_int, *ptr_const_int_02 = &num_const_int;  // const int& / const int*
```

- 只穿透**top-level const**（对象本身只读），**不穿透low-level const**（指向/绑定物只读）
- 对const对象**取地址**是low-level const，推导得`const int*`
- 想要top-level const须显式写`const auto`：编译器推导避免啰嗦，不自动带上const
- `auto&`绑定const对象，按引用初始化规则补全成`const int&`

> auto按值语义推导：拷贝出一份新值，可读可写，原对象的只读与新对象无关，top-level const随之丢弃
> 指针/引用是**访问既有对象的路径**，沿路径的low-level const必须保留，否则const对象会被改写

## 7. decltype

```cpp
const int num_const_int = 0, &ref_num_int = num_const_int;
decltype(num_const_int) x = 0;        // const int: 变量 -> 原类型, const 保留
decltype(ref_num_int) y = x;          // const int&: 引用 -> 引用
decltype(ref_num_int + 1) z;          // int: 表达式结果是具体值, 不是引用
```

```cpp
int num_int = 1, *ptr_int = &num_int;
decltype(*ptr_int) ref = num_int;     // int&: 解引用 -> 引用
decltype((num_int)) ref_01 = num_int; // int&: 双括号 -> 表达式 -> 引用
```

- **decltype**：只分析表达式并返回其类型，**不实际求值**
- 操作数是**变量**：得该变量的类型，**top-level const和引用都保留**（与auto相反）
- 操作数是**表达式**：得表达式结果对应的类型；结果能作赋值左值的，得引用
- **解引用**`*ptr_int`得`int&`而非int：解引用结果可被赋值
- **双括号**`((num_int))`：变量名加括号就被当成表达式，而变量是可作左值的特殊表达式，得`int&`
- 想去掉引用属性：把引用放进算式，如`ref_num_int + 1`

> `decltype((variable))`结果永远是引用；`decltype(variable)`只有variable本身是引用时才是引用

## 8. 课件补充（代码未敲）
- `decltype(f()) sum = x;`：不会实际调用f，只用"假如调用会返回的类型"
- `decltype(cj) z;` X：结果是引用类型，引用必须初始化
- `auto &h = 42;` X 非常量引用不能绑字面量；`const auto &j = 42;` ok
- `auto &n = i, *p2 = &ci;` X：int与const int*基本类型不一致
- auto/decltype配合模板做**尾置返回类型**（如线程池commit返回std::future），留到模板课再触发
