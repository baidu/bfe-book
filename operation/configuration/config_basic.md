# BFE基础功能配置
在上一章中，基于安装包中的缺省配置，我们部署运行了BFE服务。本章将对BFE的配置文件进行详细介绍，帮助读者了解如何配置一个符合自身业务需求的BFE服务。

## 配置文件概述
BFE的所有配置文件存放在conf子目录下，该目录中包含多种的配置文件。

其中bfe.conf为BFE的基础配置。其他配置文件，按功能分类分别存放在相应子目录中。 其中，以“mod_”开头的多个目录为扩展模块的配置文件。扩展模块的配置将在后续章节中具体描述。

## 基础配置文件*bfe.conf*

*bfe.conf*包含了BFE的基础配置。读者了解配置文件，首先需要从这个文件入手。

### 配置示例

如以下示例，*bfe.conf*包括几部分：

* 服务基础配置，包括服务端口、超时时间、配置文件路径和开启的模块等等。
* TLS基础配置，包括证书文件，加密套件等。

```ini
[Server]
# listen port for http request
HttpPort = 8080
# listen port for https request
HttpsPort = 8443
# listen port for monitor request
MonitorPort = 8421

# max number of CPUs to use (0 to use all CPUs)
MaxCpus = 0

# type of layer-4 load balancer (PROXY/NONE)
Layer4LoadBalancer = ""

# tls handshake timeout, in seconds
TlsHandshakeTimeout = 30

# read timeout, in seconds
ClientReadTimeout = 60

# write timeout, in seconds
ClientWriteTimeout = 60

# if false, client connection is shutdown disregard of http headers
KeepAliveEnabled = true

# timeout for graceful shutdown (maximum 300 sec)
GracefulShutdownTimeout = 10

# max header length in bytes in request
MaxHeaderBytes = 1048576

# max URI(in header) length in bytes in request
MaxHeaderUriBytes = 8192

# routing related conf
HostRuleConf = server_data_conf/host_rule.data
VipRuleConf = server_data_conf/vip_rule.data
RouteRuleConf = server_data_conf/route_rule.data
ClusterConf = server_data_conf/cluster_conf.data

# load balancing related conf
GslbConf = cluster_conf/gslb.data
ClusterTableConf = cluster_conf/cluster_table.data

# naming related conf
NameConf = server_data_conf/name_conf.data

# moduels enabled
Modules = mod_trust_clientip
Modules = mod_block
Modules = mod_header
Modules = mod_rewrite
Modules = mod_redirect
Modules = mod_logid

# interval for get diff of proxy-state
MonitorInterval = 20

# debug flags
DebugServHttp = false
DebugBfeRoute = false
DebugBal = false
DebugHealthCheck = false

[HttpsBasic]
# tls cert conf
ServerCertConf = tls_conf/server_cert_conf.data

# tls rule
TlsRuleConf = tls_conf/tls_rule_conf.data

# ciphersuites
CipherSuites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256|TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
CipherSuites=TLS_ECDHE_RSA_WITH_RC4_128_SHA
CipherSuites=TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
CipherSuites=TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
CipherSuites=TLS_RSA_WITH_RC4_128_SHA
CipherSuites=TLS_RSA_WITH_AES_128_CBC_SHA
CipherSuites=TLS_RSA_WITH_AES_256_CBC_SHA

# supported curve perference settings
CurvePreferences=CurveP256

# support Sslv2 ClientHello 
EnableSslv2ClientHello = true

# client ca certificates base directory
ClientCABaseDir = tls_conf/client_ca

[SessionCache]
# disable tls session cache or not
SessionCacheDisabled = true

# address of cache server
Servers = "example.redis.cluster"

# prefix for cache key
KeyPrefix = "bfe"

# connection params (ms)
ConnectTimeout = 50
ReadTimeout = 50
WriteTimeout = 50

# max idle connections in connection pool
MaxIdle = 20

# expire time for tls session state (second)
SessionExpire = 3600

[SessionTicket]
# disable tls session ticket or not
SessionTicketsDisabled = true
# session ticket key
SessionTicketKeyFile = tls_conf/session_ticket_key.data
```


### 基础配置项描述

* 服务基础配置

