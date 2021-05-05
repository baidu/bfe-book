# 配置限流
本章介绍如何配置限流功能。该功能将限制特定请求的速率，以对后端服务进行保护。

## 开启限流模块
在conf/bfe.conf中，打开该模块
```ini
Modules = mod_prison
```

## 模块配置
模块配置在目录conf/mod_redirect/中，包含两个文件：

```
# ls
mod_prison.conf	prison.data
```

与其他模块配置相似，mod_prison.conf为模块基础配置文件，指向redirect规则文件。
```ini
[basic]
ProductRulePath = mod_prison/prison.data
```

示例的prison.data如下:

```json
{
	"Version": "1",
	"Config": {
		"example_product": [{
			"Name": "example_prison",
			"Cond": "req_path_prefix_in(\"/prison\", false)",
			"accessSignConf": {
				"url": false,
				"path": false,
				"query": [],
				"header": [],
				"Cookie": [
					"UID"
				]
			},
			"action": {
				"cmd": "CLOSE",
				"params": []
			},
			"checkPeriod": 10,
			"stayPeriod": 10,
			"threshold": 5,
			"accessDictSize": 1000,
			"prisonDictSize": 1000
		}]
	}
}
```
上述示例对路径为“/prison”的请求进行限流，针对cookies中不同的UID，设置上述访问限制。

## 规则配置文件

### 配置格式

| 配置项                   | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| Version                  | String<br>配置文件版本                                       |
| Config                   | Object<br>各产品线的prison规则列表                           |
| Config{k}                | String<br>产品线名称                                         |
| Config{v}                | Array<br>prison规则列表                                      |
| Config{v}[]              | Object<br>单条prison规则                                     |
| Config{v}[].Cond         | String<br>规则条件|
| Config{v}[].AccessSignConf | Object<br>计算请求签名的配置，其中签名被用来确定是否为同类请求 |
| Config{v}[].AccessSignConf.UseClientIP | Boolean<br>计算请求签名时是否使用ClientIP |
| Config{v}[].AccessSignConf.UseUrl | Boolean<br>计算请求签名时是否使用请求的Url |
| Config{v}[].AccessSignConf.UseHost | Boolean<br>计算请求签名时是否使用host |
| Config{v}[].AccessSignConf.UsePath | Boolean<br>计算请求签名时是否使用请求Path |
| Config{v}[].AccessSignConf.UseHeaders | Boolean<br>计算请求签名时是否使用header |
| Config{v}[].AccessSignConf.UrlRegexp | String<br>计算请求签名时使用URL中匹配了UrlRegexp的子串 |
| Config{v}[].AccessSignConf.[]Qeury | Array<br>计算请求签名时使用的query key |
| Config{v}[].AccessSignConf.[]Header | Array<br>计算请求签名时使用的header key |
| Config{v}[].AccessSignConf.[]Cookie | Array<br>计算请求签名时使用的cookie key |
| Config{v}[].Action | Object<br>规则动作 |
| Config{v}[].Action.Cmd | String<br>规则动作名称  |
| Config{v}[].Action.Params | Array<br>规则动作参数列表 |
| Config{v}[].CheckPeriod | Integer<br>检测周期（秒） |
| Config{v}[].StayPeriod | Integer<br>命中规则后的封禁时长 :  惩罚时长（秒） |
| Config{v}[].Threshold | Integer<br>限流阈值 |
| Config{v}[].AccessDictSize | Integer<br>访问统计表大小 |
| Config{v}[].PrisonDictSize | Integer<br>访问封禁表大小 |

### 规则动作
| 动作                      | 描述                               |
| ------------------------- | ---------------------------------- |
| CLOSE                     | 关闭用户连接                     |
| FINISH                    | 回复403响应并关闭用户连接     |
| PASS                      | 正常转发请求 |
| REQ_HEADER_SET            | 修改请求头部                   |

<br>

## 限制特定维度的流量

通过accessSignConf字段，我们可以指定请求统计的维度，判断统计值是否达到限流阈值。
可配置的维度包括：

* UseClientIP

对请求按clientIp进行统计做限流。可以限制每个clientIP的请求速度。

* UseUrl：

对请求按URL进行统计。可以限制对每一个URL的请求速度。

* UseHost

对请求按Host进行统计。可以限制对每一个Host的请求速度。

* UsePath

对请求按Path进行统计。可以限制对每一种Path的请求速度。

* UrlRegexp

对请求的URL做正则匹配，以匹配结果为维度进行统计。可以实现按URL中的部分内容来进行限流。

* header

以请求的特定Header为维度来做统计流量。可以限制该Header所标识的每一种消息的请求速度。

* Cookie

以请求中的特定Cookie为维度进行统计。可以限制该Cookie标识的每一种请求的请求速度。比如，如果我们用Cookie中的UID来标识不同用户，可以通过指定UID，来限制每个用户的访问速度。

* query

以指定的query为维度，统计请求量来做限流。

* UseHeaders

以每个请求中的所有Header为维度来进行统计(合并一个消息的所有header)。限制不同Header的每一种消息的请求速度。

## 设置限流门限

通过下述参数配置限流：

* checkPeriod

设置统计周期，单位秒。

* stayPeriod

被限流后的惩罚时长。在该时间段内，该维度访问请求都将被限制。

* threshold

限制的阈值。一个维度的统计数量达到该阈值将触发限流。

* accessDictSize

访问统计表的大小。统计表保存了当前配置的维度的所有统计值。比如，如果以ClientIp为维度进行的统计，该表包含每个ClientIp的访问量。

* prisonDictSize

访问封禁表的大小。按维度统计后，每类命中限流的请求，都会被保存在封禁表中。所以，封禁表保存了当前所有处于封禁状态的某类请求。比如某频繁访问，被限流的IP地址等。

## 设置限流动作

某类请求命中限流规则后，可以在"action"中配置对该类请求的后续动作。
* CLOSE

直接关闭请求的连接。

* PASS

正常转发，不做任何处理。

* FINISH

响应后关闭连接。

* REQ_HEADER_SET

正常转发，在请求header中插入指定key和value。key和value在params中指定。
