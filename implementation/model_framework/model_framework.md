# 模块框架

BFE定义了一套模块机制，通过编写模块插件来快速开发新功能。与模块相关的代码包含以下部分：

| 代码目录    | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| bfe_module  | 包含了模块基础类型的定义，例如模块的接口、模块回调类型、回调链类型、回调框架类型 |
| bfe_moudles | 包含了内置模块的实现，例如流量管理、安全防攻击、流量可见性等类别的各种模块 |
| bfe_server  | 实现了连接/请求处理的生命周期的管理，并在关键处理阶段设置了回调点。注册在这些回调点的模块回调函数链，将被顺序执行，从而支持对连接/请求的定制化处理 |

## 模块基础类型

### 模块的定义

BFE模块代表一个高内聚、低耦合、实现特定流量处理功能（例如检测、过滤、改写等）的代码单元。

每个模块需满足如下模块接口(BfeModule)，其中：

- Name() 方法返回模块的名称
- Init() 方法用于执行模块的初始化

```go
// bfe_module/bfe_module.go

type BfeModule interface {
    // Name returns the name of module.
    Name() string

    // Init initializes the module. The cbs are callback handlers for  
    // processing connection or request/response. The whs are web monitor 
    // handlers for exposing internal status or reloading specified 
    // configuration. The cr is the root path of module configuration 
    // files.
    Init(cbs *BfeCallbacks, whs *web_monitor.WebHandlers, cr string) error
}
```

模块除了需要实现以上基础接口外，往往还需实现特定回调接口，针对连接或请求实现自定义逻辑。关于回调接口的类型在下文详述。


### 模块的回调类型

在BFE中按需求场景定义了如下5种回调类型，用于处理连接、请求或响应。

```go
// bfe_module/bfe_filter.go

// RequestFilter filters incomming requests and return a response or nil.
// Filters are chained together into a HandlerList.
type RequestFilter interface {
    FilterRequest(request *bfe_basic.Request) (int, *bfe_http.Response)
}

// ResponseFilter filters outgoing responses. This can be used to modify 
// the response before it is sent.
type ResponseFilter interface {
    FilterResponse(req *bfe_basic.Request, res *bfe_http.Response) int
}

// AcceptFilter filters incoming connections.
type AcceptFilter interface {
    FilterAccept(*bfe_basic.Session) int
}

// ForwardFilter filters to forward request
type ForwardFilter interface {
    FilterForward(*bfe_basic.Request) int
}

// FinishFilter filters finished session(connection)
type FinishFilter interface {
    FilterFinish(*bfe_basic.Session) int
}
```

各回调类型的详细说明如下：

| 回调类型       | 说明                                                       | 示例自定义逻辑                                               |
| -------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| RequestFilter  | 回调函数用于处理到达的请求，并可直接返回响应或继续向下执行 | 检测请求并限流；重定向请求；修改请求内容并继续执行           |
| ResponseFilter | 回调函数用于处理到达的响应，并可修改响应并继续向下执行     | 修改响应的内容；对响应进行压缩；统计响应状态码；记录请求日志 |
| AcceptFilter   | 回调函数用于处理到达的连接，并可关闭连接或并继续向下执行   | 检测连接并限流；记录连接的TLS握手信息并继续执行              |
| ForwardFilter  | 回调函数用于处理待转发的请求，并可修改请求的目的后端实例   | 修改请求的目标后端实例并继续执行                             |
| FinishFilter   | 回调函数用于处理完成的连接                                 | 记录会话日志并继续执行                                       |

回调函数具有以下几种返回值，决定了回调函数执行完成时，后续的操作

```go
// bfe_module/bfe_handler_list.go

// Return value of handler.
const (
    BfeHandlerFinish   = 0 // to close the connection after response
    BfeHandlerGoOn     = 1 // to go on next handler
    BfeHandlerRedirect = 2 // to redirect
    BfeHandlerResponse = 3 // to send response
    BfeHandlerClose    = 4 // to close the connection directly, with no data sent.
)
```

各返回值代表含义如下：

| 回调函数返回值     | 含义说明                               |
| ------------------ | -------------------------------------- |
| BfeHandlerFinish   | 直接返回响应，在响应发送完成后关闭连接 |
| BfeHandlerGoOn     | 继续向下执行                           |
| BfeHandlerRedirect | 直接返回重定向响应                     |
| BfeHandlerResponse | 直接返回指定的响应                     |
| BfeHandlerClose    | 直接关闭连接                           |




### 回调链类型

回调链(HandlerList)代表了多个回调函数构成的有序列表。一个回调链中的回调函数类型是相同的。

```go
// bfe_module/bfe_handler_list.go

// HandlerList type.
const (
    HandlersAccept   = 0 // for AcceptFilter
    HandlersRequest  = 1 // for RequestFilter
    HandlersForward  = 2 // for ForwardFilter
    HandlersResponse = 3 // for ResponseFilter
    HandlersFinish   = 4 // for FinishFilter
)

type HandlerList struct {
    handlerType int        // type of handlers (filters)
    handlers    *list.List // list of handlers (filters)
}
```



### 回调框架

回调框架管理了BFE中所有回调点对应的回调链。


```go
// bfe_module/bfe_callback.go

// Callback point.
const (
    HandleAccept         = 0
    HandleHandshake      = 1
    HandleBeforeLocation = 2
    HandleFoundProduct   = 3
    HandleAfterLocation  = 4
    HandleForward        = 5
    HandleReadResponse   = 6
    HandleRequestFinish  = 7
    HandleFinish         = 8
)

type BfeCallbacks struct {
    callbacks map[int]*HandlerList
}
```

