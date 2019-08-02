---
layout: post
title: Spring Boot 最佳实践
categories: SpringBoot
description: springboot 经验
keywords: springboot, 翻译
---

## 使用自动配置

Spring Boot 的一个主要特性是它使用了自动配置。 这是 Spring Boot 中使代码简单工作的部分。 当在类路径上检测到特定的 jar 文件时，它将被激活。

使用它的最简单的方法是依赖于 Spring Boot Starters。 所以，如果你想和 Redis 互动，你可以从以下内容开始:

````xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
````

如果你想使用 MongoDB，你需要:

````xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
````
等等... ... 通过依赖这些启动器，您依赖于经过测试和验证的配置，这些配置将在一起工作得很好。 这有助于避免可怕的 [Jar Hell](https://dzone.com/articles/what-is-jar-hell)。

可以使用以下注释属性从 Auto-configuration 中排除某些类:

```java
@EnableAutoConfiguration（exclude = {ClassNotToAutoconfigure.class}）
```

但只有在绝对必要的情况下才应该这样做。

关于自动配置的官方文档可以在[这里](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html)找到。




## 使用 Spring Initializr 启动新的 Spring Boot 项目

Spring Initializr (Spring 初始化 https://start.Spring.io/ )为您提供了一个非常简单的方法来启动一个新的 Spring Boot 项目，并加载您可能需要的依赖项。

使用 Initializr 创建应用程序可以确保您正在选择经过测试和批准的依赖项，这些依赖项可以很好地与 Spring 自动配置一起工作。 你甚至可能发现一些你不知道存在的新的集成。



## 考虑为常见的组织问题创建您自己的自动配置

如果您在一个严重依赖 Spring Boot 的组织中工作，并且您有需要解决的共同问题，那么您可以[创建自己的自动配置](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-auto-configuration.html)。

这个任务比较复杂，所以你需要考虑什么时候投资是值得的。 单个自动配置比多个定制配置更容易维护，所有配置都略有不同。

如果您要将库发布到开放源码，那么提供一个 Spring Boot 配置将极大地方便成千上万用户的采用。



## 正确地组织代码结构

虽然允许您有很多自由，但是有一些基本的规则值得遵循，然后列出您的源代码。

- 避免使用缺省包。 确保所有的东西(包括你的入口点)都包含在一个命名良好的包中。 这样，您将避免有关装配和组件扫描相关的意外情况
- 保持你的 Application.java 在顶级目录中
- 我建议将控制器和服务放在面向功能的模块中，但这是可选的。 一些非常好的开发人员建议把所有的控制器放在一起。 不论怎样，坚持一种风格！




## 让你的“控制器”保持干净和专注

`Controller` 应该是很简单的。 您可以在这里阅读 [Controller pattern explained as part of GRASP](https://en.wikipedia.org/wiki/GRASP_(object-oriented_design)#Controller)。 您希望控制器进行协调和委派，而不是执行实际的业务逻辑。 以下是一些关键的做法:

- 控制器应该是无状态的！ 控制器是默认的单例模式，给它们任何状态都会导致严重的问题
- 控制器不应该执行业务逻辑，而是依赖于委托
- 控制器应该处理应用程序的 HTTP 层， 这不应该传递给服务
- 控制器应该围绕一个用例 / 业务能力来设计

要深入了解这里的内容，可以开始讨论设计 REST APIs 的最佳实践。 不管你是否想使用 Spring Boot，这些都是值得学习的。




## 围绕业务能力建立你的服务

`Services ` 是 Spring Boot 的另一个核心概念。 我发现最好是围绕业务功能 / 域 / 用例（随便你怎么称呼它）构建服务。

使用诸如 `AccountService`、 `UserService`、 `PaymentService` 等服务的应用程序比使用 `DatabaseService`、 `ValidationService`、 `CalculationService` 等服务的应用程序更容易处理。

您可以决定使用Controler和Service之间的一对一映射， 那将是理想的情况。但这并不意味着`Service`之间不能互相利用！



## 让你的数据库成为一个细节——从核心逻辑中抽象出来

我曾经不确定如何最好地处理 Spring Boot 中的数据库交互。 在阅读了罗伯特 · 马丁(robertc.Martin)的[Clean Architecture](https://www.e4developer.com/2018/07/14/discovering-clean-architecture-with-uncle-bob/) 之后，我更加清楚了。

您希望将数据库逻辑从服务中抽象出来。 理想情况下，您不希望服务知道它正在与哪个数据库通信。 使用一些抽象来封装对象的持久性。

> Robert c. Martin 充满激情地主张将您的数据库变成一个“细节”。 这意味着不将应用程序耦合到特定的数据库。 过去很少有人会转换数据库。 我注意到，随着 Spring Boot 和现代 microservices 的开发，事情变得更快了。




## 保持您的业务逻辑与 Spring Boot 代码无关

考虑到“ Clear Architecture”的经验教训，您还应该保护您的业务逻辑。 这是非常诱人的组合各种春天启动代码在那里... 不要这样做。 如果您能够抵制诱惑，您将保持业务逻辑的可重用性。

部分服务成为库是很常见的。 如果不从代码中删除大量 Spring 注释，则更容易创建。




## 赞成构造函数注入

这个来自 Phil Webb (现任 Spring Boot 的领导者,@phillip Webb)。

使您的业务逻辑免受Spring Boot代码侵入的一种方法是依赖构造函数注入。 `@autowired` 注释不仅在构造函数上是可选的，而且您还可以轻松地在没有 Spring 的情况下实例化 bean。



## 熟悉并发模型

写过的最受欢迎的文章之一是 "[Introduction to Concurrency in Spring Boot](https://www.e4developer.com/2018/03/30/introduction-to-concurrency-in-spring-boot/)"。 我认为这是因为这个领域经常被误解和忽视。 随之而来的就是各种问题。

在 Spring 中，Controller 和 Service 是默认是单例的。 如果不小心的话，可能会导致并发性问题。 您通常还要处理有限的线程池。 熟悉这些概念。

如果您正在使用 Spring Boot 应用程序的新 WebFlux 样式，我已经解释了它的工作原理，在 "[Spring’s WebFlux / Reactor Parallelism and Backpressure](https://www.e4developer.com/2018/04/28/springs-webflux-reactor-parallelism-and-backpressure/)"。



## 加强配置管理的外部化

这一点超越了 Spring Boot，尽管这是人们开始创建多个类似服务时常见的问题..。

您可以手动配置 Spring 应用程序。 如果你正在处理几十个 Spring Boot 应用程序，你需要成熟你的 Spring 组态管理。

我推荐两种主要方法:

- 使用配置服务，比如 [Spring Cloud Config](https://spring.io/projects/spring-cloud-config)。
- 将所有配置存储在环境变量中(可以基于 git 存储库提供)

这两个选项（第二个选项多一些）都要求您在 DevOps 领域有所建树，但这在微服务领域是意料之中的。



## 提供全局异常处理

您确实需要一种处理异常的一致方法。 Spring Boot 提供了两种主要的实现方式:

- 你应该使用 `HandlerExceptionResolver`用于定义全局异常处理策略
- 你也可以在控制器上添加 `@exceptionhandler`，如果你想在某些情况下更加具体，这可能会很有用

这与 Spring 非常相似，Baeldung 有一篇关于 [用 Spring 处理 REST 错误](https://www.baeldung.com/exception-handling-for-rest-with-spring) 的详细文章，非常值得一读。



## 使用日志框架

您可能已经意识到了这一点，但是您应该使用 Logger 记录日志，而不是使用 System.out.println ()手动完成日志记录。 这在没有任何配置的 Spring Boot 中很容易实现。 只需获得该类的日志记录器实例:

````java
Logger logger = LoggerFactory.getLogger(MyClass.class);
````

这很重要，因为它允许您根据需要设置不同的日志级别。



## 测试你的代码

这不是特定于 Spring Boot 的，但是值得注意！ 测试你的代码。 如果您没有编写测试，那么您从一开始就在编写遗留代码。

如果其他人来到你的代码库，很快，改变任何东西都可能变得危险。 如果有多个服务相互依赖，那么风险就更大了。

因为有 Spring Boot 的最佳实践，所以您应该考虑在消费者驱动的契约中使用 [Spring Cloud Contract](https://spring.io/projects/spring-cloud-contract)。 它将使您与其他服务的集成更容易使用。




## 使用测试片使您的测试更容易和更集中

使用 Spring Boot 测试代码可能很棘手——您需要初始化数据层、连接众多服务、模拟事物... ... 实际上并不需要那么困难！ 答案是-使用测试片。

使用测试片，您可以根据需要只连接应用程序的部分。 这可以为您节省很多时间，并确保您的测试不与您没有使用的东西耦合。 有一篇名为 [Custom test slice with Spring Boot 1.4](https://spring.io/blog/2016/08/30/custom-test-slice-with-spring-boot-1-4) 的博客文章解释了这种技术。



## 总结

多亏了 Spring Boot，编写基于 Spring 的微服务变得比以往任何时候都容易。 我希望通过这些最佳实践，您的实现过程不仅会很快，而且从长远来看会更加健壮和成功。 祝你好运！



>翻译自：
>
>- [Spring Boot – Best Practices](https://www.e4developer.com/2018/08/06/spring-boot-best-practices/#more-2109)

