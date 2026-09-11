## 1. string 构造
```cpp
std::string str1;                  // 默认构造，空串
std::string str2 = "HelloString!";
std::string str3(str2);            // 拷贝构造
std::string str4(str2, 0, 5);      // 从 pos=0 取 5 个字符 → "Hello"
std::string str5(7, 'c');          // 重复 7 个 'c' → "ccccccc"
```
-  **头文件**`<string>`
-  **str4(str2, pos, len)**参数是"起点 + 长度"，和 substr 的参数含义一致

## 2. 输入输出
```cpp
std::string input;
std::cin >> input;                 // 读到第一个空白字符为止
```
-  **cin >>**遇空格截断：输入 "hello world" 只读进 "hello"
-  **getline**读含空格的整行：`std::getline(std::cin, line);`（练习1 靠它逐行读文本）

## 3. 拼接
```cpp
std::string str1 = first + second;   // + 生成新串
second += first;                     // += 尾接到自身
first.append("Append");              // append 效果同 +=
```

## 4. 比较（字典序）
```cpp
std::string apple_A = "Apple";
std::string apple_a = "apple";
if (apple_A < apple_a) { ... }   // 成立：apple_A is less than apple_a
```
-  **逐字符比 ASCII**'A'(65) < 'a'(97)，大写排在小写前面，实测 "Apple" < "apple"
-  **运算符**`==  !=  <  >  <=  >=` 都能直接用于 string

> 字典序：从第 0 位开始逐个比字符的 ASCII 值，分出大小即停止；比较的就是编码数值，跟语义无关。

## 5. 查找 find
```cpp
std::string text = "The quick brown fox jumped over the lazy dog";
std::string word = "foxs";              // 故意写错，找不到
size_t pos = text.find(word);
if (pos == std::string::npos) {
    std::cout << "No match found" << std::endl;
}
```
-  **返回值**找到返回子串起始位置；找不到返回 `std::string::npos`
-  **判断套路**必须先判 `pos == std::string::npos` 再用 pos

> npos 是 size_t 最大值，充当"无效位置"的哨兵；直接 `<< pos` 会打印 18446744073709551615（2^64-1）。size_t 是无符号整数，所以 find 的返回值类型是它。

## 6. 替换 replace
```cpp
std::string word_from = "cats";
std::string word_to = "dogs";
size_t pos = text.find(word_from);
if (pos != std::string::npos) {
    text.replace(pos, word_from.length(), word_to);   // I like dogs
}
```
-  **参数**`replace(pos, len, new)`：从 pos 起删 len 个字符，换成 new
-  **搭配**先 find 拿位置再 replace；len 用 `word_from.length()`，不手数字符数

## 7. 子串 substr
```cpp
std::string str = "Hello, World!";
std::string substr_01 = str.substr(7, 5);   // "World"
std::string substr_02 = str.substr(7);      // "World!"，省略长度=取到末尾
```

## 8. size / length / capacity
```cpp
std::string str = "Hello, World!";
std::cout << str.size() << str.length() << str.capacity();
str += "crash";                             // 追加后再看三者
```
-  **size / length**完全等价，返回字符个数
-  **实测**初始 size=13、capacity=15；`+= "crash"` 后 size=18 超过 15，capacity 一次跳增到更大

> capacity 表示已分配内存能容纳的字符数（>= size）。预留富余空间是为了追加时不必每加一个字符就重新分配；装不下时才一次性扩容（常见策略是翻倍）。

## 9. cctype 与单字符
```cpp
std::string str = "ABCDE";
if (isupper(str[0])) { ... }        // str[0] 是大写
for (size_t i = 0; i < str.length(); i++) {
    std::cout << "#" << i << " : " << str[i] << std::endl;
}
```
-  **头文件**`<cctype>`，函数收单个字符：isupper、tolower 等
-  **str[i]**下标访问单个字符；循环变量用 size_t 与 length() 返回类型匹配

## 10. at 越界与 [] 的区别
```cpp
std::string str = "ABCDEFGHIJKLMNOPQRST";
try {
    std::cout << str.at(100) << std::endl;
} catch (std::out_of_range& e) {
    std::cout << "out of range" << std::endl;
}
```
-  **at(pos)**带边界检查，越界抛 `std::out_of_range`，可用 try/catch 捕获
-  **[]**不检查，越界是未定义行为（可能不报错但结果不可信）

