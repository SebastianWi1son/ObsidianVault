# 知识点：hal 依赖结构 — legacy 与新代码对比（2026-08-23）

> 背景：cyclotron foc 库 D5/D16 决策后的分层梳理。legacy（STM32H743 双轴 C 裸机）vs 新代码（纯算法层 + 单点 IO）。
> 关联：`2026-08-23-回调设计-函数指针与ctx.md`（姊妹篇：那篇讲 **ctx 通道**，这篇讲 **依赖结构 + 跨平台考量**）；`foc/docs/FOC_CORE_PSEUDOCODE.md` D5/D16

## 一、legacy 形态（事实）

```c
// foc.h：回调挂在 motor->hw 上，全部无参（无 ctx）
float (*get_angle_cb)(void);                 // 物理角 [0, 2π)
void  (*set_pwm_cb)(float ua, float ub, float uc);  // 相电压
```

- 调用点**散落**在 foc.c 各处（`if (motor->hw.set_pwm_cb) motor->hw.set_pwm_cb(...)`，带可空检查）
- 无 ctx → 双轴云台每轴手写适配器样板（已知痛点，D5）
- 对齐段直接调 set_pwm_cb 发波（foc.c 对齐流程内）
- 裸机产物，不可 PC 测；回调实现在平台层，绑定 STM32

## 二、新代码形态（D5 + D16 后）

```cpp
// hal.hpp：全部带 ctx（自家接口，非厂商库）
using SetPwmFn = void (*)(void* ctx, float ua, float ub, float uc);
// 依赖结构：算法层（transforms/svpwm/angle_tracking/alignment）零 IO 依赖
//            hal 仅被 foc_core 依赖 —— 单一 IO 出口（对齐/闭环统一发波路径）
```

- alignment 纯算法化（D16）：tick 输出 `TickResult{state, u, v, w}` 数据，不发波
- foc_core 是唯一 IO 接触点：`hw.set_pwm(ctx, ar.u, ar.v, ar.w)`
- PC 可测：测试注入 fake 回调（ctx 指向假对象）
- 跨平台：hal 实现文件由构建系统编译期选择

## 三、对比表

| 维度 | legacy | 新代码 | 变化本质 |
|---|---|---|---|
| 依赖位置 | 散落（foc.c 各发波点直接调） | 集中（foc_core 单点出口） | 散 → 聚 |
| 上下文 | 无 ctx（全局/每轴样板） | ctx 穿透 | 无通道 → 通道（手搓闭包） |
| 对齐发波 | 对齐段直接调 set_pwm_cb | 纯算法输出数据，统一出口 | 执行者 → 计算器 |
| 可测性 | 裸机不可测 | fake 注入 PC 可测 | 硬件 → 可替换 |
| 跨平台 | 绑定 STM32 | 编译期分发（实现文件切换） | 运行时 → 编译期 |
| 回调粒度 | 相电压浮点 + 物理角（同新代码） | 同左（签名差异挂起项待校准） | 不变 |

## 四、知识点（原则提炼）

1. **回调形态的动机决定评价**：同一个函数指针——
   - 为**跨平台**做回调 = 过度设计（legacy 建议，正确：跨平台是编译期已知的，运行时多态白付成本）
   - 为**测试注入**做回调 = 正当（PC 可测是核心工程约束）
   - 我们的 hal 回调动机是后者；跨平台由实现文件承担（编译期分发：opaque 类型 + 平台 .c/.cpp + 构建系统选择）

2. **跨平台差异分层**（STM32 TIM/CCR vs TI C2000 ePWM 查证）：
   - 配置/初始化层差异大：寄存器体系、位宽、时钟、极性、死区、影子寄存器装载时机（STM32 OCxPE vs TI LOADAMODE）、TI 独有 PWM 中心硬件触发 ADC
   - 运行时动作层差异小：FOC 每周期只做"更新一个比较值"——通用薄接口
   - 推论：**只抽象运行时动作（set duty/enable），不抽象初始化（时钟树/引脚复用）**——初始化交给平台实现

3. **依赖位置 = 可测性**：IO 接触点越少越散，测试注入越难；单点出口 + 纯算法层让"测算法"与"测 IO"彻底分离（算法层纯数学可全测，IO 层薄到只需 fake）

4. **执行者组件纯算法化**（D16 模式）：任何"自带动作"的组件（对齐、未来的开环测试模式）→ 输出数据由编排层统一送 IO；编排层只多一行机械转发，换来算法层零 IO + 统一出口

5. **命名**：HAL = Hardware Abstraction Layer（硬件抽象层），业界通用术语（Windows HAL/STM32 HAL/Linux 同源），我们的 hal.hpp 是自家抽象接口，与厂商库撞名是术语常态，不因歧义改名

## 五、一句话记忆

**IO 只在一处（编排层），算法全是计算器；回调为测试而生，跨平台交给编译期。**
