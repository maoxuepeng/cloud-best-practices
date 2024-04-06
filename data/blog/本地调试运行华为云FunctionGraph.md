---
date: 2021-07-28
title: 本地调试运行华为云FunctionGraph
tags: [Serverless]
---

函数服务（FaaS）对开发者屏蔽了进程级别的基础设施，开发者只能看到代码，那么如何支持典型的本地开发与调试的需求，是FaaS服务提供商需要思考的问题。[AWS SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) 提供了本地调试与运行函数代码的工具链。华为云函数服务在这方面也有一些探索，提供了华为云Serverless Sandbox（HSS）工具，本文介绍如何在本地运行华为云函数服务代码。

## 准备环境

HSS工具依赖Docker，因此Windows上运行会比较麻烦，选择了在Windows上启动WSL，搭建运行环境。

安装Docker

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## 安装HSS工具

下载工具包

```shell
wget https://obs.cn-north-1.myhuaweicloud.com/functionstage-sandbox/hss-1.0.10.linux-amd64.tgz
```

解压

```shell
tar zxf hss-1.0.10.linux-amd64.tgz

drwxr-x---  4 mao mao     4096 Jul 28 12:05 ./
drwxr-xr-x  3 mao mao     4096 Jul 28 11:40 ../
-rw-------  1 mao mao    19674 Jul 11  2019 README.md
-rwxr-x---  1 mao mao      297 Jul 28 11:55 config.yaml*
-rw-------  1 mao mao 36995042 Jul 11  2019 fss-sandbox-csharp2.0-runtime.tar.gz
-rw-------  1 mao mao 39701922 Jul 11  2019 fss-sandbox-csharp2.1-runtime.tar.gz
-rw-------  1 mao mao  2012330 Jul 11  2019 fss-sandbox-go1.8-runtime.tar.gz
-rw-------  1 mao mao 75654679 Jul 11  2019 fss-sandbox-java8.0-runtime.tar.gz
-rw-------  1 mao mao 11916472 Jul 11  2019 fss-sandbox-nodejs6.10-runtime.tar.gz
-rw-------  1 mao mao 13342910 Jul 11  2019 fss-sandbox-nodejs8.10-runtime.tar.gz
-rw-------  1 mao mao 30998019 Jul 11  2019 fss-sandbox-python3.6-runtime.tar.gz
-rwxr-x---  1 mao mao  7698496 Jul 11  2019 hss*
-rw-r--r--  1 mao mao        5 Jul 28 11:57 hss_history.tmp
drwxr-x--- 10 mao mao     4096 Jul 28 12:05 template/
drwxr-x---  4 mao mao     4096 Jul 11  2019 test/
```

将 ```hss``` 可执行文件加入到系统 ```PATH``` 环境变量。

## 配置

官方文档介绍的配置放在 ```config.yaml```，但是修改此文件并不能正确完成配置。

正确的配置方案如下。

1. 执行 ```hss```进入命令行交互模式。
2. 在交互模式下，配置参数

查看当前配置项

```shell
$hss> info
2021/07/28 14:55:46 INFO:
RegionName:cn-north-4
ProjectName:cn-north-4
AccessKey:<your access key>
SecretKey:<your secret key>
ProjectId:<your project id>
FGSAddress:
IAMAddress:https://iam.cn-north-4.myhuaweicloud.com
SMNAddress:https://smn.cn-north-4.myhuaweicloud.com
APIGAddress:https://apig.cn-north-4.myhuaweicloud.com
OBSAddress:https://obs.cn-north-4.myhuaweicloud.com
Timeout:30
ProxyServerAddress:
ProxyUserName:
```

按照帮助指引，通过命令行配置 ak、sk、projectId、porjectName

