---
tags: [vsftpd, linux, docker]
date: 2018-05-11
title: vsftpd与一个客户的故事
---

### 问题总结
#### 问题描述
使用docker运行vsftp服务器（vsftp镜像为：panubo/vsftpd），发现命令端口可以连接，但是数据端口无法连接。ftp模式为Passive。
#### 问题原因
使用的vsftpd镜像中配置的Passive端口与docker-proxy打开的端口不一致导致。

下图是vsftpd镜像中配置的Passive端口，范围为4559-4564：
![](/15-Hours/_image/docker-vsftpd-01.png)

而启动的时候配置的是21100-21110
![](/15-Hours/_image/docker-vsftpd-02.png)

#### 解决方法
因此，修改启动的端口范围，同时在云主机的安全组打开这个端口范围就可以了，参考下图：
1. 启动命令
```
sudo docker run --rm -it -p 8021:21 -p 4559-4564:4559-4564 -e FTP_USER=test -e FTP_PASSWORD=test -v /home/test:/srv panubo/vsftpd
```
2. 云主机安全组范围
![](/15-Hours/_image/docker-vsftpd-03.png)
同时，由于容器里面都是只读文件系统，如果ftp要上传文件，需要从host挂载目录到容器中，否则上传会报533错误码。
挂载host文件的时候，要保证容器里面的用户有host机器上目录的写权限，如果只是测试使用，可以把host目录权限改成777验证。

