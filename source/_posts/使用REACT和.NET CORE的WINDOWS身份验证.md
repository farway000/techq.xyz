### 基于REACT和.NET CORE集成WINDOWS身份验证

有很多方法可以向您的应用程序添加身份验证。虽然OAuth是最常见的一种，但这并不是您唯一的选择。今天，我将向您展示如何通过React和.NET Core简单地完成Windows身份验证。



### 探索我们的选择

在深入探讨之前，让我们简要讨论一些可用的其他选项。了解您的选择，可以使您根据自己的情况做出最佳（受过良好教育）的决定。这绝不是关于替代方案的详尽讨论，而只是其中一些较流行的替代方案。

[Okta](https://www.okta.com/)是一家身份和访问管理公司，提供基于云的解决方案。他们有[Active Directory](https://www.okta.com/active-directory/)的[插件/提供程序](https://www.okta.com/active-directory/)。他们的站点包含一个教程，如何[开始向](https://developer.okta.com/code/react/)您的React应用程序[添加身份验证](https://developer.okta.com/code/react/)。在商业解决方案方面，该解决方案经常被推荐使用。

[Auth0](https://auth0.com/)是另一个具有良好关注度的商业解决方案。他们有专门针对将[Active Directory / LDAP与React结合](https://auth0.com/authenticate/react/active-directory/)使用的教程。

[IdentityServer](https://identityserver.io/)是开源替代方案。就像其他人一样，他们提供了有关[如何实施Windows身份验证的说明](https://docs.identityserver.io/en/latest/topics/windows.html)。他们有一个有关[如何实现javascript客户端](http://docs.identityserver.io/en/release/quickstarts/7_javascript_client.html)（例如React）的示例。这是有关[在Identity Server 4中使用SPA（反应/角度）UI的文章](https://medium.com/@piotrkarpaa/using-spa-react-angular-ui-with-identity-server-4-dc1f57e90b2c)。

当然，我们今天在这里讨论的选择是推出您自己的解决方案。

#### 为什么不选择交钥匙解决方案？

尽管您的原因可能有所不同，但我想出了一些原因。

1. 基础结构/资源不足，无法设置服务（适用于Identity Server 4）。
2. 资金/预算不足，无法支付SAAS提供商的费用。
3. 您位于防火墙之内，并且React和.NET应用程序都位于同一网络上。
4. 您喜欢挑战和/或喜欢重新发明轮子（哈！）

好的，也许最后一个有点有趣。一般来说，使用交钥匙解决方案*是*最好的选择，但不是*您的*选择。在我遇到的这个特殊用例中，这不是我想要的。然而。

### 入门

为了构建此应用程序，我们需要两件事：

1. .NET Core API项目–该项目将处理身份验证，授权以及API调用
2. React应用程序–该项目是我们的GUI

我为该项目学习和应用的东西是Google-fu精通和反复试验编码的结合。希望汇总我的经验将使您（我的读者）比我更容易。

我首先创建一个文件夹来包含我的API和React项目，然后运行`dotnet new api -o ReactWindowsAuth`以搭建一个新的API项目。（这实际上是一个谎言，我使用VS来创建应用程序，但是我们假装使用了CLI）。从那里开始，我运行`npm create-react-app test-app`了一个名为的基本React应用程序`test-app`。就个人而言，我喜欢将Visual Studio用于.NET代码，将VSCode用于几乎所有其他内容，因此我在VS中打开了新的API项目并生成了解决方案文件。

从这里开始，我们现在可以开始使用React和.NET Core实施Windows身份验证了！

### 框架.NET Core API

我们需要做的第一件事就是确保我们的应用程序以Windows身份验证运行。由于我使用VS生成了项目并提供了Docker支持，因此我不得不做一些您可能不需要做的事情。

#### 启用Windows身份验证

我要做的第一件事是将调试启动器从Docker切换到IIS Express。

![切换VS默认启动器](https://www.seeleycoder.com/wp-content/uploads/2020/06/react-ad-auth-01.png)切换默认启动

接下来，我需要打开我的`launchSettings.json`并`"windowsAuthentication": true`在`iisSettings`钥匙下进行设置。

[![在launchSettings.json中启用Windows身份验证](https://www.seeleycoder.com/wp-content/uploads/2020/06/react-ad-auth-02-1.png)](https://www.seeleycoder.com/wp-content/uploads/2020/06/react-ad-auth-02-1.png)启用Windows身份验证

好吧，让我们备份一秒钟。为了使Windows身份验证起作用，您将需要在IIS或IIS Express中进行托管。您也可以使用[Kestrel](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/windowsauth?view=aspnetcore-3.1&tabs=visual-studio#kestrel)和[HTTP.sys](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/windowsauth?view=aspnetcore-3.1&tabs=visual-studio#httpsys)托管来完成此操作，但出于本文的方便，让我们集中讨论IIS Express。如果您想使它在Docker和/或Linux上运行，您将要使用Kestrel。

我们的应用程序现在可以与Windows身份验证一起使用了，但是如果我们现在启动它，它仍然不会使用。我们还有很多工作要做。

#### 配置Windows身份验证

现在我们已经设置了API以通过IIS使用Windows身份验证，我们需要使API本身意识到这一点。为此，我们需要对进行一些调整`Startup.cs`。 [MSDN上有文档](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/windowsauth?view=aspnetcore-3.1&tabs=visual-studio#iisiis-express)，但我们也可以在这里进行阅读。

您需要做的第一件事是`services.AddAuthentication(IISDefaults.AuthenticationScheme);`在您的`ConfigureServices`方法中添加任何位置。这利用了`Microsoft.AspNetCore.Server.IISIntegration`命名空间。

![将AD身份验证添加到ConfigureServices](https://www.seeleycoder.com/wp-content/uploads/2020/06/react-ad-auth-03.png)添加Windows身份验证

接下来，您需要配置Windows身份验证需要保护哪些控制器或动作。顺便说一下，这就是为什么我们`"anonymousAuthentication": true`独自留在`launchSettings.json`。这里的一个用例是，如果您使用的是Swagger，并且希望匿名访问文档并且仅保护API本身。

在我的用例中，我假装我希望所有API控制器都需要身份验证。鉴于此，我创建了一个`WebControllerBase.cs`，并用`[Authorize]`属性对其进行了装饰。请注意，您可以改为用[Authorize]标记每个控制器。就是说，我的理由实际上是押韵的，在以后的第二部分中，我将进一步阐述这个想法。现在，只需滚动即可。

![具有[Authorize]属性的WebControllerBase基本实现](https://www.seeleycoder.com/wp-content/uploads/2020/06/react-ad-auth-04.png)WebControllerBase，将[Authorize]标记为需要身份验证

接下来，只需让我们现有的控制器和新控制器继承自`WebControllerBase`而不是即可`ControllerBase`。

![从WebControllerBase继承以要求Windows身份验证](https://www.seeleycoder.com/wp-content/uploads/2020/06/react-ad-auth-05.png)现在，我们现有的控制器继承自WebControllerBase

在网络上的其他示例中，您可能会看到人们说您需要添加`app.UseAuthentication()`该`Configure(IApplicationBuilder app, IWebHostEnvironment env)`方法。如果仅针对IIS / IIS Express，则不会。也就是说，添加它不会伤害您。我不会判断你是否愿意。我的源代码有它。

#### 测试一下！

现在，我们可以对其进行测试了。在WeatherForecastController顶部打一个断点：以调试模式获取并运行Web应用程序。达到断点时，添加监视`HttpContext.User`并向下钻取`.Identity.Name`。（可选）只需在方法顶部添加此行：`var user = HttpContext.User?.Identity?.Name ?? "N/A";`并查看结果。您的应用程序现在正在报告您当前的Windows用户名，对吗？

### 哇，等等，您还没有完成！

我们已经接近了，但是如果我们想在React中使用它，我们还有很多工作要做。CORS。别说了。什么是CORS？ [CORS是跨域资源共享](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)，这是您要允许[this]访问[that]的一种非常真实的方式。在现实世界中，默认情况下将浏览器配置为禁止通过脚本发起的HTTP请求，除非接收端明确允许。

所以……让我们做一下，这样这个API可以接受来自React应用程序的CORS请求，对吧？我们不会在这里变得很花哨。为了使此操作生效，我们需要在中添加一些设置，`appsettings.json`然后在中进行其他设置`Startup.cs`。

#### appsettings.json

首先，我们去`appsettings.json`添加以下内容：`"CorsOrigins": [ "http://localhost:3000" ],`。是的，它是数组类型。为什么？主要是因为它为您提供了选择。假设在开发中，我打开了通往机器的通道，以便某人（甚至我）可以从另一台设备访问该应用程序。显然，它们不会通过本地主机名或IP地址连接到localhost。所有这些都是我可能要设置的选项。

#### 启动文件

接下来，我们需要[将CORS添加](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-3.1)到我们的服务和中间件中。在ConfigureServices中，请添加以下代码：

```
// add this class somewhere outside of the Startup class
public class Constants
{
	public const string CORS_ORIGINS = "CorsOrigins";
}

services.AddCors(opt =>
{
	opt.AddPolicy("CorsPolicy", builder => builder
		.AllowAnyHeader()
		.AllowAnyMethod()
		.WithOrigins(Configuration.GetSection(Constants.CORS_ORIGINS).Get<string[]>())
		.AllowCredentials());
});
```

此代码的简要说明。它允许传递任何标头，使用任何http方法（GET，POST，PUT，DELETE等），必须来自配置中特定的来源之一，并允许在标头中传递凭据。在您自己的应用程序中，您可以更改许多设置。您可能不会更改其中任何一个。至少您现在知道它们了。

接下来，我们需要添加`app.UseCors("CorsPolicy")`到我们的`Configure(app, env)`方法中。请注意，这是中间件和[中间件顺序](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-3.1#middleware-order)。在这种情况下，它需要跟从`app.UseRouting()`但在此之前`app.UseAuthentication()`和`app.UseAuthorization()`。顺便说一句，如果您添加了中间件，但中间件工作不正常，则应检查其注册顺序。

![现在，Configure已准备好将Windows身份验证与React结合使用](https://www.seeleycoder.com/wp-content/uploads/2020/06/react-ad-auth-06-1.png)现在使用app.UseCors（“ CorsPolicy”）配置方法

现在我们准备好让我们的React应用程序与Windows身份验证挂钩了！

### 带有React的Windows身份验证–连接起来！

对此感到兴奋吗？我知道我是。这比您想象的要容易。准备好了吗？

您需要做的就是在`fetch`请求中添加两个属性：`credentials: "include"`和`mode: 'cors'`。

将会发生的情况是，如果您访问的站点与您不在同一域（或计算机）上，则浏览器将提示您输入该Active Directory，LDAP或计算机实例的凭据。成功进行身份验证后，浏览器将其存储以备将来使用。如果您在完全相同的计算机或域上，则不会提示凭据。

话虽如此，您可以（并且可能应该）设置服务f调用的方式，因此不必在各处都输入相同的垃圾。当我第一次开始使用React时，我很快意识到，设置一些获取帮助程序来启动它比较容易。我意识到的第二件事是，在React代码中将所有.NET API控制器与“服务”进行匹配更加容易。

考虑到这一点，让我们看一下我的提取帮助器和示例服务。

#### fetch-helpers.js

这是我在此测试应用程序中拥有的一些基础知识：

```
export const handleResponse = (response) => {
  return response.text().then((text) => {
    const data = text && JSON.parse(text);
    if (!response.ok) {
      const error = (data && data) || response.statusText;
      return Promise.reject(error);
    }

    return data;
  });
};

export const requestBase = (() => {
  if (typeof window !== "undefined") {
    return {
      method: "POST",
      credentials: "include",
      mode: 'cors',
      headers: new Headers({
        Accept: "application/json",
        "Content-Type": "application/json",
      }),
    };
  } else
    return {
      method: "POST",
      credentials: "include",
      // mode: 'cors',
      headers: {
        Accept: "application/json",
        "Content-Type": "application/json",
      },
    };
})();
```

值得注意的是，`requestBase`如果它不是NodeJS（经过渲染的React），则可以具有不同的返回对象。很难发现差异，但是无浏览器版本将标头设置为Headers对象的实例，而浏览器版本仅设置JSON对象。

#### weather-api.js

接下来，让我们看看我们的服务如何使用fetch-helpers。首先，下面的apiBase是完整的URL。显然，您不会这样做，但实际上会将React的baseUrl设置为更高的级别（通常在HTML级别）。这是示例代码，请加一点盐。

```
import { handleResponse, requestBase } from "../_helpers";

const apiBase = "https://localhost:44387/weatherforecast";

class WeatherForecastService {
  getForecasts() {
    let request = Object.assign({}, requestBase, { method: "GET" });
    let url = `${apiBase}`;

    return fetch(url, request).then(handleResponse);
  }

  getProtectedForecast() {
    let request = Object.assign({}, requestBase, { method: "GET" });
    let url = `${apiBase}/5`;

    return fetch(url, request).then(handleResponse);
  }
}

const instance = Object.freeze(new WeatherForecastService());
export { instance as WeatherForecastService };
```

我在这里所做的只是公开我希望React可以访问的方法，将它们包装在`baseRequest`from的周围，并`fetch-helpers`使用我的`handleResponse`from 来处理响应`fetch-helpers`，然后传递回调用方。

#### 最后但同样重要的是，将其连接起来！

现在我们已经铺设了所有管道，现在该连接所有东西了。我只是直接编辑App.js。告我。对于该示例，我将导入所有三个API服务文件，然后`const`为我要处理的每个按钮设置一些功能。其中的每一个都仅注销到控制台，而不用花费大量精力。最后，当然是按钮本身。由于该文件在修改后相当庞大，因此我仅摘录了按钮事件之一以及调用它的React按钮组件本身。

```
const getProtectedForecast = () => {
console.log("attempting...");

WeatherForecastService.getProtectedForecast()
  .then((response) => {
	console.log("response: ", JSON.stringify(response));
	console.log("oh boy!");
  })
  .catch((err) => {
	console.error(err);
  });
};

<button type="button" onClick={getProtectedForecast}>
Get protected forecast
</button>
```

![React中需要Windows身份验证的API调用的结果](https://www.seeleycoder.com/wp-content/uploads/2020/06/react-ad-auth-07.png)这是我们荣耀的结果

### 重要环节– Windows身份验证基于角色的安全性

虽然上述设置是非常基本的，但它缺少一个非常重要的部分，不是吗？基于角色的安全性。在这里考虑一下此内容，这是对第2部分的非常简短的介绍，而我将在不久的将来写一篇文章。如果你已经点击周围的代码，而阅读这篇文章，你可能已经注意到在Startup.cs以下行：ConfigureServices： `services.AddTransient();`。如果您没有注意到它，那是可以的，因为我现在将谈论它。

在对用户进行身份验证之后但在获得授权之前，授权提供者将调用您的自定义`IClaimsTransformation`实现（如果提供）。 [参见MSDN](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.iclaimstransformation?view=aspnetcore-3.1)。在我的实现中（位于中`ClaimsTransformer`），您将看到我只是在随意添加角色“ Super-awesome”作为我们`WindowsIdentity`用于其声明类型的相同声明类型。

![IClaimsTransformation的ClaimsTransformer实现者](https://www.seeleycoder.com/wp-content/uploads/2020/06/react-ad-auth-08.png)IClaimsTransformation的ClaimsTransformer实现者

接下来，您可能已经注意到`WeatherForecastController`我添加了一个`GetAnother`方法调用，该方法调用需要“超级棒”角色。在此之下，我还添加了一种`GetFail`方法，该方法将始终失败，因为用户不在“管理员”角色中。这些方法都没有连接到React应用程序中，并且要求您直接在浏览器中单击它们以查看它们是否有效（或可能无效）。

![基于Controller动作的基于角色的Windows身份验证](https://www.seeleycoder.com/wp-content/uploads/2020/06/react-ad-auth-09.png)基于Controller动作的基于角色的Windows身份验证

### 结论

将Windows身份验证连接到您的React应用程序并不困难。这也是非常基本的。交钥匙解决方案可以使您走得更快，更远，但是如果您没有基础设施或现金，仍然可以自己动手做。和往常一样，我博客文章中的代码可以在[GitHub](https://github.com/fooberichu150/react-windows-auth)上[找到](https://github.com/fooberichu150/react-windows-auth)。