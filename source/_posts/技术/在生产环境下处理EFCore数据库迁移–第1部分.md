# 在生产环境下处理EFCore数据库迁移–第1部分

安德鲁·洛克（Andrew Lock）撰写了精彩的系列文章《在ASP.NET Core中的应用程序启动时运行异步任务》，其中他以“迁移数据库”为例，介绍了您可以在启动时执行的操作。

在他该系列的第3部分中，他介绍了为什么在启动时迁移数据库并不总是最佳选择。我决定编写一系列有关可以安全迁移数据库的不同方法的系列文章，即使用Entity Framework Core（EF Core）更改数据库的架构。

这是本系列的第一部分，介绍如何**创建迁移**，而第二部分则介绍如何将迁移应用于数据库，特别是生产数据库。在撰写本文时，我使用了与安德鲁类似的方法，即，我尝试使用EF Core的优点和缺点，一路介绍创建迁移脚本的所有方式。

> 注意：Andrew和我彼此认识，因为我们同时在为Manning Publications撰写书籍：Andrew的书是“ ASP.NET Core in Action，而我写的书是“ Entity Framework Core in Action) ”。我们分享了当作家的辛劳和喜悦，但是安德鲁在ASP.NET Core方面的工作更加艰辛–他的书长700页，而我的书“只有” 500页。

## TL; DR –创建迁移的摘要

注意：单击链接可直接转到涵盖该点的部分。

- 可以将两种类型的迁移应用于数据库：

  - 添加新的表，列等，称为*不间断的更改*（简单）。
- 更改列/表并需要复制数据，称为*重大更改*（困难）。
  
- 有五种方法可以在EF Core中创建迁移

  - 使用EF Core创建迁移-简单，但不能处理所有可能的迁移。
  - 使用EF Core创建迁移，然后手动修改迁移-中到难，但处理所有可能的迁移。
  - 使用第三方迁移构建器来编写C＃迁移-很难，因为您需要自己编写迁移，但是您不需要了解SQL。
  - 使用SQL数据库比较工具比较数据库并输出SQL更改脚本–很简单，但是您确实需要对SQL有一定的了解。
  - 通过复制EF Core的SQL来编写自己的SQL迁移脚本 –很难理解，可以很好地控制，但您确实需要了解SQL。

- 如何确保您的迁移有效–使用CompareEfSql工具。

## 设置场景–关于创建迁移，我们应该问什么问题？

有很多迁移数据库模式的方法，在开发中，几乎可以使用任何方法。但是，当涉及到迁移生产数据库（即实际用户正在使用的数据库）时，它就变得非常严重。弄错了，至少会给您的用户带来不便，并且更糟的是，甚至会丢失您数据库中的（宝贵）数据！

在获得更新数据库模式部分之前，我们需要构建迁移脚本，该脚本将包含模式以及可能的数据更改。要构建适当的迁移脚本，我们需要问自己一些有关需要应用到数据库的更改类型的重要问题。所需的迁移将是：

1. 一个非重大更改，也就是说，它只是增加了新的栏目，表格等，这可能而旧的软件仍然运行应用，即旧的软件将与迁移后的数据库一起运行。
2. 一个重大更改，即有些数据必须复制或迁移过程中转化，无法应用，而旧的软件，即旧的软件会遇到与迁移后的数据库错误（中断服务）。

本文中也介绍了我们正在使用EF Core，它带来的一些好处和限制。好处是，在大多数情况下，EF Core可以自动创建所需的迁移。

约束条件是应用迁移后的数据库必须与EF Core通过查看您的DbContext和映射的类建立的数据库软件模型匹配–我指的是带有大写M的EF Core模型，因为存在一个名为DbContext中的模型，其中包含类和数据库之间的完整映射。

> 注意：我将介绍迁移，在这些迁移中，您可以控制映射到数据库的类的控制和EF Core配置-有时也称为代码优先方法。
>
> 我不会介绍另一种替代方法是，您直接控制数据库，并使用称为  scaffolding 的EF Core命令为您创建实体类和EF Core配置。采用这种方法迁移很简单–只需重新搭建数据库即可。