不同回调点可注册的回调函数类型不同，具体情况如下：

| 回调点               | 回调点含义             | 回调函数类型   |
| -------------------- | ---------------------- | -------------- |
| HandleAccept         | 与用户端连接建立成功后 | AcceptFilter   |
| HandleHandshake      | 与用户端TLS握手成功后  | AcceptFilter   |
| HandleBeforeLocation | 请求执行租户路由前     | RequestFilter  |
| HandleFoundProduct   | 请求执行租户路由后     | RequestFilter  |
| HandleAfterLocation  | 请求执行集群路由后     | RequestFilter  |
| HandleForward        | 请求向后端转发前       | ForwardFilter  |
| HandleReadResponse   | 读取到后端响应头部时   | ResponseFilter |
| HandleRequestFinish  | 向用户端发送响应完成时 | ResponseFilter |
| HandleFinish         | 与用户端连接处理完成时 | FinishFilter   |

## BFE内置模块

在BFE源码的 bfe_modules/<module_name> 目录中内置了大量的扩展模块。简要说明如下：

| 模块类别   | 模块名称           | 模块说明                                                     |
| ---------- | ------------------ | ------------------------------------------------------------ |
| 流量管理   | mod_rewrite        | 根据自定义条件，修改请求的URI                                |
|            | mod_header         | 根据自定义条件，修改请求或响应的头部                         |
|            | mod_redirect       | 根据自定义条件，对请求进行重定向                             |
|            | mod_geo            | 基于地理信息字典，通过用户IP获取相关的地理信息               |
|            | mod_tag            | 根据自定义的条件，为请求设置Tag标识                          |
|            | mod_logid          | 用来生成请求标识及会话标识                                   |
|            | mod_trust_clientip | 基于配置信任IP列表，检查并标识访问用户真实IP是否属于信任IP   |
|            | mod_doh            | 支持DNS over HTTPS                                           |
|            | mod_compress       | 根据自定义条件，支持对响应主体压缩                           |
|            | mod_errors         | 根据自定义条件，将响应内容替换为/重定向至指定错误页          |
|            | mod_static         | 根据自定义条件，返回静态文件作为响应                         |
|            | mod_userid         | 根据自定义的条件，为新用户自动在Cookie中添加用户标识         |
|            | mod_markdown       | 根据自定义条件，将响应中markdown格式内容转换为html格式       |
| 安全防攻击 | mod_auth_basic     | 根据自定义的条件，支持HTTP基本认证                           |
|            | mod_auth_jwt       | 根据自定义的条件，支持JWT (JSON Web Token)认证               |
|            | mod_auth_request   | 根据自定义的条件，支持将请求转发认证服务进行认证。           |
|            | mod_block          | 根据自定义的条件，对连接或请求进行封禁                       |
|            | mod_prison         | 根据自定义的条件，限定单位时间用户的访问频次                 |
|            | mod_waf            | 根据自定义的条件，对请求执行应用防火墙规则检测或封禁         |
|            | mod_cors           | 根据自定义的条件，设置跨源资源共享策略                       |
|            | mod_secure_link    | 根据自定义的条件，对请求签名或有效期进行验证                 |
| 流量可见性 | mod_access         | 以指定格式记录请求日志和会话日志                             |
|            | mod_key_log        | 以NSS key log格式记录TLS会话密钥, 便于基于解密分析TLS密文流量诊断分析 |
|            | mod_trace          | 根据自定义的条件，为请求开启分布式跟踪                       |
|            | mod_http_code      | 统计HTTP响应状态码                                           |

我们可以通过访问BFE实例的 http://localhost:8299/monitor/module_handlers 监控地址，查看到当前运行的BFE实例中所有的回调点、在各回调点注册的模块回调函数列表以及顺序。

## 连接/请求处理及回调函数的调用

BFE在转发过程对连接/请求对处理及回调点如模块设计章节图示。在各回调点，BFE将查询指定回调点注册的回调链，并按顺序执行回调链中的各回调函数。

例如，BFE在接收到用户请求头时，将查询在HandleBeforeLocation回调点注册的回调链，然后按序执行各回调函数，并基于返回值决定后续操作。

```go
// bfe_server/reverseproxy.go

// Get callbacks for HandleBeforeLocation
hl = srv.CallBacks.GetHandlerList(bfe_module.HandleBeforeLocation)
if hl != nil {
    // process FilterRequest handlers
    retVal, res = hl.FilterRequest(basicReq)
    basicReq.HttpResponse = res
		
    switch retVal {
    case bfe_module.BfeHandlerClose:
        // Close the connection directly (without reply)
        action = closeDirectly
        return
			
    case bfe_module.BfeHandlerFinish:
        // Close the connection after response
        action = closeAfterReply
        basicReq.BfeStatusCode = bfe_http.StatusInternalServerError
        return

    case bfe_module.BfeHandlerRedirect:
        // Make an redirect
        Redirect(rw, req, basicReq.Redirect.Url, basicReq.Redirect.Code, basicReq.Redirect.Header)
        isRedirect = true
        basicReq.BfeStatusCode = basicReq.Redirect.Code
        goto send_response

    case bfe_module.BfeHandlerResponse:
        // Send the generated response
        goto response_got
    }
}
```


