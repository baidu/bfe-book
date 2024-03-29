# 日志机制

“打印日志”是一般程序经常使用到的机制，看起来很简单。但深入研究，会发现日志也有很多需要注意的细节。下面对BFE开源项目中的日志机制进行说明。

## 日志类型

在BFE中，对不同用途的日志做出明确的区分。

+ 访问日志（Access Log）

  由程序处理外部请求所触发的日志。也包括由于外部建立连接所触发的日志。

+ 服务日志（Server Log）

  由和处理外部请求无关的操作所触发的日志。如：配置加载，程序执行异常（非外部请求处理）等。
  
+ 密钥日志（Key Log）
  由程序处理TLS连接所记录的TLS主密钥日志。

不同日志的使用场景有很大的差异。服务日志主要用于反映程序的运行状态，一般数据量较小，常用于系统监控、故障诊断场景。访问日志的数据量较大，可以用于业务请求的数据分析。密钥日志一般按需抽样启用，并与第三方工具配合解密及诊断密文抓包流量。由于这样的差异，不同日志应该分开输出，并使用不同的机制来做后续的处理。例如，服务日志可以和监控系统进行联动查询，用于故障精确定位；访问日志可以使用大数据分析平台来分析和存储。

在实践中，常见的错误是将访问日志混杂在服务日志中打印。尤其是对很多在处理请求过程中发生的错误，有些程序会把相关的信息打印在服务日志中。这会增大服务日志的数据量，使严重的系统错误信息被淹没在访问信息中。这样的问题应在程序编写中尽量避免。

## 日志打印的注意事项

对一般的程序来说，日志的输出应满足以下要求：

+ 日志的输出不能阻塞程序正常的处理流程

  日志的输出涉及到磁盘IO操作，在部分场景可能出现阻塞的情况。有部分程序由于实现不当，业务主逻辑对日志输出模块为“同步调用”，在日志打印阻塞时，主逻辑也被阻塞。

  正确的方法是，将业务主逻辑对日志模块的调用修改为“异步方式”。在最坏情况下，磁盘IO操作阻塞只会导致日志打印失败，而不会阻塞业务主逻辑的执行。

+ 应支持日志文件的切割和滚动覆盖

  日志文件的大小会随着程序的运行而持续增长，如果不做任何处理，会将磁盘空间用尽，从而导致系统问题。很多打印日志的基础库都支持对日志文件做定期的切割，并可以指定日志文件滚动覆盖的参数。在切割的日志文件数量超过指定的数量后，会删除历史的日志文件。

  一些程序未使用“内置”的日志基础库来实现切割和滚动覆盖的功能，而采用在crontab中定期运行脚本的方式来实现。这样的方式虽然也可以达到类似的效果，但是提高了运维的复杂性。时常出现由于遗忘增加crontab配置、或错误修改crontab配置导致的日志超限问题。建议尽量使用“内置”的日志基础库来实现以上功能。

BFE的服务日志使用[https://github.com/baidu/go-lib](https://github.com/baidu/go-lib)下的log库来输出。log库在开源的log4go基础上进行了封装和修改，增加了“按照时间切割日志”的功能。在log4go中，实现了“异步写入”的机制，日志信息首先被提交到一个队列，然后由独立的go routine读出并输出。如果队列超限，则日志信息会被丢弃。

## BFE的访问日志

BFE的访问日志由BFE的扩展模块mod_access来输出。BFE的访问日志中包括“请求”（Request）和“会话”（Session）两类日志。会话对应于客户端和BFE间建立的TCP连接，一个会话中可能包含多个请求。

mod_access提供了模板配置能力，用户可以对请求日志和会话日志中输出的数据字段做定制。


## links
上一章：[第十三章 监控机制](../../design/monitor/monitor.md)  
下一章：[第十五章 超时设置](../../design/timeout/timeout.md)