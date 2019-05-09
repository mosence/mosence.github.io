---
title: 
date: 2019-04-24 09:34:00
tags:
---

flinkx 是基于 flinx为底层数据传输中间处理，进行input,output组件化的项目。

可以实现json配置一个简单数据流程，通过flinkx处理后提交由flink集群调用。

* 部署

[flinkx](https://github.com/DTStack/flinkx)

[flink](https://flink.apache.org/) 

flinkx clone源码后通过maven编译，最终生成 flinkx 的 bin,lib,plugins

flink 按照文档一步一步下载运行就行了，集群间自动发现。

* 使用

简单调用如下

```shell
./bin/flinkx -job ./temp/sftp_to_sftp.json -plugin ./plugins
```

sftp_to_sftp.json，一个简易的本地文件抽取，抽取文件并替换分隔符，添加处理标识。
```json
{
    "job": {
        "setting": {
            "speed": {
                "channel": 1
            },
            "errorLimit": {
                "record": 0,
                "percentage": 0.02
            }
        },
        "content": [
            {
                "reader": {
                    "name": "ftpreader",
                    "parameter": {
                        "protocol": "sftp",
                        "host": "localhost",
                        "port": 22,
                        "username": "root",
                        "column": [
                            {
                                "index": 0
                            },
                            {
                                "index": 1
                            },
                            {
                                "value": "isRead",
                                "type": "string"
                            }
                        ],
                        "path": "/data/flink/test/readData/",
                        "encoding": "UTF-8",
                        "fieldDelimiter": "\\t"
                    }
                },
                "writer": {
                    "name": "ftpwriter",
                    "parameter": {
                        "protocol": "sftp",
                        "host": "localhost",
                        "port": 22,
                        "username": "root",
                        "writeMode": "overwrite",
                        "path": "/data/flink/test/writeData/",
                        "fieldDelimiter": ",",
                        "column": [
                            {
                                "name": "ID",
                                "type": "string"
                            },
                            {
                                "name": "NAME",
                                "type": "string"
                            },
                            {
                                "name": "STATUS",
                                "type": "string"
                            }
                        ]
                    }
                }
            }
        ]
    }
}

```
/data/flink/test/readData/test.txt
```text
1	NAME1
2	NAME2
```

最终生成结果 0.20190509143030.cb219e90-da3a-439b-b749-3f01b07a7930.csv
```
1,NAME1,isRead
2,NAME2,isRead
```

问题1： 生成的结果默认是 .csv格式，并且名称重新定义了，不知道能不能可控？