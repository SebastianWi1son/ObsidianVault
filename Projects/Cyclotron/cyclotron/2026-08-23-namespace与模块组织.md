# 对话总结：namespace 用法与模块组织（2026-08-23）

> 纯事实总结。背景：cyclotron foc 库（C++17），`foc::` 分层命名空间（D1），svpwm::write 实现讲解
> 关联：`foc/docs/FOC_MATH_SPEC.md`（§1/§2）、`foc/docs/FOC_CORE_PSEUDOCODE.md`（§3/§4）

## 一、namespace 是什么

命名空间 = 姓氏/文件夹。`foc::svpwm::write` = "foc 公司 → svpwm 部门 → write 函数"。不同部门可以有同名函数不冲突。

## 二、四个使用场景

**① 头文件声明**（只写签名）：

```cpp
#pragma once

namespace foc::svpwm {          // C++17 简写 = namespace foc { namespace svpwm {
struct Result { float ua, ub, uc; };
Result write(float uq, float ud, float angle_elec,
             float voltage_limit, float voltage_supply);
Result modulate(float ua_ac, float ub_ac, float uc_ac, float voltage_supply);
}  // namespace foc::svpwm
```

**② 实现文件定义**（名字不带前缀——已经站在门牌号里）：

```cpp
#include "foc/svpwm.hpp"

namespace foc::svpwm {
Result write(...) { ... }        // 直接写 write，不用 foc::svpwm::write
}
```

**③ 调用方**：`foc::svpwm::write(...)`；.cpp 里可 `using foc::svpwm::write;`。**头文件里禁止 using**（会把名字泄露给所有包含者）。

**④ 文件私有常量**（内部链接，不导出符号）：

```cpp
namespace {                       // 匿名 namespace
constexpr float TWO_PI = 6.28318530718f;
}
```

## 三、本项目的实践

- 分层：`foc::transforms` / `foc::svpwm` / `foc::angle_tracker` / `foc::alignment` / `foc::core` / `foc::algo`（D1）
- 兄弟 namespace 互相引用必须带前缀：svpwm.cpp 里调 `transforms::inv_clarke(...)`
- 声明/定义分离：transforms 是 inline 头文件实现（零状态纯函数，无 .cpp）；svpwm 声明在 .hpp、实现在 .cpp（D10）
- 头文件用 `#pragma once`（不是 include guard 宏）；namespace 与 include guard 是两回事

## 四、要点备忘

- `namespace foc::svpwm {}` 是 `namespace foc { namespace svpwm {} }` 的 C++17 简写
- 匿名 namespace = 本编译单元私有（等价 static 的现代写法），适合实现文件的内部常量/辅助函数
- using namespace 只在 .cpp 用；using 声明按函数/类型粒度，比整空间引入更可控
