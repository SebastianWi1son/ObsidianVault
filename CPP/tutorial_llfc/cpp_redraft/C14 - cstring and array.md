第 14 课实操主线：数组 → 数组与指针 → C 风格字符串 → 手写 my_strcat。代码全部来自 d7-13 工程（src/main.cpp、src/my_string.cpp、inc/my_string.h）。

## 1. 数组的定义与初始化

- **维度必须编译期已知**：只有常量表达式（constexpr）能当维度
- **大小固定**：不能像 vector 一样运行期增长
- **不可拷贝赋值**：数组不能初始化/赋值给另一个数组

```cpp
// src/main.cpp
// 不清楚元素的确切个数还是尽量用vector
// 数组定义时，'元素类型'，'数组个数'都必须确定
constexpr unsigned int MAX_SIZE = 1024;   // 常量表达式才能做维度
int array[MAX_SIZE] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
// int array2[MAX_SIZE] = array;          // X 数组不允许直接拷贝和赋值
```

> 为什么"不清楚个数就用 vector"：维度是数组类型的一部分，编译期写死，换来的是零动态分配、更好的运行时性能；vector 灵活但要管理增长。

## 2. 指针数组 / 数组指针 / 数组引用

- **指针数组**：元素是指针的数组
- **数组指针**：指向"整个数组"的指针
- **数组引用**：绑定整个数组的引用
- 声明必须带大小 `[3]`：大小是类型信息的一部分

```cpp
// src/main.cpp
int * ptr_array[3];                  // array[3] element: ptr(int*)
int array[3] = { 1, 2, 3 };
int(*ptr2array)[3] = &array;         // ptr's a poiner to a array with size 3
int(&ref2array)[3] = array;          // ref's a reference to a array with size 3
// the ptr/ref declaration include the subject's info
```

> 为什么括号不能省：`[]` 优先级高于 `*`，从名字由内向外读——`ptr_array` 先结合 `[]` 是数组；`(*ptr2array)` 被括号抢先结合 `*`，才是指针。

## 3. 元素个数：sizeof 与补零

- `sizeof(array) / sizeof(元素)` 求元素个数
- 列表初始化没给足的元素自动补 0

```cpp
// src/main.cpp
int array[10] = { 1, 2, 3, 4, 5 };
// print all elements including '0'
for (size_t i = 0; i < sizeof(array) / sizeof(0); ++i)   // sizeof(0) == sizeof(int)
    std::cout << array[i] << " ";
std::cout << std::endl;    // 实测：1 2 3 4 5 0 0 0 0 0
```

> 为什么 `sizeof(0)` 也能用：`0` 是 int 字面值，`sizeof(0)` 即 `sizeof(int)`，恰好等于元素大小；更通用的写法是第 5 节的 `sizeof(array[0])`。

## 4. 数组名会退化成首元素指针

- 打印 `&array[0]`、`array`、`&array`：三者数值相同
- 数组名用在表达式里，自动转换成指向首元素的指针（decay）

```cpp
// src/main.cpp
int array[10] = { 1, 2, 3, 4, 5 };
std::cout << "array[0] addr      : " << &array[0] << std::endl;
std::cout << "array addr (array) : " << array << std::endl;    // 同一个地址值
std::cout << "array addr (&array): " << &array << std::endl;   // 同一个地址值
```

```cpp
// src/main.cpp
int * first_addr = array;           // 自动转成 &array[0]
int * first_addr2 = &array[0];
assert(first_addr == first_addr2);  // 二者相等
```

> 为什么 auto 和 decltype 结果不同：`auto ia(array)` 的初始值已退化成 `int*`；`decltype(array)` 不发生退化，拿到的就是"10 个 int 的数组"类型本身。

```cpp
// src/main.cpp
auto ia(array);                          // ia 是 int*，指向首元素
decltype(array) ia2 = { 2, 4, 6, 8 };    // ia2's also a array with size 10，后 6 个补 0
```

数组与指针的更多关系见 [[C6 - pointer]]。

## 5. 指针当迭代器用

- 指向元素的指针支持 `++ / * / !=`，与 vector 迭代器同款
- 老写法手工算尾后指针；C++11 用 `std::begin / std::end`
- 同数组内两个指针相减 = 元素距离

```cpp
// src/main.cpp —— iterator
int array[10] = { 1, 2, 3, 4, 5 };
int *ptr = array;    // 指向 array[0]
++ptr;               // 前移一个元素
std::cout << "*ptr value now: " << *ptr << std::endl;   // *ptr value now: 2
```

```cpp
// src/main.cpp —— 老资历玩法
int array[5] = { 1, 2, 3, 4, 5 };
int * ptr_end = array + sizeof(array) / sizeof(array[0]);   // 尾后指针
for (int * itptr = array; itptr != ptr_end; ++itptr)
    std::cout << *itptr << " ";    // 实测：1 2 3 4 5
std::cout << std::endl;
```

```cpp
// src/main.cpp —— C++ 11
auto ptr_beg = std::begin(array);
auto ptr_end = std::end(array);
for ( ; ptr_beg != ptr_end; ++ptr_beg)
    std::cout << *ptr_beg << " ";    // 实测：1 2 3 4 5
int size = ptr_end - array;          // 指针相减得元素个数
std::cout << "size: " << size << std::endl;   // size: 5
```

## 6. C 风格字符串：'\0' 结尾

