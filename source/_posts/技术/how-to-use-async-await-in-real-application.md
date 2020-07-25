---
title:  一文看懂"async"和“await”关键词是如何简化了C#中多线程的开发过程
date: 2020-7-25 17:35
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# **实际应用程序中的async和await**

### 一文看懂"async"和“await”关键词是如何简化了C#中多线程的开发过程

当我们使用需要长时间运行的方法（即，用于读取大文件或从网络下载大量资源）时，在同步的应用程序中，应用程序本身将停止运行，直到活动完成。在这些情况下，**异步**编程非常有用：它使我们能够并行执行不同任务，并在需要时等待其完成。

有这种方法编程许多不同的模型类型：**APM**（异步编程模型），基于事件（异步模型**EAP**），以及**TAP**，基于任务的（异步模型**任务**）。让我们看看如何使用关键字async和await在C＃中实现第三个方法。

编写异步代码的主要问题之一是可维护性：实际上，这种编程方法会使代码复杂化。幸运的是，C＃5引入了一种简化的方法，在该方法中，编译器运行由开发人员先前完成的艰巨任务，并且应用程序保留类似于同步代码的逻辑结构。

让我们举个例子。假设我们有一个.NET Core项目，我们应该在其中管理三个实体：Area，Company和Resource。

```c#
public class Area
{
    public int Id { get; set; }
 
    [Required]
    [StringLength(255)]
    public string Name { get; set; }
}
  
public class Company
{
    public int Id { get; set; }
 
    [Required]
    [StringLength(255)]
    public string Name { get; set; }
}
 
public class Resource
{
    public int Id { get; set; }
 
    [Required]
    [StringLength(255)]
    public string Name { get; set; }
}
```

 现在假设我们应该使用Entity Framework Core将这些实体的值保存在数据库中。其**DbContext**是： 

```c#
public class AppDbContext : DbContext
{
    public DbSet<Area> Areas { get; set; }
    public DbSet<Company> Companies { get; set; }
    public DbSet<Resource> Resources { get; set; }
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)  {}
  
    override protected void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Area> ().HasData(
            new Area { Id = 1, Name = "Area1"},
            new Area { Id = 2, Name = "Area2"},
            new Area { Id = 3, Name = "Area3"},
            new Area { Id = 4, Name = "Area4"},
            new Area { Id = 5, Name = "Area5"});
        modelBuilder.Entity<Company> ().HasData(
            new Area { Id = 1, Name = "Company1"},
            new Area { Id = 2, Name = "Company2"},
            new Area { Id = 3, Name = "Company3"},
            new Area { Id = 4, Name = "Company4"},
            new Area { Id = 5, Name = "Company5"});
        modelBuilder.Entity<Resource>().HasData(
            new Area { Id = 1, Name = "Resource1"},
            new Area { Id = 2, Name = "Resource2"},
            new Area { Id = 3, Name = "Resource3"},
            new Area { Id = 4, Name = "Resource4"},
            new Area { Id = 5, Name = "Resource5"});
    }
}
```

从代码中可以看到，我们插入了一些示例数据进行处理。现在假设我们要使用*Controller API*公开这些数据，既单独（针对每个实体），又使用将它们全部联接在一起的方法，并通过一次调用返回它们。

使用同步方法，*Controller API* 将是：

```c#
[ApiController]
[Route("[controller]")]
public class DataController : ControllerBase
{
    private readonly AppDbContext db = null;
     
    public DataController(AppDbContext db)
    {
        this.db = db;
    }
 
    public IActionResult Get()
    {
        var areas = this.GetAreas();
        var companies = this.GetCompanies();
        var resources = this.GetResources();
        return Ok(new { areas = areas, companies = companies, resources = resources });
    }
  
    [Route("areas")]
    public Area[] GetAreas() 
    {
        return this.db.Areas.ToArray();
    }
 
    [Route("companies")]
    public Company[] GetCompanies() 
    {
        return this.db.Companies.ToArray();
    }
  
    [Route("resources")]
    public Resource[] GetResources() 
    {
        return this.db.Resources.ToArray();
    }
}
```

 Get（）方法在其中调用返回单个结果的三个方法，并等待每个方法的执行完成后再传递到下一个结果。这三种方法互不相关，因此您无需等待其中一种方法的执行即可调用另一种方法。然后，您可以创建三个独立的任务以并行执行。
第一种方法可以基于该方法**运行**中的**任务**类队列指定的作业是在跑步*线程池*，并返回一个**任务**对象，它代表了这项工作。这样，方法可以在线程池的不同线程上同时运行： 

```
public IActionResult Get()
{
    var areas = Task.Run(() = > this.GetAreas());
    var companies = Task.Run(() = > this.GetCompanies());
    var resources = Task.Run(() = > this.GetResources());        
    Task.WhenAll(areas, companies, resources);
    return Ok(new { areas = areas.Result, companies = companies.Result, resources = resources.Result });
}
```

**Task**的**Result**属性包含详细说明的结果。方法*WhenAll*允许暂停当前线程执行，直到所有*Task*完成。运行代码，我们可以注意到一个有趣的事情：调用中断，并启动以下异常：

