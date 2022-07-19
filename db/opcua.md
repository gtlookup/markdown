https://documentation.unified-automation.com/uasdkc/1.9.2/html/index.html

所有的动态内存都是通过 `OpcUa_Alloc` 分配、`OpcUa_Free` 释放

# 编译sdk

```bash
# 1. 安装编译工具
apt install libssl-dev
apt install cmake
# 2. 进到sdk目录，编译sdk
./buildExamples.sh -c Debug -s ON # ON：动态链接库，OFF：静态链接库
```

# client

**UaClient_Discovery：**结构体允许应用发现 `server` 并找到对应的 `endpoints`

**UaClient_Subscription：**与一个服务的订阅，拥有一个或多个 `MonitoredItems`，`MonitoredItems` 用于监听单个节点所产生的数据或事件 。订阅由一个 `session` 拥有。

## session

### 概述

1）`UaClient_Session` 类代表客户端应用与单个 `server` 之间的连接

2）维护一个活动的订阅列表，监听连接状态、处理自动重连，并通知应用连接状态的变化

3）一个进程可创建多个连接到不同 `server` 的 `session` 对象

4）`UaClient_Session` 非线程安全，只能从由主线程使用

### 连接处理及重连

1）调用 `UaClient_Session_BeginConnect` 创建并维护一个服务的 `session`

2）连成功后读取 `server` 上的状态来监控连接（以下可在 `Session` 中配置）

- UaClient_Session::WatchdogTime：读取周期时间
- UaClient_Session::WatchdogTimeout：读取超时时间

3）通过 `UaClient_Session_ConnectionStatusChanged_CB` 接收连接状态变化

4）重连由 `SDK` 完成，但需要在重连过程中手动创建个新 `session`

- UaClient_ConnectionStatus::UaClient_ConnectionStatus_SessionAutomaticallyRecreated：表示重连状态
- 重连后，所有节点要重新注册；如果客户端维护缓存，也需要重新创建

5）初次连接失败，可设置 `UaClient_Session::AutomaticReconnect` 为 `OpcUa_True` 告诉客户端重连

### SessionId

server 内部创建的句柄，用于在 C / S 两端之间通信。根据协议，与安全有关，且不该在 C / S 端之外可见

### UserData

1）**`UaClient_Session::pUserData`** 是客户端定义指向任意类型数据的指针，需要手动释放其内存资源

2）可用于区分 `UaClient_Session_ConnectionStatusChanged_CB` 等回调函数中的 `session`

## 订阅

### 概述

1）`UaClient_Subscription` 类表示 server 上一个订阅，线程不安全，需在主线程中使用

2）通过 `UaClient_Subscription_Callback` 这个回调监听 `MonitoredItems` 及其内部的数据和事件

3）`UaClient_Subscription` 不维护参数和 `MonitoredItems` 的本地副本

4）如果订阅失效，客户端需手动重新向 `server` 端发起订阅

5）无效的订阅会通过 `UaClient_Subscription_StatusChanged_CB` 报告

### SubscriptionId

1）`UaClient_Subscription::SubscriptionId` 是 `server` 内部创建的句柄，用于在 C / S 两端通信

2）由于 `SubscriptionId` 是向另一个 `session` 转移订阅的必要条件，因此会暴露给 sdk 用户

### UserData

1）`UaClient_Subscription::pUserData` 是用户定义的指向任意类型的指针，需用户手动释放其内存

2）可用于区分 `UaClient_Subscription_StatusChanged_CB` 等回调函数中的订阅

3）是调用每个服务端方法的参数

### 监听

1）`MonitoredItemId` 是 `OpcUa_MonitoredItemCreateResult` 中返回的一个服务端定义的句柄，是修改或删除被监听项目的必要每件

2）`ClientHandle` 是用户分配的，每个 `UaClient_Subscription_Callback` 实例是唯一的

3）如果不同订阅分配了唯一的 `ClientHandles`，则 `UaClient_Subscription::pUserData` 与数据变化或事件回调无关

​      此情况下 `UaClient_Subscription::pUserData` 句柄只与订阅状态变化回调有关
