---
title: 'Google 是如何设计 Ruby Serverless Runtime 的？'
summary: 'Google 在设计 Ruby Serverless Runtime 时面临的一些设计问题，做出的决策以及为什么做出这些决策。'
tags: ['google', 'ruby', 'serverless']
date: 2021-02-01
author: donghui
featuredImagePreview: '/images/2021-02-01-designing-a-ruby-serverless-runtime/google.png'
---

![cover](/images/2021-02-01-designing-a-ruby-serverless-runtime/google.png)

> 原文：https://daniel-azuma.com/blog/2021/01/20/designing-a-ruby-serverless-runtime
>
> 作者：Daniel Azuma（Google）
>
> 译者：donghui

2021年1月中旬，Google 宣布了 Cloud Functions  的 Ruby 运行时公测。Cloud Functions 是 Google 的函数即服务（Faas）平台。在过去的一年时间里，Google Cloud Functions 对 Ruby 语言的支持已经落后于其他语言，但是我们现在已经赶上了，我想我会分享该产品背后的一些设计过程。

本文不是传统的设计文档。我不会逐步介绍设计本身。相反，我想讨论我们面临的一些设计问题，做出的决策以及为什么做出这些决策。因为这是一个关于如何将 Ruby 约定与公共云约定融合的有趣练习。我认为，我们做出的一些权衡，代表着整个 Ruby 社区随着行业的发展而面临的挑战。

## 一种实现 Ruby Serverless 化的方式
为 Serverless 产品提供 Ruby 支持比您预期的要复杂得多。从最基本的角度来看，语言运行时只是 Ruby 的安装，并且可以肯定的是，配置 Ruby 镜像并将其安装在 VM 上并不难。但是，当您将 “Serverless” 加入其中时，事情会变得更加复杂。Severless 不仅仅是自动维护和扩容。这是对计算资源的完全不同的思考方式，这与过去15年中我们学到的有关部署 Ruby 应用程序的许多知识背道而驰。当 Google Cloud 的 Ruby 团队承担为 Cloud Functions 设计 Ruby 运行时的任务时，我们还承担了一项艰巨的任务，即提出一种 Ruby 方式来实现 Serverless。在坚持我们社区所熟悉的 Ruby 习惯、实践和工具的同时，我们还必须重新思考如何在几乎每个层次上进行 web 应用程序开发，从代码到依赖、持久化、测试等等。

本文将研究我们在设计的五个不同方面的方法：函数语法、并发性和生命周期、测试、依赖项和标准。在每种情况下，我们都将在忠于 Ruby 本色的重要性与拥抱新的 Serverless 范式的愿望之间保持一个平衡。我们非常努力地保持与传统 Ruby 工作方式的连续性，并且还从 Google Cloud Functions 其他语言运行时中汲取了经验，并借鉴了其他云提供商的 Serverless 产品所树立的先例。但是，在少数情况下，我们选择另辟蹊径。我们之所以这么做，是因为我们觉得当前的方法要么是滥用了语言功能，要么是误导和鼓励了关于 Serverless 应用开发的错误想法。

某些决策最终有可能被证明是错误的。这就是我现在提供这篇文章的原因。讨论我们已经做的事情，并开始讨论我们作为 Ruby 社区实践 Serverless 应用程序开发的方式。好消息是 Ruby 是一种非常灵活的语言，随着我们的学习和需求的发展，我们将有很多机会适应它。

因此，让我们看一下我们做出的一些初始设计决策和权衡以及做出这些决策的原因。

## 函数化 Ruby
“函数即服务”（FaaS）当前是较流行的 Serverless 范式之一。Google Cloud Functions 只是一种实现。许多其他主要的云提供商都拥有自己的 FaaS 产品，并且也有开源实现。

当然，这种想法是使用一种编程模型，该模型不以 Web 服务器为中心，而是以函数为中心：无状态的代码片段，它们接受输入参数并返回结果。这似乎是一个简单的、几乎显而易见的术语变化，但实际上具有深远的意义。

![function](/images/2021-02-01-designing-a-ruby-serverless-runtime/function.png)

