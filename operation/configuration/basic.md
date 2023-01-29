# BFE服务的基础配置
上一章对如何下载运行BFE进行了介绍，用安装包中的缺省配置运行了BFE程序。

本章将对BFE的配置进行进一步介绍，通过一个简单的负载均衡的例子，说明如何基于BFE配置一个基础的负载均衡系统。

## 场景说明

对于负载均衡，基本能力就是将从客户端的请求，转发到一组后端服务实例上去。BFE的概念中，相同功能的后端服务定义为一个集群。

下面我们通过一个简单的例子，展示如何配置BFE转发到后端集群上。

现在有一个域名为“example.org”网站，后端有三台服务器，instance-1,instance-2,instance-3。需要通过BFE将HTTP请求转发到后端三台服务器上。

如下示意图：

![scenario](./img/scenario.png)

在后续配置中，三个实例将定义为在同一个逻辑集群clusterA和逻辑子集群subCluster1中：

![cluster vs instance](./img/cluster_instance.png)



## 修改基础配置文件*bfe.conf*

*bfe.conf*包含了BFE的基础配置。读者了解配置文件，可以从这个文件入手。该配置文件包含大量配置选项，具体含义将在后续进行描述。

在bfe.conf中可以查看缺省端口配置，用户可以根据需要，修改监听的端口。。

```ini
[Server]
# listen port for http request
HttpPort = 8080
# listen port for https request
HttpsPort = 8443
# listen port for monitor request
MonitorPort = 8421
```

## 转发配置流程

确定基础配置文件bfe.conf后，用户可以设置转发相关的配置。

![](img/conf_flow.png)

转发配置的推荐流程如上图所示，主要步骤为：
1. 配置域名：配置需要处理的域名，这也用于区分租户/产品线。
2. 配置转发规则：定义消息到后端集群的转发/映射规则，即按什么规则转发消息到某个后端集群。
3. 配置后端集群的属性：设置后端集群的一些参数，比如后端建连的属性，健康检查方式等。
4. 配置后端集群实例：配置后端集群实例和权重信息，包括子集群的权重，子集群中的实例的权重等等。

## 案例说明

下面将以添加一个租户example_product为例，详细描述如何为该租户创建相关配置。

### 产品线域名配置 host_rule.data

conf/server_data_conf/host_rule.data是BFE的产品线域名表配置文件。

host_rule.data中的"HostTags"字段定义了租户信息，我们添加一个名为"example_product"的租户。租户下可以定义多个域名的tag。字段"Hosts"定义了该tag包含的域名。例子中的tag：exampleTag，包含了需要支持的域名 example.org。文件中的tag由用户自行指定。

该配置文件的示例如下：
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

### 转发规则配置 route_rule.data
conf/server_data_conf/route_rule.data 是BFE的分流配置文件。

在上述例子中，将定义一个后端集群cluster_A，它将代表后端服务的集群。

以下示例中，“Cond”使用了条件原语req_host_in()，将请求header的Host域为"example.org"的消息，发送到后端集群cluster_A。

```json
{
    "Version": "1",
    "ProductRule": {
        "example_product": [
            {
                "Cond": "req_host_in(\"example.org\")",
                "ClusterName": "cluster_A"
            },
            {
                "Cond": "default_t()",
                "ClusterName": "cluster_default"
            }
        ]
    }
}
```

### 后端集群配置 cluster_conf.data

conf/server_data_conf/cluster_conf.data 为后端集群的配置文件。包含了后端集群“cluster_A”的相关配置。包括：后端基础配置、健康检查配置、GSLB基础配置和集群基础配置。

上述的例子，可以使用以下配置：

```json
{
    "Version": "1",
    "Config": {
        "cluster_A": {
            "BackendConf": {
                "TimeoutConnSrv": 2000,
                "TimeoutResponseHeader": 50000,
                "MaxIdleConnsPerHost": 0,
                "RetryLevel": 0
            },
            "CheckConf": {
                "Schem": "http",
                "Uri": "/",
                "Host": "example.org",
                "StatusCode": 200,
                "FailNum": 10,
                "CheckInterval": 1000
            },
            "GslbBasic": {
                "CrossRetry": 0,
                "RetryMax": 2
            },
            "ClusterBasic": {
                "TimeoutReadClient": 30000,
                "TimeoutWriteClient": 60000,
                "TimeoutReadClientAgain": 30000,
            }
        }
    }
}
```

cluster_conf.data中的配置项较多，各个项的具体描述如下：

* 后端基础配置BackendConf 


