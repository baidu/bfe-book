# Support More Protocols

This chapter describes how to configure BFE to support more protocols, such as HTTP/2, SPDY, TLS, WebSocket, FastCGI, etc.

## HTTP/2 Configuration
To support HTTP/2 over TLS, users need to modify the TLS rule configuration file "conf/tls_conf/tls_rule_conf.data".

In this file, for the product to be configured, modify the "NextProtos" and add the "h2" option, which means adding HTTP2 support in the ALPN negotiation of TLS handshake. If the client supports HTTP/2, the established connection will use HTTP/2.

As shown in the following example, we modified the product "example_product" and added "h2".

```json
"example_product": {
    ...
    "NextProtos": ["h2", "http/1.1"],
    ...
}
```

Use curl to connect to the HTTPS port of BFE for testing. You can see that the connection has used HTTP/2.

```
# curl -v -k -H "Host: example.org"  https://localhost:8443
* Rebuilt URL to: https://127.0.0.1:8443/
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
...
* ALPN, server accepted to use h2
...
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7fabe6000000)
> GET / HTTP/2
> Host: example.org
> User-Agent: curl/7.54.0
> Accept: */*
...
```

The above "NextProtos" configuration includes both "h2" and "http/1.1", which means that HTTP/2 and HTTP/1.1 will be supported at the same time, of which HTTP/2 has a higher priority.

If you want to use only HTTP/2, you can set "NextProtos" to "h2; level=2". Level is the negotiation level, and 2 means "required". Here is an example:

```json
"example_product": {
    ...
    "NextProtos": ["h2;level=2"],
    ...
}
```

## SPDY Configuration
Similar to HTTP/2, set "spdy/3.1" in "NextProtos" to enable SPDY support.

```json
"example_product": {
    ...
    "NextProtos": ["spdy/3.1", "http/1.1"],
    ...
}
```

## WebSocket Configuration
### Support WS(WebSocket over TCP)

No special configuration is required to support ws, just configure the backend services that support ws and correct forwarding routes.

Use curl to connect to the HTTP port of BFE for testing, with header: "Connection: Upgrade" and "Upgrade: websocket". We can see that the connection is upgraded to use websocket, as follows:

```
# curl -v \
       -H "Host: example.org" \
       -H "Connection: Upgrade"  \
       -H "Upgrade: websocket" \
       -H "sec-websocket-key:1234567890"  \
       http://127.0.0.1:8080
* Rebuilt URL to: http://127.0.0.1:8080/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
> GET / HTTP/1.1
> Host: example.org
> User-Agent: curl/7.54.0
> Accept: */*
> Connection: Upgrade
> Upgrade: websocket
> sec-websocket-key:1234567890
> 
< HTTP/1.1 101 Switching Protocols
< Connection: Upgrade
< Sec-Websocket-Accept: qrAsbG+EeIo8ooFLgckbiuFt1YE=
< Upgrade: websocket
```

### Support WSS(WebSocket over TLS)

To support WSS, set the following in "NextProtos":

```json
"example_product": {
    ...
    "NextProtos": ["stream;level=2"],
    ...
}
```
Use curl to connect to the HTTPS port of BFE for testing. You can see that the connection is also upgraded to use WebSocket:

```
# curl -v -k \
       -H "Host: example.org" \
       -H "Connection: Upgrade"  \
       -H "Upgrade: websocket" \
       -H "sec-websocket-key:1234567890"  \
       https://127.0.0.1:8443

* Rebuilt URL to: https://127.0.0.1:8443/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8443 (#0)
...
> GET / HTTP/1.1
> Host: example.org
> User-Agent: curl/7.54.0
> Accept: */*
> Connection: Upgrade
> Upgrade: websocket
> sec-websocket-key:1234567890
> 
< HTTP/1.1 101 Switching Protocols
< Connection: Upgrade
< Sec-Websocket-Accept: qrAsbG+EeIo8ooFLgckbiuFt1YE=
< Upgrade: websocket
...
```

## Protocols for Connecting Backend Services

BFE supports connecting back-end servers using different protocols. We can specify this configuration in the backend cluster configuration file "conf/server_data_conf/cluster_conf.data". Supported protocols include:

* HTTP

* h2c

* FastCGI

### HTTP

HTTP is the default protocol used to connect to the backend, without special configuration.

### h2c

To support h2c, set the "Protocol" in "BackendConf" to "h2c".

```json
"cluster_example": {
    ...
    "BackendConf": {
        "Protocol": "h2c",
        ...
    },
    ...
}
```

### FastCGI
To support FastCGI, set the "Protocol" in "BackendConf" to "fcgi". Other parameters that can be set include "Root" for root path of accessing file and "EnvVars" for the environment variable.

```json
"cluster_example": {
    "BackendConf": {
        "Protocol": "fcgi",
        ...
        "FCGIConf": {
            "Root": "/home/work",
            "EnvVars": {
                "VarKey": "VarVal"
            }    
        }
    }
}
```
