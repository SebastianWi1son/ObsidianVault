# sensor -> angle

`SensorTask` 负责真实 SPI/I2C 访问，TIM2 ISR 只读缓存。

```text
cached raw angle
 -> d_raw
 -> 跨 0/2pi 圈数修正
 -> abs_angle
 -> delta / dt
 -> velocity LPF
```

这个拆分的关键不是“多了一个任务”，而是禁止 1 kHz FOC ISR 被阻塞总线拖住。

编码器异常不在这里决定系统是否继续运行；本卡只生产角度与健康状态，最终策略交给 Safety。

