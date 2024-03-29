---
title: 那些与EF有关的错误认知
date: 2020-5-15 19:07
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# 前言

这是一个对话性的讨论，它讨论了一个严重的问题趋势，我发现在由初级团队到架构师团队的各种规模的组织中，EntityFramework的利用率都很高。这不是一个如何做的问题,这也不适合新手。如果有什么能激发您的想法，或者您对我提到的事情感到好奇，那么Google是您的朋友。这也是我的第一篇博客文章。欢迎批评。

# 历史和功能介绍（按版本）

首先，让我们简单回顾一下EF随时间推移推出的功能。这绝不是详尽无遗的，当然也不会通过对主要版本的更新列出所有内容。它只是提醒了迄今为止EF的故事。

##### EF / EF 3.5

· DB First

##### EF 4.0

· Lazy Loading

· Migrations

· POCOs

##### EF 5

· Enums

· Spatial

##### EF 6

· Async

· Interception

· Logging

· NuGet Installation

· Recovery

##### EF 7 / Core

· Code First Only

· In-Memory Support

· Limited Batching

· Nonrelational Support

看到这种零散的发布以及Microsoft在开发领域的普遍声誉，Entity Framework有了一个不好的惊喜，并不是因为它为我将要解决的问题找借口而已。根据您正在使用的次要版本，功能似乎随机出现。因此，即使安装了相同的主要版本，您仍会习惯某些东西，转到另一个环境，提出声明并在其现有框架上进行尝试，即使它没有安装，您也仍然会“告诉您”的样子有助于进一步加深已经确立的地位。

基本上，EF的典型故事如下：

高级人士：“让我们使用EF和仓储模式！”

其他开发人员：“ Idk，还没听说过好消息。”

高级：“不，太好了！看到这个例子吗？”

开发人员：“嗯，好的。”

起初，它的工作原理在可接受的范围内。然而，随着它的增长，迟缓开始出现，人们开始抱怨。由于我们行业中偿还技术债务的状况非常糟糕，或者由于完全拒绝首先查看仓储模式中的技术债务，因此，据称聪明的人的整个部门都袖手旁观，只是得出结论，`”EF是垃圾"`，并不是说他们的使用是垃圾。

我在这里要说明的是后者，并向您展示如何避免该陷阱。

# EF使Sql查询过程得以抽象

在我职业生涯的早期，我开始直接通过ADO使用经典的ASP和SQL Server。我在一个非常小的网络部门工作，因此我经常不得不亲自进入数据库来创建表和执行任务。在一系列复制/粘贴部署，生产测试等过程中，我很快熟悉了SQL Server的所有技巧。“那这个呢？*不会，该产品页面仍然无法加载。*那个怎么样？！*不，仍然无法加载。*来吧...这个？*成功！*通过几乎在黑暗中绊倒，我对索引，视图，复制，安全权限等等非常了解，当时甚至还没读完高中。

使用结构化的环境输入我的前几个地方，然后使用Entity Framework时让我非常脸红。它完全没有我习惯的任何选择。因此，我跳上船，但是没多久就开始抱怨。如果您有金鱼的记忆，让我重申我习惯于随意调整所有杠杆。遇到问题时，我会进行调查。通常，我发现诸如索引利用率之类的关键组件被完全忽略了。当我提出这些问题时，我被告知EF因不知道如何利用它们而感到过失，而我们是在这里处理业务问题，而不是由Microsoft来为他们做。老实说，我基本上还是一个初级开发人员。我有什么理由不同意呢?

## 仓储模式存在的问题