- C 风格字符串不是类型，是"char 数组 + '\0' 收尾"的约定
- 字面值 `"Ha"` 实际是 `const char[3]`：2 个可见字符 + 1 个 '\0'
- `strlen` 数到 '\0' 为止，不计 '\0'

```cpp
// src/main.cpp
const char * msg = "Ha";    // const char[3]  !!!
std::cout << "string length: " << strlen(msg) << std::endl;   // string length: 2
```

> 为什么 '\0' 重要：cstring 系列函数（strlen/strcpy/strcat）全靠 '\0' 判断结尾，数组自己不记长度。没有 '\0' 就顺着内存一直读下去——越界访问与安全漏洞的根源。

```cpp
// src/main.cpp —— strlen vs sizeof
char string[100] = "Hello World!";
std::cout << "string length: " << strlen(string) << std::endl;   // 12：内容长度
std::cout << "string arr size:" << sizeof(string) << std::endl;  // 100：数组总字节
```

## 7. strcpy / strcat：拷贝与拼接

- `strcpy(dest + strlen(dest), src)`：把 dest 的结尾当写入起点，等效拼接
- `strcat(dest, src)`：语义相同的正规追加
- 两者都不检查容量：dest 必须装得下（自己的注释：可能越界访问）

```cpp
// src/main.cpp
char dest[20] = "Hello, ";
const char * src = "CSting!";
strcpy(dest + strlen(dest), src);    // 可能越界访问
std::cout << dest << std::endl;      // 实测：Hello, CSting!
```

```cpp
// src/main.cpp —— 另一个 scope，各自独立
char dest[20] = "Hello, ";
const char * src = "CSting!";
strcat(dest, src);
std::cout << dest << std::endl;      // 实测：Hello, CSting!
```

对照 [[C11 - string]]：std::string 用 `+ / +=` 拼接，空间自动管理，没有这类越界风险。

## 8. 与 std::string / vector 衔接

- 数组 → vector：给首元素地址 + 尾后地址即可初始化
- string → C 字符串：`c_str()` 返回 `const char *`

```cpp
// src/main.cpp
int array_int[10] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
std::vector<int> vec_int(std::begin(array_int), std::end(array_int));
for (auto temp : vec_int) std::cout << temp << " ";    // 实测：1 2 3 4 5 6 7 8 9 10
```

```cpp
// src/main.cpp
std::string str_std("Hello, World!");
const char * str_c = str_std.c_str();   // 指向以 '\0' 结尾的字符数组
std::cout << str_c << std::endl;        // 实测：Hello, World!
```

## 9. 手写 my_string：my_strcat（练习 2，本课亮点）

接口对齐标准库 strcat，声明放在自己的头文件里（带 header guard）：

```cpp
// inc/my_string.h
char* my_strcat(char *dst, const char *src);
```

```cpp
// src/my_string.cpp（自己的注释原样保留）
char* my_strcat(char* dst, const char* src) {   // dst可原地修改，src是副本
    char* cat_ptr = dst + strlen(dst);   // 将工作ptr设至dst尾后位置
    while (true) {
        *cat_ptr = *src;
        ++cat_ptr;
        ++src;
        if (*src == '\0') break;
    }
    return dst;
}
```

```cpp
// src/main.cpp —— Test 1: my_strcat
char dst[20] = "Hello, ";
const char * src = "CSting!";
my_strcat(dst, src);
std::cout << dst << std::endl;    // 实测：Hello, CSting!
```

实现思路：

- **定位结尾**：`dst + strlen(dst)` 正好落在 dst 原有的 '\0' 上，从这个位置开始覆盖写入
- **复制节奏**：先 `*cat_ptr = *src` 复制，再双指针前移，最后前瞻 `*src` 决定是否 break
- **指针副本**：`++src` 改的是形参副本，调用方的指针不受影响，所以最后能直接 `return dst`
- **const 保护源**：src 指向的内容只读，所有写操作都落在 dst 上

模仿了 std::string 的哪些行为（对照 [[C11 - string]]）：

- 拼接语义 ≈ `str += "CSting!"` / `str.append(...)`
- 返回目标本身 ≈ `operator+=` 返回 *this，支持链式调用
- **内部存储完全不同**：my_strcat 不拥有内存，字符存在调用方给的 char 数组里，容量与结尾全靠 '\0' 约定；std::string 自己持有缓冲区并记录 size，追加时自动扩容

> 隐藏细节（值得重敲一遍验证）：`break` 在 `++src` 之后判断——最后一轮把 `!` 写入、src 前移到 '\0' 就退出了，**'\0' 本身没被写进 dst**。这次输出正确，靠的是 `char dst[20] = "Hello, "` 聚合初始化把未列出的字节全填 0（第 3 节的补零行为），dst[14] 本来就是 0；若 dst 是一块没清零的脏缓冲，输出后面会跟乱码。修法：循环结束后补一句 `*cat_ptr = '\0';`。

## 10. 课件有、本次没敲的要点

- **strcmp 比较**：C 字符串之间用 `== / <` 比的是指针地址，要比较内容需调 strcmp（相等返回 0，否则正/负值）
- **strlen 雷区**：`char ca[] = {'C','P','P'}` 无 '\0' 结尾，strlen 会越界扫描，结果未定义
- **练习 1 my_strcpy 未做**：本次只手写了练习 2 的 my_strcat
- **解引用与指针运算**：`*(ia + 4)` 不等于 `*ia + 4`；下标本是指针解引用，`ia[1] == *(ia + 1)`
- **c_str() 时效**：string 后续被改动，之前返回的 const char* 可能失效
