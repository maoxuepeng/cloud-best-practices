---
date: 2019-6-26
title: Dockerfile最佳实践
tags: [Dockerfile, 最佳实践]
---

容器的OCI标准定义了容器镜像规范，容器镜像包与传统的压缩包(zip/tgz等)相比有两个关键区别点：1）分层存储；2）打包即部署。

分层存储可以极大减少镜像更新时候拉取镜像包的时间，通常应用程序更新升级都只是更新业务层（如Java程序的jar包），而镜像中的操作系统Lib层、运行时（如Jre）层等文件不会频繁更新。因此新版本镜像实质有变化的只有很小的一部分，在更新升级时候也只会从镜像仓库拉取很小的文件，所以速度很快。

打包即部署是指在容器镜像制作过程包含了传统软件包部署的过程（安装依赖的操作系统库或工具、创建用户、创建运行目录、解压、设置文件权限等等），这么做的好处是把应用及其依赖封装到了一个相对封闭的环境，减少了应用对外部环境的依赖，增强了应用在各种不同环境下的行为一致性，同时也减少了应用部署时间。

基于分层存储与打包即部署的特性，容器可以更快的部署、运行、保持环境一致性，这些特性都是容器相对于传统虚拟机打包部署的优势。这两个点都是通过dockerfile来承载的，因此如何编写高效、安全、规范易用的dockerfile是容器实践中关键的一个环节。


## Dockerfile 最佳实践概览

<iframe width='853' height='480' src='https://embed.coggle.it/diagram/XRMuwHTTRuswFEzY/10ceb37829fb4c938ce91fd474ed68721b5d7fda8a383ff5a583ff398a8b4b26' frameborder='0' allowfullscreen></iframe>

## 1. 规范与安全

### 1.1. FROM

#### 1.1.1 优先使用最小功能集的基础镜像

