---
title:  如何在我们的Asp.NET Core应用程序中使用ElasticSearch高级功能
date: 2021-4-25 18:56
tags: 技术
author: 邹溪源
categories:
  - 技术
---


在上[一篇文章中](https://www.blexin.com/en-US/Article/Blog/How-to-integrate-ElasticSearch-in-ASPNET-Core-70)，我们讨论了将ElasticSearch用作简单的全文本搜索引擎，如何快速安装和配置它以及如何将其集成到我们的.NET Web应用程序中。

今天，我们仍然要在电子商务网站中向您展示如何使用ElasticSearch的许多功能来改善搜索。

我们使用了没有嵌套类的平面Product类来轻松管理搜索，但是这种方法有很多限制。然后，我们引入了一个新的数据模型，以便任何对象都是要建模的实体。一个文档可以包含无限数量的相关字段和值（数组，简单和复杂类型），并保存为JSON文档。

我们的模型产品类别已变为：

```plain
public class Product
{
    public int Id { get; set; }
    public string Ean { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public Brand Brand { get; set; }
    public Category Category { get; set; }
    public Store Store { get; set; }
    public decimal Price { get; set; }
    public string Currency { get; set; }
    public int Quantity { get; set; }
    public float Rating { get; set; }
    public DateTime ReleaseDate { get; set; }
    public string Image { get; set; }
    public List<Review> Reviews { get; set; }
}
```
其中品牌，类别，商店评论和用户类别分别是：
```plain
public class Brand
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
}
 
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
}
 
public class Store
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
}
 
public class Review
{
    public int Id { get; set; }
    public short Rating { get; set; }
    public string Description { get; set; }
    public User User { get; set; }
}
 
public class User
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string IPAddress { get; set; }
    public GeoIp GeoIp { get; set; }
}
```
GeoIp是NEST库中用于地理数据的类。
产品索引已被简单地命名为*产品*。我们以这种方式创建和配置了它：

```plain
client.Indices.Create(“products”, index => index
    .Map<Product>(x => x.AutoMap())
    .Map<Brand>(x => x.AutoMap())
    .Map<Category>(x => x.AutoMap())
    .Map<Store>(x => x.AutoMap())
    .Map<Review>(x => x.AutoMap())
    .Map<User>(x => x.AutoMap()
        .Properties(props => props
            .Keyword(t => t.Name("fullname"))
            .Ip(t => t.Name(dv => dv.IPAddress))
            .Object<GeoIp>(t => t.Name(dv => dv.GeoIp))
        )
    )
)
```
我们专门为ElasticSearch索引创建一个名为*fullname*的新属性，用于名为*fullname*的User类*，*并定义了将要处理的地理信息。
为了使我们的产品能够在索引之前进行处理，一种有用的方法是*摄取节点*，即进行文档预处理的节点。接收节点拦截所有索引请求，甚至是批量索引请求，并将所有定义的转换应用于其内容，然后将文档发还给索引API。

必须通过以下参数在配置文件*elasticsearch.yml中*启用摄取：

`node.ingest: true`

在我们的示例中，我们使用相同的节点进行搜索和摄取，我们不需要编写代码来管理摄取节点，但是，如果我们要拥有一组专用的摄取节点，则必须配置ElasticSearch客户如下： 

```plain
var pool = new StaticConnectionPool(new [] 
{
    new Uri("http://ingestnode1:9200"),
    new Uri("http://ingestnode2:9200"),
    new Uri("http://ingestnode3:9200")
});
var settings = new ConnectionSettings(pool);
var client = new ElasticClient(settings);
```
为了对文档进行预处理，需要在建立索引之前定义一个管道，该管道指定一组能够转换该文档的过程。有许多默认过程可供使用。例如：GeoIP从IP地址获取地理信息，JSON将字符串转换为JSON对象，小写和大写，Drop删除与某些参数匹配的文档。您也可以创建自定义过程。
我们在项目中使用的管道是：

```plain
client.Ingest.PutPipeline("product-pipeline", p => p
                .Processors(ps => ps
                    .Uppercase<Brand>(s => s
                        .Field(t => t.Name)
                    )
                    .Uppercase<Category>(s => s
                        .Field(t => t.Name)
                    )
                    .Set<User>(s => s.Field("fullname")
                        .Value(s.Field(f => f.FirstName) + " " + 
                             s.Field(f => f.LastName)))
                    .GeoIp<User>(s => s
                        .Field(i => i.IPAddress)
                        .TargetField(i => i.GeoIp)
                    )
                )
            );
```
该管道处理文档，以便：
* Brand.Name和Category.Name将通过大写输入以大写形式索引；
* User.fullname将包含名字和姓氏（设置摄取）；
* User.IPAddress将成为地理定位的地理地址（GeoIp提取）。

管道以ElasticSearch集群状态保存，要使用它们，您必须在索引请求中指定管道参数，以便摄取节点知道必须使用哪个管道：

```plain
client.Bulk(b => b
    .Index("products")
    .Pipeline("product-pipeline")
    .Timeout("5m") 
    .Index<Person>(/*snip*/)
    .Index<Person>(/*snip*/)
    .Index<Person>(/*snip*/)
    .RequestConfiguration(rc => rc
        .RequestTimeout(TimeSpan.FromMinutes(5)) 
    )
);
```
通过这种方式，我们定义了索引编制过程，以便我们可以根据需要获取文档清单。
在使用创建的管道为文档建立索引之后，我们可以通过使用浏览器http：// localhost：9200 / products / _search进行访问来检查它们。我们得到类似于以下结果：

![图片](https://uploader.shimo.im/f/BilwmQvVpad7ea2q.png!thumbnail)

如上一篇文章所述，搜索过程基于文档分析。这是第一个阶段的令牌化过程（将文本分成小块，称为令牌），另一个是规范化过程（它允许您查找与不等于搜索词但足够相似以至于相关的令牌的匹配项）为搜索建立索引的文本。分析仪执行此过程。 

分析仪由三个主要部分组成：

1. 0个或多个字符过滤器
2. 1个分词器
3. 0个或多个令牌过滤器

![图片](https://uploader.shimo.im/f/dd2aDkD0aXTCSVUH.png!thumbnail)有一些默认的分析器可以使用，但是，为了根据我们的要求提高搜索的准确性，我们创建了一个自定义分析器。

定制分析器使我们能够在分析过程中控制令牌化之前对文档的任何更改，如何将其转换为令牌以及如何对其进行规范化。

这是我们的自定义分析器：

```plain
var an = new CustomAnalyzer();
an.CharFilter = new List<string>();
an.CharFilter.Add("html_strip");
an.Tokenizer = "edgeNGram";
an.Filter = new List<string>();
an.Filter.Add("standard");
an.Filter.Add("lowercase");
an.Filter.Add("stop");
 
settings.Analysis.Tokenizers.Add("edgeNGram", new Nest.EdgeNGramTokenizer
{
    MaxGram = 15,
    MinGram = 3
});
 
settings.Analysis.Analyzers.Add("product-analyzer", an);
```
我们的分析器使用标准的标记化方法，创建3至15个字符的小写标记。我们可以将分析器添加到一个或多个字段的索引中，也可以将其添加为标准分析器。
```plain
client.CreateIndex("products", c => c
    // Analyzer added only for the property Description of Product
    .AddMapping<Product>(e => e
        .MapFromAttributes()
        .Properties(p => p.String(s => s.Name(f => f.Description)
        .Analyzer("product-analyzer")))
    )
    //Analyzer added as default
        .Analysis(analysis => analysis
            .Analyzers(a => a
            .Add("default", an)
        )
    )
)
```
创建自定义分析器时，可以使用测试API对其进行测试。即使对于默认分析仪，也可以执行这些测试。
```plain
var analyzeResponse = client.Indices.Analyze(a => a
    .Tokenizer("standard")
    .Filter("lowercase", "stop")
    .Text("Lorem ipsum dolor sit amet, consectetur...")
);
```
我们还可以使用数据聚合来提供通过搜索查询聚合的数据，它基于可以组成以获得复杂聚合的简单块。聚合有不同类型，每种类型都有定义的范围和输出。
它们可以分为：

* 桶装：具有关键和标准的容器；
* 指标：根据一组文档计算的指标；
* 矩阵：在不同文档字段上进行的一系列操作，以矩阵样式生成数据；
* 管道：更多聚合的聚合。

在我们的案例中，我们使用汇总来获取品牌，类别，价格范围的产品数量。在以下示例中，我们找到了产品价格的汇总：

```plain
s => s
    .Query(...)
    .Aggregations(aggs => aggs
        .Average("average_price", avg => avg.Field(p => p.Price))
        .Max("max_price", avg => avg.Field(p => p.Price))
        .Min("min_price", avg => avg.Field(p => p.Price))
    )
```
另一个有用的聚合是根据品牌，商店或类别进行分组：
```plain
s => s
     .Query(...)
     .Aggregations(aggs => aggs
         .ValueCount("products_for_category", avg => avg.Field(p => p.Category.Name))
         .ValueCount("products_for_brand", avg => avg.Field(p => p.Brand.Name))
         .ValueCount("products_for_store", avg => avg.Field(p => p.Store.Name))
     )
```
这样，我们可以实时获取针对类别，品牌和商店的搜索产品数量。汇总数据还可以用于创建仪表板，甚至可以使用动态过滤器（类似于电子商务）来组织搜索，并且显然可以用于统计目的。
## **改善您的搜索**

如您所知，我们对任何搜索结果都有分数。等级是从0到1的数字，它确定搜索参数如何接近该结果。得分主要取决于三个参数：搜索词的频率，倒排文档的频率和字段长度。

要从得分中排除得分过低的人，我们可以使用MinScore：

```plain
s => s
     .MinScore(0.5)
     .Query(...)
```
这样，我们可以排除分数低于0.5的所有结果。
建议者允许您使用与搜索文本相似的术语来搜索ElasticSearch索引。例如，完成建议器对于自动完成很有用，它会在键入文本时引导您获得最佳和更相关的结果。该完成建议程序经过优化，可以尽快返回结果，但是它使用启用了快速查找的结构并需要资源。

在我们的案例中，我们实现了基于产品名称的自动完成方法，该方法将在搜索框中键入以下内容时被调用：

```plain
s => s
    .Query(...)
    .Suggest(su => su
        .Completion("name", cs => cs
            .Field(f => f.Name)
            .Fuzzy(f => f
                .Fuzziness(Fuzziness.Auto)
            )
            .Size(5)
        )
    )
```
更好的搜索的另一有用方法是*索引boost*。当您搜索更多索引时，可以为这些索引分配一个乘数，这样一来，一个索引的结果将比另一个显示更多。您可以将其用于商业目的，与供应商达成协议或使我们的产品脱颖而出。
索引提升的一个示例是：

```plain
s => s
    .Query(...)
    .IndicesBoost(b => b
        .Add("products-1", 1.5)
        .Add("products-2", 1)
    )
```
在此示例中，我们将乘数1.5乘以1的结果，乘以1乘以2的结果，这样乘积1的结果将被更频繁地显示。
改进搜索的另一种方法是通过一些参数对它们进行排序。就我们而言，我们可以：

```plain
s => s
    .Query()
    .Sort(ss => ss
        .Descending(SortSpecialField.Score)
        .Descending(p => p.Price)
        .Descending(p => p.ReleaseDate)
        .Ascending(SortSpecialField.DocumentIndexOrder)
    )
```
我们将评分，价格，发布日期以及最终索引顺序设置为更高的优先级。
## **运行项目**

我们的示例项目是一个.NET Core MVC WebApi应用程序，该应用程序提供一个搜索框和一个仪表板，其中的仪表板会根据键入的文本自动刷新数据。首次运行项目时，我们可以加载由Bogus插件创建的n个Product对象。还有其他伪造类可以为品牌，类别，商店，评论和用户构建随机对象。它允许您拥有一个数据库来执行我们的搜索。

```plain
var productFaker = new Faker<Product>()
    .CustomInstantiator(f => new Product())
        .RuleFor(p => p.Id, f => f.IndexFaker)
        .RuleFor(p => p.Ean, f => f.Commerce.Ean13())
        .RuleFor(p => p.Name, f => f.Commerce.ProductName())
        .RuleFor(p => p.Description, f => f.Lorem.Sentence(f.Random.Int(5, 20)))
        .RuleFor(p => p.Brand, f => f.PickRandom(brands))
        .RuleFor(p => p.Category, f => f.PickRandom(categories))
        .RuleFor(p => p.Store, f => f.PickRandom(stores))
        .RuleFor(p => p.Price, f => f.Finance.Amount(1, 1000, 2))
        .RuleFor(p => p.Currency, "€")
        .RuleFor(p => p.Quantity, f => f.Random.Int(0, 1000))
        .RuleFor(p => p.Rating, f => f.Random.Float(0, 1))
        .RuleFor(p => p.ReleaseDate, f => f.Date.Past(2))
        .RuleFor(p => p.Image, f => f.Image.PicsumUrl())
        .RuleFor(p => p.Reviews, f => reviewFaker.Generate(f.Random.Int(0, 1000))
    )
```
![图片](https://uploader.shimo.im/f/MPsAaaeIuepm2I2Y.png!thumbnail)

在页面中间，有一个仪表板，我们在其中使用了本文介绍的过滤器，分析器和方法。在顶部的搜索框中键入一些文本时，将建议相关产品，并且仪表板内容将根据搜索文本进行更新。

## **结论**

在本文中，我向您展示了如何使用Elasticsearch对复杂的实际场景进行有效的处理，分析和搜索数据。希望我对这个话题感兴趣。

[此处](https://github.com/enricobencivenga/ProductElasticSearchAdvanced)提供[了](https://github.com/enricobencivenga/ProductElasticSearchAdvanced)带有本文中使用的代码的示例项目。

