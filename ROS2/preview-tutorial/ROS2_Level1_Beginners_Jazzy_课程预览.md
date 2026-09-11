# 课程预览：ROS 2 for Beginners (ROS Jazzy - 2026) — Level 1

> 这是 Edouard Renard「ROS 2 for Beginners」系列的第 **1 级（Level 1）**，也是你（ROS 2 纯新手）的**正式起点**。
> 配套文档：《ROS2_TF_URDF_RViz_Gazebo_课程预览.md》（Level 2，学完本课再打开）。
> 本文目的：① 了解这门课讲什么、怎么学、装什么环境；② 提前建立对 Level 1 全程英文术语的概念，降低全英听课门槛。

---

## 目录

1. [课程基本信息](#1-课程基本信息)
2. [这门课讲什么](#2-这门课讲什么)
3. [适合谁 / 需要什么基础](#3-适合谁--需要什么基础)
4. [内容走向预览](#4-内容走向预览)
5. [上课前要准备什么](#5-上课前要准备什么)
6. [全英课程学习建议](#6-全英课程学习建议)
7. [核心英文缩写与术语速查](#7-核心英文缩写与术语速查)（重点章节）
8. [Level 1 → 2 → 3 学习路线](#8-level-1--2--3-学习路线)

---

## 1. 课程基本信息

| 项目 | 内容 |
|---|---|
| **课程名称** | ROS 2 for Beginners (ROS Jazzy - 2026) |
| **一句话副标题** | Master ROS2 Basics and Become a Robot Operating System Developer（掌握 ROS 2 基础，成为机器人开发者） |
| **讲师** | Edouard Renard（软件工程师 / 机器人讲师，也是《ROS 2 from Scratch》一书作者） |
| **课程链接** | https://www.udemy.com/course/ros2-for-beginners/ |
| **版本** | 基于 **ROS 2 Jazzy Jalisco（2024 LTS）**，2026 内容更新 —— 目前系列里**最新**的 Level 1 |
| **总时长** | 约 13 小时 14 分 |
| **评分 / 人数** | ≈ 4.7 分，约 31,000+ 学员（参考数据，以页面为准） |
| **语言覆盖** | **Python 与 C++ 双语教学**（每个概念两种语言各讲一遍，任选一种跟） |
| **课程语言** | 全英文 |
| **额外渠道** | 同一门课也发布在 Manning、O'Reilly 等平台（买 Udemy 版即可） |

---

## 2. 这门课讲什么

一句话：**用 ROS 2 让"多个程序互相通信"这件事变得不神秘**，然后动手写出自己的 ROS 2 程序。

课程核心知识点（学完你将会）：

| 学到什么                             | 大白话                                                            |
| -------------------------------- | -------------------------------------------------------------- |
| 安装 ROS 2 环境                      | Ubuntu + ROS 2 Jazzy 装好，会开工作空间                                 |
| **Node（节点）**                     | 知道 ROS 程序 = 一个个独立小进程                                           |
| **Topic + Publisher/Subscriber** | 写代码实现"一个节点发数据、另一个节点收数据"                                        |
| **Message（消息）**                  | 自己定义要传的数据格式（.msg 文件）                                           |
| **Service（服务）**                  | 写"一问一答"式的请求/响应程序                                               |
| **Parameter（参数）**                | 不用改代码就能调整节点的配置                                                 |
| **Launch File（启动文件）**            | 一键启动多个节点                                                       |
| **ros2 命令行工具**                   | 调试三板斧：`ros2 node list` / `ros2 topic list` / `ros2 topic echo` |
| **一个完整小项目**                      | 基于 **Turtlesim（乌龟模拟器）**，从零搭一个完整 ROS 2 应用（含练习与期末多步骤项目）          |

> 教学特点：**代码实战驱动**（不是理论课）、Python/C++ 双版本任选、每节配小练习、结尾有综合项目。缺点（据反馈）：对底层架构讲得不深、安装步骤偶有坑——这正是本文第 5 节要帮你提前避开的。

---

## 3. 适合谁 / 需要什么基础

| 维度     | 要求                                                                 |
| ------ | ------------------------------------------------------------------ |
| ROS 经验 | ❌ **完全不需要**（不需要 ROS 1 或 ROS 2 经验，专门为零基础设计）                         |
| 编程     | 建议有一点 **Python 或 C++** 基础（哪怕只写过简单脚本）。两种都不会也别慌，但先花几天补 Python 基础体验更顺 |
| Linux  | 会基本终端操作即可（`cd`、`ls`、`mkdir` 等），不会也问题不大，视频会带你                       |
| 英语     | 全英授课，建议配合本文件第 7 章术语表 + 英文字幕                                        |
| 机器配置   | 能流畅跑 Ubuntu 的电脑（建议 ≥ 8GB 内存；虚拟机则 ≥ 16GB 更稳）                        |

---

## 4. 内容走向预览

> 按课程宣传、教学习惯与学员反馈归纳的"大概率节奏"（确切分节标题以 Udemy 页面为准，登录后可见 Section List）。

1. **开篇 & 环境安装**
   - 课程简介、安装 Ubuntu/ROS 2 Jazzy、配置工作空间、跑通第一个示例。
2. **认识 ROS 2：从 Turtlesim 开始**
   - 用官方"乌龟"演示（`turtlesim`）直观感受：一个节点发指令、一个节点动起来；用 `rqt_graph` 看到节点和话题的连线图。
3. **写第一个程序：Publisher / Subscriber**
   - Python 版与 C++ 版各写一遍："hello world" 级别的节点互发消息。
4. **深入 Node 与 Topic**
   - 名字、命名空间、消息类型；`ros2 topic info/echo/hz` 等调试命令。
5. **自定义 Message（.msg）**
   - 定义自己的数据结构并在发布者/订阅者中使用。
6. **Service（.srv）与 Parameter**
   - 写服务端/客户端、调参数。
7. **Launch File**
   - 用 `.launch.py` 一次性把整套系统拉起来。
8. **综合项目（Final Project）**
   - 用 Turtlesim 环境把前面学的全部串起来，做一个完整应用（多步骤、需要动脑）。

> 高频命令会反复出现：`colcon build`、`source install/setup.bash`、`ros2 run`、`ros2 pkg create`、`ros2 launch`。

---

## 5. 上课前要准备什么

这是学员反馈里"最容易卡住"的部分，提前准备可省大量时间：

1. **系统选择（关键）**
   - ROS 2 **Jazzy 官方支持 Ubuntu 24.04**。三种方式任选：
     - **① 双系统**（推荐：性能最好、最省心，适合主力学习机）；
     - **② 虚拟机 VirtualBox / VMware**（不想动硬盘分区就选它，注意给足内存和 CPU）；
     - **③ WSL2（Windows 里跑 Ubuntu）**——能用，但涉及 GUI（rqt、RViz）时偶尔有坑，新手不优先推荐。
   - ⚠️ 不要装错 Ubuntu 版本（Jazzy 别配 22.04，Humble 才配 22.04）。
2. **课前 Linux 三件套**：终端里会 `cd / ls / mkdir / nano`（或 vim），会 `sudo` 装软件。
3. **准备两个窗口的习惯**：ROS 学习 90% 时间 = 终端 A + 终端 B（一个跑节点、一个敲命令看数据）。
4. **编程语言二选一**：建议**先跟 Python**（上手快、报错友好），学完有空再回头补 C++ 版本对比理解。两者概念完全一样。
5. 打开**英文字幕**，准备好做笔记的文档。

---

## 6. 全英课程学习建议

1. 先扫第 7 章术语表，"混脸熟"后再看视频。
2. 语速快就开 0.75x + 字幕；关键词（node、topic、publish、subscribe、callback）反复出现，听几次自然就熟。
3. **老师敲代码/命令时暂停自己敲一遍**——不敲等于没学。
4. 安装环节卡住，先看课程 Q&A 区，再 Google 报错原文（全英文报错直接复制搜索最快）。
5. 心态：ROS 报错是常态，"见错 → 搜错 → 解决"就是这门课真正教你的能力。

---

## 7. 核心英文缩写与术语速查

> 本章是重点。Level 1 的词汇集中在**安装环境、通信机制、命令行、代码编程**四个主题。
> （建模/仿真类词汇如 TF、URDF、RViz、Gazebo 在配套的 Level 2 文档里，这里不重复。）

### 7.1 环境与安装

| 术语 | 说明 / 全称 | 大白话 |
|---|---|---|
| **Ubuntu** | —— | 最常用的 Linux 发行版，ROS 2 官方支持系统 |
| **发行版 Distribution** | —— | ROS 按年发版本。本课是 **Jazzy Jalisco（2024 LTS）**；LTS=长期支持版（5 年） |
| **LTS** | Long Term Support | 长期支持版，稳定、更新久——选 ROS 就选 LTS |
| **Terminal** | 终端 | 命令行窗口，ROS 的一切操作几乎都在这发生 |
| **Virtual Machine (VM)** | 虚拟机 | 在现有系统里"套一个"Ubuntu（VirtualBox 等软件） |
| **Dual Boot** | 双系统 | 硬盘上装两个系统，开机选择进哪个 |
| **WSL2** | Windows Subsystem for Linux | 微软官方让 Windows 直接跑 Linux 的方案 |
| **Docker** | —— | 容器技术（可选方案，先不学也行） |
| **Shell** | —— | 终端里解释命令的程序（默认 bash） |
| **source** | —— | 让环境配置立即生效的命令，如 `source /opt/ros/jazzy/setup.bash` —— **每次开新终端基本都要先跑** |

### 7.2 顶层概念（第一节课必讲）

| 术语            | 全称 / 说明                   | 大白话                                                            |
| ------------- | ------------------------- | -------------------------------------------------------------- |
| **ROS 2**     | Robot Operating System 2  | 机器人软件**中间件/通信框架**（不是操作系统），让节点间能收发数据                            |
| **DDS**       | Data Distribution Service | ROS 2 底层的通信协议标准（不用深究，知道"底层靠它传数据"即可）                            |
| **Workspace** | 工作空间                      | 你的代码总目录，含 `src`（源码）、`build`（编译中间物）、`install`（装好的可执行）、`log`（日志） |
| **Package**   | 功能包                       | ROS 代码基本单位：一个包 ≈ 一个功能模块（如"摄像头驱动包""导航包"）                        |
| **colcon**    | ——                        | ROS 2 的编译工具：`colcon build` 编译你 src 里所有包                        |
| **ament**     | ——                        | 构建系统名，建包时选 `ament_python`（Python包）或 `ament_cmake`（C++包）        |

### 7.3 通信机制核心词（本课核心）

| 术语                  | 全称 / 说明                  | 大白话                                                            |
| ------------------- | ------------------------ | -------------------------------------------------------------- |
| **Node**            | 节点                       | ROS 程序里一个独立小进程（如"键盘控制节点""乌龟显示节点"）                              |
| **Topic**           | 话题                       | 节点间**单向持续**传数据的"频道"，如 `/turtle1/cmd_vel`（乌龟速度指令）               |
| **Publisher**       | 发布者                      | 往话题上"发"数据的节点                                                   |
| **Subscriber**      | 订阅者                      | 从话题上"收"数据的节点                                                   |
| **Message (msg)**   | 消息                       | 话题里数据的**格式定义**（如 Twist=线速度+角速度），自定义消息写 `.msg` 文件               |
| **Rate**            | 频率                       | 每秒发几次（如 `rate = 10` = 每秒 10 条）；单位 Hz（赫兹）                       |
| **Callback**        | 回调函数                     | 收到数据时自动被调用的函数——**订阅者编程的核心**                                    |
| **Spin**            | 旋转/自旋                    | 让节点"转起来"持续处理消息的函数（`rclpy.spin(node)`）——不 spin 程序就跑不起来          |
| **QoS**             | Quality of Service（服务质量） | 消息传输可靠性设置（如"丢了重发 vs 丢了就丢"），初学先默认，见到知道是啥                        |
| **Service**         | 服务                       | **一问一答**通信：客户端（Client）发请求（Request），服务端（Server）回响应（Response）    |
| **srv**             | Service 文件格式             | 定义服务"问什么答什么"的 `.srv` 文件（`---` 分隔上下两段）                          |
| **Client / Server** | 客户端 / 服务端                | 服务通信的两个角色                                                      |
| **Parameter**       | 参数                       | 节点的"旋钮/配置项"，`ros2 param set` 运行时就能改                            |
| **Action**          | 动作                       | 长耗时任务通信（goal→feedback→result）——**主要在 Level 3 详讲**，Level 1 提到即可 |

### 7.4 接口与消息类型（写代码会遇到）

| 术语 | 说明 | 大白话 |
|---|---|---|
| **Interface** | 接口 | 消息类型的总称（msg/srv/action 都算） |
| **std_msgs** | 标准消息库 | 基础类型包：`String`（字符串）、`Int32`（整数）、`Float32`（小数）、`Bool`（布尔） |
| **geometry_msgs** | 几何消息库 | 几何相关：`Twist`（速度）、`Point`、`Pose`（位置+朝向）——乌龟遥控用的就是 Twist |
| **sensor_msgs** | 传感器消息库 | 传感器数据：`LaserScan`（激光）、`Image`（图像）等（本课稍带，Level 2 常用） |
| **Header** | 消息头 | 很多消息自带的公共段：`stamp`（时间戳）+ `frame_id`（所属坐标系） |
| **frame_id** | 坐标系 ID | 这条数据是"在哪个坐标系下测的"（Level 2 讲 TF 时重点） |
| **Dependency** | 依赖 | 你的包要依赖哪些库/包，写在 `package.xml` 里；忘了写最常见的报错就是找不到依赖 |

### 7.5 命令行速查（背熟这组就能完成 80% 操作）

| 命令 | 作用 | 备注 |
|---|---|---|
| `ros2 run <包名> <节点名>` | 运行某个包里的一个节点 | 例：`ros2 run turtlesim turtlesim_node` |
| `ros2 node list` | 列出当前所有节点 | 排障第一步 |
| `ros2 node info <节点>` | 查看某节点的收发话题、服务 | |
| `ros2 topic list` | 列出所有话题 | 加 `-t` 显示类型 |
| `ros2 topic echo <话题>` | 实时打印某话题数据 | **调试神器**，看数据有没有在传 |
| `ros2 topic info <话题>` | 查看话题类型、谁在发谁在收 | |
| `ros2 topic hz <话题>` | 测话题更新频率 | 判断是否卡死 |
| `ros2 topic pub <话题> <类型> "<数据>"` | 手动往话题发一条数据 | 测订阅者用 |
| `ros2 interface show <类型>` | 查看消息类型字段 | 例：`ros2 interface show geometry_msgs/msg/Twist` |
| `ros2 service list / call` | 列出/调用服务 | |
| `ros2 param list / get / set` | 查看/修改参数 | |
| `ros2 pkg create <名字>` | 新建一个功能包 | 后面常跟 `--build-type ament_python` 等参数 |
| `ros2 launch <包> <文件.launch.py>` | 一键启动整套节点 | |
| `colcon build` | 编译工作空间 | 只编一个包加 `--packages-select <包名>` |
| `source install/setup.bash` | 让新编译的包生效 | 新终端基本都要先跑 |
| `rqt_graph` | 图形化显示节点/话题关系 | 理解通信架构的利器 |

### 7.6 编程相关词汇（跟 Python/C++ 视频时遇到）

| 术语 | 说明 | 大白话 |
|---|---|---|
| **rclpy / rclcpp** | ROS 客户端库（Python / C++） | 写 ROS 代码时 `import rclpy` / `#include "rclcpp/rclcpp.hpp"` 的就是它 |
| **Entry point** | 程序入口 | Python 包 `setup.py` 里把命令名映射到函数的地方——配错会报 "command not found" |
| **Executable** | 可执行文件 | 编译后能直接运行的程序（C++ 包相关） |
| **CMakeLists.txt** | —— | C++ 包的构建配置脚本 |
| **setup.py / setup.cfg** | —— | Python 包的构建/打包配置 |
| **package.xml** | —— | 每个包都有的"元信息"：名字、依赖、许可证（别漏写） |
| **Doxygen / Docstring** | 代码注释 | 不重要，知道是注释即可 |

### 7.7 高频课堂口语 + 常见报错英文

**课堂用语**（同 Level 2 文档，此处不重复，可互相参照）：
- "Open a terminal and type…" 打开终端输入…；"Let's run the node" 运行节点；
- "You should see the turtle moving" 你应该看到乌龟在动；
- "This error means…" 这个报错意味着…；"Don't worry, it's normal" 别担心，这很正常。

**高频报错 → 秒懂：**

| 报错片段 | 意思 | 常见原因 |
|---|---|---|
| `command not found` | 命令不存在 | 没 `source` 环境 / 没 `colcon build` / 名字拼错 |
| `Package '<xx>' not found` | 找不到包 | 没编译 / 没 source / 包名拼错 |
| `resource not found` | 资源找不到 | Python 包入口点没配好 |
| `Unable to talk to the master` | （ROS1 报错） | 你装错教程了，ROS 2 没有 master 概念 |
| `No executable found` | 找不到可执行文件 | 没编译 / CMake 没加 target |
| `segmentation fault (core dumped)` | 段错误 | C++ 指针/内存问题，检查代码 |
| `ModuleNotFoundError` | Python 找不到模块 | 依赖没装 / 环境没 source |
| `port already in use` | 端口占用 | 上个进程没关干净，`kill` 掉再试 |

---

## 8. Level 1 → 2 → 3 学习路线

Edouard Renard 的「ROS 2 for Beginners」是一个系列，按顺序学：

| 级别 | 课程 | 内容 | 适合 |
|---|---|---|---|
| **Level 1**（本课） | ROS 2 for Beginners (ROS Jazzy - 2026) | 通信基础：node/topic/msg/service/param/launch，双语 | 纯新手 ← **你现在在这** |
| **Level 2** | ROS 2 for Beginners Level 2 – TF \| URDF \| RViz \| Gazebo | 坐标变换 + 机器人建模 + RViz/Gazebo 仿真 | 学完 Level 1 的人 |
| **Level 3** | ROS 2 for Beginners Level 3 – Advanced Concepts | Action、生命周期节点、Executors 等进阶 | 有 ROS 2 基础的人 |

**进阶选方向**（Level 2/3 之后按兴趣）：Nav2（导航）/ MoveIt（机械臂）/ SLAM（建图定位）/ 真机。

---

### 附：30 秒自测（学完 Level 1 你应该能回答）

1. ROS 2 里"一个程序进程"叫什么？
2. publisher 和 subscriber 通过什么机制通信？数据格式叫什么？
3. 改代码后要让程序生效，先敲哪两个命令？（提示：build + source）
4. 想实时看某个话题在传什么数据，用哪条命令？
5. `ros2 node list` 和 `ros2 topic list` 分别看什么？
6. 服务（service）和话题（topic）的本质区别是什么？（一问一答 vs 持续广播）

> 这 6 题答案是整门课的核心骨架。现在答不上来没关系——学完回来再测，全对就说明基础扎实，可以进军 Level 2 了。
