---
title:  ASP.NET Core中的内存中缓存
date: 2020-7-20 21:05
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# **ASP.NET Core中的内存缓存**

 **让我们看看如何通过缓存优化ASP.NET Core应用程序性能** 

我相信，在我们的工作中，每个人都收到来自客户的请求或来自我们应用程序用户的反馈，以提高响应速度。

如果在编写代码时仅使用**最佳实践**还不够，那么我们肯定需要使用缓存来微调我们的应用程序。

缓存包括将那些不经常更改的信息存储在某个地方。频率是我们应用程序的业务要求。

在本文中，我们将看到ASP.NET Core可用于缓存的内容。

## IMemoryCache和IDistributedCache

这两个接口代表.NET Core中用于缓存的*内置*机制。您可能已经听说过的所有其他技术都是这两个接口的实现。在本文中，我们将详细介绍*内存缓存*，而在以后的文章中将研究分布式缓存。

## 在ASP.NET Core中启用内存中缓存

我们可以使用**ConfigureServices**方法将ASP.NET内存中缓存合并到应用程序中。您可以在**Startup**类中启用*内存中缓存*，如下面的代码片段所示。

```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.AddMemoryCache();
}
```

 该**AddMemoryCache**方法允许我们注册*IMemoryCache*接口，如上面提到的，将被用于高速缓存的基础。下面我们看到框架中接口的定义： 

```c#
public interface IMemoryCache : IDisposable
{
    bool TryGetValue(object key, out object value);
    ICacheEntry CreateEntry(object key);
    void Remove(object key);
}
```

 接口中提供的方法并不是唯一可用于缓存的方法：我们将在后面看到，存在各种扩展来丰富可用的API并极大地促进其使用。例如： 

```c#
public static class CacheExtensions
{
    public static TItem Get<titem>(this IMemoryCache cache, object key);
     
    public static TItem Set<titem>(this IMemoryCache cache, object key, TItem value, MemoryCacheEntryOptions options);
 
    public static bool TryGetValue<titem>(this IMemoryCache cache, object key, out TItem value);
    ...
}
```

 注册后，该接口可以注入到我们要使用它的类构造函数中，如下所示： 

```c#
private IMemoryCache cache;
public MyCacheController(IMemoryCache cache)
    {
        this.cache = cache;
    }
```

在下面的部分中，我们将研究如何在ASP.NET Core中使用缓存API来存储和检索对象。

## 使用IMemoryCache存储和检索项目

要使用**IMemoryCache**接口写入对象，请使用**Set （）**方法，如以下代码片段所示。

```c#
[HttpGet]
public string Get()
{
    cache.Set(“MyKey”, DateTime.Now.ToString());
    return “This is a test method...”;
}
```

 此方法接受两个参数，第一个是键，用于标识缓存的对象，第二个参数是我们要存储的值。要从缓存中检索对象，请使用**Get （）**方法，如下面的代码片段所示。 

```c#
[HttpGet(“{key}”)]
public string Get(string key)
{
    return cache.Get<string>(key);
}
```

如果我们不确定缓存中是否存在特定的密钥，则可以使用TryGetValue（）方法进行帮助：它返回一个布尔值，指示所请求的密钥存在或不存在。

下面是如何使用*TryGetValue*修改*Get（）*方法的方法。

```c#
[HttpGet(“{key}”)]
public string Get(string key)
{
    string obj;
    if (!cache.TryGetValue<string>(key, out obj))
     
        obj = DateTime.Now.ToString();
        cache.Set<string>(key, obj);
     
    return obj;
}
```

 可用的另一种方法是**GetOrCreate（）**方法，该方法验证所需密钥的存在，否则，该方法将为您创建它。 

```c#
[HttpGet(“{key}”)]
public string Get(string key)
{
    return cache.GetOrCreate<string>(“key”,
        cacheEntry => {
            return DateTime.Now.ToString();
        });
}
```



## 如何在ASP.NET Core中的缓存数据上设置过期策略

当我们使用IMemoryCache存储对象时，**MemoryCacheEntryOptions**类为我们提供了各种技术来管理缓存数据的到期时间。

 我们可以指定一个固定的时间，在该时间之后某个特定的密钥将过期（**绝对过期**），或者如果在某个特定的时间（**滑动过期**）之后没有访问它，则它可以过期。此外，还可以使用**Expiration Token**在缓存的对象之间创建依赖关系。这里有一些例子 

