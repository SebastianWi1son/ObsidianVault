# safety coverage

## 已覆盖

- yaw / pitch 编码器通信、恢复状态和稳定门槛。
- 过速、VBUS 欠压/过压、pitch 目标限位、yaw 目标突变。
- current/torque 模式下的过流检查；注意 pitch `max_current = 0` 时等价于未启用有效过流保护。
- 故障锁存、PWM 清零、禁止故障状态 MotorRun、透明诊断。

## 当前不完整

- ADC current sense 只有数值路径，没有 stale、rail stuck、断线、零漂超界和双通道一致性诊断。
- 没有温度、驱动器 fault pin、功率级短路/欠压锁定输入。
- 电压/电流模式的热保护策略尚未形成 I²t 模型。
- 通信失联策略未定义。

## 期望

把每个 fault source 都写成：`检测条件 -> debounce -> severity -> latch policy -> clear condition -> telemetry evidence`。

比赛固件可以先追求“故障明确停机且容易定位”；产品级还需要冗余、降级运行与系统化验证。

