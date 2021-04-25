# [在ASP.NET Core中的应用启动时运行异步任务-第1部分](https://andrewlock.net/series/running-async-tasks-on-app-startup-in-asp-net-core/)

这是该系列的第一篇文章：[在ASP.NET Core中的应用程序启动时运行异步任务](https://andrewlock.net/series/running-async-tasks-on-app-startup-in-asp-net-core/)。

1. 第1部分-用于运行异步任务的内置选项（本文）
2. 第2部分-运行异步任务的两种方法
3. 第3部分-对异步任务示例和其他可能解决方案的反馈
4. 第4部分-使用运行状况检查在ASP.NET Core中运行异步任务
5. 第5部分-在ASP.NET Core 3.0中的应用启动时运行异步任务

有时，您需要先执行一次性初始化逻辑，然后应用才能正常启动。例如，您可能要[验证您的配置正确](https://andrewlock.net/adding-validation-to-strongly-typed-configuration-objects-in-asp-net-core/)，填充缓存或[运行数据库迁移](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/#apply-migrations-at-runtime)。在这篇文章中，我将介绍可用的选项，并展示一些我认为可以很好解决问题的简单方法和扩展点。

我首先描述使用来运行同步任务的内置解决方案`IStartupFilter`。然后，我逐步介绍了用于运行异步任务的各种选项。您可以（但可能不应该）使用`IStartupFilter`或`IApplicationLifetime`事件来运行异步任务。您可以使用该`IHostedService`界面运行一次性任务，而不会阻止应用程序启动。但是，唯一真正的解决方案是在*program.cs中*手动运行任务。在我的下一篇文章中，我将显示一个建议的建议，该建议使此过程更容易一些。

## 为什么我们需要在应用启动时运行任务？[![img](https://andrewlock.net/assets/img/icons-link.svg)](https://andrewlock.net/running-async-tasks-on-app-startup-in-asp-net-core-part-1/#why-do-we-need-to-run-tasks-on-app-startup-)

在应用能够启动并开始处理请求之前，通常需要运行各种初始化代码。在ASP.NET Core应用程序中，显然有很多事情需要发生，例如：

- 确定当前的托管环境
- 从*appsettings.json*和环境变量加载配置
- 依赖项注入容器的配置
- 依赖项注入容器的*构建*
- 中间件管道的配置

所有这些步骤都需要执行以引导应用程序。但是，*在*`WebHost`运行并开始侦听请求*之前*，通常要执行一次性任务。例如：

- [检查您的强类型配置是否有效](https://andrewlock.net/adding-validation-to-strongly-typed-configuration-objects-in-asp-net-core/)。
- 使用数据库或API中的数据启动/填充缓存
- 在启动应用程序之前运行数据库迁移。（[这通常不是一个好主意，但对于某些应用程序可能已经足够了](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/#apply-migrations-at-runtime)）。

有时，这些任务不*具有*被运行之前，你的应用程序启动服务请求。例如，缓存启动示例-如果其性能良好，则在启动之前查询缓存是否无关紧要。另一方面，您几乎可以肯定要在应用程序开始处理请求之前迁移数据库！

有一些示例表明ASP.NET Core框架本身需要一次性的初始化任务。[数据保护子系统](https://docs.microsoft.com/en-us/aspnet/core/security/data-protection/introduction?view=aspnetcore-2.1)就是一个很好的例子，该[子系统](https://docs.microsoft.com/en-us/aspnet/core/security/data-protection/introduction?view=aspnetcore-2.1)用于瞬时加密（cookie值，防伪令牌等）。在应用程序可以开始处理任何请求之前，必须初始化此子系统。为了解决这个问题，他们使用`IStartupFilter`。

## 与同步运行任务 `IStartupFilter`[![img](https://andrewlock.net/assets/img/icons-link.svg)](https://andrewlock.net/running-async-tasks-on-app-startup-in-asp-net-core-part-1/#running-tasks-synchronously-with-istartupfilter)

我之前已经写过有关的文章`IStartupFilter`，因为在您的工具栏中可以使用它来定制应用程序是一个非常有用的界面：

- [在ASP.NET Core中探索IStartupFilter](https://andrewlock.net/exploring-istartupfilter-in-asp-net-core/)（`IStartupFilter`的简介）
- [通过Middleware Analysis软件包了解您的中间件管道](https://andrewlock.net/understanding-your-middleware-pipeline-with-the-middleware-analysis-package/)（剧透警报-使用`IStartupFilter`）
- [向ASP.NET Core中的强类型配置对象添加验证](https://andrewlock.net/adding-validation-to-strongly-typed-configuration-objects-in-asp-net-core/)（再次使用`IStartupFilter`）

如果您不熟悉过滤器，建议阅读我的介绍性文章，但是我将在此处提供一个简短的摘要。

`IStartupFilter`在配置中间件管道的过程中执行（通常在中完成`Startup.Configure()`）。它们允许您通过插入额外的中间件，对它进行分叉或执行许多其他操作来定制由应用程序实际创建的中间件管道。例如，[以下`AutoRequestServicesStartupFilter` ](https://github.com/aspnet/Hosting/blob/27e4e1aca3863389098b9be6dd9c5b7c030180b8/src/Microsoft.AspNetCore.Hosting/Internal/AutoRequestServicesStartupFilter.cs)所示在管道的开头插入了一个新的中间件：

```csharp
public class AutoRequestServicesStartupFilter : IStartupFilter
{
    public Action<IApplicationBuilder> Configure(Action<IApplicationBuilder> next)
    {
        return builder =>
        {
            builder.UseMiddleware<RequestServicesContainerMiddleware>();
            next(builder);
        };
    }
}
```

这很有用，但是与在应用程序启动时运行一次性任务有什么关系？

该`IStartupFilter`所提供的主要功能是，在设置了配置并配置了依赖项注入容器之后，但在应用程序准备启动之前，可以钩住早期的应用程序启动过程。这意味着您可以将依赖注入与`IStartupFilter`s一起使用，因此几乎可以运行任何代码。而[`DataProtectionStartupFilter`](https://github.com/aspnet/DataProtection/blob/2.2.0-preview3/src/Microsoft.AspNetCore.DataProtection/Internal/DataProtectionStartupFilter.cs)则用于初始化数据保护系统。我使用了类似的`IStartupFilter`方法来提供[对强类型配置的热验证](https://andrewlock.net/adding-validation-to-strongly-typed-configuration-objects-in-asp-net-core/)。

另一个非常有用的功能是，它允许您通过在DI容器中注册服务来添加要执行的任务。这意味着作为源代码作者，您可以注册要在应用程序启动时运行的任务，而无需应用程序作者显式调用它。

那么为什么我们不能只`IStartupFilter`在启动时用来运行异步任务呢？

问题在于`IStartupFilter`基本上是*同步的*。该`Configure()`方法（您可以在上面的代码中看到）没有返回a `Task`，因此[尝试通过async进行同步不是一个好主意](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md#asynchrony-is-viral)。我稍后再讨论，但是现在绕道而行。

## 为什么不使用健康检查？[![img](https://andrewlock.net/assets/img/icons-link.svg)](https://andrewlock.net/running-async-tasks-on-app-startup-in-asp-net-core-part-1/#why-not-use-health-checks-)

ASP.NET Core 2.2为ASP.NET Core应用程序引入了运行状况检查功能，该功能使您可以查询通过HTTP端点公开的应用程序的“运行状况”。部署后，编排引擎（如Kubernetes）或反向代理（如HAProxy和NGINX）可以查询此终结点，以检查您的应用是否准备好开始接收请求。

您*可以*使用运行状况检查功能来确保您的应用程序在所有必需的一次性任务完成之前不会开始服务请求（即从运行状况检查端点返回“运行状况”状态）。但是，这有一些缺点：

- 在`WebHost`已执行的一次性任务前，探针会启动。尽管他们不会收到“实际”请求（仅运行状况检查请求），但这仍然可能是一个问题。
- 它引入了额外的复杂性。除了添加代码以运行一个任务之外，您还需要添加运行状况检查以测试任务是否完成，并同步任务的状态。
- 应用程序的启动仍会延迟到所有任务都完成为止，因此不太可能减少启动时间。
- 如果任务失败，则该应用将继续以“死”状态运行，在此状态下，运行状况检查将永远不会通过。这可能是可以接受的，但就我个人而言，我更喜欢一个应用程序立即失败。
- 运行状况检查方面仍未定义*如何*实际运行任务，仅定义任务是否成功完成。您仍然需要确定一种在启动时运行任务的机制。

对我来说，健康检查似乎不适合一次性任务的情况。对于我描述的某些示例，它们可能很有用，但我认为它们并不适合所有情况。我真的很想能够在应用启动时运行一次性任务，然后再运行`WebHost`。

## 运行异步任务[![img](https://andrewlock.net/assets/img/icons-link.svg)](https://andrewlock.net/running-async-tasks-on-app-startup-in-asp-net-core-part-1/#running-asynchronous-tasks)

我花了很长时间讨论所有*无法*实现目标的方法，以及一些解决方案！在本节中，我将介绍一些运行异步任务（即返回a `Task`并需要`await`-ing的任务）的可能性。有些比其他的要好，有些则应避免，但我想介绍各种选择。

为了具体讨论，我将考虑[数据库迁移示例](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/#apply-migrations-at-runtime)。在EF Core中，您可以在运行时通过调用迁移数据库`myDbContext.Database.MigrateAsync()`，其中`myDbContext`是应用程序的实例`DbContext`。

> 该方法还有一个同步版本`Database.Migrate()`，但只是假装目前没有！

### 1.使用 `IStartupFilter`[![img](https://andrewlock.net/assets/img/icons-link.svg)](https://andrewlock.net/running-async-tasks-on-app-startup-in-asp-net-core-part-1/#1-using-istartupfilter)

前面已经介绍了如何在应用程序启动时`IStartupFilter`用于运行*同步*任务。不幸的是，运行*异步*任务的唯一方法是使用“异步同步”方法，我们称之为`GetAwaiter().GetResult()`：

> 警告：此代码使用了不合适的`async`做法。

```csharp
public class MigratorStartupFilter: IStartupFilter
{
    // We need to inject the IServiceProvider so we can create 
    // the scoped service, MyDbContext
    private readonly IServiceProvider _serviceProvider;
    public MigratorStartupFilter(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public Action<IApplicationBuilder> Configure(Action<IApplicationBuilder> next)
    {
        // Create a new scope to retrieve scoped services
        using(var scope = _seviceProvider.CreateScope())
        {
            // Get the DbContext instance
            var myDbContext = scope.ServiceProvider.GetRequiredService<MyDbContext>();

            //Do the migration by blocking the async call
            myDbContext.Database.MigrateAsync()
                .GetAwaiter()   // Yuk!
                .GetResult();   // Yuk!
        }

        // don't modify the middleware pipeline
        return next;
    }
}
```

这很可能不会引起任何问题-此代码仅在应用程序启动时运行，然后再处理请求，因此似乎不太可能出现死锁。但是坦率地说，我不能肯定地说，我会尽可能避免这样的代码。

> [大卫·福勒](https://twitter.com/davidfowl)（[David Fowler](https://twitter.com/davidfowl)）整理了[一些正确的异步编程指南](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/0ba7625050f975f8a7df1df57c80ad08da250541/AsyncGuidance.md)。我强烈建议您阅读！

### 2.使用`IApplicationLifetime`事件[![img](https://andrewlock.net/assets/img/icons-link.svg)](https://andrewlock.net/running-async-tasks-on-app-startup-in-asp-net-core-part-1/#2-using-iapplicationlifetime-events)

我之前没有讨论过太多，但是当您的应用程序[通过`IApplicationLifetime`界面](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime?view=aspnetcore-2.1)启动和关闭时，您可以收到通知。我这里不做详细介绍，因为它对于我们的目的有一些问题：

- `IApplicationLifetime`使用`CancellationToken`s来注册回调，这意味着您只能同步执行回调。从本质上讲，这意味着您无论采取什么操作，都必须坚持使用[异步](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/0ba7625050f975f8a7df1df57c80ad08da250541/AsyncGuidance.md#avoid-using-taskresult-and-taskwait)模式进行[同步](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/0ba7625050f975f8a7df1df57c80ad08da250541/AsyncGuidance.md#avoid-using-taskresult-and-taskwait)。
- 该`ApplicationStarted`事件仅在开始了WebHost后运行，所以任务应用程序启动时接受请求后运行。

鉴于其没有解决`IStartupFilter`的异步同步问题，也没有阻止应用启动，因此我们将`IApplicationLifetime`继续进行下一个可能性。

### 3. `IHostedService`用于运行异步任务[![img](https://andrewlock.net/assets/img/icons-link.svg)](https://andrewlock.net/running-async-tasks-on-app-startup-in-asp-net-core-part-1/#3-using-ihostedservice-to-run-asynchronous-tasks)

`IHostedService`允许ASP.NET Core应用[在应用生命周期内在后台执行长时间运行的任务](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/multi-container-microservice-net-applications/background-tasks-with-ihostedservice)。它们有许多不同的用途-您可以使用它们在计时器上运行定期任务，处理其他消息传递范例，例如RabbitMQ消息或许多其他事情。在ASP.NET Core 3.0中，[甚至ASP.NET Web主机也可能建立在之上`IHostedService`](https://github.com/aspnet/Hosting/pull/1580)。

在`IHostedService`本质上是异步的，同时具有`StartAsync`和`StopAsync`功能。这对我们来说很棒，因为这意味着不再需要通过异步进行同步！将数据库迁移器实现为托管服务可能看起来像这样：

```csharp
public class MigratorHostedService: IHostedService
{
    // We need to inject the IServiceProvider so we can create 
    // the scoped service, MyDbContext
    private readonly IServiceProvider _serviceProvider;
    public MigratorStartupFilter(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public async Task StartAsync(CancellationToken cancellationToken)
    {
        // Create a new scope to retrieve scoped services
        using(var scope = _seviceProvider.CreateScope())
        {
            // Get the DbContext instance
            var myDbContext = scope.ServiceProvider.GetRequiredService<MyDbContext>();

            //Do the migration asynchronously
            await myDbContext.Database.MigrateAsync();
        }
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        // noop
        return Task.CompletedTask;
    }
}
```

不幸的是，`IHostedService`这不是我们所希望的灵丹妙药。它允许我们编写真正的异步代码，但是它有两个问题：

- `IHostedService`s 的典型实现期望`StartAsync`函数返回相对较快。对于后台服务，预计您将异步启动该服务，但是大部分工作将在该启动代码之外进行（[请参阅docs中的示例](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/multi-container-microservice-net-applications/background-tasks-with-ihostedservice#implementing-ihostedservice-with-a-custom-hosted-service-class-deriving-from-the-backgroundservice-base-class)）。因此，“内联”迁移数据库不是问题，但是它将阻止其他`IHostedService`s的启动，这可能会或可能不会发生。
- `IHostedService.StartAsync()`被称为*后*的`WebHost`启动，所以你不能用这种方式来运行的任务*之前，*您的应用程序启动。

最大的问题是第二个问题-应用程序将在`IHostedService`运行数据库迁移之前开始接受请求，这不是我们想要的。回到绘图板。

### 4.在Program.cs中手动运行任务[![img](https://andrewlock.net/assets/img/icons-link.svg)](https://andrewlock.net/running-async-tasks-on-app-startup-in-asp-net-core-part-1/#4-manually-running-tasks-in-program-cs)

到目前为止，没有显示解决方案可以提供完整的解决方案。他们要么需要在异步编程上使用同步（虽然在应用程序启动的上下文中可以这样做，但并不鼓励这样做），或者不阻止应用程序启动。到目前为止，有一个简单的解决方案我已经忽略了，那就是停止尝试使用框架机制，而是自己完成这项工作。

ASP.NET Core模板中使用的默认*Program.cs*会`IWebHost`在`Main`函数中生成并运行一条语句：

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

但是，在`Build()`创建之后`WebHost`，但是在您调用之前，没有什么阻止您运行代码`Run()`。再加上允许您的`Main`函数异步的C＃7.1功能，我们有一个合理的解决方案：

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        IWebHost webHost = CreateWebHostBuilder(args).Build();

        // Create a new scope
        using (var scope = webHost.Services.CreateScope())
        {
            // Get the DbContext instance
            var myDbContext = scope.ServiceProvider.GetRequiredService<MyDbContext>();

            //Do the migration asynchronously
            await myDbContext.Database.MigrateAsync();
        }

        // Run the WebHost, and start accepting requests
        // There's an async overload, so we may as well use it
        await webHost.RunAsync();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

该解决方案具有许多优点：

- 我们现在正在执行真正的异步，不需要通过异步进行同步
- 我们可以异步执行任务
- 在执行任务后，该应用才接受请求
- 此时已构建了DI容器，因此我们可以使用它来创建服务。

不幸的是，这并非所有的好消息。我们仍然缺少一些东西：

- 即使已经构建了DI容器，也没有中间件管道。这不会发生，直到你打电话`Run()`或`RunAsync()`上`IWebHost`。到那时，构建了中间件管道，`IStartupFilter`执行了，然后启动了应用程序。如果您的异步任务需要在以下任何步骤中进行配置，那么您就不走运了
- 通过将服务添加到DI容器，我们失去了自动运行任务的能力。我们必须记住要手动运行任务。

如果这些警告不是问题，那么我认为此最终选择将为解决该问题提供最佳解决方案。在我的下一篇文章中，我将展示一些方法，我们可以采用这个基本示例并在其基础上进行构建，以使某些操作变得更容易使用。

## 摘要[![img](https://andrewlock.net/assets/img/icons-link.svg)](https://andrewlock.net/running-async-tasks-on-app-startup-in-asp-net-core-part-1/#summary)

在本文中，我讨论了在应用启动时异步运行任务的需求。我描述了这样做的一些挑战。对于同步任务，`IStartupFilter`它提供了一个有用的钩子，可以连接到ASP.NET Core应用程序的启动过程，但是运行异步任务需要通过异步编程进行同步，这通常是一个坏主意。我描述了许多用于运行异步任务的可能选项，我发现最好的方法是在*Program.cs中* “手动”运行任务，并在构建`IWebHost`和运行它之间进行。在下一篇文章中，我将提供一些代码来简化此模式的使用。