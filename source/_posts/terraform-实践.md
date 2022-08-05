---
title: Terraform 实践
tags:
  - Terraform
  - 运维
  - 阿里云
id: '12360'
categories:
  - - skills
date: 2021-03-10 10:20:31
---

# Terraform 介绍

## 什么是 Terraform

Terraform是一种开源工具，用于安全高效地预览，配置和管理云基础架构和资源。

### 概览

[HashiCorp Terraform](https://www.terraform.io/?spm=a2c4g.11186623.2.4.35fd376d540nB0 "HashiCorp Terraform") 是一个IT基础架构自动化编排工具，可以用代码来管理维护 IT 资源。Terraform的命令行接口（CLI）提供一种简单机制，用于将配置文件部署到阿里云或其他任意支持的云上，并对其进行版本控制。它编写了描述云资源拓扑的配置文件中的基础结构，例如虚拟机、存储帐户和网络接口。 Terraform是一个高度可扩展的工具，通过 Provider 来支持新的基础架构。Terraform能够让您在阿里云上轻松使用 简单模板语言 来定义、预览和部署云基础结构。您可以使用Terraform来创建、修改、删除ECS、VPC、RDS、SLB等多种资源。
<!--more-->
### 优势

**将基础结构部署到多个云** Terraform适用于多云方案，将类似的基础结构部署到阿里云、其他云提供商或者本地数据中心。开发人员能够使用相同的工具和相似的配置文件同时管理不同云提供商的资源。 **自动化管理基础结构** Terraform能够创建配置文件的模板，以可重复、可预测的方式定义、预配和配置ECS资源，减少因人为因素导致的部署和管理错误。能够多次部署同一模板，创建相同的开发、测试和生产环境。 **基础架构即代码（Infrastructure as Code）** 可以用代码来管理维护资源。允许保存基础设施状态，从而使您能够跟踪对系统（基础设施即代码）中不同组件所做的更改，并与其他人共享这些配置 。 **降低开发成本** 您通过按需创建开发和部署环境来降低成本。并且，您可以在系统更改之前进行评估。

## 应用场景

Terraform可以对基础设施进行编码，利用代码来进行资源的增删查改。 **创建基础设施** 您可以使用Terraform创建和管理ECS、VPC和SLB等基础资源。 **均衡负载业务流量** 您可以将访问流量按照定义的转发规则分发到指定的后端服务器（ECS实例），提高应用系统对外的服务能力，消除单点故障。 **自动伸缩** 根据您的业务需求和策略自动调整弹性计算资源，在业务需求增长时无缝增加ECS实例满足计算需要，在业务需求下降时自动减少ECS实例节约成本。 **集群管理** 您可以使用Terraform快速创建专有网络的集群。 在阿里云中启动kubernetes集群，并且在集群中创建VPC、交换机和NAT网关等资源，请参见示例模板kubernetes module。 **配置函数计算服务** 阿里云函数计算是事件驱动的全托管计算服务。通过函数计算，您无需管理服务器等基础设施，只需编写代码并上传。借助于函数计算，您可以快速构建任何类型的应用和服务，无需管理和运维。

# 安装和配置 Terraform

## 在阿里云 Cloud Shell 中使用 Terraform

阿里云 Cloud Shell 集成了 Terraform 组件：https://help.aliyun.com/document\_detail/95841.html?spm=a2c4g.11186623.6.545.4e13190cErHLYf

## 在本地安装和配置 Terraform

### 准备

云：阿里云 机器信息： [![](https://i.loli.net/2021/03/10/2hVQR16KHNmcPxM.jpg)](https://i.loli.net/2021/03/10/2hVQR16KHNmcPxM.jpg)

### 安装

下面是 Centos 7安装，其它操作系统请参照官网：https://www.terraform.io/docs/cli/install/yum.html

```bash
# Centos
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum install terraform


```

安装好后输入 terrafrom 命令出现如下内容即为安装成功 [![](https://i.loli.net/2021/03/10/HRvLJSMqE8OVKWI.jpg)](https://i.loli.net/2021/03/10/HRvLJSMqE8OVKWI.jpg)

### 在阿里云创建 RAM 用户

**登录 RAM 控制台：** https://ram.console.aliyun.com/?spm=a2c4g.11186623.2.8.33fd16f2IgYtaY#/overview **创建名为Terraform的RAM用户，并为该用户创建AccessKey：** [![](https://i.loli.net/2021/03/10/MJBos1pK7muZkfw.jpg)](https://i.loli.net/2021/03/10/MJBos1pK7muZkfw.jpg) 创建如下： [![](https://i.loli.net/2021/03/10/QXDa6Ocjset35iw.jpg)](https://i.loli.net/2021/03/10/QXDa6Ocjset35iw.jpg) **添加用户权限** [![](https://i.loli.net/2021/03/10/wZdFzlI4ACby8QL.jpg)](https://i.loli.net/2021/03/10/wZdFzlI4ACby8QL.jpg) 我这里是测试所以就加了所有权限，生产环境请根据实际情况添加 [![](https://i.loli.net/2021/03/10/khAOdc8ICTV54bE.jpg)](https://i.loli.net/2021/03/10/khAOdc8ICTV54bE.jpg)

### 添加环境变量

```bash
sudo vim /etc/profile
# 添加如下内容
# key secret 填写自己 RAM 用户生成的，REGION 为区域
export ALICLOUD_ACCESS_KEY="LTAI4G7W8JG*********"
export ALICLOUD_SECRET_KEY="8eMc5Y0SfHPXppigtb******"
export ALICLOUD_REGION="cn-hangzhou"


# 使配置生效
source /etc/profile
```

# 实践

```bash
# 创建执行目录，后续操作在下面目录中执行
mkdir /opt/terraform
cd /opt/terraform
# 初始化
terraform init
```

## 云服务器 ECS

### 创建一台 ECS 实例

*   创建专有网络和交换机

```bash
vim terraform.tf
# 输入以下内容
resource "alicloud_vpc" "vpc" {
  name       = "tf_test_foo"
  cidr_block = "172.16.0.0/12"
}
resource "alicloud_vswitch" "vsw" {
  vpc_id            = alicloud_vpc.vpc.id
  cidr_block        = "172.16.0.0/21"
  availability_zone = "cn-hangzhou-b"
}
#
# 创建
Terraform apply
# 命令行查看或者登陆控制台查看
terraform show
```

[![](https://i.loli.net/2021/03/10/r2RO56ilqyLcupH.jpg)](https://i.loli.net/2021/03/10/r2RO56ilqyLcupH.jpg)

*   在上一部创建的专有网络中创建一个安全组，并添加一个允许任何地址访问的安全组规则 在 terraform.tf 文件中添加以下内容

```text
resource "alicloud_security_group" "default" {
  name = "default"
  vpc_id = alicloud_vpc.vpc.id
}

resource "alicloud_security_group_rule" "allow_all_tcp" {
  type              = "ingress"
  ip_protocol       = "tcp"
  nic_type          = "intranet"
  policy            = "accept"
  port_range        = "1/65535"
  priority          = 1
  security_group_id = alicloud_security_group.default.id
  cidr_ip           = "0.0.0.0/0"
}


```

运行terraform apply开始创建

*   创建 ECS 实例 在 terraform.tf 文件中添加以下内容

```text
resource "alicloud_instance" "instance" {
  # cn-beijing
  availability_zone = "cn-hangzhou-b"
  security_groups = alicloud_security_group.default.*.id

  # series III
  instance_type        = "ecs.n2.small"
  system_disk_category = "cloud_efficiency"
  image_id             = "ubuntu_18_04_64_20G_alibase_20190624.vhd"
  instance_name        = "test_foo"
  vswitch_id = alicloud_vswitch.vsw.id
  internet_max_bandwidth_out =10
  password = "P@ss12345"
}
```

运行terraform apply开始创建,创建完成控制台 [![](https://i.loli.net/2021/03/10/tLDSP7sNM4JVX6H.jpg)](https://i.loli.net/2021/03/10/tLDSP7sNM4JVX6H.jpg) 以上就是创建 ECS 的操作，其它操作可以查看阿里云官网文档： https://help.aliyun.com/product/95817.html?spm=a2c4g.11186623.6.540.423c278brGHPCC

# 参考

## Terraform 常用命令

https://help.aliyun.com/document\_detail/145531.html?spm=a2c4g.11174283.6.573.4e9511e91k8F33