---
date: 2018-08-15
title: SQLServer-Express版本Docker镜像通过K8S部署失败
tags: [windows1709, docker, kubernetes, SQLServer]
---

最近在给一个客户做windows docker image，需要把SQLServer也运行在Docker内，使用SQLServer Express版本（免费，但是只能使用2G内存），发现微软官方SQLServer Express版本镜像，通过Kubernetes部署失败。


### 问题现象
1. SQLServer Express版本，通过K8S部署后sa账号无法登录（原因是启用sa账号失败）
```
如果执行sqlcmd时候指定 -S 参数则没问题
```
2. 通过手动执行```docker run ...```没有问题
3. SQLServer Developer版本，通过K8S部署没有此问题

### 问题分析
#### 镜像源代码启用sa账号执行sqlcmd时候未指定Server
在SQLServer-Express版本官方Dockerfile以及[start.ps1](https://github.com/Microsoft/mssql-docker/blob/master/windows/mssql-server-windows-express/start.ps1)脚本中，会通过sqlcmd执行SQL命令设置密码并启用sa登录：

```
if($sa_password -ne "_")
{
    Write-Verbose "Changing SA login credentials"
    $sqlcmd = "ALTER LOGIN sa with password=" +"'" + $sa_password + "'" + ";ALTER LOGIN sa ENABLE;"
    & sqlcmd -Q $sqlcmd
}
```
 
这里执行```sqlcmd -Q $sqlcmd```的时候，并未指定连接的Server地址，按照微软官方文档说明，不指定```-S```参数，默认使用```COMPUTERNAME```。

#### 手动执行 docker run ... 与 使用K8S启动容器，容器内的hostname与COMPUTERNAME有差别
- 手动执行 ```docker run ...```，在docker内查询到的hostname与COMPUTERNAME一致；
- 通过K8S启动docker，docker内的hostname为pod名称；如果pod是无状态，那么hostname是随机数；如果pod名称是有状态，那么hostname是固定值

通过上面两点分析，尝试使用有状态应用在K8S中部署SQLServer Express，并设置环境变量```COMPUTERNAME=${POD_NAME}```，使得在容器内看到的COMPUTERNAME与hostname一致，但是问题任然存在。


