## 1. 通用流程（Custom Interface）

### 1.1 前置知识：interface 分类
1. **.msg     -- topic**
2. **.srv       -- service**
3. **.action -- action**

### 1.2 包结构
**想自定义先建包:**
```zsh
cd workspace/src
ros2 pkg create --build-type ament_cmake my_interfaces
```

```zsh
my_interfaces
	msg/
		HWStatus.msg
	srv/
		SetLED.srv

	CMakeLists.txt
	package.xml
```

### 1.3 package.xml & CMakeLists.txt
```xml
<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

```cmake
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
	"msg/HWStatus.msg"
	"srv/SetLED.srv"
)
```

### 1.4 接口文件定义
#### 1.4.1 .msg（topic 用）
```
int64 temperature
bool are_motors_ready
string debug_message
```

#### 1.4.2 .srv（service 用，--- 分 Request/Response）
```
int64 led_number
int64 state
---
bool success
```

### 1.5 构建与接口验证
**colcon build**
```zsh
colcon build --packages-select my_interfaces
```
**source**
```zsh
source workspace/install/setup.zsh
```
**interface verification**
```zsh
ros2 interface show my_interfaces/msg/HWStatus
ros2 interface list | grep my_interfaces
```

### 1.6 消费（Consumption）

#### 1.6.1 C++ 侧三件套

1. **package.xml**：
```xml
<depend>my_interfaces</depend>
```
2. **CMakeLists.txt**：
```cmake
find_package(my_interfaces REQUIRED)
ament_target_dependencies(目标 rclcpp my_interfaces)
```
3. **代码**：
```cpp
#include "my_interfaces/msg/hw_status.hpp"
using HWStatus = my_interfaces::msg::HWStatus;
```

#### 1.6.2 Python 侧

- **package.xml**：
```xml
<depend>my_interfaces</depend>
```
- **代码**：
```python
from my_interfaces.msg import HWStatus
```

#### 1.6.3 CLion（桌面启动不继承终端环境！）

Settings → CMake → Environment 填：

```
CMAKE_PREFIX_PATH=/opt/ros/jazzy;<工作区>/install/my_interfaces
AMENT_PREFIX_PATH=/opt/ros/jazzy;<工作区>/install/my_interfaces
```

改完 Reset Cache and Reload Project


## 2. 示例

### 2.1 custom msg：hw_status_publisher

```cpp
// hw_status_publisher.cpp 关键行
#include "my_interfaces/msg/hw_status.hpp"
using HWStatus = my_interfaces::msg::HWStatus;

// 创建发布者
hw_status_publisher_ = this->create_publisher<HWStatus>("hardware_status", 10);

// 发布消息
auto msg = HWStatus();
msg.temperature = 45;
msg.are_motors_ready = true;
msg.debug_message = "All systems go";
hw_status_publisher_->publish(msg);
```

### 2.2 custom srv：led_panel & battery

```cpp
// led_panel.cpp
class LEDPanel : public rclcpp::Node {
public:
    LEDPanel(): Node("led_panel") {
        led_states_.led_states = {0, 0, 0};

        srv_set_led_ = this->create_service<SetLED>("set_led",
            std::bind(&LEDPanel::set_led, this, std::placeholders::_1, std::placeholders::_2));

        led_state_publisher_ = this->create_publisher<LEDStateArray>("led_states", 10);
        timer_ = this->create_wall_timer(std::chrono::seconds(1), std::bind(&LEDPanel::publish_led_state, this));
    }
private:
    void set_led(const SetLED::Request::SharedPtr &request,
                 const SetLED::Response::SharedPtr &response) {

        int64_t led_number = request->led_number;
        int64_t state = request->state;

        if (led_number > 3 || led_number < 1) {
            response->success = false;
            RCLCPP_WARN(this->get_logger(), "Invalid led number requested. Please try again.");
            return;
        }

        if (state != 0 && state != 1) {
            response->success = false;
            RCLCPP_WARN(this->get_logger(), "Invalid state requested. Please try again.");
            return;
        }

        this->led_states_.led_states[led_number - 1] = state;
        response->success = true;
        this->publish_led_state();
    }

    void publish_led_state() {
        auto msg = LEDStateArray();
        msg.led_states = this->led_states_.led_states;
        this->led_state_publisher_->publish(msg);
    }

    rclcpp::Publisher<LEDStateArray>::SharedPtr led_state_publisher_;
    rclcpp::TimerBase::SharedPtr timer_;
    rclcpp::Service<SetLED>::SharedPtr srv_set_led_;
    LEDStateArray led_states_;
};
```

```cpp
// battery
class Battery : public rclcpp::Node {
public:
    Battery(): Node("battery"), battery_state_("full"), tick_count_(0) {
        set_led_client_ = this->create_client<SetLED>("set_led");
        battery_timer_ = this->create_wall_timer(std::chrono::milliseconds(500),
            std::bind(&Battery::check_battery_state, this));
    }
private:
    void call_set_led(int64_t led_number, int64_t state) {
        1. wait for service

        2. build request
        // call and wait for response
        set_led_client_->async_send_request(request, [this](rclcpp::Client<SetLED>::SharedFuture future)
        {
            auto response = future.get();
            RCLCPP_INFO(this->get_logger(), "Set LED success: %s", response->success ? "true" : "false");
        });
    }

    void check_battery_state() {
        tick_count_++;
        if (battery_state_ == "full" && tick_count_ >= 4) {
            battery_state_ = "empty";
            tick_count_ = 0;
            RCLCPP_INFO(this->get_logger(), "Battery is empty! Charging battery...");
            call_set_led(3, 0);
        }
        else if (battery_state_ == "empty" && tick_count_ >= 6) {
            battery_state_ = "full";
            tick_count_ = 0;
            RCLCPP_INFO(this->get_logger(), "Battery is full again!");
            call_set_led(3, 1);
        }
    }

    std::string battery_state_;
    rclcpp::Client<SetLED>::SharedPtr set_led_client_;
    rclcpp::TimerBase::SharedPtr battery_timer_;
    size_t tick_count_;
};
```

![[Logic Map - LED_Panel & Battery.canvas]]

## 3. 运行验证

**前提：服务端必须先跑**（否则 set_led 服务不存在，调用会失败）：

```zsh
# 终端 A：服务端（保持运行）
ros2 run my_cpp_pkg led_panel

# 终端 B：客户端（可选）
ros2 run my_cpp_pkg battery

# 终端 C：观察数据流 + 调用服务
ros2 topic echo /led_states
ros2 service call /set_led my_interfaces/srv/SetLED "{led_number: 1, state: 1}"
```
