# In-depth Understanding of BFE
English | [中文](./README-CN.md)

This book focuses on BFE open source project, introduces the relevant technical principles of network access, explains the design idea of BFE open source software, and how to build a network front-end platform based on BFE open source software. Readers with development capabilities can also develop BFE extension modules according to their own needs or contribute code to BFE open source projects according to the instructions in this book.


## BFE Open Source Project

BFE is Baidu's unified layer-7 load balancing platform. The BFE platform has been under construction since 2012. By the end of 2020, the BFE platform has forwarded more than trillions of requests per day, with a daily peak of more than 10 million QPS, which has great influence in the industry. In July 2019, BFE's forwarding engine was open-sourced, and was accepted by CNCF (Cloud Native Computing Foundation) as the "Sandbox Project" in June 2020.

Address of BFE open source project: https://github.com/bfenetworks/bfe

## Authors

| 姓名        | Github ID                                           |
| ----------- | --------------------------------------------------- |
| Miles Zhang | [mileszhang2016](https://github.com/mileszhang2016) |
| Sijie Yang  | [iyangsj](https://github.com/iyangsj)               |
| Ming Dai    | [daimg](https://github.com/daimg)                   |
| Chunhua Tao | [ohscartao](https://github.com/ohscartao)           |

## Copyright Statement

This book adopts [Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/) Permission. The distribution copyright belongs to [Publishing House of Electronics Industry](https://www.phei.com.cn/). Do not reprint, print and distribute without authorization.

The copyright of this book belongs to the BFE open source community. The author of this book reserves the right of authorship for the content he wrote, and the remuneration will be used for the construction of the BFE open source community.

# Table of contents

## Background

+ [Chap1 Introduction to BFE](./en_us/background/what-is-bfe.md)

## Principles
+ [Chap2 Introduction to Network Front End](./en_us/frontend_principle/introduction/introduction.md)
+ [Chap3 Trend of Network Front End Technology](./en_us/frontend_principle/trend/trend.md)
+ [Chap4 Introduction to Network Load Balancing Technology](./en_us/frontend_principle/load_balance/load_balance.md)

## Designs
+ [Chap5 Design Considerations of BFE](./en_us/design/ideas/ideas.md)
+ [Chap6 Comparison to Similar Systems](./en_us/design/comparison/comparison.md)
+ [Chap7 Forwarding Model of BFE](./en_us/design/model/model.md)
+ [Chap8 Traffic Routing](./en_us/design/route/route.md)
+ [Chap9 Traffic Scheduling](./en_us/design/gslb/gslb.md)
+ [Chap10 Plugin Architecture](./en_us/design/module/module.md)
+ [Chap11 Health Check](./en_us/design/health_check/health_check.md)
+ [Chap12 Traffic Limiting](./en_us/design/limit/limit.md)
+ [Chap13 Status Monitoring](./en_us/design/monitor/monitor.md)
+ [Chap14 Logging Mechanism](./en_us/design/log/log.md)
+ [Chap15 Timeout Setting](./en_us/design/timeout/timeout.md)
+ [Chap16 Configuration Management](./en_us/design/configuration/configuration.md)
+ [Chap17 HTTPS Optimization](./en_us/design/https/https.md)
+ [Chap18 Information Passthrough](./en_us/design/info_pass_through/pass_through.md)

## Operations
+ [Chap19 Installation And Deployment of BFE Service](./en_us/operation/installation/installation.md)
+ [Chap20 Basic Configuration of BFE Service](./en_us/operation/config_basic/basic.md)
+ [Chap21 Configure Load Balancing Algorithm And Session Stickiness](./en_us/operation/config_scheduling/config_scheduling.md)
+ [Chap22 Configure HTTPS Service](./en_us/operation/config_https/config_https.md)
+ [Chap23 Configure Rewrite](./en_us/operation/config_rewrite/config_rewrite.md)
+ [Chap24 Configure Redirect](./en_us/operation/config_redirect/config_redirect.md)
+ [Chap25 Configure Traffic Limiting](./en_us/operation/config_traffic_limit/config_traffic_limit.md)
+ [Chap26 Support More Protocols](./en_us/operation/config_protocols/config_protocols.md)

## Implementations
+ [Chap27 Layout of BFE Code Base](./en_us/implementation/source_layout/source_layout.md)
+ [Chap28 Process Model](./en_us/implementation/process_model/process_model.md)
+ [Chap29 Processing of Connections and Requests](./en_us/implementation/life_of_a_request/life_of_a_request.md)
+ [Chap30 Module Framework](./en_us/implementation/module_framework/module_framework.md)
+ [Chap31 Request Routing](./en_us/implementation/routing/routing.md)
+ [Chap32 Load Balancing](./en_us/implementation/balancing/balancing.md)
+ [Chap33 Implementation of Core Protocols](./en_us/implementation/protocol/protocol.md)

## Development
+ [Chap 34 How to Develop BFE Extension Module](./en_us/develop/how_to_write_module/how_to_write_module.md)

## Appendix
+ [Appendix1 Multi-process GC mechanism of BFE](./en_us/appendix/multi_process_gc/multi_process_gc.md)
