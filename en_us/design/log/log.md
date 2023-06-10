# Logging Mechanism

"Print Log" is a mechanism that is often used by programs, which seems very simple. However, after in-depth study, you will find that there are many details that need attention in the usage of log. The following describes the logging mechanism in the BFE open source project.

## Types of Logging

In BFE, we make a clear distinction between logs for different purposes.

+ Access Log

  The log triggered by the BFE program processing requests. It also includes logs triggered by connection establishment.

+ Server Log

  Logs triggered by operations unrelated to processing requests. For example: configuration loading, program exception (non request processing), etc.

+ Key Log

  The TLS master key log is recorded for the TLS connections processed by BFE.

The scenarios of different logs vary greatly. The server log is mainly used to reflect the running status of the program. Generally, it has a small amount of data and is often used in system monitoring and fault diagnosis scenarios. The access log has a large amount of data and can be used for data analysis of requests. The key log is generally enabled by sampling on demand, and works with third-party tools to decrypt and diagnose the encrypted packet capture traffic. Due to such differences, different logs should be output separately and different mechanisms should be used for subsequent processing. For example, the server log can be linked with the monitoring system for precise fault location; Access logs can be analyzed and stored using the big data analysis platform.

In practice, the common mistake is to mix the access log with the server log to print. Especially for many errors that occur during the processing of requests, some programs will print relevant information in the server log. This will increase the amount of data in the server log, so that serious system error information will be submerged in the access information. Such problems should be avoided in programming.

## Precautions for Log Printing

For normal programs, the output of the log should meet the following requirements:

+ The output of the log should not block the normal processing of the program

  The output of the log involves disk IO operations. Blocking may occur in some scenarios. For some programs, due to improper implementation, the  main logic calls the logging module synchronously. When the log printing is blocked, the main logic is also blocked.

  The correct method is to change the call of the main logic to the log module to "asynchronous mode". In the worst case, disk IO blocking will only cause failure of log printing , but will not block the main logic.

+ It should support cutting and rolling overwriting of log files

  The size of the log file will continue to grow as the program runs. If no processing is done, the disk space will be exhausted, resulting in system problems. Many libraries for printing logs support periodical cutting of log files and can specify parameters for log file rollover. When the number of log files exceeds the specified number, the historical log files will be deleted.

  Some programs do not use the "built-in" log library to realize the functions of cutting and rolling overwriting, but use the way of running scripts periodically by crontab. Although this method can achieve similar results, it increases the complexity of operation and maintenance. Log overloaded problems often occur due to forgetting to add the crontab configuration or incorrectly modifying the crontab configuration. It is recommended to use the "built-in" log library as much as possible.

BFE uses log library in [https://github.com/baidu/go-lib](https://github.com/baidu/go-lib) to output server log. The log library is encapsulated and modified on the basis of the open source log4go, adding the function of "cutting logs according to time". In log4go, the "asynchronous write" mechanism is implemented. Log information is first submitted to a queue, and then read and output by an independent go routine. If the queue exceeds the limit, the log information will be discarded.

## Access Log of BFE

Access log of BFE is outputed by extension module mod_access. BFE's access logs include "Request" and "Session" logs. The session corresponds to the TCP connection established between the client and BFE. A session may contain multiple requests.

mod_access provides template configuration capabilities, and users can customize the data fields output in the request log and session log.

## links
Previous: [Chap13 Status Monitoring](../../../en_us/design/monitor/monitor.md)  
Next: [Chap15 Timeout Setting](../../../en_us/design/timeout/timeout.md)