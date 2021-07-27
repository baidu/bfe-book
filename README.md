# 深入理解BFE
本书围绕BFE开源项目，向读者介绍网络接入的相关技术原理，说明BFE开源软件的设计思想，及如何基于BFE开源软件搭建网络接入平台。具有开发能力的读者也可根据本书的说明，按照自己的需要开发BFE的扩展模块，或者向BFE开源项目贡献代码。

本书已经由电子工业出版社正式出版，书名为《[万亿级流量转发 - BFE核心技术与实现](https://segmentfault.com/a/1190000040400268)》。
![book](./book.png)

可通过扫描下方的二维码优惠购买。

![code](./code.png)


## BFE开源项目

BFE是百度统一的七层负载均衡接入转发平台。BFE平台从2012年开始建设，截至2020年底，BFE平台每日转发的请求超过万亿，日峰值请求超过1000万QPS，在业界有巨大影响力。2019年7月，BFE的转发引擎对外开源，并于2020年6月被CNCF（云原生计算基金会）接受为“沙盒项目”（Sandbox Project），这是网络方向中国首个被CNCF接受的开源项目。

BFE开源项目地址: https://github.com/bfenetworks/bfe

## 本书作者

| 姓名   | Github ID                                           |
| ------ | --------------------------------------------------- |
| 章淼   | [mileszhang2016](https://github.com/mileszhang2016) |
| 杨思杰 | [iyangsj](https://github.com/iyangsj)               |
| 戴明   | [daimg](https://github.com/daimg)                   |
| 陶春华 | [ohscartao](https://github.com/ohscartao)           |

## 版权许可

本书采用[署名-非商业性使用-相同方式共享 4.0（CC BY-NC-SA 4.0）](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)许可，发行版权归属于电子工业出版社博文视点，未经授权请勿转载、印刷和发行。

本书著作权归属于BFE开源社区，本书作者对其所编写的内容保留署名权，稿酬将用于BFE开源社区建设。



# 全书目录

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
+ [第十七章 HTTPS优化机制](design/https/https.md)
+ [第十八章 信息的透传](./design/info_pass_through/pass_through.md)

## 操作篇
+ [第十九章 BFE服务的安装部署](./operation/installation/installation.md)
+ [第二十章 BFE服务的基础配置](./operation/configuration/basic.md)
+ [第二十一章 配置负载均衡算法及会话保持](./operation/configuration/proxy.md)
+ [第二十二章 配置HTTPS服务](./operation/configuration/https.md)
+ [第二十三章 配置rewrite](./operation/configuration/rewrite.md)
+ [第二十四章 配置redirect](./operation/configuration/redirect.md)
+ [第二十五章 配置限流](./operation/configuration/prison.md)
+ [第二十六章 支持更多协议](./operation/configuration/protocol.md)

## 实现篇
+ [第二十七章 BFE的代码组织](implementation/source_layout/source_layout.md)
+ [第二十八章 进程模型](implementation/process_model/process_model.md)
+ [第二十九章 请求处理流程及响应](implementation/life_of_a_request/life_of_a_request.md)
+ [第三十章 模块框架](implementation/model_framework/model_framework.md)
+ [第三十一章 请求路由](implementation/routing/routing.md)
+ [第三十二章 负载均衡](implementation/balancing/balancing.md)
+ [第三十三章 核心协议实现](implementation/protocol/protocol.md)

## 开发篇
+ [第三十四章 如何开发BFE扩展模块](./develop/how_to_write_module/how_to_write_module.md)

## 附录篇
+ [附1 BFE的多进程GC机制](./appendix/multi_process_gc/multi_process_gc.md)
