## 1. Basic Usage (XML)
### 1.1 最小 launch 文件
```xml
<launch>
    <node pkg="my_py_pkg" exec="number_publisher" />
    <node pkg="my_cpp_pkg" exec="number_counter" />
</launch>
```

### 1.2 运行
```zsh
ros2 launch my_ros_bringup number_app.launch.xml
ros2 launch my_ros_bringup number_app.launch.xml --show-args   # 看这个文件有哪些 arg
```
- **两段式:** `ros2 launch <pkg> <文件名>`, 扩展名**不能省**(不做补全)
- 只给包名 → 报错 `file 'None' was not found ...`

### 1.3 node 元素属性
| 属性 | 作用 | 例 |
| --- | --- | --- |
| `pkg` | 包名 | my_py_pkg |
| `exec` | 可执行名(不是 .py/.cpp 文件名) | number_publisher |
| `name` | **改节点名**(覆盖代码里的) | my_number_publisher |
| `namespace` | 命名空间前缀 | /robot_01 |

### 1.4 remap & param
```xml
<node pkg="my_py_pkg" exec="number_publisher" name="my_number_publisher" namespace="/robot_01">
    <remap from="number" to="my_number" />       <!-- 话题改名 -->
    <param name="number_to_publish" value="6" /> <!-- 传参数 -->
</node>
```
- `remap` 只有 `from` / `to` 两个属性 → 写成 `name`/`value` 会报 `Attribute 'from' ... not found in Entity 'remap'`
- `param` 只有 `name` / `value`

## 2. namespace & 名字里的 `/` (重点)
### 2.1 节点全名怎么来的
```text
节点全名 = /<namespace>/<name>
```
实测: `namespace="/robot_01"` + `name="my_number_publisher"` → `ros2 node list` 显示 `/robot_01/my_number_publisher`

### 2.2 node name 不能带 `/`
| launch 写法 | 结果 |
| --- | --- |
| `name="node_a"` | ✅ |
| `name="/node_b"` | ❌ `Invalid node name: must not contain characters other than alphanumerics or '_'` |

- `name` 只允许 **字母 / 数字 / 下划线**
- `ros2 node list` 里开头的 `/` 是"完全限定名(FQN)"的格式, **不是 name 的一部分**
- **代码与 launch 规则一致:** `Node("number_publisher")` 也不能带 `/`

### 2.3 namespace 带不带 `/` 等价
| 写法 | 结果 |
| --- | --- |
| `namespace="ns_plain"` | /ns_plain/node_x |
| `namespace="/ns_plain"` | /ns_plain/node_x |

### 2.4 真正分"相对名 / 绝对名"的是话题名 & 服务名
```text
/xxx  → 绝对名: 不拼 namespace
xxx   → 相对名: 拼 namespace 前缀
```
实测(节点在 `namespace="/ns_rel"` 下):
| 写法 | 实际话题 |
| --- | --- |
| `<remap from="number" to="rel_number" />` | /ns_rel/rel_number |
| `<remap from="number" to="/abs_number" />` | /abs_number (逃出 ns) |

同一条规则也适用于**代码里**:
```python
self.create_publisher(Int64, "rel_topic", 10)    # → /ns_code/rel_topic
self.create_publisher(Int64, "/abs_topic", 10)   # → /abs_topic
```

### 2.5 统一心智
| 名称 | 能带 `/` 吗 | 带 `/` 的含义 |
| --- | --- | --- |
| node name | ❌ 不能(语法非法) | — |
| namespace | ✅ 能, 带不带等价 | 只是写法 |
| topic / service 名 | ✅ 能 | 绝对名, 逃出 namespace |

**惯例:** 代码里话题名**不写** `/`(用相对名) → namespace 才能生效 → 同一份代码可以同时给多个机器人跑(`/robot_01`, `/robot_02`)

