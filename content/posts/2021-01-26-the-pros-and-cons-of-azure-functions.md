---
title: 'Azure Functions 的优势与挑战'
summary: 'Azure Functions 在开发人员中越来越受欢迎，这些开发人员主要是 .NET 开发人员。它有哪些优缺点呢？赶紧来一探究竟！'
tags: ['azure', 'serverless']
date: 2021-01-26
author: donghui
featuredImagePreview: '/images/2021-01-26-the-pros-and-cons-of-azure-functions/azure.png'
---

> 原文：https://talkingserverless.com/2020/11/24/the-pros-and-cons-of-azure-functions/
> 
> 作者：rishidot
>
> 译者：donghui

![azure](/images/2021-01-26-the-pros-and-cons-of-azure-functions/azure.png)

### 引言
Azure Functions 在开发人员中越来越受欢迎，这些开发人员主要是 .NET 开发人员。Microsoft提供了许多不同的应用程序部署平台，包括容器、PaaS、WebApps、Azure Functions、Azure Logic Apps 等。借助如此多样化的产品组合，Microsoft 将 Azure Functions 定位为企业级的 Serverless 产品。该文章将重点介绍 Microsoft 的 Serverless 平台如何满足开发人员的需求。

与 AWS Lambda 不同，Microsoft 的 Serverless 产品在开发人员中的采用速度较慢，但该平台得到了熟悉 Microsoft 开发工具链的 .NET 开发人员的大力支持。Azure Functions 是一个支持 .NET、Java、Node.js 和 Python 的多语言平台。它与 Github、Visual Studio 和 Visual Studio Code 以及其 DevOps 产品 Azure Pipelines 很好地集成在一起。该平台支持从 Web 应用程序到 API 到机器学习工作流的各种用例。像 Catalyst 平台与 Zoho Create 低代码平台集成一样，Azure Functions 也与 Microsoft 低代码平台 Azure Power Apps 集成。

### Azure Functions 的优势
* 与 AWS 和 Google Cloud 的函数即服务（FaaS）产品相比，Azure 更加注重企业。它使用 “Durable Functions” 扩展为有状态应用程序提供支持。多数企业都希望使用 Serverless 部署有状态应用程序，并且该平台非常适合满足此需求。与对长时间运行任务的支持和对高成本计划中的实例支持相比，Microsoft 将 Azure Functions 定位为企业级无服务器平台。
* 支持 .NET 语言并与这些开发人员使用的工具包进行更深入的集成，Microsoft 瞄准了企业开发人员
* 默认情况下配置身份验证，从而消除了企业开发人员的额外开销
* 与托管的 Azure Functions 产品一起，可以将功能代码部署在 App Service（PaaS）、Kubernetes、Azure Stack 和 IoT Edge 上，从而使其成为用于云、混合云、本地、边缘和 IoT 部署的通用平台。


### Azure Functions 的挑战
* 定价模型很复杂。尽管高级计划和专用计划针对的是对成本不太严格的企业客户，但事实证明它们很昂贵。打开常驻实例设置将使与函数即服务相关的成本优势丧失。低端消费计划在支持的功能方面有一些相当严格的限制。此外，与存储相关的定价还不清楚，用户可能会因使用服务的方式而感到意外。提防隐藏成本，例如使用函数计算添加存储帐户。
* 使用消费计划进行扩展存在一些限制，甚至高级计划的应用程序函数数量上限也为100。选择正确的计划时要意识到这些限制。

### 总结
虽然 Azure Functions 更适合企业客户，但它不像市场上某些竞争产品那样对开发人员友好。由于复杂的定价模型和按需版本中的函数限制，Azure Functions 不适合单个开发人员和较小的公司。诸如 AWS Lambda 之类的产品适合这些开发人员。但是，如果您是使用 Azure 的企业客户或具有跨云、边缘和 IoT 的基础架构的企业客户，则 Azure Functions 非常适合您的需求。

