# A. 虚函数实现多态-标准示例
## 1. 头文件声明 (`virtual `- `override`)
```cpp
Vehicle.h

// Base Class Vehicle
class Vehicle {     // 纯粹接口
public:
	virtual void start_engine() = 0;  // virtual & default
    virtual ~Vehicle() = default;     // virtual & default
};

// Derived Class Car
class Car : public Vehicle {
public:
    void start_engine() override;
    ~Car() override;
};

// Derived Class Plane
class Plane : public Vehicle {
public:
    void start_engine() override;
    ~Plane() override;
};
```
- **无脑记**: **做接口** **基类**就给`virtual`

## 2. `cpp`写实现
```cpp
Vehicle.cpp

void Car::start_engine() {
    std::cout << "Car started engine! " << std::endl;
}

Car::~Car() {
    std::cout << "Car successfully destructed! " << std::endl;
}

void Plane::start_engine() {
    std::cout << "Plane started engine! " << std::endl;
}

Plane::~Plane() {
    std::cout << "Plane successfully destructed! " << std::endl;
}
```
## 3. 为什么要virtual

> 父子类**同名函数**，**非虚：** 子类**覆盖**父类--覆盖是危险歧义，应该消灭
> 父子类同**名函数**，**虚：** 子类**重写**父类(override)

# B. 动态分配&内存管理
## 1. `new` - `delete`
```cpp
	Vehicle *car_lpy = new Car();
	Vehicle *plane_lpy = new Plane();

    // ...if there's block here
    // the mem would never be freed

    delete car_lpy;         // 手动delete
    delete plane_lpy;

    car_lpy = nullptr;      // 防悬垂
    plane_lpy = nullptr;
```
### 1a. 任何`new`的对象都得手动`delete` 
- `Vehicle *car_lpy = new Car();`
- `delete car_lpy;`

### 1b. 指针防悬垂
> `delete`后的**内存空间释放**
> 指针却仍**指向已释放的空间**
- delete释放内存
- **清空指针**指向 `ptr_subject = nullptr;`

## 2. new & delete底层
### 2a. `new Car()`
- operator new 分配内存
- 在内存上调用Car构造函数
### 2b. `delete p`
- **调用**析构函数
- operator delete(底层free) 归还内存

> delete很有可能在**执行前**就遭遇**中途return**/**异常路径**
> 析构和内存释放遭跳过 -> 内存泄漏
> 现代采用[[C20 - Polymorphism#d. unique_ptr|`unique_ptr`]]由编译器保证执行

### 2c. `free`释放内存不过问对象
> `free` 一次释放**一块连续**内存
>  不需要过问该块内存是什么类型

## 3.  析构, `delete`, `free`间关系

比喻模型
- 内存: 土地
- 对象: 房子
- 构造: 盖房 (construct)
- 析构: 拆房 (destruct)
### a. `new Car()`
- 申请**sizeof(Car)的地(malloc)**
- **按Car的说明书**盖房
### b. `delete p`
- **按说明书**拆房(destruct)
- 还地
> **虚析构**意义，让编译器找**正确的说明书**析构


## 4. `unique_ptr`
### a. 独占所有权的 RAII 指针包装
- 离开作用域**自动 delete **被管理对象
### b. 编译期禁止复制
- 只能 move **转移所有权**
### c. 使用前提
- 环境有堆、代码本来在用 new/delete

# C. 切片Slicing
> 派生类对象直接放进基类对象里，会把派生类新增的那部分切掉
> 放进基类指针/引用里则不会

# D. 嵌入式开发 `Embedded Develop`

> **90% 嵌入式开发不需要动态分配**
> **实时系统**中引入**动态分配和管理** (X)

## 1.  误区: 单片机没heap，不能动态分配
### 1a. 主流单片机Heap情况
- **TM32F103(Keil):** 启动文件 `startup_xxx.s` 中可改 `Heap_Size EQU 0x200`
- **MSPM0G7 (TI GCC/Clang):**  链接脚本划有堆区，默认偏小，需调大
- **8051:** 可用 `init_mempool` 手动给 malloc 划内存

### 1b. 有堆 ≠ 能用 new/delete
- 还需要 **C++ 运行时**提供 **`operator new`**（如 newlib-nano）

### 1c. 有堆与有堆之间亦有差异 
- `FreeRTOS` 自带**独立堆(`pvPortMalloc` / heap_4.c)** 与 **C 库堆**是两套系统

## 2.  嵌入式中动态分配行为
### a. **作死**：
- 中断/实时路径里 new（分配时间不确定）
- 频繁造删不同大小对象（碎片化，最终 malloc 失败）
- 中途 return 路径漏 delete

### b. **标准（零堆）**
- 静态实例 + 基类指针数组（多态不需要堆，指针指向成员/全 局对象即可）
- 对象池（`std::array` 预建 N 槽 + 空闲标记，O(1) acquire/release，上限封死）

### c. **极端必须动态**：
- 数量由外部决定（如 TCP 连接数）
- 对象生命周期跨作用域（事件投递到其他任务）
- 跑 Linux 的板子（内存充裕，动态分配是常态）
 
> 边界：数量/生命周期编译期可知 → 静态；运行期可知但有硬上限 → 池；无上限 → 堆

## 3. 模板 -- 直接声明
```cpp
// 直接声明
Car car;                     // 直接声明（栈上）：进作用域盖房，出作用域自动拆
static Car car;              // 静态：程序启动盖房，程序结束自动拆
Car fleet[4];                // 数组：数量写死在固件里
class Controller { Car motor_; };   // 成员：随宿主对象盖/拆
// new
Car* p = new Car();          // 堆：运行期才盖房，数量/时机运行期决定
delete p;                    // 必须手动拆；忘了拆 = 资源泄漏
```

>  默认全部"直接声明"
> `new` 只在"运行期才知道数量/生命周期"时用，且必须封顶（对象池）
