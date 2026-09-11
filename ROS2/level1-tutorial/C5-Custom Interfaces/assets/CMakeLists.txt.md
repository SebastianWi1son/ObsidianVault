```cmake
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
        "msg/HardwareStatus.msg"
        "msg/LEDStateArray.msg"
        "srv/ComputeRectangleArea.srv"
        "srv/SetLED.srv"
)
```
