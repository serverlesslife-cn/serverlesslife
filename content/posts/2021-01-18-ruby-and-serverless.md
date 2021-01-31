---
title: '说说 Ruby 与 Serverless'
summary: '你所知道的那些 Faas 平台都支持 Ruby 了吗？'
tags: ['ruby', 'serverless']
date: 2021-01-18
author: donghui
featuredImagePreview: '/images/2021-01-18-ruby-and-serverless/ruby-cover.png'
---

![ruby](/images/2021-01-18-ruby-and-serverless/ruby-cover.png)

## 引言
最近查阅 Serverless 相关资讯，注意到一个 Ruby Serverless 框架——Jets。

心中便有一些疑问：为什么会有这个项目？它是用来做什么的？作为一门小众语言，有哪些 Serverlss 平台支持了 Ruby 语言？

既然有这么多疑问，于是就想理一理 Ruby 与 Serverless 之间的关系，便开始书写此文。

## Ruby 简介
Ruby，它由日本的松本行弘于1993年创立，它的 logo 是一颗闪亮、美丽的红宝石。

最近关于 Ruby 的最大新闻是：2020年12月25日，Ruby 3.0.0 正式发布。
Ruby 3.0.0 的目标是更高的性能、并发性和更安全的类型。尤其是在性能上，松本行弘表示「Ruby 3 会比 Ruby 2 快 3 倍」。

![ruby-3-news](/images/2021-01-18-ruby-and-serverless/ruby-3-news.png)

## 有哪些流行的项目是用 Ruby 开发的？
![ruby-apps](/images/2021-01-18-ruby-and-serverless/ruby-apps.png)

有哪些流行的项目是用 Ruby 开发的呢？这里首先要提的是全球著名的代码社交平台 Github，它是开源项目的沃土。

GitHub 最初就是使用 Ruby on Rails 构建的，它是 Ruby 社区创建的一个项目，Github 的流行 Ruby 社区功不可没。

值得一提的是，近些年，开发者在找工作时，有时 GitHub 开源项目经历也成了一个加分项。

类似的，作为 GitHub 的开源替代产品，Gitlab 支持私有化部署，它也是使用 Ruby on Rails 构建的。

如果要在内网搭建代码管理平台，GitLab 绝对是首选；曾经工作过的公司代码管理平台无一例外都是使用 GitLab 搭建的。

比较流行的 CI 服务，Travis CI 也是由 Ruby 开发的，它是一个托管的 CI 服务平台，与 GitHub 紧密集成。

使用过 GitHub 的开发者应该知道它，如果你在 GitHub 上有开源项目，就可以免费使用 Travis CI 构建自己的 CI 流水线。

Jekyll 是一个简单的博客形态的静态站点生成器，它也是使用 Ruby 开发的。使用 GitHub Pages + Jekyll，可以轻而易举地在 GitHub 上免费发布网站。

当然还有很多使用 Ruby 开发的项目，一些 Ruby 开发的其他服务或应用，可以参考：
`https://github.com/markets/awesome-ruby#services-and-apps`

## Ruby Serverless 框架——Jets
![ruby-jets](/images/2021-01-18-ruby-and-serverless/ruby-jets.png)

Jets 是一个 Ruby serverless 框架，可以让你轻松创建和部署服务。

它包括了构建 API 并将其部署到 AWS Lambda 所需要的一切。

Jets 也非常适合编写将 AWS 服务和资源粘合在一起的函数。

Jets 是一个脚手架，你只需要专注编写代码，Jets 会将代码转换为 Lambda 函数和其他 AWS 资源（如：API Gateway、S3、DynamoDB）。

AWS Lambda 支持许多事件触发器，下面是 Jets 支持的事件列表：
* CloudWatch Log Events
* CloudWatch Rule Events
* DynamoDB Events
* IoT Events
* Kinesis Events
* S3 Events
* SNS Events
* SQS Events
Jets 可以构建许多体系结构。下面是传统的 Web 架构示例，可以使用 Jets 轻松完成：

![ruby-jets-arch](/images/2021-01-18-ruby-and-serverless/ruby-jets-arch.png)

此外，Jets 的文档特别丰富，上手成本也低，对于 Ruby 开发者来说，绝对值得一试。

## 哪些 Serverless Faas 平台支持 Ruby？
平心而论，Ruby 是一门小众的编程语言，尤其是在国内，日常工作中很少用到。

据统计，Serverless Faas 最常用的语言是 NodeJS，其次是 Python，Ruby 用的少。

有哪些公有云 Serverless Faas 平台支持 Ruby 语言呢？

