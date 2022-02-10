---
title:   浅议C#客户端和服务端通信的几种方法：Rest和GRPC
date: 2020-12-20 21:05
tags: 技术
author: 邹溪源
categories:
  - 技术
---

# 浅议C#客户端和服务端通信的几种方法：Rest和GRPC

 在C＃客户端和C＃服务器之间进行通信的方法有很多。一些功能强大，而其他功能则不是很多。有些非常快，有些则不是。知道不同的选择很重要，这样您才能决定最适合自己的选择。

本文将介绍当今最流行的技术，以及为何如此广泛地使用它们。我们将讨论REST，gRPC及其两者之间的所有内容。 

## 最佳方案

让我们考虑一下我们希望如何在最佳环境中使客户端与服务器之间的通信看起来像。我在想像这样的东西：

```
// on client side
public void Foo()
{
    var server = new MyServer(new Uri("https://www.myserver.com/");)
    int sum = server.Calculator.SumNumbers(12,13); 
}
```



```

// on server side
class CalculatorController : Controller{
    public int SumNumbers(int a, int b)
    {
        return a + b;
    }
}
```

我当然想要完整的Intellisense。当我单击`server`并`.` 希望Visual Studio显示所有控制器时。当我单击`CalculatorController`和时`.`，我想查看所有操作。我还想要一流的性能，很少的网络负载和双向通信。而且我想要一个能够完美处理版本控制的强大系统，这样我就可以毫不费力地部署新的客户端版本和新的服务器版本。

要求太多吗？

请注意，我在这里谈论的是**无状态**API。这等效于C＃项目，其中只有两种类型的类：

