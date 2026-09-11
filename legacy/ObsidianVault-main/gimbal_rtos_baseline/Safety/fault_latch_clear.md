# fault latch / safety_clear

```text
fault source
 -> app_safety_emergency_stop(error)
 -> first fault latched
 -> PWM = 0 + control stopped
 -> telemetry keeps running
```

`safety_clear` 不是重新启动电机。

清除前先检查 yaw/pitch 编码器已经进入 `STABLE`。通过后只做：

- 清 `error_report` 与 safety fault latch。
- 记录 clear result。
- 保持 motor disabled，等待新的 `MotorRun`。

这解释了故障恢复流程为何必须是：传感器恢复稳定 -> clear 被接受 -> 再发 MotorRun。

当前优点：错误可观测，清除失败也有原因，不会无声恢复。