| 配置项            | 描述                           |
| --------------------- | ----------------------- |
| BackendConf.TimeoutConnSrv        | Integer<br>连接后端的超时时间，单位是毫秒<br>默认值2000 |
| BackendConf.TimeoutResponseHeader | Integer<br>从后端读响应头的超时时间，单位是毫秒<br>默认值60000 |
| BackendConf.MaxIdleConnsPerHost   | Integer<br>BFE实例与每个后端的最大空闲长连接数<br>默认值2 |
| BackendConf.MaxConnsPerHost   | Integer<br>BFE实例与每个后端的最大长连接数，0代表无限制<br>默认值0 |
| BackendConf.RetryLevel            | Integer<br>请求重试级别。0：连接后端失败时，进行重试；1：连接后端失败、转发GET请求失败时均进行重试<br>默认值0 |

<br>
* 健康检查配置CheckConf

| 配置项                   | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| CheckConf.Schem         | String<br>健康检查协议，支持HTTP和TCP<br>默认值 HTTP         |
| CheckConf.Uri           | String<br>健康检查请求URI (仅HTTP)<br>默认值 /health_check   |
| CheckConf.Host          | String<br>健康检查请求HOST (仅HTTP)<br>默认值 ""             |
| CheckConf.StatusCode    | Integer<br>期待返回的响应状态码 (仅HTTP)<br>默认值 0，代表任意状态码 |
| CheckConf.FailNum       | Integer<br>健康检查启动阈值（转发请求连续失败FailNum次后，将后端实例置为不可用状态，并启动健康检查）<br>默认值5 |
| CheckConf.SuccNum       | Integer<br>健康检查成功阈值（健康检查连续成功SuccNum次后，将后端实例置为可用状态）<br>默认值1 |
| CheckConf.CheckTimeout  | Integer<br>健康检查的超时时间，单位是毫秒<br>默认值0（无超时）|
| CheckConf.CheckInterval | Integer<br>健康检查的间隔时间，单位是毫秒<br>默认值1000 |

<br>

* GSLB基础配置GslbBasic

| 配置项                           | 描述                                       |
| -------------------------------- | ------------------------------------------ |
| GslbBasic.CrossRetry             | Integer<br>跨子集群最大重试次数<br>默认值0 |
| GslbBasic.RetryMax               | Integer<br>子集群内最大重试次数<br>默认值2 |

<br>

### 配置后端集群实例
该部分配置包括：
* 子集群负载均衡配置 conf/cluster_conf/gslb.data
* 实例负载均衡配置 conf/cluster_conf/cluster_table.data

#### 

**glsb.data**
该文件描述了一个集群所包含的子集群的权重。在上述例子中，我们只需要为集群“cluster_A”定义一个子集群"subCluster1"，权重为100。

```json
{
    "Clusters": {
        "cluster_A": {
            "GSLB_BLACKHOLE": 0,
            "subCluster1": 100
        }
    },
    "Hostname": "",
    "Ts": "0"
}
```

**cluster_table.data**

该文件描述了子集群中的实例的信息。示例的子集群"subCluster1"将包含后端的三个实例instance-1、instance-2、instance-3。同时指定了每个实例的IP地址。

```json
{
    "Config": {
        "cluster_A": {
            "subCluster1": [
                {
                    "Addr": "192.168.2.1",
                    "Name": "instance-1",
                    "Port": 8080,
                    "Weight": 10
                },
                {
                    "Addr": "192.168.2.2",
                    "Name": "instance-2",
                    "Port": 8080,
                    "Weight": 10
                },
                {
                    "Addr": "192.168.2.3",
                    "Name": "instance-3",
                    "Port": 8080,
                    "Weight": 10
                }
            ]
        }
    }, 
    "Version": "1"
}
```
完成上述配置后，重新启动bfe实例，让配置生效。


## 服务访问验证
在客户端使用curl命令，访问bfe的地址和端口，可以看到看到请求被转发到后端服务器实例。

    > 注意：curl命令需要通过参数 -H “Host: example.org”, 指定请求的header中的Host字段，否则会收到500错误。 


## 配置的重新加载

上述配置修改后，也可以通过动态方式重新加载配置。

BFE的配置可大致分为两部分：常规配置和动态配置。常规配置，比如bfe.conf，重新加载需要重启BFE进程，新配置才能生效。

动态配置，可以通过访问监控端口，缺省为8421，来触发配置文件的重新读取，无需重启BFE进程，对服务无影响。具体可参考[配置管理](../../design/configuration/configuration.md)。
