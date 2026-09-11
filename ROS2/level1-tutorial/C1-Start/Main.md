## 1. Source
### 1.1 source 是什么
> setup.zsh 这类文件里全是 `export XXX=...`
> source把指定文件里的命令带到**当前终端进程内**执行
> 若直接执行是开**子进程**跑，变量修改**传不回当前终端**

### 1.2 三条铁律
1. **环境随shell生灭:** 同一终端source一次管到关终端
2. **环境随进程继承:** source后启动的任何程序都继承终端环境
3. **新终端仅初始化:** 新终端只继承~/.zshrc

### 1.3 两层配置
- **ROS2系统级:** 让终端认识ROS2命令, rclcpp, demo_node
- **工作区项目级:** 让ROS2认识项目build包

### 1.4 症状
1. 找不到"**ros2 命令**, **rclpy**"→ 补**ROS2系统层**
```zsh
zsh: command not found: ros2
# fix
source /opt/ros/jazzy/setup.zsh
```
2. 找不到"**你的包/可执行/模块**"→ 补**项目层**
```zsh
Package 'my_cpp_pkg' not found
# fix
source ./install/setup.zsh
```
3. TIPS: **clion继承环境**  & **不占用终端前台**
```zsh
# 终端解绑 不占用前台 静默运行 继承环境
nohup clion >/dev/null 2>&1 & disown
```

| 片段            | 作用                            |
| ------------- | ----------------------------- |
| `nohup clion` | 忽略 SIGHUP 挂断信号 → 关终端不杀 CLion  |
| `>/dev/null`  | stdout 丢进黑洞 → 日志不刷屏           |
| `2>&1`        | stderr 复制到 fd1 去向（黑洞）→ 报错也不刷屏 |
| `&`           | 后台执行，shell 立即返回提示符            |
| `disown`      | 从作业表除名，shell 退出不等它不清理         |

## 2. System Build
![[System Build Logic Map.canvas]]
- **colcon build:** 本质解决**多packages间协作**问题
- **clion build:** 一次只针对**单个excutable**进行构建，**无法看到同工作区其他自定义包**

## 3. New ROS2 Project
1. **source** [[#1. Source|ROS2系统层]]
2. `cd workspace/`
3. 进`cd src/`
4. 建包 `ros2 pkg create my_pkg --build-type ament_cmake --dependencies rclcpp`
5. 回工作区 cd .. 
6. 首次build `colcon build` -> install/, log/, build/
7. 进包开工 `cd my_pkg/src/`
8. **coding**
9. 回工作区
10. 正式build `colcon build --packages-select my_pkg`
11. **source** [[#1. Source|项目层]]
12. 运行 `ros2 run pkg_name executable`

## 4. Demo Talker & Listener

![[Logic Map talker & listener.canvas]]

## 5. DDS节点发现
```zsh
ros2 node list
#output
/cpp_node
/talker
```
- 去中心化发现，**无中央注册表**
- **可见条件:** 同`ROS_DOMAIN_ID` + 网络可达
- 列表粒度: 仅证明参与者**进程级存在**(不作为节点健康真相)

**拓展: CLI查询层级**
- **拓扑/在线:** ros2 node list
- **发布/订阅:** ros2 node info
- **数据流真相:** ros2 topic echo
> CLI查询常由**daemon**节点执行
> 各工具本身也是**节点**

## 6. The First Program

### 6.1 `class Node`
```cpp
// Derived from rcl universal basic node
class MyNode: public rclcpp::Node {
public:
	MyNode(): Node("node_name"), counter_(0) {
		timer_ = this->create_wall_timer(std::chrono::seconds(1),
									std::bind(&MyNode::timer_callback, this));
	}
	
private:
	void timer_callback() {
		counter_++;
		RCLCPP_INFO(this->get_logger(), "Hello, %d", counter_);
	}								 
	
	rclcpp::TimerBase::SharedPtr timer_;
	int counter_;
};
```

`create_wall_timer(1s, bind(timer_callback, this))`
- 注册1s的定时事件event挂进系统 
- bind好的回调callback存定时器身上
> create_wall_timer是软件定时器，ms级且有抖动
> 返回shared_ptr, 必须存成员变量 (存局部变量函数结束 即析构 即定时器销毁)

### 6.2 `main`
```cpp
int main(int argc, char **argv) {
	rclcpp::init(argc, argv);                 // rcl init
	auto node = std::make_shared<MyNode>();   // create a MyNode with RAII
	
	rclcpp::spin(node);                       // 节点注入spin事件泵, 循环执行
	
	rclcpp::shutdown();                        // 反初始化
	return 0;
}
```

`spin` (executor)
- 阻塞等待事件，然后取出执行