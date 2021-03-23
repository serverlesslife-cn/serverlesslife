---
title: 'Serverless Framework Pro 实践之 CI/CD'
summary: '了解下 Serverless Framework Pro 的 CI/CD 吧！'
tags: ['aws lambda', 'serverless framework', 'devops', 'ci', 'cd' ]
date: 2021-03-22
author: donghui
featuredImagePreview: '/images/2021-03-22-serverless-framework-pro-practice-ci-cd/serverless-framework-pro.png'
---

![cover](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/serverless-framework-pro.png)

Serverless Framework Pro 版是个 SaaS 应用，是个托管的 Dashboard。

本文主要实践 Serverless Framework Pro 的 CI/CD 功能。

## 配置 CI/CD 功能

CI/CD 功能是 Service 级别的，点击 Service 处的 settings 开始配置：

![settings](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/settings.png)

CI/CD 一边连接的是 Git 代码平台（目前支持 GitHub 和 BitBucket），另一边连接的是云服务（目前仅支持 AWS）。

![ci-cd](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/ci-cd-01.png)

代码工具这里选择 GitHub，点击 connect，会弹出一个对话框，需要 GitHub 进行授权，
授权后会在 GitHub 用户下安装 Serverless app：

![github-app](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/github-app.png)

在 git 和 aws 配置好后，选择代码仓库和 base 目录：

![repo-settings](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/repo-settings.png)

构建设置中，可以选择部署到哪个 region，也可以配置指定文件变化时才触发构建：

![build-settings](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/build-settings.png)

分支部署中，可以指定哪个分支部署到哪个 stage
（注意：branch 和 stage 都必须是目前存在的，如果新增了分支，就必须手动修改配置）：

![branch-settings](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/branch-settings.png)

预览部署，可以在创建 PR 时，自动部署一个环境，以便预览。

这个环境所在的 stage 名称和分支名称一样（注意：这里需要考虑预览环境和分支环境是否会覆盖的问题）

可以选择在分支删除时，删除对应的 stage 和资源；

也可以选择部署到指定的 stage，但是如果有多个到 master 分支的 PR，环境会互相覆盖。

![preview-deploys](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/preview-deploys.png)

通知支持在部署开始、结束时，发送一些消息：

![notify-settings](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/notify-settings.png)

## 触发 CI/CD，查看运行情况

根据上面配置的规则，让我们看下整个 CI/CD 流程。

* 往 dev 分支提交一下代码，便会自动部署到 stage： dev-stage；
* 创建一个 到 dev 分支到 master 分支的 Pull Request，便会自动部署到 stage：dev；
* 合并这个  Pull Request 到 master 分支便会将 master 分支部署到 stage：prod；
* 合并  Pull Request 后删除 dev 分支，便会删除 stage：dev 和对应资源。

点击 Serverless Dashboard 左侧 ci/cd 菜单，CI/CD 部署记录截图如下：

![deploys-history](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/deploys-history.png)

在每个 stage 的 deploys 页面，也可以看到部署记录：

![service-deploys](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/service-deploys.png)

GitHub 提交记录处，可以看到 CI/CD 部署记录链接，点击便可跳转到 Serverless Dashboard 页面：

![github-ci-cd](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/github-ci-cd.png)

此外，GitHub PR 页面也可以看到部署记录，预览部署截图如下：

![pr-ci-cd](/images/2021-03-22-serverless-framework-pro-practice-ci-cd/pr-ci-cd.png)

## 问题与思考
遇到的问题以及思考：
* 代码仓库下拉菜单无法获取某个 GitHub 组下面的仓库
* 官方博客推荐将不同的 stage 部署到不同的 aws 账户下，做到资源安全隔离。但是目前 Serverless Dashboard 并不支持不同的 stage 绑定不同的 aws provider。
* CI/CD 比较基础，无法做一些定制化操作。

## 总结
本文实践了如何在 Serverless Dashboard 配置 CI/CD，以及通过代码提交或 Pull Request 事件触发 CI/CD，完整体验了 CI/CD 流程。最后提出自己的一些观点。

## 参考
* https://www.serverless.com/cicd/
* https://www.serverless.com/learn/guides/cicd/
