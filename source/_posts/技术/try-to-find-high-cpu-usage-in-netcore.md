---
title:  在.NET应用程序中分析CPU使用率过高的问题
date: 2020-04-9 21:24
tags: 技术
author: 邹溪源
categories:
  - 技术
---

作者:胡安·帕勃罗·希达，JUAN PABLO SCIDA是一位软件架构师，在软件开发方面拥有10多年的经验。他是经过认证的.NET和Java开发人员。在过去的几年中，他还热衷于使用Node.js，MongoDB和Erlang。

原文来自：[https://www.toptal.com/dot-net/hunting-high-cpu-usage-in-dot-net](https://www.toptal.com/dot-net/hunting-high-cpu-usage-in-dot-net)

软件开发可能是一个非常复杂的过程。作为开发人员，我们需要考虑很多不同的变量。有些不在我们的控制之下，有些在实际代码执行时对我们来说是未知的，有些则由我们直接控制。 [.NET开发人员](https://www.toptal.com/dot-net)也毫不例外。

考虑到这样的现实情况，当我们在受控环境中工作时，事情通常会按计划进行。假设就是我们的开发机器或我们可以完全访问的集成环境。我们可以使用工具来分析影响我们的代码和软件的不同变量。我们也不必处理服务器的繁重负载，也不必处理并发用户尝试同时执行相同操作的情况。

在可描述和安全的情况下，我们的代码通常可以正常工作，但是在生产环境下，如果处于过度负载或其他一些外部因素的影响，可能会发生意外问题。生产环境的软件性能很难分析。在大多数情况下，我们必须在理论上处理潜在的问题：我们知道可能会发生问题，但无法测试。这就是为什么我们需要以我们所用语言的最佳实践和文档为基础进行开发，并避免[常见错误](https://www.toptal.com/c-sharp/top-10-mistakes-that-c-sharp-programmers-make)。

如前所述，当软件上线时，可能会出错，并且代码可能会以我们未计划的方式开始执行。当我们不得不处理问题而又无法调试或确定发生了什么情况时，我们可能会遇到这种情况。在这种情况下我们该怎么办？

![图片](https://uploader.shimo.im/f/nLECb8q0ecw5UN4q.png!thumbnail)

如果某个进程长时间使用超过90％的CPU，则我们会遇到麻烦


在本文中，我们将分析基于Windows的服务器上. net web应用程序的高CPU使用率的实际案例场景、涉及到的识别问题的过程，以及更重要的问题，为什么会出现这个问题以及我们如何解决它。

CPU使用率和内存消耗是广泛讨论的主题。通常，很难确定某个特定进程应使用的资源（CPU，RAM，I / O）的正确数量以及持续的时间段。尽管可以肯定的是-如果某个进程长时间使用了超过90％的CPU，那么我们将特别麻烦，因为在这种情况下服务器将无法处理任何其他请求。

这是否意味着流程本身存在问题？不必要。该过程可能需要更多的处理能力，或者正在处理大量数据。首先，我们唯一能做的就是尝试确定发生这种情况的原因。

所有操作系统都有几种不同的工具来监视服务器中发生的事情。Windows服务器专门具有任务管理器[Performance Monitor](https://technet.microsoft.com/en-us/library/cc749115.aspx)，在本例中，我们使用了[New Relic Servers](http://newrelic.com/server-monitoring)，它是监视服务器的绝佳工具。

## 最初症状和问题分析
部署应用程序后，在头两周的时间里，我们开始看到服务器的CPU使用率达到峰值，这使服务器无响应。为了使其再次可用，我们必须重新启动它，并且该事件在该时间段内发生了3次。如前所述，我们使用New Relic Servers作为服务器监视器，它表明w3wp.exe在服务器崩溃时，该进程占用了94％的CPU。

Internet信息服务（IIS）工作进程是Windows进程（w3wp.exe），它运行Web应用程序，并负责处理发送到特定应用程序池的Web服务器的请求。IIS服务器可能有多个应用程序池（和几个不同的w3wp.exe进程），这些池可能会产生问题。根据该进程具有的用户（这在New Relic报告中显示），我们确定问题出在我们的.NET C＃Web表单旧版应用程序。

.NET Framework与Windows调试工具紧密集成在一起，因此，我们要做的第一件事是查看事件查看器和应用程序日志文件，以查找有关正在发生的事情的有用信息。无论我们是否在事件查看器中记录了一些异常，它们都没有提供足够的数据来进行分析。这就是为什么我们决定更进一步并收集更多数据的原因，因此当事件再次发生时，我们将做好准备。

## 数据采集
收集用户模式进程转储的最简单方法是使用[Debug Diagnostic Tools v2.0](https://www.microsoft.com/en-us/download/details.aspx?id=49924)或仅使用DebugDiag。DebugDiag具有一组用于收集数据（DebugDiag集合）和分析数据（DebugDiag分析）的工具。

因此，让我们开始定义使用调试诊断工具收集数据的规则：

1. 打开DebugDiag集合，然后选择Performance。![图片](https://uploader.shimo.im/f/pUEaTdjUQ34gG4NI.png!thumbnail)
2. 选择Performance Counters并单击Next。
3. 点击Add Perf Triggers。
4. 展开Processor（不是Process）对象，然后选择% Processor Time。请注意，如果您使用的是Windows Server 2008 R2，并且具有64个以上的处理器，请选择该Processor Information对象而不是该Processor对象。
5. 在实例列表中，选择_Total。
6. 单击Add，然后单击确定OK。
7. 选择新添加的触发器，然后单击确定Edit Thresholds。![图片](https://uploader.shimo.im/f/YLVYmJ9IXEsw2vyW.png!thumbnail)
8. Above在下拉菜单中选择。
9. 将阈值更改为80。
10. 输入20秒数。您可以根据需要调整该值，但请注意不要指定小数秒，以防止错误触发。![图片](https://uploader.shimo.im/f/o4CEqvo7SO4tN36e.png!thumbnail)
11. 点击OK。
12. 点击Next。
13. 点击Add Dump Target。
14. Web Application Pool从下拉菜单中选择。
15. 从应用程序池列表中选择您的应用程序池。
16. 点击OK。
17. 点击Next。
18. Next再点击一次。
19. 如果需要，请输入规则名称，并记下转储的保存位置。您可以根据需要更改此位置。
20. 点击Next。
21. 选择Activate the Rule Now并单击Finish。

描述的规则将创建一组小型转储文件，这些文件的大小将非常小。最终转储将是具有完整内存的转储，并且该转储会更大。现在，我们只需要等待高CPU事件再次发生即可。

将转储文件保存在所选文件夹中后，我们将使用DebugDiag Analysis工具来分析收集的数据：

1. 选择性能分析器。![图片](https://uploader.shimo.im/f/TMJu2hTrqlgsk72i.png!thumbnail)
2. 添加转储文件。![图片](https://uploader.shimo.im/f/FiLI9Fm8THwFbtHC.png!thumbnail)
3. 开始分析。

DebugDiag将花费几分钟（或数分钟）来解析转储并提供分析。完成分析后，您将看到一个网页，其中包含摘要以及有关线程的大量信息，类似于以下内容：

![图片](https://uploader.shimo.im/f/g0Ju109AJTgK2mGc.png!thumbnail)

正如您在摘要中看到的那样，有一条警告说：“在一个或多个线程上检测到转储文件之间的CPU使用率过高。” 如果单击建议，我们将开始了解应用程序存在问题的地方。我们的示例报告如下所示：

![图片](https://uploader.shimo.im/f/8yUwhsUG7wg9LalW.png!thumbnail)

正如我们在报告中看到的那样，有一个关于CPU使用率的模式。所有CPU使用率高的线程都与同一类相关。在跳到代码之前，让我们看一下第一个。

![图片](https://uploader.shimo.im/f/UejsVGxAkYEiBRaH.png!thumbnail)

这是我们遇到的第一个线程的细节。对我们来说有趣的部分是：

![图片](https://uploader.shimo.im/f/oBHnRyDQWvU0RxZP.png!thumbnail)

在这里，我们有一个代码调用，GameHub.OnDisconnected()该代码触发了有问题的操作，但是在此调用之前，我们有两个Dictionary调用，它们可以使您对发生的事情有所了解。让我们看一下.NET代码，看看该方法在做什么：

public override Task OnDisconnected() {

    	try

    	{

        	var userId = GetUserId();

        	string connId;

        	if (onlineSessions.TryGetValue(userId, out connId))

            	onlineSessions.Remove(userId);

    	}

    	catch (Exception)

    	{

        	// ignored

    	}

    	return base.OnDisconnected();

    	}

我们显然在这里有问题。报告的调用堆栈说问题出在字典上，在这段代码中我们正在访问字典，特别是引起问题的那一行是：

if (onlineSessions.TryGetValue(userId, out connId))

这是字典声明：

static Dictionary<int, string> onlineSessions = new Dictionary<int, string>();

## .NET代码有什么问题？
具有面向对象编程经验的每个人都知道静态变量将由此类的所有实例共享。让我们更深入地了解.NET世界中静态的含义。

根据.NET C＃规范：

>使用[static](https://msdn.microsoft.com/en-us/library/98f28cdx.aspx)修饰符声明一个静态成员，该成员属于类型本身而不是特定对象。

这就是.NET C＃语言规范关于[静态类和成员的说明](https://msdn.microsoft.com/en-us/library/79b3xss3.aspx)：

>与所有类类型一样，当加载引用该类的程序时，.NET Framework公共语言运行库（CLR）将加载静态类的类型信息。程序无法确切指定何时加载类。但是，可以保证在程序中首次引用该类之前，将其加载并初始化其字段并调用其静态构造函数。静态构造函数仅被调用一次，并且静态类在程序所在的应用程序域的生存期内保留在内存中。
>非静态类可以包含静态方法，字段，属性或事件。即使没有创建该类的实例，该静态成员也可以在该类上调用。始终通过类名称而不是实例名称访问静态成员。无论创建多少个类实例，静态成员只有一个副本。静态方法和属性无法访问其包含类型的非静态字段和事件，并且除非在方法参数中显式传递了实例变量，否则它们无法访问任何对象的实例变量。

这意味着静态成员属于类型本身，而不是对象。它们也由CLR加载到应用程序域中，因此静态成员属于承载应用程序的进程，而不是特定线程。

鉴于Web环境是多线程环境，因为每个请求都是由w3wp.exe进程产生的新线程；考虑到静态成员是该过程的一部分，我们可能会遇到以下情况：几个不同的线程尝试访问静态（由多个线程共享的）变量的数据，这最终可能会导致多线程问题。

线程安全性下的Dictionary [文档](https://msdn.microsoft.com/en-us/library/xfhwa508%28v=vs.100%29.aspx)声明以下内容：

>Dictionary<TKey, TValue>只要不修改集合，A 就可以同时支持多个阅读器。即使这样，通过集合进行枚举本质上也不是线程安全的过程。在极少的枚举与写访问竞争的情况下，必须在整个枚举期间锁定集合。要允许多个线程访问该集合进行读写，您必须实现自己的同步。

此声明解释了为什么我们可能会遇到此问题。根据转储信息，问题出在字典的FindEntry方法上：

![图片](https://uploader.shimo.im/f/RrbDyIKOrqQQ2qt0.png!thumbnail)

如果查看字典的FindEntry [实现，](http://referencesource.microsoft.com/#mscorlib/system/collections/generic/dictionary.cs,bcd13bb775d408f1)我们可以看到该方法遍历内部结构（存储桶）以查找值。

因此，以下.NET代码枚举了集合，这不是线程安全的操作。

```
public override Task OnDisconnected() {
    	try
    	{
        	var userId = GetUserId();
        	string connId;
        	if (onlineSessions.TryGetValue(userId, out connId))
            	onlineSessions.Remove(userId);
    	}
    	catch (Exception)
    	{
        	// ignored
    	}
    	return base.OnDisconnected();
	}
```
## 结论
正如我们在转储中看到的那样，有多个线程试图同时迭代和修改共享资源（静态字典），最终导致迭代进入无限循环，从而导致线程消耗超过90％的CPU。 。

有几种可能的解决方案。我们首先实现的方法是锁定和同步对字典的访问，但会损失性能。那时服务器每天都崩溃，因此我们需要尽快解决此问题。即使这不是最佳解决方案，它也解决了该问题。

解决这个问题的下一步是分析代码并找到最优解决方案。重构代码是一个选项:新的ConcurrentDictionary类可以解决这个问题，因为它只锁定在一个桶级别，这将提高整体性能。尽管这是一大步，还需要进一步的分析。