对 Ruby 而言，面临的第一个挑战是，与许多其他编程语言不同，在 Ruby 中函数并不是一等公民。Ruby 首先是一种面向对象的语言。当我们编写代码并将其封装在 def 中时，我们正在编写一个方法，这是响应发送给对象的消息而运行的代码。这是一个重要的区别，因为组成方法调用上下文的对象和类不是 Serverless 抽象的一部分。因此，它们的存在会使 Serverless 的应用程序复杂化，甚至在我们编写应用程序时误导我们。

例如，某些 FaaS 框架使您可以使用 def 在 Ruby 文件的顶层编写函数：

```ruby
def handler(event:, context:)
  "Hello, world!"
end
```

虽然这段代码看起来很简单，但重要的是要记住它实际上做了什么。它将这个“函数”添加为 Object 类的私有方法，Object 类是 Ruby 类层次结构的基类。换句话说，Ruby 虚拟机中的几乎每个对象都添加了“函数”。(当然，除非应用程序在加载文件时更改了主对象和类上下文，这种技术会带来其他风险。)在最好的情况下，这打破了封装和单一职责。在最坏的情况下，它可能会干扰应用程序的功能、依赖关系，甚至是 Ruby 标准库。这就是为什么这种“顶级”方法在简单的单文件 Ruby 脚本和 Rakefiles 中很常见，但在大型 Ruby 应用程序中不推荐使用。

Google Ruby 团队认为这个问题很严重，所以我们选择了一种不同的语法，将函数写成块的形式:

```ruby
require "functions_framework"
FunctionsFramework.http("handler") do |request|
  "Hello, world!"
end
```

这提供了一种类似于 Ruby 的方式来定义函数而无需修改 Object 基类。它还有一些附带好处：
* 名称（在这种情况下为 “handler”）只是一个字符串参数。它不必是合法的 Ruby 方法名称，也不必担心它与 Ruby 关键字冲突。
* 块比方法具有更多的传统词法作用域，因此其行为与其他语言中的函数更相似。
* 块语法使管理函数定义更加容易。例如，可以干净地“undefine”函数，这对于测试很重要。

当然，需要权衡取舍。其中：
* 语法稍微有些冗长。
* 它需要一个库来提供用于将函数定义为块的接口。（这里，Ruby 通过使用 Functions Framework 库跟随了 Cloud Functions 的其他语言运行时。）

我们认为，为了实现正确区分函数的目标，这些权衡是值得的。

## 共享或不共享
并发性是很难的。这是 Serverless 设计(特别是函数即服务)的一个关键观察点：我们生活在一个并发的世界中，我们需要各种方法来应对。函数范式通过坚持函数不共享状态(除非通过外部持久化系统，如队列或数据库)来解决并发性问题。这实际上是我们选择使用块语法而不是方法语法的另一个原因。方法隐含对象，对象以实例变量的形式携带状态，这些状态在无状态 FaaS 环境中可能无法正常工作。回避方法是一种微妙但有效的语法方法，可以阻止我们知道的存在问题的实践。

也就是说，如果需要共享资源，比如数据库连接池，该怎么办?何时初始化这些资源，如何访问它们?

为此，Ruby 运行时支持启动函数，这些函数可以初始化资源并将它们传递给函数调用方。重要的是，启动函数可以创建资源，而普通函数只能读取它们。

```ruby
require "functions_framework"


# Use an on_startup block to initialize a shared client and store it in
# the global shared data.
FunctionsFramework.on_startup do
  require "google/cloud/storage"
  set_global :storage_client, Google::Cloud::Storage.new
end

# The shared storage_client can be accessed by all function invocations
# via the global shared data.
FunctionsFramework.http "storage_example" do |request|
  bucket = global(:storage_client).bucket "my-bucket"
  file = bucket.file "path/to/my-file.txt"
  file.download.to_s
end
```

注意，我们选择了定义特殊方法 global 和 set_global 来与全局资源交互。顺便说一下，这些不是 Object 上的方法，而是作为函数上下文使用的特定类上的方法。同样，我们可以使用更传统的习惯用法，如 Ruby 全局变量，甚至构造函数和实例变量，将信息从启动代码传递给函数调用方。然而，这些语法可能传递了错误的东西。我们不是在普通的 Ruby 类和方法中编写共享数据是正常的，而是在 Serverless 的函数中编写共享数据是危险的(即使可能的话)，我们认为语法上强调区别是很重要的。这些特殊方法是经过深思熟虑的设计决策，以防止在并发存在时出现危险的实践。

