## 1.回顾总览层级关系

 ```cpp
   ① node        /turtlesim（进程里能开口说话的单元）
      │
   ② service     /clear /spawn /kill ...（挂在节点上的"服务台"）
      │          
      │
   ③ type        std_srvs/srv/Empty（服务名对应的"信纸格式合同"）
      │          三段式命名：包名 std_srvs / 类别 srv / 类型名 Empty
      │
   ④ interface   .srv 文件内容：
				    request 字段
				    --- 
				    response 字段
 ```
### 1. `node`
```zsh
ros2 node list
# output
/turtlesim
```
### 2. `service`
```zsh
ros2 service list
# output
/clear
```
### 3. `type`
```zsh
ros2 service type /clear
# output
std_srvs/srv/Empty
```
### 4. `interface`
```zsh
ros2 interface show std_srvs/srv/Empty
# output
---
```

**call service need exact interface**
```zsh
ros2 service call /clear std_srvs/srv/Empty   # /clear need Empty interface
# output
waiting for service to become available...
requester: making request: std_srvs.srv.Empty_Request()

response:
std_srvs.srv.Empty_Response()
```

> **node**是**人**
> **service**是**服务台**
> **type**是申请服务的**表格格式**
> **interface**是**具体的表格长什么样**


## 2.Empty 的深意--- srv 格式的灵魂

 > Q:为什么 `interface show std_srvs/srv/Empty` 输出就一个 `---` ?

### 1. `.srv`文件格式
 ```zsh
Request  字段区
---      分隔线
Response 字段区
 ```
 >  Empty: 空请求 + 空响应
 >  纯"命令触发"服务
 >  只要"执行"（清屏）
 >  "遥控指令"都用 Empty

