---
title: '使用 CODING DevOps 部署 Serverless 应用'
summary: '本文讲述如何使用 CODING DevOps 自动部署 Serverless 应用。'
tags: ['serverlesslife', 'serverless framework', 'coding devops']
date: 2021-03-07
author: donghui
featuredImagePreview: '/images/2021-03-08-use-coding-devops-deploy-serverless-application/cover.png'
---

![cover](/images/2021-03-08-use-coding-devops-deploy-serverless-application/cover.png)

## 前言
2021年年初，使用 Serverless Framework  在提腾讯云上部署了一个个人博客：serverlesslife.cn。

源码托管在 GitHub 上：[https://github.com/serverlesslife-cn/serverlesslife](https://github.com/serverlesslife-cn/serverlesslife)

本文将讲述如何使用 CODING DevOps 自动部署 Serverless 应用。

## 从 GitHub Actions 到 CODING DevOps
CI/CD 已成为软件开发环节的标配，倡导将一切自动化，这里期待在代码提交到 master 分支后就会自动部署应用。

因为代码 托管在 GitHub 上，首先考虑的是使用 GitHub Actions 部署应用。
然而在使用 GitHub Actions 时，总是会超时失败，这是因为 GitHub Actions 官方托管服务器在国外，在部署到国内的环境时，网络延迟很大，从而导致失败。

于是暂时放弃使用 GitHub Actions 部署战点，并考虑使用国内的免费 CI/CD 工具，在调研后选择了腾讯旗下的 CODING DevOps。

考虑到国内拉取 GitHub 代码会比较慢，这里首先使用 GitHub Actions 将代码自动同步到 coding.net 的代码仓库，然后再使用 CODING 持续集成进行自动化部署。

## 使用 GitHub Actions 将代码自动同步到 CODING
GitHub Actions 有一个特别好的功能是：有一个 GitHub Marketplace，目前有 7000 多个 Action，开发者可以从中挑选适合自己的 Action。
开发者也可以定义自己的 Action，也可以将自己的 Action 发布到 GitHub Marketplace。
这里定制了一个 Action 用于同步代码到 CODING：[https://github.com/marketplace/actions/sync-repo-to-coding](https://github.com/marketplace/actions/sync-repo-to-coding)

每个仓库可以配置多个 workflow，在 GitHub 仓库的 .github/workflows 下添加用来定义 workflow 的 YAML 文件即可。
同步代码到 CODING 的 workflow 的 YAML 文件内容如下：
```
name: sync-to-coding

on:
  push:
    branches:
      - master  # Set a branch to deploy

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
    - name: Sync to CODING
      uses: serverlesslife-cn/sync-repo-to-coding@master
      env:
          # 在 GitHub Settings->Secrets 配置 CODING_PRIVATE_KEY
          SSH_PRIVATE_KEY: ${{ secrets.CODING_PRIVATE_KEY }}
      with:
          # 注意替换为你的 GitHub 源仓库地址
          github-repo: "git@github.com:serverlesslife-cn/serverlesslife.git"
          # 注意替换为你的 CODING 目标仓库地址
          coding-repo: "git@e.coding.net:donghui1/serverlesslife/serverlesslife.git"
```

每次提交代码到 master 分支后，都会触发 sync-to-coding 任务，Actions 日志截图如下：

![github-actions-log](/images/2021-03-08-use-coding-devops-deploy-serverless-application/github-actions-log.png)


## 准备 CI 所需的 Docker 镜像
CODING 构建部署过程中需要用到 Docker 镜像，如：hugo 和 serverless，分别用于构建与部署阶段。

从 DockerHub 挑选了下面符合要求的两个镜像：
* https://hub.docker.com/r/cibuilds/hugo/
* https://hub.docker.com/r/amaysim/serverless/

然后从 DockerHub 下载到本地，再手动上传到了 CODING 的 Docker 制品仓库。
这样在构建过程中可以快速下载镜像（如果从 DockerHub 下载，不仅速度慢，还有下载次数限制）。

![docker-image](/images/2021-03-08-use-coding-devops-deploy-serverless-application/docker-image.png)


## 配置 CODING 持续集成构建计划
CODING 持续集成功能是基于 Jenkins 二次开发的，支持 Jenkins Pipeline。
如果熟悉 Jenkins，那么上手 CODING 持续集成就会很容易。
值得一提的是 CODING 持续集成提供了图形化编辑生成 Jenkinsfile 的功能，大大降低了使用成本。
当然图形化编辑器也有美中不足之处，它不会支持所有 Jenkins 步骤。

下面是使用图形化编辑器可视化编辑 Jenkins Pipeline 的截图：

![pipeline](/images/2021-03-08-use-coding-devops-deploy-serverless-application/pipeline.png)

添加所需的环境变量：
这里需要添加两个环境变量：TENCENT_SECRET_ID、TENCENT_SECRET_KEY，用于登录腾讯云。
为了避免密码明文显示在控制台，添加这里的环境变量时要勾选「保密」。

![env](/images/2021-03-08-use-coding-devops-deploy-serverless-application/env.png)

![env](/images/2021-03-08-use-coding-devops-deploy-serverless-application/env-2.png)


最终的 Jenkinsfile 内容如下：
```
pipeline {
  agent any
  stages {
      stage('Checkout') {
        steps {
          checkout([
            $class: 'GitSCM',
            branches: [[name: GIT_BUILD_REF]],
            userRemoteConfigs: [[
              url: GIT_REPO_URL,
              credentialsId: CREDENTIALS_ID
            ]]])
          }
      }

      stage('EnvSetUp') {
        steps {
          sh 'touch .env'
          sh 'echo TENCENT_SECRET_ID=${TENCENT_SECRET_ID} >> .env'
          sh 'echo TENCENT_SECRET_KEY=${TENCENT_SECRET_KEY} >> .env'
        }
      }

      stage('Build') {
        agent {
          docker {
            reuseNode 'true'
            registryUrl 'https://donghui1-docker.pkg.coding.net'
            registryCredentialsId "${env.DOCKER_REGISTRY_CREDENTIALS_ID}"
            image 'serverlesslife/hugo/hugo:latest'
          }
        }
        steps {
          sh 'HUGO_ENV=production hugo'
        }
      }


      stage('Deploy') {
        agent {
          docker {
            reuseNode 'true'
            registryUrl 'https://donghui1-docker.pkg.coding.net'
            registryCredentialsId "${env.DOCKER_REGISTRY_CREDENTIALS_ID}"
            image 'serverlesslife/serverless/serverless:2.28.0'
            args '-e TZ="Asia/Shanghai"'
          }
        }
        steps {
          sh 'serverless deploy'
        }
      }
  }
}
```

对于 Jenkinsfile 做如下说明：
* pipeline 由 agent 、stages 和 post 组成，其中 stages 下包括一系列 stage，而 stage 下又有 steps，steps 下则是一些指令
* stages 下包括多个 stage：Checkout、EnvSetUp、Build、Deploy
    * Checkout 阶段用于检出代码，这里是私有仓库，用到了 Credentials 插件
    * EnvSetUp 阶段用于生成 .env 文件，用于自动登录腾讯云
    * Build 阶段主要是用于构建，这里使用 hugo 镜像以及 hugo 命令进行构建
    * Deploy 阶段则是使用 serverless 镜像以及 serverless deploy 命令部署应用，其中 docker args 中 TZ="Asia/Shanghai” 表示设置时区中国区，这样 serverless 命令行将会将应用部署到腾讯云，否则默认部署到 AWS
* 需要特别说明的是用于拉取代码的 CREDENTIALS_ID 环境变量和用于拉取 Docker 镜像的 DOCKER_REGISTRY_CREDENTIALS_ID 是 CODING 持续集成平台提供的，无需手动设置

设置触发规则：
这里设置在代码推送到 master 分支时，自动触发构建。

![trigger-rule](/images/2021-03-08-use-coding-devops-deploy-serverless-application/trigger-rule.png)


至此，整个持续集成构建计划的配置就完成了。

此后，每次提交代码到 GitHub 仓库的 master 后，GitHub Actions 便会将代码自动同步到 coding 的代码仓库，然后便会触发 CODING 持续集成来自动部署网站。

![ci-log](/images/2021-03-08-use-coding-devops-deploy-serverless-application/ci-log.png)

