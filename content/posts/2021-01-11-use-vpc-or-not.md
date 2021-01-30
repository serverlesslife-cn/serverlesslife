---
title: '在函数计算中到底该不该使用 VPC？'
summary: '如非必要，请不要在函数计算中使用 VPC'
tags: ['vpc', 'serverless']
date: 2021-01-11
author: donghui
featuredImagePreview: '/images/2021-01-11-use-vpc-or-not/vpc.png'
---

![vpc](/images/2021-01-11-use-vpc-or-not/vpc.png)

### 什么是 VPC？
VPC 全称 Virtual Private Cloud，中文翻译过来是虚拟私有云，但也有的云厂商将其译为私有网络或专有网络。

比如：华为云将 VPC 译为虚拟私有网络；阿里云、腾讯云、百度智能云将 VPC 译为私有网络，新网将 VPC 译为专有网络。

使用 VPC 技术，可以在公有云上，构建一个隔离的、私密的网络环境。并可以在 VPC 中定义安全组、子网、IP地址段、路由表、带宽等网络特性。不同 VPC 之间完全逻辑隔离。此外，VPC 还支持多种方式连接 Internet，如弹性 IP 、NAT 网关等。

### 函数是否需要配置 VPC？
函数计算中，如果需要访问云上的其他服务（如：数据库、Redis 等），并且这些服务只有内网可以访问，那么就需要配置 VPC，将函数与其他服务部署到同一个 VPC。

下面是来自阿里云函数计算官方文档中，关于函数是否需要使用 VPC 功能的决策流程：

![aliyun-vpc](/images/2021-01-11-use-vpc-or-not/aliyun-vpc-0.png)


### 函数配置 VPC 带来的影响
函数配置 VPC 最大的影响是会带来冷启动，增加函数启动时间。冷启动是函数计算最大的挑战之一。如非必要，请不要使用 VPC。

### 函数计算如何开启 VPC？
各种函数计算服务都是如何开启 VPC 的呢？下面分别以 AWS Lambda、腾讯云云函数 SCF、阿里云函数计算为例进行说明。


#### AWS Lambda 开启 VPC：
单击左侧导航栏的【函数服务】，选择某个函数，配置—>VPC，点击【编辑】，选择所需选项（vpc 、子网、安全组）。

![lambda-vpc](/images/2021-01-11-use-vpc-or-not/lambda-vpc.png)

#### 腾讯云云函数 SCF 开启 VPC：
单击左侧导航栏的【函数服务】，选择某个函数，函数管理—>函数配置—>网络配置，启用【私有网络】，选择所需选项（vpc 和子网）。SCF 支持公网和私有网络 VPC 同时开启，并且支持固定出口 IP。

![scf-vpc](/images/2021-01-11-use-vpc-or-not/scf-vpc.png)

#### 阿里云函数计算开启 VPC：
单击左侧导航栏的【服务及函数】，选择某个服务，服务配置—>网络配置，勾选【允许函数访问 VPC 内资源】，选择所需选项（专有网络、交换机、安全组）。

![aliyun-vpc](/images/2021-01-11-use-vpc-or-not/aliyun-vpc-1.png)

从上面可以看出，不同云厂商的函数计算服务，开启 VPC 的方式不尽相同，所需要的 VPC 选项也不完全一样。
阿里云函数计算的配置是以服务为维度的，服务是函数的逻辑分组；腾讯云云函数和 AWS Lambda 的配置都是以函数为维度的，它们都是通过标签属性对函数进行标记、分类、过滤的。

### 总结
本文首先介绍了什么是 VPC，然后说明了函数计算中何时需要配置 VPC 以及其影响，最后举例说明了云厂商如何配置 VPC。
总而言之，一句话：“如非必要，请不要在函数计算中使用 VPC”。

### 参考
* https://lumigo.io/aws-lambda-deployment/lambda-vpc/
* https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html
* https://cloud.tencent.com/document/product/583/19703
* https://help.aliyun.com/document_detail/72959.html

