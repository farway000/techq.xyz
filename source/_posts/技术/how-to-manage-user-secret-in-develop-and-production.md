---
title:  在Asp.NET Core中如何优雅的管理用户机密数据
date: 2020-6-9 20:05
tags: 技术
author: 邹溪源
categories:
  - 技术
---



# 在Asp.NET Core中如何优雅的管理用户机密数据

## 背景

#### 回顾

在软件开发过程中，使用配置文件来管理某些对应用程序运行中需要使用的参数是常见的作法。在早期VB/VB.NET时代，经常使用.ini文件来进行配置管理；而在.NET FX开发中，我们则倾向于使用web.config文件，通过配置appsetting的配置节来处理；而在.NET Core开发中，我们有了新的基于json格式的appsetting.json文件。

无论采用哪种方式，其实配置管理从来都是一件看起来简单，但影响非常深远的基础性工作。尤其是配置的安全性，贯穿应用程序的始终，如果没能做好安全性问题，极有可能会给系统带来不可控的风向。

#### 源代码比配置文件安全么？

有人以为把配置存放在源代码中，可能比存放在明文的配置文件中似乎更安全，其实是“皇帝的新装”。

在前不久，笔者的一位朋友就跟我说了一段故事：他说一位同事在离职后，直接将曾经写过的一段代码上传到github的公共仓库，而这段代码中包含了某些涉及到原企业的机密数据，还好被github的安全机制提前发现而及时终止了该行为，否则后果不堪设想。

