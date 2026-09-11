# RTOS diagnostics

`diag_page:4` 当前测得 stack headroom：

```text
Control 688
Sensor  664
Comm    484
Tele    700
late counts all 0
```

这些值是“最低剩余栈空间”，不是 heap。稳定且有余量说明当前任务栈配置可接受。

诊断意义：

- stack headroom 判断单个任务是否接近溢出。
- heap current/minimum 判断动态内存历史低点。
- late count 判断周期任务是否错过 deadline。
- heartbeat age 判断模块是否还在推进。

FreeRTOS 在本工程里的价值主要是明确执行上下文和所有权，而不是把每个函数都包装成任务。

