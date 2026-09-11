## 1. begin 与 end：获取迭代器
```cpp
std::vector<int> vec = { 1, 2, 3, 4 };
auto vec_begin = vec.begin();   // 指向第一个元素 1
auto vec_end = vec.end();       // 指向"尾元素的下一位置"，不指示任何元素
```
-  **获取方式**不是取地址 &，是容器的成员 begin()/end()；两者返回类型相同，用 auto 接
-  **end()**尾后迭代器（off-the-end），只是"处理完了"的标记，不能解引用、不能 ++
-  **同款接口**string 也有，见 [[C11 - string]]；[[C12 - vector]] 的 insert/erase 位置参数就是迭代器

> 迭代器是"泛化指针"：像指针一样解引用、++，但只能指向容器内的元素，不暴露容器内部表示。好处是同一套写法对 vector、string、list 都有效。

## 2. 判空：空容器时 begin == end
```cpp
std::vector<int> vec_empty;
auto iter_begin = vec_empty.begin();
auto iter_end = vec_empty.end();
if (iter_begin != iter_end) std::cout << "vector is not empty" << std::endl;
else std::cout << "vector is empty" << std::endl;   // 实测输出：vector is empty
```
-  **判空套路**begin 和 end 相等 ⟺ 容器为空（没有元素时"首元素位置"就是"尾后位置"）
-  **适用**string 判空同理：`str.begin() != str.end()`

## 3. 解引用：读 + 写元素
```cpp
std::string str = "hello world";
std::cout << "str: " << str << std::endl;       // 实测：str: hello world
if (str.begin() != str.end()) {
    auto iter = str.begin();
    *iter = toupper(*iter);                     // 解引用读出 'h'，大写后写回
    std::cout << "str: " << str << std::endl;   // 实测：str: Hello world
}
```
-  ***iter**就是 iter 所指的元素本身，非常量容器可读可写
-  **前提**解引用的迭代器必须合法且确实指示某个元素；解引用尾后/非法迭代器是未定义行为
-  **对照**下标版 `s[0] = toupper(s[0])` 的迭代器写法，同 [[C11 - string]] 改大小写

## 4. const_iterator 与 cbegin / cend
```cpp
std::vector<int>::const_iterator cit;    // 迭代器类型：只能读，不能写
std::vector<int> vec = { 1, 2, 3 };
for (cit = vec.cbegin(); cit != vec.cend(); ++cit)
    std::cout << *cit << " ";            // 实测：1 2 3
*cit = 10;                               // X const_iterator 只读，写它编译错误
```
```cpp
const std::vector<int> numbers = { 10, 20, 30 };
auto it_01 = numbers.begin();    // 对象是 const，begin() 返回的就是 const_iterator
auto it_02 = numbers.cbegin();   // C++11：非常量对象也能主动要 const 迭代器
```
-  **iterator / const_iterator**容器自己定义的两种迭代器类型，类比"指向常量的指针"（[[CPP/cpp_redraft/C7 - const]]）
-  **返回类型由对象决定**const 对象的 begin()/end() 返回 const_iterator；cbegin()/cend() 永远返回 const_iterator
-  **精确类型**同 size_type 一样无需关心，auto 接住即可

## 5. 解引用 + 成员访问
```cpp
std::vector<std::string> strGroup = { "helloworld", "crash" };
if (strGroup.begin() != strGroup.end()) {       // is vector empty
    auto iter = strGroup.begin();
    if ((*iter).empty())                        // is the first element empty ""
        std::cout << "strGroup is not empty" << std::endl;
    else
        std::cout << "strGroup is empty" << std::endl;   // 实测走这里
}
```
-  ***iter 是个 string 对象**，再接 `.empty()` 访问其成员；圆括号必不可少
-  **实测**首元素 "helloworld" 非空 → 走 else 输出 "strGroup is empty"（打印文案说的是 strGroup，实际判断的是第一个元素）
-  **it->mem**等价 `(*it).mem`，本课没敲；[[C12 - vector]] 第 10 节已用过 `iter->id`

