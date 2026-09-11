## 1. class 定义与访问控制
-  **class**把数据（成员变量）和操作数据的函数（成员函数）打包成自定义类型；**对象**是类的一个实例
-  **public:** 对外接口（构造/析构/modify_/get_）；**private:** 内部数据，类外不可直接访问

```cpp
// --- student.h ---
class Student {
public:
    long get_id() const;
private:
    long id_;
    std::string name_;
    int age_;
};
```

```cpp
// main.cpp
// std::cout << lpy.id_ << std::endl;   // X id_ 是 private，类外不可访问
std::cout << lpy.get_id() << std::endl; // 只能走 public 接口
```

> 封装为什么把成员藏进 private：外界只能经 public 的 modify_/get_ 读写，类就管住了自己的状态——以后改成员名、给 setter 加校验，调用方一行不用动

## 2. .h 声明与 .cpp 定义分离
- 类定义放 .h（成员 + 函数声明），定义放 student.cpp，同 [[C3 - header and source]] 的分离思路
- **Student::** 作用域符：.cpp 里定义成员函数时标明属于哪个类

```cpp
// --- student.cpp ---
long Student::get_id() const { return id_; }   // Universal API 系列
// --- C9 structure.md 原写法，class 同样适用 ---
void Student::printInfo() const {   // .cpp 里 Student::printInfo() const 定义
    std::cout << "ID: " << id << ...
}
```

- **const 成员函数**：getter/printInfo 承诺只读，不修改对象状态；header guard 名沿用旧项目的 `DAY8_14_..._H`，guard 名只需全工程唯一

## 3. 构造函数：一个类四个重载
### a. 默认构造与参数化构造
```cpp
// --- student.cpp ---
Student::Student() : id_(0), name_("Average Joe"), age_(0) {
    data_ = new int();   // 值初始化为 0，堆内存
}  // 默认初始化
Student::Student(long id, const std::string & name, int age)
    : id_(id), name_(name), age_(age) {
    data_ = new int();
}     // 初始化列表
```

- **参数初始化列表**：成员创建时直接初始化，而非先默认构造再赋值 → [[C9 - structure#3. 构造函数|C9 已学]]
- 无参/有参按参数个数构成**重载**，按调用形式选择

### b. 拷贝构造与深浅拷贝
```cpp
// --- student.cpp ---
Student::Student(const Student &student)
    : id_(student.id_), name_(student.name_), age_(student.age_) {
    // data_ = student.data_;       // 浅拷贝 (double free风险)
    *data_ = *(student.data_);      // 深拷贝
}
```

> 浅拷贝只复制指针的值，两个对象的 data_ 指向同一块堆内存，析构时 delete 两次 → double free；深拷贝是各自 new 一块，再复制所指的值

### c. 移动构造
```cpp
// --- student.cpp ---
Student::Student(Student &&student)
    : id_(std::move(student.id_)),
      name_(std::move(student.name_)),
      age_(student.age_),
      thread_(std::move(student.thread_)) {
    data_ = new int();
}
```

- **std::move**把源对象的资源搬走而非复制；标量（long/int）的 move 就是拷贝
- **std::thread 不可拷贝**：含 thread_ 成员的类，拷贝/析构都要显式写（课件 5.5 场景，析构里 join 正为它）

## 4. 析构函数
```cpp
// --- student.cpp ---
Student::~Student() {
    std::cout << "Student::~Student()" << std::endl;
    if (thread_.joinable()) {
        thread_.join();   // 等线程结束
    }
    delete data_;
    std::cout << "Student::~Student() Success" << std::endl;
}
```

- 对象生命周期结束自动调用：释放 data_、收尾 thread_；.h 注释"**析构函数(一定公有)**"——离开作用域时编译器要自动调 dtor，private 则无权调用

## 5. static 成员：全类共享一份数据
```cpp
// --- student.h ---
static int money_;     // class 公共属性
// --- student.cpp ---
int Student::money_ = 100;
void Student::cost_money(int cost) { money_ -= cost; }
```

- money_ 不属于任何对象：lpy 花 15、lwb 再花 10，两边 get_money() 都是 75；**类外定义**——static 成员必须在 .cpp 定义一次（同 [[C3 - header and source]] 的唯一定义问题）

> static 成员是"class 公共属性"，所有对象共享同一份数据；余额这类统计量适合放 static，而非每个对象复制一份

## 6. 友元与 << 运算符重载
```cpp
// --- student.h ---
friend std::ostream & operator << (std::ostream & out, const Student& student);   // 友元重载
// --- student.cpp ---
std::ostream & operator << (std::ostream & out, const Student& student) {
    out << "id:" << student.id_ << " name:" << student.name_ << " age:" << student.age_;
    return out;
}
```

- **friend**：让非成员函数获得访问 private 的权限；operator<< 左操作数是 ostream，只能做成非成员函数，定义处不带 Student::；C9 的 `std::cout << lpy;   // X` 到这里解掉了 → [[C9 - structure#5. 成员访问与输出限制|C9 输出限制]]

## 7. 对象创建与实测输出
```cpp
// --- main.cpp ---
Student lpy(202421020315, "crash", 21);   // 参数化构造
Student lwb;                              // 默认构造（无括号；带 () 会被解析成函数声明）
lwb.modify_id(202421020316);              // 点运算符访问成员函数
Student lpy_fake(lpy);                    // 拷贝构造
Student lpy_new(std::move(lpy_fake));     // 移动构造
```

实测输出（g++ -std=c++17 编译运行）：assa
```text
id:202421020315 name:crash age:21
85
id:202421020316 name:Average Joe age:0
75
75
id:202421020315 name:crash age:21
id:202421020315 name:crash age:21
id:202421020315 name: age:21
```
- 85 → 75：money_ 全类共享（100 - 15 - 10）
- 末行 `name:` 为空：lpy_fake 的 name_ 被 move 走，id/age 是标量照旧——**moved-from 对象**有效但值不确定
- 析构打印共 4 组，顺序与构造**相反**（lpy_new → lpy_fake → lwb → lpy）

## 8. struct vs class
- C9 的 `struct Student { int id; std::string name; };` 已写过构造函数、初始化列表、成员函数分离 → [[C9 - structure]]
- 换成 class 后上面一切原样成立，**唯一硬性差别是默认访问权限**（struct 默认 public，class 默认 private）：C9 没写访问符所以 main 里可直接 `lpy.id`，class 不写 public: 则对外全是 private
- 习惯分工：纯数据聚合用 struct，要封装（private 数据 + public 接口）用 class

## 9. 课件补充（本轮没敲代码）
- **protected**：仅本类、友元和派生类可访问（继承阶段再碰）；另有友元类 `friend class`
- **this 指针**：指向调用成员函数的当前对象；`this->value = value` 区分同名成员/参数；`return *this` 支持链式调用；静态成员函数没有 this
- **= delete / = default**（C++11）：删除指定构造函数 / 显式要求合成默认版本
- **operator= 赋值重载**：课件 MyString 例（自赋值检查 → delete[] 旧资源 → 深拷贝）；运算符重载只能用已有运算符且不改优先级
- 构造/析构顺序：A 含 B 类型成员时先构造 B 再构造 A，析构相反
- 练习项目：自定义 MyString（默认/有参/拷贝构造 + 赋值/比较/输出重载 + 析构）
