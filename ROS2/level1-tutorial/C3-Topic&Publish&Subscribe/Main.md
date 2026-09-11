## 1. Topic
- **Unidirectional** data stream
- **Anonymous**
- Message **Type**
> data信息的` tag `
> 由各节点内部DDS信使recognize

## 2. Publisher
```cpp
class PublishNode : public rclcpp::Node {
public:
	PublishNode(): Node("publisher"), robot_name_("R2D2") {
		// member publisher
		publisher_ = this->create_publisher<xxString>("topic_news", 10);
		// 定时器挂事件, bind 'publish_news'
		timer_ = this->create_wall_timer(std::chrono::milliseconds(500),
						std::bind(&PublishNode::publish_news, this));
	}
private:
	void publish_news() {     // tasks
		auto msg = String();
		msg.data = std::string("Hi, this is id:xx from publisher");
		publisher_->publish(msg);
	}
	
	std::string robot_name_;                           // robot id
	rclcpp::Publisher<String>::SharedPtr publisher_;   // publisher
	rclcpp::TimerBase::SharedPtr timer_;               // timer
};
// ps: 代码存在简化，不可直接运行
```

## 3. Subscriber
```cpp
class Subscribe : public rclcpp::Node {
public:
	Subscribe(): Node("subscribe") {
		// 初始化subscriber订阅topic name, 同时挂回传callback_news到接收事件
		subscriber_ = this->create_subscription<String>("topic_news", 10,
			std::bind(&Subscribe::callback_news, this, std::placeholders::_1));
	}
private:
	void callback_news(const String::SharedPtr msg) {    // call back news
		RCLCPP_INFO(this->get_logger(), "%s", msg->data.c_str());
	}
	
	rclcpp::Subscription<String>::SharedPtr subscriber_; // subscriber
};
// ps: 代码存在简化，不可直接运行
```

## 4. Add Dependencies
1. `package.xml`
```xml
<depend>example_interfaces</depend>
```
2. `CMakeLists.txt`
```CMake
find_package(example_interfaces REQUIRED)
```
3. `xxx.cpp`
```cpp
#include "example_interfaces/msg/string.hpp"
```

## 5. Universal Template
```cpp
class MyNode : public rclcpp::Node {
public:
	constructor() {
		注册事件;
		事件绑定callback_tasks;
	}
private:
	callback_tasks定义/声明;
	需要维护的私有成员;
};
```

