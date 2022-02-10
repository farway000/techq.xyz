---
title:  ASP.NET Core中的内存中缓存
date: 2020-7-21 21:05
tags: 技术
author: 邹溪源
categories:
  - 技术
---

# **ASP.NET Core中的分布式缓存**

在上[一篇文章中](https://www.blexin.com/en-US/Article/Blog/In-memory-caching-in-ASPNET-Core-45)，我解释了如何使用***内存缓存***在ASP.NET Core应用程序中管理***缓存***。如果您的应用程序托管在单个服务器上，则可以使用这种类型的缓存。

那.NET Core框架可以使用哪些工具在云中的分布式方案中进行缓存呢？

## IDistributedCache接口

该框架提供以下接口：

```
public interface IDistributedCache
{
    byte[] Get(string key);
    Task <byte[]> GetAsync(string key);

    void Set(string key, byte[] value, DistributedCacheEntryOptions options);
    Task SetAsync(string key, byte[] value, DistributedCacheEntryOptions options);
     
    void Refresh(string key);
    Task RefreshAsync(string key);
     
    void Remove(string key);
    Task RemoveAsync(string key);

}
```

如果您还记得上一篇文章（**IMemoryCache**）中使用的接口，则可能会注意到一些相似之处，但甚至有很多区别，例如：

1. 异步方法
2. 刷新方法（无需请求数据即可更新缓存键）
3. 基于字节而非基于对象的方法

到目前为止，Microsoft提供了3种实现：一种是**内存中**的，主要用于应用程序的开发阶段，第二种是在**SQL Server上**，最后一种是基于**REDIS**。
要使用基于REDIS的版本，必须安装NuGet软件包**Microsoft.Extensions.Caching.Redis**。

## 分布式内存缓存

该实现由框架提供，并将我们的数据保存在内存中。它不是完全分布式的缓存，因为数据是从应用程序实例保存在其所在的服务器上的。在某些情况下可能会有用：

- 开发与测试
- 在生产中使用单个服务器时，内存消耗不是问题。

无论如何，我们将使我们的应用程序仅在必要时才使用“真实的”分布式解决方案（用于保存的接口是相同的）。
要启用此IDistributedCache实现，您只需要在Startup类中注册它，如下所示：

[?](https://www.blexin.com/en-US/Article/Blog/Distributed-cache-in-ASPNET-Core-53#)

```C#
`services.AddDistributedMemoryCache();`
```

## 分布式SQL Server缓存

此实现使分布式缓存可以将SQL Server数据库用作存储。有一些配置步骤。第一步包括创建用于保留数据的表。
有一个命令行命令工具，可通过简单的指令轻松实现

[?](https://www.blexin.com/en-US/Article/Blog/Distributed-cache-in-ASPNET-Core-53#)

```
`dotnet sql-cache ``create` `<``connection` `string > <``schema` `> <``table` `>`
```

允许创建具有以下结构的表：

[?](https://www.blexin.com/en-US/Article/Blog/Distributed-cache-in-ASPNET-Core-53#)

```
`CREATE` `TABLE` `[dbo].[CacheTable](``  ``[Id] [nvarchar](449) ``NOT` `NULL``,``  ``[Value] [varbinary](``max``) ``NOT` `NULL``,``  ``[ExpiresAtTime] [datetimeoffset](7) ``NOT` `NULL``,``  ``[SlidingExpirationInSeconds] [``bigint``] ``NULL``,``  ``[AbsoluteExpiration] [datetimeoffset](7) ``NULL``,`` ``CONSTRAINT` `[pk_Id] ``PRIMARY` `KEY` `CLUSTERED ``(``  ``[Id] ``ASC``)``WITH` `(PAD_INDEX = ``OFF``, STATISTICS_NORECOMPUTE = ``OFF``, ``  ``IGNORE_DUP_KEY = ``OFF``, ALLOW_ROW_LOCKS = ``ON``, ``  ``ALLOW_PAGE_LOCKS = ``ON``) ``ON` `[``PRIMARY``]``) ``ON` `[``PRIMARY``] TEXTIMAGE_ON [``PRIMARY``]` `CREATE` `NONCLUSTERED ``INDEX` `[Index_ExpiresAtTime] ``ON` `[dbo].[CacheTable]``(``  ``[ExpiresAtTime] ``ASC``)``WITH` `(PAD_INDEX = ``OFF``, STATISTICS_NORECOMPUTE = ``OFF``, ``  ``SORT_IN_TEMPDB = ``OFF``, DROP_EXISTING = ``OFF``, ``  ``ONLINE = ``OFF``, ALLOW_ROW_LOCKS = ``ON``, ``  ``ALLOW_PAGE_LOCKS = ``ON``) ``ON` `[``PRIMARY``]`
```

配置此实现的第二个也是最后一个步骤，即在我们的应用程序中进行注册。在Startup类的ConfigureServices方法内，我们添加以下代码块：

[?](https://www.blexin.com/en-US/Article/Blog/Distributed-cache-in-ASPNET-Core-53#)

```c#
`services.AddDistributedSqlServerCache(o =>``{``  ``o.ConnectionString = Configuration[``"ConnectionStrings:Default"``];``  ``o.SchemaName = ``"dbo"``;``  ``o.TableName = ``"Cache"``;``});`
```

## 分布式Redis缓存

在分布式缓存场景中，Redis的使用非常广泛。Redis是一个快速存储的数据存储，它是开源的并且是键值类型。它提供的响应时间不到1毫秒，从而允许在各个领域中的每个实时应用程序每秒接收数百万个请求。
如果您的应用程序的基础结构基于Azure云，则可以使用服务**Azure Redis缓存**，并且可以从您的订阅中对其进行配置。

要在Redis上使用分布式缓存，必须安装软件包Microsoft.Extensions.Caching.Redis。
第一步是在Startup.class中配置服务[?](https://www.blexin.com/en-US/Article/Blog/Distributed-cache-in-ASPNET-Core-53#)

```
`public` `void` `ConfigureServices(IServiceCollection services)``{`` ``// Add framework services.`` ``// ... altri servizi ...`` ` ` ``services.AddDistributedRedisCache(cfg => `` ``{``  ``cfg.Configuration = Configuration.GetConnectionString(``"redis"``);`` ``});``}`
```

如果使用的是Azure Redis缓存，则可以通过访问面板的“访问键”部分找到连接字符串。

![img](https://www.blexin.com/storage/articles/cache/img02.png)

## 如何使用IDistributedCache

接口IDistributedCache的使用特别简单：如果您已经阅读了有关接口IMemoryCache的使用的文章，那么您将在命令中找到几点。如果要在控制器中使用它，则需要首先将实例注入构造函数中，如下面的代码所示：

[?](https://www.blexin.com/en-US/Article/Blog/Distributed-cache-in-ASPNET-Core-53#)

```
`public` `class` `HomeController : Controller``{``  ``private` `readonly` `IDistributedCache _cache;` `  ``public` `HomeController(IDistributedCache cache)``  ``{``    ``_cache = cache;``  ``}``  ``public` `async Task  Index()``  ``{``    ``await _cache.SetStringAsync(``"TestString"``, ``"TestValue"``);` `    ``var value = _cache.GetString(``"TestString"``);` `    ``return` `View();``  ``}``}`
```

如果让此代码运行，并在Index方法的最后一行插入一个断点，则会得到以下结果：

![img](https://www.blexin.com/storage/articles/cache/img01.png)

我们可以使用**DistributedCacheEntryOptions**类轻松检查缓存的持续时间。在下面的代码中，我们创建一个实例，将持续时间设置为一小时。

[?](https://www.blexin.com/en-US/Article/Blog/Distributed-cache-in-ASPNET-Core-53#)

```
`public` `async Task  Index()``{``  ``var options = ``new` `DistributedCacheEntryOptions``  ``{``    ``AbsoluteExpiration = DateTime.Now.AddHours(1)``  ``};` `  ``await _cache.SetStringAsync(``"TestString"``, ``"TestValue"``, options);` `  ``var value = _cache.GetString(``"TestString"``);` `  ``return` `View();``}`
```

## 结论与建议

在我们的应用中使用哪种IDistributedCache实现的决定取决于某些因素。在Redis和SQL之间进行选择（我将内存实现仅保留在测试和开发中，所以我将其保留在外）应该基于对您可用的基础结构，性能要求和开发团队的经验进行选择。如果团队对Redis感到放心，这将是最佳选择。SQL实现仍然是一个很好的解决方案，但应记住，数据恢复将不会提供出色的性能，因此应谨慎选择要缓存的数据。