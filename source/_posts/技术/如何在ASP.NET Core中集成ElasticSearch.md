---
title:  如何在ASP.NET Core中集成ElasticSearch
date: 2021-4-25 18:56
tags: 技术
author: 邹溪源
categories:
  - 技术
---

[查看原文](https://www.blexin.com/en-US/Article/Blog/How-to-integrate-ElasticSearch-in-ASPNET-Core-70)

![图片](https://uploader.shimo.im/f/mXys791UMG7cuOvS.png!thumbnail)

我敢打赌，您肯定会被要求向Web应用程序中添加高级搜索功能，而且通常是全文的类似Google的搜索。

在技​​术电子商务的开发过程中，我们被要求允许用户对产品进行高级研究，以便他们可以高效，完全地找到所需的内容。

我们基于对象的所有字段上给定字符串的搜索尝试了自定义搜索的实现。为了优化时间，我们尝试在服务和数据库级别之间添加一个缓存层，以避免对数据库造成过多压力，但是我们对结果不满意。然后，我们在市场上搜索了可以满足我们需求的第三方产品，经过深入分析，我们选择采用ElasticSearch：一个基于REST协议的，可管理研究和分析的分布式，易于适应的搜索引擎同样，也方便了数据的外推和转换。 

具体来说，我们正在谈论基于Apache Lucene的开源全文搜索引擎，该引擎可用于管理文档的索引和研究。让我们尝试了解基本概念。

ElasticSearch将数据存储在一个或多个索引中。ES的索引与SQL DB的索引非常相似，因为我们使用它来存储和读取文档。

文档是ElasticSearch世界的主要实体。它由一组具有名称和一个或多个值的字段组成。每个文档可能具有一组字段，并且没有给出任何架构或定义的结构。这只是一个JSON对象。

所有文档在存储之前都经过分析。这种分析过程（称为映射）是通过过滤数据内容（例如，删除HTML标签）并将其标记化来执行的，以便将文档拆分为标记。

ElasticSearch中的每个文档都有一个类型。这样就可以将各种文档类型存储在同一索引上，并为几种类型获取几种映射。

ElasticSearch服务器的单个实例称为Node。在很多情况下，单个节点就足够了，但是有时您需要管理故障，或者您有太多数据无法使用单个节点进行管理。在这种情况下，您可以使用多节点集群，这是一组协同工作的节点来管理比单个实例无法处理的更大的负载。您可以配置群集，以便即使某些节点不可用，也可以保证搜索和管理功能。

为了使群集正常运行，ElasticSearch将数据分布在Apache Lucene的多个物理索引上。这些索引称为“ 碎片”，而扩展过程称为“ 碎片”。ElasticSearch自动管理分片，因此最终用户似乎只是一个大索引。

副本是分片的副本，可用于以原始分片的相同模式进行查询。

副本可减轻无法处理所有请求的单个节点上的负载，并提供更高的数据安全性，因为如果您丢失了原始分片中的数据，则可以在副本上对其进行恢复。

ElasticSearch收集了大量有关集群状态，索引设置的信息，并将它们存储到网关中。

从结构上讲，ElasticSearch基于一些简单的关键概念：

* 默认设置和值使得默认配置足以立即使用ElasticSearch。
* 它以分布式方式工作。节点自动成为集群的一部分，并且在设置过程中，节点尝试加入集群。
* 没有SPOF的P2P体系结构（单点故障）。节点自动连接到群集的其他计算机以更改数据和相互监视；
* 只需在集群中添加新节点，就可以轻松地进行扩展，无论是在数据量上还是在容量上。
* 在组织索引中的数据方面没有任何限制。允许用户修改数据模型而不会对搜索产生任何影响；
* NRT（近实时）搜索和版本控制。由于其分布式特性，无法避免延迟和位于不同节点上的数据之间的差异。因此，它提供了版本控制机制。

当ElasticSearch节点启动时，它使用多播（或单播，如果已配置）来查找同一集群中的其他节点并连接到它们。

![图片](https://uploader.shimo.im/f/pnS8DrzVMpeVRMZF.png!thumbnail)

在群集中，选择一个节点作为主节点。该节点负责管理集群状态和将分片分配给节点的过程。主节点读取集群状态，并在需要时启动恢复模式，该模式允许知道哪些分片可用，并指定其中一个作为主分片。这样，即使群集没有可用的全部资源，它也似乎可以正常工作。然后，主节点查找重复的分片，并将其作为副本处理。

在标准运行期间，主节点检查所有可用节点是否正常工作。如果其中之一在配置的时间范围内不可用，则将该节点视为已损坏，并运行容错过程。容错的主要活动是平衡已损坏节点的群集和碎片，并分配一个负责这些碎片的新节点。然后，对于每个主分片丢失，将定义一个在可用副本之间选择的新主分片。

![图片](https://uploader.shimo.im/f/yflWUlBxnnvxdfvj.png!thumbnail)

如前所述，ElasticSearch提供了一些API REST，可供每个能够发送HTTP请求和接收HTTP响应的系统使用（大多数开发框架的所有浏览器和库）。

ElasticSearch请求由一些包含的已定义URL发送。最终是JSON主体。响应也是JSON文档。

ElasticSearch提供了四种索引数据的方式。

1. 索引API：它允许将文档发送到已定义的索引；
2. 批量API：它允许通过HTTP协议发送多个文档；
3. UDP批量API：它允许通过任何协议发送多个文档（更快但更不可靠）；
4. 插件：在节点上执行，它们从外部系统获取数据。

重要的是要记住，索引只是在主分片上而不是在其副本上，因此，如果将索引请求发送到不包含主分片或可能包含其副本的节点，则该请求将转发到主分片。

![图片](https://uploader.shimo.im/f/TlwERyJAobkhowVG.png!thumbnail)

使用**Query API**执行搜索。使用**查询DSL**（基于JSON的语言来构建复杂的查询），可以：

* 使用各种类型的查询，包括简单查询，短语，范围，布尔值，空间查询和其他查询；
* 通过组合简单查询来构建复杂查询；
* 通过排除不符合选定条件的文档而不影响其分数来过滤文档；
* 查找与其他文件相似的文件；
* 查找给定短语的建议或更正；
* 查找与给定文档匹配的查询。

搜索不是一个单阶段的简单过程，但是，通常可以将其分为两个阶段：**scatter（分散）**，在其中查询索引的所有相关分片；**gather（收集）**，在其中收集，处理和排序所有宝贵的结果。

![图片](https://uploader.shimo.im/f/rkT6GBkfGNGBek4r.png!thumbnail)

弄脏你的手！

ES提供了云和本地两种使用方式。如果要在Windows计算机上安装它，则需要具有Java虚拟机的更新版本（[https://www.elastic.co/support/matrix#matrix_jvm](https://www.elastic.co/support/matrix#matrix_jvm)），然后可以从ElasticSearch下载中下载一个zip文件。页面（[https://www.elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch)）并将其提取到磁盘上的文件夹中，例如*C：\ Elasticsearch*。

要执行它，您可以运行*C：\ Elasticsearch \ bin \ elasticsearch.bat*。

如果要将ElasticSearch用作服务，以便可以使用Windows工具启动或停止它，则需要在文件*C：\ Elasticsearch \ config \ jvm.options中*添加一行。

对于32位系统，您必须键入*-Xss320k*，对于64位系统*-Xss1m。*

更改此设置后，您必须打开命令提示符或Powershell并执行*C：\ Elasticsearch \ bin \ elasticsearch-service.bat*。可用的命令包括*安装*，*删除*，*启动*，*停止*和*管理器*。

要创建服务，我们必须输入：*C：\ Elasticsearch \ bin \ elasticsearch-service.bat install*

要管理服务，我们键入：  *C：\ Elasticsearch \ bin \ elasticsearch-service.bat管理*器，该*管理器 *打开Elastic Service Manager，这是一个GUI，可通过该GUI进行有关服务的自定义设置并管理其状态。

默认cluster.name和node.name是**elasticsearch**分别和你的主机名。如果您打算继续使用该群集或添加更多节点，则最好通过在*elasticsearch.yml*文件中对其进行修改来将这些默认值更改为唯一名称。

我们可以通过浏览http：// localhost：9200 /来验证ElasicSearch的正确执行。如果一切正常，我们将得到以下结果：

![图片](https://uploader.shimo.im/f/oSuUe7d2zVK7uf9J.png!thumbnail)

为了实现基于.NET Core的解决方案，我们使用了**NEST**软件包，可以通过以下命令安装该软件包：

```c#
dotnet add package NEST
```
NEST允许我们在索引和搜索文档以及节点和分片的管理中本地使用所有ElasticSearch功能。
为了管理NEST插件，我们创建了ElasticsearchExtensions类： 

```c#
public static class ElasticsearchExtensions
{
    public static void AddElasticsearch(this IServiceCollection services, IConfiguration configuration)
    {
        var url = configuration["elasticsearch:url"];
        var defaultIndex = configuration["elasticsearch:index"];
 
        var settings = new ConnectionSettings(new Uri(url))
            .DefaultIndex(defaultIndex);
 
        AddDefaultMappings(settings);
 
        var client = new ElasticClient(settings);
 
        services.AddSingleton(client);
 
        CreateIndex(client, defaultIndex);
    }
 
    private static void AddDefaultMappings(ConnectionSettings settings)
    {
        settings
            DefaultMappingFor<Product>(m => m
                .Ignore(p => p.Price)
                .Ignore(p => p.Quantity)
                .Ignore(p => p.Rating)
            );
    }
 
    private static void CreateIndex(IElasticClient client, string indexName)
    {
        var createIndexResponse = client.Indices.Create(indexName,
            index => index.Map<Product>(x => x.AutoMap())
        );
    }
}
```
在其中我们找到对象的配置和映射，在本例中为Product类。在此类中，我们决定忽略在索引阶段存储价格，数量和评级。
通过以下指令在Startup.cs中调用此类：

```c#
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddElasticsearch(Configuration);
}
```
这使我们能够在启动时加载的所有设置，在修改它们**elasticsearch**的第*appsettings.json*文件，在其中我们插入如下一行：
```c#
"elasticsearch": {
        "index": "products",
        "url": "http://localhost:9200/"
}
```
**索引**表示选择用来存储文档的默认索引，而**url**是我们的ElasticSearch实例的地址。
我们的产品对象定义如下：

```c#
public class Product
{
public int Id { get; set; }
public string Ean { get; set; }
public string Name { get; set; }
public string Description { get; set; }
public string Brand { get; set; }
public string Category { get; set; }
public string Price { get; set; }
public int Quantity { get; set; }
public float Rating { get; set; }
public DateTime ReleaseDate { get; set; }
}
```
如前所述，可以分别或在列表中为产品建立索引。
在我们的产品服务中，我们实现了两种方式：

```c#
public async Task SaveSingleAsync(Product product)
{
    if (_cache.Any(p => p.Id == product.Id))
    {
        await _elasticClient.UpdateAsync<Product>(product, u => u.Doc(product));
    }
    else
    {
        _cache.Add(product);
        await _elasticClient.IndexDocumentAsync(product);
    }
}
 
public async Task SaveManyAsync(Product[] products)
{
    _cache.AddRange(products);
    var result = await _elasticClient.IndexManyAsync(products);
    if (result.Errors)
    {
        // the response can be inspected for errors
        foreach (var itemWithError in result.ItemsWithErrors)
        {
            _logger.LogError("Failed to index document {0}: {1}",
                itemWithError.Id, itemWithError.Error);
        }
    }
}
 
public async Task SaveBulkAsync(Product[] products)
{
    _cache.AddRange(products);
    var result = await _elasticClient.BulkAsync(b => b.Index("products").IndexMany(products));
    if (result.Errors)
    {
        // the response can be inspected for errors
        foreach (var itemWithError in result.ItemsWithErrors)
        {
            _logger.LogError("Failed to index document {0}: {1}",
                itemWithError.Id, itemWithError.Error);
        }
    }
}
```
在这里我们使用_cache数组来进一步缓存产品列表。
对于多模式，我们也实现了批量版本，这使我们能够在更短的时间内索引大量文档，并且我们已经处理了日志插入中的任何错误。

请注意，SaveSingleAsync方法通过检查缓存数组来管理文档的插入和修改。

对于文档删除，我们实现了DeleteAsync方法：

```c#
public async Task DeleteAsync(Product product)
{
    await _elasticClient.DeleteAsync<Product>
(product);
 
    if (_cache.Contains(product))
    {
        _cache.Remove(product);
    }
}
```
GetSearchUrl方法允许我们获取用于管理页面调度的URL。
出于开发目的，我们实现了ReIndex方法，该方法允许我们删除索引上的所有文档，然后一次又一次地导入它们。这对于导入现有和未加载文档的列表很有用。

```c#
//Only for development purpose
[HttpGet("/search/reindex")]
public async Task<IActionResult>ReIndex()
{
    await _elasticClient.DeleteByQueryAsync<Product>(q => q.MatchAll());
 
    var allProducts = (await _productService.GetProducts(int.MaxValue)).ToArray();
 
    foreach (var product in allProducts)
    {
        await _elasticClient.IndexDocumentAsync(product);
    }
 
    return Ok($"{allProducts.Length} product(s) reindexed");
}
```
出于示例目的，我们创建了一个界面，该界面允许我们通过**Bogus**插件添加N个动态生成的产品，并管理产品的CRUD。
运行项目后，我们将看到以下屏幕：

![图片](https://uploader.shimo.im/f/FPxABNr9ykoPq9Xe.png!thumbnail)

例如，如果我们尝试将10种产品添加到索引中，在文本框中输入**10**，然后单击“ **导入文档”**按钮，则可以使用搜索框查看结果，也可以直接从浏览器中浏览到http页面： // localhost：9200 / products / _search，我们将在其中得到这样的结果：

![图片](https://uploader.shimo.im/f/qUPIjUmdg2if7IaE.png!thumbnail)

本文中使用的代码可[在此处获得](https://github.com/enricobencivenga/ElasticSearch)。