## 测试为首
强大的测试文化是 Ruby 社区的核心。流行的框架，如 Rails，承认了这一点，并通过提供测试工具和脚手架作为框架的一部分来鼓励主动测试，Google Cloud Functions 的 Ruby 运行时也遵循了这一点，为 Serverless 的函数提供了测试工具。

FaaS 范式实际上非常适合测试。函数本质上是容易测试的，只需传入参数并对结果进行断言即可。特别是，您不需要启动 web 服务器来运行测试，因为 web 服务器不是抽象的一部分。Ruby 运行时提供了一个 helper方 法模块，用于创建作为输入使用的 HTTP 请求和云事件对象，除此之外，大多数测试都非常容易编写。

然而，我们遇到的主要测试挑战之一与测试初始化代码有关。确实，这是 Google Ruby团队成员在使用其他框架(包括 Rails)时遇到的一个问题：很难测试应用程序的初始化过程，因为框架的初始化通常发生在测试之外，在它们运行之前。因此，我们设计了一种测试方法来隔离函数的整个生命周期，包括初始化。这允许我们在测试中运行初始化，甚至重复它多次，允许不同方面的测试：

```ruby
require "minitest/autorun"
require "functions_framework/testing"


class MyTest < Minitest::Test
  # Include testing helper methods
  include FunctionsFramework::Testing


  def test_startup_tasks
    # Run the lifecycle, and test the startup tasks in isolation.
    load_temporary "app.rb" do
      globals = run_startup_tasks "storage_example"
      assert_kind_of Google::Cloud::Storage, globals[:storage_client]
    end
  end


  def test_storage_request
    # Rerun the entire lifecycle, including the startup tasks, and
    # test a function call.
    load_temporary "app.rb" do
      request = make_get_request "https://example.com/foo"
      response = call_http "storage_example", request
      assert_equal 200, response.status
    end
  end
end
```

load_temporary 方法在沙箱中加载函数定义，将它们及其初始化与其他测试运行隔离开来。该方法和其他 helper 方法定义在 FunctionsFramework::Testing 模块中，可以包含在 minitest 或 rspec 测试中。

到目前为止，我们只为 Ruby 运行时提供了基本的测试工具，我希望随着用户开发更多的应用程序和识别出更多常见的测试模式，我们会在工具集中大量增加这些工具。但我坚信测试工具是任何库的重要组成部分，特别是那些声称是框架或运行时的库，所以它是我们设计的核心部分。

## 可依赖的运行时
大多数重要的 Ruby 应用程序都需要第三方 gems。对于使用 Google Cloud Functions 的 Ruby 应用程序，我们至少需要一个 gem，即 functions_framework，它提供了编写函数的 Ruby 接口。您可能还需要其他 gems 来处理数据、进行身份验证并与其他服务集成等等。依赖项管理是任何运行时框架的关键部分。

我们围绕依赖项管理做出了几个设计决策。而第一个也是最重要的就是拥抱 Bundler。

我知道这听起来有点无聊。现在大多数 Ruby 应用程序都在使用 Bundler，而且很少有替代方案，很少有广泛使用的。但我们实际上更进一步，将 Bundler 深入到我们的基础架构中，要求应用程序使用它来处理云函数。我们这么做是因为，确切地知道应用将如何管理它的依赖关系将允许我们实现一些重要的优化。

![dependency](/images/2021-02-01-designing-a-ruby-serverless-runtime/dependency.png)

对于一个好的 FaaS 系统来说，部署和冷启动的速度至关重要。在 serverless 的世界中，您的代码可能会快速连续地更新、部署和拆除许多次，因此消除瓶颈(如解析和安装依赖项)是至关重要的。因为我们为依赖项管理标准化了一个系统，所以我们能够主动地缓存依赖项。我们认为，实现这样的缓存所带来的性能提升，以及 Rubygems.org 基础架构所减少的负载，远远超过了不能使用 Bundler 的替代方案所带来的灵活性降低。