| 配置项       | 描述          |
| ---------- | --------------- |
| Server.HttpPort                | Integer<br>HTTP监听端口<br>默认值8080 |
| Server.HttpsPort               | Integer<br>HTTPS(TLS)监听端口<br>默认值8443 |
| Server.MonitorPort             | Integer<br>Monitor监听端口<br>默认值8421 |
| Server.MaxCpus                 | Integer<br>最大使用CPU核数; 0代表使用所有CPU核<br>默认值0 |
| Server.Layer4LoadBalancer      | String<br>四层负载均衡器类型(PROXY/NONE)<br>默认值NONE |
| Server.TlsHandshakeTimeout     | Integer<br>TLS握手超时时间，单位为秒<br>默认值30 |
| Server.ClientReadTimeout       | Integer<br>读客户端超时时间，单位为秒<br>默认值60 |
| Server.ClientWriteTimeout      | Integer<br>写客户端超时时间，单位为秒<br>默认值60 |
| Server.GracefulShutdownTimeout | Integer<br>优雅退出超时时间，单位为秒，最大300秒<br>默认值10 |
| Server.KeepAliveEnabled        | Boolean<br>与用户端连接是否启用HTTP KeepAlive<br>默认值True |
| Server.MaxHeaderBytes          | Integer<br>请求头部的最大长度，单位为Byte<br>默认值1048576 |
| Server.MaxHeaderUriBytes       | Integer<br>请求头部URI的最大长度，单位为Byte<br>默认值8192 |
| Server.HostRuleConf            | String<br>租户域名表配置文件路径<br>默认值server_data_conf/host_rule.data |
| Server.VipRuleConf             | String<br>租户VIP表配置文件路径<br>默认值server_data_conf/vip_rule.data |
| Server.RouteRuleConf           | String<br>转发规则配置文件路径<br>默认值server_data_conf/route_rule.data |
| Server.ClusterConf             | String<br>后端集群相关配置文件路径<br>默认值server_data_conf/cluster_conf.data |
| Server.GslbConf                | String<br>子集群级别负载均衡配置文件(GSLB)路径<br>默认值cluster_conf/gslb.data |
| Server.ClusterTableConf        | String<br>实例级别负载均衡配置文件路径<br>默认值cluster_conf/cluster_table.data |
| Server.NameConf                | String<br>名字与实例映射表配置文件路径<br>默认值server_data_conf/name_conf.data |
| Server.Modules                 | String<br>启用的模块列表; 启用多个模块请增加多行Modules配置，参见配置示例<br>默认值空 |
| Server.MonitorInterval         | Integer<br>Monitor数据统计周期，单位为秒<br>默认值20 |
| Server.DebugServHttp           | Boolean<br>是否开启反向代理模块调试日志<br>默认值False |
| Server.DebugBfeRoute           | Boolean<br>是否开启流量路由模块调试日志<br>默认值False |
| Server.DebugBal                | Boolean<br>是否开启负载均衡模块调试日志<br>默认值False |
| Server.DebugHealthCheck        | Boolean<br>是否开启健康检查模块调试日志<br>默认值False |


* TLS基础配置

