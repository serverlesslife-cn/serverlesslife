---
title: 'Serverless Framework Pro 实践'
summary: '了解下 Serverless Framework Pro 吧！'
tags: ['aws lambda', 'serverless framework']
date: 2021-03-12
author: donghui
featuredImagePreview: '/images/2021-03-15-serverless-framework-pro-practice-01/serverless-framework-pro.png'
---

![cover](/images/2021-03-15-serverless-framework-pro-practice-01/serverless-framework-pro.png)

## 认识 Serverless Framework Pro
如果你了解 Serverless，那么说起  Serverless Framework，你应该也不陌生。

Serverless Framework 是业界非常受欢迎的 Serverless 应用框架，开发者无需关心底层资源即可部署完整可用的 Serverless 应用架构。

此前经常使用 Serverless Framework CLI 部署函数或应用。

Serverless Framework CLI 是个开源的 Serverless 命令行工具。

这里要提到的是，除了开源版外，Serverless Framework 还有 Pro 版。

![os-pro](/images/2021-03-15-serverless-framework-pro-practice-01/os-pro.png)

Serverless Framework Pro 版是个 SaaS 应用，是个托管的 Dashboard，与开源版相比，它提供了一些增强功能：
* “0”配置 debug
* CI/CD
* 故障排查
* 告警
* 安全保护
* ……

Pro 有三种版本：
* Free
* Team
* Enterprise

![price](/images/2021-03-15-serverless-framework-pro-practice-01/price.png)

目前 Pro 功能只适用于 AWS Lambda，并且运行时也只支持最流行的运行时：NodeJS 和 Python。

## 使用 GitHub 账号登陆 Serverless Framework Pro
下面来一步步免费体验下 Serverless Framework Pro。

![try](/images/2021-03-15-serverless-framework-pro-practice-01/try.png)

点击“Try Pro for free”，开始体验，便跳转到登陆页面

可以使用 github 账号和 google 账号登录，也可以使用用户名和密码登录（当然需要提前注册）

![login](/images/2021-03-15-serverless-framework-pro-practice-01/try.png)

这里使用 github 账号授权登陆，github 授权后填写一个用户名

![set-username](/images/2021-03-15-serverless-framework-pro-practice-01/set-username.png)

进入首页

![index](/images/2021-03-15-serverless-framework-pro-practice-01/index.png)

## 部署一个应用

点击”create app”或“deploy now”后，会出现一些模板

![template](/images/2021-03-15-serverless-framework-pro-practice-01/template.png)

这里选择 “python REST API”模板

![python-rest-api](/images/2021-03-15-serverless-framework-pro-practice-01/python-rest-api.png)

下面需要输入应用名，然后点 “deploy”

![deploy](/images/2021-03-15-serverless-framework-pro-practice-01/deploy.png)

下面给出了部署指引：
* 设置 aws credential
* 安装 serverless 命令行并初始化应用
* 使用 serverless deploy 命令部署应用

![deploy-now](/images/2021-03-15-serverless-framework-pro-practice-01/deploy-now.png)

注意：使用 serverless deploy 命令部署时，命令前面需要设置下参数，或者将其设置为环境变量：
* SERVERLESS_PLATFORM_VENDOR=aws，要设置下这个参数，否则国内默认部署到腾讯云
* AWS_ACCESS_KEY_ID=<your-key-here>
* AWS_SECRET_ACCESS_KEY=<your-secret-key-here>

获取 AWS Credential 请参考：https://www.serverless.com/framework/docs/providers/aws/guide/credentials/

下面是 serverless deploy 命令的输出日志，已成功部署应用到 aws：

![console](/images/2021-03-15-serverless-framework-pro-practice-01/console.png)

登陆 aws 控制台，可以看到上面部署的应用，应用下面有很多不同类型的资源：

![aws-app](/images/2021-03-15-serverless-framework-pro-practice-01/aws-app.png)

## 了解 Serverless Dashboard
打开 Serverless Dashboard 首页，可以看到应用列表，应用有三个层级（app、service、stage）：

![apps](/images/2021-03-15-serverless-framework-pro-practice-01/apps.png)

apps 菜单下具体某个应用页可以看到有很多的 Tab 页：
* overview
* explorer
* api endpoints
* functions
* deploys
* alerts
* outputs
* parameters
* providers

每个页面有很多指标图表，例如 overview 页面有如下图表：
* api requests
* api errors
* function invocations
* function errors
* api requests by endpoint
* invocations by function
* api errors by endpoint
* invocation errors by function
* timeouts by function
* cold starts by function

overview 页面截图如下：

![overview-1](/images/2021-03-15-serverless-framework-pro-practice-01/overview-1.png)

![overview-2](/images/2021-03-15-serverless-framework-pro-practice-01/overview-2.png)

explorer 页面截图如下，可以根据一定条件对 api 请求进行筛选：

![explorer](/images/2021-03-15-serverless-framework-pro-practice-01/explorer.png)

要强调下的是，在 explorer—> invocations 页面还可以看到冷启动相关的调用信息：

![invocations](/images/2021-03-15-serverless-framework-pro-practice-01/invocations.png)

点击调用历史中的某次调用可以进入详情页，例如下面是某次调用的详细信息，如果出现了问题，也很容易定位：

![detail](/images/2021-03-15-serverless-framework-pro-practice-01/detail.png)

api endpoints 页面截图如下，它从 api endpoint 的维度展示一些指标信息：

![api](/images/2021-03-15-serverless-framework-pro-practice-01/api.png)

functions 页面截图如下，它从函数的维度展示一些指标信息：

![function](/images/2021-03-15-serverless-framework-pro-practice-01/function.png)

deploys 页面截图如下，可以看到部署历史，还可以看到某次部署 serverless.yml 的变化：

![deploys](/images/2021-03-15-serverless-framework-pro-practice-01/deploys.png)

## 总结
本文首先介绍了 Serverless Framework Pro，然后使用 GitHub 账号登陆，接着通过它往AWS 部署了一个应用，最后对 Dashboard 进行了简单说明。
整体感觉 Serverless Framework Pro 对 AWS 支持的还是很友好的，值得尝试。
