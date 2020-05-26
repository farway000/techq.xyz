---
title:  .TDD学习笔记（一）单元测试
date: 2020-5-3 19:07
tags: 技术
author: 邹溪源
categories:
  - 技术
---
---
typora-root-url: ..\..\..\..\image.techq.xyz\images\improve-webapi
---

# 提升WebAPI性能的几个小建议

本文作者：德本德拉·达什（Debendra Dash）

原文来自： https://www.c-sharpcorner.com/article/Tips-And-Tricks-To-Improve-WEB-API-Performance/ 



Web API是Microsoft作为.NET框架的一部分而开发的一项技术，它使用户能够与异构平台进行通信，包括网站，移动设备和桌面应用程序等。在编写Web API时，我们应该关注其性能和响应时间。在这里，我列出了在提高Web API性能时需要考虑的几点。

 ![1](/1.png)

 

## 在Web API中使用并行编程 

 

Web API逻辑主要处理2个重要功能-将数据发布到服务器以进行插入或更新，以及从服务器获取数据。当我们有成千上万的记录要从服务器获取时，响应时间非常高。这是因为有时我们将不得不遍历数据源，进行几次更改，然后将数据发送到客户端。而且，一个简单的foreach循环是一个单线程循环，该循环逐个顺序处理数据以给出结果集。

 因此，在这种情况下，建议在从服务器获取数据时使用并行foreach循环。由于并行的foreach循环在多线程环境中工作，因此执行速度将比foreach循环更快。 

### 并行Foreach循环的执行过程

~~~c#


```c#
List<Employee> li = new List<Employee>();  
li.Add(new Employee { Id = 1001, Name = "Sambid", Company = "DELL", Location = "Bangalore" });  
li.Add(new Employee { Id = 1002, Name = "Sumit", Company = "DELL", Location = "Bangalore" });  
li.Add(new Employee { Id = 1003, Name = "Koushal", Company = "Infosys", Location = "New Delhi" });  
li.Add(new Employee { Id = 1004, Name = "Kumar", Company = "TCS", Location = "Bangalore" });  
li.Add(new Employee { Id = 1005, Name = "Mohan", Company = "Microsoft", Location = "Hyderabad" });  
li.Add(new Employee { Id = 1006, Name = "Tushar", Company = "Samsung", Location = "Hyderabad" });  
li.Add(new Employee { Id = 1007, Name = "Jia", Company = "TCS", Location = "Pune" });  
li.Add(new Employee { Id = 1008, Name = "KIRAN", Company = "CTS", Location = "Bangalore" });  
li.Add(new Employee { Id = 1009, Name = "Rinku", Company = "CGI", Location = "Bangalore" });  
```
~~~

 这是Employee类:

```c#
public class Employee  
{  
    public int Id { get; set; }  
    public string Name { get; set; }  
    public string Company { get; set; }  
    public string Designation { get; set; }  
    public string Location { get; set; }  
       
}  
```

 这就是我们可以使用Parallel Foreach循环的方式。 

![](/parallel.png)

该循环将立即执行并给出结果。在Foreach循环的情况下，它一一给出结果。假设结果集中有1000条记录，则循环将执行一次1000次以得到结果。

 

**注意：** 当您要获取的记录数量很少时，请不要使用Parallel foreach循环。

## 使用异步编程来处理并发的HTTP请求

 

同步编程中发生的事情是，每当有请求执行Web API时，就会将线程池中的线程分配给要执行的请求。该线程被阻塞，直到执行该过程并返回结果为止。

![](/as1.png)

在这里，T2 线程一直处于阻塞状态，直到它处理请求并返回结果为止；如果遇到长的执行循环，则它将花费大量时间并一直等待到结束。

 

假设我们只有3个线程，并且所有线程都已分配给队列中正在等待的三个请求。在这种情况下，如果第四个请求在我们同步实现时出现，它将发出错误，因为现在它没有任何线程可以处理该请求。

 

因此，要处理更多数量的并发HTTP请求，我们必须使用异步编程。

 

异步请求处理程序的操作有所不同。当请求到达Web API控制器时，它将获取其线程池线程之一，并将其分配给该请求。

 

在进程开始执行的同时，线程将返回到线程池。执行完成后，为该请求分配了另一个线程来带来该请求，因此，该线程将不会等到进程执行完成后才返回到线程池以处理另一个请求。

![](/final.png)

 因此，在这里我给出了一个小的注册示例，以及如何在Web API中使用异步编程。 

