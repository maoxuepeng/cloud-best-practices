---
date: 2019-8-6
title: 在Kubernetes中使用CRI-O运行时
tags: [Kubernetes, CRI-O, Runtime]
---

在[容器实践线路图](https://best.practices.cloud/2019/07/20/%E5%AE%B9%E5%99%A8%E5%AE%9E%E8%B7%B5%E8%B7%AF%E7%BA%BF%E5%9B%BE.html)中介绍了容器技术选型，关于容器运行时，提到了CRI规范与CRI-O实现，使用CRI-O可以在运行时完全替代docker。CRI-O提供了crictl工具，类似docker client，可以pull镜像、ps容器进程、attach到容器进程内等等，除了build与tag/push镜像没提供之外，其他都有了。至于为何不提供镜像build/tag/push操作，[官方解释是crictl不是替代docker](https://github.com/kubernetes-sigs/cri-tools/issues/438)，呵呵。
本文介绍如何使用CRI-O运行时替换docker运行时，基于CRI-O对接Kubernetes编排。本文所有环境都基于 CentOS7.6 操作系统，内核版本为 3.10.0-957.21.3.el7.x86_64 。 

## 1. 安装CRI-O

1. 启用内核模块

```bash
modprobe overlay
modprobe br_netfilter
```

2. 设置内核参数

```bash
# Setup required sysctl params, these persist across reboots.
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```

3. 安装CRI-O

```bash
# Install prerequisites
yum install yum-utils
yum-config-manager --add-repo=https://cbs.centos.org/repos/paas7-crio-311-candidate/x86_64/os/

# Install CRI-O
yum install --nogpgcheck cri-o
```

4. 配置 CRI-O，设置 crio pause 镜像下载地址为阿里云镜像仓库

默认配置在 ```/etc/crio/crio.conf``` 

```bash
sed -i "s/pause_image = .*/pause_image = \"registry.cn-hangzhou.aliyuncs.com\/google_containers\/pause:3.1\"/g" /etc/crio/crio.conf

```

5. 启动 CRI-O

```systemctl start crio```

6. 验证 CRI-O 部署

```bash
curl -v --unix-socket /var/run/crio/crio.sock http://localhost/info
```

```json
{"storage_driver":"overlay","storage_root":"/var/lib/containers/storage","cgroup_driver":"systemd","default_id_mappings":{"uids":[{"container_id":0,"host_id":0,"size":4294967295}],"gids":[{"container_id":0,"host_id":0,"size":4294967295}]}}s
```

## 2. 安装kubeadm/kubelet/kubectl

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

# 将 SELinux 设置为 permissive 模式(将其禁用)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable kubelet && systemctl start kubelet
```

不过谷歌的包仓库大陆无法访问，使用阿里云镜像站替换:

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 将 SELinux 设置为 permissive 模式(将其禁用)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

```

CRI-O 运行时使用的 ```cgroup driver``` 为 ```systemd``` ，因此需要设置 ```kubelet``` 参数保持一致：

```bash
echo "KUBELET_CGROUP_ARGS=--cgroup-driver=systemd" >> /etc/sysconfig/kubelet

# !!在 kubelet 启动文件 /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf 增加 KUBELET_CGROUP_ARGS 参数 !!
# ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS $KUBELET_CGROUP_ARGS

systemctl daemon-reload
systemctl enable kubelet
systemctl restart kubelet
```

**确保 kubelet 启动参数中有 KUBELET_CGROUP_ARGS 设置的值:**

```bash
# systemctl status -l kubelet
kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since ...
     Docs: https://kubernetes.io/docs/
 Main PID: 15971 (kubelet)
   CGroup: /system.slice/kubelet.service
           └─15971 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=remote --container-runtime-endpoint=/var/run/crio/crio.sock --cgroup-driver=systemd
```

否则后续会因 cgroup driver 不匹配(crio使用的cgroup driver是systemd, kubelet使用的是cgroupfs)报错:

```
kuberuntime_sandbox.go:68] CreatePodSandbox for pod "kube-scheduler-k8s-master-01_kube-system(...)" failed: rpc error: code = Unknown desc = cri-o configured with systemd cgroup manager, but did not receive slice as parent: /kubepods/burstable/...
```

## 3. 使用 kubeadm 安装 Kubernetes

1. 确定使用哪种网络插件实现，kubeadm 只支持 CNI 网络插件；这里使用 Flannel 插件

2. 修改 kubeadm 的默认仓库为阿里云仓库镜像

由于 Kubernetes 镜像仓库在 ```k8s.gcr.io``` 上，大陆无法访问，需要使用阿里云的镜像站，因此需要修改 kubeadm 默认配置。

```bash
# 导出默认配置
kubeadm config print init-defaults > kubeadm-init-config.yaml

#修改仓库镜像地址与kubernetes版本号
sed -i "s/imageRepository: .*/imageRepository: registry.cn-hangzhou.aliyuncs.com\/google_containers/g" kubeadm-init-config.yaml
sed -i "s/kubernetesVersion: .*/kubernetesVersion: v1.15.1/g" kubeadm-init-config.yaml
sed -i "s/criSocket: .*/criSocket: \/var\/run\/crio\/crio.sock/g" kubeadm-init-config.yaml
address=`ifconfig eth0 | egrep "inet\s" | awk '{print $2}'` && sed -i "s/advertiseAddress: .*/advertiseAddress: ${address}/g" kubeadm-init-config.yaml

# 在 kubeadm-init-config.yaml 增加 podSubnet: 参数，如
...
kind: ClusterConfiguration
networking:
  podSubnet: 10.244.0.0/16
...
```

修改后的 ```kubeadm-init-config.yaml``` :

```

apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.128.165
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/crio/crio.sock
  name: k8s-master-01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.15.1
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16
scheduler: {}

```

3. kubeadm init

```bash
kubeadm init  --config ~/kubeadm-init-config.yaml
```

如果 ```kubeadm init``` 命令出错了，修复之后重试，需要先执行 ```kubeadm reset``` 。

4. kubectl 连接到 Kubernetes 集群

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

如果使用 ```root``` 用户，则直接使用:

```bash
export KUECONFIG=/etc/kubernetes/admin.conf
```

5. 安装网络插件 Flannel 

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml
```

## 4. 增加 node 节点到集群

1. 在 node 上执行上面的大步骤1/2，安装 CRI-O 与 kubeadm/kubelet/crio-tools

2. 登录到node节点上，执行 join 的命令

```bash
kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

如果忘记了token，在Master节点上通过如下命令查看:

```bash
kubeadm token list
```

如果token过期了，在Master节点上通过如下命令重新生成:

```bash
kubeadm token create
```

```--discovery-token-ca-cert-hash``` 参数的值，在管理节点上通过如下命令获取:

```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

## 5. 部署一个 nginx 应用

[部署一个 nginx 应用](https://kubernetes.io/zh/docs/tasks/run-application/run-stateless-application-deployment/)

## Reference

[install-kubeadm](https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/)

[使用kubeadm 部署 Kubernetes(国内环境)](https://juejin.im/post/5b8a4536e51d4538c545645c)

[too hard to install k8s in china](https://github.com/kubernetes/kubernetes/issues/56038)