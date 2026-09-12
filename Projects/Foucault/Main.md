> 输入量测流, 内部维护"上一时刻估计" + “算法特有状态”, 输出当前最优估计

## 1. concept
### 1.1 `旋转`和`姿态`在四元数等价
- 一个**旋转动作**可由**单位四元数**表示
- 一个**姿态状态**可以看作是**world系转至body系的旋转**

### 1.2 Hamilton Product
- 
- **叠加旋转获得姿态**是一种**手段**

## 1.3 Mahony与EKF差异
1. Mahony修正correct在角速度omega, 与外推predict操作耦合
```cpp
estimator<Mahony>::predict 耦合外推+修正
```
2. EKF修正correct与predict外推解耦

### 1.4 `cos(θ/2), û·sin(θ/2)` 的第二项怎么变成 `½ω·dt`？
1. 旋转 θ 弧度、绕轴 û 的四元数 :
```
( cos(θ/2),  û·sin(θ/2) )
```
2. 这一帧转过的角度:
```
θ = |ω|·dt
```
3. ω 可写成"长度 × 方向"：ω = |ω|·û
```
第 3 步：把第二项里的 θ 换成 ω·dt
        û·sin(θ/2)
          ≈ û·(θ/2)        ← θ 小时 sin(θ/2) ≈ θ/2（弧度制）
          = (θ·û)/2
          = (ω·dt)/2       ← 用第 2 步：θ·û = ω·dt
          = ½·ω·dt         ✓
```
ω·dt 这个向量本身就是"轴 û + 角 θ"

## 2. 代码实现与理论的差异
### 2.1 外推
1. **理论:** 通过在旧姿态值叠加此刻q_dot, 获得新值q_k+1
```
q_k + q_dot ---> q_k+1
```

2. **代码实现:** 
```cpp
math::Quat::intergrate(const Vec3<T>& omega, T dt) {
	T half = T(0.5) * dt;
    T wx = omega.x_, wy = omega.y_, wz = omega.z_;
    // q_dot =  1/2 * q ⊗ (0, w1, w2, w3)
    // q += q_dot
    T q0 = q0_ + (-q1_ * wx - q2_ * wy - q3_ * wz) * half;
    T q1 = q1_ + ( q0_ * wx + q2_ * wz - q3_ * wy) * half;
    T q2 = q2_ + ( q0_ * wy - q1_ * wz + q3_ * wx) * half;
    T q3 = q3_ + ( q0_ * wz + q1_ * wy - q2_ * wx) * half;

    q0_ = q0; q1_ = q1; q2_ = q2; q3_ = q3;
    return normalize();
}
```
- 没有q_dot
- 没有两个时刻的q_k, q_k+1, 只有一个q自我更新

### 2.2 为何要归一化
```

```
> **一阶近似**使 结果四元数**模长大于1**
> 而**只有单位四元数才可以表示旋转/姿态**

## 3. replay
### 3.1 predict与observe顺序问题
真实项目中推荐使用: predict在前，observe在后
实际回放replay样本中: observe在前，predict在后


