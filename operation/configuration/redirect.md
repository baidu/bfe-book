## 配置redirect
本章介绍如何配置HTTP重定向（redirect）。该功能将对收到的请求返回重定向响应码，指示客户端跳转到新的URL。

## 开启redirect
在conf/bfe.conf中，打开该模块
```ini
Modules = mod_redirect
```

## 模块配置
模块配置在目录conf/mod_redirect/中，包含两个文件：

```
$ ls
mod_redirect.conf	redirect.data
```

mod_redirect.conf为模块基础配置文件，指向重定向规则文件，通常无需修改。
```
$ cat mod_redirect.conf 
[basic]
DataPath = mod_redirect/redirect.data
```

redirect.data 包含重定向规则，可动态加载。安装包中的示例中的配置文件如下：

```
{
    "Version": "1",
    "Config": {
        "example_product": [
            {
                "Cond": "req_path_prefix_in(\"/redirect\", false)",
                "Actions": [
                    {
                        "Cmd": "URL_SET",
                        "Params": ["https://example.org"]
                    }
                ],
                "Status": 301
            }
        ]
    }
}

```
上述配置为产品线*example_product*中增加了一个规则：对满足条件"Cond"（请求路径的前缀为“/redirect”）的请求，执行"Actions"动作（重定向到https://example.org），返回HTTP响应码为301。

## 重定向动作
"Actions"中的"Cmd"指示了如何设置重定向中的URL。

### URL_SET
重定向请求到指定URL，参数为重定向的URL。

示例：
```json    
{
    "Cmd": "URL_SET", 
    "Params": ["http://www.example.com/more"]
}
```

结果：
| | |
|-|-|
|请求     | http://www.example.com/unknown |
|重定向到  | http://www.example.com/more |

<br />

### URL_FROM_QUERY
将query中的某个key的值设置为重定向地址。参数为query中该key的名字。

示例：
```json    
{
    "Cmd": "URL_FROM_QUERY", 
    "Params": ["url"]
}
```

结果：
| | |
|-|-|
|请求   | http://www.example.com/redirect?url=http://news.example.com |
|重定向到   | http://news.example.com |

<br />

### URL_PREFIX_ADD
设置重定向URL为当前URL加上特定前缀。参数为需要增加的前缀。

示例：
```json    
{
    "Cmd": "URL_PREFIX_ADD", 
    "Params": ["/v1"]
}
```

结果：
| | |
|-|-|
|请求   | http://www.example.com/test.html |
|重定向到   | http://www.example.com/v1/test.html |

<br />

### SCHEME_SET
设置重定向URL的scheme为HTTP或者HTTPS。参数指定为http或者https。

示例：
```json    
{
    "Cmd": "SCHEME_SET", 
    "Params": ["https"]
}
```

结果：
| | |
|-|-|
|请求   | http://www.example.com/index.html |
|重定向到   | https://www.example.com/index.html |

<br />

## links
上一章：[第二十三章 配置rewrite](../../operation/configuration/rewrite.md)  
下一章：[第二十五章 配置限流](../../operation/configuration/prison.md)