| 配置项        | 描述               |
| ------------ | ------------------- |
| HttpsBasic.ServerCertConf         | String<br>[服务端证书与密钥的配置](tls_conf/server_cert_conf.data.md)文件路径<br>默认值tls_conf/server_cert_conf.data |
| HttpsBasic.TlsRuleConf            | String<br>[TLS协议参数配置](tls_conf/tls_rule_conf.data.md)文件路径<br>默认值tls_conf/tls_rule_conf.data |
| HttpsBasic.CipherSuites           | String<br>启用的加密套件列表; 启用多个套件请增加多行cipherSuites配置，详见示例<br>默认值TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256&#124;TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256<br>TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256&#124;TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256<br>TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256&#124;TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256<br>TLS_ECDHE_RSA_WITH_RC4_128_SHA<br>TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA<br>TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA<br>TLS_RSA_WITH_RC4_128_SHA<br>TLS_RSA_WITH_AES_128_CBC_SHA<br>TLS_RSA_WITH_AES_256_CBC_SHA |
| HttpsBasic.CurvePreferences       | String<br>启用的ECC椭圆曲线，详见示例<br> 默认值CurveP256 |
| HttpsBasic.EnableSslv2ClientHello | Boolean<br>针对SSLv3协议，启用对SSLv2格式ClientHello的兼容<br>默认值True |
| HttpsBasic.ClientCABaseDir        | String<br>客户端根CA证书基目录; 注意：证书文件后缀约定必须是 ".crt"<br> 默认值tls_conf/client_ca |
| SessionCache.SessionCacheDisabled | Boolean<br>是否禁用TLS Session Cache机制<br>默认值False |
| SessionCache.Servers              | String<br>Cache服务的访问地址<br>默认值无 |
| SessionCache.KeyPrefix            | String<br>缓存key前缀<br>默认值bfe |
| SessionCache.ConnectTimeout       | Integer<br>连接Cache服务的超时时间, 单位毫秒<br>默认值50 |
| SessionCache.ReadTimeout          | Integer<br>读取Cache服务的超时时间, 单位毫秒<br>默认值50 |
| SessionCache.WriteTimeout         | Integer<br>写入Cache服务的超时时间, 单位毫秒<br>默认值50 |
| SessionCache.MaxIdle              | Integer<br>与Cache服务的最大空闲长连接数<br>默认值20 |
| SessionCache.SessionExpire        | Integer<br>存储在Cache服务中会话信息的过期时间, 单位秒<br>默认值3600 |
| SessionTicket.SessionTicketsDisabled | Boolean<br>是否禁用TLS Session Ticket<br>默认值True|
| SessionTicket.SessionTicketKeyFile   | String<br>[Session Ticket Key配置](tls_conf/session_ticket_key.data.md)文件路径<br>默认值tls_conf/session_ticket_key.data |



## 反向代理功能的配置
确定基础配置文件bfe.conf后，用户可以按照以下流程，配置BFE的反向代理功能（负载均衡）

### 配置流程
用户可能需要修改多个配置文件。对于这些文件，可以按照以下顺序进行修改：

![](img/conf_flow.png)

如上图所示：
1. 配置域名：配置需要处理的域名，这也用于区分租户/产品线。

2. 配置转发规则：定义消息到后端集群的转发/映射规则，即按什么规则转发消息到某个后端集群。

3. 配置后端集群的属性：设置后端集群的一些参数，比如后端建连的属性，健康检查方式等。

4. 配置后端集群实例：配置后端集群实例和权重信息，包括子集群的权重，子集群中的实例的权重等等。

下面将以添加一个租户example_product为例，详细描述如何为该租户创建相关配置。

### 产品线域名配置 host_rule.data

conf/server_data_conf/host_rule.data是BFE的产品线域名表配置文件。

#### 配置示例

如下示例，host_rule.data的"HostTags"字段定义了租户信息，我们添加了名为"example_product"的租户。租户下可以定义多个域名的tag。字段"Hosts"定义了该tag包含的域名。例子中的tag：exampleTag，包含了一个域名 example.org。
tag由用户自行指定。

```json
{
    "Version": "1",
    "DefaultProduct": null,
    "Hosts": {
        "exampleTag":[
            "example.org"
        ]
    },
    "HostTags": {
        "example_product":[
            "exampleTag"
        ]
    }
}

```
#### 配置项具体描述

| 配置项         | 描述                                   |
| -------------- | -------------------------------------- |
| Version        | String<br> 配置文件版本                |
| DefaultProduct | String<br>默认的产品线名称             |
| Hosts          | Object<br>域名标签和域名列表的映射关系 |
| Hosts{k}       | String<br>域名标签                     |
| Hosts{v}       | Object<br>域名列表                     |
| Hosts{v}[]     | String<br>域名信息                     |
| HostTags       | Object<br>产品线和域名标签的映射关系   |
| HostTags{k}    | String<br>产品线名称                   |
| HostTags{v}    | String<br>域名标签列表                 |
| HostTags{v}[]  | String<br>域名标签                     |


### 转发规则配置 route_rule.data
conf/server_data_conf/route_rule.data 是BFE的分流配置文件。

#### 配置示例
以下示例中，“Cond”使用了条件原语req_host_in()，将请求header的Host域为"example.org"的消息，发送到后端集群cluster_example1。
该集群的具体配置在后面步骤中详述。
```json
{
    "Version": "1",
    "ProductRule": {
        "example_product": [
            {
                "Cond": "req_host_in(\"example.org\")",
                "ClusterName": "cluster_example"
            },
            {
                "Cond": "default_t()",
                "ClusterName": "cluster_default"
            }
        ]
    }
}
```


