
Q1: 我产出什么?
Q2: 代码include了谁的header?
Q3:我的target靠什么跑?
Q4: 产物要给别人用吗?

通用模板（每行注释 = 减法指南）

 ```cmake
   cmake_minimum_required(VERSION 3.8)   # ← 骨架，永远留
   project(my_app)                       # ← 骨架，永远留

   # ===== ① 编译选项区 =====
   if(CMAKE_COMPILER_IS_GNUCXX)
     add_compile_options(-Wall -Wextra)  # 警告全开；学习期留，产品发布可删
   endif()
   # set(CMAKE_CXX_STANDARD 17)          # 嵌入式常用；ROS 包删（ament 有默认）

   # ===== ② 依赖区：用了谁的包就加谁，没有就整段留空 =====
   find_package(ament_cmake REQUIRED)          # 仅 ROS 包：colcon 的钩子，ROS 必留
   find_package(rclcpp REQUIRED)               # 写 rclcpp 节点才留
   find_package(example_interfaces REQUIRED)   # 用它的消息类型才留
   # find_package(OpenCV REQUIRED)             # 嵌入式用第三方库：加在同一区

   # ===== ③ 产物区：一个可执行/库 = 一段，有几个写几段 =====
   add_executable(app src/main.cpp)            # target 本体
   # —— 以下两行是"嵌入式接依赖"的写法 ——
   # target_include_directories(app PRIVATE inc)      # 有自己头文件才留
   # target_link_libraries(app pthread)               # 链接系统库才留
   # —— 这一行是"ROS 接依赖"的写法（三合一，替代上面两行）——
   ament_target_dependencies(app rclcpp example_interfaces) # ROS 必留，漏=头找不到

   # ===== ④ 安装区：产物要不要给别人/给工具找到？ =====
   install(TARGETS app DESTINATION lib/${PROJECT_NAME})  # ROS：ros2 run 靠它，必留
   # 嵌入式自己跑 build/app → ④ 整段删掉

   ament_package()   # ROS 包收尾，必须在文件末尾（嵌入式删）
 ```


## 一. 创建项目
### 1.  Initialization
CMake **Target** 思想: `app`
```cpp
cmake_minimum_required(VERSION 3.28)  // CMake minimum

project(cmake_project_name)           // camke project

set(CMAKE_CXX_STANDARD 17)            // cpp standard
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(app                    // executable target
	// target app 的 cpp file
	src/main.cpp
	src/motor.cpp
)
```
### 2.  头文件`include`路径
```cpp
target_include_directories(app PRIVATE
	${PROJECT_SOURCE_DIR}/inc
)
// target 'app' 的include path 'inc'
```

### 3. 链接库 (暂不了解)
```cpp
target_link_libraries(app PRIVATE pthread)
```

## 二. `build`目录
### 1. `build`目录污染(易踩坑)
```cpp
build/CMakeCache.txt
```
>CMake 会把配置结果、缓存、生成的构建文件等保存在 `build directory` 中；如果项目位置、工具链、CMake 配置等发生变化，旧缓存**可能**导致配置异常。

**修改项目结构**后:
```bash
rm -rf build
mkdir build
cd build
cmake ..
```

## SUM
>现代 CMake 核心不是“告诉 CMake 怎么编译每个文件”
>而是**描述 target** & **描述 Target 之间的关系**
