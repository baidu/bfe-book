# 核心协议实现

BFE的HTTP/HTTP2/SPDY/WebSocket/TLS等网络协议一般基于golang官方开源协议库二次开发，并进行定制以更好满足反向代理的需求场景，例如优化性能、完善防攻击机制、增加探针等。

本章将重点介绍HTTP2协议的实现，其中HTTP2实现相对最复杂，SPDY的实现与HTTP2的实现非常相似，不再赘述。关于其它协议的实现，可参考代码结构说明章节查阅对应源码，也可在BFE开源社区提问交流。


## HTTP2协议

### HTTP2代码的组织

在bfe_https目录下可以看到包含了如下代码：

```
$ls bfe_http
errors.go       flow_test.go   headermap.go  http2_test.go     server_test.go  transport.go   z_spec_test.go
errors_test.go  frame.go       hpack         priority_test.go  state.go        write.go
flow.go         frame_test.go  http2.go      server.go         testdata        writesched.go
```

各文件的功能说明如下：

| 类别         | 文件名或子目录 | 说明                                    |
| ------------ | -------------- | --------------------------------------- |
| 流处理层     | server.go      | 协议连接核心处理逻辑                    |
|              | flow.go        | 流量控制窗口                            |
|              | writesched.go  | 协议帧发送优先级队列                    |
| 帧处理层     | frame.go       | 协议帧定义及解析                        |
|              | write.go       | 协议帧发送方法                          |
|              | hpack/         | 协议头部压缩算法HPACK                   |
| 基础数据类型 | headermap.go   | 常见请求头部定义                        |
|              | errors.go      | 协议错误定义                            |
|              | state.go       | 协议内部状态指标                        |
| 辅助工具     | transport.go   | 封装了HTTP2客户端；仅用于与后端实例通信 |




### HTTP2连接处理模块

BFE在接收到一个HTTP2连接后，除了创建连接处理主协程, 还会创建多个子协程配合完成协议逻辑的处理。单个HTTP2协议连接处理模块结构如图所示。

![http2 goroutines](http2_goroutines.png)

模块内部结构自底向上划分为三个层级：

**1.帧处理层**

- 帧处理层实现HTTP2协议帧序列化、压缩及传输
- 帧处理层包含两个独立收发协程，分别负责协议帧的接收与发送
- 帧处理层与流处理层通过管道通信 (RecvChan/SendChan/WroteChan)

**2.流处理层**

- 流处理层实现协议核心逻辑，例如：流创建、流数据传输、流关闭; 多路复用、流优先级、流量控制等
- 流处理层为每流创建Request/ResponseWriter实例，并在独立协程中运行应用逻辑

**3.接口层**

- 为HTTP应用Handler提供标准Request/ResponseWriter4实现, 屏蔽HTTP2协议数据传输细节
- HTTP应用Handler运行在Stream Goroutine协程中
- HTTP应用Handler通过Request实例获取HTTP 请求（读取自特定HTTP2流）
- HTTP应用Handler通过ResponseWriter实例发送HTTP响应（发往特定HTTP2流）




### HTTP2连接相关协程及关系 

  每个HTTP2连接中各协程基于CSP(Communicating Sequential Processes)并发模型协作，各协程的角色分工及交互关系如下：

**1.帧处理层的协程**

每个HTTP2连接同时包含2个读写携程负责从连接上接收或发送HTTP2协议帧

 * 帧接收协程(Frame Recv Goroutine) 从连接上读取HTTP2协议帧并放入帧接收队列
 * 帧发送协程(Frame Send Goroutine) 从帧发送队列获取帧并写入连接，同时将写结果放入写结果队列WroteChan

**2.流处理层的协程**

主协程与其它协程通过管道(golang Chan)进行通信, 例如:

 * BodyReadChan：请求处理协程读取请求Body后，通过BodyReadChan向主协程发送读结果消息，主协议接收到消息执行流量控制操作并更新流量控制窗口
 * WriteMsgChan: 请求处理协程发送响应后，通过WriteMsgChan向主协程发送写申请消息，主协议接收到消息后，转换为HTTP2数据帧并放入流发送队列。在合适到时机
 * ReadChan/SendChan/WroteChan：从连接上获取或发送HTTP2协议帧

**3.接口层的协程**

每个HTTP2连接为应用层封装了Request对象及ResponseWriter对象，并创建独立的请求处理协程（Stream Goroutine）处理请求并返回响应

 * Stream Goroutine 从Request对象中获取请求
 * Stream Goroutine 向ResponseWriter对象发送响应