#### 配置项具体描述

| 配置项                       | 描述                                                       |
| ---------------------------- | ---------------------------------------------------------- |
| Version                      | String<br>配置文件版本                                     |
| ProductRule                  | Object<br>各产品线的分流规则配置                           |
| ProductRule[k]               | String<br>产品线名称                                       |
| ProductRule[v]               | Object<br>分流规则表，包含多条有序分流规则                 |
| ProductRule[v][]             | Object<br>分流规则                                         |
| ProductRule[v][].Cond        | String<br>分流条件, 语法详见[Condition](../../condition/condition_grammar.md) |
| ProductRule[v][].ClusterName | Object<br>目的集群                                         |

### 后端集群配置 cluster_conf.data

conf/server_data_conf/cluster_conf.data 后端集群的配置文件。

#### 配置示例
该示例定义了后端集群“cluster_example”的相关配置。包括：后端基础配置、健康检查配置、GSLB基础配置和集群基础配置。


```json
{
    "Version": "1",
    "Config": {
        "cluster_example": {
            "BackendConf": {
                "TimeoutConnSrv": 2000,
                "TimeoutResponseHeader": 50000,
                "MaxIdleConnsPerHost": 0,
                "RetryLevel": 0
            },
            "CheckConf": {
                "Schem": "http",
                "Uri": "/healthcheck",
                "Host": "example.org",
                "StatusCode": 200,
                "FailNum": 10,
                "CheckInterval": 1000
            },
            "GslbBasic": {
                "CrossRetry": 0,
                "RetryMax": 2,
                "HashConf": {
                    "HashStrategy": 0,
                    "HashHeader": "Cookie:UID",
                    "SessionSticky": false
                }
            },
            "ClusterBasic": {
                "TimeoutReadClient": 30000,
                "TimeoutWriteClient": 60000,
                "TimeoutReadClientAgain": 30000,
            }
        },
        "fcgi_cluster_example": {
            "BackendConf": {
                "Protocol": "fcgi",
                "TimeoutConnSrv": 2000,
                "TimeoutResponseHeader": 50000,
                "MaxIdleConnsPerHost": 0,
                "RetryLevel": 0,
                "FCGIConf": {
                    "Root": "/home/work",
                    "EnvVars": {
                        "VarKey": "VarVal"
                    }    
                }
            },
            "CheckConf": {
                "Schem": "http",
                "Uri": "/healthcheck",
                "Host": "example.org",
                "StatusCode": 200,
                "FailNum": 10,
                "CheckInterval": 1000
            },
            "GslbBasic": {
                "CrossRetry": 0,
                "RetryMax": 2,
                "HashConf": {
                    "HashStrategy": 0,
                    "HashHeader": "Cookie:UID",
                    "SessionSticky": false
                }
            },
            "ClusterBasic": {
                "TimeoutReadClient": 30000,
                "TimeoutWriteClient": 60000,
                "TimeoutReadClientAgain": 30000,
                "ReqWriteBufferSize": 512,
                "ReqFlushInterval": 0,
                "ResFlushInterval": -1,
                "CancelOnClientClose": false
            }
        }
    }
}
```

#### 配置项具体描述

*  配置文件结构

| 配置项     | 描述                           |
| ---------- | ------------------------------ |
| Version    | String<br>配置文件版本         |
| Config     | Object<br>各集群的转发配置参数 |
| Config[k]  | String<br>集群名称             |
| Config[v]  | Object<br>集群转发配置参数     |

Config配置项的具体定义为：

* 后端基础配置

