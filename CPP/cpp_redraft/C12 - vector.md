## 1. 辅助函数（本课打印工具）
```cpp
void printVec(const std::vector<int>& vec) {
    for (int temp : vec) std::cout << temp << " ";
}
void printSizeCapacity(const std::vector<int>& vec) {
    std::cout << "Size: " << vec.size() << " Capacity: " << vec.capacity() << std::endl;
}
```
-  **头文件**`<vector>`，find 另需 `<algorithm>`
-  **printSizeCapacity**全课反复调用，专门观察 size/capacity 变化

## 2. 构造与初始化
```cpp
std::vector<int> vec1;                   // 默认初始化，空
std::vector<int> vec2(5, 1);             // 指定大小和统一初值
std::vector<int> vec3 = { 1, 2, 3 };     // 初始化列表
std::vector<int> vec4(vec3);             // 拷贝构造
std::vector<int> vec5(std::move(vec3));  // 移动构造
```
-  **(n, val)**n 个 val；**{...}**按内容逐个给
-  **move**vec5 接管 vec3 的内存，vec3 变空（不再剩 1 2 3）
-  **元素类型任意**换成 `std::vector<std::string>` 即可（见 [[C11 - string]]）

> vector 是类模板，`<int>` 告诉它存什么类型。它自己管理内存、大小动态可变——这是和数组最大的区别。

## 3. size / capacity 与扩容观察
```cpp
std::cout << size(vec2) << " " << vec2.capacity() << std::endl;   // 5 5
vec2.push_back(100);                       // 装满了再塞一个
std::cout << size(vec2) << " " << vec2.capacity() << std::endl;   // 6 10
```
-  **两种写法**自由函数 `size(vec2)`（ADL 找到 std::size）与成员 `vec2.size()` 等价
-  **size**元素个数；**capacity**已分配内存能容纳的个数，>= size
-  **实测**vec2(5,1) push_back 后 capacity 5→10，一次扩一倍；vec4 拷贝 {1,2,3} 是 3/3，按需分配

> 扩容要重新分配内存并搬移旧元素，代价高，所以 capacity 按倍数增长留富余，摊薄每次 push_back 的成本。和 string 的 capacity 是同一思路（[[C11 - string]] 第 8 节）。

## 4. 增删：push_back / pop_back / insert / erase
```cpp
vec.push_back(10);                    // 尾部追加
vec.pop_back();                       // LIFO，删尾部
vec.insert(vec.begin() + 1, 25);      // 位置 1 插入 25
vec.erase(vec.begin() + 1);           // 删位置 1
vec.push_back("str");                 // X 元素类型必须匹配 int
```
-  **insert/erase**位置用迭代器表达：`begin() + 1` 即下标 1
-  **printVec 逐次实测**10 20 30 → 10 20 → 10 25 20 → 10 20
-  **迭代器定位**begin()/end() 详见 [[C13 - iterator]]

## 5. clear 只清元素、不还容量
```cpp
vec.clear();                  // 不完全析构，只clear元素
printSizeCapacity(vec);       // Size: 0, Capacity: 3（容量还在）
```
-  **clear 后**size=0 但 capacity 不变，内存没还给系统

## 6. swap 立即回收容量
```cpp
std::vector<int> empty_vec;
empty_vec.swap(vec);    // 立即回收容量的机制：容量换给 empty_vec
// ---- After swap ---- vec: 0/0；empty_vec: 0/3，块结束析构才真正释放
```
-  **机制**空 vector 与 vec 交换，弥补 clear 不还内存；旧容量随 empty_vec 出作用域析构
-  **通用版**main.cpp 里定义的模板，匿名空 vector 一步换走

```cpp
template<typename T>
void freeVecCapacity(std::vector<T>& vec) { std::vector<T>().swap(vec); }
```