很多基础镜像都提供不同功能集的版本，根据实际情况选择最小功能集的版本，功能越少体积越小，引入安全漏洞的风险越低。
如[Node镜像](https://hub.docker.com/_/node)，最小的才不到30M，最大的超过200M。

#### 1.1.2 显示指定基础镜像的版本，禁用latest

**latest** 是一个不确定的版本号，依赖 latest 会导致每次构建出的镜像是不可预知的。

#### 1.1.3 指定依赖最具体的镜像版本

依赖基础镜像版本时候，尽量指定到最具体的版本，这样不确定性最小。如[Node镜像](https://hub.docker.com/_/node)，版本号有 **12**, **12.4**, **12.4.0**，依赖 **12.4.0** 是不确定性最小的。

#### 1.1.4 条件允许建议将依赖的基础镜像在本地仓库mirror一份防止相同tag的镜像内容不同

容器镜像仓库中的tag是可以被复写的，同一个tag在不同的时间对应内容可能不同，因此如果条件允许，将依赖的镜像mirror一份到本地，这样可以保证
每次构建时候依赖的基础镜像是不变的。

#### 1.1.5 显示指定基础镜像的平台架构

Docker 1.10 版本开始，Docker官方镜像仓库与Docker-EE/Docker-CE都支持[镜像多平台功能](https://blog.docker.com/2017/09/docker-official-images-now-multi-platform/)，这里的多平台包含不同类型操作系统（Windows/Linux），不同CPU体系（X86/ARM64）。

利用镜像多平台功能，在```docker pull ...```命令中可以不用显示指定平台类型，docker客户端会根据当前的平台架构与镜像仓库协商（基于[manifest list特性](https://github.com/docker/distribution/blob/master/docs/spec/manifest-v2-2.md#manifest-list)），拉取对应平台的镜像。

通过这个特性可以保持用户界面的简洁性，客户端执行 ```docker pull node:12``` 命令，在ARM64机器上，则会拉取 ```arm64v8/node:12``` 镜像，在Linux机器上，则拉取 ```node:12``` 镜像。

但是这种方式也引入了不确定性，不确定性表现为2方面：第一方面为不同平台上版本存在差异，可能依赖的版本只在一个平台有；第二个方面为有些时间可能在Linux环境下构建ARM镜像，这样会导致构建出错误镜像。


### 1.2. LABEL
#### 1.2.1 通过LABEL指令增加镜像元数据（作者，时间，描述等）

通过Label增加镜像作者，构建时间，描述等信息，让使用者得到更多关于镜像信息。

#### 1.2.2 在Label中增加securitytxt规范

通过Label增加镜像对应应用的 [securitytxt](https://securitytxt.org/) 信息。
[securitytxt](https://securitytxt.org/) 是 [IETF](https://www.ietf.org/) 组织起草的一份规范，目的定义一套互联网服务提供者与安全漏洞发现者之间交互的规范，使得安全漏洞能够被闭环。

### 1.3. WORKDIR

#### 1.3.1 使用WORKDIR指定工作目录，避免绝对路径扩散

RUN, COPY 命令都要使用到绝对路径，定义好 WORKDIR 会使得 Dockerfile 移植性更好，更容易维护。

### 1.4 ENV与ARG

#### 1.4.1 勿使用ENV与ARG传递敏感信息

ENV, ARG 的值都会被记录下来，通过 ```docker image history``` 命令可以查看到，因此不要将敏感信息传递给ENV与ARG。

如果需要在dockerfile中使用密钥或凭证，使用 [mount secret](https://medium.com/@tonistiigi/build-secrets-and-ssh-forwarding-in-docker-18-09-ae8161d066) 方式。

### 1.5. RUN

#### 1.5.1 上下文依赖的命令在同一层完成（一个RUN指令）

由于 docker build cache 机制，有上下文依赖的命令放到同一个RUN指令中执行。

假如有如下dockefile片段：

```
...
RUN apt-get update
RUN apt-get install -y nginx
...
```

过了一段时间之后，需要修改一下上述dockerfile，增加一个安装包

```
...
RUN apt-get update
RUN apt-get install -y nginx python
...
```

此时构建镜像，docker 比对缓存，```RUN apt-get update``` 这一层已经存在，使用缓存。这时候apt仓库中的python或nginx可能已经有新版本了，由于没有执行 ```apt-get update``` ，因此安装的并不是最新版本。

#### 1.5.2 考虑build cache的时效性，使用--no-cache参数禁用cache

通过构建参数 ```build --no-cache``` 显示使得缓存失效，不过缓存失效会增加构建时长，需要综合考虑。

#### 1.5.3 使用 set -o pipefail 避免管道错误被忽略

假如在dockerfile中执行命令 ``` RUN wget -O - https://some.site | wc -l > /number``` ，如果 ```wget -O - https://some.site``` 执行失败了，但是 ```wc -l``` 是成功的，因此并不会报错退出。

使用 ```set -o pipefail``` 避免管道错误被忽略。上述命令修改为： ```RUN set -o pipefail && wget -O - https://some.site | wc -l > /number``` 。

### 1.6 ADD与COPY

#### 1.6.1 优先使用COPY，比ADD更简单明了

ADD 命令功能相对 COPY 更复杂，包含从internet下载，解压等，优先使用 COPY 。

#### 1.6.2 禁止使用ADD从远程URL下载包，使用curl或wget先下载，使用ADD从本地解压到镜像内

不推荐使用 ADD 命令从远程下载一个软件包解压到镜像内，推荐使用 curl 下载到本地，然后再解压到镜像内，并将原始文件删除。

### 1.7 USER

#### 1.7.1 禁用ROOT用户运行应用，为应用创建用户与用户组

root用户运行应用程序存在安全风险，为每个应用单独创建一个运行用户。

#### 1.7.2 如果应用依赖特定的UID/GID，则创建用户/用户组时候显示指定

由于dockerfile中创建用户/用户组的 UID/GID 是不固定的，如果应用程序依赖 UID/GID，创建用户时候显示指定。

#### 1.7.4 应用运行用户的Shell设置为/sbin/nologin

应用程序运行用户设置Shell为 /sbin/nologin 。

#### 1.8 EXPOSE

### 1.8.1 使用EXPOSE指令指明Listen端口与协议

通过 EXPOSE 指令申明应用 listen 的端口与协议，让应用运维人员者简单明了得知端口。

```
EXPOSE 80/tcp
```

### 1.9 VOLUME

#### 1.9.1 使用VOLUME申明镜像中需要写入数据的目录

程序持久化或临时文件目录，使用 VOLUME 指令申明，让应用运维人员简单明了得知需要挂载的卷。

### 1.10 日志

#### 1.10.1 将标准日志与错误日志分别输出到stdout与stderr

日志输出到标准输出与错误输出，方便查看与采集日志。参考 [Nginx 的 dockerfile](https://github.com/nginxinc/docker-nginx/blob/b749353968a57ebd9da17e12d23f1a5fb62f9de9/mainline/stretch/Dockerfile) ：

```
# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log
```

### 1.11 CMD与ENTRYPOINT

#### 1.11.1 优先使用CMD/ENTRYPOINT指令的EXEC格式设置镜像的默认执行程序，使得应用进程PID为1

推荐将容器内的应用程序设置为PID 1，这样对容器内应用管理相对简单，1号进程退出后整个容器也就退出了。

在dockerfile中使用 CMD 或 ENTRYPOINT 指令设置容器镜像的默认启动进程， CMD/ENTRYPOINT 有两种执行方式：exec 与 /bin/sh ，使用 exec 方式直接执行应用程序可执行文件设置PID为1，使用 /bin/sh 方式的话，1号进程就是 /bin/sh 。

exec 方式： ```CMD["java", "/same/args"]```, ```ENTRYPOINT ["java", "/same/args"]``` ， /bin/sh 方式： ```CMD java /same/args```, ```ENTRYPOINT java /same/args``` 。

#### 1.11.2 使用ENTRYPOINT封装镜像的固定行为，使用CMD配合输入可变参数

ENTRYPOINT 与 CMD 功能类似，他们区别是 ENTRYPOINT 不能被 docker run 的命令行参数覆盖，而 CMD 可以； ENTRYPOINT 通常配合 CMD 使用，使用 CMD 设置 ENTRYPOINT 的默认参数，同时支持在 docker run 设置参数传递给 ENTRYPOINT。

参考[Redis的ENTRYPOINT](https://github.com/docker-library/redis/blob/6845f6d4940f94c50a9f1bf16e07058d0fe4bc4f/5.0/docker-entrypoint.sh)写法。

对于切换用户，推荐使用 gosu 替换 sudo 。

#### 1.11.3 使用ENTRYPOINT执行运行前准备工作

如果在运行应用程序之前需要做一些准备工作，如检查文件/目录权限等，那么可以在 ENTRYPOINT 的脚本中完成。

#### 1.11.4 正确理解K8S的command与args与docker镜像的ENTRYPOINT与CMD的关系

通过K8S调度docker镜像可以覆盖dockerfile中的 ENTRYPOINT 与 CMD，他们之间的关系参考[如下文章](https://yeasy.gitbooks.io/docker_practice/image/dockerfile/entrypoint.html)。

1.12 使用[hadolint](https://github.com/hadolint/hadolint)工具检查dockerfile

[hadolint](https://github.com/hadolint/hadolint) 定义了一套规则与检查工具，可以在项目中使用。

## 2. 效率

### 2.1 体积小

#### 2.1.1. 选择最小的基础镜像

参考 1.1.1 。

#### 2.1.2 禁止使用 ```chmod -R /a/root/path```，更改具体某个/类文件的权限

由于dockerfile构建镜像使用的是分层只读文件系统，如果使用 chmod 更改一个大目录权限，相当于复制了这个目录下所有文件，会导致镜像体积变大。

#### 2.1.3 分阶段构建

Docker 17.05 版本中引入了分阶段构建（{Multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)），通过分阶段构建可以减少镜像大小。

#### 2.1.4 dockerfile书写顺序按照更新频度升序排序

Docker镜像的分层是子层依赖父层，对应到 Docker build cache 机制中，如果某一层未命中缓存，那么其剩下的层都不会使用缓存。
因此如果需要最大利用缓存机制，推荐将变化频度低的层尽量放上层，变化频度高的层放下层。

#### 2.1.5 禁止使用类似apt-get upgrade系统级更新，更新具体需要的软件包

apg-get upgrade 会使得镜像大小不可控，不知道更新了多少软件包。

#### 2.1.6 相关度高/更新频度一致的命令，写到同一个RUN指令中

同样是考虑缓存命中率，变化频度一致的命令放到同一层。

### 2.2 构建快

#### 2.2.1 使用独立的目录作为build context

Docker在构建镜像时候有一个 build context 概念，build context 在 docker build 指定一个目录，docker 会将 build context 目录内所有文件加载到内存，作为build context。

build context 目录内内容越少，加载速度越快，建议使用独立的目录作为build context，只拷贝需要的文件到 build context 目录。

#### 2.2.2 使用.dockerignore文件

使用 .dockerignore 文件排除目录下不需要加载到 build context 内的文件或目录。

#### 2.2.3 从stdin接收dockerfile会使得build context大小为0

Docker 支持从stdin输入 dockerfile ，这种方式不会加载任何本地文件到docker的build context中。

## Reference
[Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

[「Allen 谈 Docker 系列」docker build 的 cache 机制](http://open.daocloud.io/docker-build-de-cache-ji-zhi/)

[理解Docker容器的进程管理](https://yq.aliyun.com/articles/5545)

[为容器设置启动时要执行的命令及其入参](https://kubernetes.io/zh/docs/tasks/inject-data-application/define-command-argument-container/)

[ENTRYPOINT 入口点](https://yeasy.gitbooks.io/docker_practice/image/dockerfile/entrypoint.html)

[hadolint规范与检查工具](https://github.com/hadolint/hadolint)

[Build secrets and SSH forwarding in Docker 18.09](https://medium.com/@tonistiigi/build-secrets-and-ssh-forwarding-in-docker-18-09-ae8161d066)

[dcoker-slim](https://github.com/docker-slim/docker-slim)