| 配置项    | 描述                |
| ------- | -------------------- |
| BackendConf.Protocol              | String<br>后端服务的协议，当前支持http和fcgi, 默认值http |
| BackendConf.TimeoutConnSrv        | Integer<br>连接后端的超时时间，单位是毫秒<br>默认值2 |
| BackendConf.TimeoutResponseHeader | Integer<br>从后端读响应头的超时时间，单位是毫秒<br>默认值60 |
| BackendConf.MaxIdleConnsPerHost   | Integer<br>BFE实例与每个后端的最大空闲长连接数<br>默认值2 |
| BackendConf.RetryLevel            | Integer<br>请求重试级别。0：连接后端失败时，进行重试；1：连接后端失败、转发GET请求失败时均进行重试<br>默认值0 |
| BackendConf.FCGIConf              | Object<br>FastCGI 协议的配置                              |
| BackendConf.FCGIConf.Root         | String<br>网站的Root文件夹位置                            |
| BackendConf.FCGIConf.EnvVars      | Map[string]string<br>拓展的环境变量                       |

* 健康检查配置

| 配置项                   | 描述                |
| ----------------------- | ------------------ |
| CheckConf.Schem         | String<br>健康检查协议，支持HTTP和TCP<br>默认值 HTTP         |
| CheckConf.Uri           | String<br>健康检查请求URI (仅HTTP)<br>默认值 /health_check   |
| CheckConf.Host          | String<br>健康检查请求HOST (仅HTTP)<br>默认值 ""             |
| CheckConf.StatusCode    | Integer<br>期待返回的响应状态码 (仅HTTP)<br>默认值 0，代表任意状态码 |
| CheckConf.FailNum       | Integer<br>健康检查启动阈值（转发请求连续失败FailNum次后，将后端实例置为不可用状态，并启动健康检查）<br>默认值5 |
| CheckConf.SuccNum       | Integer<br>健康检查成功阈值（健康检查连续成功SuccNum次后，将后端实例置为可用状态）<br>默认值1 |
| CheckConf.CheckTimeout  | Integer<br>健康检查的超时时间，单位是毫秒<br>默认值0（无超时）|
| CheckConf.CheckInterval | Integer<br>健康检查的间隔时间，单位是毫秒<br>默认值1 |

* GSLB基础配置

| 配置项         | 描述              |
| ---------- | ------------------- |
| GslbBasic.CrossRetry             | Integer<br>跨子集群最大重试次数<br>默认值0 |
| GslbBasic.RetryMax               | Integer<br>子集群内最大重试次数<br>默认值2 |
| GslbBasic.BalanceMode            | String<br>负载均衡模式(WRR: 加权轮询; WLC: 加权最小连接数)<br>默认值WRR |
| GslbBasic.HashConf               | Object<br>会话保持的HASH策略配置 |
| GslbBasic.HashConf.HashStrategy  | Integer<br>会话保持的哈希策(ClientIdOnly, ClientIpOnly, ClientIdPreferred)<br>默认值ClientIpOnly |
| GslbBasic.HashConf.HashHeader    | String<br>会话保持的hash请求头 |
| GslbBasic.HashConf.SessionSticky | Boolean<br>是否开启会话保持（开启后，可以保证来源于同一个用户的请求可以发送到同一个后端）<br>默认值False |

* 集群基础配置

| 配置项                  | 描述              |
| ---------------------- | ----------------- |
| ClusterBasic.TimeoutReadClient      | Integer<br>读用户请求wody的超时时间，单位为毫秒<br>默认值30 |
| ClusterBasic.TimeoutWriteClient     | Integer<br>写响应的超时时间，单位为毫秒<br>默认值60 |
| ClusterBasic.TimeoutReadClientAgain | Integer<br>连接闲置超时时间，单位为毫秒<br>默认值60 |


### 配置后端集群实例
该部分配置包括：
* 子集群负载均衡配置 conf/cluster_conf/gslb.data
* 实例负载均衡配置 conf/cluster_conf/cluster_table.data

#### 配置示例

* glsb.data
该文件描述了一个集群所包含的子集群的权重。示例中的集群“cluster_example”只包含了一个子集群"example.bfe.bj"，权重为100。

```json
{
    "Clusters": {
        "cluster_example": {
            "GSLB_BLACKHOLE": 0,
            "example.bfe.bj": 100
        }
    },
    "Hostname": "gslb-sch.example.com",
    "Ts": "20190101000000"
}
```

* cluster_table.data

该文件描述了子集群中的实例的信息。示例的子集群"example.bfe.bj"包含了一个实例"example_hostname"，其IP地址为"192.168.2.1"，该实例的端口为8080，权重为10。