## 第1部分。创建迁移脚本的五种方法

正如我在上一节中所述，我们创建的任何迁移脚本都必须将数据库迁移到与EF Core Model匹配的状态。例如，如果迁移在表中添加了新列，则映射到该表的实体类必须具有与该新列匹配的属性。如果数据库架构的EF Core的模型确实与数据库匹配，则您可能会在查询或写入中发生错误。如果迁移脚本与该数据库的EF Core模型匹配，则将其称为创建“可用”数据库。

毫无疑问，EF Core创建的迁移的有效性– EF Core创建了它，因此它将是有效的。但是，如果我们需要编辑迁移，或者我们自己进行迁移构建，那么我们需要非常小心，就EF Core而言，迁移会创建一个“可用”数据库。这是我考虑过很多的事情。

这是创建迁移脚本的方法的列表。

- 创建C＃迁移脚本
  1. 标准EF Core迁移脚本：使用EF Core的Add-Migration命令创建C＃迁移脚本。
  2. 手动修改的EF Core迁移脚本：使用EF Core的Add-Migration命令创建C＃迁移脚本，然后对其进行手动编辑以添加EF Core遗漏的位。
  3. 使用第三方迁移构建器，例如FluentMigrator。这样的工具使您可以用C＃编写自己的迁移脚本。
- 创建SQL迁移脚本。
  1. 使用SQL数据库比较工具。它将最后一个数据库架构与EF Core创建的新数据库架构进行比较，并生成一个SQL脚本，该脚本会将旧数据库迁移到新数据库架构。
  2. 编写自己的SQL迁移脚本。称职的SQL编写者可以通过捕获SQL EF Core用来创建数据库的方式来编写SQL迁移脚本。

这是一个摘要图，可让您对这五种方法进行总体回顾，并就其易用性和局限性提出个人看法。

