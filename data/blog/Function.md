---
date: 2018-09-01
title: Function
tags: [Cloud-Service-Mapping, Computing, Function]
---

## Levels of support
There are three levels of support:
- **Generally available (GA)** - Fully supported and approved for production use.
- **Preview** - Not yet supported but is expected to reach GA status in the future.
- **No** - Not support

## 特性对比 Featrue Mapping

### 运行时 Runtime

|   Runtime      | huawei cloud  |  aliyun    |  tencent cloud   |    aws    |    azure    |  google cloud  |
| -------------- | :-----------: | :--------: | :--------------: | :-------: | :---------: | :------------: |
| Node.js 4.3    | No            | No         | No               | GA        | No          | No             | 
| Node.js 6      | GA            | GA         | GA               | GA        | No          | No             |
| Node.js 8      | GA            | GA         | GA               | GA        | No          | No             |
| Python 2.x     | GA            | GA         | GA               | GA        | No          | No             |
| Python 3.x     | GA            | GA         | GA               | GA        | GA          | No             |
| Java 8         | GA            | GA         | GA               | GA        | Preview     | No             |
| Go 1.8         | GA            | No         | No               | GA        | No          | No             |
| .net 1.x       | No            | No         | No               | GA        | GA          | No             |
| .net 2.x       | No            | No         | No               | GA        | Preview     | No             |
| PHP 5          | No            | No         | GA               | No        | No          | No             |
| PHP 7          | No            | No         | GA               | No        | No          | No             |
| Javascript     | No            | No         | No               | GA        | No          | GA             |

### 触发器 Trigger

|   Trigger      | huawei cloud  |  aliyun    |  tencent cloud   |    aws    |    azure    |  google cloud  |
| -------------- | :-----------: | :--------: | :--------------: | :-------: | :---------: | :------------: |
| API 网关       | API Gateway   | HTTP       | No               | API Gateway | HTTP      | HTTP           | 
| 对象存储       | OBS           | OSS         | COS            | S3        | Blog Storage | Cloud Storage  | 
| 消息主题订阅   | SMN           | No         | CMQ             | SNS       | No          | No             | 
| 消息队列拉取   | DMS           | No         | CKafka          | SQS       | Quque          | No             | 
| 日志          | LTS           | 日志服务    | No             | CloudWatchLogs | No          | No             | 
| 资源操作      | CTS           | No         | No               | CloudWatchEvents | No          | No             | 
| 定时器       | TIMER          |TIMER       | TIMER           | TIMER     | TIMER          | No             | 
| 数据流       | DIS           | No         | GA               | Kinesis Data Streams | No          | No             | 
| 内容分发     | No            | No        | GA               | CloudFront        | No          | No             | 
| NoSQL       | NO            | No         | GA               | DynamonDB        | No          | No             | 
| 堆栈        | NO            | No         | GA               | CloudFormation   | No          | No             | 
| IOT         | NO           | No         | GA               | IOT        | No          | No             | 

### 监控指标 Metric

|   Metric       | huawei cloud  |  aliyun    |  tencent cloud   |    aws    |    azure    |  google cloud  |
| -------------- | :-----------: | :--------: | :--------------: | :-------: | :---------: | :------------: |
| 调用次数        | YES           | YES        | No               | YES       | Unknown     | Unknown      | 
| 最大运行时间    | YES           | No         | GA               | YES       | Unknown     | Unknown      |
| 最小运行时间    | YES           | No         | GA               | YES       | Unknown     | Unknown      |
| 平均运行时间    | YES           | No         | GA               | YES       | Unknown     | Unknown      |
| 错误次数        | YES           | No         | GA               | YES       | Unknown     | Unknown      |
| 错误率          | No            | No         | GA               | YES       | Unknown     | Unknown      |
| 死信错误        | No            | No         | No               | YES       | Unknown     | Unknown      |
| 并行限制错误    | YES           | No         | No               | YES       | Unknown     | Unknown      |
| 资源使用量GB-S  | No            | YES        | No               | No        | Unknown     | Unknown      |
| 出流量          | No            | No         | GA               | No        | Unknown     | Unknown      |


### 配额 Limits

|   Runtime      | huawei cloud  |  aliyun    |  tencent cloud   |    aws    |    azure    |  google cloud  |
| -------------- | :-----------: | :--------: | :--------------: | :-------: | :---------: | :------------: |
| 最大运行时间    | 300s          | Unknown    | Unknown          | 300s      | 300s        | 540s           |
| 并发执行数      | Unknown       | Unknown    | Unknown          | 1000/account/region | Unknown     | Unknown        |

### 函数部署 Deployments

|   Runtime      | huawei cloud  |  aliyun    |  tencent cloud   |    aws    |    azure    |  google cloud  |
| -------------- | :-----------: | :--------: | :--------------: | :-------: | :---------: | :------------: |
| 函数部署        | ZIP upload    | Unknown    | Unknown          | ZIP upload | OneDrive,Local Git,GitHub,BitBucker,Dropbox,External REpository        | Unknown           |


## References
[Microsoft Azure Functions vs. Google Cloud Functions vs. AWS Lambda: fight for serverless cloud domination continues](https://cloudacademy.com/blog/microsoft-azure-functions-vs-google-cloud-functions-fight-for-serverless-cloud-domination-continues/)