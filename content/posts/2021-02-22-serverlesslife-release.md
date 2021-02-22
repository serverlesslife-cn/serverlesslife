---
title: 'serverlesslife.cn 上线背后的故事～'
summary: '细数 serverlesslife.cn 上线背后的故事～'
tags: [serverlesslife', 'serverless framework']
date: 2021-02-22
author: donghui
featuredImagePreview: '/images/2021-02-22-serverlesslife-release/serverlesslife.png'
---

![cover](/images/2021-02-22-serverlesslife-release/serverlesslife.png)

## 引言
2021年年初，使用 Serverless Framework  在提腾讯云上部署了一个个人博客：serverlesslife.cn。
整体下来体会到了 Serverless 带来的一些便利：
* 不需要管理或运维服务器
* 按使用付费（有免费额度）
也体验到了 Serverless Framwwork CLI 的便利性：它大大降低了操作复杂度，用户体验完胜控制台。

当然，也有不好的体验，它总是让人难以忘怀，在实践过程中也踩到了产品的一些坑，均已反馈。
吐槽不好的体验，是为了让它变得更好！

在整个实践中，学到了很多新的知识，了解了一些背后的逻辑，本文将对它们做一下分享。

## 本次实践涉及到的腾讯云云服务
* Serverless Framework
* COS（对象存储）
* API 网关
* DNSPod 域名注册、域名备案、域名解析
* SSL 证书
* CDN（内容分发网络）

## 域名购买与备案
2021/01/21 在 DNSPod（2011 年被腾讯收购）购买了域名 serverlesslife.cn 并进行了实名认证。
购买域名后，在中国大陆，要使用域名提供服务，还需要进行域名备案。
2021/1/31 在「腾讯云网站备案」小程序上提交了备案申请；
经过了两轮审核（腾讯云审核+管局审核）后，在 8 天后的 2021/02/07 审核通过。

## Serverless Framework 简介
Serverless Framework 是 serverless.com 推出的一个流行的 Serverless 框架，它可以将 Serverless 函数/应用部署到不同的云厂商的 Serverless 平台。在国内腾讯云与 serverless.com 达成战略合作，对它进行了很多定制，做了很多组件，使得很容易将 Serverless 函数/应用部署到腾讯云。

## 使用 Hugo 搭建个人博客
要搭建个人独立博客，有很多开源的建站工具可以用，比如：WordPress、Hexo、Jekyll、Hugo 等等，不胜枚举。
因为之前用过 Hugo，并且比较喜欢它，这里使用了 Hugo 来搭建个人博客。
Hugo 是由一个 Go 语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。
Hugo 易扩展指的是它有丰富的主题可以选择，这是笔者喜欢它的原因之一，每个人都可以选择自己喜欢的主题。

在 Hugo 站点初始化后，在众多主题中选择了主题： LoveIt（多么好听的名字呀！）。
主题也有很多参数，可以根据自己的需求按需配置。

基本框架搭建完成后，如果要编写博文，只需要添加 Markdown 文件和相关静态资源文件。
Hugo 支持本地实时预览，可以一边写 markdown 文件，一边就能通过浏览器实时查看效果。
当然部署到生产服务器，需要进行编译，编译成站点的任务只要一条 hugo 命令就能完成。

站点源码托管在 GitHub 上：https://github.com/serverlesslife-cn/serverlesslife

同时使用 GitHub Actions 将代码同步到了 Gitee：https://gitee.com/serverlesslife/serverlesslife

## 部署站点
在站点编译后，使用 Serverless Framework CLI 便可将它部署到腾讯云。
Serverless Framework CLI 需要一个配置文件 serverless.yml，此时配置文件内容如下：
```yaml
component: website # (必填) 引用 component 的名称，当前用到的是 tencent-website 组件
name: serverlesslife-2021 # (必填) 该 website 组件创建的实例名称
app: serverlesslife-2021 # (可选) 该 website 应用名称
stage: prod # (可选) 用于区分环境信息，默认值是 dev

inputs:
  src:
    src: ./public
    index: index.html
  region: ap-guangzhou
  bucketName: serverlesslife-2021
  protocol: https
```
其中 public 目录是 hugo 编译之后站点文件所在目录。
使用 serverless deploy 命令进行部署操作，如果没有在本地的 .env 配置 secretid 和 secretkey，需要使用微信进行扫描登录。
部署过程中，会在 COS 创建一个 bucket 并将 public 目录下的文件上传到这个 bucket 中，然后会生成一个腾讯云四级域名的访问地址。其中该 bucket 前缀是 serverlesslife-2021，工具会自动加上腾讯云账号的 APPID 作为后缀。

Serverless Framework 控制台下，会有一个应用名称为 serverlesslife-2021 的应用，这个应用会有一个腾讯云四级域名的访问地址，如下所示：
![serverless-framework](/images/2021-02-22-serverlesslife-release/serverless-framework.png)


在 COS 对象存储的存储桶列表页，可以看到有一个名称为 serverlesslife-2021-1259061164  的 bucket，如下所示：
![cos](/images/2021-02-22-serverlesslife-release/cos.png)


## 自定义域名 + SSL 证书 + 自动刷新 CDN
配置「自定义域名 + SSL 证书 + 自动刷新 CDN」还需要在 serverless.yml 中增加一些配置信息，整个 serverless.yml 文件如下：
```yaml
component: website # (必填) 引用 component 的名称，当前用到的是 tencent-website 组件
name: serverlesslife-2021 # (必填) 该 website 组件创建的实例名称
app: serverlesslife-2021 # (可选) 该 website 应用名称
stage: prod # (可选) 用于区分环境信息，默认值是 dev


inputs:
  src:
    src: ./public
    index: index.html
  region: ap-guangzhou
  bucketName: serverlesslife-2021
  protocol: https
  hosts:
    - host: serverlesslife.cn
      autoRefresh: true #开启自动 CDN 刷新，用于快速更新和同步加速域名中展示的站点内容
      onlyRefresh: false #建议首次部署后，将此参数配置为 true，即忽略其他 CDN 配置，只进行刷新操作
      https:
        switch: on
        http2: on
        certInfo:
          certId: 'kBM9GLPt'
```
上面的配置文件支持配置多个域名，每个域名下面还可以配置其他信息，如 SSL 证书 ID、自动刷新 CDN 等。
使用自定义域名时，如果需要配置 SSL 证书，那么就必须使用 CDN，因为在 CDN 下可以配置证书。

SSL 证书需要事先在 SSL 证书控制台申请，这里申请了一个免费证书，证书有效期一年，申请成功后会有一个 ID：kBM9GLPt，如下所示：
![ssl](/images/2021-02-22-serverlesslife-release/ssl.png)


在配置好域名相关信息后，Serverless Framework 在部署后做了大量的事情，大大简化了配置成本。
如果不用 Serverless Framework 的话，那么就需要在不用云服务的控制台多个地方来回进行配置。

下面看下在配置「自定义域名 + SSL 证书 」背后，Serverless Framework 都做了哪些事情：

1、CDN 控制台下，证书管理—>配置证书，将域名和证书关联到了一起，如下所示：
![cdn](/images/2021-02-22-serverlesslife-release/cdn-1.png)

2、CDN 控制台下，域名管理—>添加域名，新增了一条记录，为 serverlesslife.cn 开启了静态加速，将 COS 静态网站作为源站点，并会生成 CNAME：
![cdn](/images/2021-02-22-serverlesslife-release/cdn-2.png)

3、COS 控制台下，点进 serverlesslife-2021-1259061164 存储桶，在域名传输与管理—>自定义 CDN 加速域名处，也会看到有一条记录：
![cdn](/images/2021-02-22-serverlesslife-release/cos-cdn.png)


此时，要正常访问域名，还需要手动配置下域名解析：打开 DNSPod 控制台，为 serverlesslife.cn 添加一条 CNAME 记录，记录值为 CDN 生成的 CNAME：serverlesslife.cn.cdn.dnsv1.com。
![dns](/images/2021-02-22-serverlesslife-release/dns.png)


注意：在首次部署后，将 onlyRefresh  参数配置为 true，即忽略其他 CDN 配置，只进行刷新操作，否则部署时间会相对比较长。

## CI/CD
CI/CD 已成为软件开发环节的标配，倡导将一切自动化，期待在代码提交到 master 分支后就会自动部署。
曾使用 GitHub Actions，但是因为官方托管服务器在国外，部署到国内的环境时，总会超时，暂时放弃 GitHub Actions。
未来计划使用 GitHub Actions 将代码自动同步到 coding.net ，然后使用 Coding CI 进行自动化部署。
 
## 参考
* https://hugoloveit.com/
* https://cloud.tencent.com/document/product/1154/39276
* https://github.com/serverless-components/tencent-website/blob/master/docs/configure.md
