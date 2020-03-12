[本文来源](https://www.stevejgordon.co.uk/httpclient-connection-pooling-in-dotnet-core)*于史蒂夫·戈登（Steve Gordon）是Microsoft MVP，Pluralsight的作者，布莱顿（英国西南部城市）的高级开发人员和社区负责人。他的个人博客为：*[www.stevejgordon.co.uk](http://www.stevejgordon.co.uk)。

***导读：***

*.NET Core（从2.1开始）中的HttpClient执行连接池和这些连接的生命周期管理。这支持使用单个HttpClient实例，通过单例减少了套接字耗尽的机会，同时确保连接定期重新连接以反映DNS更改。*

## 回顾HttpClient的历史
HttpClient最初是作为NuGet包开始的，该包可以选择包含在.NET Framework 4.0项目中。在.NET Framework 4.5中，它作为BCL（基本类库）的一部分在框中提供。它建立在预先存在的HttpWebRequest实现之上。在.NET Framework中，ServicePoint API可用于控制和管理HTTP连接，包括通过为端点配置ConnectionLeaseTimeout来设置连接寿命。

![图片](https://uploader.shimo.im/f/oKgF6qKs4QYLHxcE.png!thumbnail)

.NET Core 1.0最初于2016年6月发布。与.NET Framework中可用的版本相比，此第一个版本的API接口要小得多，主要用于构建ASP.NET Core Web应用程序。由于.NET Core 1.0是HttpClient，因此提供了API。但是，不包括用于HttpWebRequest和ServicePoint的API。.NET Core 1.0中的HttpClient直接建立在使用非托管代码的OS平台API之上，Windows API使用WinHTTP，Linux和Mac使用LibCurl。

![图片](https://uploader.shimo.im/f/eQR5yORTYbAFS4Th.png!thumbnail)

到2016年8月，很快就注意到，重新使用HttpClient实例以防止套接字耗尽的建议有一个相当麻烦的副作用。Oren Novotny（译者注：.NET基金会执行董事，.NET团队的项目经理）揭开了一个长期存在的GitHub问题，题为“ [Singleton HttpClient doesn’t respect DNS changes](https://github.com/dotnet/corefx/issues/11224) ”(单例HttpClient不遵守DNS 更改）。在此问题中，人们认识到重新使用单个HttpClient实例将导致连接无限期保持打开状态，因此，DNS更改可能会导致请求失败或与过时的终结点通信。

在.NET Core 2.0中，添加了HttpWebRequest以支持.NET Standard 2.0。它位于HttpClient实现的顶层，这与.NET Framework 4.5+中的工作原理相反。还添加了ServicePoint，尽管它的许多API接口要么要么会抛出未实现的异常，要么根本就没有实现。

![图片](https://uploader.shimo.im/f/phP4JDje5EI92hn3.png!thumbnail)

## 自.NET CORE 2.1以来的变化
这种有问题的行为导致团队不同团队进行了两项工作。ASP.NET团队开始研究**Microsoft.Extensions.Http**包，该包的主要功能是**IHttpClientFactory**。这个针对HttpClient实例自用的工厂还包括基础HttpMessageHandler链的生命周期管理。如果您想了解有关此功能的更多信息，可以查看我的[系列博客文章](https://www.stevejgordon.co.uk/introduction-to-httpclientfactory-aspnetcore)，我将在此介绍。 

IHttpClientFactory功能是作为ASP.NET Core 2.1的一部分发布的，对于许多人来说，这是一个很好的折衷方案，它解决了连接重用以及生命周期管理的问题。

在同一时间范围内，.NET团队正在研究自己的解决方案。该团队也在.NET Core 2.1中发布，在HttpClient的处理程序链的核心引入了一个新的**SocketsHttpHandler**。该处理程序直接建立在Socket API之上，并在托管代码中实现HTTP。这项工作的一部分包括连接池系统以及为这些连接设置最大生存期的能力。此功能将是本文其余部分的重点。

![图片](https://uploader.shimo.im/f/1Uacoirz6ok1CYur.png!thumbnail)

但是在开始之前，我想指出，虽然默认情况下从.NET Core 2.1启用了SocketsHttpHandler，但实现仅限于HTTP / 1.1通信。那些需要HTTP / 2的用户必须禁用该功能并使用较旧的处理程序链，该处理程序链像以前一样依赖非托管代码，并且不包括连接池。

幸运的是，.NET Core 3.0中已消除了此限制，并且现在提供了HTTP/2支持。这应该使用基于适合所有对象的SocketsHttpHandler链的HttpClient。

## 什么是连接池？
SocketsHttpHandler为每个唯一端点建立连接池，您的应用程序通过HttpClient向该唯一端点发出出站HTTP请求。在对端点的第一个请求上，当不存在现有连接时，将建立一个新的HTTP连接并将其用于该请求。该请求完成后，连接将保持打开状态并返回到池中。

对同一端点的后续请求将尝试从池中找到可用的连接。如果没有可用的连接，并且尚未达到该端点的连接限制，则将建立新的连接。达到连接限制后，请求将保留在队列中，直到连接可以自由发送它们为止。

我一直在研究此实现的内部代码，并可能在以后的博客文章中对池的行为进行更深入的分析。

## 如何控制连接池
有三个主要设置可用于控制连接池的行为。

**PooledConnectionLifetime**，定义连接在池中保持活动状态的时间。此生存期到期后，将不再为将来的请求而合并或发出连接。

**PooledConnectionIdleTimeout**，定义闲置连接在未使用时在池中保留的时间。一旦此生存期到期，空闲连接将被清除并从池中删除。

**MaxConnectionsPerServer**，定义每个端点将建立的最大出站连接数。每个端点的连接分别池化。例如，如果最大连接数为2，则您的应用程序将请求发送到两个[www.github.com](http://www.github.com/)和[www.google.com](http://www.google.com/)，总共可能最多有4个打开的连接。

默认情况下，从.NET Core 2.1开始，更高级别的HttpClientHandler将SocketsHttpHandler用作内部处理程序。没有任何自定义配置，将应用连接池的默认设置。

该**PooledConnectionLifetime**默认是无限的，因此，虽然经常使用的请求，连接可能会无限期地保持打开状态。该**PooledConnectionIdleTimeout**默认为2分钟，如果在连接池中长时间未使用将被清理。**MaxConnectionsPerServer**默认为int.MaxValue，因此连接基本上不受限制。

如果希望控制这些值中的任何一个，则可以手动创建SocketsHttpHandler实例，并根据需要进行配置。

```
var socketsHandler = new SocketsHttpHandler
	{
	    PooledConnectionLifetime = TimeSpan.FromMinutes(10),
	    PooledConnectionIdleTimeout = TimeSpan.FromMinutes(5),
	    MaxConnectionsPerServer = 10
	};
	

	var client = new HttpClient(socketsHandler);
```
在前面的示例中，对SocketsHttpHandler进行了配置，以使连接将最多在10分钟后停止重新发出并关闭。如果闲置5分钟，则连接将在池的清理过程中被更早地删除。我们还将最大连接数（每个端点）限制为十个。如果我们需要并行发出更多出站请求，则某些请求可能会排队等待，直到10个池中的连接可用为止。
要应用处理程序，它将被传递到HttpClient的构造函数中。

### 测试连接寿命
以这个示例程序为例：

```
using System;
	using System.Net.Http;
	using System.Threading.Tasks;
	

	namespace HttpConnectionPoolingSamples
	{
	    class Program
	    {
	        static async Task Main(string[] args)
	        {
	            var ips = await Dns.GetHostAddressesAsync("www.google.com");
	

	            foreach (var ipAddress in ips)
	            {
	                Console.WriteLine(ipAddress.MapToIPv4().ToString());
	            }
	            
	            var socketsHandler = new SocketsHttpHandler
	            {
	                PooledConnectionLifetime = TimeSpan.FromMinutes(10),
	                PooledConnectionIdleTimeout = TimeSpan.FromMinutes(5),
	                MaxConnectionsPerServer = 10
	            };
	

	            var client = new HttpClient(socketsHandler);
	            
	            for (var i = 0; i < 5; i++)
	            {
	                _ = await client.GetAsync("https://www.google.com");
	                await Task.Delay(TimeSpan.FromSeconds(2));
	            }
	

	            Console.WriteLine("Press a key to exit...");
	            Console.ReadKey();
	        }
	    }
	}
```
使用我们刚刚讨论的设置，此代码依次向同一端点发出5个请求。在每个请求之间，它会暂停两秒钟。该代码还输出从DNS检索到的Google服务器的IPv4地址。我们可以使用此IP地址来查看通过PowerShell中发出的netstat命令对其打开的连接：
```
netstat -ano | findstr 216.58.211
```
在我的例子中，此命令的输出为：
```
TCP   192.168.1.139:53040   216.58.211.164:443   ESTABLISHED   20372
```
我们可以看到，在这种情况下，到远程端点的连接只有1个。在每个请求之后，该连接将返回到池中，因此在发出下一个请求时可以重新使用。
如果我们更改连接的生存期，以使它们在1秒后过期，那么我们可以测试这对行为的影响：

```
using System;
	using System.Net;
	using System.Net.Http;
	using System.Threading.Tasks;
	

	namespace HttpConnectionPoolingSamples
	{
	    class Program
	    {
	        static async Task Main(string[] args)
	        {
	            var ips = await Dns.GetHostAddressesAsync("www.google.com");
	

	            foreach (var ipAddress in ips)
	            {
	                Console.WriteLine(ipAddress.MapToIPv4().ToString());
	            }
	

	            var socketsHandler = new SocketsHttpHandler
	            {
	                PooledConnectionLifetime = TimeSpan.FromSeconds(1),
	                PooledConnectionIdleTimeout = TimeSpan.FromSeconds(1),
	                MaxConnectionsPerServer = 10
	            };
	

	            var client = new HttpClient(socketsHandler);
	            
	            for (var i = 0; i < 5; i++)
	            {
	                _ = await client.GetAsync("https://www.google.com");
	                await Task.Delay(TimeSpan.FromSeconds(2));
	            }
	

	            Console.WriteLine("Press a key to exit...");
	            Console.ReadKey();
	        }
	    }
	}
```

```
TCP   192.168.1.139:53115   216.58.211.164:443   TIME_WAIT     0
TCP   192.168.1.139:53116   216.58.211.164:443   TIME_WAIT     0
TCP   192.168.1.139:53118   216.58.211.164:443   TIME_WAIT     0
TCP   192.168.1.139:53120   216.58.211.164:443   TIME_WAIT     0
TCP   192.168.1.139:53121   216.58.211.164:443   ESTABLISHED   25948
```
在这种情况下，我们可以看到使用了五个连接。其中的前四个在1秒后从池中删除，因此无法在下一个请求中重复使用。结果，每个请求都打开了一个新连接。现在，原始连接处于TIME_WAIT状态，并且操作系统无法将其重新用于新的出站连接。最终连接显示为ESTABLISHED，因为我在它过期之前就抓住了它。
### 测试最大连接数
对于下一个测试用例，我们将使用以下程序：

```
using System;
	using System.Diagnostics;
	using System.Linq;
	using System.Net;
	using System.Net.Http;
	using System.Threading.Tasks;
	

	namespace HttpConnectionPoolingSamples
	{
	    class Program
	    {
	        static async Task Main(string[] args)
	        {
	            var ips = await Dns.GetHostAddressesAsync("www.google.com");
	

	            foreach (var ipAddress in ips)
	            {
	                Console.WriteLine(ipAddress.MapToIPv4().ToString());
	            }
	

	            var socketsHandler = new SocketsHttpHandler
	            {
	                PooledConnectionLifetime = TimeSpan.FromSeconds(60),
	                PooledConnectionIdleTimeout = TimeSpan.FromMinutes(20),
	                MaxConnectionsPerServer = 2
	            };
	

	            var client = new HttpClient(socketsHandler);
	

	            var sw = Stopwatch.StartNew();
	

	            var tasks = Enumerable.Range(0, 200).Select(i => client.GetAsync("https://www.google.com"));
	

	            await Task.WhenAll(tasks);
	

	            sw.Stop();
	

	            Console.WriteLine($"{sw.ElapsedMilliseconds}ms taken for 200 requests");
	

	            Console.WriteLine("Press a key to exit...");
	            Console.ReadKey();
	        }
	    }
	}
```
该代码将MaxConnectionsPerServer限制为2。然后启动200个任务，每个任务都向同一端点发出HTTP请求。这些任务将同时运行。所有请求竞争所花费的时间将写入控制台。
在我的机器上运行此命令后，输出为：

```
 8013ms taken for 200 requests
```
如果使用netstat查看连接，则根据定义的限制，我们可以看到两个已建立的连接。
已建立

```
TCP   192.168.1.139:52780   216.58.204.36:443   ESTABLISHED   16076
TCP   192.168.1.139:52780   216.58.204.36:443   ESTABLISHED   16076
```
如果我们调整此代码以允许MaxConnectionsPerServer = 10，则可以重新运行该应用程序。这次所花费的时间减少了大约4倍。
```
2123ms taken for 200 requests
```
当我们查看连接时，我们可以看到确实建立了十个连接。
```
TCP   192.168.1.139:52798   216.58.204.36:443   ESTABLISHED   30856
TCP   192.168.1.139:52799   216.58.204.36:443   ESTABLISHED   30856
TCP   192.168.1.139:52800   216.58.204.36:443   ESTABLISHED   30856
TCP   192.168.1.139:52801   216.58.204.36:443   ESTABLISHED   30856
TCP   192.168.1.139:52802   216.58.204.36:443   ESTABLISHED   30856
TCP   192.168.1.139:52803   216.58.204.36:443   ESTABLISHED   30856
TCP   192.168.1.139:52804   216.58.204.36:443   ESTABLISHED   30856
TCP   192.168.1.139:52805   216.58.204.36:443   ESTABLISHED   30856
TCP   192.168.1.139:52806   216.58.204.36:443   ESTABLISHED   30856
TCP   192.168.1.139:52807   216.58.204.36:443   ESTABLISHED   30856
```
结果，提高了吞吐量。我们允许更多的出站连接，因此可以更快地处理请求队列，并通过额外的连接并行发出更多请求。
## 我还需要IHttpClientFactory吗？
这是一个非常合乎逻辑的问题，可能是该帖子的结果。IHttpClientFactory的功能之一是HttpMessageHandler链的生命周期管理，因此也是基础连接的生命周期管理。有了HttpClient和SocketsHttpHandler可以达到相同效果的知识，我们是否需要使用IHttpClientFactory？

我的观点是，IHttpClientFactory除了帮助管理连接生存期外还有其他好处，并且在发出出站HTTP请求时仍然可以增加价值。它提供了一种很好的模式，可以使用[命名或类型化的客户端方法](https://www.stevejgordon.co.uk/httpclientfactory-named-typed-clients-aspnetcore)为HttpClient实例定义逻辑配置。后来有类型的客户是我个人的最爱。

这些逻辑客户端的流畅配置方法还使[定制的DelegatingHandlers](https://www.stevejgordon.co.uk/httpclientfactory-aspnetcore-outgoing-request-middleware-pipeline-delegatinghandlers)与客户端的使用非常简单明了。这包括ASP.NET团队对该方法的扩展，以便[与Polly集成，](https://www.stevejgordon.co.uk/httpclientfactory-using-polly-for-transient-fault-handling)以便轻松地对出站请求应用弹性和瞬时故障处理。

即使没有生命周期管理，我也希望在将来的一段时间内将工厂用于我的应用程序。根据我在网上看到的讨论，很有可能在将来的版本中，寿命管理功能将从IHttpClientFactory中弃用和/或删除，因为它解决的问题不再适用。

## 摘要
在本文中，我们看到自从.NET Core 2.1发布以来，使用默认的SocketsHttpHandler实现时，将维护连接池。使用池的设置，我们可以控制连接的生存期并限制每个端点可能创建的出站连接的数量。

我们还讨论了IHttpClientFactory不仅具有连接生存期管理的优点和功能，因此仍然是一个有价值的工具。

