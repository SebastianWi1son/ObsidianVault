## 1.  Basic Usage
```cpp
// Declaration
this->declare_parameter("param_01", 1.0);
// Call
this->get_paramter("param_01").as_double();
```

```zsh
ros2 param list     
# print param_01 value 
ros2 param get /node param_01     
# run node with param_01 2.0
ros2 run my_pkg my_node --ros-args -p param_01:=2.0
```