```json
{
    "Config": {
        "cluster_example": {
            "example.bfe.bj": [
                {
                    "Addr": "192.168.2.1",
                    "Name": "example_hostname",
                    "Port": 8080,
                    "Weight": 10
                }
            ]
        }
    }, 
    "Version": "1"
}
```

#### 配置项具体描述

* glsb.data

| 配置项   | 描述                                            |
| -------- | ---------------------------------------------- |
| Hostname | String<br>配置文件生成来源信息                            |
| Ts       | String<br>配置文件生成的时间戳                            |
| Clusters | Object<br>各集群中子集群的分流权重 |
| Clusters{k} | String<br>集群名称 |
| Clusters{v} | Object<br>集群内子集群之间分流权重 |
| Clusters{v}{k} | String<br>子集群名称<br>保留GSLB_BLACKHOLE代表黑洞子集群，分配到该子集群的流量将被丢弃，用于过载保护 |
| Clusters{v}{v} | Integer<br>子集群承接流量的权重<br>子集群承接流量的权重取值范围 0～100,各子集群分流权重之和应等于 100 |

* cluster_table.data

| 配置项  | 描述                            |
| ------- | ----------------------------- |
| Version | String<br>配置文件版本 |
| Config  | Object<br>各集群信息配置 |
| Config{k} | String<br>集群名称 |
| Config{v} | Object<br>集群详细配置信息 |
| Config{v}{k} | String<br>子集群名称 |
| Config{v}{v} | Object<br>子集群配置信息，包含多个实例配置 |


Config配置项的具体定义


| 配置项  | 描述                   |
| ------- | --------------------- |
| Addr    | String<br>实例监听地址 |
| Port    | Integer<br>实例监听端口 |
| Weight  | Integer<br>实例权重 |
| Name    | String<br>实例名称 |


## HTTPS相关配置

### 配置简介
HTTPS配置包括加密套件、证书文件的配置和TLS协议相关参数的配置。

bfe.conf中包含了支持的加密套件，可以添加多项。
```ini
CipherSuites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256|TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
```

另外，bfe.conf中也包含了另外的HTTPS相关的配置文件的的路径。

```ini
ServerCertConf = tls_conf/server_cert_conf.data
TlsRuleConf = tls_conf/tls_rule_conf.data
```

其中tls_conf/server_cert_conf.data用于配置证书。tls_conf/tls_rule_conf.data用于定义TLS配置参数。

### 证书配置server_cert_conf.data
tls_conf/server_cert_conf.data指示了证书文件和私钥文件的位置。一般情况下，可将证书文件放置在tls_conf/certs目录下。

#### 配置示例

```json
{
    "Version": "1",
    "Config": {
        "Default": "example.org",
        "CertConf": {
            "example.org": {
                "ServerCertFile": "tls_conf/certs/server.crt",
                "ServerKeyFile" : "tls_conf/certs/server.key"
            }
        }
    }
}
```

#### 配置项具体描述

| 配置项    | 描述                     |
| -------- | ---------------------- |
| Version  | String<br>配置文件版本                                       |
| Config   | Object<br>证书配置信息                                       |
| Config.Default  | String<br>默认证书名称; 必配选项, 默认证书须包含在证书列表(CertConf)中 |
| Config.CertConf | Object<br>所有证书列表 |
| Config.CertConf{k} | String<br>证书名称; 证书名称禁止命名为"BFE_DEFAULT_CERT" |
| Config.CertConf{v} | Object<br>证书相关文件路径 |
| Config.CertConf{v}.ServerCertFile | String<br>证书文件路径 |
| Config.CertConf{v}.ServerKeyFile | String<br>证书关联密钥文件路径 |
| Config.CertConf{v}.OcspResponseFile | String<br>证书关联OCSP Stple文件路径<br>可选配置 |



### TLS协议配置tls_rule_conf.data

tls_rule_conf.data配置了TLS协议参数。

#### 配置示例

```json
{
    "Version": "1",
    "DefaultNextProtos": ["h2", "http/1.1"],
    "Config": {
        "example_product": {
            "VipConf": [
                "10.199.4.14"
            ],
            "SniConf": null,
            "CertName": "example.org",
            "NextProtos": [
                "h2",
                "http/1.1"
            ],
            "Grade": "B",
            "ClientCAName": ""
        }
    }
}
```


