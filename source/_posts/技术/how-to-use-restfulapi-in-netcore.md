---
title:  WPF学习路线概述
date: 2020-02-23 16:28
tags: 技术
author: 一步一步的构建整洁、可维护的RESTful APIs
categories:
  - 技术
---
译者荐语：利用周末的时间，本人拜读了长沙.NET技术社区翻译的技术文章《[微软RESTFul API指南](http://techq.club/2019/08/02/%E6%8A%80%E6%9C%AF/Microsoft%20REST%20API%E6%8C%87%E5%8D%97/)》，打算按照步骤写一个完整的教程，后来无意中看到了这篇文章，与我要写的主题有不少相似之处，特意翻译下来。前方高能。

![图片](https://uploader.shimo.im/f/U68C9NmcWwwWHATD.png!thumbnail)一步一步的构建整洁、可维护的RESTful APIs

[查看原文](https://www.techq.xyz/2020/02/23/%E6%8A%80%E6%9C%AF/WPF%E5%AD%A6%E4%B9%A0%E8%B7%AF%E7%BA%BF%E6%A6%82%E8%BF%B0/)

# 总览
RESTful不是一个新名词。它是一种架构风格，这种架构风格使用Web服务从客户端应用程序接收数据和向客户端应用程序发送数据。其目标是集中不同客户端应用程序将使用的数据。

选择正确的工具来编写RESTful服务至关重要，因为我们需要关注可伸缩性，维护，文档以及所有其他相关方面。在[ASP.NET](https://docs.microsoft.com/en-us/aspnet/) Core为我们提供了一个功能强大、易于使用的API，使用这些API将很好的实现这个目标。

在本文中，我将向您展示如何使用ASP.NET Core框架为“几乎”现实世界的场景编写结构良好的RESTful API。我将详细介绍常见的模式和策略以简化开发过程。

我还将向您展示如何集成通用框架和库，例如[Entity Framework Core](https://docs.microsoft.com/en-us/ef/core/)和[AutoMapper](https://automapper.org/)，以提供必要的功能。

# **先决条件**
我希望您了解面向对象的编程概念。

即使我将介绍[C＃编程语言](https://docs.microsoft.com/en-us/dotnet/csharp/)的许多细节，我还是建议您具有该主题的基本知识。

我还假设您知道什么是REST，[HTTP协议](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)如何工作，什么是API端点以及什么是[JSON](https://www.json.org/)。[这是](https://medium.freecodecamp.org/restful-services-part-i-http-in-a-nutshell-aab3bfedd131)关于此主题[的出色的入门教程](https://medium.freecodecamp.org/restful-services-part-i-http-in-a-nutshell-aab3bfedd131)。最后，您需要了解关系数据库的工作原理。

要与我一起编码，您将必须安装[.NET Core 2.2](https://dotnet.microsoft.com/download)以及[Postman](https://www.getpostman.com/)（我将用来测试API的工具）。我建议您使用诸如[Visual Studio Code之](https://code.visualstudio.com/)类的代码编辑器来开发API。选择您喜欢的代码编辑器。如果选择Visual Studio Code作为您的代码编辑器，建议您安装[C＃扩展](https://code.visualstudio.com/docs/languages/csharp)以更好地突出显示代码。

您可以在本文末尾找到该API的Github的链接，以检查最终结果。

# **范围**
让我们为一家超市编写一个虚构的Web API。假设我们必须实现以下范围：

* *创建一个RESTful服务，该服务允许客户端应用程序管理超市的产品目录。它需要公开端点以创建，读取，编辑和删除产品类别，例如乳制品和化妆品，还需要管理这些类别的产品。*
* *对于类别，我们需要存储其名称。对于产品，我们需要存储其名称，度量单位（例如，按重量测量的产品为KG），包装中的数量（例如，如果一包饼干是10，则为10）及其各自的类别。*

为了简化示例，我将不处理库存产品，产品运输，安全性和任何其他功能。这个范围足以向您展示ASP.NET Core的工作方式。

要开发此服务，我们基本上需要两个API 端点（译者注：指控制器）：一个用于管理类别，一个用于管理产品。在JSON通讯方面，我们可以认为响应如下：

```
API endpoint: /api/categories
JSON Response (for GET requests):
{
  [
    { "id": 1, "name": "Fruits and Vegetables" },
    { "id": 2, "name": "Breads" },
    … // Other categories
  ]
}
```
```
API endpoint: /api/products
JSON Response (for GET requests):
{
  [
    {
      "id": 1,
      "name": "Sugar",
      "quantityInPackage": 1,
      "unitOfMeasurement": "KG"
      "category": {
        "id": 3,
        "name": "Sugar"
      }
    },
    … // Other products
  ]
}
```
让我们开始编写应用程序。
# **第1步-创建API**
首先，我们必须为Web服务创建文件夹结构，然后我们必须使用[.NET CLI工具](https://docs.microsoft.com/en-us/dotnet/core/tools/?tabs=netcore2x)来构建基本的Web API。打开终端或命令提示符（取决于您使用的操作系统），并依次键入以下命令：

```
mkdir src/Supermarket.API

cd src/Supermarket.API

dotnet new webapi
```
前两个命令只是为API创建一个新目录，然后将当前位置更改为新文件夹。最后一个遵循Web API模板生成一个新项目，这是我们正在开发的应用程序。您可以阅读有关这些命令和其他项目模板的更多信息，并可以通过[检查此链接](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-new?tabs=netcore21)来生成其他项目模板。
现在，新目录将具有以下结构：

![图片](https://uploader.shimo.im/f/LuICwkhqHDsD5RcN.png!thumbnail)

项目结构

## 结构概述
ASP.NET Core应用程序由在类中配置的一组[中间件](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-2.2)（应用程序流水线中的小块应用程序，用于处理请求和响应）组成Startup。如果您以前已经使用过[Express.js](https://expressjs.com/)之类的框架，那么这个概念对您来说并不是什么新鲜事物。

```
public class Startup
	{
	    public Startup(IConfiguration configuration)
	    {
	        Configuration = configuration;
	    }
	

	    public IConfiguration Configuration { get; }
	

	    // This method gets called by the runtime. Use this method to add services to the container.
	    public void ConfigureServices(IServiceCollection services)
	    {
	        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
	    }
	

	    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
	    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
	    {
	        if (env.IsDevelopment())
	        {
	            app.UseDeveloperExceptionPage();
	        }
	        else
	        {
	            // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
	            app.UseHsts();
	        }
	

	        app.UseHttpsRedirection();
	        app.UseMvc();
	    }
	}
```
当应用程序启动时，将调用类中的Main** **方法Program。它使用启动配置创建默认的Web主机，通过HTTP通过特定端口（默认情况下，HTTP为5000，HTTPS为5001）公开应用程序。
```
namespace Supermarket.API
	{
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
	}
```
看一下文件夹中的ValuesController类Controllers。它公开了API通过路由接收请求时将调用的方法/api/values。
```
[Route("api/[controller]")]
	[ApiController]
	public class ValuesController : ControllerBase
	{
	    // GET api/values
	    [HttpGet]
	    public ActionResult<IEnumerable<string>> Get()
	    {
	        return new string[] { "value1", "value2" };
	    }
	

	    // GET api/values/5
	    [HttpGet("{id}")]
	    public ActionResult<string> Get(int id)
	    {
	        return "value";
	    }
	

	    // POST api/values
	    [HttpPost]
	    public void Post([FromBody] string value)
	    { 
	    }
	

	    // PUT api/values/5
	    [HttpPut("{id}")]
	    public void Put(int id, [FromBody] string value)
	    {   
	    }
	

	    // DELETE api/values/5
	    [HttpDelete("{id}")]
	    public void Delete(int id)
	    {  
	    }
	}
```
如果您不了解此代码的某些部分，请不要担心。在开发必要的API端点时，我将详细介绍每一个。现在，只需删除此类，因为我们不会使用它。
# **第2步-创建领域模型**
我将应用一些设计概念，以使应用程序简单易维护。

编写可以由您自己理解和维护的代码并不难，但是您必须牢记您将成为团队的一部分。如果您不注意如何编写代码，那么结果将是一个庞然大物，这将使您和您的团队成员头痛不已。听起来很极端吧？但是相信我，这就是事实。

![图片](https://uploader.shimo.im/f/vMGZaW8zLiA39wxb.png!thumbnail)

衡量好代码的标准是WTF的频率。原图来自[smitty42](https://www.flickr.com/photos/smitty/)，发表于[filckr](https://www.flickr.com/photos/smitty/2245445147)。该图遵循CC-BY-2.0。

在Supermarket.API目录中，创建一个名为的新文件夹Domain。在新的领域文件夹中，创建另一个名为的文件夹Models。我们必须添加到此文件夹的第一个模型是Category。最初，它将是一个简单的[Plain Old CLR Object（POCO）](https://en.wikipedia.org/wiki/Plain_old_CLR_object)类。这意味着该类将仅具有描述其基本信息的属性。

```
using System.Collections.Generic;
	

	namespace Supermarket.API.Domain.Models
	{
	    public class Category
	    {
	        public int Id { get; set; }
	        public string Name { get; set; }
	        public IList<Product> Products { get; set; } = new List<Product>();
	    }
	}
```
该类具有一个Id** **属性（用于标识类别）和一个Name属性。以及一个Products** **属性。最后一个属性将由**Entity Framework Core使用**，大多数ASP.NET Core应用程序使用ORM将数据持久化到数据库中，以映射类别和产品之间的关系。由于类别具有许多相关产品，因此在面向对象的编程方面也具有合理的思维能力。
我们还必须创建产品模型。在同一文件夹中，添加一个新Product类。

```
namespace Supermarket.API.Domain.Models
	{
	    public class Product
	    {
	        public int Id { get; set; }
	        public string Name { get; set; }
	        public short QuantityInPackage { get; set; }
	        public EUnitOfMeasurement UnitOfMeasurement { get; set; }
	

	        public int CategoryId { get; set; }
	        public Category Category { get; set; }
	    }
	}
```
该产品还具有ID和名称的属性。属性QuantityInPackage，它告诉我们一包中有多少个产品单位（请记住应用范围的饼干示例）和一个UnitOfMeasurement** **属性，这是表示一个[枚举类型](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/enum)，它表示可能的度量单位的枚举。最后两个属性，CategoryId** **和Category将由ORM用于映射的产品和类别之间的关系。它表明一种产品只有一个类别。

让我们定义领域模型的最后一部分，EUnitOfMeasurement** **枚举。

按照惯例，枚举不需要在名称前以*“ E”*开头，但是在某些库和框架中，您会发现此前缀是将枚举与接口和类区分开的一种方式。

```
using System.ComponentModel;
	

	namespace Supermarket.API.Domain.Models
	{
	    public enum EUnitOfMeasurement : byte
	    {
	        [Description("UN")]
	        Unity = 1,
	

	        [Description("MG")]
	        Milligram = 2,
	

	        [Description("G")]
	        Gram = 3,
	

	        [Description("KG")]
	        Kilogram = 4,
	

	        [Description("L")]
	        Liter = 5
	    }
	}
```
该代码非常简单。在这里，我们仅定义了几种度量单位的可能性，但是，在实际的超市系统中，您可能具有许多其他度量单位，并且可能还有一个单独的模型。
注意，【Description】特性应用于所有枚举可能性。特性是一种在C＃语言的类，接口，属性和其他组件上定义元数据的方法。在这种情况下，我们将使用它来简化产品API端点的响应，但是您现在不必关心它。我们待会再回到这里。

我们的基本模型已准备就绪，可以使用。现在，我们可以开始编写将管理所有类别的API端点。

# **第3步-类别API**
在Controllers文件夹中，添加一个名为的新类CategoriesController。

按照惯例，该文件夹中所有后缀为*“ Controller”的类*都将成为我们应用程序的控制器。这意味着他们将处理请求和响应。您必须从[命名空间](https://docs.microsoft.com/pt-br/dotnet/csharp/language-reference/keywords/namespace)【Microsoft.AspNetCore.Mvc】继承Controller。

命名空间由一组相关的类，接口，枚举和结构组成。您可以将其视为类似于Java语言[模块](https://medium.freecodecamp.org/javascript-modules-a-beginner-s-guide-783f7d7a5fcc)或Java [程序包](https://docs.oracle.com/javase/tutorial/java/package/packages.html)的东西。

新的控制器应通过路由/api/categories做出响应。我们通过Route** **在类名称上方添加属性，指定占位符来实现此目的，该占位符表示路由应按照惯例使用不带控制器后缀的类名称。

```
using Microsoft.AspNetCore.Mvc;
	

	namespace Supermarket.API.Controllers
	{
	    [Route("/api/[controller]")]
	    public class CategoriesController : Controller
	    {
	    }
	}
```
让我们开始处理GET请求。首先，当有人/api/categories通过GET动词请求数据时，API需要返回所有类别。为此，我们可以创建**类别服务**。
从概念上讲，服务基本上是定义用于处理某些业务逻辑的方法的类或接口。创建用于处理业务逻辑的服务是许多不同编程语言的一种常见做法，例如[身份验证和授权](https://medium.com/@evandro.ggomes/json-web-token-authentication-with-asp-net-core-2-0-b074b0cfc870)，付款，复杂的数据流，缓存和需要其他服务或模型之间进行某些交互的任务。

使用服务，我们可以将请求和响应处理与完成任务所需的真实逻辑隔离开来。

该服务，我们要创建将首先定义一个单独的行为**，**或**方法**：一个list方法。我们希望该方法返回数据库中所有现有的类别。

为简单起见，在这篇博客中，我们将不处理数据分页或过滤，（译者注：基于RESTFul规范，提供了一套完整的分页和过滤的规则）。将来，我将写一篇文章，展示如何轻松处理这些功能。

为了定义C＃（以及其他面向对象的语言，例如Java）中某事物的预期行为，我们定义一个**interface**。一个接口告诉某些事情应该如何工作，但是**没有实现行为的真实逻辑**。逻辑在实现接口的类中实现。如果您不清楚此概念，请不要担心。一段时间后您将了解它。

在Domain文件夹中，创建一个名为的新目录Services。在此添加一个名为ICategoryService的接口。按照惯例，所有接口都应以C＃中的大写字母*“ I”*开头。定义接口代码，如下所示：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Supermarket.API.Domain.Models;
	

	namespace Supermarket.API.Domain.Services
	{
	    public interface ICategoryService
	    {
	         Task<IEnumerable<Category>> ListAsync();
	    }
	}
```
该ListAsync方法的实现必须**异步**返回类别的可枚举对象。
Task封装返回的类表示异步。由于必须等待数据库完成操作才能返回数据，因此我们需要考虑执行此过程可能需要一段时间，因此我们需要使用异步方法。另请注意*“Async”*后缀。这是一个约定，告诉我们的方法应异步执行。

我们有很多约定，对吗？我个人喜欢它，因为它使应用程序易于阅读，即使你在一家使用.NET技术的公司是新人。

![图片](https://uploader.shimo.im/f/vQLzU6MpZkoChf0Y.png!thumbnail)

*“-好的，我们定义了此接口，但是它什么也没做。有什么用？”*

如果您来自Javascript或其他非强类型语言，则此概念可能看起来很奇怪。

接口使我们能够从实际实现中抽象出所需的行为。使用称为[依赖注入](https://medium.freecodecamp.org/a-quick-intro-to-dependency-injection-what-it-is-and-when-to-use-it-7578c84fa88f)的机制，我们可以实现这些接口并将它们与其他组件隔离。

基本上，当您使用依赖项注入时，您可以使用接口定义一些行为。然后，创建一个实现该接口的类。最后，将引用从接口绑定到您创建的类。

*”-听起来确实令人困惑。我们不能简单地创建一个为我们做这些事情的类吗？”*

让我们继续实现我们的API，您将了解为什么使用这种方法。

更改CategoriesController代码，如下所示：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Microsoft.AspNetCore.Mvc;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Domain.Services;
	

	namespace Supermarket.API.Controllers
	{
	    [Route("/api/[controller]")]
	    public class CategoriesController : Controller
	    {
	        private readonly ICategoryService _categoryService;
	        
	        public CategoriesController(ICategoryService categoryService)
	        {
	            _categoryService = categoryService;   
	        }
	

	        [HttpGet]
	        public async Task<IEnumerable<Category>> GetAllAsync()
	        {
	            var categories = await _categoryService.ListAsync();
	            return categories;
	        }
	    }
	}
```
我已经为控制器定义了一个构造函数（当创建一个类的新实例时会调用一个构造函数），并且它接收的实例ICategoryService。这意味着实例可以是任何实现服务接口的实例。我将此实例存储在一个私有的只读字段中_categoryService。我们将使用此字段访问类别服务实现的方法。
顺便说一下，下划线前缀是表示字段的另一个通用约定。特别地，[.NET](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/general-naming-conventions)的[官方命名约定指南](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/general-naming-conventions)不建议使用此[约定](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/general-naming-conventions)，但是这是一种非常普遍的做法，可以避免使用*“ this”*关键字来区分类字段和局部变量。我个人认为阅读起来要干净得多，并且许多框架和库都使用此约定。

在构造函数下，我定义了用于处理请求的方法/api/categories。该HttpGet** **属性告诉ASP.NET Core管道使用该属性来处理GET请求（可以省略此属性，但是最好编写它以便于阅读）。

该方法使用我们的CategoryService实例列出所有类别，然后将类别返回给客户端。框架管道将数据序列化为JSON对象。IEnumerable类型告诉框架，我们想要返回一个类别的枚举，而Task类型(使用async关键字修饰)告诉管道，这个方法应该异步执行。最后，当我们定义一个异步方法时，我们必须使用await关键字来处理需要一些时间的任务。

好的，我们定义了API的初始结构。现在，有必要真正实现类别服务。

# **步骤4-实现类别服务**
在API的根文件夹（即Supermarket.API文件夹）中，创建一个名为的新文件夹Services。在这里，我们将放置所有服务实现。在新文件夹中，添加一个名为CategoryService的新类。更改代码，如下所示：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Domain.Services;
	

	namespace Supermarket.API.Services
	{
	    public class CategoryService : ICategoryService
	    {
	        public async Task<IEnumerable<Category>> ListAsync()
	        {
	        }
	    }
	}
```
以上只是接口实现的基本代码，我们暂时仍不处理任何逻辑。让我们考虑一下列表方法应该如何实现。
我们需要访问数据库并返回所有类别，然后我们需要将此数据返回给客户端。

服务类不是应该处理数据访问的类。我们将使用一种称为“仓储模式”的设计模式，定义仓储类，用于管理数据库中的数据。

在使用仓储模式时，我们定义了repository 类，该类基本上封装了处理数据访问的所有逻辑。这些仓储类使方法可以列出，创建，编辑和删除给定模型的对象，与操作集合的方式相同。在内部，这些方法与数据库对话以执行CRUD操作，从而将数据库访问与应用程序的其余部分隔离开。

我们的服务需要调用类别仓储，以获取列表对象。

从概念上讲，服务可以与一个或多个仓储或其他服务“对话”以执行操作。

创建用于处理数据访问逻辑的新定义似乎是多余的，但是您将在一段时间内看到将这种逻辑与服务类隔离是非常有利的。

让我们创建一个仓储，该仓储负责与数据库通信，作为持久化保存类别的一种方式。

# **步骤5-类别仓储和持久层**
在该Domain文件夹内，创建一个名为的新目录Repositories。然后，添加一个名为的新接口ICategoryRespository。定义接口如下：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Supermarket.API.Domain.Models;
	namespace Supermarket.API.Domain.Repositories
	{
	    public interface ICategoryRepository
	    {
	         Task<IEnumerable<Category>> ListAsync();
	    }
	}
```
初始代码基本上与服务接口的代码相同。
定义了接口之后，我们可以返回服务类并使用的实例ICategoryRepository返回数据来完成实现list方法。

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Domain.Repositories;
	using Supermarket.API.Domain.Services;
	

	namespace Supermarket.API.Services
	{
	    public class CategoryService : ICategoryService
	    {
	        private readonly ICategoryRepository _categoryRepository;
	

	        public CategoryService(ICategoryRepository categoryRepository)
	        {
	            this._categoryRepository = categoryRepository;
	        }
	

	        public async Task<IEnumerable<Category>> ListAsync()
	        { 
	            return await _categoryRepository.ListAsync();
	        }
	    }
	}
```
现在，我们必须实现类别仓储的真实逻辑。在这样做之前，我们必须考虑如何访问数据库。
*顺便说一句，我们仍然没有数据库！*

我们将使用Entity Framework Core（为简单起见，我将其称为***EF Core***）作为我们的数据库ORM。该框架是ASP.NET Core的默认ORM，并公开了一个友好的API，该API使我们能够将应用程序的类映射到数据库表。

EF Core还允许我们先设计应用程序，然后根据我们在代码中定义的内容生成数据库。此技术称为**Code First**。我们将使用Code First方法来生成数据库（实际上，在此示例中，我将使用内存数据库，但是您可以轻松地将其更改为像SQL Server或MySQL服务器这样的实例数据库）。

在API的根文件夹中，创建一个名为的新目录Persistence。此目录将包含我们访问数据库所需的所有内容，例如仓储实现。

在新文件夹中，创建一个名为的新目录Contexts，然后添加一个名为的新类AppDbContext。此类必须继承DbContext，EF Core通过DBContext用来将您的模型映射到数据库表的类。通过以下方式更改代码：

```
using Microsoft.EntityFrameworkCore;
	

	namespace Supermarket.API.Domain.Persistence.Contexts
	{
	    public class AppDbContext : DbContext
	    {
	        public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
	        {
	        }
	    }
	}
```
我们添加到此类的构造函数负责通过依赖注入将数据库配置传递给基类。稍后您将看到其工作原理。
现在，我们必须创建两个DbSet属性。这些属性是将模型映射到数据库表的集合（唯一对象的集合）。

另外，我们必须将模型的属性映射到相应的列，指定哪些属性是主键，哪些是外键，列类型等。我们可以使用称为[Fluent API](http://www.entityframeworktutorial.net/efcore/fluent-api-in-entity-framework-core.aspx)的功能来覆盖OnModelCreating方法，以指定数据库映射。更改AppDbContext类，如下所示：

该代码是如此直观。

```
using Microsoft.EntityFrameworkCore;
	using Supermarket.API.Domain.Models;
	

	namespace Supermarket.API.Persistence.Contexts
	{
	    public class AppDbContext : DbContext
	    {
	        public DbSet<Category> Categories { get; set; }
	        public DbSet<Product> Products { get; set; }
	

	        public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
	

	        protected override void OnModelCreating(ModelBuilder builder)
	        {
	            base.OnModelCreating(builder);
	            
	            builder.Entity<Category>().ToTable("Categories");
	            builder.Entity<Category>().HasKey(p => p.Id);
	            builder.Entity<Category>().Property(p => p.Id).IsRequired().ValueGeneratedOnAdd();
	            builder.Entity<Category>().Property(p => p.Name).IsRequired().HasMaxLength(30);
	            builder.Entity<Category>().HasMany(p => p.Products).WithOne(p => p.Category).HasForeignKey(p => p.CategoryId);
	

	            builder.Entity<Category>().HasData
	            (
	                new Category { Id = 100, Name = "Fruits and Vegetables" }, // Id set manually due to in-memory provider
	                new Category { Id = 101, Name = "Dairy" }
	            );
	

	            builder.Entity<Product>().ToTable("Products");
	            builder.Entity<Product>().HasKey(p => p.Id);
	            builder.Entity<Product>().Property(p => p.Id).IsRequired().ValueGeneratedOnAdd();
	            builder.Entity<Product>().Property(p => p.Name).IsRequired().HasMaxLength(50);
	            builder.Entity<Product>().Property(p => p.QuantityInPackage).IsRequired();
	            builder.Entity<Product>().Property(p => p.UnitOfMeasurement).IsRequired();
	        }
	    }
	}
```
我们指定我们的模型应映射到哪些表。此外，我们设置了主键，使用该方法HasKey，该表的列，使用Property方法，和一些限制，例如IsRequired，HasMaxLength**，**和ValueGeneratedOnAdd，这些都是使用FluentApi的方式基于Lamada 表达式语法实现的（链式语法）。
看一下下面的代码：

```
builder.Entity<Category>()
       .HasMany(p => p.Products)
       .WithOne(p => p.Category)
       .HasForeignKey(p => p.CategoryId);
```
在这里，我们指定表之间的关系。我们说一个类别有很多产品，我们设置了将映射此关系的属性（Products，来自Category类，和Category，来自Product类）。我们还设置了外键（CategoryId）。
如果您想学习如何使用EF Core配置一对一和多对多关系，以及如何完整的使用它，请看一下[本教程](https://www.learnentityframeworkcore.com/relationships)。

还有一种用于通过HasData方法配置种子数据的方法：

```
builder.Entity<Category>().HasData
(
  new Category { Id = 100, Name = "Fruits and Vegetables" },
  new Category { Id = 101, Name = "Dairy" }
);
```
默认情况下，在这里我们仅添加两个示例类别。这对我们完成后进行API的测试来说是非常有必要的。
>**注意：**我们在Id这里手动设置属性，因为内存提供程序的工作机制需要。我将标识符设置为大数字，以避免自动生成的标识符和种子数据之间发生冲突。
>>真正的关系数据库提供程序中不存在此限制，因此，例如，如果要使用SQL Server等数据库，则不必指定这些标识符。如果您想了解此行为，请检查[此Github问题](https://github.com/aspnet/EntityFrameworkCore/issues/6872)。

在实现数据库上下文类之后，我们可以实现类别仓储。添加一个名为新的文件夹Repositories里面Persistence的文件夹，然后添加一个名为新类BaseRepository。

```
using Supermarket.API.Persistence.Contexts;
	

	namespace Supermarket.API.Persistence.Repositories
	{
	    public abstract class BaseRepository
	    {
	        protected readonly AppDbContext _context;
	

	        public BaseRepository(AppDbContext context)
	        {
	            _context = context;
	        }
	    }
	}
```
此类只是我们所有仓储都将继承的**抽象类**。抽象类是没有直接实例的类。您必须创建直接类来创建实例。
在BaseRepository接受我们的实例，AppDbContext通过依赖注入暴露了一个受保护的属性称为（只能是由子类访问一个属性）_context，即可以访问我们需要处理数据库操作的所有方法。

在相同文件夹中添加一个新类CategoryRepository。现在，我们将真正实现仓储逻辑：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Microsoft.EntityFrameworkCore;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Domain.Repositories;
	using Supermarket.API.Persistence.Contexts;
	

	namespace Supermarket.API.Persistence.Repositories
	{
	    public class CategoryRepository : BaseRepository, ICategoryRepository
	    {
	        public CategoryRepository(AppDbContext context) : base(context)
	        {
	        }
	

	        public async Task<IEnumerable<Category>> ListAsync()
	        {
	            return await _context.Categories.ToListAsync();
	        }
	    }
	}
```
仓储继承BaseRepository和实现ICategoryRepository。
注意实现list方法是很简单的。我们使用Categories数据库集访问类别表，然后调用扩展方法ToListAsync，该方法负责将查询结果转换为类别的集合。

EF Core [将我们的方法调用转换为SQL查询](https://docs.microsoft.com/en-us/ef/core/querying/overview)，这是最有效的方法。这种方式仅当您调用将数据转换为集合的方法或使用方法获取特定数据时才执行查询。

现在，我们有了类别控制器，服务和仓储库的代码实现。

我们将关注点分离开来，创建了只执行应做的事情的类。

测试应用程序之前的最后一步是使用ASP.NET Core依赖项注入机制将我们的接口绑定到相应的类。

# **第6步-配置依赖注入**
现在是时候让您最终了解此概念的工作原理了。

![图片](https://uploader.shimo.im/f/GYbYOJqzMqsxEBg3.png!thumbnail)

在应用程序的根文件夹中，打开Startup类。此类负责在应用程序启动时配置各种配置。

该ConfigureServices和Configure方法通过框架管道在运行时调用来配置应用程序应该如何工作，必须使用哪些组件。

打开ConfigureServices方法。在这里，我们只有一行配置应用程序以使用MVC管道，这基本上意味着该应用程序将使用控制器类来处理请求和响应（在这段代码背后发生了很多事情，但目前您仅需要知道这些）。

我们可以使用ConfigureServices访问services参数的方法来配置我们的依赖项绑定。清理类代码，删除所有注释并按如下所示更改代码：

```
using Microsoft.AspNetCore.Builder;
	using Microsoft.AspNetCore.Hosting;
	using Microsoft.AspNetCore.Mvc;
	using Microsoft.EntityFrameworkCore;
	using Microsoft.Extensions.Configuration;
	using Microsoft.Extensions.DependencyInjection;
	using Supermarket.API.Domain.Repositories;
	using Supermarket.API.Domain.Services;
	using Supermarket.API.Persistence.Contexts;
	using Supermarket.API.Persistence.Repositories;
	using Supermarket.API.Services;
	

	namespace Supermarket.API
	{
	    public class Startup
	    {
	        public IConfiguration Configuration { get; }
	

	        public Startup(IConfiguration configuration)
	        {
	            Configuration = configuration;
	        }
	

	        public void ConfigureServices(IServiceCollection services)
	        {
	            services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
	

	            services.AddDbContext<AppDbContext>(options => {
	                options.UseInMemoryDatabase("supermarket-api-in-memory");
	            });
	

	            services.AddScoped<ICategoryRepository, CategoryRepository>();
	            services.AddScoped<ICategoryService, CategoryService>();
	        }
	

	        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
	        {
	            if (env.IsDevelopment())
	            {
	                app.UseDeveloperExceptionPage();
	            }
	            else
	            {
	                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
	                app.UseHsts();
	            }
	

	            app.UseHttpsRedirection();
	            app.UseMvc();
	        }
	    }
	}
```
看一下这段代码：
```
services.AddDbContext<AppDbContext>(options => {

  options.UseInMemoryDatabase("supermarket-api-in-memory");
  
});
```
在这里，我们配置数据库上下文。我们告诉ASP.NET Core将其AppDbContext与内存数据库实现一起使用，该实现由作为参数传递给我们方法的字符串标识。通常，在编写[集成测试](https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-2.2)时才会使用内存数据库，但是为了简单起见，我在这里使用了内存数据库。这样，我们无需连接到真实的数据库即可测试应用程序。
这些代码行在内部配置我们的数据库上下文，以便使用确定作用域的生存周期进行依赖注入。

scoped生存周期告诉ASP.NET Core管道，每当它需要解析接收AppDbContext作为构造函数参数的实例的类时，都应使用该类的相同实例。如果内存中没有实例，则管道将创建一个新实例，并在给定请求期间在需要它的所有类中重用它。这样，您无需在需要使用时手动创建类实例。

如果你想了解其他有关生命周期的知识，可以阅读[官方文档](https://docs.microsoft.com/pt-br/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-2.2)。

依赖注入技术为我们提供了许多优势，例如：

* 代码可重用性；
* 更高的生产力，因为当我们不得不更改实现时，我们无需费心去更改您使用该功能的一百个地方；
* 您可以轻松地测试应用程序，因为我们可以使用mock（类的伪实现）隔离必须测试的内容，而我们必须将接口作为构造函数参数进行传递。
* 当一个类需要通过构造函数接收更多的依赖关系时，您不必手动更改正在创建实例的所有位置（**太赞了！**）。

配置数据库上下文之后，我们还将我们的服务和仓储绑定到相应的类。

```
services.AddScoped<ICategoryRepository, CategoryRepository>();

services.AddScoped<ICategoryService, CategoryService>();
```
在这里，我们还使用了scoped生存周期，因为这些类在内部必须使用数据库上下文类。在这种情况下，指定相同的范围是有意义的。
现在我们配置了依赖绑定，我们必须在Program类上进行一些小的更改，以便数据库正确地初始化种子数据。此步骤仅在使用内存数据库提供程序时才需要执行（请参阅[此Github问题](https://github.com/aspnet/EntityFrameworkCore/issues/11666)以了解原因）。

```
using System;
	using System.Collections.Generic;
	using System.IO;
	using System.Linq;
	using System.Threading.Tasks;
	using Microsoft.AspNetCore;
	using Microsoft.AspNetCore.Hosting;
	using Microsoft.Extensions.Configuration;
	using Microsoft.Extensions.DependencyInjection;
	using Microsoft.Extensions.Logging;
	using Supermarket.API.Persistence.Contexts;
	

	namespace Supermarket.API
	{
	    public class Program
	    {
	        public static void Main(string[] args)
	        {
	            var host = BuildWebHost(args);
	

	            using(var scope = host.Services.CreateScope())
	            using(var context = scope.ServiceProvider.GetService<AppDbContext>())
	            {
	                context.Database.EnsureCreated();
	            }
	

	            host.Run();
	        }
	

	        public static IWebHost BuildWebHost(string[] args) =>
	            WebHost.CreateDefaultBuilder(args)
	            .UseStartup<Startup>()
	            .Build();
	    }
	}
```
由于我们使用的是内存提供程序，因此有必要更改Main方法 添加“ context.Database.EnsureCreated();”代码以确保在应用程序启动时将“创建”数据库。没有此更改，将不会创建我们想要的初始化种子数据。
实现了所有基本功能后，就该测试我们的API端点了。

# **第7步-测试类别**
在API根文件夹中打开终端或命令提示符，然后键入以下命令：

```
dotnet run
```
上面的命令启动应用程序。控制台将显示类似于以下内容的输出：
```
info: Microsoft.EntityFrameworkCore.Infrastructure[10403]

Entity Framework Core 2.2.0-rtm-35687 initialized ‘AppDbContext’ using provider ‘Microsoft.EntityFrameworkCore.InMemory’ with options: StoreName=supermarket-api-in-memory

info: Microsoft.EntityFrameworkCore.Update[30100]

Saved 2 entities to in-memory store.

info: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[0]

User profile is available. Using ‘C:\Users\evgomes\AppData\Local\ASP.NET\DataProtection-Keys’ as key repository and Windows DPAPI to encrypt keys at rest.

Hosting environment: Development

Content root path: C:\Users\evgomes\Desktop\Tutorials\src\Supermarket.API

Now listening on: https://localhost:5001

Now listening on: http://localhost:5000

Application started. Press Ctrl+C to shut down.
```
您可以看到调用了EF Core来初始化数据库。最后几行显示应用程序在哪个端口上运行。
打开浏览器，然后导航到 [http](http://localhost:5000/api/categories)[:](http://localhost:5000/api/categories)[//localhost](http://localhost:5000/api/categories)[:](http://localhost:5000/api/categories)[5000/api/categories](http://localhost:5000/api/categories) （或控制台输出上显示的URL）。如果您发现由于HTTPS导致的安全错误，则只需为应用程序添加一个例外。

浏览器将显示以下JSON数据作为输出：

```
[
  {
     "id": 100,
     "name": "Fruits and Vegetables",
     "products": []
  },
  {
     "id": 101,
     "name": "Dairy",
     "products": []
  }
]
```
在这里，我们看到配置数据库上下文时添加到数据库的数据。此输出确认我们的代码正在运行。
您使用很少的代码行创建了GET API端点，并且由于当前API项目的架构模式，您的代码结构确实很容易更改。

现在，该向您展示在由于业务需要而不得不对其进行更改时，更改此代码有多么容易。

# **步骤8-创建类别资源**
如果您还记得API端点的规范，您会注意到我们的实际JSON响应还有一个额外的属性：**products数组**。看一下所需响应的示例：

```
{
  [
    { "id": 1, "name": "Fruits and Vegetables" },
    { "id": 2, "name": "Breads" },
    … // Other categories
  ]
}
```
产品数组出现在我们当前的JSON响应中，因为我们的Category模型具有Products，EF Core需要的属性，以正确映射给定类别的产品。
我们不希望在响应中使用此属性，但是不能更改模型类以排除此属性。当我们尝试管理类别数据时，这将导致EF Core引发错误，并且也将破坏我们的领域模型设计，因为没有产品的产品类别没有意义。

要返回仅包含超级市场类别的标识符和名称的JSON数据，我们必须创建一个**资源类**。

[资源类](https://restful-api-design.readthedocs.io/en/latest/resources.html)是一种包含将客户端应用程序和API端点之间进行交换的类型，通常以JSON数据的形式出现，以表示一些特定信息的类。

来自API端点的所有响应都**必须**返回资源。

将真实模型表示形式作为响应返回是一种不好的做法，因为它可能包含客户端应用程序不需要或没有其权限的信息（例如，用户模型可以返回用户密码的信息） ，这将是一个很大的安全问题）。

我们需要一种资源来仅代表我们的类别，而没有产品。

现在您知道什么是资源，让我们实现它。首先，在命令行中按**Ctrl + C**停止正在运行的应用程序。在应用程序的根文件夹中，创建一个名为Resources的新文件夹。在其中添加一个名为的新类CategoryResource。

```
namespace Supermarket.API.Resources
	{
	    public class CategoryResource
	    {
	        public int Id { get; set; }
	        public string Name { get; set; }
	    }
	}
```
我们必须将类别服务提供的类别模型集合映射到类别资源集合。
我们将使用一个名为[AutoMapper](https://automapper.org/)的库来处理对象之间的映射。AutoMapper是.NET世界中非常流行的库，并且在许多商业和开源项目中使用。

在命令行中输入以下命令，以将AutoMapper添加到我们的应用程序中：

```
dotnet add package AutoMapper

dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```
要使用AutoMapper，我们必须做两件事：
* 注册它以进行依赖注入；
* 创建一个类，该类将告诉AutoMapper如何处理类映射。

首先，打开Startup课程。在该ConfigureServices方法的最后一行之后，添加以下代码：

```
services.AddAutoMapper();
```
此行处理AutoMapper的所有必需配置，例如注册它以进行依赖项注入以及在启动过程中扫描应用程序以配置映射配置文件。
现在，在根目录中，添加一个名为的新文件夹Mapping，然后添加一个名为的类ModelToResourceProfile。通过以下方式更改代码：

```
using AutoMapper;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Resources;
	

	namespace Supermarket.API.Mapping
	{
	    public class ModelToResourceProfile : Profile
	    {
	        public ModelToResourceProfile()
	        {
	            CreateMap<Category, CategoryResource>();
	        }
	    }
	}
```
该类继承Profile了AutoMapper用于检查我们的映射如何工作的类类型。在构造函数上，我们在Category模型类和CategoryResource类之间创建一个映射。由于类的属性具有相同的名称和类型，因此我们不必为其使用任何特殊的配置。
最后一步包括更改类别控制器以使用AutoMapper处理我们的对象映射。

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using AutoMapper;
	using Microsoft.AspNetCore.Mvc;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Domain.Services;
	using Supermarket.API.Resources;
	

	namespace Supermarket.API.Controllers
	{
	    [Route("/api/[controller]")]
	    public class CategoriesController : Controller
	    {
	        private readonly ICategoryService _categoryService;
	        private readonly IMapper _mapper;
	

	        public CategoriesController(ICategoryService categoryService, IMapper mapper)
	        {
	            _categoryService = categoryService;
	            _mapper = mapper;
	        }
	

	        [HttpGet]
	        public async Task<IEnumerable<CategoryResource>> GetAllAsync()
	        {
	            var categories = await _categoryService.ListAsync();
	            var resources = _mapper.Map<IEnumerable<Category>, IEnumerable<CategoryResource>>(categories);
	            
	            return resources;
	        }
	    }
	}
```
我更改了构造函数以接收IMapper实现的实例。您可以使用这些接口方法来使用AutoMapper映射方法。
我还更改了GetAllAsync使用Map方法将类别枚举映射到资源枚举的方法。此方法接收我们要映射的类或集合的实例，并通过[通用类型定义](https://www.geeksforgeeks.org/c-generics-introduction/)定义必须映射到什么类型的类或集合。

注意，我们只需将新的依赖项（IMapper）注入构造函数，就可以轻松地更改实现，而不必修改服务类或仓储。

依赖注入使您的应用程序可维护且易于更改，因为您不必中断所有代码实现即可添加或删除功能。

您可能意识到，不仅控制器类，而且所有接收依赖项的类（包括依赖项本身）都会根据绑定配置自动解析为接收正确的类。

依赖注入如此的Amazing，不是吗？

![图片](https://uploader.shimo.im/f/wGoOlHek0agFA3kQ.png!thumbnail)

现在，使用dotnet run命令再次启动API，然后转到[http](http://localhost:5000/api/categories)[:](http://localhost:5000/api/categories)[//localhost](http://localhost:5000/api/categories)[:](http://localhost:5000/api/categories)[5000/api/categories](http://localhost:5000/api/categories)以查看新的JSON响应。

![图片](https://uploader.shimo.im/f/xb8S1G8qcWQW3MSN.png!thumbnail)

这是您应该看到的响应数据

我们已经有了GET端点。现在，让我们为POST（**创建**）类别创建一个新端点。

# **第9步-创建新类别**
在处理资源创建时，我们必须关心很多事情，例如：

* 数据验证和数据完整性；
* 授权创建资源；
* 错误处理；
* 正在记录。

在本教程中，我不会显示如何处理身份验证和授权，但是您可以阅读[JSON Web令牌身份验证](https://medium.com/@evandro.ggomes/json-web-token-authentication-with-asp-net-core-2-0-b074b0cfc870)教程，了解如何轻松实现这些功能。

另外，有一个非常流行的框架称为**ASP.NET Identity**，该框架提供了有关安全性和用户注册的内置解决方案，您可以在应用程序中使用它们。它包括与EF Core配合使用的提供程序，例如IdentityDbContext可以使用的内置程序。您可以[在此处了解更多信息](https://docs.microsoft.com/en-us/aspnet/identity/overview/getting-started/introduction-to-aspnet-identity)。

让我们编写一个HTTP POST端点，该端点将涵盖其他场景（日志记录除外，它可以根据不同的范围和工具进行更改）。

在创建新端点之前，我们需要一个新资源。此资源会将客户端应用程序发送到此端点的数据（在本例中为类别名称）映射到我们应用程序的类。

由于我们正在创建一个新类别，因此我们还没有ID，这意味着我们需要一种资源来表示仅包含其名称的类别。

在Resources文件夹中，添加一个新类SaveCategoryResource：

```
using System.ComponentModel.DataAnnotations;
	

	namespace Supermarket.API.Resources
	{
	    public class SaveCategoryResource
	    {
	        [Required]
	        [MaxLength(30)]
	        public string Name { get; set; }
	    }
	}
```
注意Name属性上的Required和MaxLength特性。这些属性称为[数据注释](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations?view=netframework-4.7.2)。ASP.NET Core管道使用此元数据来验证请求和响应。顾名思义，类别名称是必填项，最大长度为30个字符。
现在，让我们定义新API端点的形状。将以下代码添加到类别控制器：

```
[HttpPost]
	public async Task<IActionResult> PostAsync([FromBody] SaveCategoryResource resource)
	{
	}
```
我们使用HttpPost特性告诉框架这是一个HTTP POST端点。
注意此方法的响应类型Task<IActionResult>。控制器类中存在的方法称为**动作**，它们具有此签名，因为在应用程序执行动作之后，我们可以返回一个以上的可能结果。

在这种情况下，如果类别名称无效或出现问题，我们必须返回**400代码（错误请求）**响应，该响应通常包含一条错误消息，客户端应用程序可以使用该错误消息来解决该问题，或者我们可以如果一切正常，则对数据进行**200次响应（成功）**。

可以将多种类型的操作类型用作响应，但是通常，我们可以使用此接口，并且ASP.NET Core将为此使用默认类。

该FromBody属性告诉ASP.NET Core将请求正文数据解析为我们的新资源类。这意味着当包含类别名称的JSON发送到我们的应用程序时，框架将自动将其解析为我们的新类。

现在，让我们实现路由逻辑。我们必须遵循一些步骤才能成功创建新类别：

* 首先，我们必须验证传入的请求。如果请求无效，我们必须返回包含错误消息的错误请求响应；
* 然后，如果请求有效，则必须使用AutoMapper将新资源映射到类别模型类。
* 现在，我们需要调用我们的服务，告诉它保存我们的新类别。如果执行保存逻辑没有问题，它将返回一个包含我们新类别数据的响应。如果没有，它应该给我们一个指示，表明该过程失败了，并可能出现错误消息。
* 最后，如果有错误，我们将返回错误的请求。如果没有，我们将新的类别模型映射到类别资源，并向客户端返回包含新类别数据的成功响应。

这似乎很复杂，但是使用为API构建的服务架构来实现此逻辑确实很容易。

让我们开始验证传入的请求。

# **步骤10-使用模型状态验证请求主体**
ASP.NET Core控制器具有名为ModelState的属性。在执行我们的操作**之前，**该属性在请求执行期间填充。它是ModelStateDictionary的实例，该类包含诸如请求是否有效以及潜在的验证错误消息之类的信息。

如下更改端点代码：

```
[HttpPost]
	public async Task<IActionResult> PostAsync([FromBody] SaveCategoryResource resource)
	{
		if (!ModelState.IsValid)
			return BadRequest(ModelState.GetErrorMessages());
	}
```
这段代码检查模型状态（在这种情况下为请求正文中发送的数据）是否无效，并检查我们的数据注释。如果不是，则API返回错误的请求（状态代码400），以及我们的注释元数据提供的默认错误消息。
该ModelState.GetErrorMessages()方法尚未实现。这是一种[扩展方法](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)（一种扩展现有类或接口功能的方法），我将实现该方法将验证错误转换为简单的字符串以返回给客户端。

Extensions在我们的API的根目录中添加一个新文件夹，然后添加一个新类ModelStateExtensions。

```
using System.Collections.Generic;
	using System.Linq;
	using Microsoft.AspNetCore.Mvc.ModelBinding;
	

	namespace Supermarket.API.Extensions
	{
	    public static class ModelStateExtensions
	    {
	        public static List<string> GetErrorMessages(this ModelStateDictionary dictionary)
	        {
	            return dictionary.SelectMany(m => m.Value.Errors)
	                             .Select(m => m.ErrorMessage)
	                             .ToList();
	        }
	    }
	}
```
所有扩展方法以及声明它们的类都应该是**静态的**。** **这意味着它们不处理特定的实例数据，并且在应用程序启动时仅被加载一次。
this参数声明前面的关键字告诉C＃编译器将其视为扩展方法。结果是我们可以像此类的常规方法一样调用它，因为我们在要使用扩展的地方包含的特定的using代码。

该扩展使用[LINQ查询](https://www.tutorialsteacher.com/linq/what-is-linq)，这是.NET的非常有用的功能，它使我们能够使用链式语法来查询和转换数据。此处的表达式将验证错误方法转换为包含错误消息的字符串列表。

Supermarket.API.Extensions在进行下一步之前，将名称空间导入Categories控制器。

```
using Supermarket.API.Extensions;
```
让我们通过将新资源映射到类别模型类来继续实现端点逻辑。
# **步骤11-映射新资源**
我们已经定义了映射配置文件，可以将模型转换为资源。现在，我们需要一个与之相反的新配置项。

ResourceToModelProfile在Mapping文件夹中添加一个新类：

```
using AutoMapper;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Resources;
	

	namespace Supermarket.API.Mapping
	{
	    public class ResourceToModelProfile : Profile
	    {
	        public ResourceToModelProfile()
	        {
	            CreateMap<SaveCategoryResource, Category>();
	        }
	    }
	}
```
这里没有新内容。由于依赖注入的魔力，AutoMapper将在应用程序启动时自动注册此配置文件，而我们无需更改任何其他位置即可使用它。
现在，我们可以将新资源映射到相应的模型类：

```
[HttpPost]
	public async Task<IActionResult> PostAsync([FromBody] SaveCategoryResource resource)
	{
		if (!ModelState.IsValid)
			return BadRequest(ModelState.GetErrorMessages());
	

		var category = _mapper.Map<SaveCategoryResource, Category>(resource);
	}
```
# **第12步-应用请求-响应模式来处理保存逻辑**
现在我们必须实现最有趣的逻辑：保存一个新类别。我们希望我们的服务能够做到。

由于连接到数据库时出现问题，或者由于任何内部业务规则使我们的数据无效，因此保存逻辑可能会失败。

如果出现问题，我们不能简单地抛出一个错误，因为它可能会停止API，并且客户端应用程序也不知道如何处理该问题。另外，我们可能会有某种日志记录机制来记录错误。

保存方法的约定（即方法的签名和响应类型）需要指示我们是否正确执行了该过程。如果处理正常，我们将接收类别数据。如果没有，我们至少必须收到一条错误消息，告诉您该过程失败的原因。

我们可以通过应用**request-response模式**来实现此功能。这种企业设计模式将我们的请求和响应参数封装到类中，以封装我们的服务将用于处理某些任务并将信息返回给正在使用该服务的类的信息。

这种模式为我们提供了一些优势，例如：

* 如果我们需要更改服务以接收更多参数，则不必破坏其签名；
* 我们可以为我们的请求和/或响应定义标准合同；
* 我们可以在不停止应用程序流程的情况下处理业务逻辑和潜在的失败，并且我们不需要使用大量的try-catch块。

让我们为处理数据更改的服务方法创建一个标准响应类型。对于这种类型的每个请求，我们都想知道该请求是否被正确执行。如果失败，我们要向客户端返回错误消息。

在Domain文件夹的内部Services，添加一个名为的新目录Communication。在此处添加一个名为的新类BaseResponse。

```
namespace Supermarket.API.Domain.Services.Communication
	{
	    public abstract class BaseResponse
	    {
	        public bool Success { get; protected set; }
	        public string Message { get; protected set; }
	

	        public BaseResponse(bool success, string message)
	        {
	            Success = success;
	            Message = message;
	        }
	    }
	}
```
那是我们的响应类型将继承的抽象类。
抽象定义了一个Success属性和一个Message属性，该属性将告知请求是否已成功完成，如果失败，该属性将显示错误消息。

请注意，这些属性是必需的，只有继承的类才能设置此数据，因为子类必须通过构造函数传递此信息。

>**提示：**为所有内容定义基类不是一个好习惯，因为[基类会耦合您的代码](https://en.wikipedia.org/wiki/Fragile_base_class)并阻止您轻松对其进行修改。优先使用[组合而不是继承](https://medium.com/humans-create-software/composition-over-inheritance-cb6f88070205)。
>>在此API的范围内，使用基类并不是真正的问题，因为我们的服务不会增长太多。如果您意识到服务或应用程序会经常增长和更改，请避免使用基类。

现在，在同一文件夹中，添加一个名为的新类SaveCategoryResponse。

```
using Supermarket.API.Domain.Models;
	

	namespace Supermarket.API.Domain.Services.Communication
	{
	    public class SaveCategoryResponse : BaseResponse
	    {
	        public Category Category { get; private set; }
	

	        private SaveCategoryResponse(bool success, string message, Category category) : base(success, message)
	        {
	            Category = category;
	        }
	

	        /// <summary>
	        /// Creates a success response.
	        /// </summary>
	        /// <param name="category">Saved category.</param>
	        /// <returns>Response.</returns>
	        public SaveCategoryResponse(Category category) : this(true, string.Empty, category)
	        { }
	

	        /// <summary>
	        /// Creates am error response.
	        /// </summary>
	        /// <param name="message">Error message.</param>
	        /// <returns>Response.</returns>
	        public SaveCategoryResponse(string message) : this(false, message, null)
	        { }
	    }
	}
```
响应类型还设置了一个Category属性，如果请求成功完成，该属性将包含我们的类别数据。
请注意，我为此类定义了三种不同的构造函数：

* 一个私有的，它将把成功和消息参数传递给基类，并设置Category属性。
* 仅接收类别作为参数的构造函数。这将创建一个成功的响应，调用私有构造函数来设置各自的属性；
* 第三个构造函数仅指定消息。这将用于创建故障响应。

因为C＃支持多个构造函数，所以我们仅通过使用不同的构造函数就简化了响应的创建过程，而无需定义其他方法来处理此问题。

现在，我们可以更改服务界面以添加新的保存方法合同。

更改ICategoryService接口，如下所示：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Domain.Services.Communication;
	

	namespace Supermarket.API.Domain.Services
	{
	    public interface ICategoryService
	    {
	         Task<IEnumerable<Category>> ListAsync();
	         Task<SaveCategoryResponse> SaveAsync(Category category);
	    }
	}
```
我们只需将类别传递给此方法，它将处理保存模型数据，编排仓储和其他必要服务所需的所有逻辑。
请注意，由于我们不需要任何其他参数来执行此任务，因此我不在此处创建特定的请求类。[计算机编程中](https://www.techopedia.com/definition/20262/keep-it-simple-stupid-principle-kiss-principle)有一个名为[KISS](https://www.techopedia.com/definition/20262/keep-it-simple-stupid-principle-kiss-principle)的[概念](https://www.techopedia.com/definition/20262/keep-it-simple-stupid-principle-kiss-principle) —Keep It Simple，Stupid的简称。基本上，它说您应该使您的应用程序尽可能简单。

设计应用程序时请记住这一点：**仅**应用**解决问题所需的内容**。**不要过度设计您的应用程序。**

现在我们可以完成端点逻辑：

```
[HttpPost]
	public async Task<IActionResult> PostAsync([FromBody] SaveCategoryResource resource)
	{
		if (!ModelState.IsValid)
			return BadRequest(ModelState.GetErrorMessages());
	

		var category = _mapper.Map<SaveCategoryResource, Category>(resource);
		var result = await _categoryService.SaveAsync(category);
	

		if (!result.Success)
			return BadRequest(result.Message);
	

		var categoryResource = _mapper.Map<Category, CategoryResource>(result.Category);
		return Ok(categoryResource);
	}
```
在验证请求数据并将资源映射到我们的模型之后，我们将其传递给我们的服务以保留数据。
如果失败，则API返回错误的请求。如果没有，API会将新类别（现在包括诸如new的数据Id）映射到我们先前创建的类别CategoryResource，并将其发送给客户端。

现在，让我们为服务实现真正的逻辑。

**第13步—数据库逻辑和工作单元模式**

由于我们要将数据持久化到数据库中，因此我们需要在仓储中使用一种新方法。

向ICategoryRepository接口添加AddAsync新方法：

```
public interface ICategoryRepository
	{
		 Task<IEnumerable<Category>> ListAsync();
		 Task AddAsync(Category category);
	}
```
现在，让我们在真正的仓储类中实现此方法：
```
public class CategoryRepository : BaseRepository, ICategoryRepository
	{
		public CategoryRepository(AppDbContext context) : base(context)
		{ }
	

		public async Task<IEnumerable<Category>> ListAsync()
		{
			return await _context.Categories.ToListAsync();
		}
	

		public async Task AddAsync(Category category)
		{
			await _context.Categories.AddAsync(category);
		}
	}
```
在这里，我们只是在集合中添加一个新类别。
当我们向中添加类时DBSet<>，EF Core将开始跟踪模型发生的所有更改，并在当前状态下使用此数据生成将插入，更新或删除模型的查询。

当前的实现只是将模型添加到我们的集合中，但是**我们的数据仍然不会保存**。

在上下文类中提供了SaveChanges的方法，我们必须调用该方法才能真正将查询执行到数据库中。我之所以没有在这里调用它，是因为[仓储不应该持久化数据](https://programmingwithmosh.com/entity-framework/common-mistakes-with-the-repository-pattern/)，它只是一种内存集合对象。

即使在经验丰富的.NET开发人员之间，该主题也引起很大争议，但是让我向您解释为什么您不应该在仓储类中调用SaveChanges方法。

我们可以从概念上将仓储像.NET框架中存在的任何其他集合一样。在.NET（和许多其他编程语言，例如Javascript和Java）中处理集合时，通常可以：

* 向其中添加新项（例如，当您将数据推送到列表，数组和字典时）；
* 查找或过滤项目；
* 从集合中删除一个项目；
* 替换给定的项目，或更新它。

想一想现实世界中的清单。想象一下，您正在编写一份购物清单以在超市购买东西（*巧合，不是吗？*）。

在列表中，写下您需要购买的所有水果。您可以将水果添加到此列表中，如果放弃购买就删除水果，也可以替换水果的名称。但是您无法**将**水果**保存**到列表中。用简单的英语说这样的话是没有意义的。

>**提示：**在使用面向对象的编程语言设计类和接口时，请尝试使用自然语言来检查您所做的工作是否正确。
>>例如，说人实现了person的接口是有道理的，但是说一个人实现了一个帐户却没有道理。

如果您要“保存”水果清单（在这种情况下，要购买所有水果），请付款，然后超市会处理库存数据以检查他们是否必须从供应商处购买更多水果。

编程时可以应用相同的逻辑。仓储不应保存，更新或删除数据。相反，他们应该将其委托给其他类来处理此逻辑。

将数据直接保存到仓储中时，还有另一个问题：**您不能使用transaction**。

想象一下，我们的应用程序具有一种日志记录机制，该机制存储一些用户名，并且每次对API数据进行更改时都会执行操作。

现在想象一下，由于某种原因，您调用了一个更新用户名的服务（这是不常见的情况，但让我们考虑一下）。

您同意要更改虚拟用户表中的用户名，首先必须更新所有日志以正确告诉谁执行了该操作，对吗？

现在想象我们已经为用户和不同仓储中的日志实现了update方法，它们都调用了SaveChanges。如果这些方法之一在更新过程中失败，会发生什么？最终会导致数据不一致。

只有在一切完成之后，我们才应该将更改保存到数据库中。为此，我们必须使用[transaction](https://en.wikipedia.org/wiki/Database_transaction)，这基本上是大多数数据库实现的功能，只有在完成复杂的操作后才能保存数据。

*“-好的，所以如果我们不能在这里保存东西，我们应该在哪里做？”*

处理此问题的常见模式是[工作单元模式](https://docs.microsoft.com/en-us/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application)。此模式包含一个类，该类将我们的AppDbContext实例作为依赖项接收，并公开用于开始，完成或中止事务的方法。

在这里，我们将使用工作单元的简单实现来解决我们的问题。

Repositories在Domain层的仓储文件夹Repositories内添加一个新接口IUnitOfWork：

```
using System.Threading.Tasks;
	

	namespace Supermarket.API.Domain.Repositories
	{
	    public interface IUnitOfWork
	    {
	         Task CompleteAsync();
	    }
	}
```
如您所见，它仅公开一种将异步完成数据管理操作的方法。
现在让我们添加实际的实现。

在Persistence层RepositoriesRepositories文件夹中的添加一个名为的UnitOfWork的新类：

```
using System.Threading.Tasks;
	using Supermarket.API.Domain.Repositories;
	using Supermarket.API.Persistence.Contexts;
	

	namespace Supermarket.API.Persistence.Repositories
	{
	    public class UnitOfWork : IUnitOfWork
	    {
	        private readonly AppDbContext _context;
	

	        public UnitOfWork(AppDbContext context)
	        {
	            _context = context;     
	        }
	

	        public async Task CompleteAsync()
	        {
	            await _context.SaveChangesAsync();
	        }
	    }
	}
```
这是一个简单，干净的实现，仅在使用仓储修改完所有更改后，才将所有更改保存到数据库中。
如果研究工作单元模式的实现，则会发现实现回滚操作的更复杂的模式。

由于**EF Core已经在后台实现了仓储模式和工作单元**，因此我们不必在意回滚方法。

*“ - 什么？那么为什么我们必须创建所有这些接口和类？”*

将持久性逻辑与业务规则分开在代码可重用性和维护方面具有许多优势。如果直接使用EF Core，我们最终将拥有更复杂的类，这些类将很难更改。

想象一下，将来您决定将ORM框架更改为其他框架，例如[Dapper](https://www.c-sharpcorner.com/article/crud-operation-in-asp-net-core-2-0-using-dapper-orm/)，或者由于性能而必须实施纯SQL查询。如果将查询逻辑与服务耦合在一起，将很难更改该逻辑，因为您必须在许多类中进行此操作。

使用仓储模式，您可以简单地实现一个新的仓储类并使用依赖注入将其绑定。

因此，基本上，如果您直接在服务中使用EF Core，并且必须进行一些更改，那么您将获得：

就像我说的那样，EF Core在后台实现了工作单元和仓储模式。我们可以将DbSet<>属性视为仓储。而且，SaveChanges仅在所有数据库操作成功的情况下才保留数据。

现在，您知道什么是工作单元以及为什么将其与仓储一起使用，让我们实现真实服务的逻辑。

```
public class CategoryService : ICategoryService
	{
		private readonly ICategoryRepository _categoryRepository;
		private readonly IUnitOfWork _unitOfWork;
	

		public CategoryService(ICategoryRepository categoryRepository, IUnitOfWork unitOfWork)
		{
			_categoryRepository = categoryRepository;
			_unitOfWork = unitOfWork;
		}
	

		public async Task<IEnumerable<Category>> ListAsync()
		{
			return await _categoryRepository.ListAsync();
		}
	

		public async Task<SaveCategoryResponse> SaveAsync(Category category)
		{
			try
			{
				await _categoryRepository.AddAsync(category);
				await _unitOfWork.CompleteAsync();
				
				return new SaveCategoryResponse(category);
			}
			catch (Exception ex)
			{
				// Do some logging stuff
				return new SaveCategoryResponse($"An error occurred when saving the category: {ex.Message}");
			}
		}
	}
```
多亏了我们的解耦架构，我们可以简单地将实例UnitOfWork作为此类的依赖传递。
我们的业务逻辑非常简单。

首先，我们尝试将新类别添加到数据库中，然后API尝试保存新类别，将所有内容包装在try-catch块中。

如果失败，则API会调用一些虚构的日志记录服务，并返回指示失败的响应。

如果该过程顺利完成，则应用程序将返回成功响应，并发送我们的类别数据。简单吧？

>**提示：**在现实世界的应用程序中，您不应将所有内容包装在通用的try-catch块中，而应分别处理所有可能的错误。
>>简单地添加一个try-catch块并不能解决大多数可能的失败情况。请确保正确实现错误处理。

测试我们的API之前的最后一步是将工作单元接口绑定到其各自的类。

将此新行添加到类的ConfigureServices方法中Startup：

```
services.AddScoped<IUnitOfWork, UnitOfWork>();
```
现在让我们测试一下！
**第14步-使用Postman测试我们的POST端点**

重新启动我们的应用程序dotnet run。

我们无法使用浏览器测试POST端点。让我们使用**Postman**测试我们的端点。这是测试RESTful API的非常有用的工具。

打开**Postman**，然后关闭介绍性消息。您会看到这样的屏幕：

![图片](https://uploader.shimo.im/f/k9uqO0tzHP8sXyGu.png!thumbnail)

屏幕显示测试端点的选项

GET默认情况下，将所选内容更改为选择框POST。

在Enter request URL字段中输入API地址。

我们必须提供请求正文数据以发送到我们的API。单击Body菜单项，然后将其下方显示的选项更改为raw。

Postman将在右侧显示一个Text选项，将其更改为JSON (application/json)并粘贴以下JSON数据：

```
{
  "name": ""
}
```
![图片](https://uploader.shimo.im/f/aMktVnVAavwvF3Q3.png!thumbnail)发送请求前的屏幕

如您所见，我们将向我们的新端点发送一个空的名称字符串。

点击Send按钮。您将收到如下输出：

![图片](https://uploader.shimo.im/f/5yuSjCfYKD0ELqiA.png!thumbnail)

如您所见，我们的验证逻辑有效！

您还记得我们为端点创建的验证逻辑吗？此输出是它起作用的证明！

还要注意右侧显示的400状态代码。该BadRequest结果自动将此状态码的响应。

现在，让我们将JSON数据更改为有效数据，以查看新的响应：

![图片](https://uploader.shimo.im/f/zDpJNG3Yl8Q0XimL.png!thumbnail)

最后，我们期望得到的结果

API正确创建了我们的新资源。

到目前为止，我们的API可以列出和创建类别。您学到了很多有关C＃语言，ASP.NET Core框架以及构造API的通用设计方法的知识。

让我们继续我们的类别API，创建用于更新类别的端点。

从现在开始，由于我向您解释了大多数概念，因此我将加快解释速度，并专注于新主题，以免浪费您的时间。 Let’s go!

# **第15步-更新类别**
要更新类别，我们需要一个HTTP PUT端点。

我们必须编写的逻辑与POST逻辑非常相似：

* 首先，我们必须使用来验证传入的请求ModelState。
* 如果请求有效，则API应使用AutoMapper将传入资源映射到模型类。
* 然后，我们需要调用我们的服务，告诉它更新类别，提供相应的类别Id和更新的数据；
* 如果Id数据库中没有给定的类别，我们将返回错误的请求。我们可以使用NotFound结果来代替，但是对于这个范围而言，这并不重要，因为我们向客户端应用程序提供了错误消息。
* 如果正确执行了保存逻辑，则服务必须返回包含更新的类别数据的响应。如果不是，它应该给我们指示该过程失败，并显示一条消息指示原因；
* 最后，如果有错误，则API返回错误的请求。如果不是，它将更新的类别模型映射到类别资源，并将成功响应返回给客户端应用程序。

让我们将新PutAsync方法添加到控制器类中：

```
[HttpPut("{id}")]
	public async Task<IActionResult> PutAsync(int id, [FromBody] SaveCategoryResource resource)
	{
		if (!ModelState.IsValid)
			return BadRequest(ModelState.GetErrorMessages());
	

		var category = _mapper.Map<SaveCategoryResource, Category>(resource);
		var result = await _categoryService.UpdateAsync(id, category);
	

		if (!result.Success)
			return BadRequest(result.Message);
	

		var categoryResource = _mapper.Map<Category, CategoryResource>(result.Category);
		return Ok(categoryResource);
	}
```
如果将其与POST逻辑进行比较，您会注意到这里只有一个区别：HttPut属性指定给定路由应接收的参数。
我们将调用此端点，将类别指定Id 为最后一个URL片段，例如/api/categories/1。ASP.NET Core管道将此片段解析为相同名称的参数。

现在我们必须UpdateAsync在ICategoryService接口中定义方法签名：

```
public interface ICategoryService
	{
		Task<IEnumerable<Category>> ListAsync();
		Task<SaveCategoryResponse> SaveAsync(Category category);
		Task<SaveCategoryResponse> UpdateAsync(int id, Category category);
	}
```
现在让我们转向真正的逻辑。
# **第16步-更新逻辑**
首先，要更新类别，我们需要从数据库中返回当前数据（如果存在）。我们还需要将其更新到我们的中DBSet<>。

让我们在ICategoryService界面中添加两个新的方法约定：

```
public interface ICategoryRepository
	{
		Task<IEnumerable<Category>> ListAsync();
		Task AddAsync(Category category);
		Task<Category> FindByIdAsync(int id);
		void Update(Category category);
	}
```
我们已经定义了FindByIdAsync方法，该方法将从数据库中异步返回一个类别，以及该Update方法。请注意，该Update方法不是异步的，因为EF Core API不需要异步方法来更新模型。
现在，让我们在CategoryRepository类中实现真正的逻辑：

```
public async Task<Category> FindByIdAsync(int id)
	{
		return await _context.Categories.FindAsync(id);
	}
	

	public void Update(Category category)
	{
		_context.Categories.Update(category);
	}
```
最后，我们可以对服务逻辑进行编码：
```
public async Task<SaveCategoryResponse> UpdateAsync(int id, Category category)
	{
		var existingCategory = await _categoryRepository.FindByIdAsync(id);
	

		if (existingCategory == null)
			return new SaveCategoryResponse("Category not found.");
	

		existingCategory.Name = category.Name;
	

		try
		{
			_categoryRepository.Update(existingCategory);
			await _unitOfWork.CompleteAsync();
	

			return new SaveCategoryResponse(existingCategory);
		}
		catch (Exception ex)
		{
			// Do some logging stuff
			return new SaveCategoryResponse($"An error occurred when updating the category: {ex.Message}");
		}
	}
```
API尝试从数据库中获取类别。如果结果为null，我们将返回一个响应，告知该类别不存在。如果类别存在，我们需要设置其新名称。
然后，API会尝试保存更改，例如创建新类别时。如果该过程完成，则该服务将返回成功响应。如果不是，则执行日志记录逻辑，并且端点接收包含错误消息的响应。

现在让我们对其进行测试。首先，让我们添加一个新类别Id以使用有效类别。我们可以使用播种到数据库中的类别的标识符，但是我想通过这种方式向您展示我们的API将更新正确的资源。

再次运行该应用程序，然后使用Postman将新类别发布到数据库中：

![图片](https://uploader.shimo.im/f/fxIzjvpz7Y0kf8XP.png!thumbnail)

添加新类别以供日后更新

使用一个可用的数据Id，将POST 选项更改PUT为选择框，然后在URL的末尾添加ID值。将name属性更改为其他名称，然后发送请求以检查结果：

![图片](https://uploader.shimo.im/f/VMXmxLVsZNsqqXSK.png!thumbnail)

类别数据已成功更新

您可以将GET请求发送到API端点，以确保您正确编辑了类别名称：

![图片](https://uploader.shimo.im/f/G5ipYxpQk5gJSVBI.png!thumbnail)

那是现在GET请求的结果

我们必须对类别执行的最后一项操作是排除类别。让我们创建一个HTTP Delete端点。

# **第17步-删除类别**
删除类别的逻辑确实很容易实现，因为我们所需的大多数方法都是先前构建的。

这些是我们工作路线的必要步骤：

* API需要调用我们的服务，告诉它删除我们的类别，并提供相应的Id;
* 如果数据库中没有具有给定ID的类别，则该服务应返回一条消息指出该类别；
* 如果执行删除逻辑没有问题，则服务应返回包含我们已删除类别数据的响应。如果没有，它应该给我们一个指示，表明该过程失败了，并可能出现错误消息。
* 最后，如果有错误，则API返回错误的请求。如果不是，则API会将更新的类别映射到资源，并向客户端返回成功响应。

让我们开始添加新的端点逻辑：

```
[HttpDelete("{id}")]
	public async Task<IActionResult> DeleteAsync(int id)
	{
		var result = await _categoryService.DeleteAsync(id);
	

		if (!result.Success)
			return BadRequest(result.Message);
	

		var categoryResource = _mapper.Map<Category, CategoryResource>(result.Category);
		return Ok(categoryResource);
	}
```
该HttpDelete属性还定义了一个id 模板。
在将DeleteAsync签名添加到我们的ICategoryService接口之前，我们需要做一些小的重构。

新的服务方法必须返回包含类别数据的响应，就像对PostAsyncand UpdateAsync方法所做的一样。我们可以SaveCategoryResponse为此目的重用，但在这种情况下我们不会保存数据。

为了避免创建具有相同形状的新类来满足此要求，我们可以将我们重命名SaveCategoryResponse为CategoryResponse。

如果您使用的是Visual Studio Code，则可以打开SaveCategoryResponse类，将鼠标光标放在类名上方，然后使用选项Change All Occurrences*** ***来重命名该类：

![图片](https://uploader.shimo.im/f/9F3zYANcrMUdFlMe.png!thumbnail)

确保也重命名文件名。

让我们将DeleteAsync方法签名添加到ICategoryService 接口中：

```
public interface ICategoryService
	{
		Task<IEnumerable<Category>> ListAsync();
		Task<CategoryResponse> SaveAsync(Category category);
		Task<CategoryResponse> UpdateAsync(int id, Category category);
		Task<CategoryResponse> DeleteAsync(int id);
	}
```
在实施删除逻辑之前，我们需要在仓储中使用一种新方法。
将Remove方法签名添加到ICategoryRepository接口：

```
void Remove(Category category);
```
现在，在仓储类上添加真正的实现：

```
public void Remove(Category category)
	{
		_context.Categories.Remove(category);
	}
```
EF Core要求将模型的实例传递给Remove方法，以正确了解我们要删除的模型，而不是简单地传递Id。
最后，让我们在CategoryService类上实现逻辑：

```
public async Task<CategoryResponse> DeleteAsync(int id)
	{
		var existingCategory = await _categoryRepository.FindByIdAsync(id);
	

		if (existingCategory == null)
			return new CategoryResponse("Category not found.");
	

		try
		{
			_categoryRepository.Remove(existingCategory);
			await _unitOfWork.CompleteAsync();
	

			return new CategoryResponse(existingCategory);
		}
		catch (Exception ex)
		{
			// Do some logging stuff
			return new CategoryResponse($"An error occurred when deleting the category: {ex.Message}");
		}
	}
```
这里没有新内容。该服务尝试通过ID查找类别，然后调用我们的仓储以删除类别。最后，工作单元完成将实际操作执行到数据库中的事务。
*“-嘿，但是每个类别的产品呢？为避免出现错误，您是否不需要先创建仓储并删除产品？”*

答案是**否定的**。借助[EF Core跟踪机制](https://docs.microsoft.com/en-us/ef/core/querying/tracking)，当我们从数据库中加载模型时，框架便知道了该模型具有哪些关系。如果我们删除它，EF Core知道它应该首先递归删除所有相关模型。

在将类映射到数据库表时，我们可以禁用此功能，但这在本教程的范围之外。如果您想了解此功能，[请看这里](https://entityframeworkcore.com/saving-data-cascade-delete)。

现在是时候测试我们的新端点了。再次运行该应用程序，并使用Postman发送DELETE请求，如下所示：

![图片](https://uploader.shimo.im/f/VRCjsPelqx4qADIx.png!thumbnail)

如您所见，API毫无问题地删除了现有类别

我们可以通过发送GET请求来检查我们的API是否正常工作：

![图片](https://uploader.shimo.im/f/iyiKvuB7e0IfSTQf.png!thumbnail)我们已经完成了类别API。现在是时候转向产品API。

# **步骤18-产品API**
到目前为止，您已经学习了如何实现所有基本的HTTP动词来使用ASP.NET Core处理CRUD操作。让我们进入实现产品API的下一个层次。

我将不再详细介绍所有HTTP动词，因为这将是详尽无遗的。在本教程的最后一部分，我将仅介绍GET请求，以向您展示在从数据库查询数据时如何包括相关实体，以及如何使用Description我们为EUnitOfMeasurement 枚举值定义的属性。

将新控制器ProductsController添加到名为Controllers的文件夹中。

在这里编写任何代码之前，我们必须创建产品资源。

让我刷新您的记忆，再次显示我们的资源应如何：

```
{
 [
  {
   "id": 1,
   "name": "Sugar",
   "quantityInPackage": 1,
   "unitOfMeasurement": "KG"
   "category": {
   "id": 3,
   "name": "Sugar"
   }
  },
  … // Other products
 ]
}
```
我们想要一个包含数据库中所有产品的JSON数组。
JSON数据与产品模型有两点不同：

* 测量单位以较短的方式显示，仅显示其缩写。
* 我们输出类别数据**而不**包括CategoryId属性。

为了表示度量单位，我们可以使用简单的字符串属性代替枚举类型（顺便说一下，我们没有JSON数据的默认枚举类型，因此我们必须将其转换为其他类型）。

现在，我们现在要塑造新资源，让我们创建它。ProductResource在Resources文件夹中添加一个新类：

```
namespace Supermarket.API.Resources
	{
	    public class ProductResource
	    {
	        public int Id { get; set; }
	        public string Name { get; set; }
	        public int QuantityInPackage { get; set; }
	        public string UnitOfMeasurement { get; set; }
	        public CategoryResource Category {get;set;}
	    }
	}
```
现在，我们必须配置模型类和新资源类之间的映射。
映射配置将与用于其他映射的配置几乎相同，但是在这里，我们必须处理将EUnitOfMeasurement枚举转换为字符串的操作。

您还记得StringValue应用于枚举类型的属性吗？现在，我将向您展示如何使用.NET框架的强大功能：[反射 API](https://www.tutorialspoint.com/csharp/csharp_reflection.htm)提取此信息。

反射 API是一组强大的资源工具集，可让我们提取和操作元数据。许多框架和库（包括ASP.NET Core本身）都利用这些资源来处理许多后台工作。

现在让我们看看它在实践中是如何工作的。将新类添加到Extensions名为的文件夹中EnumExtensions。

```
using System.ComponentModel;
	using System.Reflection;
	

	namespace Supermarket.API.Extensions
	{
	    public static class EnumExtensions
	    {
	        public static string ToDescriptionString<TEnum>(this TEnum @enum)
	        {
	            FieldInfo info = @enum.GetType().GetField(@enum.ToString());
	            var attributes = (DescriptionAttribute[])info.GetCustomAttributes(typeof(DescriptionAttribute), false);
	

	            return attributes?[0].Description ?? @enum.ToString();
	        }
	    }
	}
```
第一次看代码可能会让人感到恐惧，但这并不复杂。让我们分解代码定义以了解其工作原理。
首先，我们定义了一种[通用方法](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/)（一种方法，该方法可以接收不止一种类型的参数，在这种情况下，该方法由TEnum声明表示），该方法接收给定的枚举作为参数。

由于enum是C＃中的保留关键字，因此我们在参数名称前面添加了@，以使其成为有效名称。

该方法的第一步是使用该方法获取参数的类型信息（类，接口，枚举或结构定义）GetType。

然后，该方法使用来获取特定的枚举值（例如Kilogram）GetField(@enum.ToString())。

下一行找到Description应用于枚举值的所有属性，并将其数据存储到数组中（在某些情况下，我们可以为同一属性指定多个属性）。

最后一行使用较短的语法来检查我们是否至少有一个枚举类型的描述属性。如果有，我们将返回Description此属性提供的值。如果不是，我们使用默认的强制类型转换将枚举作为字符串返回。

?.操作者（[零条件运算](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/conditional-operator)）检查该值是否null访问其属性之前。

??运算符（[空合并运算符](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator)）告诉应用程序在左边的返回值，如果它不为空，或者在正确的，否则价值。

现在我们有了扩展方法来提取描述，让我们配置模型和资源之间的映射。多亏了AutoMapper，我们只需要多一行就可以做到这一点。

打开ModelToResourceProfile类并通过以下方式更改代码：

```
using AutoMapper;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Extensions;
	using Supermarket.API.Resources;
	

	namespace Supermarket.API.Mapping
	{
	    public class ModelToResourceProfile : Profile
	    {
	        public ModelToResourceProfile()
	        {
	            CreateMap<Category, CategoryResource>();
	

	            CreateMap<Product, ProductResource>()
	                .ForMember(src => src.UnitOfMeasurement,
	                           opt => opt.MapFrom(src => src.UnitOfMeasurement.ToDescriptionString()));
	        }
	    }
	}
```
此语法告诉AutoMapper使用新的扩展方法将我们的EUnitOfMeasurement值转换为包含其描述的字符串。简单吧？您可以[阅读官方文档](http://docs.automapper.org/en/stable/Inline-Mapping.html)以了解完整语法。
注意，我们尚未为category属性定义任何映射配置。因为我们之前为类别配置了映射，并且由于产品模型具有相同类型和名称的category属性，所以AutoMapper隐式知道应该使用各自的配置来映射它。

现在，我们添加端点代码。更改ProductsController代码：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using AutoMapper;
	using Microsoft.AspNetCore.Mvc;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Domain.Services;
	using Supermarket.API.Resources;
	

	namespace Supermarket.API.Controllers
	{
	    [Route("/api/[controller]")]
	    public class ProductsController : Controller
	    {
	        private readonly IProductService _productService;
	        private readonly IMapper _mapper;
	

	        public ProductsController(IProductService productService, IMapper mapper)
	        {
	            _productService = productService;
	            _mapper = mapper;
	        }
	

	        [HttpGet]
	        public async Task<IEnumerable<ProductResource>> ListAsync()
	        {
	            var products = await _productService.ListAsync();
	            var resources = _mapper.Map<IEnumerable<Product>, IEnumerable<ProductResource>>(products);
	            return resources;
	        }
	    }
	}
```
基本上，为类别控制器定义的结构相同。
让我们进入服务部分。将一个新IProductService接口添加到Domain层中的Services文件夹中：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Supermarket.API.Domain.Models;
	

	namespace Supermarket.API.Domain.Services
	{
	    public interface IProductService
	    {
	         Task<IEnumerable<Product>> ListAsync();
	    }
	}
```
您应该已经意识到，在真正实现新服务之前，我们需要一个仓储。
IProductRepository在相应的文件夹中添加一个名为的新接口：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Supermarket.API.Domain.Models;
	

	namespace Supermarket.API.Domain.Repositories
	{
	    public interface IProductRepository
	    {
	         Task<IEnumerable<Product>> ListAsync();
	    }
	}
```
现在，我们实现仓储。除了必须在查询数据时返回每个产品的相应类别数据外，我们几乎必须像对类别仓储一样实现。
默认情况下，EF Core在查询数据时不包括与模型相关的实体，因为它可能非常慢（想象一个具有十个相关实体的模型，所有相关实体都有自己的关系）。

要包括类别数据，我们只需要多一行：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Microsoft.EntityFrameworkCore;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Domain.Repositories;
	using Supermarket.API.Persistence.Contexts;
	

	namespace Supermarket.API.Persistence.Repositories
	{
	    public class ProductRepository : BaseRepository, IProductRepository
	    {
	        public ProductRepository(AppDbContext context) : base(context)
	        {
	        }
	

	        public async Task<IEnumerable<Product>> ListAsync()
	        {
	            return await _context.Products.Include(p => p.Category)
	                                          .ToListAsync();
	        }
	    }
	}
```
请注意对的调用Include(p => p.Category)。我们可以链接此语法，以在查询数据时包含尽可能多的实体。执行选择时，EF Core会将其转换为联接。
现在，我们可以ProductService像处理类别一样实现类：

```
using System.Collections.Generic;
	using System.Threading.Tasks;
	using Supermarket.API.Domain.Models;
	using Supermarket.API.Domain.Repositories;
	using Supermarket.API.Domain.Services;
	

	namespace Supermarket.API.Services
	{
	    public class ProductService : IProductService
	    {
	        private readonly IProductRepository _productRepository;
	    
	        public ProductService(IProductRepository productRepository)
	        {
	            _productRepository = productRepository;
	        }
	

	        public async Task<IEnumerable<Product>> ListAsync()
	        {
	            return await _productRepository.ListAsync();
	        }
	    }
	}
```
让我们绑定更改Startup类的新依赖项：
```
public void ConfigureServices(IServiceCollection services)
	{
	    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
	

	    services.AddDbContext<AppDbContext>(options =>
	    {
	        options.UseInMemoryDatabase("supermarket-api-in-memory");
	    });
	

	    services.AddScoped<ICategoryRepository, CategoryRepository>();
	    services.AddScoped<IProductRepository, ProductRepository>();
	    services.AddScoped<IUnitOfWork, UnitOfWork>();
	

	    services.AddScoped<ICategoryService, CategoryService>();
	    services.AddScoped<IProductService, ProductService>();
	

	    services.AddAutoMapper();
	}
```
最后，在测试API之前，让我们AppDbContext在初始化应用程序时更改类以包括一些产品，以便我们看到结果：
```
protected override void OnModelCreating(ModelBuilder builder)
	{
	    base.OnModelCreating(builder);
	    
	    builder.Entity<Category>().ToTable("Categories");
	    builder.Entity<Category>().HasKey(p => p.Id);
	    builder.Entity<Category>().Property(p => p.Id).IsRequired().ValueGeneratedOnAdd().HasValueGenerator<InMemoryIntegerValueGenerator<int>>();
	    builder.Entity<Category>().Property(p => p.Name).IsRequired().HasMaxLength(30);
	    builder.Entity<Category>().HasMany(p => p.Products).WithOne(p => p.Category).HasForeignKey(p => p.CategoryId);
	

	    builder.Entity<Category>().HasData
	    (
	        new Category { Id = 100, Name = "Fruits and Vegetables" }, // Id set manually due to in-memory provider
	        new Category { Id = 101, Name = "Dairy" }
	    );
	

	    builder.Entity<Product>().ToTable("Products");
	    builder.Entity<Product>().HasKey(p => p.Id);
	    builder.Entity<Product>().Property(p => p.Id).IsRequired().ValueGeneratedOnAdd();
	    builder.Entity<Product>().Property(p => p.Name).IsRequired().HasMaxLength(50);
	    builder.Entity<Product>().Property(p => p.QuantityInPackage).IsRequired();
	    builder.Entity<Product>().Property(p => p.UnitOfMeasurement).IsRequired();
	

	    builder.Entity<Product>().HasData
	    (
	        new Product
	        {
	            Id = 100,
	            Name = "Apple",
	            QuantityInPackage = 1,
	            UnitOfMeasurement = EUnitOfMeasurement.Unity,
	            CategoryId = 100
	        },
	        new Product
	        {
	            Id = 101,
	            Name = "Milk",
	            QuantityInPackage = 2,
	            UnitOfMeasurement = EUnitOfMeasurement.Liter,
	            CategoryId = 101,
	        }
	    );
	}
```
我添加了两个虚构产品，将它们与初始化应用程序时我们播种的类别相关联。
该测试了！再次运行API并发送GET请求以/api/products使用Postman：

![图片](https://uploader.shimo.im/f/h9cMeoAIZg4vyRgj.png!thumbnail)

就是这样！恭喜你！

现在，您将了解如何使用解耦的代码架构使用ASP.NET Core构建RESTful API。您了解了.NET Core框架的许多知识，如何使用C＃，EF Core和AutoMapper的基础知识以及在设计应用程序时要使用的许多有用的模式。

您可以检查API的完整实现，包括产品的其他HTTP动词，并检查Github仓储：

[evgomes / supermarket-api](https://github.com/evgomes/supermarket-api)

[使用ASP.NET Core 2.2构建的简单RESTful API，展示了如何使用分离的，可维护的……创建RESTful服务](https://github.com/evgomes/supermarket-api)[。github.com](https://github.com/evgomes/supermarket-api)

# **结论**
ASP.NET Core是创建Web应用程序时使用的出色框架。它带有许多有用的API，可用于构建干净，可维护的应用程序。创建专业应用程序时，可以将其视为一种选择。

本文并未涵盖专业API的所有方面，但您已学习了所有基础知识。您还学到了许多有用的模式，可以解决我们每天面临的模式。

希望您喜欢这篇文章，希望对您有所帮助。期待你的反馈，以便我能进一步提高。

**进一步学习的可用参考资料**

[.NET Core教程-Microsoft文档](https://docs.microsoft.com/en-us/dotnet/core/tutorials/)

[ASP.NET Core文档-Microsoft文档](https://docs.microsoft.com/zh-cn/aspnet/#pivot=core&panel=core_tutorials)