于是，笔者顺手查了一下由于有意或无意泄露企业机密，造成企业损失的案例，发现还真不少。例如[大疆前员工通过 Github 泄露公司源代码，被罚 20 万、获刑半年](https://www.infoq.cn/article/RZzfel1m6-h8pSK8TTC9)  这起案件，也是一个典型的案例。

该员工离职后，将包含关键配置信息的源代码上传到github的公共仓库，被黑客利用，使得大量用户私人数据被黑客获取，该前员工最终被刑拘。 ![大疆前员工通过Github泄露公司源代码，被罚20万、获刑半年](https://static.geekbang.org/infoq/5cc32361f3500.png?imageView2/0/w/800) 

 图片来源：[ http://www.digitalmunition.com/WhyIWalkedFrom3k.pdf](http://www.digitalmunition.com/WhyIWalkedFrom3k.pdf) 

大部分IT公司都会在入职前进行背景调查，而一旦有案底，可能就已经与许多IT公司无缘；即便是成为创业者，也可能面临无法跟很多正规企业合作的问题。

#### 小结

所以，安全性问题不容小觑，哪怕时间再忙，也不要急匆匆的就将数据库连接字符串或其他包含敏感信息的内容轻易的记录在源代码或配置文件中。在这个点上，一旦出现问题，往往都是非常严重的问题。

## 如何实现

在.NET FX时代，我们可以使用对web.config文件的关键配置节进行加密的方式，来保护我们的敏感信息，在.NET Core中，自然也有这些东西，接下来我将简述在开发环境和生产环境下不同的配置加密手段，希望能够给读者带来启迪。

#### 开发环境

在开发环境下，我们可以使用visual studio 工具提供的用户机密管理器，只需0行代码，即可轻松完成关键配置节的处理。

##### 机密管理器概述

根据[微软官方文档]( https://docs.microsoft.com/zh-cn/dotnet/architecture/microservices/secure-net-microservices-web-applications/developer-app-secrets-storage ) 的描述：

> ASP.NET Core [机密管理器](https://docs.microsoft.com/zh-cn/aspnet/core/security/app-secrets#secret-manager)工具提供了开发过程中在源代码外部保存机密的另一种方法 。 若要使用机密管理器工具，请在项目文件中安装包 Microsoft.Extensions.Configuration.SecretManager 。 如果该依赖项存在并且已还原，则可以使用 `dotnet user-secrets` 命令来通过命令行设置机密的值。 这些机密将存储在用户配置文件目录中的 JSON 文件中（详细信息随操作系统而异），与源代码无关。
>
> 机密管理器工具设置的机密是由使用机密的项目的 `UserSecretsId` 属性组织的。 因此，必须确保在项目文件中设置 UserSecretsId 属性，如下面的代码片段所示。 默认值是 Visual Studio 分配的 GUID，但实际字符串并不重要，只要它在计算机中是唯一的。
>
> ```XML
> <PropertyGroup>
>    <UserSecretsId>UniqueIdentifyingString</UserSecretsId>
> </PropertyGroup> 
> ```

Secret Manager工具允许开发人员在开发ASP.NET Core应用程序期间存储和检索敏感数据。敏感数据存储在与应用程序源代码不同的位置。由于Secret Manager将秘密与源代码分开存储，因此敏感数据不会提交到源代码存储库。但机密管理器不会对存储的敏感数据进行加密，因此不应将其视为可信存储。敏感数据作为键值对存储在JSON文件中。最好**不要**在开发和测试环境中使用生产机密。[查看引文]( https://nvisium.com/blog/2019/05/02/Dev-Secrets-and-the-ASP-NET-Core-Secret-Manager.html )。

##### 存放位置

在windows平台下，机密数据的存放位置为：

```JSON
%APPDATA%\Microsoft\UserSecrets\\secrets.json
```

而在Linux/MacOs平台下，机密数据的存放位置为：

```json
 ~/.microsoft/usersecrets/<user_secrets_id>/secrets.json 
```

 在前面的文件路径中， ``将替换`UserSecretsId`为 *.csproj*文件中指定的值。 

##### 在Windows环境下使用机密管理器

在windows下，如果使用Visual Studio2019作为主力开发环境，只需在项目右键单击，选择菜单【管理用户机密】，即可添加用户机密数据。   

![](/images/how-to-manage-user-secret-in-develop-and-production_1.png)


在管理用户机密数据中，添加的配置信息和传统的配置信息没有任何区别。

> {
>   "ConnectionStrings": {
>     "Default": "Server=xxx;Database=xxx;User ID=xxx;Password=xxx;"
>   }
> }

我们同样也可以使用IConfiguration的方式、IOptions<T>的方式，进行配置的访问。

##### 在非Windows/非Visual Studio环境下使用机密管理器

完成安装dotnet-cli后，在控制台输入 

```dotnet-cli
dotnet user-secrets init 
```

 前面的命令将在`UserSecretsId` .csproj 文件的`PropertyGroup`中添加 *.csproj*一个元素。 `UserSecretsId`是对项目是唯一的Guid值。 

```xml
 <PropertyGroup>  
 	<TargetFramework>netcoreapp3.1</TargetFramework>
    <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId> 
 </PropertyGroup> 
```

设置机密

```dotnet-cli
 dotnet user-secrets set "Movies:ServiceApiKey" "12345" 
```

列出机密

```dotnet-cli
 dotnet user-secrets list 
```

删除机密

```dotnet-cli
 dotnet user-secrets remove "Movies:ConnectionString" 
```

清除所有机密

```dotnet-cli
 dotnet user-secrets clear 
```

#### 生产环境

机密管理器为开发者在开发环境下提供了一种保留机密数据的方法，但在开发环境下是不建议使用的，如果想在生产环境下，对机密数据进行保存该怎么办？

按照微软官方文档的说法，推荐使用[Azure Key Vault ]( https://docs.microsoft.com/zh-cn/dotnet/architecture/microservices/secure-net-microservices-web-applications/azure-key-vault-protects-secrets ) 来保护机密数据，但。。我不是贵云的用户（当然，买不起贵云不是贵云太贵，而是我个人的问题[手动狗头]）。

其次，与Azure Key Valut类似的套件，例如其他云，差不多都有，所以都可以为我们所用。

但。。如果您如果跟我一样，不想通过第三方依赖的形式来解决这个问题，那不如就用最简单的办法，例如AES加密。

##### 使用AES加密配置节 

该方法与平时使用AES对字符串进行加密和解密的方法并无区别，此处从略。  

##### 使用数据保护Api（DataProtect Api实现）

在平时开发过程中，能够动手撸AES加密是一种非常好的习惯，而微软官方提供的数据保护API则将这个过程进一步简化，只需调Api即可完成相应的数据加密操作。

关于数据保护api， [Savorboard](https://home.cnblogs.com/u/savorboard/) 大佬曾经写过3篇博客讨论这个技术问题，大家可以参考下面的文章来获取信息。

[ASP.NET Core 数据保护（Data Protection 集群场景）【上】]( https://www.cnblogs.com/savorboard/p/dotnetcore-data-protection.html )

[ASP.NET Core 数据保护（Data Protection 集群场景）【中】]( https://www.cnblogs.com/savorboard/p/dotnet-core-data-protection.html )

[ASP.NET Core 数据保护（Data Protection 集群场景）【下】]( https://www.cnblogs.com/savorboard/p/dotnetcore-data-protected-farm.html )

(接下来我要贴代码了，如果没兴趣，请出门左拐，代码不能完整运行，[查看代码]( https://stackoverflow.com/questions/36062670/encrypted-configuration-in-asp-net-core )）

首先，注入配置项

```C#
 public static IServiceCollection AddProtectedConfiguration(this IServiceCollection services, string directory)
        {
            services
                .AddDataProtection()
                .PersistKeysToFileSystem(new DirectoryInfo(directory))
                .UseCustomCryptographicAlgorithms(new ManagedAuthenticatedEncryptorConfiguration
                {
                    EncryptionAlgorithmType = typeof(Aes),
                    EncryptionAlgorithmKeySize = 256,
                    ValidationAlgorithmType = typeof(HMACSHA256)
                });
            ;

            return services;
        }
```

其次，实现对配置节的加/解密。（使用AES算法的数据保护机制）

```c#

public class ProtectedConfigurationSection : IConfigurationSection
    {
        private readonly IDataProtectionProvider _dataProtectionProvider;
        private readonly IConfigurationSection _section;
        private readonly Lazy<IDataProtector> _protector;

        public ProtectedConfigurationSection(
            IDataProtectionProvider dataProtectionProvider,
            IConfigurationSection section)
        {
            _dataProtectionProvider = dataProtectionProvider;
            _section = section;

            _protector = new Lazy<IDataProtector>(() => dataProtectionProvider.CreateProtector(section.Path));
        }

        public IConfigurationSection GetSection(string key)
        {
            return new ProtectedConfigurationSection(_dataProtectionProvider, _section.GetSection(key));
        }

        public IEnumerable<IConfigurationSection> GetChildren()
        {
            return _section.GetChildren()
                .Select(x => new ProtectedConfigurationSection(_dataProtectionProvider, x));
        }

        public IChangeToken GetReloadToken()
        {
            return _section.GetReloadToken();
        }

        public string this[string key]
        {
            get => GetProtectedValue(_section[key]);
            set => _section[key] = _protector.Value.Protect(value);
        }

        public string Key => _section.Key;
        public string Path => _section.Path;

        public string Value
        {
            get => GetProtectedValue(_section.Value);
            set => _section.Value = _protector.Value.Protect(value);
        }

        private string GetProtectedValue(string value)
        {
            if (value == null)
                return null;

            return _protector.Value.Unprotect(value);
        }
    }
```

再次，在使用前，先将待加密的字符串转换成BASE64纯文本，然后再使用数据保护API对数据进行处理，得到处理后的字符串。

```c#
private readonly IDataProtectionProvider _dataProtectorTokenProvider;
public TokenAuthController( IDataProtectionProvider dataProtectorTokenProvider)
{
}
[Route("encrypt"), HttpGet, HttpPost]
public string Encrypt(string section, string value)
{
     var protector = _dataProtectorTokenProvider.CreateProtector(section);
     return protector.Protect(value);
}
```

再替换配置文件中的对应内容。

```json
{
  "ConnectionStrings": {
    "Default": "此处是加密后的字符串"
  }
}
```

然后我们就可以按照平时获取IOptions<ConnectStrings>的方式来获取了。

## 问题

公众号【DotNET骚操作】号主【周杰】同学提出以下观点：

**1、在生产环境下，使用AES加密，其实依然是一种不够安全的行为，充其量也就能忽悠下产品经理，毕竟几条简单的语句，就能把机密数据dump出来。**

 也许在这种情况下，我们应该优先考虑accessKeyId/accessSecret，尽量通过设置多级子账号，通过授权Api的机制来管理机密数据，而不是直接暴露类似于数据库连接字符串这样的关键配置信息。另外，应该定期更换数据库的密码，尽量将类似的问题可能造成的风险降到最低。数据保护api也提供的类似的机制，使得开发者能够轻松的管理机密数据的时效性问题。

**2、配置文件放到CI/CD中，发布的时候在CI/CD中进行组装，然后运维只是负责管理CI/CD的账户信息，而最高机密数据，则由其他人负责配置。**

嗯，我完全同意他的第二种做法，另外考虑到由于运维同样有可能会有意无意泄露机密数据，所以如果再给运维配备一本《刑法》，并让他日常补习【侵犯商业秘密罪】相关条款，这个流程就更加闭环了。

## 结语

本文简述了在.NET Core中，如何在开发环境下使用用户机密管理器、在生产环境下使用AES+IDataProvider的方式来保护我们的用户敏感数据。由于时间仓促，如有考虑不周之处，还请各位大佬批评指正。