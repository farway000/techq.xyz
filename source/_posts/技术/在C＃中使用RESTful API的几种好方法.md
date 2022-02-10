---
title:  在C＃中使用RESTful API的几种好方法
date: 2020-04-28 23:28
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# 在C＃中使用RESTful API的几种好方法

 [Vladimir Pecanac](https://code-maze.com/author/codemaze_blog/) 

通过Web开发的路径，您发现自己迟早需要处理外部API（应用程序编程接口）。在本文中，我的目标是列出在C＃项目中使用RESTful API的方法的最全面列表，并通过一些简单示例向您展示如何做到这一点。

阅读该文章后，您将更深入地了解可以使用哪些选项，以及下次需要使用[RESTful API](https://code-maze.com/ultimate-aspnet-core-3-web-api/)时如何选择正确的选项。

## 什么是RESTful API？

因此，在开始之前，您可能想知道[API](https://code-maze.com/ultimate-aspnet-core-3-web-api/)代表什么，以及RESTful的全部含义是什么？

简而言之，API是软件应用程序之间的层。您可以将请求发送到[API](https://code-maze.com/ultimate-aspnet-core-3-web-api/)，并从中获得响应。API隐藏了软件应用程序具体实现的所有细节，并公开了您用于与该应用程序通信的接口。

整个互联网是由API组成的大型蜘蛛网。我们使用API在应用程序之间通信和关联信息。我们有一个[API](https://code-maze.com/ultimate-aspnet-core-3-web-api/)，可以处理几乎所有内容。您每天使用的大多数服务都有自己的API（GoogleMaps，Facebook，Twitter，Instagram，天气门户…）

RESTful部分意味着API是根据REST（表示状态传输）的原理和规则来实现的，REST是网络的基础架构原理。RESTful API在大多数情况下会返回纯文本，JSON或XML响应。更详细地解释REST不在本文的讨论范围之内，但是您可以在我们的文章[REST API最佳实践中](https://code-maze.com/top-rest-api-best-practices/)阅读有关REST的更多信息。

## 如何使用RESTful API

好吧，让我们进入整个故事中最重要的部分。

有几种方法可以在C＃中使用RESTful API：

- [**HttpWebRequest/Response Class**](https://code-maze.com/different-ways-consume-restful-api-csharp/#HttpWebRequest)
- [**WebClient Class**](https://code-maze.com/different-ways-consume-restful-api-csharp/#webclient)
- [**HttpClient Class**](https://code-maze.com/different-ways-consume-restful-api-csharp/#HttpClient)
- [**RestSharp NuGet Package**](https://code-maze.com/different-ways-consume-restful-api-csharp/#RestSharp)
- [**ServiceStack Http Utils**](https://code-maze.com/different-ways-consume-restful-api-csharp/#ServiceStack)
- **[Flurl](https://code-maze.com/different-ways-consume-restful-api-csharp/#flurl)**
- **[DalSoft.RestClient](https://code-maze.com/different-ways-consume-restful-api-csharp/#dalsoft)**

这些中的每一个都有优点和缺点，因此让我们仔细研究它们，看看它们提供了什么。

例如，我们将通过GitHub API收集有关RestSharp回购版本及其发布日期的信息。此信息是公开可用的，您可以在此处查看原始JSON响应的外观： [RestSharp版本](https://api.github.com/repos/restsharp/restsharp/releases)

我们将利用Json.NET库的帮助来反序列化获得的响应。同样，对于某些示例，我们将使用库的内置反序列化机制。选择哪种方式取决于您，因为没有正确的方法。（您可以在[源代码中](https://github.com/CodeMazeBlog/ConsumeRestfulApisExamples)看到这两种机制的实现）。

我期望通过接下来的几个示例得到一个反序列化`JArray`（为简单起见），其中包含RestSharp发布信息。之后，我们可以遍历它以获得以下结果。

 ![img](https://code-maze.com/wp-content/uploads/2017/06/RestSharp-releases.png) 

## HttpWebRequest / Response类

这是`WebRequest` 类的特定于HTTP的实现，该实现最初用于处理HTTP请求，但已过时并由`WebClient`该类代替 。

该`HttpWebRequest` 提供细粒度控制的要求制定过程的每一个环节。您可以想象，这可能是一把双刃剑，您很容易浪费大量时间来微调您的请求。另一方面，这可能正是您针对特定案例所需要的。

`HttpWebRequest` 类不会阻止用户界面，也就是说，我相信您会同意这一点，这一点非常重要。

`HttpWebResponse` 类为传入的响应提供了一个容器。

这是有关如何使用这些类使用API的简单示例。

```
public class HttpWebRequestHandler : IRequestHandler
{
    public string GetReleases(string url)
    {
        var request = (HttpWebRequest)WebRequest.Create(url);

        request.Method = "GET";
        request.UserAgent = RequestConstants.UserAgentValue;
        request.AutomaticDecompression = DecompressionMethods.Deflate | DecompressionMethods.GZip;

        var content = string.Empty;

        using (var response = (HttpWebResponse)request.GetResponse())
        {
            using (var stream = response.GetResponseStream())
            {
                using (var sr = new StreamReader(stream))
                {
                    content = sr.ReadToEnd();
                }
            }
        }

        return content;
    }
}
```

 尽管是一个简单的示例，但是当您需要处理更复杂的方案（例如，发布表单信息，授权等）时，它会变得更加复杂。 

## WebClient类别

这个类对`HttpWebRequest`的包装。它通过`HttpWebRequest`从开发人员中提取的细节来简化流程。该代码更容易编写，并且您通过这种方式犯错误的可能性较小。如果您想编写更少的代码，而不用担心所有细节，并且执行速度是不重要的，请考虑使用`WebClient`class。

这个示例应该使您大致了解`WebClient`与`HttpWebRequest`/ `HttpWebResponse`方法相比使用起来要容易得多。

```
public string GetReleases(string url)
{
    var client = new WebClient();
    client.Headers.Add(RequestConstants.UserAgent, RequestConstants.UserAgentValue);

    var response = client.DownloadString(url);

    return response;
}
```

容易得多，对吗？

除了其他`DownloadString`方法，`WebClient`类还提供了许多其他有用的方法，使我们的生活更轻松。我们可以轻松地使用它来操作字符串，文件或字节数组，并且价格比`HttpWebRequest`/ `HttpWebResponse`方法要慢几毫秒。

无论是`HttpWebRequest`/ `HttpWebResponse`和`WebClient`类在旧版本的.NET可供选择。如果您对其他产品感兴趣，请务必查看[MSDN](https://msdn.microsoft.com/en-us/library/system.net.webclient(v=vs.110).aspx)`WebClient`。

## HttpClient类

`HttpClient` 是“新人”，它提供了旧库所缺乏的一些现代.NET功能。例如，您可以使用的单个实例发送多个请求`HttpClient`，它不绑定到特定的HTTP服务器或主机，而是使用async / await机制。

您可以[在此视频中](https://channel9.msdn.com/Events/Build/2013/4-092)找到[使用HttpClient](https://channel9.msdn.com/Events/Build/2013/4-092)的[五个很好的理由](https://channel9.msdn.com/Events/Build/2013/4-092)：

- 强类型标题。
- 共享缓存，cookie和凭据
- 访问cookie和共享cookie
- 控制缓存和共享缓存。
- 将您的代码模块注入ASP.NET管道。清洁和模块化的代码。

`HttpClient`在我们的示例中，这是实际的：

```
public string GetReleases(string url)
{
    using (var httpClient = new HttpClient())
    {
        httpClient.DefaultRequestHeaders.Add(RequestConstants.UserAgent, RequestConstants.UserAgentValue);

        var response = httpClient.GetStringAsync(new Uri(url)).Result;

        return response;
    }
}
```

为了简单起见，我同步实现了它。每个`HttpClient`方法都应异步使用，应该以这种方式使用。

另外，我还要提到一件事。**是否`HttpClient`应该包装在using块中还是在应用程序级别上进行静态讨论。尽管它实现了`IDisposable`，但似乎通过将它包装在using块中，[会使应用程序出现故障并获得SocketException](https://aspnetmonsters.com/2016/08/2016-08-27-httpclientwrong/)。**而在ANKIT博客中，提供了基于[很多有利于静态初始化的](https://ankitvijay.net/2016/09/25/dispose-httpclient-or-have-a-static-instance/)的`HttpClient`性能测试结果是。**请务必阅读这些博客文章，**因为它们可以帮助您更了解该`HttpClient` 库的正确用法。（原文编写时间比较旧，在新版的.NET Core3.1中，相关问题已经解决）

并且不要忘记，由于是新的，`HttpClient`是.NET 4.5以上版本才有，因此在某些旧项目中使用它可能会遇到麻烦。

## RestSharp

RestSharp是标准.NET库的OpenSource替代品，也是目前最酷的.NET库之一。它以NuGet软件包的形式提供，出于某些原因，您应该考虑尝试一下。

就像`HttpClient`RestSharp 一样，它是一个现代而全面的库，易于使用且令人愉悦，同时仍支持旧版本的.NET Framework。它具有内置的[身份验证](https://github.com/restsharp/RestSharp/wiki/Authenticators)和[序列化/反序列化机制，](https://github.com/restsharp/RestSharp/wiki/Deserialization)但允许您使用自定义[机制](https://github.com/restsharp/RestSharp/wiki/Deserialization)覆盖它们。它[可跨平台使用，](https://github.com/restsharp/RestSharp/wiki/Cross-Platform-Support)并支持OAuth1，OAuth2，基本，NTLM和基于参数的身份验证。您可以选择同步或异步工作。该库还有很多其他功能，而这些只是它提供的众多好处中的一部分。有关RestSharp的用法和功能的详细信息，您可以访问[GitHub上](https://github.com/restsharp/RestSharp/wiki/Getting-Started)的RestSharp [页面](https://github.com/restsharp/RestSharp/wiki/Getting-Started)。

现在，让我们尝试使用RestSharp get获取RestSharp版本的列表。

```
public string GetReleases(string url)
{
    var client = new RestClient(url);

    var response = client.Execute(new RestRequest());

    return response.Content;
}
```

很简单。RestSharp非常灵活，拥有使用RESTful API时几乎可以实现所有功能所需的所有工具。

在此示例中要注意的一件事是，由于示例的一致性，我没有使用RestSharp的反序列化机制，这有点浪费，但是我鼓励您使用它，因为它确实非常容易和方便。因此，您可以轻松地制作一个这样的容器：

```
public class GitHubRelease
{
    [JsonProperty(PropertyName = "name")]
    public string Name { get; set; }
    [JsonProperty(PropertyName = "published_at")]
    public string PublishedAt { get; set; }
}
```

 然后使用该`Execute`方法直接反序列化对该容器的响应。您可以仅添加所需的属性，并使用属性`JsonProperty`将它们映射到C＃属性（很好的触摸）。由于我们在响应中获得了发布列表，因此我们将`List` 用作包含类型。 

```
public List<GitHubRelease> GetDeserializedReleases(string url)
{
    var client = new RestClient(url);

    var response = client.Execute<List<GitHubRelease>>(new RestRequest());

    return response.Data;
}
```

一种非常直接而优雅的方式来获取我们的数据。

RestSharp不仅具有发送`GET`请求的功能，还可以自己探索并观察它的酷炫之处。

在RestSharp案例中要补充的最后一点是，其存储库需要维护者。如果您想了解更多有关这个很棒的库的信息，我敦促您前往[RestSharp存储库](https://github.com/restsharp/RestSharp)，帮助该项目继续发展并变得更好。

## ServiceStack Http实用程序

另一个库，但与RestSharp不同，ServiceStack似乎得到了适当维护，并与现代[API](https://code-maze.com/ultimate-aspnet-core-3-web-api/)趋势保持同步。ServiceStack功能列表令人印象深刻，并且肯定具有各种应用程序。

在这里对我们最有用的是演示如何使用外部RESTful API。ServiceStack具有一种专门的方式来处理称为[Http Utils的](http://docs.servicestack.net/http-utils)第三方HTTP API 。

让我们看看如何首先使用Json.NET解析器来获取RestSharp版本是如何使用ServiceStack Http Utils。

```
public string GetReleases(string url)
{
    var response = url.GetJsonFromUrl(webReq =>
    {
        webReq.UserAgent = RequestConstants.UserAgentValue;
    });

    return response;
}
```

 您还可以选择将其留给ServiceStack解析器。我们可以重用本文前面定义的`Release`类 。 

```
public List<GitHubRelease> GetDeserializedReleases(string url)
{
    var releases = url.GetJsonFromUrl(webReq =>
    {
        webReq.UserAgent = RequestConstants.UserAgentValue;
    }).FromJson<List<GitHubRelease>>();

    return releases;
}
```

如您所见，无论哪种方式都可以正常工作，并且您可以选择是获取字符串响应还是立即反序列化它。

尽管ServiceStack是我们偶然发现的最后一个库，但令我感到惊讶的是，它使用起来如此容易，而且我认为它将来可能成为我处理API和服务的首选工具。

## Flurl

评论库中许多人要求的图书馆之一，并在Internet上受到许多人的喜爱，但仍吸引着人们。

Flurl代表Fluent Url Builder，这是库构建其查询的方式。对于不熟悉flurl的做事方式的人来说，flurl只是意味着库的构建方式是将方法链接在一起以实现更高的可读性，类似于人类语言。

为了使事情更容易理解，让我们举一些例子（这个例子来自官方文档）：

```
// Flurl will use 1 HttpClient instance per host
var person = await "https://api.com"
    .AppendPathSegment("person")
    .SetQueryParams(new { a = 1, b = 2 })
    .WithOAuthBearerToken("my_oauth_token")
    .PostJsonAsync(new
    {
        first_name = "Claire",
        last_name = "Underwood"
    })
    .ReceiveJson<Person>();
```

您可以看到方法如何链接在一起以完成“句子”。

在后台，Flurl使用HttpClient或通过自己的语法糖增强HttpClient库。因此，这意味着Flurl是一个异步库，因此请牢记这一点。

与其他高级库一样，我们可以通过两种不同的方式来做到这一点：

```
public string GetReleases(string url)
{
    var result = url
        .WithHeader(RequestConstants.UserAgent, RequestConstants.UserAgentValue)
        .GetJsonAsync<List<GitHubRelease>>()
        .Result;

    return JsonConvert.SerializeObject(result);
}
```

这种方式相当糟糕，因为我们只是序列化结果，以便稍后对其进行反序列化。如果您使用的是Flurl之类的库，则不应以这种方式进行操作。

更好的做事方式是：

```
public List<GitHubRelease> GetDeserializedReleases(string url)
{
    var result = url
        .WithHeader(RequestConstants.UserAgent, RequestConstants.UserAgentValue)
        .GetJsonAsync<List<GitHubRelease>>()
        .Result;

    return result;
}
```

 随着`.Result`我们强迫代码的同步行为。使用Flurl的实际和预期方式如下所示： 

```
public async Task<List<GitHubRelease>> GetDeserializedReleases(string url)
{
    var result = await url
        .WithHeader(RequestConstants.UserAgent, RequestConstants.UserAgentValue)
        .GetJsonAsync<List<GitHubRelease>>();

    return result;
}
```

这展示了Flurl库的全部潜力。

如果您想了解更多有关如何在不同的现实生活场景使用Flurl，看看我们的 [消费GitHub的API（REST）随着](https://code-maze.com/consuming-github-api-rest-with-flurl/)[Flurl](https://code-maze.com/wp-admin/post.php?post=4822&action=edit) 文章

总而言之，它就像广告一样：易于使用，现代，可读性和可测试性。您对这个库还有什么期望？要开源？[签](https://github.com/tmenier/Flurl)出： [Flurl存储库](https://github.com/tmenier/Flurl)，如果您愿意，可以贡献自己的力量！

## DalSoft.RestClient

现在，此列表与该列表中的任何内容都有些不同。但这一点有所不同。

让我们看看如何使用DalSoft.RestClient来使用GitHub API，然后谈论我们已完成的工作。

首先，您可以通过输入以下内容，通过NuGet软件包管理器下载DalSoft.RestClient： `Install-Package DalSoft.RestClient`

或通过.NET Core CLI： `dotnet add package DalSoft.RestClient`

两种方法都可以。

拥有图书馆后，我们可以执行以下操作：

```
public string GetReleases(string url)
{
    dynamic client = new RestClient(RequestConstants.BaseUrl,
        new Headers { { RequestConstants.UserAgent, RequestConstants.UserAgentValue } });

    var response = client.repos.restsharp.restsharp.releases.Get().Result.ToString();

    return response;
}
```

 或最好使用DalSoft.RestClient在充分利用其功能的同时立即反序列化响应： 

```
public async Task<List<GitHubRelease>> GetDeserializedReleases(string url)
{
    dynamic client = new RestClient(RequestConstants.BaseUrl,
        new Headers { { RequestConstants.UserAgent, RequestConstants.UserAgentValue } });

    var response = await client.repos.restsharp.restsharp.releases.Get();

    return response;
}
```

因此，让我们稍微讨论一下这些例子。

乍一看，它似乎并不比我们使用的其他一些现代库简单得多。

但这归结为形成请求的方式，那就是利用RestClient的动态特性。例如，我们的BaseUrl是`https://api.github.com` ，我们需要进入`https://api.github.com/repos/restsharp/restsharp/releases`。我们可以通过动态创建客户端，然后通过链接Url的“部分”来形成Url来做到这一点：

```
await client.repos.restsharp.restsharp.releases.Get();
```

形成请求的一种非常独特的方法。还有一个非常灵活的！

因此，一旦我们设置了基本的网址，就可以轻松地使用不同的端点。

还值得一提的是，我们得到的JSON响应会自动进行类型转换。如您在第二个示例中看到的那样，我们方法的返回值是`Task>.` So，该库足够聪明，可以将响应转换为我们的类型（依赖于Json.NET）。这使我们的生活更加轻松。

除了易于理解和使用之外，DalSoft.RestClient还具有现代库应具备的所有功能。它是可**配置的，异步的，可扩展的，可测试的，并且支持多个平台**。

我们仅演示了DalSoft.RestClient功能的一小部分。如果您对使用DalSoft.RestClient感兴趣，请转至[我们的文章，](https://code-maze.com/dalsoft-restclient-consume-any-rest-api/)以学习如何在不同情况下使用它，或参阅 [GitHub官方仓库](https://github.com/DalSoft/DalSoft.RestClient)和[文档](https://restclient.dalsoft.io/)。

## 其他选择

对于您的特定问题，还有许多其他选项可用。您可以使用任何这些库来使用特定的RESTful API。例如，[octokit.net专门 ](https://github.com/octokit/octokit.net)用于GitHub API，[Facebook SDK](https://github.com/facebook-csharp-sdk/facebook-csharp-sdk) 用于使用Facebook API，并且还有许多其他功能可用于任何用途。

虽然这些库是专门为这些API而设计的，并且可能擅长于它们的用途，但它们的用途是有限的，因为您经常需要在应用程序中连接多个[API](https://code-maze.com/ultimate-aspnet-core-3-web-api/)。这可能会导致每个实现都有不同的实现方式，以及更多的依赖关系，这可能导致重复并且容易出错。库越具体，其灵活性就越差。

## GitHub上的源代码

[GitHub上的源代码](https://github.com/CodeMazeBlog/ConsumeRestfulApisExamples)

## 结论

因此，总而言之，我们已经讨论了可用于使用RESTful API的不同工具。我们已经提到了一些.NET库，可以这样做`HttpWebRequest`，`WebClient`和`HttpClient`，以及一些惊人的第三方工具，如RestSharp和ServiceStack。您还对这些工具进行了简短的介绍，并给出了一些非常简单的示例来向您展示如何开始使用它们。

我认为您现在至少有95％准备使用一些REST。继续展开翅膀，探索并找到更多有趣且有趣的方式来使用和连接不同的API。