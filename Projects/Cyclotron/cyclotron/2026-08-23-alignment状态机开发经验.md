# 知识点：alignment 状态机开发经验（2026-08-23）

> 背景：cyclotron foc 库 alignment 组件从设计到落地的完整经验（D3 非阻塞状态机 / D16 纯算法 / D17 速度判据）。
> 关联：`foc/docs/ALIGNMENT.md`（设计专题）、`FOC_CORE_PSEUDOCODE.md` §6、Vault `2026-08-23-hal依赖结构`（D16 背景）。

## 一、判稳方案演进（核心收获）

**问题起点**：legacy 判据 `|Δ| < 0.1 rad @ 50ms 间隔`（= 2 rad/s）。非阻塞化后间隔变 1ms，阈值未缩放 → **语义漂移**：0.1 rad/ms = 100 rad/s 才算"动"，宽松 50 倍。

**方案谱系**：
| 方案 | 判据 | 评价 |
|---|---|---|
| A 保持现状 | 0.1 rad @ 1ms | 静止够用；匀速旋转误判 |
| B 阈值缩放 | 0.002 rad | 等效 legacy 但抗噪差 |
| C **速度判据** | `|Δ|/dt < 2.0 rad/s` | 与采样间隔解耦，语义正（采纳） |
| D 电流判据 | Id 带宽 | MCSDK 路线，v1 无电流采样不可行 |

**三参数三角色**（判稳的完整决策）：
- `settle_max_speed`（2.0 rad/s）→ 判稳**标准**（准确性，零位误差上界 = max_speed·dt）
- `settle_samples`（2）→ **确认次数**（抗噪去抖，连续 N 拍）
- `settle_timeout`（0.5 s）→ **等待上限**（容错 → FAULT）

## 二、别家对齐做法（"判稳"在业界的位置）

- **SimpleFOC**：不判稳（信任斜坡停稳），对齐角 -90° = 1.5π 与我们一致；无 fault 概念
- **QDrive**：固定延时 + SAMPLE_COUNT 次采样平均（用平均吸收残余误差）+ 多极对一致性校验（偏差>20% → EncoderError）；绝对零位（平均 pp 极对）
- **MCSDK**：电流域 Validation Tick（连续 N 个速度环 Id 在带宽内）——唯一认真判稳的
- 结论：**判稳 = FAULT 容错的传感器**；SimpleFOC/QDrive 敢不判稳是因为放弃失败检测

## 三、状态机代码组织（Q3 约定）

- **tick = 纯路由表**（switch + case），状态逻辑拆 `do_xxx`（do_ramp/do_settle）
- 转移集中在 do_xxx 内；参数按需传（do_ramp(raw,dt) 的 raw 用于记判稳基准）
- 无动作态（IDLE/LOCKED/FAULT）直接 break，不设空函数
- 公共类型（State/Fault/TickResult）namespace 级；类内只留方法 + private 状态

## 四、TickResult 快照语义（重要）

**tick 返回的 state_ 是"进入本 tick 时"的状态**：LOCKED/FAULT 转移发生在 do_xxx 内部、tick 构造结果帧之后 → **转移那一拍，返回帧里是旧状态**。

推论：
- 控制流判断用**实时 getter**（`is_locked()`/`fault()`），不用返回帧的 state
- 返回帧的 state 只作调试/日志（"本 tick 从哪态进入"）
- 测试断言同理：转移拍用 getter，或等下一拍再断 state

## 五、测试教训（黑盒测试 alignment）

1. **浮点累积边缘**：`10 × 0.001f` 累加可能 = 0.0099998 < 0.01f → "数精确 tick"断言必炸。解法：大余量（喂 20 tick 而非 10）或 do-while 推进到状态变化
2. **转移拍断言用 getter**（见四）：`check_state(r.state_, LOCKED)` 在转移拍必失败——改用 `a.is_locked()`
3. **波形验证用独立参考**：RAMP 波形与公开组件 `svpwm::write` 独立计算逐 tick 对比（黑盒 + 复用已验证组件）
4. **真 bug 会被测试抓到**：Bug1（计数 else 挂错层级 → 永不达标）+ Bug2（超时嵌稳定分支 → 不稳永不超时）各有一个用例锁定
5. **区分性断言设计**：跳变清零用例的关键是"跳变后只稳 1 次必须不 LOCKED"——正确实现（清零）与错误实现（不清零）在这个断言上分道

## 六、enum class 与 C 整数宇宙

- C 的 enum = int：裸常量、算术、按位或全合法（legacy 风格 `FAULT + SETTLE_TIMEOUT` 组合码）
- C++17 `enum class`：强制 `State::LOCKED` 限定 + 无算术 → "进入 FAULT 态 + 记原因码"必须拆两行
- 教训：`FAULT + SETTLE_TIMEOUT` 即使 C 里合法也只是数值相加，语义上从不是"进态+记码"——C 的"整数宇宙"让这种错误能编译通过

## 七、超时与判稳的优先级（同拍竞态）

do_settle 内两个转移（超时 → FAULT / 判稳达标 → LOCKED）**同拍可能同时满足**：
- 判稳先、超时后 → 成功变失败（FAULT 覆盖 LOCKED）
- 超时先、判稳后（无 return）→ 失败变成功（LOCKED 覆盖 FAULT + fault_ 残留矛盾）
- 正确：**超时优先 + return**（超时后不再判稳）——超时是"策略放弃"，不应被同拍的判稳成功翻转
- 触发条件：timeout ≤ samples·dt（病态配置）或"转子恰在 deadline 前连续稳定"（真实但低概率）

## 八、命名审阅沉淀

- `t_` → `elapsed_`（本状态已持续时长，RAMP/SETTLE 共用）
- `locked()` → `is_locked()`（ed 形容词不像谓词；is_ 前缀 = std 风格）
- `write`（svpwm）暗含碰硬件——挂起项，可改名 calc
- `sv` 变量名歧义（space vector?）→ `r`（result）
- 常量 `k3PI_2` = 3π/2 命名清晰（胜过裸 4.71238）