Google Cloud Functions 的 Ruby 运行时的另一个特性，或者可能是怪癖，是如果 gem lockfile 丢失或不一致，部署将失败。我们需要这个 Gemfile.lock 在部署时存在。这是执行最佳实践的另一个决策。如果在部署期间重新解析了锁文件，那么您的构建可能是不可重复的，并且您可能没有针对测试时使用的相同依赖项运行。我们通过要求一个最新的 Gemfile.lock 来避免这个问题。同样，我们能够强制执行这一点，因为我们需要使用 Bundler。

## 新旧标准
最后，好的设计依赖于标准和现有技术。为了在 Ruby 中定义健壮的函数，我们不得不进行一些创新，但在表示函数参数时，已经有现成的库或新兴标准可供遵循。

例如，在近期内，许多函数将响应 web hook，并需要关于传入 HTTP 请求的信息。设计一个表示 HTTP 请求的类并不困难，但是 Ruby 社区已经有了用于这类事情的标准 API: Rack。我们采用 Rack 请求类作为事件参数，并支持标准的 Rack 响应作为返回值。
require "functions_framework"

```ruby
FunctionsFramework.http "http_example" do |request|
  # request is a Rack::Request object.
  logger.info "I received #{request.request_method} from #{request.url}!"
  # You can return a standard Rack response array, or use one of
  # several convenience formats.
  [200, {}, "ok"]
end
```

这不仅提供了一个熟悉的 API，而且还使它易于与其他基于 Rack 的库集成。例如，很容易将 Sinatra 应用程序置于云函数之上，因为它们都能支持 Rack。

从长远来看，我们越来越希望函数即服务（Faas）能够作为事件系统中的一个组件。基于事件的架构正在迅速普及，经常围绕事件队列，如 Apache Kafka。事件体系结构的一个关键元素是描述事件本身的标准方法，事件发送方、代理、传输和使用者都理解这种标准。

Google Cloud Functions 支持 CNCF CloudEvents，这是一个描述和交付事件的新兴标准。除了 HTTP 请求之外，云函数还可以接收 CloudEvent 形式的数据，运行时甚至会在调用函数时将一些遗留事件类型转换为 CloudEvent。

```ruby
require "functions_framework"


FunctionsFramework.cloud_event "my_handler" do |event|
  # event is a CloudEvent object defined by the cloud_events gem
  logger.info "I received a CloudEvent of type #{event.type}!"
end
```

为了在 Ruby 中支持 CloudEvent，Google Ruby 团队与 CNCF Serverless 工作组密切合作，甚至自愿接管了用于 CloudEvent 的Ruby SDK 的开发。这是一项繁重的工作，但我们认为能够使用官方的、标准的 Ruby 接口至关重要，即使我们必须自己实现它。

## Serverless 的未来
“Serverless” 和“函数即服务”的主机托管在过去几年里引起了很多人的兴趣。我认为对于大多数工作负载来说，它到底有多有用还没有定论，但可能性是有趣的。“零”devops，自动维护和扩容，不需要维护服务器，只需要为实际使用的计算资源付费。最近，我把这个博客从一个个人的 Kubernetes 集群迁移到了 Google 托管的 Cloud Run 服务上，并将我的每月账单从几十美元降到了几美分。

也就是说，Serverless 是思考计算资源的一种根本不同的方式，作为一个行业，我们对其影响的理解还很早期。当我的团队为 Google Cloud Functions 设计 Ruby 运行时，我们注意到 serverless 范式与我们的常规 Ruby 实践交互的方式。在某些情况下，就像测试一样，它鼓励我们在 Ruby 文化的优点上加倍下注。在另一些情况下，就像在严格意义上讲没有函数的语言中如何表达和标记函数一样，它挑战了我们关于如何呈现代码并传达其意图的想法。

但在所有情况下，设计运行时的经验提醒我，我们处在一个不断变化的行业中。Serverless 只是一系列变化中的最新一个，这些变化包括公共云，甚至包括 Rails 和 Ruby 本身。目前还不清楚这种 serverless 技术会持续多久，但它已经出现了，我们需要以好奇心、创造力和不把已知事物视为理所当然的意愿来回应。