AggregateException：发生一个或多个错误。（在上一个操作完成之前，第二个操作在此上下文上开始。这通常是由使用相同DbContext实例的不同线程引起的。有关如何避免DbContext线程问题的更多信息，请参见[https://go.microsoft.com。 / fwlink /？linkid = 2097913。](https://go.microsoft.com/fwlink/?linkid=2097913)）

此错误消息告诉我们，方法在不同的线程上同时执行，但是由于它们使用与**DbContext** 相同的实例来连接数据库，
因此引发了异常，**DbContext**类无法确保*线程安全的*功能：我们可以轻松地绕过此问题，避免了.NET Core 的*依赖项注入*引擎创建单个实例，而我们为每种方法创建了单独的实例。作为示例，让我们看看方法GetAreas（）会如何变化：

```
public class DataController : ControllerBase
{
    private readonly DbContextOptionsBuilder <AppDbContext> optionsBuilder = null;
     
    public DataController(IConfiguration configuration)
    {
        this.optionsBuilder = new DbContextOptionsBuilder <AppDbContext> ()
            .UseSqlite(configuration.GetConnectionString("DefaultConnection"));
    }
  
    [Route("areas")]
    public Area[] GetAreas() 
    {
        using(var db = new AppDbContext(this.optionsBuilder.Options))
        {
            return db.Areas.ToArray();
        }
    }
}
```

 好吧，现在可以了。我们应该注意，EFCore提供了一些方法，例如，与方法*ToArrayAsync*一样，使用相同的**DbContext**进行异步调用，该方法从IQueryable *创建一个数组，该数组  异步枚举它。此方法返回**Task **，它是表示异步操作的活动。这样，我们不再需要使用*Task.Run（）： 

```
public IActionResult Get()
{
    var areas = this.GetAreas();
    var companies = this.GetCompanies();
    var resources = this.GetResources();
    Task.WhenAll(areas, companies, resources);
    return Ok(new { areas = areas.Result, companies = companies.Result, resources = resources.Result });
}
  
[Route("areas")]
public Task<Area[]> GetAreas() 
{
    return db.Areas.ToArrayAsync();
}
```

无论如何，Microsoft不能保证这些异步方法在每种情况下都能工作，因为DbContext尚未设计为线程安全的。您可以查询此链接以获取更多信息：[https](https://docs.microsoft.com/en-us/ef/core/querying/async) : [//docs.microsoft.com/zh-cn/ef/core/querying/async](https://docs.microsoft.com/en-us/ef/core/querying/async)

使用Entity Framework Core时，最佳实践是在启动另一个异步操作之前，为每个异步操作都拥有一个DbContext或等待每个异步操作完成。
当我们必须进行异步调用并返回结果时，这种最佳做法是可以的。但是，如果我们想在返回结果之前对结果进行一些操作，会发生什么？如果我们想向列表中添加元素怎么办？我们应该等待结果，添加元素，然后返回修改后的列表：

```
[Route("companies")]
public Task<Company[]> GetCompanies() 
{
    using (var db = new AppDbContext(this.optionsBuilder.Options))
    {
        var data = this.db.Companies.ToListAsync().Result;
        data.Insert(0, new Company() { Id = 0, Name = "-"});
        return data.ToArray();
    }
}
```

 不幸的是，该代码无法编译，因为data.ToArray（）返回的是数组而不是Task。实际上，这里我们需要三个线程：主调用方（Get（）），数据库查询（this.db.Companies.ToListAsync（））和一个线程，该线程将一个值添加到列表中。我们有三种方法可以做到这一点：让我们用三种单一方法来查看它们。我们已经看到的第一个，可以使用Task.Run（）方法： 

```
[Route("companies")]
public Task<Company[]> GetCompanies()
{
     return Task.Run(() =>
     {
          using (var db = new AppDbContext(this.optionsBuilder.Options))
          {
               var data = db.Companies.ToList();
               data.Insert(0, new Company() { Id = 0, Name = "-" });
               return data.ToArray();
          }
     });
}
```

 作为替代方案，我们可以使用方法**ContinueWith（）**，该方法可以应用于任务，并且可以在上一个方法完成后立即指定要运行的新任务： 

```
[Route("resources")]
public Task <Resource[]> GetResources()
{
     using (var db = new AppDbContext(this.optionsBuilder.Options))
     {
          return db.Resources.ToListAsync()
              .ContinueWith(dataTask = >
              {
                   var data = dataTask.Result;
                   dataTask.Result.Insert(0, new Resource() { Id = 0, Name = "-" });
                   return data.ToArray();
              });
     }
}
```

 我们可以让编译器执行“垃圾代码”，并使用关键字**async**和**await**，这可以为我们创建Task： 

```
[Route("areas")]
public async Task <Area[]> GetAreas()
{
     using (var db = new AppDbContext(this.optionsBuilder.Options))
     {
          var data = await db.Areas.ToListAsync();
          data.Insert(0, new Area() { Id = 0, Name = "-" });
          return data.ToArray();
     }
}
```

正如您在最后一种方法中看到的那样，代码更加简单，并且向我们隐藏了Task的创建，从而使我们可以异步返回。让我们想象一下一个场景，其中调用不止一个，并且这种方法如何使一切变得更加线性。

重构的副作用是方法GetAreas（）已成为**异步操作**。这个事实意味着，当不同的请求到达此API时，分配给该请求的线程池的线程将被释放以供其他请求使用，直到DbContext终止数据提取为止。

我希望我能引起您足够的兴趣来深入分析该论点。在许多情况下，使用async和await非常方便，并且除了使代码更加简洁和线性外，还可以提高应用程序的一般性能。

可以在这里找到代码：[https](https://github.com/fvastarella/Programmazione-asincrona-con-async-await)：[//github.com/fvastarella/Programmazione-asincrona-con-async-await](https://github.com/fvastarella/Programmazione-asincrona-con-async-await)