## 7. 访问：[ ] 与 at
```cpp
std::cout << fruits[0] << std::endl;      // apple
std::cout << fruits.at(1) << std::endl;   // banana
try {    // 我敲的 at(1) 是有效索引，不抛
    std::cout << fruits.at(1) << std::endl;
} catch (std::out_of_range& e) { std::cerr << "Exception: " << e.what() << std::endl; }
```
-  **[ ]**不检查越界，越界是未定义行为；**at**带边界检查，越界抛 `std::out_of_range`
-  **课件示例**用 at(5)（只有 3 个元素）才真的抛，被 catch 接住
-  **同 string**行为与 [[C11 - string]] 第 10 节完全一致

## 8. 遍历：range for 与 iterator
### a. C++11 range for
```cpp
for (auto temp : fruits) std::cout << temp << " ";   // apple banana orange
```
### b. iterator 遍历并修改
```cpp
std::vector<int> num = { 1, 2, 3, 4, 5, 6 };
for (auto it = num.begin(); it != num.end(); ++it) std::cout << *it << " ";   // 1~6
auto iter = num.end() - 1;      // end() 指尾后，减 1 才是最后一个元素
*iter = 20060321;
std::cout << num.at(5) << std::endl;   // 20060321
```
-  **解引用赋值**迭代器可写，改的就是容器里的元素
-  **系统梳理**begin/end 与迭代器分类见 [[C13 - iterator]]

## 9. 二维 vector
```cpp
std::vector<std::vector<int>> matrix(3, std::vector<int>(4, 0));
for (size_t i = 0; i < 3; ++i)              // fill matrix
    for (size_t j = 0; j < 4; ++j) matrix[i][j] = i * 4 + j + 1;
for (auto row : matrix)
    for (auto element : row) std::cout << element << "\t";   // 1~12 三行四列
```
-  **构造**外层 3 份"内层 vector(4, 0)"作初值
-  **双层遍历**外层拿 row，内层拿 element；填矩阵下标用 size_t，与 size() 类型匹配

## 10. vector 存 struct
```cpp
struct Student {
    int id;
    std::string name;      // 用到 [[C11 - string]]
    float grade;
    Student(int id, std::string name, float grade) : id(id), name(name), grade(grade) {}
};
```
```cpp
students.push_back(lpy);                 // 对象直接入容器
for (auto const& student : students)     // auto const
    std::cout << student.id << " " << student.name << " " << student.grade << std::endl;
for (auto iter = students.begin(); iter != students.end(); ++iter)
    std::cout << iter->id << " " << iter->name << std::endl;
```
-  **auto const&**只读引用遍历，不逐个拷贝 Student
-  **iter->**迭代器解引用取成员，写法同指针
-  **细节**main.cpp 没写 `#include <string>` 也能编译（被间接包含），稳妥起见应显式写

## 11. std::find 查找
```cpp
std::string target_fruit = "cherry";
auto it = std::find(fruits.begin(), fruits.end(), target_fruit);
if (it != fruits.end()) std::cout << "Found: " << *it << std::endl;   // Found: cherry
else std::cout << "Not found: " << target_fruit << std::endl;
```
-  **头文件**`<algorithm>`
-  **返回**找到返回指向该元素的迭代器；找不到返回 end()——先判再用的套路同 string 的 npos

## 12. reserve / shrink_to_fit
```cpp
std::vector<int> num(100, 10);
num.reserve(1000);       // 只扩容量不加元素 → printSizeCapacity: 100 / 1000
num.shrink_to_fit();     // 请求收缩到 size → 100 / 100
```
-  **reserve(n)**提前留够内存，避免反复扩容搬元素
-  **shrink_to_fit**把容量收回 size 附近，释放富余（是请求，不保证精确）

> 已知要 push 很多元素时先 reserve 一次到位：扩容次数从翻倍增长的多次降到 1 次，也保住旧迭代器不失效。

## 13. 课件补充（本课代码没敲）
-  **empty()**判空返回 bool
-  **front() / back()**首元素、尾元素的引用
-  **assign()**整个重新赋值
-  **std::sort**`sort(v.begin(), v.end())` 升序；传 lambda `[](int a, int b){ return a > b; }` 降序
-  **std::reverse**反转元素顺序
-  **std::distance(begin, it)**find 之后用它求下标位置
-  **示例项目**菜单式学生/库存管理：find_if 按 id 定位 + erase 删除 + 循环菜单
