# gimbal RTOS baseline

这份笔记对应当前已经烧录验证的半完全体工程。

入口：[[gimbal_three_lines.canvas]]

- [[FOC_current.canvas]]：传感器值如何进入控制环并最终写 PWM。
- [[Tele_current.canvas]]：命令和显示是两条方向相反的数据链。
- [[Safety_current.canvas]]：故障如何被发现、锁存、显示和清除。
- [[FreeRTOS_current.canvas]]：执行上下文与数据所有权，不是第四套业务逻辑。

阅读原则：先沿箭头追变量，再打开函数卡片。红色/缺口卡片表示目前工程还没有闭环的部分。

冻结基线：`0ff5c7e`。

