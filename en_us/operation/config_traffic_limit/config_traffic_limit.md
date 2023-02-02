# Configure Traffic Limiting
This chapter describes how to configure the traffic limiting function. This function will limit the rate of specific requests to protect backend services.

## Enable Traffic Limiting
In conf/bfe.conf, enable the module

```ini
Modules = mod_prison
```

## Configuration
The configuration is located in the directory conf/mod_prison/, contains two files:

```
# ls
mod_prison.conf	prison.data
```

Similar to other module configurations, mod_prison.conf is the basic configuration file of the module, pointing to the prison rule file.

```ini
[basic]
ProductRulePath = mod_prison/prison.data
```

The example prison.data is as follows:

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
The above example restricts the requests with the path of "/prison". Among them, "accessSignConf" indicates the dimension of traffic limiting. See the following description for details. In this example, the "UID" in cookies will be counted to limit the traffic for different "UIDs".

## Limit Traffic for Specific Dimensions

Through the fields in "accessSignConf", we can specify the dimension to count the request and determine whether the count value reaches the traffic limiting threshold.

Configurable dimensions include:

* UseClientIP: Limit the requests by clientIp. You can limit the request speed of each clientIP.

* UseUrl: Counts requests by URL. You can limit the speed of requests for each URL.

* UseHost: Count requests by Host. You can limit the speed of requests for each host.

* UsePath: Count requests by Path. You can limit the speed of requests for each path.

* UrlRegexp: Match the requested URL with regular expression, and make statistics based on the matching results. You can limit the requests according to part of the content in the URL.

* Header: Use the specific header of the request as the dimension to make statistics on traffic. You can limit the request speed of each requests identified by the header.

* Cookies: Statistics are based on the specific cookies in the request. You can limit the request speed of each request identified by the cookie. For example, if we use the UID in the cookie to identify different users, we can limit the access speed of each user by specifying the UID.

* Query: Take the specified query as the dimension, and count the number of requests to limit the traffic.

* UseHeaders: Use all headers in each request as the dimension for statistics (merge all headers of a request). Limit the request speed of each reqeust of different headers.

## Set Threshold

Configure traffic limiting through the following parameters:

* CheckPeriod: set the statistical period in seconds.

* StayPeriod: the penalty duration after being restricted. During this time period, all requests for this dimension will be restricted.

* Threshold: the threshold of traffic limit. When the statistical quantity of a dimension reaches the threshold, the traffic limit will be triggered.

* AccessDictSize: The size of the access statistics table. The statistics table saves all the statistics of the currently configured dimension. For example, if the ClientIp dimension is used for statistics, the table contains the access statistics of each ClientIp.

* PrisonDictSize: The size of the blocking table. After counted by dimension, each type of requests that hits the limit threshold will be saved in the blocking table. Therefore, the blocking table saves all currently blocked requests of a certain type. For example, an IP address that is frequently accessed and restricted.

## Set Action

After a certain type of request hits the traffic limit rule, you can configure the subsequent actions for this type of request in "action".

* CLOSE: Close the connection directly.

* PASS: Forward normally without any processing.

* FINISH: Close the connection after responding.

* REQ_HEADER_SET: Forward normally, insert the specified key and value in the request header. Key and value are specified in params.
