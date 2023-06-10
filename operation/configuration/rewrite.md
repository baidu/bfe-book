# 配置rewrite
本章介绍如何配置HTTP rewrite。该功能对收到的HTTP请求消息进行修改，再转发到后端服务。

## 开启rewrite
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

rewrite.data 包含rewrite规则，可动态加载。安装包中的示例配置文件如下：

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
上述配置为产品线*example_product*中增加了一个规则：对满足条件"Cond"的请求，执行"Actions"动作（包含动作名"Cmd"和对应的参数），如果"Last"为true，停止执行后续动作，否则继续匹配下一条规则。

最终，该规则将修改Path为/rewrite开头的请求，为其增加路径前缀/bfe/，也就是将Path从/rewrite变为/bfe/rewrite。


## 重写动作

"Actions"中的"Cmd"指示了具体的重写动作。

### HOST_SET
设置请求header中的Host值，参数为需设置的值。

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
将Path中的第一段设置为Host的值，并在Path中去掉该段。例如：如果Path为/x.example.com/xxxx，设置Host为 x.example.com，PATH为/xxxx。

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
替换域名中的特定后缀。两个参数分别为被替换的后缀字符串和替换后的字符串。

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
设置Path为指定值，参数为的新Path值。

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
为Path增加前缀，参数为需增加的前缀。

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
删除Path前缀，参数为需删除的前缀。

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

## links
上一章：[第二十二章 配置HTTPS服务](../../operation/configuration/https.md)  
下一章：[第二十四章 配置redirect](../../operation/configuration/redirect.md)