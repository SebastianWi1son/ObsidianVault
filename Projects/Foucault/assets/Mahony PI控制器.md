```cpp
math::Vec3f omega = gyro;
if (is_meas_) {
    // 修正折进角速度：ω_corrected = ω + Kp·e + Ki·∫e
    omega.x_ += cfg_.kp_ * e_last_.x_ + cfg_.ki_ * e_int_.x_;
    omega.y_ += cfg_.kp_ * e_last_.y_ + cfg_.ki_ * e_int_.y_;
    omega.z_ += cfg_.kp_ * e_last_.z_ + cfg_.ki_ * e_int_.z_;
}
```