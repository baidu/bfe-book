# 《深入理解BFE》（《BFE Inside》）
本书围绕BFE开源项目，向读者介绍网络接入的相关技术原理，说明BFE开源软件的设计思想，及如何基于BFE开源软件搭建网络接入平台。具有开发能力的读者也可根据本书的说明，按照自己的需要开发BFE的扩展模块，或者向BFE开源项目贡献代码。

+ 主编：章淼，杨思杰，戴明
+ BFE开源项目内部wiki: http://wiki.baidu.com/pages/viewpage.action?pageId=1065820757

## 背景篇
+ [第一章 BFE简介](./background/what-is-bfe.md)

## 原理篇
+ [第二章 网络前端接入技术简介](./frontend_principle/introduction/introduction.md)
+ [第三章 网络前端接入技术的发展趋势](./frontend_principle/trend/trend.md)
+ [第四章 网络负载均衡技术简介](./frontend_principle/load_balance/load_balance.md)

## 设计篇
+ [第五章 BFE的设计思想](./design/ideas/ideas.md)
+ [第六章 BFE和相关开源项目的对比](./design/comparison/comparison.md)
+ [第七章 BFE的转发模型](./design/model/model.md)
+ [第八章 BFE的路由转发机制](./design/route/route.md)
+ [第九章 BFE的内网流量调度机制](./design/gslb/gslb.md)
+ [第十章 BFE的模块插件机制](./design/module/module.md)
+ [第十一章 健康检查机制](./design/health_check/health_check.md)
+ [第十二章 限流机制](./design/limit/limit.md)
+ [第十三章 监控机制](./design/monitor/monitor.md)
+ [第十四章 日志机制](./design/log/log.md)
+ [第十五章 超时设置](./design/timeout/timeout.md)
+ [第十六章 配置管理](./design/configuration/configuration.md)
+ 第十七章 TLS优化机制
+ [第十八章 信息的透传](./design/info_pass_through/pass_through.md)

## 操作篇
+ [第十九章 BFE服务的安装部署](./operation/installation/installation.md)
+ [第二十章 BFE服务的基础配置](./operation/configuration/config_basic.md)
+ [第二十一章 配置负载均衡算法及会话保持](./operation/configuration/config_proxy.md)
+ [第二十二章 配置HTTPS服务](./operation/configuration/config_https.md)

## 实现篇
+ 第二十三 源代码结构概述
+ 第二十四章 进程模型
+ 第二十五章 请求处理与响应
+ 第二十六章 模块框架
+ 第二十七章 请求路由
+ 第二十八章 负载均衡
+ 第二十九章 协议实现

## 开发篇
+ [第二十九章 如何开发BFE扩展模块](./develop/how_to_write_module/how_to_write_module.md)

## 附录篇
+ [附1 BFE的多进程GC机制](./appendix/multi_process_gc/multi_process_gc.md)