## 11. 大小写转换
### a. transform + lambda → 大写
```cpp
std::string str = "hell world";
std::transform(str.begin(), str.end(), str.begin(),
               [](unsigned char c) -> int { return std::toupper(c); });
// HELL WORLD
```
### b. 迭代器 for → 小写
```cpp
for (auto iter = str.begin(); iter != str.end(); ++iter) {
    *iter = std::tolower(*iter);    // 解引用直接改原字符
}
```
-  **头文件**`<algorithm>`（transform）
-  **迭代器**begin/end 遍历见 [[C13 - iterator]]

## 12. stringstream
### a. 拼：<<
```cpp
std::stringstream ss_01;
ss_01 << "Value: " << 42 << ", " << 3.14;   // 数字自动变文本
std::string result = ss_01.str();           // Value: 42, 3.14
```
### b. 拆：>>
```cpp
std::string data = "123 45.67 Hello";
std::stringstream ss_02(data);
ss_02 >> num_int >> num_double >> str_01;   // 按空格分词，按目标类型转
```
### c. 复用要先 clear + str("")
```cpp
ss_01.clear();      // 清流的状态标志
ss_01.str("");      // 清流里的内容
ss_01 << numStr;
ss_01 >> num;
```
### d. 更直接的 stoi / stod
```cpp
int num = std::stoi("256");      // string → int
double pi = std::stod("3.14");   // string → double
```
-  **头文件**`<sstream>`
-  **clear 只清状态**内容要另用 str("") 清，复用 stringstream 两句都要写

## 13. 正则 regex
```cpp
std::string text = "The quick brown fox jumps over the lazy dog.";
std::regex pattern(R"(\b\w{5}\b)");       // 恰好 5 个字符的单词

std::sregex_iterator it(text.begin(), text.end(), pattern);
std::sregex_iterator end;                 // 默认构造即为结尾哨兵
while (it != end) {
    std::cout << (*it).str() << std::endl;   // quick / brown / jumps
    ++it;
}
```
-  **R"(...)"**原始字符串字面值，里面的 `\` 不用写成 `\\`
-  **sregex_iterator**遍历所有匹配；解引用后 .str() 取匹配到的文本
-  **\b**单词边界；**\w{5}**恰好 5 个字母数字字符
-  **旧笔记并入**`\w+` 字母、数字、下划线；`\.?` 可选的点 `.`（原 regex.md）

> 整串校验用 regex_match（练习2）：`std::regex_match(email, pattern)`，整个字符串完全匹配才返回 true。

## 14. 练习记录
### a. test1 文本分析器
-  **输入**getline 逐行读进 ostringstream，拼成一整段文本
-  **分词**stringstream + `while (ss >> word)` 按空格切单词
-  **清洗**remove_if + erase 去标点，transform 转小写，统一大小写便于计数
-  **统计**`map<string, int>` 记次数、顺便维护最长单词
-  **查询**cin.clear() 后再 cin >> 目标单词，用 map.find 查出现次数

### b. test2 邮箱格式验证
```cpp
const std::regex pattern(R"((\w+)(\.?\w+)*@(\w+)(\.\w+)+)");
return std::regex_match(email, pattern);
```
-  **流程**cin >> email → isValidEmail() → 输出 Valid / Invalid
-  **拆正则**`(\w+)` 用户名段；`(\.?\w+)*` 可重复的 ".xxx" 段；`@`；`(\w+)(\.\w+)+` 域名至少带一节 ".com"

## 15. 课件补充（本课代码没写）
-  **to_string**数字转 string：`std::to_string(3.14)`
-  **C 风格互转**`std::string s(cstr)` 构造；`const char* p = s.c_str()`（只读指针）
-  **empty / insert**`s.empty()` 判空；`s.insert(5, ",")` 在位置 5 插入
-  **erase(pos, len)**简单版删除：`str.erase(5, 7)` 从位置 5 删 7 个字符
-  **find_first_of / find_last_of**查找字符集合中任意字符出现的位置
