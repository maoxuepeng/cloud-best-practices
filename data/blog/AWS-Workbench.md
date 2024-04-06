---
date: 2021-09-13
title: AWS-Workbench
tags: [Cloud, AWS, Management&Goverance]
---

在[AWS组织（Organization）应用场景](https://best.practices.cloud/2021/08/08/AWS%E7%BB%84%E7%BB%87(Organization)%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF.html)与[与AWS Organization集成的云服务介绍](https://best.practices.cloud/2021/08/15/%E4%B8%8EAWS-Organization%E9%9B%86%E6%88%90%E7%9A%84%E4%BA%91%E6%9C%8D%E5%8A%A1%E4%BB%8B%E7%BB%8D.html)两篇文章中，介绍了AWS组织与配合组织使用的云服务，汇总组织与这些云服务配合使用所提供的功能，我们可以推导出其背后的动机，是为了给各种级别的"组织"提供满足其需要的**数字化平台**。

## 数字化平台

AWS组织与管理&治理服务组合使用，满足企业与组织对数字化平台的需求。分析AWS管理与治理类别的服务，得出组合这些服务使用的场景：3组功能，支撑3个场景。

![](/images/aws/aws-management-and-govern.png)

其中，工作台场景，AWS提供 Service Workbench解决方案以及参考开源实现。企业、组织机构可参考此开源实现，在AWS上构建自有的数字平台。

## AWS service workbench

### 功能介绍

Service Workbench on AWS 是一项新的 AWS 解决方案实施，可帮助 IT 团队以安全、可重复和联合的方式提供研究人员所需的数据、工具和计算能力的访问权限控制。

#### 功能

借助 Service Workbench on AWS，研究人员不必再为配置和管理云基础设施担忧。相反，他们可以专注于研究任务，并在几分钟（而不是几个月）之内完成基础工作。研究人员可以使用 Service Workbench on AWS 快速、安全地构建研究环境，并与机构内部和跨机构的同行共享数据。通过自动创建基准研究环境，简化数据访问，并提供成本透明度，研究人员和 IT 部门可以节省时间，并实现研究可再现性。

- 极简入驻：支持创建本地用户、对接Idp，方便用户接入。
- 全面管控：Organization+IAM实施策略与权限管控，Service Catalog实施云资源与服务目录管控。
- 协同工作：提供workspace type（Catalog里的Product）与workflow（基于Step Function编排工作流），支撑多人协同工作。
- DevOps：提供ci/cd流水线。
- 可扩展：workspace type（Catalog Product）、工作台界面、工作流等可以定制与扩展。

#### 组件与部署架构

![](/images/aws/ServiceWorkbenchOnAWSArchitectureDiagram.png)


### 安装

Workbench 是一个Serverless应用，通过 [Serverless Framework](https://serverless.com/) 部署。部署过程中需要一台客户端机器，官方建议的是一台EC2。

#### 安装必要工具

1. 安装 node 以及 serverless framework 工具

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
source ~/.bashrc
nvm install 12
npm install -g serverless pnpm hygen

```

2. 安装Go

```bash
sudo rpm --import https://mirror.go-repo.io/centos/RPM-GPG-KEY-GO-REPO
sudo curl -s https://mirror.go-repo.io/centos/go-repo.repo | tee /etc/yum.repos.d/go-repo.repo
sudo yum install golang

```

3. 配置 serverless 工具连接到 AWS

```bash
serverless config credentials --provider aws --key <key> --secret <secret>
```

4. 安装镜像打包工具 [packer](https://www.packer.io/)

```bash
wget https://releases.hashicorp.com/packer/1.7.4/packer_1.7.4_linux_amd64.zip
unzip packer_1.7.4_linux_amd64.zip
cp packer /usr/local/bin
```

#### 安装 Workbench

1. 下载 Workbench安装包

```bash
wget https://github.com/awslabs/service-workbench-on-aws/archive/refs/tags/v3.3.1.zip -o serviceworkbench.zip
unzip serviceworkbench.zip
```

2. [配置环境参数](https://github.com/awslabs/service-workbench-on-aws/blob/mainline/docs/docs/installation_guide/installation/pre-installation/conf-settings.md)

```bash
STAGE_NAME=dev
WORKSPACE=sw
cp /home/ec2-user/service-workbench-on-aws-3.3.1/main/config/settings/example.yml /home/ec2-user/service-workbench-on-aws-3.3.1/main/config/settings/dev.yml
```

3. 安装 Workbench

```bash
export STAGE_NAME=dev
cd /home/ec2-user/service-workbench-on-aws-3.3.1
./script/environment-deploy.sh ${STAGE_NAME}
```

4. 制作AMI（可以与上一步并行）

```bash
pnpx sls build-image -s ${STAGE_NAME}
```

5. 获得访问地址

步骤3安装完成后，通过如下命令获得访问地址。

```bash
./home/ec2-user/service-workbench-on-aws-3.3.1/main/script/get-info.sh ${STAGE_NAME}
```

### 配置

#### Idp

1. 在 AWS Contigo 控制台创建用户池（User Pool）


2. 在 Auth0 上配置 Contigo 为 SP（Service Provider）

- ```USER_POOL_ID``` 为 Contigo 的用户池ID。
- ```STAGE_NAME```, ```SOLUTION_NAME``` 为 ${STAGE_NAME}.yml 中的配置项。

```json
{
"audience": "urn:amazon:cognito:sp:POOL_ID",
"Mappings": {
"Email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
"name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
"given_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname",
"family_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname"
},
"logout": {
"callback": "https://STAGE_NAME-SOLUTION_NAME.auth.REGION.amazoncognito.com/saml2/logout"
},
"nameIdentifierFormat": "urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"
}
```

https://dev-sw.auth.ap-southeast-1.amazoncognito.com/saml2/idpresponse

3. 配置SAML2文件

将从 Auth0 下载的 SAML2 配置文件放到如下目录下

```bash
cp main/solution/post-demployment/config/saml-metadata/auth0_com-metadata.xml
```

4. 部署

```bash
bash script/environment-deploy.sh ${STAGE_NAME}
```

### 升级

### 卸载

1. 删除3个CloudFormation堆栈。

2. 删除s3桶

```bash
aws s3 ls | awk '{print $3}' | egrep '' | > /tmp/s3.txt
while read x; do aws s3 rb s3://${x} --dryrun; done </tmp/s3
while read x; do aws s3 rb s3://${x} ; done </tmp/s3
```

3. 删除 servicecatalog 中的 portfolio 与 product 

```bash
aws servicecatalog search-products-as-admin
```


## Reference

[workbench-on-aws](https://github.com/awslabs/service-workbench-on-aws)

[Service Workbench on AWS](https://aws.amazon.com/cn/solutions/implementations/service-workbench-on-aws/)