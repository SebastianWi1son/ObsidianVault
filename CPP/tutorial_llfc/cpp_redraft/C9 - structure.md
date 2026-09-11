## 1. 结构体定义与成员
-  **struct**把相关数据打包成一个自定义类型，成员用点运算符访问

```cpp
struct Student {
    int id;
    std::string name;
    float grade;
};
```

> 散落的 id/name/grade 打包成一个类型后，传递、存储、返回都以 Student 为单位

## 2. typedef struct 写法

```cpp
// --- d4-9-structure/src/main.cpp ---
typedef struct Student {
    int id;
    std::string name;
    float grade;
} Student;
```

- 结尾的 Student 是**类型别名**，之后写 `Student lpy;` 不必带 struct 前缀
- C 语言习惯（C 里必须 `struct Student lpy;`）；C++ 中类型名直接可用，typedef 可省

## 3. 构造函数

```cpp
// --- d4-9-structure ---
// 创建对象时调用，传入参数赋值给各个成员变量
Student(int studentID, std::string studentName, float studentGrade)
    :id(studentID), name(studentName), grade(studentGrade) {}
```

- **参数初始化列表**：成员创建时直接初始化，而非先默认构造再赋值
- 构造函数与结构体**同名**、无返回类型

### a. 无参/有参构造并存

```cpp
// --- test-StudentManageSys/inc/stud_manage.h ---
Student()
    :ID(0), name(""), grade(0.0f) {}   // 无参：成员全默认值
Student(long StudentID, std::string StudentName, float StudentGrade)
    :ID(StudentID), name(StudentName), grade(StudentGrade) {}
```

- 参数个数不同构成**重载**，按调用形式选择

## 4. 两种初始化写法

```cpp
Student lpy = { 20315, "crash", 83.5f };   // 聚合初始化，按声明顺序填值
Student lwb(20316, "shen", 98.5);          // 调用构造函数
```

> 前者按成员声明顺序赋值，直观；后者走构造函数，可设默认值/检查，更可控

## 5. 成员访问与输出限制

```cpp
// std::cout << lpy << std::endl;  // X 需要重载 << 运算符
std::cout << lpy.id << " " << lpy.name << std::endl;
```

- struct 没有自带输出格式，只能**逐个成员 cout**

## 6. 声明与定义分离

```cpp
// --- stud_manage.h ---
struct Student {
    void printInfo() const;   // struct 内只声明
};
```

```cpp
// --- stud_manage.cpp ---
void Student::printInfo() const {   // Student:: 指明所属类型
    std::cout << "ID: " << id << ...
}
```

- **Student::** 作用域符：.cpp 里定义成员函数时标明属于哪个 struct
- **const 成员函数**：承诺不修改成员，只读场景可调用

## 7. 练习记录
### a. 示范项目 StudMangeSys（vector 方案）

- struct 定义进 .h（成员 + 构造函数 + printInfo），include guard 防重定义 → [[C4 - redefinition]]

```cpp
// --- stud_manage.cpp ---
void addStudent(std::vector<Student>& students, int id, const std::string& name, float grade) {
    students.push_back(Student(id, name, grade));   // 引用传参，改动作用于调用方
}

void displayStudents(const std::vector<Student>& students) {
    for (const auto& student : students)   // const auto& 只读遍历免拷贝
        student.printInfo();
}
```

- **vector<Student>&**：要改就普通引用，只读就 const 引用 → [[C12 - vector]]
- searchStudentByID：遍历比对 id，命中 printInfo() 后 return
- runSystem() 内 do-while 菜单循环，main.cpp 只剩 `runSystem();`

### b. 自测项目 test-StudentManageSys（extern 数组方案）

```cpp
// --- stud_manage.h ---
constexpr int MAX_STUDS = 10;   // 编译期常量放 .h 共享
extern Student database_stud[MAX_STUDS];   // 声明全局数据库
```

```cpp
// --- stud_manage.cpp ---
Student database_stud[MAX_STUDS];   // 唯一一份定义
static int num_registered;          // 已注册计数，仅本文件可见

void register_student(const Student &student) {
    database_stud[num_registered] = student;
    num_registered++;
}
```

```cpp
Student* search_student_by_id(const int &id_target) {
    for (int i = 0; i < num_registered; ++i)
        if (database_stud[i].ID == id_target) return &database_stud[i];
    return nullptr;   // 找不到返回空指针
}
```

- 调用端：`print_student_info(*search_student_by_id(20316));` 解引用取结构体
- **extern 数组 vs vector**：容量编译期固定 vs 动态增长
- **static** 计数器藏在 .cpp：外界只能经 register_student 写数据库

## 8. 课件补充（原笔记未记）
- **struct vs class**：仅默认访问权限不同（public vs private），封装等留给 [[C19 - class]]
- 嵌套结构体：成员可以是另一个 struct（Person 内放 Address）
- 结构体数组：`Student students[3] = {{1001,"A",89.5f}, ...};`
- 结构体指针：`carPtr->brand` 等价 `(*carPtr).brand`
- `using Student = struct {...};` 同样起别名
- 值传递会整体拷贝一份结构体，传参优先引用
