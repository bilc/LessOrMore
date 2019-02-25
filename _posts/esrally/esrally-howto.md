---
layout: post
title:  esrally使用  
date:   2019-2-25 
categories: document
tag: elasticsearch 
---

* content
{:toc}




# esrally

本文描述使用esrally对自建集群进行测试的方法(esrally还有其他用法不做讨论) 

<!-- vim-markdown-toc GFM -->

* [1. esrally 介绍](#1-esrally-介绍)
* [2. 测试准备](#2-测试准备)
    * [2.1 document.json](#21-documentjson)
    * [2.2 index.json](#22-indexjson)
    * [2.3 编写track.json](#23-编写trackjson)
* [3. 命令](#3-命令)

<!-- vim-markdown-toc -->

## 1. esrally 介绍

esrally是基于用户视角的对elasticsearch进行测试的工具。

术语：
- rally:汽车拉力赛
- track:赛道,压测方案
- team/car: es instance 
- race: 一场比赛 
- tournament: 锦标赛 

track.json是esrally的压测方案定义文件,包含以下几部分  
- indices:索引定义 
- templates:indices template，少使用
- corpora:数据集文件定义  
- operations:具体操作,可以没有，直接在schedule或者challenge内定义
- schedule:执行操作时的负载
- challenge:区分不同测试场景,比如append和update，便于分开统计 

示例:  
```
{
    "version": 2,
    "description": "xxx",
    "indices": [
        {
            "name": "xxxx",
            "body": "xxx你定义的索引格式请求文件 index.json ",
            "types": ["_doc"]
        }
    ],
    "corpora": [
        {
            "name": "xxxx",
            "documents": [
                {
                    "source-file": "es请求文件:目前仅支持bulk格式",
                    "document-count": 100,
                    "uncompressed-bytes": 84553
                }
            ]
        }
    ],
    "challenges":[
        {
            "name":"xxx",
            "schedule": [
                {
                    "operation": {
                        "name": "putdata",
                        "operation-type": "bulk",
                        "bulk-size": 5
                    }
                    "clients": 4,
                    "warmup-iterations": 1000,
                    "iterations": 1000,
                    "target-throughput": 100
                }
            ]
        }
    ]
}
```
具体字段参考https://esrally.readthedocs.io/en/latest/track.html 

教程视频 https://www.bilibili.com/video/av27114309/?spm_id_from=333.788.videocard.0  (某些字段可能已经不适用)

## 2. 测试准备   

测试前需要准备一下文件  
- document.json: 测试请求文件
- index.json: 创建索引请求文件  
- track.json: 测试case描述文件 

### 2.1 document.json  

格式只支持bulk,参加[eventdata_sample.json](./eventdata_sample.json)  
示例 
```
{"agent": "\"Mozilla/5.0 (Windows NT 10.0; WOW64; rv:44.0) Gecko/20100101 Firefox/44.0\"", "useragent": {"os": "Windows 10", "os_name": "Windows 10", "name": "Firefox"}, "geoip": {"country_name": "Russian Federation", "location": [44.00749999999999, 56.326899999999995]}, "clientip": "93.120.181.0", "referrer": "\"https://www.elastic.co/guide/en/logstash/current/index.html\"", "request": "/favicon-96x96.png", "bytes": 0, "verb": "GET", "response": 304, "httpversion": "1.1", "@timestamp": "2017-07-03T07:51:49.999Z", "message": "93.120.181.0 - - [2017-07-03T07:51:49.999Z] \"GET /favicon-96x96.png HTTP/1.1\" 304 0 \"-\" \"https://www.elastic.co/guide/en/logstash/current/index.html\" \"Mozilla/5.0 (Windows NT 10.0; WOW64; rv:44.0) Gecko/20100101 Firefox/44.0\""}
```

### 2.2 index.json

将数据存入es中进行建模,index创建请求参考https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html  

将请求写入文件，例如 
[index.json](./index.json)


### 2.3 编写track.json  

主要自定义challenge部分  
示例：
[track.json](./track.json)

## 3. 命令  

查看测试track
```
esrally list tracks --track-path=xx
```
运行track
```
esrally --track-path=xx  \
--trget-hosts=127.0.0.1:9192 \
--pipline=benchmark-only \
--cluster-health=yellow
```
