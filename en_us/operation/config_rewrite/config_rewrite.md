# Configure Rewrite
This chapter describes how to configure HTTP rewrite. This function modifies the received HTTP request and forwards it to the backend service.

## Enable Rewrite
In conf/bfe.conf, enable the module

```ini
Modules = mod_write
```

## Configuration
The configuration is located in the directory conf/mod_rewrite/, contains two files:

```
$ ls
mod_rewrite.conf	rewrite.data
```

mod_rewrite.conf is the basic configuration file of the module, pointing to the rewrite rule file. mod_rewrite.conf usually does not need to be modified.

```
$ cat mod_rewrite.conf 
[basic]
DataPath = mod_rewrite/rewrite.data
```

rewrite.data contains rewrite rules and can be loaded dynamically. The sample configuration file in the installation package is as follows:

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
The above configuration adds a rule for product *example_product*: for the request that meets the condition identified by "Cond", execute the actions identified by "Actions" (including the action name identified by "Cmd" and the corresponding parameter). If "Last" is true, stop the execution of subsequent actions, otherwise continue to match the next rule.

Finally, the rule will modify the request whose path starts with "/rewrite" and add the path prefix "/bfe/" to it, that is, change the path from "/rewrite" to "/bfe/rewrite".


## Actions

The "Cmd" in "Actions" indicates the specific rewrite action.

### HOST_SET
Set the Host in the request header. The parameter is the value to be set.

Example:
```json    
{
    "Cmd": "HOST_SET", 
    "Params": ["www.example.com"]
}
```

Result:
| | |
|-|-|
|Original value   | http://abc.example.com |
|After rewrite   | http://www.example.com |

### HOST_SET_FROM_PATH_PREFIX
Set the first section in Path to the value of Host and remove it from Path. For example, if the Path is "/x.example.com/xxxx", set the Host to "x.example.com", and the PATH to "/xxxx".

Example:
```json  
{
    "cmd": "HOST_SET_FROM_PATH_PREFIX", 
    "params": []
}
```

Result:
| | |
|-|-|
|Original value   | http://www.example.com/test.example.com/xxxx |
|After rewrite   | http://test.example.com/xxxx |


### HOST_SUFFIX_REPLACE
Replace the specific suffix in the domain name. The two parameters are the suffix string to be replaced and the new string.

Example:
```json  
{
    "cmd": "HOST_SUFFIX_REPLACE", 
    "params": ["net", "com"]
}
```

Result:
| | |
|-|-|
|Original value   | http://www.example.net |
|After rewrite   | http://www.example.com |

### PATH_SET
Set the Path to the specified value and the parameter to the new value of Path .

Example:
```json  
{
    "cmd": "PATH_SET", 
    "params": ["/index"]
}
```

Result:
| | |
|-|-|
|Original value   | http://www.example.com/current |
|After rewrite   | http://www.example.com/index |

### PATH_PREFIX_ADD
Add a prefix for Path. The parameter is the prefix to add.

Example:
```json  
{
    "cmd": "PATH_PREFIX_ADD", 
    "params": ["/index"]
}
```

Result:
| | |
|-|-|
|Original value   | http://www.example.com/current |
|After rewrite   | http://www.example.com/index/current |

### PATH_PREFIX_TRIM
Delete the Path prefix. The parameter is the prefix to delete.

Example:
```json  
{
    "cmd": "PATH_PREFIX_TRIM", 
    "params": ["/service"]
}
```

Result:
| | |
|-|-|
|Original value   | http://www.example.com/service/index.html |
|After rewrite   | http://www.example.com/index.html |

### QUERY_ADD
Add Query. The parameter specifies the key and value of the query to be added.

Example:
```json  
{
    "cmd": "QUERY_ADD", 
    "params": ["name", "alice"]
}
```

Result:
| | |
|-|-|
|Original value   | http://www.example.com/ |
|After rewrite   | http://www.example.com/?name=alice |


### QUERY_RENAME
Rename the query. The parameter specifies the original name and new name of the key.

Example:
```json  
{
    "cmd": "QUERY_ADD", 
    "params": ["name", "user"]
}
```

Result:
| | |
|-|-|
|Original value   | http://www.example.com/?name=alice |
|After rewrite   | http://www.example.com/?user=alice |

### QUERY_DEL
Delete the specified Query. The parameter specifies the name of the key.

Example:
```json  
{
    "cmd": "QUERY_ADD", 
    "params": ["name"]
}
```

Result:
| | |
|-|-|
|Original value   | http://www.example.com/?name=alice |
|After rewrite   | http://www.example.com/ |

### QUERY_DEL_ALL_EXCEPT
Delete all other keys except the specified key in Query.

Example:
```json  
{
    "cmd": "QUERY_DEL_ALL_EXCEPT", 
    "params": ["name"]
}
```

Result:
| | |
|-|-|
|Original value   | http://www.example.com/?name=alice?key1=value2&key2=value2 |
|After rewrite   | http://www.example.com/?name=alice |


## links
Previous: [Chap22 Configure HTTPS Service](../../../en_us/operation/config_https/config_https.md)  
Next: [Chap24 Configure Redirect](../../../en_us/operation/config_redirect/config_redirect.md)