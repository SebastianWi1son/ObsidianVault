**Q1** source 和直接执行 setup.zsh 的区别？
→ 直接执行开子进程，变量传不回；source 在当前 shell 内执行，立即生效。

**Q2** 本机手动 source 用什么后缀？为什么？
→ `.zsh`。登录 shell 是 zsh；zsh 不读 .bashrc。

**Q3** 报 `ros2: command not found` 缺哪层？报找不到 my_py_pkg 缺哪层？
→ 前者缺第 1 层（/opt/ros/jazzy）；后者缺第 2 层（工作区 install/setup.zsh）。

**Q4** 为什么换项目要开新终端？
→ 旧终端环境残留（AMENT_PREFIX_PATH 累积），同名包会被遮蔽。

**Q5** 为什么不能直接 `./install/.../py_node`？
→ 它是包装脚本，需靠 egg-info 元数据定位入口；不 source 时 PYTHONPATH 没有 site-packages。

**Q6** `info("Hello", str(x))` 为什么报错？C++ 版为什么可以？
→ rclpy 的 info 只收一个字符串参数；C++ RCLCPP_INFO 支持 printf 风格多参数。

**Q7** 改完 .py 忘了 colcon build，ros2 run 会怎样？
→ 跑的仍是 install 里旧副本，改动不生效。

**Q8** CLion 找不到 rclcpp/rclcpp.hpp 的根因？
→ CLion 进程缺 CMAKE_PREFIX_PATH=/opt/ros/jazzy（从桌面启动不继承终端 source 的环境）。

**Q9** ament_cmake 找到了还报 No module named 'ament_package'？
→ 缺 PYTHONPATH=/opt/ros/jazzy/lib/python3.12/site-packages。

**Q10** CLion 里 build 成功 = ros2 run 能跑吗？
→ 不一定。CLion 用 Ninja 编进 cmake-build-debug（仅供 IDE 内运行）；ros2 run 用的是 colcon 的 install 产物，需另行 colcon build。

**Q11** source 完回车了没跟 nohup，要重开终端吗？
→ 不用。环境驻留当前 shell，之后任意时刻启动 CLion 都继承。

**Q12** 为什么 colcon build 后"新终端"还要 source install？
→ 新终端是干净 shell；而包装脚本/ros2 run 都依赖 PYTHONPATH 与 AMENT_PREFIX_PATH 注入。