[![img](https://www.thereformedprogrammer.net/wp-content/uploads/2019/01/FiveTypesOfMigrations.png)](https://www.thereformedprogrammer.net/wp-content/uploads/2019/01/FiveTypesOfMigrations.png)

现在，让我们依次看一看。

### 1a。标准EF Core C＃迁移脚本

这是EF Core提供的标准迁移技术。Microsoft官方文档中提供了[充分的文档记录](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/)，总而言之，您运行了一个名为Add-Migration的命令，该命令将三个C＃文件添加到您的应用程序，其中包含使用Add-Migration 迁移现有数据库以匹配当前EF Core设置/配置所需的更改。

| 好处   | ·自动构建迁移 ·无需学习SQL ·包括还原迁移功能                 |
| ------ | ------------------------------------------------------------ |
| 坏处   |                                                              |
| 局限性 | 标准迁移无法处理重大更改（但请参见1b）。 不处理SQL功能，例如SQL用户定义的函数（但请参见1b）。 |
| 提示   | 运行“添加迁移”方法时请注意错误消息。如果EF Core检测到可能丢失数据的更改，它将输出一条错误消息，但仍会创建迁移文件。您必须更改迁移脚本，否则将丢失数据–请参阅第1b节。 ·如果您的DbContext在另一个注册了DbContext的程序集中，则需要在构建中使用MigrationsAssembly方法，并且很可能需要在DbContext程序集中实现IDesignTimeDbContextFactory。 |
| 结论   | 这是处理迁移的一种非常简单的方法，并且在许多情况下效果很好。问题是，如果迁移无法满足您的需求，将会发生什么情况。幸运的是，有很多方法可以解决这个问题。 |

#### 

参考：[Microsoft的有关创建迁移的文档](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/#create-a-migration)。

### 1b。手工修改的EF Core C＃迁移脚本

关于EF Core的Add-Migration命令的好处是，它以C＃迁移文件为起点，但是您可以自己编辑这些文件以添加代码来处理重大更改或添加/更新数据库的SQL部分。Microsoft提供了通过复制数据处理重大更改的示例。

| 好处   | 与标准迁移相同+ ·能够自定义迁移。 ·能够包含SQL功能，例如SQL用户定义的功能。 |
| ------ | ------------------------------------------------------------ |
| 坏处   | ·您需要了解数据库中正在隐藏的内容。 ·可能难以决定如何编辑文件，例如，您是否保留了EF Core的所有内容，然后对其进行了更改，还是删除了EF Core部件并自己完成了？ |
| 局限性 | 没有简单的方法来检查迁移是否正确（但请参阅稍后的CompareEfSql）。 |
| 提示   | 与标准迁移相同。                                             |
| 结论   | 非常适合进行较小的更改，但由于经常将C＃命令与SQL混合使用，因此进行较大的更改可能很困难。这就是为什么我不使用EF Core迁移的原因之一。 |

参考：[Microsoft手动修改迁移的示例](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/#customize-migration-code)。

### 1c。使用第三方C＃迁移构建器

安德鲁·洛克（Andrew Lock）向我指出了一种使用FluentMigrator编写迁移的方法。这与EF迁移的工作原理类似，但是您必须完成详细说明更改的所有艰苦工作。好消息是FluentMigrator的命令非常明显。

| 好处       | 不需要学习SQL。 能明显的看到更改了什么，即“代码作为文档”。   |
| ---------- | ------------------------------------------------------------ |
| 坏处       | ·您必须确定自己所做的更改。 不保证产生“正确的”迁移（但请参阅稍后的CompareEfSql）。 |
| **局限性** | - 没有 -                                                     |
| **提示**   | 请注意，FluentMigrator有一个“ Migration Runners”，可以将更新应用于数据库，但也可以输出SQL脚本。 |
| 结论       | 我自己没有真正的经验。感觉这是EF Core迁移的一种更清晰的语法，但是您必须自己完成所有工作。 |

参考：[GitHub的FluentMigrator](https://github.com/fluentmigrator/fluentmigrator)。

### 2a。使用SQL数据库比较工具

[![img](https://www.thereformedprogrammer.net/wp-content/uploads/2019/01/SQLServerObjectExplorerCompareSchema.png)](https://www.thereformedprogrammer.net/wp-content/uploads/2019/01/SQLServerObjectExplorerCompareSchema.png)

有免费的和商业的工具可以比较两个数据库并创建一个SQL更改脚本，该脚本将旧数据库架构迁移到新数据库架构。

Visual Studio 2017（所有版本）中的“视图”选项卡下内置了一个名为“ *SQL Server Object Explorer* ”的“免费”比较工具。如果右键单击数据库，则可以访问“比较模式”工具（请参见右图），该工具可以生成SQL更改脚本。

在*SQL Server的对象资源管理器*工具是非常好的，但是没有这个（可惜）多文档。其他商业系统包括Redgate的SQL Compare。

| 好处       | 为您构建正确的SQL迁移脚本。                                  |
| ---------- | ------------------------------------------------------------ |
| **坏处**   | ·您需要对数据库有一点了解。 ·并非所有的SQL比较工具都生成还原脚本。 |
| **局限性** | 不处理重大更改-需要人工输入。                                |
| **提示**   | 请注意SQL比较工具，该工具可以输出日光下的所有设置，以确保设置正确。EF Core的迁移非常简单，例如“ CREATE TABLE…”，因此应该这样做。如果您有任何特定设置，则将它们构建到数据库create中。 |
| **结论**   | 我在难以手动编码的大型迁移中使用了SQL Server对象资源管理器。对不熟悉SQL语言的人非常有用，尤其有用。 |

### 2b。手工编码SQL迁移脚本

这听起来确实很困难-编写自己的SQL迁移，但是手头上有很多帮助，无论是来自SQL比较工具（参见上文），还是查看SQL EF Core用于创建数据库的帮助。这意味着我可以查看并复制以构建结论SQL迁移脚本的SQL。

| **好处**   | 完全控制数据库结构，包括EF Core不会添加的部分，例如用户定义的函数，列约束等。 |
| ---------- | ------------------------------------------------------------ |
| **坏处**   | ·您必须了解基本的SQL，如CREATE TABLE等。 ·您必须确定自己所做的更改（但有帮助） ·不能进行自动还原迁移。 ·不保证产生“正确的”迁移（但请参阅稍后的CompareEfSql）。 |
| **局限性** | - 没有 -                                                     |
| **提示**   | ·我使用一个单元测试来捕获EF Core的确保创建方法的日志输出。那让我得到了实际的SQL EF Core输出。然后，我寻找最后一个数据库的差异。这使得编写SQL迁移更加容易。   ·通过应用所有迁移（包括新迁移）创建数据库，然后运行CompareEfSql来检查数据库是否与EF Core的当前数据库模型匹配，从而对迁移进行单元测试。 |
| **结论**   | 这是我使用的，在CompareEfSql工具的帮助下。如果EF Core的迁移功能非常好，为什么还要处理所有这些麻烦呢？这是结论原因： ·完全控制数据库结构，包括EF Core不会添加的部分，例如用户定义的函数，列约束等。 ·由于我正在编写SQL，因此使我考虑了数据库的各个方面。更改–该属性是否可以为空？我需要索引吗？等 ·通过手动修改EF Core的迁移系统来应对重大变化并非易事。我还是坚持使用SQL迁移。 这是针对想要完全控制和可视化迁移的开发人员的。 |

您可以捕获EF Core的SQL输出以创建数据库，但是可以在调用方法 EnsureCreated  （ EnsureCreated  方法用于创建单元测试数据库）时捕获EF Core的日志记录。因为为EF Core设置日志记录有些复杂，所以我在EfCore.TestSupport库中添加了辅助方法来处理该问题。这是一个示例单元测试，它创建一个新的SQL数据库并捕获EF Core生成的SQL命令。

```c#
[RunnableInDebugOnly]
public void CaptureSqlEfCoreCreatesDatabaseToConsole()
{
    //SETUP
    var options = this.CreateUniqueClassOptionsWithLogging<BookContext>(
        log => _output.WriteLine(log.Message));
    using (var context = new BookContext(options))
    {
 
        //ATTEMPT
        context.Database.EnsureDeleted();
        context.Database.EnsureCreated();
    }
}
```

让我们看一下这段代码的每一行

- 第5行。这是一个EfCore.TestSupport方法，为您的DbContext创建选项。此版本使用包含类名的数据库名称。我这样做是因为xUnit测试类是并行运行的，所以我想要此单元测试类的唯一数据库。
- 第6行。我使用以... WithLogging结尾的选项生成器的版本，该版本允许我捕获日志输出。在这种情况下，我将日志的Message部分直接输出到单元测试输出窗口。
- 第11和12行。首先，我确保删除数据库，以便在我调用确保创建时，将使用由当前DbContext的配置和映射的类定义的架构来创建一个新的数据库。

以下是在单元测试输出中捕获的部分输出。这为您提供了EF Core用于创建整个架构的确切SQL。您确实只需要提取与迁移有关的部分，但是至少您可以将所需的部分剪切并粘贴到SQL迁移脚本中。

```sql
CREATE DATABASE [EfCore.TestSupport-Test_TestEfLogging];
Executed DbCommand (52ms) [Parameters=[], CommandType='Text', CommandTimeout='60']
IF SERVERPROPERTY('EngineEdition') <> 5
BEGIN
    ALTER DATABASE [EfCore.TestSupport-Test_TestEfLogging] SET READ_COMMITTED_SNAPSHOT ON;
END;
Executed DbCommand (5ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE [Authors] (
    [AuthorId] int NOT NULL IDENTITY,
    [Name] nvarchar(100) NOT NULL,
    CONSTRAINT [PK_Authors] PRIMARY KEY ([AuthorId])
);
Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
CREATE TABLE [Books] (
    [BookId] int NOT NULL IDENTITY,
    [Title] nvarchar(256) NOT NULL,
-- rest of SQL left out
```

## 如何确保您的迁移有效–使用CompareEfSql工具

在创建迁移的描述中，我曾多次提到CompareEfSql。该工具将数据库与EF Core首次用于DbContext时创建的数据库模型进行比较。通过DbContext实例中的Model属性访问此模型，是通过查看DbContext配置以及DbSet和DbQuery属性来构建结论EF Core的。

这使开发人员可以根据EF Core Model测试现有数据库，并在错误消息不同的情况下为您提供错误消息。我发现这是一个非常强大的工具，它使我可以手动编码SQL迁移，并确保它们是正确的有一些小限制。这是一个示例单元测试，如果数据库架构与EF Core的模型不匹配，该测试将失败。

```c#
[Fact]
public void CompareViaContext()
{
    //SETUP
    var options = … options that point to the database to check;
    using (var context = new BookContext(options))
    {
        var comparer = new CompareEfSql();
 
        //ATTEMPT
        //This will compare EF Core model of the database 
        //with the database that the context's connection points to
        var hasErrors = comparer.CompareEfWithDb(context);
 
        //VERIFY
        //The CompareEfWithDb method returns true if there were errors.
        //The comparer.GetAllErrors property returns a string
        //where each error is on a separate line
        hasErrors.ShouldBeFalse(comparer.GetAllErrors);
    }
}
```

我喜欢这个工具，它位于EFCore.TestSupport开源库中。它使我能够构建迁移，并确保它们能够正常工作。我也将其作为正常的单元测试来运行，它会立即告诉我是否是我或另一个同事更改了EF Core的设置。

您可以在名为EF Core的文章中获得对该工具的更详细的描述：完全控制数据库模式及其许多功能和配置可以在[CompareEfSql文档页面中找到](https://github.com/JonPSmith/EfCore.TestSupport/wiki/9.-EfSchemaCompare)。

> 注意：我最初是为EF6.x构建此版本的（请参阅此旧文章），但是由于EF6.x并未完全公开其内部模型而受到限制。
>
> 有了EF Core，我可以做更多的事情，现在我可以检查几乎所有内容，并且因为我利用了EF Core的脚手架服务，所以它适用于EF Core支持的任何数据库。

## 结论–第1部分

本系列的这一部分将介绍如何创建有效的迁移，而第二部分则涉及将迁移应用于数据库。本文列出了使用EF Core时用于创建数据库迁移的所有适用方法-优缺点。如您所见，EF Core的Add-Migration命令确实很好，但是并不能涵盖所有情况。

由您决定要遇到的迁移类型，以及您希望对数据库架构进行何种级别的控制。如果您仅使用EF Core的标准迁移（1a）就可以摆脱困境，那么这将使您的生活更轻松。但是，如果您预期会发生重大变化，或者需要设置额外的SQL功能，那么您现在知道可用的选项。

令人担心的部分出现在part2中-将迁移应用于生产数据库。更改包含关键业务数据需求（需求！）的数据库，请仔细计划和测试。您需要考虑如果（何时！）迁移因错误而失败时该怎么办。

我放弃EF6中的EF迁移的最初原因是它在启动时自动迁移运行良好，直到它在部署时引发错误！确实很难找到迁移中的错误-仅此一项就使我远离使用EF迁移（那时候回想起了这篇[老文章](https://www.thereformedprogrammer.net/handling-entity-framework-database-migrations-in-production-part-1-applying-the-updates/)）。

EF Core的迁移处理要比EF6更好：已可以实现自动迁移（感激！），并且EF Core迁移对git-merge更加友好，仅提及两个更改。但是，我构建SQL迁移脚本的方式使我比正在运行Add-Migration时要更加仔细地思考自己在做什么。EF Core是一个非常出色的O / RM，但有时确实有许多隐藏功能。

创建SQL迁移脚本使我从数据库的角度考虑了迁移问题，而且我经常会对数据库和C＃代码的一些细微调整，以使数据库更好的运行。