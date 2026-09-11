## 1. 前置知识 - Parameter 是什么
- 节点**运行时可改**的配置: 不用改代码、不用重编译
- 流程: 节点**声明**(declare) → 外部**注入**值(CLI / launch / yaml)
- 声明时定类型 → **静态类型**: 之后 set 只能同类型
- **用途:** 把"会变的量"从代码里抽出来(发布频率、目标值、开关)

## 2. Basic Usage
### 2.1 代码侧
```cpp
// Declaration (声明 + 默认值; 类型由默认值推导)
this->declare_parameter("param_01", 1.0);                 // double
this->declare_parameter("number_to_publish", 1);          // int64
// Read (取值, 按类型取)
this->get_parameter("param_01").as_double();
this->get_parameter("number_to_publish").as_int();
```
```python
self.declare_parameter("number_to_publish", 2)
self.get_parameter("number_to_publish").value
```

### 2.2 CLI
```zsh
ros2 param list                            # 列出运行中节点的全部参数
ros2 param get /number_publisher number_to_publish
ros2 param set /number_publisher number_to_publish 42   # 运行时改
```

### 2.3 启动时注入
```zsh
ros2 run my_pkg my_node --ros-args -p param_01:=2.0
```

## 3. 类型 & C++ 的坑
| 类型 | C++ 声明 | C++ 读取 | Python 读取 |
| --- | --- | --- | --- |
| int64 | `declare_parameter("x", 1)` | `.as_int()` | `.value` |
| double | `declare_parameter("x", 1.0)` | `.as_double()` | `.value` |
| bool | `declare_parameter("x", true)` | `.as_bool()` | `.value` |
| string | `declare_parameter("x", "abc")` | `.as_string()` | `.value` |

**C++ 铁律:** `declare_parameter` 是模板函数, **必须给默认值**让编译器推导类型
- `declare_parameter("x")` → **编译失败**: `couldn't deduce template parameter`
- `declare_parameter<int64_t>("x")` → 能编译, 但**没有初始值** → 运行时报 `UninitializedStaticallyTypedParameterException`

> Python 是动态类型, 无此问题 —— `self.declare_parameter("x", 1)` 一行搞定

## 4. 优先级 (谁覆盖谁)
```text
代码默认值  <  launch / yaml 文件  <  命令行 -p
```
- 高优先级覆盖低优先级
- 命令行永远最高 → 临时调试不用改任何文件

## 5. launch 里传参
```xml
<node pkg="my_py_pkg" exec="number_publisher" name="my_number_publisher">
    <param name="number_to_publish" value="6" />
</node>
```
- 是 `<param name= value=>`, **不是** `<remap>` (remap 只有 from/to)

## 6. 参数文件 (dump / load)
```zsh
ros2 param dump /number_publisher > params.yaml   # 导出当前全部参数
ros2 param load /number_publisher params.yaml     # 重新载入
```

## 7. 运行时改了参数, 代码怎么知道? (参数回调)
- 默认: `param set` 只改存储值, **代码不会自动感知**
- 想让"改了就生效": 注册回调

```cpp
this->add_on_set_parameters_callback(
    std::bind(&MyNode::on_param_change, this, std::placeholders::_1));

rcl_interfaces::msg::SetParametersResult on_param_change(
    const std::vector<rclcpp::Parameter> & params) {
    for (const auto & p : params) {
        if (p.get_name() == "number_to_publish") { /* 立即生效 */ }
    }
    rcl_interfaces::msg::SetParametersResult result;
    result.successful = true;        // 设 false 可以"拒绝这次修改"
    return result;
}
```
```python
from rcl_interfaces.msg import SetParametersResult

self.add_on_set_parameters_callback(self.on_param_change)

def on_param_change(self, params):
    for p in params:
        if p.name == "number_to_publish": ...
    return SetParametersResult(successful=True)
```

## 8. 实战 - number_publisher 参数化
```cpp
// 频率参数 → 控制 timer 周期
this->declare_parameter("number_to_publish", 1);
this->declare_parameter("frequency_to_publish", 2.0);
double f = this->get_parameter("frequency_to_publish").as_double();

timer_ = this->create_wall_timer(
    std::chrono::milliseconds((int)(1000.0 / f)),
    std::bind(&NumberPublish::callback_publish_number, this));
```
```zsh
ros2 run my_cpp_pkg number_publisher --ros-args -p frequency_to_publish:=5.0
```
