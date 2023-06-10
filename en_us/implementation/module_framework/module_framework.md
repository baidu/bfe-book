# Module Framework

BFE defines a set of module mechanisms to quickly develop new functions by writing plug-ins. The code related to the module includes the following parts:

| Code Directory | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| bfe_module     | It includes the definition of module basic types, such as module interface, module callback type, callback chain type, callback framework type. |
| bfe_moudles    | It includes the implementation of built-in modules, such as various modules of traffic management, anti attack, traffic visibility, etc. |
| bfe_server     | It realizes the management of the life cycle of connection/request processing, and sets the callback point in the key processing stage. The module callback function chain registered at these callback points will be executed in order to support the customized processing of connections/requests. |

## Module Basic Types

### Definition of Module

BFE module represents a code unit with high cohesion, low coupling, and specific traffic processing functions (such as detection, filtering, rewriting, etc.).

Each module shall meet the following module interfaces (BfeModule), among which:

- The Name() method returns the name of the module

- The Init() method is used to initialize the module

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

In addition to the above basic interfaces, modules often need to implement specific callback interfaces to implement custom logic for connections or requests. The types of callback interfaces are detailed below.


### Type of Callback

In BFE, the following five callback types are defined according to different scenarios, which are used to process connections, requests or responses.

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

The detailed description of each callback type is as follows:

| Callback Type  | Description                                                  | Sample customized logic                                      |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| RequestFilter  | The callback function is used to process the incoming request and can directly return the response or continue to execute downward | Check the request and limit the traffic; Redirect request; Modify the request content and continue execution |
| ResponseFilter | The callback function is used to process the incoming response, modify the response and continue to execute downward | Modify the response content; Compress the response; Make statistics on response status codes; Record request log |
| AcceptFilter   | The callback function is used to process the incoming connection, and can close the connection or continue to execute downward | Check connection and limit traffic; Record the TLS handshake information of the connection and continue |
| ForwardFilter  | The callback function is used to process the request to be forwarded and modify the target backend instance of the request | Modify the target backend of the request and continue execution |
| FinishFilter   | The callback function is used to process the completed connection | Log the session and continue                                 |

The return value of the callback function determines the subsequent operation when the callback function is completed. The return value of the callback function has the following possible values:

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

The meaning of each return value is as follows:

| Return value of callback function | Description                                |
| --------------------------------- | ------------------------------------------ |
| BfeHandlerFinish                  | send response, then close connection.      |
| BfeHandlerGoOn                    | go on to next callback function.           |
| BfeHandlerRedirect                | redirect directly.                         |
| BfeHandlerResponse                | send response.                             |
| BfeHandlerClose                   | close connection without sending response. |


### HandlerList

HandlerList represents an ordered list of multiple callback functions. The type of callback functions in a callback function chain should be the same.

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

### Callback Framework

The callback framework manages the callback chain corresponding to all callback points in BFE.


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

The types of callback functions that can be registered at different callback points are different, as follows:

| Callback Point       | Description                                                  | Type of Callback Function |
| -------------------- | ------------------------------------------------------------ | ------------------------- |
| HandleAccept         | after TCP connection with client is established              | AcceptFilter              |
| HandleHandshake      | after SSL/TLS handshake with client is finished              | AcceptFilter              |
| HandleBeforeLocation | before the destination product for the request is identified | RequestFilter             |
| HandleFoundProduct   | after the destination product is identified                  | RequestFilter             |
| HandleAfterLocation  | after the destination cluster is identified                  | RequestFilter             |
| HandleForward        | after the destination subcluster is identified, and before the request is forwarded | ForwardFilter             |
| HandleReadResponse   | after response from backend is received by BFE               | ResponseFilter            |
| HandleRequestFinish  | after response from backend is forwarded by BFE              | ResponseFilter            |
| HandleFinish         | after connection with client is closed                       | FinishFilter              |


## Processing of Connection/Request  and Invoke of Callback Function

BFE's processing of connection/request and corresponding callback points in the forwarding process are shown in [Plugin Architecture](../../design/module/module. md). At each callback point, BFE will query the callback function chain registered at the specified callback point and execute the callback functions in the chain in order.

For example, when BFE receives the request header, it will query the callback function chain registered at callback point HandleBeforeLocation, then execute each callback function in order, and determine the subsequent operation based on the return value.

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


## links
Previous: [Chap29 Processing of Connections and Requests](../../../en_us/implementation/life_of_a_request/life_of_a_request.md)  
Next: [Chap31 Request Routing](../../../en_us/implementation/routing/routing.md)