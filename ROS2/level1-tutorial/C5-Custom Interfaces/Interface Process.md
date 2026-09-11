### 接口文件如何变成代码里的类型?
   - 文件：`pkg/msg/HWStatus.msg`（CamelCase 文件名）
   - C++ 头文件：`pkg/msg/hw_status.hpp`（→ snake_case）
   - C++ 类型：`pkg::msg::HWStatus`
![[Interface Process.canvas]]
**生成命名映射规则**
- HWStatus.msg   → hw_status.hpp   / 类型 HWStatus
- SetLED.srv         → set_led.hpp         / 类型 SetLED（含 Request / Response 两个子类）
