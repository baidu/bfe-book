# 重写（rewrite)的配置
本章介绍如何配置HTTP URL重写（rewrite）。该功能对收到的HTTP请求消息的URL进行修改，再转发到后端服务。

## 开启重写
在conf/bfe.conf中，打开该模块
```ini
Modules = mod_write
```

## 模块配置
模块配置在目录conf/mod_rewrite/中，包含两个文件:
```
$ ls
mod_rewrite.conf	rewrite.data
```

mod_rewrite.conf为模块基础配置文件，指向rewrite规则文件，通常无需修改。

```
$ cat mod_rewrite.conf 
[basic]
DataPath = mod_rewrite/rewrite.data
```

rewrite.data 包含rewrite规则，可动态加载。安装包中的示例中的配置文件如下：

```
$ cat rewrite.data 
{
    "Version": "1",
    "Config": {
        "example_product": [
            {
                "Cond": "req_path_prefix_in(\"/rewrite\", false)",
                "Actions": [
                    {
                        "Cmd": "PATH_PREFIX_ADD",
                        "Params": [
                            "/bfe/"
                        ]
                    }
                ],
                "Last": true
            }
        ]
    }
}

```
其为产品线*example_product*中增加了一个规则：
1. 对满足条件"Cond"的请求，
2. 执行"Actions"动作，包含动作名"Cmd"和对应的参数，
3. 如果"Last"为true，停止执行后续动作，否则继续匹配下一条规则。

最终效果，该规则修改了请求中以/rewrite开头的Path，增加一段路径/bfe/。Path从/rewrite变为/bfe/rewrite。

## 规则配置文件
### 配置格式
规则配置文件rewrite.data的定义如下：

| 配置项                   | 描述                  |
| ------------------------ | --------------------- |
| Version                  | String<br>配置文件版本           |
| Config                   | Object<br>各产品线的重写规则列表    |
| Config{k}                | String<br>产品线名称           |
| Config{v}                | Object<br>重写规则列表        |
| Config{v}[]              | Object<br>重写规则          |
| Config{v}[].Cond         | String<br>规则条件 |
| Config{v}[].Action       | Object<br>规则动作          | 
| Config{v}[].Action.Cmd   | Object<br>规则动作名称          | 
| Config{v}[].Action.Param | Object<br>规则动作参数列表      | 
| Config{v}[].Last         | Boolean<br>当该项为true时，命中某条规则后，不再向后匹配 |

### 规则动作
规则文件中的"Cmd"支持下述值，具体含义如下：

| 动作                      | 描述                               |
| ------------------------- | ---------------------------------- |
| HOST_SET                  | 设置host                           |
| HOST_SUFFIX_REPLACE       | 替换域名后缀                           |
| HOST_SET_FROM_PATH_PREFIX | 根据path前缀设置host               |
| PATH_SET                  | 设置path                           |
| PATH_PREFIX_ADD           | 增加path前缀                       |
| PATH_PREFIX_TRIM          | 删除path前缀                       |
| QUERY_ADD                 | 增加query                          |
| QUERY_DEL                 | 删除query                          |
| QUERY_RENAME              | 重命名query                        |
| QUERY_DEL_ALL_EXCEPT      | 删除除指定key外的所有query         |

## 重写动作的详细描述

### HOST_SET
设置请求的header中的Host字段。

示例：
```json    
{
    "Cmd": "HOST_SET", 
    "Params": ["www.example.com"]
}
```

结果：
| | |
|-|-|
|原始值   | http://abc.example.com |
|修改后   | http://www.example.com |

<br />

### HOST_SET_FROM_PATH_PREFIX
根据path前缀设置host：如果Path为/x.example.com/xxxx，设置Host为 x.example.com，uri为xxxx。

示例：
```json  
{
    "cmd": "HOST_SET_FROM_PATH_PREFIX", 
    "params": []
}
```

结果：
| | |
|-|-|
|原始值   | http://www.example.com/test.example.com/xxxx |
|修改后   | http://test.example.com/xxxx |

<br />


### HOST_SUFFIX_REPLACE
替换域名后缀。指定两个参数，分别为被替换的字符串和替换后的字符串。

示例：
```json  
{
    "cmd": "HOST_SUFFIX_REPLACE", 
    "params": ["net", "com"]
}
```

结果：
| | |
|-|-|
|原始值   | http://www.example.net |
|修改后   | http://www.example.com |

<br />

### PATH_SET
设置Path，参数指定新的Path。

示例：
```json  
{
    "cmd": "PATH_SET", 
    "params": ["/index"]
}
```

结果：
| | |
|-|-|
|原始值   | http://www.example.com/current |
|修改后   | http://www.example.com/index |

<br />

### PATH_PREFIX_ADD
为path增加前缀，参数指定需增加的前缀。

示例：
```json  
{
    "cmd": "PATH_PREFIX_ADD", 
    "params": ["/index"]
}
```

结果：
| | |
|-|-|
|原始值   | http://www.example.com/current |
|修改后   | http://www.example.com/index/current |

<br />

### PATH_PREFIX_TRIM
删除Path前缀，参数指定需删除的前缀。

示例：
```json  
{
    "cmd": "PATH_PREFIX_TRIM", 
    "params": ["/service"]
}
```

结果：
| | |
|-|-|
|原始值   | http://www.example.com/service/index.html |
|修改后   | http://www.example.com/index.html |

<br />

### QUERY_ADD
增加Query，参数指定需增加的query的key和value。

示例：
```json  
{
    "cmd": "QUERY_ADD", 
    "params": ["name", "alice"]
}
```

结果：
| | |
|-|-|
|原始值   | http://www.example.com/ |
|修改后   | http://www.example.com/?name=alice |

<br />


### QUERY_RENAME
对Query重命名，参数指定key的原名字和新名字。

示例：
```json  
{
    "cmd": "QUERY_ADD", 
    "params": ["name", "user"]
}
```

结果：
| | |
|-|-|
|原始值   | http://www.example.com/?name=alice |
|修改后   | http://www.example.com/?user=alice |

<br />

### QUERY_DEL
删除指定Query，参数指定key的名字。

示例：
```json  
{
    "cmd": "QUERY_ADD", 
    "params": ["name"]
}
```

结果：
| | |
|-|-|
|原始值   | http://www.example.com/?name=alice |
|修改后   | http://www.example.com/ |

<br />

### QUERY_DEL_ALL_EXCEPT
删除Query中指定key外所有其他key。

示例：
```json  
{
    "cmd": "QUERY_DEL_ALL_EXCEPT", 
    "params": ["name"]
}
```

结果：
| | |
|-|-|
|原始值   | http://www.example.com/?name=alice?key1=value2&key2=value2 |
|修改后   | http://www.example.com/?name=alice |

<br />
