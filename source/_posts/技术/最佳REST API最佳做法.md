---
title:   使用REST API 最佳实践简述
date: 2020-5-12 19:07
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# 使用REST API 最佳实践简述

Facebook，Google，Github，Netflix，Amazon和Twitter等许多巨头都拥有自己的REST（ful）API，您可以访问它们来获取甚至写入数据。

但是，为什么所有都需要REST？

那样好吗，为什么如此盛行？

当然，这不是传达消息的唯一方法吗？

REST和HTTP有什么区别？

好吧，事实证明**REST非常灵活，并且与** Internet所基于的主要协议**HTTP兼容**。由于它是一种架构风格而不是标准，因此它**提供了实现各种设计最佳实践的大量自由**。以及听说它与语言无关？你一定觉得它很棒吧。

在此博客文章中，我们的目标是尽可能清楚地解释REST，以便您可以清楚地了解何时以及如何使用REST，以及它的本质。

我们将介绍一些基础知识和定义，并展示一些**REST API最佳实践**。这应该为您提供了以您喜欢的任何编码语言实现REST API所需的全部知识。

如果您对HTTP不太熟悉，建议您阅读我们的[HTTP系列文章](https://code-maze.com/http-series/)，或者至少阅读其中的[第1部分](https://code-maze.com/http-series-part-1/)，这样您可以更轻松地理解这些资料。

因此，在这篇文章中，我们将讨论：

**关于REST：**

- [什么是REST？](https://code-maze.com/top-rest-api-best-practices/#whatisrest)
- [REST是否绑定到HTTP？](https://code-maze.com/top-rest-api-best-practices/#restbound)
- [REST和HATEOAS支持](https://code-maze.com/top-rest-api-best-practices/#hateoas)
- [RESTful API是什么意思？](https://code-maze.com/top-rest-api-best-practices/#whatdoesitmean)
- [REST过多又称为RESTafarian综合征](https://code-maze.com/top-rest-api-best-practices/#restafarian)

**REST API最佳做法：**

- [抽象与具体API](https://code-maze.com/top-rest-api-best-practices/#abstractvsconcrete)
- [URI格式（名词，不是动词）。正确网址与错误网址示例](https://code-maze.com/top-rest-api-best-practices/#urlformat)
- [错误处理](https://code-maze.com/top-rest-api-best-practices/#errorhandling)
- [状态码](https://code-maze.com/top-rest-api-best-practices/#statuscodes)
- [安全](https://code-maze.com/top-rest-api-best-practices/#security)
- [REST API版本控制](https://code-maze.com/top-rest-api-best-practices/#versioning)
- [文件的重要性](https://code-maze.com/top-rest-api-best-practices/#documentation)

## 那么REST本质上是什么？

REST（代表性状态转移）是[Roy Fielding](https://en.wikipedia.org/wiki/Roy_Fielding)在其博士学位中创立的一种建筑风格。UC Irvine的论文“ [体系结构样式和基于网络的软件体系结构设计](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) ”。 他与HTTP 1.1同步提出了这种观点。

我们主要将REST用作**在万维网上的计算机系统之间进行通信的**一种方式。

## REST是否绑定到HTTP？

根据定义，似乎与Http强制绑定？其实并非如此。尽管您可以将其他一些应用程序协议与REST一起使用，但是在实现REST时，[HTTP](https://code-maze.com/http-series/)仍然是应用程序协议中无可争议的冠军。

## REST和HATEOAS支持

**作为应用程序状态引擎的** HATEOAS或**超媒体** 是每个可扩展且灵活的REST API的重要功能。

该[HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)约束建议，客户端和服务器通信完全采用了[超媒体](https://en.wikipedia.org/wiki/Hypermedia)。

使用超媒体有几个优点：

- 使API设计人员能够在每个响应中包括他们所能提供的一切，以正确地提供一件事以及与相关端点的超媒体链接，从而使设计脱钩
- 帮助API更优雅地发展和成熟
- 为用户提供更深入地探索API的方法

因此很明显，HATEOAS在**设计时考虑了耐用性**。

GitHub的工作方式如下： 

` GET https://api.github.com/users/codemazeblog`

响应：  

```javascript
{
  "login": "CodeMazeBlog",
  "id": 29179238,
  "avatar_url": "https://avatars0.githubusercontent.com/u/29179238?v=4",
  "gravatar_id": "",
  "url": "https://api.github.com/users/CodeMazeBlog",
  "html_url": "https://github.com/CodeMazeBlog",
  "followers_url": "https://api.github.com/users/CodeMazeBlog/followers",
  "following_url": "https://api.github.com/users/CodeMazeBlog/following{/other_user}",
  "gists_url": "https://api.github.com/users/CodeMazeBlog/gists{/gist_id}",
  "starred_url": "https://api.github.com/users/CodeMazeBlog/starred{/owner}{/repo}",
  "subscriptions_url": "https://api.github.com/users/CodeMazeBlog/subscriptions",
  "organizations_url": "https://api.github.com/users/CodeMazeBlog/orgs",
  "repos_url": "https://api.github.com/users/CodeMazeBlog/repos",
  "events_url": "https://api.github.com/users/CodeMazeBlog/events{/privacy}",
  "received_events_url": "https://api.github.com/users/CodeMazeBlog/received_events",
  "type": "User",
  "site_admin": false,
  "name": "Code Maze",
  "company": "Code Maze",
  "blog": "https://code-maze.com",
  "bio": "A practical programmers' resource.",
  ...
}
```

如您所见，除了客户端请求的关键信息之外，您还可以在响应中找到一堆相关的超媒体链接，这些链接将您带到您可以自由浏览的API的其他部分。

## RESTful API是什么意思？

“ RESTful”意味着一些功能：

- **[客户端-服务器体系结构](https://code-maze.com/http-series-part-2)：**完整的服务由作为整个系统前端的“客户端”和作为后端的“服务器”组成
- **无状态：**服务器不应在不同请求之间保存任何状态。会话状态完全由客户负责。按照REST定义： ***所有REST交互都是无状态的。也就是说，每个请求都包含连接器理解该请求所需的所有信息，而与之前的任何请求无关。（[Roy的论文ch.5.2.2](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)）\***
- **[可缓存的](https://code-maze.com/http-series-part-2/#caching)：**客户端应该能够将响应存储在缓存中以提高性能

因此，RESTful API是一项遵循这些规则的服务（希望如此），并使用[HTTP方法](https://code-maze.com/the-http-reference/#requestmethods)来操纵资源集。

但是为什么我们需要或使用RESTful API？

因为它们为我们提供了一种简单，灵活和可扩展的方式来制作可通过Internet进行通信的分布式应用程序。

## 我们可以拥有更多的REST方法吗？

是的，你猜对了。是的，我们可以🙂

正如[Mike Schinkel](http://mikeschinkel.com/about/)定义的那样，对于狂热地遵循REST的人们来说甚至还有一个短语 [。](http://mikeschinkel.com/about/)

> RESTifarian是Roy T. Fielding在他的博士论文第五章中定义的REST软件架构风格的狂热支持者。论文在UCIrvine。你可以在rest - discussion邮件列表中找到野外的RESTifarians。但是要小心，RESTifarians在讨论休息的细节时可能是极其细致的，正如我最近在参与列表时所了解到的。🙂

太多的事情都是不好的。

我们需要一点**实用主义**才能做出好的应用程序和服务。了解和理解一种理论很重要，但是该理论的实现是区分不良与良好与卓越应用的区别。所以要聪明，要牢记最终用户。

因此，让我们走一些使API变得“光彩”的重要点，使用户的生活变得更加轻松。

## 抽象与具体API

在开发软件时，我们经常使用抽象和多态来获取大多数应用程序。我们想重用尽可能多的代码。

那么我们也应该这样写我们的API吗？

好吧，API并非完全如此。对于REST API，**具体要比abstract好**。你能猜出为什么吗？

让我向您展示一些示例：

让我们看两个API版本。它是最好有有一个的API `/entities`，或者有一个API `/owners`，`/blogs`并 `/blogposts` 分别？

作为开发人员，哪一个对您更具描述性？您想使用哪个API？

我总是会选择第二个。

## **URI格式（名词，不是动词）。正确网址与错误网址示例**

这是另一种REST API最佳实践。您应该如何格式化端点？

如果使用软件开发方法，您将得到如下所示的结果：

```
/getAllBlogPosts
/updateBlogPost/12
/deleteBlogPost/12
/getAuthorById/3
/deleteAuthor/3
/updateAuthor/3
```

您明白了……会有很多端点，每个端点都在做其他事情。有一个更好的系统可以解决这些问题。

将资源视为名词，将HTTP方法视为动词。如果这样做，最终将得到如下结果：

`GET /blogposts` –获取所有博客文章

`GET /blogposts/12` –获取ID为12的博客文章

`POST /blogposts` –添加新的博客文章并返回详细信息

`DELETE /blogposts/12` –删除ID为12的博客文章

`GET /authors/3/blogposts` –获取ID为3的作者的所有博客文章

这是创建API的更简洁，更精确的方法。对于最终用户而言，这是显而易见的，并且有一种解决方法。

通过使用单数而不是复数来表示资源名称，可以使其更加简洁。那取决于你。

## **错误处理**

API构建的另一个重要方面。有几种处理错误的好方法。

让我们看看顶级狗如何做到这一点：

**推特：**

- 请求： `GET https://api.twitter.com/1.1/account/settings.json`

- 响应：状态码400

  Twitter response

   

  ```
  {"errors":[{"code":215,"message":"Bad Authentication data."}]}
  ```

  

Twitter为您提供状态代码和错误代码，并简要描述了所发生错误的性质。他们让您在“ [响应代码”](https://developer.twitter.com/en/docs/basics/response-codes)页面上查找代码。

**脸书：**

- 请求： `GET https://graph.facebook.com/me/photos`

- 响应：状态码400

  Facebook Response 

  ```
  {  "error": {   "message": "An active access token must be used to query information about the current user.",   "type": "OAuthException",   "code": 2500,   "fbtrace_id": "DzkTMkgIA7V"  }}
  ```

  

Facebook为您提供了更具描述性的错误消息。

**特威里奥：**

- 请求： `GET https://api.twilio.com/2010-04-01/Accounts/1234/IncomingPhoneNumbers/1234`

- 响应：状态码404 

  ```
  <?xml version='1.0' encoding='UTF-8'?><TwilioResponse>  <RestException>    <Code>20404</Code>    <Message>The requested resource /2010-04-01/Accounts/1234/IncomingPhoneNumbers/1234 was not found</Message>    <MoreInfo>https://www.twilio.com/docs/errors/20404</MoreInfo>    <Status>404</Status>  </RestException></TwilioResponse>
  ```

  

Twilio默认为您提供XML响应，并提供指向文档的链接，您可以在其中找到错误的详细信息。

如您所见，错误处理的方法因实现而异。

重要的是 **不要让REST API的用户“挂断”**，不知道发生了什么，或者漫无目的地在StackOverflow的浪费中徘徊，寻找解释。

## **状态码**

在设计REST API时，我们通过使用[HTTP状态代码](https://code-maze.com/the-http-reference/#statuscodes)与API用户进行通信 。状态代码很多，描述了多种可能的响应。

但是，我们应该使用多少？ **我们在每种情况下都应该有严格的状态码吗？**

就像生活中的许多事情一样， [KISS原则](https://en.wikipedia.org/wiki/KISS_principle) 也适用于这里。那里有70多个状态代码。你内心了解他们吗？潜在的API用户会全部了解它们，还是会再次使用Google搜索？

大多数开发人员都熟悉最常见的状态代码：

- `**200 OK**`
- `**400 Bad Request**`
- `**500 Internal Server Error**`

从这三个开始，您可以涵盖REST API的大多数功能。

其他常见的代码包括：

- `**201 Created**`
- `**204 No Content**`
- `**401 Unauthorized**`
- `**403 Forbidden**`
- `**404 Not Found**`

我们可以使用它们来帮助用户快速找出结果。如果您感觉到状态代码的描述性不如我们在“错误处理”部分中讨论的那样，则可能应该包含某种消息。再一次，我们需要务实，通过使用 **数量有限的代码** 和描述性消息来帮助用户。

您可以在[此处](https://code-maze.com/the-http-reference)找到完整的HTTP状态代码列表，以及[在CodeMaze上总结的](https://code-maze.com/the-http-reference)其他有用的HTTP内容 。

## **安全**

关于[REST API安全的](https://blog.restcase.com/top-5-rest-api-security-guidelines/)说法不多，因为 **REST不处理安全问题**。它依赖于诸如[基本身份验证或摘要身份验证之](https://code-maze.com/http-series-part-4/)类的标准HTTP机制 。

每个请求都应 **通过HTTPS进行**。

有很多技巧可以提高REST API的安全性，但是由于REST的无状态性，因此在实施它们时必须谨慎。记住最后一个请求的状态超出了窗口，**应该**在 **客户端存储和验证状态。**

**时间戳记和日志记录** 请求也可以有所帮助。

关于这个话题还有很多要说的，但这超出了本文的范围。我们有一个不错的职位**[HTTP安全](https://code-maze.com/http-series-part-5/)** 在这里CodeMaze如果您想了解更多关于这一点。 

## **REST API版本控制**

您已经编写了REST API，它已经非常成功，许多人已经使用它并对此感到满意。但是，您拥有的多汁的新功能会破坏系统的其他部分。重大变化。

不用担心，有解决方案！

在开始制作您的API之前，我们可以通过在端点之前加上API版本来对其进行版本控制：
`https://api.example.com/v1/authors/2/blogposts/13`

这样，只要API发生重大更改，我们就可以始终增加API版本号（例如v2，v3…）。这也向用户发出信号，表明已发生了翻天覆地的变化，在使用新版本时，请务必小心。

## **文件的重要性**

这是不言而喻的。您可能是世界上最好的API设计人员，但是 **如果没有文档，您的API就像死了一样。**

**正确的文档** 对于每个软件产品和Web服务都是**必不可少**的。

我们可以通过保持一致并使用清晰和描述性的语法来帮助用户。但是，好的文档页面并没有真正的替代品。

一些很好的例子：

https://www.twilio.com/docs/api/rest/

https://developers.facebook.com/docs/

https://developers.google.com/maps/documentation/

还有很多其他...

有许多工具可以帮助您记录您的API，但是不要忘记让人参与其中，只有一个人可以正确地理解另一个人。至少现在是这样(看着你)。

## 结论

我们讨论了REST API构建的许多概念，并介绍了一些顶级REST API最佳实践。在一次提供这些API时，您可能会觉得有些奇怪或难以接受，但是请尝试自己创建REST API。并尝试实现一些您在这里学到的REST API最佳实践。 