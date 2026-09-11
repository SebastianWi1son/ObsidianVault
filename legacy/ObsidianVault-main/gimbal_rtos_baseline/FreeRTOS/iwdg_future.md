# IWDG future

目前已经有 reset cause 和 app/FOC/ADC/encoder/UART heartbeat，但还不能直接据此喂狗。

期望链路：

```text
subsystem heartbeat
 -> health coordinator
 -> required set all healthy
 -> feed IWDG
```

health coordinator 需要定义：

- 哪些 heartbeat 是启动后必须存在的。
- 各模块最大允许 age。
- 电机停机时 FOC heartbeat 是否仍必须推进。
- UART 无主机连接时是否算故障。
- 故障锁存后是继续喂狗保留诊断，还是故意复位。

在这些策略定稿前开启 IWDG，会把可诊断的软件故障变成反复复位。

