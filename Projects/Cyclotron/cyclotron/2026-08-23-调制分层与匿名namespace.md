# 2026-08-23 调制分层与匿名 namespace

## 1. svpwm 的 write/modulate 分离（D11）

三层结构（foc::svpwm）：
- `compose`（**私有**，匿名 namespace）：变换链 clamp → wrap → inv_park → inv_clarke，产出 abc 交流分量
- `modulate`（公开）：调制原子操作——中心偏置（未来 + 零序注入）
- `write`（公开）：组合入口 = modulate(compose(...))，v1 调用方只碰它

分离动机（三）：
1. **v2 插入点**：死区补偿/过调制工作在 abc 域，必须插在 compose 之后、modulate 之前。write 一体实现则插入点无处安放（污染入口或调用方复制链路）
2. **可测性**：modulate 是纯加法，可单独断言不变式；compose 可单独断言交流分量和为 0
3. **职责**：变换（数学，transforms）与调制（策略，svpwm）独立演化

死区补偿是 D11 的直接动因，但分离本身是为了"插入点 + 可测性 + 职责"。

## 2. 匿名 namespace = 内部链接（文件私有）

`namespace { ... }`（匿名 namespace）：
- 内部链接（internal linkage）：符号只在本编译单元可见，**不导出到符号表**，链接器看不到
- 等价于 C 的 `static` 函数/变量，是 C++ 现代推荐写法
- 防跨文件同名符号冲突（如各 .cpp 各自有 constrainf）

与具名 namespace 的分工：**具名 = 公开 API 的组织；匿名 = 文件私有的标记**。
结论：匿名 namespace 内的函数（constrainf）不需要额外标注"私有"——namespace 本身就是私有标识。

## 3. SVPWM vs SPWM 语义检查（2026-08-23）

**发现**：legacy `foc.c` 的 `foc_svpwm_write` = `u_alpha + center` 纯中心偏置，**本质是 SPWM 不是 SVPWM**（无零序注入）；cyclotron 照搬后同样偏离。文档锚点 2 的"ua+ub+uc=3·center"正是 SPWM 特征不变式。

| | SPWM（纯正弦+中心偏置） | SVPWM（零序注入） |
|---|---|---|
| 调制波 | 三相正弦 | 正弦 + v₀ = -(max+min)/2 |
| 相电压峰值上限 | Vdc/2（m ≤ 1.0） | Vdc/√3（m ≤ 1.1547） |
| 线电压上限 | 0.866·Vdc | 1.000·Vdc |
| 三相和 | 恒 3·center | 3·center + 3·v₀ |

修复：modulate 加一行零序注入 v₀ = -(max+min)/2（min-max 中点，carrier-based SVPWM，与经典 7 段式扇区 SVPWM 严格等价，Holmes & Lipo 结论）。
- 注入后调制波峰值 = (max-min)/2 = √3·A/2，对称化（max' = -min'）
- 线电压不变（零序在三相等量加）
- 影响：voltage_limit 上限 Vdc/2 → Vdc/√3；锚点 2 不变式需从"和=3·center"改为"线电压 ≤ 2·limit"