#### 配置项具体描述

| 配置项                 | 描述        |
| ---------------------- | ---------- |
| Version                | String<br>配置文件版本                                        |
| Config                 | Object<br>所有TLS协议配置                                     |
| Config{k}              | String<br>标签                                                |
| Config{v}              | Object<br>TLS协议配置详情                                     |
| Config{v}.CertName     | String<br>服务端证书名称（注：在server_cert_conf.data文件中定义）|
| Config{v}.NextProtos   | Object<br>TLS应用层协议列表<br>默认值为["http/1.1"]               |
| Config{v}.NextProtos[] | String<br>TLS应用层协议, 合法值包括h2, spdy/3.1, http/1.1     |
| Config{v}.Grade        | String<br>TLS安全等级, 合法值包括A+，A，B，C                  |
| Config{v}.ClientAuth   | Bool<br>是否启用TLS双向认证                                   |
| Config{v}.ClientCAName | String<br>客户端证书签发CA名称                                |
| Config{v}.VipConf      | Object<br>VIP列表（注：优先依据VIP来确定TLS配置）             |
| Config{v}.VipConf[]    | String<br>VIP                                                 |
| Config{v}.SniConf      | Object<br>域名列表，可选（注：无法依据VIP确定TLS配置时，使用SNI确定TLS配置）|
| Config{v}.SniConf[]    | String<br>域名                                                |
| DefaultNextProtos      | Object<br>支持的TLS应用层协议列表                             |
| DefaultNextProtos[]    | String<br>TLS应用层协议, 合法值包括h2, spdy/3.1, http/1.1     |



#### 安全等级说明
TLS协议配置文件中包含配置项项“Grade”，该值指定了使用的加密套件的安全级别。

BFE支持多种安全等级（A+/A/B/C）。各安全等级差异在于支持的协议版本及加密套件。A+等级安全性最高、连通性最低；C等级安全性最低、连通性最高。

* 安全等级A+

| 支持协议 | 支持加密套件 |
| -------- | ------------ |
| TLS1.2  | TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256<br>TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_OLD_SHA256<br>TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256<br>TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA<br>TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA<br>TLS_RSA_WITH_AES_128_CBC_SHA<br>TLS_RSA_WITH_AES_256_CBC_SHA |


* 安全等级A

| 支持协议 | 支持加密套件 |
| -------- | ------------ |
| TLS1.2<br>TLS1.1<br>TLS1.0 | TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256<br>TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_OLD_SHA256<br>TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256<br>TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA<br>TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA<br>TLS_RSA_WITH_AES_128_CBC_SHA<br>TLS_RSA_WITH_AES_256_CBC_SHA |

* 安全等级B

| 支持协议 | 支持加密套件 |
| -------- | ------------ |
| TLS1.2<br>TLS1.1<br>TLS1.0 | TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256<br>TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_OLD_SHA256<br>TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256<br>TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA<br>TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA<br>TLS_RSA_WITH_AES_128_CBC_SHA<br>TLS_RSA_WITH_AES_256_CBC_SHA |
| SSLv3 | TLS_ECDHE_RSA_WITH_RC4_128_SHA<br>TLS_RSA_WITH_RC4_128_SHA |


* 安全等级C

| 支持协议 | 支持加密套件 |
| -------- | ------------ |
| TLS1.2<br>TLS1.1<br>TLS1.0 | TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256<br>TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_OLD_SHA256<br>TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256<br>TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA<br>TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA<br>TLS_RSA_WITH_AES_128_CBC_SHA<br>TLS_RSA_WITH_AES_256_CBC_SHA<br>TLS_ECDHE_RSA_WITH_RC4_128_SHA<br>TLS_RSA_WITH_RC4_128_SHA |
| SSLv3 | TLS_ECDHE_RSA_WITH_RC4_128_SHA<br>TLS_RSA_WITH_RC4_128_SHA |


## 配置的重新加载

如前述章节，BFE的配置可大致分为两部分：常规配置和动态配置。常规配置，比如bfe.conf，重新加载需要重启BFE进程，新配置才能生效。

动态配置，可以通过访问监控端口，缺省为8421，来触发配置文件的重新读取，无需重启BFE进程，对服务无影响。具体可参考[配置管理](../design/configuration/configuration.md)。