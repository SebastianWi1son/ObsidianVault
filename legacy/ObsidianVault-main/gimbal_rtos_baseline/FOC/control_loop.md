# foc_angle_control_tick

```text
target_angle
 -> trajectory planner
 -> angle PID
 -> target_vel + trajectory feedforward
 -> velocity PID
```

之后按模式分支：

- `VOLTAGE`：velocity PID 输出作为 `Uq`。
- `CURRENT`：输出作为 `Iq_ref`，再经过 Clarke/Park、Iq/Id PID。
- `TORQUE`：旁路位置/速度环，`torque_ref -> Iq PID`。
- `OPEN_LOOP`：不进入本函数，由 motor_service 单独驱动。

最终：`Uq/Ud -> InvPark -> SVPWM -> ua/ub/uc -> timer compare`。

目前合理：TIM2 ISR 是唯一运行态写 PWM 的控制上下文。

未来：电流采样可信度、采样时刻与 PWM 同步仍需单独验证，不能只看数值“在更新”。