```shell
$hss> update-config --help
NAME:
   hss update-config - Update Config Region

USAGE:
   hss update-config [command options] [--config-RegionName <regionName> --config-AccessKey <AccessKey> ---config-SecretKey <SecretKey> --config-ProjectName <ProjectName> ...]

CATEGORY:
   Region Config

OPTIONS:
   --config-RegionName value, --rn value        Region RegionName
   --config-ProjectName value, --pn value       Region ProjectName
   --config-ProjectId value, --pi value         Region ProjectId
   --config-AccessKey value, --ak value         Region AccessKey
   --config-SecretKey value, --sk value         Region SecretKey
   --FGSAddress value, --fa value               Region FGSAddress
   --config-IAMAddress value, --ci value        Region IAMAddress
   --config-OBSAddress value, --co value        Region OBSAddress
   --config-SMNAddress value, --cn value        Region SMNAddress
   --config-APIGAddress value, --cg value       Region APIGAddress
   --config-Timeout value, --ct value           Region Timeout (default: 0)
   --ProxyServerAddress value, --pa value       Proxy Server Address
   --ProxyUserName value, --pu value            Proxy User Name
   --ProxyPassword value, --pp value            Proxy Password
```

## 本地运行函数

1. 使用官方文档提供的[HSAM模板文件](https://support.huaweicloud.com/tg-functiongraph/functiongraph_08_0380.html)，保存到本地。
2. 验证HSAM模板文件的正确性

```shell
~/hss/hss$ hss validate -t template/fs_template.yaml
2021/07/28 11:57:38 INFO: Switch the region successful,current region:cn-north-4
2021/07/28 11:57:38 INFO: Template: template/fs_template.yaml, successfully parsed
```

2. 生成apig触发器事件

```shell
hss localEvent apig > apig_event.json
```

cat apig_event.json

```json
{
    "body": "",
    "requestContext": {
        "apiId": "bc1dcffd-aa35-474d-897c-d53425a4c08e",
        "requestId": "11cdcdcf33949dc6d722640a13091c77",
        "stage": "RELEASE"
    },
    "queryStringParameters": {
        "responseType": "html"
    },
    "httpMethod": "GET",
    "pathParameters": {},
    "headers": {
        "accept-language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2",
        "accept-encoding": "gzip, deflate, br",
        "x-forwarded-port": "443",
        "x-forwarded-for": "127.0.0.1",
        "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
        "upgrade-insecure-requests": "1",
        "host": "50eedf92-c9ad-4ac0-827e-d7c11415d4f1.apigw.cn-north-1.huaweicloud.com",
        "x-forwarded-proto": "",
        "pragma": "no-cache",
        "cache-control": "no-cache",
        "x-real-ip": "127.0.0.1",
        "user-agent": "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:57.0) Gecko/20100101 Firefox/57.0"
    },
    "path": "/mytest",
    "isBase64Encoded": true
}
```

3. 启动本地APIG，供客户端调用

```shell
sudo ./hss apigw -e template/apig_event.json -P 8888 -t template/fs_template.yaml
2021/07/28 12:05:33 INFO: Switch the region successful,current region:cn-north-4
2021/07/28 12:05:34 INFO: Template: template/fs_template.yaml, successfully parsed
2021/07/28 12:05:34 INFO: curl http://0.0.0.0:8888/MyFuncStage1 to execution function MyFuncStage1
2021/07/28 12:05:34 INFO: Please choose to use curl to execute the above function.
...
```

4. 调用函数

```shell
curl http://localhost:8888/MyFuncStage1
```

输出如下

```json
Hello message: {"body": "", "requestContext": {"apiId": "bc1dcffd-aa35-474d-897c-d53425a4c08e", "requestId": "11cdcdcf33949dc6d722640a13091c77", "stage": "RELEASE"}, "queryStringParameters": {"responseType": "html"}, "headers": {"accept-language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2", "accept-encoding": "gzip, deflate, br", "x-forwarded-port": "443", "x-forwarded-for": "127.0.0.1", "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8", "upgrade-insecure-requests": "1", "host": "50eedf92-c9ad-4ac0-827e-d7c11415d4f1.apigw.cn-north-1.huaweicloud.com", "x-forwarded-proto": "", "pragma": "no-cache", "cache-control": "no-cache", "x-real-ip": "127.0.0.1", "user-agent": "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:57.0) Gecko/20100101 Firefox/57.0"}, "pathParameters": {}, "httpMethod": "GET", "path": "/mytest", "isBase64Encoded": true}

```

## 后记

华为云FunctionGraph的HSS工具以及HSAM模型，目前只能对简单的函数代码做本地调试，因为HSAM模型中只能写代码，并不能引用一个包。这样的话，不符合生产环境场景。

## Reference

[华为云Serverless Sandbox（HSS）](https://support.huaweicloud.com/tg-functiongraph/functiongraph_08_0100.html)


