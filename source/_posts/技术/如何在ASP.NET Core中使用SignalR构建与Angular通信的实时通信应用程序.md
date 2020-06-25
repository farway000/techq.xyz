---
title:  如何在ASP.NET Core中使用SignalR构建与Angular通信的实时通信应用程序
date: 2020-6-25 20:05
tags: 技术
author: 邹溪源
categories:
  - 技术
---
![图片](https://uploader.shimo.im/f/ByiYlxtK4DIP8lsr.png!thumbnail)

假设我们要创建一个监视Web应用程序，该应用程序为用户提供了一个能够显示一系列信息的仪表板，这些信息会随着时间的推移而更新。

第一种方法是在定义的时间间隔（*轮询*）定期调用API 以更新仪表板上的数据。

无论如何，还是有一个问题：如果没有更新的数据，我们会因请求而不必要地增加网络流量。一种替代方法是*长轮询*技术：如果服务器没有可用数据，则它可以使请求保持活动状态，直到发生某种情况或达到预设的超时时间为止，而不是发送空响应。如果存在新数据，则完整的响应将到达客户端。完全不同的方法是反转角色：当有新数据可用（推送）时，后端与客户端联系。

请记住，HTML 5具有标准化的WebSocket，这是一个永久的双向连接，可以在兼容的浏览器中使用Javascript接口进行配置。不幸的是，必须在客户端和服务器端都对WebSocket提供完全支持，以使其可用。然后，我们需要提供替代系统（*fallback*），无论如何，该替代系统都允许我们的应用程序运行。

微软于2013年发布了一个名为**SignalR** for **ASP.NET**的开源库，该库**已于** 2018年为ASP.NET Core进行了重写。SignalR从与通信机制有关的所有细节中进行抽象，并从可用的信息中选择最佳的一种。结果是有可能编写代码，就像我们一直处于*push-mode一样*。使用SignalR，服务器可以在其所有连接的客户端或特定客户端上调用JavaScript方法。

我们使用web-api模板创建一个ASP.NET Core项目，删除已生成的示例控制器。使用NuGet，我们将*Microsoft.AspNet.SignalR*添加到项目中，以创建**Hub**。集线器是能够调用客户端代码，发送包含所请求方法的名称和参数的消息的高级管道。作为参数发送的对象将使用适当的协议反序列化。客户端在页面代码中搜索与名称相对应的方法，如果找到该名称，则将其调用并传递反序列化的数据作为参数。

```plain
using Microsoft.AspNetCore.SignalR;
 
namespace SignalR.Hubs
{
    public class NotificationHub : Hub { }
}
```
您可能知道，在ASP.NET Core中，可以配置HTTP请求的管理管道，以添加一些**中间件**，该**中间件**可拦截请求，添加已配置的功能并使其进入下一个中间件。必须预先配置SignalR中间件，在**Startup**  类的*ConfigureServices*方法中添加扩展方法*services.AddSignalR（）*。现在，我们可以使用Startup类的*Configure*方法中的扩展方法app.UseSignalR（）将中间件添加到管道中。在此操作期间，我们可以传递配置参数，包括集线器的路由：
```plain
app.UseSignalR(route =>
{
    route.MapHub<notificationhub>("/notificationHub");
})
```
一个有趣的场景允许我们查看ASP.NET Core中的另一个有趣功能，即在**后台工作**进程上下文中*托管* SignalR Hub 。
假设我们要实现以下用例：

* 运行业务逻辑
* 等一下
* 决定是停止还是重复该过程。

在ASP.NET Core中，我们可以使用框架提供的*IHostedService*接口在.NET Core应用程序中在后台实现进程的执行。方法要实现是*StartAsync（）*和*StopAsync（） *。非常简单：StartAsync调用到主机启动，而StopAsync调用到主机关闭。

然后，我们将一个类*DashboardHostedService*添加到项目中，该类实现*IHostedService*。我们在Startup类的*ConfigureServices*方法中添加接口注册：

```plain
services.AddHostedService<dashboardhostedservice>();
```
在类构造函数*DashboardHostedService中，*我们注入*IHubContext* 访问添加到我们应用程序的集线器。
在方法StartAsync中，我们设置了一个计时器，它将每两秒钟运行一次方法DoWork（）中包含的代码。此方法发送带有四个随意生成的字符串的消息。

但是它向谁传播呢？在我们的示例中，我们正在将消息发送到所有连接的客户端。但是，SignalR提供了向单个用户或用户组发送消息的机会。在[本文中](https://docs.microsoft.com/en-us/aspnet/core/signalr/groups?view=aspnetcore-2.2)，您将找到涉及ASP.NET Core中的身份验证和授权功能的详细信息。有趣的是，用户可以同时在台式机和移动设备上连接。每个设备都有一个单独的SignalR连接，但是它们都将与同一用户关联。

```plain
using Microsoft.AspNetCore.SignalR;
using Microsoft.Extensions.Hosting;
using SignalR.Hubs;
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
 
namespace SignalR
{
    public class DashboardHostedService: IHostedService
    {
        private Timer _timer;
        private readonly IHubContext<notificationhub> _hubContext;
 
        public DashboardHostedService(IHubContext<notificationhub> hubContext)
        {
            _hubContext = hubContext;
        }
 
        public Task StartAsync(CancellationToken cancellationToken)
        {
            _timer = new Timer(DoWork, null, TimeSpan.Zero,
            TimeSpan.FromSeconds(2));
 
            return Task.CompletedTask;
        }
 
        private void DoWork(object state)
        {
            _hubContext.Clients.All.SendAsync("SendMessage", 
                new {
                    val1 = getRandomString(),
                    val2 = getRandomString(),
                    val3 = getRandomString(),
                    val4 = getRandomString()
                });
        }
 
        public Task StopAsync(CancellationToken cancellationToken)
        {
            _timer?.Change(Timeout.Infinite, 0);
 
            return Task.CompletedTask;
        }
    }
}
```
让我们看看如何管理客户端部分。例如，我们使用Angular CLI的*ng new SignalR*命令创建Angular应用程序。然后我们安装SignalR的包节点（*npm i @ aspnet / signalr*）。然后添加一个服务，该服务使我们可以连接到先前创建的集线器并接收消息。
在这里，第一种可能的方法是，基于服务getMessage（）中Observable <Message>的服务，通过使用私有声明的Subject <Message>来返回（Message是与从Object返回的对象相对应的Typescript接口。后端）：

```plain
@Injectable({
 providedIn: 'root'
})
export class SignalRService {
 private message$: Subject<message>;
 private connection: signalR.HubConnection;
 
 constructor() {
   this.message$ = new Subject<message>();
   this.connection = new signalR.HubConnectionBuilder()
   .withUrl(environment.hubUrl)
   .build();
   this.connect();
 }
 private connect() {
   this.connection.start().catch(err => console.log(err));
   this.connection.on('SendMessage', (message) => {
     this.message$.next(message);
   });
 }
 public getMessage(): Observable<message> {
   return this.message$.asObservable();
 }
 public disconnect() {
   this.connection.stop();
 }
}
```
在constructor（）内部，我们创建一个SignalR.HubConnection类型对象，该对象将用于连接到服务器。我们通过使用文件environment.ts将其传递到其中心URL：
```plain
this.connection = new signalR.HubConnectionBuilder()
   .withUrl(environment.hubUrl)
   .build();
```
构造函数还负责调用connect（）方法，该方法进行实际连接，并在控制台中记录可能的错误。
```plain
this.connection.start().catch(err => console.log(err));
this.connection.on('SendMessage', (message) => {
  this.message$.next(message);
});
```
想要显示来自后端的消息的组件（将其注入到构造函数中的服务），应该订阅getMessage（）方法并管理到达的消息。以AppComponent为例，例如：
```plain
@Component({
 selector: 'app-root',
 templateUrl: './app.component.html',
 styleUrls: ['./app.component.css']
})
export class AppComponent implements OnDestroy {
 private signalRSubscription: Subscription;
 
 public content: Message;
 
 constructor(private signalrService: SignalRService) {
   this.signalRSubscription = this.signalrService.getMessage().subscribe(
     (message) => {
       this.content = message;
   });
 }
 ngOnDestroy(): void {
   this.signalrService.disconnect();
   this.signalRSubscription.unsubscribe();
 }
}
```
使用主题<Message>允许我们同时管理更多组件，而无论从中心返回的消息（用于订阅还是用于取消订阅）都可以，但是我们必须注意对主题的粗心使用。让我们考虑以下getMessage（）版本：
```plain
public getMessage(): Observable<message> {
   return this.message$;
}
```
现在，该组件也可以使用以下简单代码发送一条消息：
```plain
const produceMessage = this.signalrService.getMessage() as Subject<any>;
 produceMessage.next( {val1: 'a'});
</any>
```
如果方法getMessage（）返回Subject <Message> asObservable，则此代码将引发异常！
我们可以在单个组件的情况下使用的第二种方法（更简单）对管理来自后端的消息感兴趣：

```plain
@Injectable({
 providedIn: 'root'
})
export class SignalrService {
 connection: signalR.HubConnection;
 
 constructor() {
   this.connection = new signalR.HubConnectionBuilder()
   .withUrl(environment.hubAddress)
   .build();
   this.connect();
 }
 
 public connect() {
   if (this.connection.state === signalR.HubConnectionState.Disconnected) {
     this.connection.start().catch(err => console.log(err));
   }
 }
 
 public getMessage(next) {
     this.connection.on('SendMessage', (message) => {
       next(message);
     });
 }
 
 public disconnect() {
   this.connection.stop();
 }
}
```
我们可以简单地将函数回调传递给方法getMessage，该函数将来自后端的消息作为参数。在这种情况下，AppComponent可以成为：
```plain
public content: IMessage;
constructor(private signalrService: SignalrService) {
   this.signalrService.getMessage(
     (message: IMessage) => {
       this.content = message;
     }
   );
}
ngOnDestroy(): void {
   this.signalrService.disconnect();
}
```
最后几行代码分别位于*app.component.html*和*app.component.css中*，以赋予一些时尚，并且该应用程序已完成。
```plain
<div style="text-align:center">
  <h1>
    DASHBOARD
  </h1>
</div>
<div class="card-container">
  <div class="card">
    <div class="container">
      <h4><b>Valore 1</b></h4>
      <p>{{content.val1}}</p>
    </div>
  </div>
  <div class="card">
    <div class="container">
      <h4><b>Valore 2</b></h4>
      <p>{{content.val2}}</p>
    </div>
  </div>
  <div class="card">
    <div class="container">
      <h4><b>Valore 3</b></h4>
      <p>{{content.val3}}</p>
    </div>
  </div>
  <div class="card">
    <div class="container">
      <h4><b>Valore 4</b></h4>
      <p>{{content.val4}}</p>
    </div>
  </div>
</div>
 
.card-container {
  display: flex;
  flex-wrap: wrap;
}
 
.card {
  box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2);
  transition: 0.3s;
  width: 40%;
  flex-grow: 1;
  margin: 10px;
}
 
.card:hover {
  box-shadow: 0 8px 16px 0 rgba(0,0,0,0.2);
}
 
.container {
  padding: 2px 16px;
}
```
我们首先启动后端，然后启动前端并检查最终结果：
![图片](https://uploader.shimo.im/f/TJ3iFEnLIx8DuW8y.gif)

看起来不错！您可以在这里找到代码：[https](https://github.com/AARNOLD87/SignalRWithAngular) : [//github.com/AARNOLD87/SignalRWithAngular](https://github.com/AARNOLD87/SignalRWithAngular)

下次见！

