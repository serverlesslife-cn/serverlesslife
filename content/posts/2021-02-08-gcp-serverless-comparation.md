---
title: 'Google 的 Serverless 产品对比：Cloud Run、Cloud Functions、App Engine'
summary: '如何在在 Cloud Run、Cloud Functions 和 App Engine 之间进行选择？'
tags: ['google', 'serverless']
date: 2021-02-08
author: donghui
featuredImagePreview: '/images/2021-02-08-gcp-serverless-comparation/gcp-serverless.png'
---

![cover](/images/2021-02-08-gcp-serverless-comparation/gcp-serverless.png)

> 来源：https://www.splunk.com/en_us/blog/devops/gcp-serverless-comparison.html
>
> 作者：Splunk
>
> 译者：donghui

Serverless 平台的主要优点是，它们使您可以专注于编写代码，而不必关心管理基础结构，自动扩容或为所用资源支付更多费用。

这使得 Serverless 计算非常适合以下用例：
* 无状态 HTTP 应用程序
* Web 和移动后端
* 实时的或事件驱动的数据处理
 

Cloud Run、Cloud Functions 和 App Engine 都是 Google Cloud 提供的 Serverless 平台，但是它们之间有细微差别，在某些情况下某个平台可能会比其他平台更受欢迎。

## Google Cloud Run：Serverless 容器
Cloud Run 由 Knative 构建， 是 Google 最新的 Serverless 产品。其他 Serverless 平台使用事件驱动函数作为部署的主要单元，而 Cloud Run 使您可以将代码打包在无状态容器中，然后通过 HTTP 请求调用它。

在 Google 完全托管环境中部署 Cloud Run 容器可为开发人员提供 Serverless 的通常优势（无需管理基础架构，按使用付费，更容易自动缩放），还支持任意数量的编程语言、库或系统二进制文件。Cloud Run 还可以在 Google Kubernetes Engine（GKE）上部署容器，并能够为后一种场景的 Serverless 容器专门配置硬件需求。

有了这种灵活性，Cloud Run 的用户可以使用他们已经用来在 Google Cloud 上打包和运行容器的工具轻松地运行 Serverless 工作负载，或者将有状态和无状态工作负载一起部署。

## Google Cloud Functions: Serverless 函数
尽管 Cloud Run 接受容器并通过 HTTP 请求来调用，但 Cloud Functions  仍然是 Google 的事件驱动型 Serverless 平台。与打包在 Docker 容器中不同，您需要将代码部署为函数。Google 支持编写 Cloud Functions，因此也可以通过 HTTP 请求调用它们，或将其设置为根据后台事件触发。
```python
def hello_get(request):

    """HTTP Cloud Function.

    Args:

        request (flask.Request): The request object.

        <http://flask.pocoo.org/docs/1.0/api/#flask.Request>

    Returns:

        The response text, or any set of values that can be turned into a

        Response object using `make_response`

        <http://flask.pocoo.org/docs/1.0/api/#flask.Flask.make_response>.

    """ 

return 'Hello World!'
```

Cloud Functions 对代码的部署方式施加了更多限制（显然易见，您需要将其打包为一个函数），并且仅支持一组特定的语言（您可以使用 JavaScript、Node.js、Python 3，或 Go 运行时），但可以使用您的云环境中的事件触发功能。


## Google App Engine: Serverless 应用
App Engine 是 Google 针对 Web 和 API 后端的完全托管的 Serverless 应用程序平台。尽管 Serverless 函数使您可以轻松地运行轻量级和独立的函数，但使用 Cloud Functions 运行更复杂的应用程序可能会很困难。对于想要构建具有多种功能的 Serverless 应用程序或保留超出单个请求范围的某种程度的上下文的开发人员，Google App Engine 提供了一种引人注目的选择。

在 Google App Engine 中，您只需获取代码并将其部署到 Google 上，然后为您消耗的资源付费-这在 App Engine 上作为包含一个或多个服务的单个资源运行。对于每种服务，您都可以部署该服务的一个或多个版本，这些版本又可以在一个或多个实例中运行，具体取决于每个版本处理的流量。

![app-engine](/images/2021-02-08-gcp-serverless-comparation/app-engine.jpg)

 如上所示，使用单个命令从您的应用程序目录在 Google App Engine 上部署 Hello World。

根据您的特定需求，您可以在两种类型的 App Engine 环境中选择一种来运行代码。如果您要运行需要快速扩容的应用程序，并且使用 App Engine 支持的特定语言版本编写，那么 Google 建议您使用标准环境。对于具有更稳定流量的应用程序，使用自定义运行时或不受支持的编程语言在 Docker 容器中运行，或者要访问在运行在 Compute Engine 上的 Google Platform 项目的其他部分，请使用 App Engine 灵活环境。

## 在 Cloud Run、Cloud Functions 和 App Engine 之间进行选择
通常，Serverless 平台最好用于构建无状态应用程序，并且无需管理基础架构。一些示例包括：
* 快速制作功能原型
* 快速自动缩放 Web 应用程序
* 为了响应后台事件执行一个任务
 
在确定哪种 Serverless 平台最适合您时，请记住以下几点：
* 如果您已经将代码打包在 Docker 容器中或正在 Google Cloud 中运行 Kubernetes 集群，请针对您的 Serverless 工作负载考虑使用 Cloud Run 或 Knative。
* 对于运行响应实时事件的代码，或在不使用容器的情况下处理请求，请使用 Cloud Functions。
* 如果您需要在一个地方放置多个函数并且只想部署整个应用程序，请使用 App Engine。