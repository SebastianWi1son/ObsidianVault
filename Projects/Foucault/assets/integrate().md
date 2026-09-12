q_dot =  1/2 * q ⊗ (0, w1, w2, w3)
q += q_dot * dt

```cpp
math::Quat::integrate(const Vec3<T>& omega, T dt) {
	T half = T(0.5) * dt;
    T wx = omega.x_, wy = omega.y_, wz = omega.z_;
    // q_dot =  1/2 * q ⊗ (0, w1, w2, w3)
    // q += q_dot * dt
    T q0 = q0_ + (-q1_ * wx - q2_ * wy - q3_ * wz) * half;
    T q1 = q1_ + ( q0_ * wx + q2_ * wz - q3_ * wy) * half;
    T q2 = q2_ + ( q0_ * wy - q1_ * wz + q3_ * wx) * half;
    T q3 = q3_ + ( q0_ * wz + q1_ * wy - q2_ * wx) * half;

    q0_ = q0; q1_ = q1; q2_ = q2; q3_ = q3;
    return normalize();
}
```