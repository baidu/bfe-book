# 支持更多协议

本章将介绍如何配置BFE以支持更多的协议，如：HTTP/2、SPDY、TLS、WebSocket、FastCGI等。

## HTTP/2配置
为支持HTTP/2 over TLS，用户需修改TLS规则配置文件 conf/tls_conf/tls_rule_conf.data。

在该文件中，对需要配置的产品线，修改"NextProtos"域，增加"h2"选项，这将在TLS握手的ALPN协商中增加对HTTP2的支持。如果客户端也支持HTTP/2，建立的连接将使用HTTP/2。
如下述实例，我们对产品线"example_product"进行修改，增加了"h2"。

```json
"example_product": {
    ...
    "NextProtos": ["h2", "http/1.1"],
    ...
}

```

使用curl连接BFE的HTTPS端口进行测试，可以看到连接已经使用HTTP/2。

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

上述"NextProtos"配置同时包含了["h2", "http/1.1"]，这表示会同时支持HTTP/2和HTTP/1.1，其中HTTP/2的优先级较高。

如果希望只使用HTTP/2，可以将"NextProtos"设置为"h2;level=2"。level为协商级别，2表示必选。如下示例：

```json
"example_product": {
    ...
    "NextProtos": ["h2;level=2"],
    ...
}
```

## SPDY配置
与HTTP/2相似，在"NextProtos"中设置"spdy/3.1"，可开启对SPDY的支持。
```json
"example_product": {
    ...
    "NextProtos": ["spdy/3.1", "http/1.1"],
    ...
}
```

## WebSocket配置
* 支持WS（WebSocket over TCP）

支持ws无需特殊配置，只需配置好支持ws的后端服务和正确的转发路由即可。

使用curl连接BFE的HTTP端口进行测试，带上header："Connection: Upgrade"和"Upgrade: websocket"。我们可以看到连接升级为使用websocket，如下：

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

* 支持WSS（WebSocket over TLS）

为支持WSS，在"NextProtos"中设置如下：

```json
"example_product": {
    ...
    "NextProtos": ["stream;level=2"],
    ...
}
```
使用curl连接BFE的HTTPS端口进行测试，可以看到连接也升级为使用WebSocket：
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

## 连接后端服务的协议

BFE支持使用不同的协议连接后端服务器。我们可以在后端集群配置文件conf/server_data_conf/cluster_conf.data中指定该配置。支持的协议包括：
* HTTP
* h2c
* FastCGI

### HTTP

HTTP为连接后端使用的缺省协议，无需特殊配置。

### h2c

为支持h2c，将"BackendConf"中的"Protocol"设置为"h2c"。

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
为支持FastCGI，将"BackendConf"中的"Protocol"设置为"fcgi"。其他可设置的参数包括请求文件的根路径"Root"和环境变量"EnvVars"。

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

## links
上一章：[第二十五章 配置限流](../../operation/configuration/prison.md)  
下一章：[第二十七章 BFE的代码组织](../../implementation/source_layout/source_layout.md)