```C#
//absolute expiration using TimeSpan
_cache.Set("key", item, TimeSpan.FromDays(1));
 
//absolute expiration using DateTime
_cache.Set("key", item, new DateTime(2020, 1, 1));
 
//sliding expiration (evict if not accessed for 7 days)
_cache.Set("key", item, new MemoryCacheEntryOptions
{
    SlidingExpiration = TimeSpan.FromDays(7)
});
 
//use both absolute and sliding expiration
_cache.Set("key", item, new MemoryCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromDays(30),
    SlidingExpiration = TimeSpan.FromDays(7)
})
 
// This method adds a trigger to refresh the data from background
private void UpdateReset()
{
    var mo = new MemoryCacheEntryOptions();
    mo.RegisterPostEvictionCallback(RefreshAllPlacessCache_PostEvictionCallback);
    mo.AddExpirationToken(new CancellationChangeToken(new CancellationTokenSource(TimeSpan.FromMinutes(35)).Token));
    Cache.Set(CACHE_KEY_PLACES_RESET, DateTime.Now, mo);
}
 
// Method triggered by the cancellation token that triggers the PostEvictionCallBack
private async void RefreshAllPlacesCache_PostEvictionCallback(object key, object value, EvictionReason reason, object state)
{
    // Regenerate a set of updated data
    var places = await GetLongGeneratingData();
    Cache.Set(CACHE_KEY_PLACES, places, TimeSpan.FromMinutes(40));
 
    // Re-set the cache to be reloaded in 35min
    UpdateReset();
}
```

## 缓存回调

**MemoryCacheEntryOptions**类提供的另一个有趣功能是允许我们注册回调的功能，当从缓存中删除一项时将执行该回调。

```c#
MemoryCacheEntryOptions cacheOption = new MemoryCacheEntryOptions()  
{  
    AbsoluteExpirationRelativeToNow = (DateTime.Now.AddMinutes(1) - DateTime.Now),  
};  
cacheOption.RegisterPostEvictionCallback(  
    (key, value, reason, substate) =>  
    {  
        Console.Write("Cache expired!");  
    }); 
```

##  缓存标签助手

到目前为止，我们已经看到.NET Core提供的API的使用，从而能够直接使用IMemoryCache接口手动在缓存中写入和读取项目。此接口还有其他实现，可能非常有用。例如，在Web环境中，如果我们使用.NET Core MVC框架，则可以使用**helper cache tag**存储部分页面。它非常简单易用：您可以将视图的一部分包装在缓存标签中以启用缓存：

```HTML
<cache>
<p>Ora: @DateTime.Now</p>
</cache>
```

 对于页面的每个后续请求（包含此标记），将从缓存中使用该段的正文。如果将其放在页面上并观察其输出，则可以轻松检查行为。当然，我们使用它的方式仅出于示例目的，但是当您尝试渲染需要大量资源的页面时，您可以欣赏它的功能。一个明显的缓存候选是视图[组件](https://www.blexin.com/en-US/Article/Blog/Kestrel-build-me-up-31)调用 

```html
<cache expires-on="@TimeSpan.FromSeconds(600)">
@await Component.InvokeAsync("BlogPosts", new { tag = "popular" })
</cache>
```

在上一个代码段中，您还可以通过属性**expires-on**来了解如何管理缓存中对象的**过期期限**。还有其他两种选择：

- 过期时间：将使用TimeSpan进行评估以指示一段时间，此后必须重新生成内容；
- 过期滑动：还应使用指示闲置时间的TimeSpan。每次从缓存中读取内容时，都会将其删除延迟。

另一个可定制方面涉及配置缓存标准的可能性。我们可能需要根据一些变量来更新缓存的对象。一些要求是由覆盖**变化逐**下列属性：

- 按路由变化：通过**路由参数**的名称（例如id）进行了增强，以指示在指示的属性更改时必须重新生成内容；
- 因查询而异：当更改**查询字符串键**时，将生成并缓存内容；
- 按用户不同：当我们显示已登录用户的特定数据时（例如，包含名称和照片的个人资料框），必须将其设置为true；
- 按标题变化：如果我们使用HTTP请求标头显示语言内容，则根据HTTP请求标头来更改缓存，例如“ Accept-Language”。
- cookie-variable-by-cookie：允许您根据cookie的内容更改缓存，我们必须指出其名称。

可以使用一个或多个按属性的属性来执行高级缓存策略，但是，有句著名的名言：“功能*强大，责任重大* ”。

## 结论

使用内存缓存可以使您将数据存储在服务器的内存中，并通过删除对外部数据源的不必要请求来帮助我们提高应用程序性能。如我们所见，它非常易于使用。

我提醒您，当您的应用程序托管在多台服务器或云托管环境中时，不能使用这种方法。在下一篇文章中，我们将讨论分布式缓存。

下次见！