- 静态类，只有静态方法。
- [POCO类](https://www.c-sharpcorner.com/UploadFile/5d065a/poco-classes-in-entity-framework/)仅具有类型为基本类型或其他POCO类的字段和属性。

在API中使用状态会带来复杂性，而这正是万恶之源。因此，为了本文的方便，让我们保持美好和无状态。

## 传统REST

REST API出现在2000年代初期，席卷了整个互联网。到目前为止，它是创建Web服务的最流行的方法。

REST为客户端到服务器的请求定义了一组固定的操作**GET，POST，PUT**和**DELETE**。每个请求都将通过包含有效负载（通常为JSON）的响应来回答。请求包含在查询本身中的参数，或者在它是POST请求时包含为有效负载（通常为JSON）的参数。

有一个称为RESTful API的标准，它定义了以下规则（您实际上不必使用它）：

- GET用于检索资源
- PUT用于更改资源状态
- POST用于创建资源
- DELETE用于删除资源

如果您到目前为止还不熟悉REST，则上面的解释可能不会减少它，因此这里有一个示例。在.NET中，内置了对REST的支持。实际上，默认情况下，ASP.NET Web API被构建为REST Web服务。这是典型的客户端和ASP.NET服务器的外观：

在服务器中：

```
[Route("People")]
public class PeopleController : Controller
{
    [HttpGet]
    public Person GetPersonById(int id)
    {
        Person person = _db.GetPerson(id);
        return person;//Automatically serialized to JSON
    }
}   
```

 在客户中： 

```
var client = new HttpClient();
string resultJson = 
    await client.GetStringAsync("https://www.myserver.com/People/GetPersonById?id=123");
Person person = JsonConvert.DeserializeObject<Person>(resultJson);
 
```

REST非常方便，但是并没有达到最佳方案。因此，让我们看看是否可以做得更好。

## ReFit

ReFit不能替代REST。相反，它建立在REST之上，并允许我们像调用简单方法一样调用服务器端点。这是通过在客户端和服务器之间共享接口来实现的。在服务器端，您的控制器将实现一个接口：

```
public interface IMyEmployeeApi
{
    [Get("/employee/{id}")]
    Task<Employee> GetEmployee(string id);
}
```

然后，在客户端，您需要包括相同的接口并使用以下代码：

```
var api = RestService.For<IMyEmployeeApi>("https://www.myserver.com");
var employee = await api.GetEmployee("abc");
 
```

就这么简单。除了几个NuGet软件包外，无需运行困难的自动化程序或使用任何第三方工具。

这更接近最佳方案。现在，我们有了IntelliSense，并且客户端和服务器之间的合同很牢固。但是还有另一种选择，在某些方面甚至更好。

## 昂首阔步

像ReFit一样，Swagger也建立在REST之上。[OpenAPI](https://swagger.io/specification/)或**Swagger**是REST API的规范。它描述了具有简单JSON文件的REST Web服务。这些文件是Web服务的API架构。它们包括：

- API中的所有路径（URL）。
- 每个路径的预期操作（GET，POST等）。每个路径可以处理不同的操作。例如，单个路径`https://mystore.com/Product`可能接受添加产品的POST操作和返回产品的GET操作。
- 每个路径和操作的预期参数。
- 每个路径的预期响应。
- 每个参数和响应对象的类型。

该JSON文件实质上是客户端和服务器之间的合同。这是一个描述一个称为[Swagger Petstore](https://bfanger.nl/swagger-explained/#operationObject)的Web服务的swagger文件的示例（为清楚起见，我删除了一些部分）：

```
{ 
   "swagger":"2.0",
   "info":{ 
      "version":"1.0.0",
      "title":"Swagger Petstore",
      "description":"A sample API that uses a petstore as an example to demonstrate features in the swagger-2.0 specification",
   },
   "host":"petstore.swagger.io",
   "basePath":"/api",
   "schemes":[ 
      "http"
   ],
   "consumes":[ 
      "application/json"
   ],
   "produces":[ 
      "application/json"
   ],
   "paths":{ 
      "/pets":{ 
         "get":{ 
            "description":"Returns all pets from the system that the user has access to",
            "operationId":"findPets",
            "produces":[ 
               "application/json",
               "application/xml",
            ],
            "parameters":[ 
               { 
                  "name":"tags",
                  "in":"query",
                  "description":"tags to filter by",
                  "required":false,
                  "type":"array",
                  "items":{ 
                     "type":"string"
                  },
                  "collectionFormat":"csv"
               },
               { 
                  "name":"limit",
                  "in":"query",
                  "description":"maximum number of results to return",
                  "required":false,
                  "type":"integer",
                  "format":"int32"
               }
            ],
            "responses":{ 
               "200":{ 
                  "description":"pet response",
                  "schema":{ 
                     "type":"array",
                     "items":{ 
                        "$ref":"#/definitions/Pet"
                     }
                  }
               },
...
```



让我们考虑一下这个结果。使用上面的JSON文件，您可以潜在地创建具有完整IntelliSense的C＃客户端。毕竟，您知道所有路径，操作，它们期望的参数，什么参数类型，什么是响应等等。

有几种工具可以做到这一点。对于服务器端，可以使用[Swashbuckle.AspNetCore](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)将Swagger添加到ASP.NET中并生成所述JSON文件。对于客户端，您可以使用[swagger-codegen](https://github.com/swagger-api/swagger-codegen)和[AutoRest](https://azure.github.io/autorest/)来使用这些JSON文件并生成客户端。让我们看一个如何做到这一点的例子：

### 将Swagger添加到ASP.NET服务器

首先添加NuGet包[Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore)。在中`ConfigureServices`，注册Swagger生成器：

```
services.AddSwaggerGen(options => 
	options.SwaggerDoc("v1", new OpenApiInfo {Title = "My Web API", Version = "v1"}));
```

在添加`Configure`方法中`Startup.cs`：

```
app.UseSwagger();
```

最后，控制器内部的动作应使用`[HttpXXX]`和`[FromXXX]`属性修饰：

```
[HttpPost]
public async Task AddEmployee([FromBody]Employee employee)
{
    //...
}
 
[HttpGet]
public async Task<Employee> Employee([FromQuery]string id)
{
    //...
}
```

就像服务器端一样简单。运行项目时，`swagger.json`将生成一个文件，可用于生成客户端。

### 使用AutoRest从Swagger生成客户端

要开始使用[AutoRest](https://github.com/Azure/autorest)，与安装[NPM](https://www.w3schools.com/nodejs/nodejs_npm.asp)：`npm install -g autorest`。安装后，您将需要使用AutoRest的命令行界面从该`swagger.json`文件生成C＃客户端。这是一个例子：

```
autorest --input-file="./swagger.json" --output-folder="GeneratedClient" --namespace="MyClient" --override-client-name="MyClient" --csharp 
```

这将产生一个`GeneratedClient`包含生成的C＃文件的文件夹。请注意，名称空间和客户端名称被覆盖。从这里，将此文件夹添加到Visual Studio中的客户端项目。

![Visual Studio中的AutoRest](https://michaelscodingspot.com/wp-content/uploads/2020/02/addAutoRestToVS.png)

您需要安装`Microsoft.Rest.ClientRuntime`NuGet软件包，因为生成的代码取决于该软件包。安装后，您可以像使用常规C＃类一样使用API：

```
var client = new MyClient();
Employee employee = client.Employee(id: "abc");
```

您可以在AutoRest的[文档中](https://azure.github.io/autorest/client/ops.html)阅读一些细微之处。而且您需要使该过程自动化，因此我建议阅读Patrik Svensson的[教程，](https://www.patriksvensson.se/2018/10/generating-api-clients-using-autorest)以获得一些好的建议以及Peter Jausovec的这篇[文章](https://medium.com/@pjausovec/creating-c-client-library-for-web-api-projects-be132c831f9c)。

Swagger的问题是JSON文件是在运行时创建的，因此这使得在CI / CD流程中实现自动化有点困难。

## 传统REST vs Swagger vs ReFit

进行选择时，请注意以下几点。

- 如果您有一个非常简单的私有REST API，则也许不必理会客户端生成和共享接口。小任务并不能证明付出额外的努力是合理的。
- Swagger支持多种语言，而ReFit仅支持.NET。Swagger还是许多工具，测试，自动化和UI工具的基础。如果您要创建一个大型的公共API，它将可能是最佳选择。
- Swagger比ReFit复杂得多。使用ReFit，只需在服务器和客户端项目中添加一个接口即可。另一方面，使用ReFit，您必须为每个控制器创建新的接口，而Swagger会自动进行处理。

但是在决定任何事情之前，请检查与REST无关的第四个选项。

## gRPC

[gRPC](https://grpc.io/)（**gRPC远程过程调用**）是Google开发的开源远程过程调用系统。它有点像REST，它提供了一种将请求从客户端发送到服务器的方式。但这在许多方面都不同，这是相同点和不同点：

- 像REST一样，gRPC与语言无关。有适用于所有流行语言的工具，包括C＃。
- gRPC是契约的基础，并使用`.proto`文件来定义契约。这有点类似于Swagger`swagger.json`和ReFit的共享界面。可以从那些文件中生成任何编程语言的客户端。
- gRPC使用[协议缓冲区（Protobuf）](https://en.wikipedia.org/wiki/Protocol_Buffers)二进制序列化。这与REST（通常序列化为JSON或XML）不同。二进制序列化较小，因此更快。
- gRPC用于使用HTTP / 2协议创建持久连接。该协议更简单，更紧凑。REST使用HTTP 1.x协议（通常为HTTP 1.1）。
- HTTP 1.1要求每个请求都进行TCP握手，而HTTP / 2则保持连接打开。
- HTTP / 2连接使用多路复用流。这意味着单个TCP连接可以支持许多流。这些流可以并行执行，而不必像HTTP 1.1中那样互相等待。
- gRPC允许双向流。

有两种使用gRPC的方法。对于.NET Core 3.0，有一个完全托管的库，称为[.NET的gRPC](https://github.com/grpc/grpc-dotnet)。对于其中的任何内容，您都可以使用[gRPC C＃](https://github.com/grpc/grpc/tree/master/src/csharp)，它是使用本机代码构建的。这并不意味着**适用于.NET的gRPC可以**替代**gRPC C＃**。让我们来看一个**用于.NET**的更新**gRPC**的示例。

### .NET的gRPC的服务器端

这不是教程，而是更多有关预期内容的一般性想法。这是示例控制器在gRPC中的外观：

```
public class GreeterService : Greeter.GreeterBase
{
    public override Task<HelloReply> SayHello(HelloRequest request,
        ServerCallContext context)
    {
        _logger.LogInformation("Saying hello to {Name}", request.Name);
        return Task.FromResult(new HelloReply 
        {
            Message = "Hello " + request.Name
        });
    }
}
```

您需要添加以下的`Configure`在`Startup.cs`：

```
app.UseEndpoints(endpoints =>
{
    endpoints.MapGrpcService<GreeterService>();
});
 
```

API在`.proto`文件中描述，该文件是项目的一部分：

```

syntax = "proto3";
 
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}
 
message HelloRequest {
  string name = 1;
}
 
message HelloReply {
  string message = 1;
}
```

 此`.proto`文件添加到`.csproj`： 

```
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" />
</ItemGroup>
```

.NET的gRPC客户端

客户端是从`.proto`文件生成的。代码本身非常简单：

```
var channel = GrpcChannel.ForAddress("https://localhost:5001");
var client = new Greeter.GreeterClient(channel);
 
var response = await client.SayHello(
    new HelloRequest { Name = "World" });
 
Console.WriteLine(response.Message);
 
```



## gRPC与REST

gRPC听起来不错。它在框架下更快，更简单。那么，我们都应该从REST变为gRPC吗？答案是，这取决于你的应用场景。

以下是一些注意事项：

从我的印象来看，使用gRPC和ASP.NET仍然不是很好。借助对REST的成熟支持，您会变得更好。就基于契约的通信而言，这很不错，除了在REST中有我们已经讨论过的类似替代方案：Swagger和ReFit。

最大的优势是性能。[根据这些基准](https://www.yonego.com/nl/why-milliseconds-matter/#gref)，在大多数情况下，gRPC更快。特别是对于大型有效载荷，Protobuf序列化确实有所作为。这意味着对于高负载服务器而言，这是一个巨大的优势。

在大型ASP.NET应用程序中从REST过渡到gRPC将非常困难。但是，如果您具有基于微服务的体系结构，那么逐步完成此过渡就变得容易得多。

## 其他沟通方式

还有其他一些我完全没有提及的通信方式，但是值得一提的是：

- [GraphQL](https://graphql.org/)是Facebook开发的API的查询语言。它允许客户端从服务器确切地要求它需要的数据。这样，您可以在服务器上仅创建一个端点，该端点将非常灵活，并且仅返回客户端所需的数据。近年来，GraphQL变得非常流行。
- [SignalR](https://github.com/SignalR/SignalR)是一项允许服务器与客户端之间进行实时双向通信的技术。SignalR不仅允许客户端始终向服务器发送请求，还允许服务器向客户端发送推送通知。这样可以查看Web应用程序中的实时更新。SignalR在ASP.NET中非常流行。
- [TcpClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.tcpclient?view=netframework-4.8)和[TcpListener](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.tcplistener?view=netframework-4.8)（在中`System.Net.Sockets`）提供基于TCP的低级连接。基本上，您将建立连接并传输字节数组。对于大型应用程序而言，它不是理想的选择，在大型应用程序中，您可以使用ASP.NET的控制器和操作在大型API中进行订购。
- [UdpClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.udpclient?view=netframework-4.8)提供了一种通过UDP协议进行通信的方法。TCP建立连接，然后发送数据，而UDP仅发送数据。TCP确保数据中没有错误，而UDP没有。UDP可以更有效地快速传输数据，您不必担心它是否可靠且没有错误。一些示例是：视频流，实时广播和IP语音（VoIP）。
- [WCF](https://docs.microsoft.com/en-us/dotnet/framework/wcf/whats-wcf)是一种较旧的技术，主要在进程之间使用基于SOAP的通信。这是一个庞大的框架，我要说的是它已不再受REST和JSON负载的欢迎。


