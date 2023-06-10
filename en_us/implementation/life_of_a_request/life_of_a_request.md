# Processing of Connections and Requests

## Establishment of Connection

The listener processing Goroutine of BFE (function BfeServer.Serve) circularly accepts the newly received client connection and creates a new Goroutine to process the connection.

```go
// bfe_server/http_server.go

func (srv *BfeServer) Serve(l net.Listener, raw net.Listener, proto string) error {
    ...
	  for {
			  // accept new connection
			  rw, e := l.Accept()
			  ...
			  // start goroutine for new connection
			  go func(rwc net.Conn, srv *BfeServer) {
				    c, err := newConn(rw, srv)
				    ...
			  	  c.serve()
			  }(rw, srv)
	  }
}
```

## Processing of Connection

BFE connection bfe_server.conn executes the serve function to process the connection, mainly including:

- Step 1 **Callback point processing**: Execute the callback function chain of callback point HandleAccept 

```go
// bfe_server/http_conn.go

// Callback for HANDLE_ACCEPT
hl = c.server.CallBacks.GetHandlerList(bfe_module.HandleAccept)
if hl != nil {
    retVal = hl.FilterAccept(c.session)
    ...
}
```

- Step 2 **Handshake and negotiation**: Perform TLS handshake (if the user initiates TLS connection)

  After the TLS handshake is successful, execute the callback function of the callback point HandleHandshake 

```go
// bfe_server/http_conn.go

// Callback for HANDLE_HANDSHAKE
hl = c.server.CallBacks.GetHandlerList(bfe_module.HandleHandshake)
if hl != nil {
    retVal = hl.FilterAccept(c.session)
    ...
}
```

​        Then select and execute the application layer protocol handler (HTTP2/SPDY/STREAM) based on the negotiation.

```go
// bfe_server/bfe_server.go

tlsNextProto[tls_rule_conf.SPDY31] = bfe_spdy.NewProtoHandler(nil)
tlsNextProto[tls_rule_conf.HTTP2] = bfe_http2.NewProtoHandler(nil)
tlsNextProto[tls_rule_conf.STREAM] = bfe_stream.NewProtoHandler(
		&bfe_stream.Server{BalanceHandler: srv.Balance})
```

+ Step 3 **Connection protocol processing**: Distinguish connection protocols, and execute:

  - If it is an HTTP (S) connection, read and process the requests in the current Goroutine in sequence

  - If it is an HTTP2/SPDY connection, read and process the request concurrently in the new Goroutine 

  - If it is a STREAM connection, the two-way forwarding of data is processed in the new Goroutine (omitted below)

​       For details of protocol implementation, please refer to the chapter [Core Protocol Implementation](../protocol/protocol. md).

## Processing of Request

The object for connection bfe_server.conn executes the serveRequest function to process the request. Although HTTP/HTTPS/HTTP2/SPDY use different methods to transmit data, BFE receives HTTP requests from the protocol layer and converts them to the same internal request type (bfe_http.Request) at the upper layer, and performs unified logical processing.

The specific process of request processing is as follows:

+ Step 1 **Callback point processing**

  Execute the callback function chain of the callback point HandleBeforeLocation 

```go
// bfe_server/reverseproxy.go

// Callback for HandleBeforeLocation
hl = srv.CallBacks.GetHandlerList(bfe_module.HandleBeforeLocation)
if hl != nil {
    retVal, res = hl.FilterRequest(basicReq)
    ...
}
```

+ Step 2 **Tenant Routing**:

  Find the tenant to which the request belongs. See the description in [Request Routing](../routing/routing. md) for details.

```go
// bfe_server/reverseproxy.go

// find product
if err := srv.findProduct(basicReq); err != nil {
    ...
}
```

+ Step 3 **Callback point processing**:

  Execute the callback function chain of the callback point HandleFoundProduct 

```go
// bfe_server/reverseproxy.go

// Callback for HandleFoundProduct
hl = srv.CallBacks.GetHandlerList(bfe_module.HandleFoundProduct)
if hl != nil {
    retVal, res = hl.FilterRequest(basicReq)
    ...
}
```

+ Step 4 **Cluster routing**:

  Find the destination cluster to which the request belongs. See the description in [Request Routing](../routing/routing. md) for details.

```go
// bfe_server/reverseproxy.go

if err = srv.findCluster(basicReq); err != nil {
    ...
}
```

- Step 5 **Callback point processing**: Execute the callback function chain of the callback point HandleAfterLocation 

```go
// bfe_server/reverseproxy.go

// Callback for HandleAfterLocation
hl = srv.CallBacks.GetHandlerList(bfe_module.HandleAfterLocation)
if hl != nil {
    ...
}
```

- Step 6 **Request pre-processing**:

  Before the final forwarding of the request, preprocess the request and obtain the forwarding parameters (such as the timeout)

- Step 7 **Load balancing and forwarding**:

  Forward HTTP requests to the downstream cluster. See the description in [Load Balancing](../balancing/balancing. md) for details

```go
// bfe_server/reverseproxy.go

res, action, err = p.clusterInvoke(srv, cluster, basicReq, rw)
basicReq.HttpResponse = res
```


- Step 8 **Callback point processing**: Execute the callback function chain of the callback point HandleReadResponse
```go
// bfe_server/reverseproxy.go

// Callback for HandleReadResponse
hl = srv.CallBacks.GetHandlerList(bfe_module.HandleReadResponse)
if hl != nil {
    ...
}
```


- Step 9 **Response sending**: send the response to the client

```go
// bfe_server/reverseproxy.go

err = p.sendResponse(rw, res, resFlushInterval, cancelOnClientClose)
if err != nil {
    ...
}
```

## End of Request

- Execute the callback function chain of the callback point HandleRequestFinish 

```go
// bfe_server/reverseproxy.go

// Callback for HandleRequestFinish
hl := srv.CallBacks.GetHandlerList(bfe_module.HandleRequestFinish)
if hl != nil {
    ...
}
```

- Check whether the connection needs to be closed (for example, the request is blocked or HTTP KeepAlive is not enabled)

- If it needs to be closed, the connection will stop reading subsequent requests and perform the closing operation


## End of Connection

Before the connection ends, you need to perform the following operations:

- Execute the callback function chain of the callback point HandleFinish

```go
// bfe_server/http_conn.go

// Callback for HandleFinish
hl := srv.CallBacks.GetHandlerList(bfe_module.HandleFinish)
if hl != nil {
    hl.FilterFinish(c.session)
}
```

- Write the "connection outgoing" buffer data and close the connection


## links
Previous: [Chap28 Process Model](../../../en_us/implementation/process_model/process_model.md)  
Next: [Chap30 Module Framework](../../../en_us/implementation/module_framework/module_framework.md)