---
title:  .NET Core使用gRPC打造服务间通信基础设施 date: 2019-10-17 20:28
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# 一、什么是RPC
rpc（远程过程调用）是一个古老而新颖的名词，他几乎与http协议同时或更早诞生，也是互联网数据传输过程中非常重要的传输机制。

利用这种传输机制，不同进程（或服务）间像调用本地进程中的方法一般进行交互，而无需关心实现细节。

rpc的主要实现流程为：

![图片](https://uploader.shimo.im/f/wMP63PQYjIoPM9k8.png!thumbnail)

1、客户端本地方法调用客户端stub（方法存根）。这个调用发生在客户端本地，并把调用参数推送到栈中。

2、客户端stub (方法存根）将这些参数打包，通过系统调用发送到服务器机器。打包的过程通常可以采用xml、json、二进制编码。打包的过程被称为marshalling。

3、客户端本地操作系统发送信息到目标服务器（可以通过自定义tcp协议或Http协议传输）。

4、服务器系统将信息传送到服务端stub（方法存根）

5、服务端stub (服务端方法存根） 解析信息。解析信息的过程可以称为 unmarshalling。

6、服务器stub (服务端方法存根） 调用程序，并通过类似的方式返回客户端。

为了让不同的客户端均能访问服务器，许多标准化的rpc组件往往会使用接口描述语言的形式，以便方便跨平台、跨语言的远程过程调用的实现。

图1：RPC 调用流程

参考维基百科：[https://zh.wikipedia.org/wiki/%E9%81%A0%E7%A8%8B%E9%81%8E%E7%A8%8B%E8%AA%BF%E7%94%A8](https://zh.wikipedia.org/wiki/%E9%81%A0%E7%A8%8B%E9%81%8E%E7%A8%8B%E8%AA%BF%E7%94%A8)

# 二、什么时候使用RPC？
HTTP和RPC是现代微服务架构中普遍采用的两种数据传输格式，在某种场合几乎都是可以完全替换的，但又具有各自不同的特点。

1、HTTP协议是一种规范、开放、通用性非常强、标准的传输协议，几乎所有的语言都支持，如果要确保各类平台都能无缝的访问数据，可以考虑使用HTTP协议。例如目前常用的RestFul规约，定义好请求方法、数据格式并以Json的形式返回参数，能够让前后端之间的对接非常便捷；之前的开发者或许用wsdl、soap的形式比较多，也都是HTTP协议的一种应用。 

2、RPC协议不仅仅是一种服务间传输的协议，也能使用于进程间的数据传输，它能极大的降低微服务间的通信成本，屏蔽通信细节，让调用者能够像调用本地方法一般调用远程方法。 相对而言，RPC可能无法在网页端提供支持，也并非所有的语言都实现了这种接口描述语言，让开发过程会相对繁琐，因此它的使用范围相对较小。虽然gRPC目前已经提供了web版的gRPC，但由于浏览器的兼容性等问题，也限制了他的应用。

# 三、什么是gRPC
gRPC可以通俗的理解为google对于RPC的一种实现形式。

参见gRPC官网的解释：

>gRPC是可以在任何环境中运行的现代开源高性能RPC框架。它可以通过可插拔的支持来有效地连接数据中心内和跨数据中心的服务，以实现负载平衡，跟踪，运行状况检查和身份验证。它也适用于分布式计算的最后一英里，以将设备，移动应用程序和浏览器连接到后端服务。

它包括四个主要特点：

1. 简单的服务定义：gRPC基于Protobuf协议构建，该协议提供了一个强大的二进制序列化工具集和语言定义服务。
2. 跨语言和平台工作：可自动为多语言或平台生成符合相应习惯的客户端和服务端存根
3. 快速启动并扩展：只需一行代码即可安装运行时环境和生成环境、并通过该框架可扩展到数百万rpc请求。
4. 双向流和集成身份验证：基于http/2的传输机制以及双向流传输和完全集成的可插入式身份验证机制。

gRPC目前广泛应用于各大互联网公司的微服务架构中，也是CNCF基金会孵化的开源基础设施组件。其官网为[https://grpc.io/](https://grpc.io/)；开源项目地址为[https://github.com/grpc/grpc](https://github.com/grpc/grpc)。

官网提供了详细的文档说明，几乎可以开箱即用，只需简单配置就能满足你的应用需求。在开源项目中也提供了完善的各种语言实现的sample示例代码，能极大的方便开发者的使用。

在gRPC中，使用的传输协议为HTTP/2，使用的数据传输的格式为Protobuf协议。

# 四、什么是Protobuf
Protobuf全称为Protocal Buffers，是一种序列化协议实现，与只类似的还有thrift。这是一种与语言中立、与实现无关、可扩展的序列化数据格式，不仅仅可以用于通信协议传输过程，也同样适用于数据存储过程。它灵活高效、性能优良、更加快速和简单。在使用Protobuf的实践中，只需定义要处理数据的数据结构，就能利用Protobuf生成相关的代码。只需使用Protobuf对数据结构进行描述（IDL)，即可在各种不同的语言或不同的数据流中对结构化数据进行轻松读写。

在上面的图1 RPC调用流程中，使用红色字体标注的（1）中，在客户端套接字和服务端套接字之间进行数据交换的数据传输机制就可以使用Protobuf。

Protocol Buffers最早是有谷歌发明用于解决索引服务器之间request/response协议的。通过慢慢发展发展和演进，目前已经具有了更多的特性：

* 自动生成的序列化和反序列化代码避免了手动解析的需要。（官方提供自动生成代码工具，各个语言平台的基本都有）
* 除了用于 RPC（远程过程调用）请求之外，人们开始将 protocol buffers 用作持久存储数据的便捷自描述格式（例如，在Bigtable中）。
* 服务器的 RPC 接口可以先声明为协议的一部分，然后用 protocol compiler 生成基类，用户可以使用服务器接口的实际实现来覆盖它们。

由于protocal buffers诞生之初主要是为了解决服务器新旧协议之间兼容性问题，所以命名为"协议缓冲区"，不过目前显然已经超出了缓冲含义的范围。而Protobuf中的术语，则使用"message"来指代在协议传输过程中定义的抽象化对象，也显然不再仅仅只是原始含义的消息所能囊括的。

# 五、Proto3协议简述
当我们使用Visual Studio 2019创建一个.NET Core下的gRPC项目时，可以看到，项目会自带一个Protos\Greet.proto文件，这便是gRPC使用的Protobuf的接口描述文件，通过定义这个描述文件，可以为生成对应的服务端、客户端方法存根，让方法调用过程更加简单。

## 1、基本的数据类型对应关系
目前最新版本的Protobuf协议为proto3协议，在这个新版的协议中，提供了以下数据类型，可以方便的对应到我们日常使用的数据类型。

![图片](https://uploader.shimo.im/f/VtfJMUyiShQW3lJh.png!thumbnail)

## 2、关键字
### 1）分配字段编号
在proto协议中，每个消息定义中的字段都有唯一的编号，用来表示消息二进制格式中的字段，且使用消息类型后不应更改。可以使用的最小编号为1，最大编号为2^29^-1 或 536,870,911，但不包括 19000 到 19999（FieldDescriptor :: kFirstReservedNumber 到 FieldDescriptor :: kLastReservedNumber），因为它们是为 Protocol Buffers实现保留的。

### 2）重复字段（repeated）
在消息中定义重复字段（repeated 关键字），允许一个message 字段中重复数值，可以理解为数组对象。

### 3）保留字段(reserved)
Protobuf中提供了保留字段(reserved 关键字)，如果在老版本的proto文件中定义了一些字段，而在新版本的协议中移除了这些字段，有可能出现协议文件不匹配的问题，则可以使用reserved关键字。这样当协议数据不匹配时，编译器会提示错误。

![图片](https://uploader.shimo.im/f/ipi0BkYiRJQFrom6.png!thumbnail)

图2 使用保留字段时，会提示错误

## 3、枚举
允许在消息中定义枚举类型。也可以将枚举类型嵌套在message中。当使用枚举类型时，需要注意：

* 枚举为 0 的是作为零值，当不赋值的时候，就会是零值。
* 为了和 proto2 兼容。在 proto2 中，零值必须是第一个值。
## 4、消息嵌套 
在proto协议中，允许嵌套组合为更加复杂的消息。

```
message SearchResponse {
  repeated Result results = 1;
}
message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```
## 5、定义服务（Services)
在proto中，如果需要对外提供接口方法，则需要使用Services。定义好services之后，protocol buffer编译器将使用所选语言生成服务接口代码和客户端与服务端方法存根。例如，

```
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply);
}
```
这里就定义了一个SayHello的方法存根。该方法将返回一个名称为 HelloReply 的消息。
如果需要定义无参数方法，或返回值为 void 的方法，需要使用* google.protobuf.Empty** ***对象，**并在头部的命名空间中，引用默认的协议文件* google/protobuf/empty.proto. *例如:

```
option csharp_namespace = "TestGRPC_Client";
import "google/protobuf/empty.proto";
package Greet;
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply);
  rpc Listen (google.protobuf.Empty) returns (google.protobuf.Empty);//启动监听
}
```
同样，也可以使用 import 引用其他协议文件。
参考《[https://github.com/halfrost/Halfrost-Field/blob/master/contents/Protocol/Protocol-buffers-encode.md](https://github.com/halfrost/Halfrost-Field/blob/master/contents/Protocol/Protocol-buffers-encode.md)》 

# 六、什么是HTTP/2
gRPC 的客户端与服务端之间的通信机制，并没有采用TCP造轮子，而是重用了HTTP/2的传输协议。HTTP2.0是超文本传输协议HTTP的下一代协议，也是在传统开发者最为熟悉的HTTP/1.1 协议格式基础上进行的升级。 与1.1相比，他提供了新的二进制格式、多路复用机制、Header压缩、服务端推送等特色，让协议请求过程能够达到更好的性能提升。限于篇幅，这里就不再赘述了。

# 七、服务端开发
我们将引入一个范例，以HelloWorld作为项目名称，在这个项目中，简单介绍在NetCore中如何使用gRPC的过程、如何使用gRPC进行简单身份验证的过程。 

服务端：

* 并提供一个登录 login 的方法、以及其配套的用户请求参数、并返回对应的响应值。
* 一个 logout 的方法，该方法返回值为void空对象。
## 1、创建项目
在Visual Studio 2019中创建一个基于gRPC的空项目。这个项目命名为HelloWorld，放置在默认目录下。如果有需要可以开启容器支持。

![图片](https://uploader.shimo.im/f/2FFJ2OCBAM00Bhjk.png!thumbnail)

![图片](https://uploader.shimo.im/f/4FBORZaHgSoC6SsF.png!thumbnail)

![图片](https://uploader.shimo.im/f/1RCzdcGIqBMVNGaY.png!thumbnail)

## 2、项目的组成结构
当我们查看这个项目时，可以看到这是一个Asp.NET Core的项目，默认的项目模板中已经集成了Asp.NET Core和gRPC.AspNetCore的组件包。

![图片](https://uploader.shimo.im/f/68dD9ROgdFAgnysk.png!thumbnail)

* Protos文件夹

后缀名为proto的是基于proto3的协议文件。

![图片](https://uploader.shimo.im/f/VsmTsYVO000sqOTB.png!thumbnail)

* Services文件夹

项目模板创建的默认请求文件，实现了在proto文件中定义的SayHello方法，并以异步的形式返回了对象HelloReply。

![图片](https://uploader.shimo.im/f/hkuhhNGe5VIqh5nc.png!thumbnail)

* 其他文件

Dockerfile：模板自动创建的Dockerfile文件，后期可以基于这个文件进行docker容器的构建。

Program: 程序运行的入口。

Startup: 程序启动项，定义AspNET Core项目启动所需的各种配置信息。 在UseRouting和UseEndPoints中间，加入UseAuthentication()和UseAuthorization()代码，以便为后期身份认证和授权。。

![图片](https://uploader.shimo.im/f/RzPnf5UY6Zg2nH0o.png!thumbnail)

## 3、创建Proto文件
在Protos文件夹右键单击，创建一个空的记事本文件（快捷键为Ctrl+Shift+A)，命名为helloworld.proto。然后再里面键入以下内容： 

```
syntax = "proto3"
import "google/protobuf/empty.proto"; //需要使用空参数和空返回值时，需要使用这个默认的协议文件 
option csharp_namespace="HelloWorld";
package Account;
service Account{
	rpc Login (LoginModel) returns (UserModel);
	rpc Logout (google.protobuf.Empty) returns (google.protobuf.Empty);
}
message LoginModel{
	string userName=1;
	string userPsw=2;
}
message UserModel{
	string NickName=1;
	string Token=2;
	Date LoginDate=3;
}
message Date{
	int32 Year=1;
	int32 Month=2;
	int32 Day=3;
	int32 Hour=4;
	int32 Minute=5;
	int32 Second=6;
	int32 FFF=7;  
} 
```
## 4、创建 AccountService文件
选择 Services 文件夹，并创建一个文件名为 AccountService的CSharp代码文件。并分别重载 Login 和 Logout 方法。

```
 public class AccountService : account.accountBase
  {
      public override Task<UserModel> Login(LoginModel request, ServerCallContext context)
      {
          return base.Login(request, context);
      }
      public override Task<Empty> Logout(Empty request, ServerCallContext context)
      {
          return base.Logout(request, context);
      }
  }
```
然后再进行代码的编写。这里我们将登录后，返回一个假的 UserModel 数据。除此之外，我们还返回了错误情况下的返回模型 BadRequest 。
```
public class AccountService : account.accountBase
{
    public override Task<StringData> Login(LoginModel request, ServerCallContext context)
    {
        if (request.UserName == "1234" && request.UserPsd == "1234")
        {
            var userModel = new UserModel
            {
                NickName = "测试用户",
                Token = Guid.NewGuid().ToString(),
            }; 
            return Task.FromResult(new StringData() { Data = Newtonsoft.Json.JsonConvert.SerializeObject(userModel) });
        }
        else
        {
            var BadRequest = new BadRequest { ErrorCode = 1, ErrorDescription = "用户名或密码错误" };
            return Task.FromResult(new StringData() { Data = Newtonsoft.Json.JsonConvert.SerializeObject(BadRequest) });
        }
    }
    public override Task<Empty> Logout(Empty request, ServerCallContext context)
    {
        return Task.FromResult(new Empty());
    }
}
public class BadRequest
{
    public int ErrorCode { get; set; }
    public string ErrorDescription { get; set; }
}
```
# 八、客户端开发
客户端，是一个基于 .NET Core 的控制台程序。在这个控制台中，我们可以实现下面功能：

* 通过输入命令 1 调用登录方法； 
* 输入命令 2 调用登出方法。
## 1、创建项目、引用依赖包
创建一个基于.NET Core的一个控制台程序，并使用 Nuget 安装组件包

![图片](https://uploader.shimo.im/f/65nIhvxaUOoVRcuN.png!thumbnail)

## 2、创建协议文件
将在服务端开发中创建的 Protos 文件夹拷贝到客户端程序中。

![图片](https://uploader.shimo.im/f/97FoONttCLYEhe2C.png!thumbnail)

并使用记事本对项目文件【HelloWorld.Client.csproj】进行编辑， 将Protobuf 文件的GrpcServices属性设置为 “Client”。

完成这些操作，编译完成，即可自动生成客户端与服务端连接的客户端方法存根。 

## 3、编写客户端方法
创建一个单独的类文件，用来编写客户端调用方法。这个类文件名称为 AccountClientImpl。 代码如下：

```
using Grpc.Net.Client;
using System;
using System.Collections.Generic;
using System.Text;
using static HelloWorld.Greeter;
using System.Threading.Tasks;
namespace HelloWorld.Client
{
    public class AccountClientImpl
    {
        private readonly GrpcChannel _grpcChannel;
        private readonly Account.AccountClient _accountClient;
        public AccountClientImpl(GrpcChannel grpcChannel, Account.AccountClient accountClient)
        {
            _grpcChannel = grpcChannel;
            _accountClient = accountClient;
        }
        public void Login()
        {
            var result = _accountClient.Login(new LoginModel() { UserName = "1234", UserPsd = "1234" });
            Console.WriteLine(result.Data);
        }
        public void Logout()
        {
            var empty = new Google.Protobuf.WellKnownTypes.Empty();
            _accountClient.Logout(empty);
        }
    }


}

```
然后再修改 Program.cs 文件，用来调用上述方法。在这个方法中，如果输入1，则执行登录方法；输入2，则执行退出方法。
```
class Program
{
    static void Main(string[] args)
        {
            var channel = GrpcChannel.ForAddress("https://localhost:5001");
            var client = new Account.AccountClient(channel);

            AccountClientImpl accountClientImpl = new AccountClientImpl(channel, client);
            if (Console.ReadLine() == "1")
            {
                accountClientImpl.Login();
            }
            else if (Console.ReadLine() == "2")
            {
                accountClientImpl.Logout(); 
            } 
            Console.ReadKey();
        }
}
```
这样就完成了我们的代码编写。
将客户端与服务端运行起来，然后在客户端代码中输入数字 1 ；即可获得我们想要的结果。

# ![图片](https://uploader.shimo.im/f/YIfL2Jaox50FI9OB.png!thumbnail)
# 九、协议与项目分离
在传统的开发过程中，由于客户端和服务端需要维护两套内容完全相同的 proto 协议文件，略显臃肿，因此我们可以通过相应的手段，将对应的文件进行分离，便于后期的维护。

## 1、移动文件
将服务端中的Protos文件移动到上一级目录。

![图片](https://uploader.shimo.im/f/btxjOQUbvBkxsBQe.png!thumbnail)

## 2、修改项目文件中的Proto文件
服务端修改为：

```
  <ItemGroup>
    <Protobuf Include="..\Protos\*.proto" GrpcServices="Server" /> 
    <Content Include="@(Protobuf)" LinkBase="" />
  </ItemGroup> 
```
客户端修改为：
```
  <ItemGroup>
    <Protobuf Include="..\Protos\*.proto" GrpcServices="Client" /> 
    <Content Include="@(Protobuf)" LinkBase="" />
  </ItemGroup> 
```
## ![图片](https://uploader.shimo.im/f/pkDaJ7ObwXsbtPTx.png!thumbnail)
如果觉得这样的展示效果不太美观，也可以将proto文件移动到Protos目录下。

## 3、重新编译
完成协议文件分离，即可对项目进行编译。

# 总结
在这个教程中，我们从PRC开始讲起，简单介绍了与gRPC相关的技术栈，练习了使用 gRPC 进行服务端和客户端程序开发的全过程，希望大家能获得收获。

第一次尝试编写入门级教程，如有不足之处还请匹配指正。