```c#
[AllowAnonymous]  
[Route("Register")]  
public async Task<IHttpActionResult> Register(RegisterBindingModel model)  
{  
    Dictionary<object, object> dict = new Dictionary<object, object>();  
    if (!ModelState.IsValid)  
    {  
        return BadRequest(ModelState);  
    }  
    var user = new ApplicationUser() { UserName = model.Email, Email = model.Email };  
    IdentityResult result = await UserManager.CreateAsync(user, model.Password);  
    if (result.Succeeded)  
    {  
        tbl_Users obj = new tbl_Users();  
        obj.Active = false;  
        obj.FirstName = model.FirstName;  
        obj.LastName = model.LastName;  
        obj.Email = model.Email;  
        obj.UserId = user.Id;  
        DefEntity.tbl_Users.Add(obj);  
        if (DefEntity.SaveChanges() == 1)  
        {  
            dict.Add("Success", "Data Saved Successfully.");  
            return Ok(dict);  
        }  
        else  
        {  
            return Content(HttpStatusCode.BadRequest, "User Details not Saved.");  
        }  
    }  
    else  
    {  
        return GetErrorResult(result);  
    }  
}
```

## 压缩Web API的结果

Web API压缩对于提高ASP.NET Web API性能非常重要。在Web中，数据以包（数据包）的形式通过网络传输，从而增加了数据包的大小，这将增加大小并增加Web API的响应时间。因此，减小数据包大小可提高Web API的加载性能。

 

通过压缩API响应，我们具有以下两个优点。

- 数据大小将减小

- 响应时间将增加（增加客户端和服务器之间的通信速度。）

  我们可以通过编码和IIS中的某些设置来压缩Web API。我已经在以下链接中介绍了使用编码对Web API进行压缩的方法。



1. - [使用DotNetZip压缩Web API响应](http://www.c-sharpcorner.com/article/compressing-web-api-response-to-using-dotnetzip/)
   - [压缩Web API响应](http://www.c-sharpcorner.com/article/compressing-web-api-response-part-two/)

- 同样，我们可以通过检查动态内容压缩模块来启用IIS压缩 。

  ![](/iis1.png)

- 因此，通过这种方式，我们可以压缩Web API响应以实现性能。

## 使用缓存提高性能

 

缓存是一种在一定时间段内将常用数据或信息存储在本地存储器中的技术。因此，下一次，当客户端请求相同的信息时，它将从本地内存中提供信息，而不是从数据库中检索信息。缓存的主要优点是它通过减少处理负担来提高性能。我们有几种方法可以在Web api中实现缓存。在下面的链接中，我描述了一种实现缓存的方法。

- [在Web API中实现缓存](http://www.c-sharpcorner.com/article/implementing-caching-in-web-api/)

  因此，在这里您将找到我们如何在Web API中实现缓存以及它将如何帮助提高性能。

## 使用高速JSON序列化器

 

我们经常使用JSON而不是XML来在服务提供者和服务客户端之间交换数据。首先，我们使用它是因为JSON是轻量级的。

 

在.NET中，有很多序列化器。最受欢迎的是Json.NET，Microsoft选择它作为Web API的默认JSON序列化器。Json.NET之所以出色，是因为它快速，健壮且可配置。

 

有几种序列化器比Json.Net更快。下面提到其中一些。

![](/ser.png)

我们可以看到Protobuf-Net和JIL是非常快速的序列化程序，因此，如果可以代替Json.net来实现它们，则显然可以提高性能。协议缓冲区或Protobuf是Google的官方序列化程序。要在.Net中使用JIL序列化程序，我已经写了一篇文章，您可以在下面的链接中进行检查。

- [在C＃中使用Jil序列化器和反序列化器库](http://www.c-sharpcorner.com/article/working-with-jil-serializer-and-deserializer-library-in-c-sharp/)



## 创建适当的数据库结构

 

为了提高任何应用程序的性能，我们应该将重点放在数据库结构上。我可以说数据库结构在提高性能方面起着重要作用。这些是我们在处理数据库时应检查的一些注意事项。

- 尝试使规范化表结构。

- 为所有表提供适当的索引，以便从表中轻松搜索结果。

- 将所有相互关联的表与外键和主键相关联。

  我认为在创建数据库结构以提高性能时，我们必须至少遵循这三个规则。

**尝试从客户端验证某些属性**

 

从客户端而不是服务器端验证某些模型属性非常好。一旦我们在未经客户端验证的情况下放置了无效数据，则它将进入服务器并检查其是否为有效数据。如果它是无效的数据，它将从服务器给出错误。因此，在这里我们可以使用客户端验证检查往返行程。因此，如果我们可以通过客户端（移动设备，网站等）进行任何可能的验证，那将是很好的。

![](/valida.jpg)

 因此，我认为，如果我们专注于以上讨论的要点，则可以以某种方式提高Web API的性能。 