## 6. ++ 前移与遍历循环
```cpp
std::vector<std::string> strs_vec = { "hello", "", "world" };
for (auto iter = strs_vec.begin(); iter != strs_vec.end() && !(*iter).empty(); ++iter)
    std::cout << *iter << " ";      // 实测：hello （遇到空串即停）
std::cout << std::endl;
// 不要在遍历时修改容器   <-- 我的原注释，原因见第 7 节
```
-  **++iter**迭代器前移一个位置；循环条件依次是"没到尾后"和"当前元素非空"
-  **求值顺序**先判 `iter != end()` 再解引用，保证不会解引用尾后迭代器

> 循环条件为什么用 != 不用 <：所有标准库容器的迭代器都定义了 ==/!=，但大多数没有 <。养成用 != 的习惯，换成任何容器写法都不用改（泛型编程）。

## 7. erase 与迭代器失效：删奇数（课件面试题）
```cpp
std::vector<int> vec = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
for (auto iter = vec.begin(); iter != vec.end(); ) {   // for 头不写 ++，循环体自己控制
    bool is_odd = ((*iter) % 2 != 0);
    if (is_odd) { iter = vec.erase(iter); continue; }  // erase 返回"下一个元素"的迭代器
    ++iter;
}
for (auto temp : vec) std::cout << temp << " ";        // 实测：2 4 6 8
```
-  **erase(iter)**删除所指元素，被删位置之后的迭代器全部失效；返回下一个有效元素的迭代器，必须接住
-  **for 头不写 ++**保留元素时手动 ++iter，删除时靠返回值前移；否则会跳元素或使用已失效的迭代器
-  **同因**push_back 可能触发扩容，旧迭代器全部失效——所以遍历中不能 push（扩容搬移见 [[C12 - vector]] 第 3 节）

> vector 元素在内存里连续存放，扩容要搬家、erase 要前移补位，旧迭代器还指着原来的位置，就成了悬空指针。

## 8. 迭代器算术与二分查找
```cpp
std::vector<int> vec = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11 };
int target = 2;
auto vec_begin = vec.begin();
auto vec_end = vec.end();
auto vec_mid = vec_begin + (vec_end - vec_begin) / 2;  // 迭代器 + 整数 = 前移若干位
```
```cpp
while (vec_mid != vec_end && *vec_mid != target) {
    if (*vec_mid < target) vec_begin = vec_mid + 1;   // 目标在右半，mid 已排除
    else vec_end = vec_mid;                           // 目标在左半，不含 mid
    vec_mid = vec_begin + (vec_end - vec_begin) / 2;  // 新范围重新取中点
}
if (vec_mid != vec_end) std::cout << "Found target: " << *vec_mid << std::endl;  // 实测：Found target: 2
else std::cout << "No Found: " << target << std::endl;
```
-  **iterator arithmetic**`it + n` 一次跨 n 个元素、`it2 - it1` 得两迭代器距离；string/vector 迭代器独有，list 等容器没有
-  **(end - begin) / 2**用距离求中点，比 `begin + size()/2` 更通用（任何 [beg, end) 范围都能用）
-  **终止条件**`*vec_mid == target` 找到；`vec_mid == vec_end` 说明找遍无果
-  **前提**序列有序；范围每次减半，O(log n)

## 9. 课件补充（本课代码没敲）
-  **箭头运算符**`it->empty()` 等价于 `(*it).empty()`
-  **push_back 失效反例**循环里 push_back 导致迭代器失效 + 死循环（下标版、迭代器版两个例子）
-  **直接取中点**`auto mid = numbers.begin() + numbers.size()/2;`，再判 `mid != end()`
-  **首个单词大写**`it != s.end() && !isspace(*it)` 版 toupper 循环（与第 6 节同构）
-  **迭代器三义**该词可指：概念本身 / 容器定义的迭代器类型 / 某个迭代器对象
-  **练习题**相邻元素和（`it + 1 != end`）、反向打印（reverse_iterator 的 rbegin/rend）、insert + 迭代器范围合并两个 vector