## 3. py & YAML launch (示范, 实际几乎只用 XML)
### 3.1 YAML
```yaml
launch:
- node:
    pkg: my_py_pkg
    exec: number_publisher
    name: node_y
    namespace: /ns2
    param:
    - name: number_to_publish
      value: 7
```

### 3.2 Python
```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(package="my_py_pkg", executable="number_publisher",
             name="node_py", namespace="/ns3",
             parameters=[{"number_to_publish": 8}],
             remappings=[("number", "py_number")]),
    ])
```

### 3.3 三格式对比
| | XML | YAML | Python |
| --- | --- | --- | --- |
| 可读性 | 好 | 一般 | 差(像代码) |
| 表达力 | 够用 | 够用 | 最强(能写循环/条件/函数) |
| 普及度 | **主流** | 几乎没人用 | 框架包里常见 |
| 老师的做法 | ✅ 只用这个 | ❌ | ❌ 主动放弃("复杂化了") |

**结论:** 主力 XML; 遇到 py / yaml 的包**能读懂**即可

## 4. include - 调用别人的 launch 文件
```xml
<include file="$(find-pkg-share my_ros_bringup)/launch/other.launch.xml" />
<include file="$(find-pkg-share my_pkg)/launch/other.launch.py" />   <!-- 混合格式: 实测可行 -->
<include file="/tmp/test.launch.yaml" />                             <!-- 绝对路径也行 -->
```
- `file` 指向**任意格式**的 launch 文件(xml / py / yaml 通吃)
- 带参数调用:
```xml
<include file="...">
    <arg name="robot_name" value="robot_02" />
</include>
```

## 5. arg - launch 文件自己的参数
```xml
<launch>
    <arg name="robot_name" default="robot_01" />
    <arg name="pub_number" default="3" />
    <node pkg="my_py_pkg" exec="number_publisher" name="arg_test" namespace="$(var robot_name)">
        <param name="number_to_publish" value="$(var pub_number)" />
    </node>
</launch>
```
```zsh
# 用默认值 → 节点 /robot_01/arg_test, 参数 3
ros2 launch my_ros_bringup arg.launch.xml
# 命令行覆盖 → 节点 /robot_99/arg_test, 参数 11
ros2 launch my_ros_bringup arg.launch.xml robot_name:=robot_99 pub_number:=11
```
- `$(var 名字)` = 取 arg 值的语法
- 好处: 一个 launch 文件, 命令行切换配置(不用改文件)

## 6. launch 文件放哪 & 怎么装 (bringup 包惯例)
- launch 文件放**独立包**(如 my_ros_bringup), 不放功能包
- ament_cmake 包的关键两行:
```cmake
find_package(ament_cmake REQUIRED)
install(DIRECTORY launch config DESTINATION share/${PROJECT_NAME}/)   # ← 忘了这行 = file not found
```
```xml
<!-- package.xml: 声明要用到的其他包 -->
<exec_depend>my_py_pkg</exec_depend>
<exec_depend>my_cpp_pkg</exec_depend>
```
- 验证有没有装进去: `ls $(ros2 pkg prefix my_ros_bringup)/share/my_ros_bringup/launch/`

## 7. 实战 - number_app.launch.xml
```xml
<launch>
    <node pkg="my_py_pkg" exec="number_publisher" name="my_number_publisher" namespace="/robot_01">
        <remap from="number" to="my_number" />
        <param name="number_to_publish" value="6" />
    </node>

    <node pkg="my_cpp_pkg" exec="number_counter" name="my_number_counter" namespace="/robot_01">
        <remap from="number" to="my_number" />
    </node>
</launch>
```
**读法:**
- 两个节点同一个 namespace → 全部落在 `/robot_01/` 下
- 两边都 remap 相对名 `number → my_number` → 实际话题 `/robot_01/my_number`, 收发对上
- 只给 publisher 传参(counter 不需要)
- 验证: `ros2 node list` → `/robot_01/my_number_publisher`; `ros2 topic list` → `/robot_01/my_number`
