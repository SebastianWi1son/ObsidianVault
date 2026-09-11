## 1. AddTwoInts
### 1.1 add_two_ints_server
```cpp
using AddTwoInts = example_interfaces::srv::AddTwoInts;

class AddTwoIntsServer : public rclcpp::Node {
public:
	AddTwoIntsServer(): Node("add_two_ints_server") {
	server_ = this->create_service<AddTwoInts>(
		"add_two_ints",    // service name
		std::bind(&AddTwoIntsServer::on_request, this, _1, _2))
	}
private:
	// "ROS框架"自动传入request, response，加工response
	void on_request(const AddTwoInts::Request::SharedPtr &request,
		const AddTwoInts::Response::SharedPtr &response) {
		
		response->sum = request->a + request->b;
	}
	
	rclcpp::Service<AddTwoInts>::SharedPtr server_;
};
// 代码存在简化不可直接运行
```

### 1.2 add_two_ints_client
```cpp
class AddTwoIntsClient : public rclcpp::Node {
public:
	AddTwoIntsClient(): Node("add_two_ints") {
		// lambda
		// 临时找来两个thread干活，干完就走
		threads_.push_back(std::thread([this]() { do_request(21, 30); }));
		threads_.push_back(std::thread([this]() { do_request(23, 40); }));
	}
	
private:
	void do_request(int a, int b) {
		// wait for service
		while (!client_->wait_for_service(std::chrono::seconds(1))) {
			RCLCPP_WARN(this->get_logger(), "Waiting for the service...");
		}
		// build request
		auto request = std::make_shared<AddTwoInts::Request>();
		request->a = a;
		request->b = b;
		// do future get response
		auto future = client_->async_send_request(request); 
		auto response = future.get();
		RCLCPP_INFO(this->get_logger(), "%ld + %ld = %ld",
						request->a, request->b, response->sum);
	}
	// private maintained member
	rclcpp::Client<AddTwoInts>::SharedPtr client_;
	std::vector<std::thread> threads_;
};
```
### 1.3 why include AddTwoInts?
```zsh
ros2 interface show example_interfaces/srv/AddTwoInts
#output
int64 a
int64 b
___
int64 sum
```
- **service的格式由接口类型定**
- AddTwoInts定义了: request{a, b} -> response{sum}
- server / client 必须**序列化格式一致(合同)**

> AddTwoInts来自官方示例接口包
> 实际项目自建接口包

## 2. Service
### 2.1 Contrast
| what's new | pub/sub         | service | Code                            |
| ---------- | --------------- | ------- | ------------------------------- |
| answer     | fire-and-forget | 必须等结果   | Request/Response                |
| waiting    | fire-and-forget | 必须等结果   | wait_for_service + future.get() |
| 指名道姓       | 匿名              | 点对点     | only one server                 |

### 2.2 深析
> **service本质是“两个节点+两个topic"之上包装一层服务的应用层**
> **service = 远程函数调用**
> **on_request = 函数体**
> **request = 参数**
> **response = 返回值**

![[ServiceLowLevelMap.canvas]]

### 2.3 Template
#### 2.3.1 Server
```cpp
class Server : public rclcpp::Node {
public:
	Server(): Node("node_name") {
	server_ = this->create_service<AddTwoInts>(
		"service_name",    // service name
		std::bind(&Server::on_request, this, _1, _2));
	}
private:
	// "ROS框架"自动传入request, response，加工response
	void on_request(const type::Request::SharedPtr  &request,
				    const type::Response::SharedPtr &response) {
		
		加工response
	}
	
	rclcpp::Service<type>::SharedPtr server_;
};
```

 **Server的参数传入就是要复杂点 -- 框架参与**
```cpp
void reset_counter(const SetBool::Request::SharedPtr  &request,
                   const SetBool::Response::SharedPtr &response) {
        // 处理request
        if (request->data) {
            counter_ = 0;
            response->success = true;
            response->message = "Counter has been reset.";
        }
        else {
            response->success = false;
            response->message = "Failed to reset.";
        }
    }
```

#### 2.3.2 Client
```cpp
class AddTwoIntsClient : public rclcpp::Node {
public:
	AddTwoIntsClient(): Node("node_name") {
		// 在线程里跑do_request
		threads_.push_back(std::thread([this]() { do_request(21, 30); }));
	}
	
private:
	void do_request(int a, int b) {
		// wait for service
		while (!client_->wait_for_service(std::chrono::seconds(1))) {...}
		
		创建request
		
		// 通过future获取response
		auto future = client_->async_send_request(request); 
		auto response = future.get();
	}
	
	rclcpp::Client<AddTwoInts>::SharedPtr client_;
	std::vector<std::thread> threads_;
};
```
**client三步:**
- 等就绪
- 造 request
- async_send_request（回调式，别用 .get() 在回调里）


