## 1. rename a node/topic at runtime
```zsh
ros2 run my_pkg my_node --ros-args --remap __node:=new_name
```

```zsh
ros2 run my_pkg my_node --ros-args --remap robot_news:new_name
```
- `node` / `topic` **名字可换**并非硬性固定 **Editable**
- `node`默认名字 和 文件名字 一致(**惯例**非硬性要求)

## 2. tips - build py pkg
```zsh
colcon build --packages-select my_pkg --symlink-install
```
- 直接替换install/目录下py文件为源码，省去build操作(**cpp不适用**)
## 3. ros2 node *
1. ros2 node list
2. ros2 node info

## 4. ros2 topic *
1. ros2 topic list                      # 话题list
2. ros2 topic echo /topic        # echo话题内容(data)
3. ros2 topic info /topic -v     # 谁在收/谁在发
4. ros2 topic hz /topic            # 实测发布频率

## 5. `rqt`
**rqt graph:**  display the node relations in graph

## 6. `turtlesim`
1. `turtlesim_node`
2. `turtle_teleop_key`
> **意义:**               话题topic数据流 -> 可见的海龟动作move
> **pub/sub闭环:** teleop发指令话题 -> turtlesim订阅并执行

## 7. 读节点名片
```zsh
ros2 node info /turtlesim
```
**Output:**
```zsh
/turtlesim # 小海龟
   Subscribers:
     /parameter_events: rcl_interfaces/msg/ParameterEvent
     /turtle1/cmd_vel: geometry_msgs/msg/Twist     # topic: pkg/msg/type
   Publishers:
     /parameter_events: rcl_interfaces/msg/ParameterEvent
     /rosout: rcl_interfaces/msg/Log
     /turtle1/color_sensor: turtlesim/msg/Color
     /turtle1/pose: turtlesim/msg/Pose
   Service Servers:
     /clear: std_srvs/srv/Empty
     /kill: turtlesim/srv/Kill
     /reset: std_srvs/srv/Empty
     /spawn: turtlesim/srv/Spawn
     /turtle1/set_pen: turtlesim/srv/SetPen
     /turtle1/teleport_absolute: turtlesim/srv/TeleportAbsolute
     /turtle1/teleport_relative: turtlesim/srv/TeleportRelative
     /turtlesim/describe_parameters: rcl_interfaces/srv/DescribeParameters
     /turtlesim/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
     /turtlesim/get_parameters: rcl_interfaces/srv/GetParameters
     /turtlesim/get_type_description: type_description_interfaces/srv/GetTypeDescription
     /turtlesim/list_parameters: rcl_interfaces/srv/ListParameters
     /turtlesim/set_parameters: rcl_interfaces/srv/SetParameters
     /turtlesim/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
   Service Clients:

   Action Servers:
     /turtle1/rotate_absolute: turtlesim/action/RotateAbsolute
   Action Clients:
```
1. **Sub/Pub(话题匹配):** "**话题名** : **消息类型**"
2. **Service/Action:** 印证"节点 = pub/sub/service/action 接口的集合体"
> `/parameter_events` & `/rosout`: 框架自动附带(参数系统+日志)

## 8. 读消息类型
```zsh
ros2 interface show geometry_msgs/msg/Twist  # pkg/msg/type
```
**Output:**
```zsh
 Vector3  linear
         float64 x
         float64 y
         float64 z
 Vector3  angular
         float64 x
         float64 y
         float64 z
```
- Twist = Vector3 linear + Vector3 angular (各含 float64 x/y/z)
- msg可嵌套msg (struct 套 struct)

## 9. `/cmd_vel` — 事实标准话题
- teleop发Twist → /turtle1/cmd_vel → turtlesim订阅执行
- 几乎所有机器人驱动都订阅 /cmd_vel 收速度指令

