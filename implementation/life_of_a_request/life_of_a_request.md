# 请求处理流程及响应



## 连接的建立

BFE的监听器处理协程（bfe_server/http_server.go: BfeServer.Serve函数)循环接受新到连接，并创建新协程处理该连接。

```go
// bfe_server/http_server.go

func (srv *BfeServer) Serve(l net.Listener, raw net.Listener, proto string) error {
	...
	for {
		// accept new connection
		rw, e := l.Accept()
		...
		// start go-routine for new connection
		go func(rwc net.Conn, srv *BfeServer) {
			c, err := newConn(rw, srv)
			...
			c.serve()
		}(rw, srv)
	}
}
```

## 连接的处理

BFE的连接对象bfe_server.conn 执行serve函数处理该连接，主要包含：

- **回调点处理**：执行HandleAccept回调点回调链函数

```go
// bfe_server/http_conn.go

// Callback for HANDLE_ACCEPT
hl = c.server.CallBacks.GetHandlerList(bfe_module.HandleAccept)
if hl != nil {
    retVal = hl.FilterAccept(c.session)
    ...
}
```


- **握手及协商**：执行TLS握手（如果用户发起TLS连接）
  - **回调点处理**：执行HandleHandshake回调点回调链函数

```go
// bfe_server/http_conn.go

// Callback for HANDLE_HANDSHAKE
hl = c.server.CallBacks.GetHandlerList(bfe_module.HandleHandshake)
if hl != nil {
    retVal = hl.FilterAccept(c.session)
    ...
}
```


  - 基于协商协议，选择并执行应用层协议Handler（HTTP2/SPDY/STREAM）

```go
// bfe_server/bfe_server.go

tlsNextProto[tls_rule_conf.SPDY31] = bfe_spdy.NewProtoHandler(nil)
tlsNextProto[tls_rule_conf.HTTP2] = bfe_http2.NewProtoHandler(nil)
tlsNextProto[tls_rule_conf.STREAM] = bfe_stream.NewProtoHandler(
		&bfe_stream.Server{BalanceHandler: srv.Balance})
```

- **连接协议处理**：区分连接的协议，执行：
  - 如果是HTTP(S)连接，在当前协程中顺序读取请求并处理
  - 如果是HTTP2/SPDY连接，在新建协程中并发读取请求并处理
  - 如果是STREAM连接，在新建协程处理数据的双向转发（下文略去）
  
  关于协议的实现说明，详见协议实现章节。
  
  

## 请求的处理

BFE的连接对象bfe_server.conn 执行serveRequest函数处理请求。虽然HTTP/HTTPS/HTTP2/SPDY/使用不同方式传输数据，但BFE从协议层接收到HTTP请求后，在上层都转化为相同的内部请求类型(bfe_http.Request)，并执行统一的逻辑处理。

请求处理的具体流程如下：

- 步骤1. **回调点处理**

  执行HandleBeforeLoation回调点回调链函数

```go
// bfe_server/reverseproxy.go

// Callback for HandleBeforeLocation
hl = srv.CallBacks.GetHandlerList(bfe_module.HandleBeforeLocation)
if hl != nil {
    retVal, res = hl.FilterRequest(basicReq)
    ...
    }
}
```

- 步骤2. **租户路由**：

  查找请求归属的租户。详见请求路由章节。

```go
// bfe_server/reverseproxy.go

// find product
if err := srv.findProduct(basicReq); err != nil {
    ...
}
```

- 步骤2.**回调点处理**：

  执行HandleFoundProduct回调点回调链函数
```go
// bfe_server/reverseproxy.go

// Callback for HandleFoundProduct
hl = srv.CallBacks.GetHandlerList(bfe_module.HandleFoundProduct)
if hl != nil {
    retVal, res = hl.FilterRequest(basicReq)
    ...
}
```

- **租户路由**：

  查找请求归属的目的集群。详见请求路由章节。
```go
// bfe_server/reverseproxy.go

if err = srv.findCluster(basicReq); err != nil {
    ...
}
```

- **回调点处理**：执行HandleAfterLocation回调点回调链函数
```go
// bfe_server/reverseproxy.go

// Callback for HandleAfterLocation
hl = srv.CallBacks.GetHandlerList(bfe_module.HandleAfterLocation)
if hl != nil {
    ...
}
```

- **请求预处理**：

  对请求进行预处理并获取转发参数（例如超时时间）

- **负载均衡及转发**：

  向下游集群转发HTTP请求。详见负载均衡章节

```go
// bfe_server/reverseproxy.go

res, action, err = p.clusterInvoke(srv, cluster, basicReq, rw)
basicReq.HttpResponse = res
```


- **回调点处理**：执行HandleReadResponse回调点回调链函数
```go
// bfe_server/reverseproxy.go

// Callback for HandleReadResponsehl = srv.CallBacks.GetHandlerList(bfe_module.HandleReadResponse)
if hl != nil {
    ...
}
```


- **响应发送**：向用户端发送响应

```go
// bfe_server/reverseproxy.go

err = p.sendResponse(rw, res, resFlushInterval, cancelOnClientClose)
if err != nil {
    ...
}
```

## 请求的结束

- 执行HandleRequestFinish回调点回调链函数
```go
// bfe_server/http_conn.go

// Callback for HandleFinish
hl := srv.CallBacks.GetHandlerList(bfe_module.HandleFinish)
if hl != nil {
    hl.FilterFinish(c.session)
}
```

- 检查连接是否需关闭（例如请求被封禁或HTTP KeepAlive未启用）
- 如需关闭，连接将停止读取后续请求并执行关闭操作


## 连接的结束

连接在结束前，还需要执行以下操作：

- 执行HandleFinish回调点回调链函数
- 写出连接缓存区数据并关闭连接