公有云 Serverless Faas 平台一般不会支持所有语言，但是它们大多支持 Custom Runtime（自定义语言）。

对于官方不支持的语言，就可以通过 Custom Runtime 来实现。

下面对各个公有云 Serverless Faas 平台支持的语言做一个梳理。

![ruby-faas](/images/2021-01-18-ruby-and-serverless/ruby-faas.png)

从上面的图表可以看出，上述 Faas 平台中 AWS Lambda 和 IBM Cloud Functions 官方提供了对 Ruby 的支持。

其中，在2018年11月29日，AWS Lambda 正式支持了 Ruby 2.5；2020年2月19日，支持了 Ruby 2.7。

这里还有一个 关于 Ruby 与 Serverless 的故事：在 AWS Lambda 未支持 Ruby 之前，Ruby 社区曾于2018年3月9日在 serverless-ruby.org 发布了一个请愿书，请求 Faas 平台支持 Ruby，共有1602个开发者签署了这个请愿书。

![ruby-want](/images/2021-01-18-ruby-and-serverless/ruby-want.png)

在 GitHub awslabs 组织的仓库中，有三个与 AWS Lambda Custom Runtime 相关的仓库，分别提供了对 Rust、C++ 和 Dart 的支持：
* https://github.com/awslabs/aws-lambda-rust-runtime
* https://github.com/awslabs/aws-lambda-cpp
* https://github.com/awslabs/aws-lambda-dart-runtime

关于 Azure Functions 对 Ruby 的支持，开发者在 GitHub 的 Azure-Functions 仓库提交了一个相关 issue：

Ruby lang. support #705  https://github.com/Azure/Azure-Functions/issues/705

或许不久的将来，将会看到基于 Custom Runtime 对 Ruby 的支持。

![ruby-azure](/images/2021-01-18-ruby-and-serverless/ruby-azure.png)

IBM Cloud Functions 基于 Apache OpenWhisk 搭建的，它是支持 Ruby 的。
除了官方支持的语言，它还支持使用 Docker 自定义运行环境。

腾讯云云函数 SCF，在官网上有一个关于 Custom Runtime 的示例：

Custom Runtime 创建 Bash 示例函数：https://cloud.tencent.com/document/product/583/47610

在 GitHub 上有两个开发者实现的 Custom Runtime 仓库，分别提供了对 Swift 和 .NET 的支持：
* https://github.com/stevapple/swift-tencent-scf-runtime
* https://github.com/dotnetcloudbase/dotnet-tencent-scf-runtime

目前没有在 GitHub 上找到关于 Ruby 的实现。

阿里云函数计算在官方文档中列出了一些基于 Custom Runtime 实现的语言（其中包含 Ruby）。

并且关于 Custom Runtime，阿里云函数计算有一个相关的 git 仓库：
https://github.com/awesome-fc/fc-custom-demo/

![ruby-aliyun](/images/2021-01-18-ruby-and-serverless/ruby-aliyun.png)

除了公有云的 Serverless Faas 平台，一些开源的 Faas 平台也提供了对 Ruby 的支持，如：Apache OpenWhisk、Kubeless、OpenFaas、Fission 等。

## 总结
本文首先介绍了 Ruby 以及使用 Ruby 构建的流行项目，紧接着介绍了 Ruby Serverless 框架——Jets，最后主要阐述了 Serverless Faas 平台对 Ruby 的支持情况，相信未来会有更多的 Faas 平台支持 Ruby。

## 参考
* https://github.com/boltops-tools/jets
* http://www.ruby-lang.org/zh_cn/news/2020/12/25/ruby-3-0-0-released/
* https://github.com/markets/awesome-ruby
* https://docs.aws.amazon.com/lambda/latest/dg/runtime-support-policy.html
* https://docs.aws.amazon.com/lambda/latest/dg/lambda-releases.html
* https://cloud.ibm.com/docs/openwhisk?topic=openwhisk-runtimes
* https://docs.microsoft.com/zh-cn/azure/azure-functions/supported-languages
* https://cloud.google.com/functions/docs/writing/
* https://cloud.tencent.com/document/product/583/9206
* https://help.aliyun.com/document_detail/74712.html
* https://cloud.baidu.com/doc/CFC/s/0k0rw0397#%E5%BC%80%E5%8F%91%E8%AF%AD%E8%A8%80
* https://support.huaweicloud.com/devg-functiongraph/functiongraph_02_0101.html
* https://www.serverless-ruby.org/
* https://github.com/apache/openwhisk-runtime-ruby
* https://kubeless.io/docs/runtimes/
* https://docs.openfaas.com/cli/templates/#ruby
* https://github.com/fission/environments