点击访问[仓储模式](http://bfy.tw/3pSK)。仓储模式的问题有两个方面。存储库模式的问题有两个方面。首先，它要求您预先声明如何绑定应用程序与数据库进行交互。即使您构建了这些超级复杂的方法，这些方法允许您传入表达式、字典或异常动态，并且您很有创造力，但是您所做的只是制造了一个维护噩梦。

“但是调用者可以定义他们所需要的！” 不，他们不能。当然，他们可以指向实体并通常定义要选择的数据的形状，但是他们无法确定字段选择之类的内容。他们没有办法说他们需要以一种友好的方式预先加载数据或延迟数据。他们不能在一次实例化中说他们也需要来自这里或那里的数据，但在下一次实例化中，他们只需要找到目标实体，除非神圣的存储库允许他们这样做。相反，您会得到这些全有或全无的决策，这些决策将您的应用程序链接在一起，我们想知道为什么它会很快降级。我真的希望你的水晶球比我的好。

其次，即使是微软自己的例子也没有使用某些实习生可能编写的适当接口。因此，我说，每个人都做错了。关于EF的常识是使用存储库模式，由于存储库模式本身的文档和示例不正确，所以没有人会让EF做它被设计用来做的事情，因为知识的来源受到了毒害。面对这种情况，我听到很多人抱怨说MVC教程的例子直接使用了DbContext，抱怨说它不够稳定，也不是说几乎没有人做得很稳定，但这是另一篇博客文章。(大多数软件直接跳转到ID，忽略了其他的。)

## 让数据库成为数据库

由于SQL Server是EF中最常用的支持数据存储，因此它不是一个干净的软件。太乱了 ，它具有大量场景的功能。如果您想让您的应用程序实际使用您所支付的巨额许可费用的一小部分，请停止将SQL约束到EF驱动的地狱荒原，而这些荒地比SELECT *还要好。然后，我们喜欢抱怨事情进展缓慢。

如果您不让EF在正确的情况下利用功能，则可能无法意识到平台的潜力。必须浪费数十亿美元的许可和开发成本，即使在应用程序以截然不同的方式增长时，使用的独特功能也只会使SQL Server的单位利用率下降。这是一种直觉，但是看到我在大型和小型公司中看到的愚蠢的朴素仓储实现，很难在这里看到我是完全错误的。这对我们自己，我们的雇主和彼此都是有害的。

实体框架仍然逐步锁定在基础数据存储的工作方式上。在SQL Server中，这意味着联接性能，视图和索引利用率，存储过程调用等。这就像将乳胶手套称为手的抽象。它不是，EF也不是它所依赖的存储机制的抽象。相反，它是一组通用的API，它们使我们能够以统一的方式访问数据。由于我刚才所说的原因（我们不能以任何方式否认或减轻基础实现的行为），这不是一个抽象。因此，我们必须在代码中考虑显式或隐式破坏抽象的那些行为。如果要假装它是抽象，我们唯一能做的就是把头埋在沙子里，然后在事情变得笨拙时继续continue吟。

最近，我遇到了—架构师—几乎对让数据库定义视图和将EF指向视图而不是表的建议感到敬畏，您知道，这让dba能够真正完成他们的工作，并使数据库能够在不破坏应用程序代码的情况下进行更改。这并不是什么难事，但问题是普遍存在的，所以大多数人在他们太熟悉的环境中都看不到过去。那么，我们该怎么做呢?

# 使用IQueryable而不是IEnumerable

正确使用Entity Framework的第一步是打破与IEnumerable的依赖关系。当谈论断开连接的商店时，这是很糟糕的。IEnumerable唯一给出的就是延迟执行。如果这是您想要脱离ORM的唯一功能，那么您就不需要ORM。IEnumerable隐瞒使用数据存储的原因在于，它们一劳永逸地固定在它们的表示中。即使应用程序增长，即使仓储中添加了新方法，返回IEnumerable的旧实现也对它们所处的新世界都是盲目，聋哑和愚蠢的。您实际上是在强迫代码与数据布局和期望一起使用。就像几年前首次实施时一样。这是开发人员的错，但应归咎于EF。

但是，IQueryable可以变形并更改为其给定的上下文。即使传递和添加了子句，它也可以评估实例，例如各个调用的需求。如果DbContext已经获取了数据，但它仍可以从高速缓存中检索实体，然后才能以非常快的速度进行重复调用，从而使热路径更加凉爽。更重要的是，它还提供了一些功能，例如，如果基础提供程序支持的话，让我们流数据；无需实例化List对象以使其对堆更友好地加载数据；检查基础类型，以便我们可以在复杂的工作流程中做出明智的决定；访问基础上下文, 等等。

这些都是使您的代码真正了解正在发生的事情而不会破坏抽象障碍的所有功能，因为EF不是抽象。顺便说一句，抽象应该是使用EF的组件，而不是EF本身。我在程序员进行的许多讨论中表达了自己的见解，这些讨论表达了一些需求，但是我们以“抽象”的名义对许多解决方案solutions之以鼻，随之而来的是我们高兴地扭曲自己的箍，这样我们就可以继续存在固体。

# 习惯匿名类型

我听说过的关于EF的最大的抱怨也许就是它检索了多少该死的数据。谁定义实体？EF？没有！你做到了 从本质上讲，您不必多怪，因为每个表的实体似乎是所有人都可以看到的。尽管如此，无论实体有多大，我们都无需受到阻碍。将匿名类型传递给EF查询将导致EF仅选择您定义的字段。可以将数十列的“无法重构”的怪物表分解为实际需要的3或4个字段。一次选择整个实体并假装无能为力的迷恋只能描述为一种大众歇斯底里的形式，我们大声疾呼，“我看不到你！”

# 使用正确的工具完成工作

您知道所有带有封面上各种工具的Microsoft Press书籍吗？您知道，除了某些人只是选择随机图像之外，还有一个原因。大多数工具不仅是螺丝起子或刨刀。有一些真正的奇怪应用没有明显的应用，但是可以肯定的是，它们有自己的目标，并且擅长于此。“正确的工具”的口号经常重复出现，但是我们并没有真正停下来思考工作，更不用说工具了。以下是EF的一些功能。

### 自.Net 2.0以来出现的SqlBulkCopy

另一个与EF数据量密切相关的大型抱怨是，EF检索到它据称无法处理大量数据的方式。我喜欢开发人员的双重性。我想让您知道，结合下面讨论的AsStreaming，反应性扩展和SqlBulkCopy，我可以在一分钟内检索，转换和推送数百万条记录，而不会费力地创建一个完全基于任何工作负载的完全基于代码的ETL解决方案从较小的记录到中等大小的记录（例如5–100亿条记录），并且仍然具有良好的性能。如果您需要更多，则有更多专用工具。但是，不要说Entity Framework无法处理大量数据。 *您的代码无法处理大量数据。*EF很好。

可悲的是，自2005年以来我们就拥有SqlBulkCopy，但我们却假装工具箱中有这个大漏洞。问题已经解决。重新发明轮子的理由为零。你猜怎么了？它也支持流！

### AsTracking与AsNoTracking

我觉得自己的成绩很差。关于EF的另一个大抱怨是它的数据缓存。您几乎总是可以告诉DbContext摆脱缓存的实体。不过，最近，我们获得了将其设置为Entity Framework Core中默认策略的能力。相反，我们可以选择要跟踪的内容，而不是不需要的内容。我很高兴地承认一个烦恼，即您仍然需要分离实体。

### 流式传输

实体框架中的查询通常在返回之前缓冲所有结果。流技术解决了这一问题，并立即让您开始处理数据进入应用程序的过程。您既可以更快地开始工作，又可以使服务器对内存更友好。

### 特殊雪花

在开发人员中，我看到了一个令人不安的趋势。缺乏探索和发明的欲望。我们想要开箱即用的解决方案，在不了解细节的情况下“可行”。即使代码不是魔术，我们仍然相信看不见的魔术。

我采用的一般方法不是构建这些固定的仓储，而是构建扩展，使我们的应用程序以我们需要它们的独特方式运行。是否希望在运行时间较长的过程中缓存数据的好处，但又不能在给定操作之外继续存在呢？对于我来说，这听起来像是DbContext的完美扩展方法，该方法可以获取一些实体，对其进行处理以获得缓存的好处，然后在返回之前清除缓存。另一种扩展方法是在操作完成后分离所有那些实体的方法。

# 不要害怕

我在这里谈论DbContext是因为有很多人对待它。它被视为一件大，笨重，笨拙的事情，如果您不小心的话，它们会偷走您的孩子。我们花了很长的时间才能使DbContext的存在只为少数几个组件所知。这将我们的实现进一步扼杀到仓储中。由于我们*必须*遍历仓储以获取任何类型的数据，因此我们需要在发生更改时定期违反“开放/关闭”原则，或者被迫接受仓储指示的决策膨胀的折衷，并且在使用时要格外小心我们打电话给它。

释放DbContext。如果模块需要数据，请不要自欺欺人，说DbContext还不是依赖项。我可以向您保证，如果您对它的可访问性感到满意，并消除“人们犯错了怎么办？！”的神秘主义。它实际上将使我们整体上变得更好。如果某人可以提交顽皮的代码，并且至少使它经过一次未经测试的生产，则您实际上就没有发布控制或质量检查。诸如隐藏DbContext之类的策略是您组织中已经流血的伤口上的权宜之计，无助于真正缓解实际问题。

# 别再找借口

我们程序员必须停止像解决我们所遇到的问题的解决方案那样行动，或者必须使用node.js和dapper的正确组合来区分它们，这并不是说它们没有合法用途，而是经常被他们当作替罪羊实体框架是一种很好的工具，可以用来做某事。我们十年来拥有的工具已经足以满足我们的大多数需求。一次又一次的错误决定最终导致错误的决定，使我们陷入困境。使您的工具适应自如。尝试新事物。可以肯定的是，我们只能怪自己。