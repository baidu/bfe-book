# BFE的模块插件机制

## 概述

为了便于增加新的功能，BFE定义了一套完整的模块插件机制，支持快速开发新的模块。

BFE模块插件机制的要点如下：

- 在BFE的转发过程中，提供多个回调点
- 对于一个模块，可以针对这些回调点、对应编写回调函数
- 在模块初始化时，把这些回调函数注册到对应的回调点
- 在处理一个连接或请求时，当执行到某个回调点，会顺序执行所有注册的回调函数

下面将简要介绍BFE中的回调点设置，并列出BFE内置的扩展模块。回调框架的详细设计，请参考实现篇中“[回调框架](../../implementation/model_framework/model_framework.md)”中的说明。在“[如何开发BFE扩展模块](../../develop/how_to_write_module/how_to_write_module.md)”一章中，结合一个具体例子对开发BFE扩展模块的方法进行了详细的说明。

## BFE的回调点设置

在BFE中，设置了以下9个回调点：

- HandleAccept: 位于和客户端的TCP连接建立后
- HandleHandshake：位于和客户端的SSL或TLS握手完成后
- HandleBeforeLocation：位于查找产品线之前
- HandleFoundProduct：位于完成查找产品线之后
- HandleAfterLocation：位于完成查找集群之后
- HandleForward：位于完成查找子集群和后端实例之后，位于转发请求之前
- HandleReadResponse：位于读取到后端响应之后
- HandleRequestFinish：位于后端响应处理完毕后
- HandleFinish：位于和客户端的TCP连接关闭后

回调点的定义，可以查看[/bfe_module/bfe_callback.go](https://github.com/bfenetworks/bfe/tree/master/bfe_module/bfe_callback.go)

BFE转发过程中的回调点如下图所示。
![bfe callback](./bfe-callback.png)



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

可以通过访问BFE实例的 http://localhost:8299/monitor/module_handlers 监控地址，查看到当前运行的BFE实例中所有的回调点、在各回调点注册的模块回调函数列表以及顺序。
## links
上一章：[第九章 BFE的内网流量调度机制](../../design/gslb/gslb.md)  
下一章：[第十一章 健康检查机制](../../design/health_check/health_check.md)