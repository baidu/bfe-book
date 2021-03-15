#  进程模型

BFE系统的转发实例由一组协同工作的进程组成（详见BFE简介章节）。这里以其中最核心的转发进程为重点介绍。

BFE的转发进程是由golang语言编写、基于golang协程实现的高并发程序。BFE的转发进程中包含了几类重要的协程:

![process_model](./process_model.png)

- **网络相关协程**：
   - 用户连接的监听协程、用户连接的处理协程, 以及协议有关的可选的用户请求处理协程。
   - 后端连接的建立协程、后端连接的读写协程

- **管理相关协程**: 
   - 针对后端健康状态的检查协程
   - 监控及热加载请求的处理协程

- **辅助相关协程**:
   - 扩展模块也可能创建协程，用于后台定期操作或异步执行处理
   - 例如：定期进行日志切割、异步更新缓存等

## 异常恢复

由于转发程序的复杂性及快速迭代，潜在PANIC问题可能难以通过线下测试完全被发现，但在一些情况下触发后会对转发集群稳定性带来非常严重的影响(例如Bug由特定类型请求触发）。因此，BFE所有网络相关协程都利用了golang内置的PANIC恢复机制，可避免在请求处理过程中由于未知Bug导致程序PANIC退出。

```
	// bfe_server/http_conn.go:serve()
	
	defer func() {
		if err := recover(); err != nil {
			log.Logger.Warn("panic: conn.serve(): %v, readTotal=%d,writeTotal=%d,reqNum=%d,%v\n%s",
				c.remoteAddr, c.session.ReadTotal, c.session.WriteTotal, c.session.ReqNum,
				err, gotrack.CurrentStackTrace(0))
			  ...
		}
		...
	}()
```

在出现PANIC后，往往仅影响单个连接或请求。同时PANIC恢复阶段输出的上下文日志，对于一些难以在线下复现的问题，基于PANIC日志也可高效分析定位出问题根因。



## 并发能力

BFE协程可以充分利用单机的多核CPU提升并发量及吞吐。但基于golang协程的并发模型也存在局限：
- 难以绑定到固定的CPU核以利用CPU亲缘性提升极限性能

- 并发能力与CPU核数并非呈线性增长(由于锁竞争)，当核数非常大时继续增加核数可能并不能提升极限性能。

  在实践中，一般通过针对流量转发场景定制最佳机型或制定合适的容器规格，来避免这个问题。 
