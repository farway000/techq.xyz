---
title:  如何使用ABP进行软件开发（2）基础概览
date: 2020-7-7 21:35
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# 简述

上一篇简述了ABP框架中的一些基础理论，包括ABP前后端项目的分层结构，以及后端项目中涉及到的知识点，例如DTO,应用服务层，整洁架构，领域对象（如实体，聚合，值对象）等。

笔者也曾经提到，ABP依赖于领域驱动设计这门方法论，由于其门槛较高，给使用者带来了不少理解上的难度。尤其是三层架构对.NET开发者影响太深，有时很难对领域驱动设计产生直观的理解。

在本文中，打算从传统的简单三层架构谈起，介绍一个实际场景下的三层业务逻辑实现，然后再与领域驱动设计中的对应实现形成对比，以便让开发者形成直观具体的印象。

# 回顾三层架构

对于.NET开发者来说，三层架构相比都不陌生，这种架构，将代码层次划分为用户界面层，业务逻辑层、数据访问层三个逻辑层次，实现了代码的关注度分离，且因其易于理解，已经成为众多.NET开发者的"条件反射"。

## 三层架构简介

三层架构就是为了符合“高内聚，低耦合”思想，把各个功能模块划分为表示层（UI）、业务逻辑层（BLL）和数据访问层（DAL）三层架构，各层之间采用接口相互访问，并通过对象模型的实体类（Model）作为数据传递的载体，不同的对象模型的实体类一般对应于数据库的不同表，实体类的属性与数据库表的字段名一致。

三层架构区分层次的目的是为了 “高内聚，低耦合”。开发人员分工更明确，将精力更专注于应用系统核心业务逻辑的分析、设计和开发，加快项目的进度，提高了开发效率，有利于项目的更新和维护工作。

## 三层架构的分层逻辑

UI层：用户界面层，实现与UI交互有关的逻辑。用于输入用户数据，输出和呈现数据。在基于WebAPI的现代Web框架中，往往会使用MVC架构，将界面的数据行为，拆分成“模型-视图-控制器”，实现了针对对UI层上关注度的进一步分离，

业务逻辑层： 业务逻辑层是用户界面层和数据访问层之间的转换层，负责完成对数据的业务组装，界面数据处理，将数据层的对象输出（转换）给用户界面层。

数据访问层：实现数据的存储（持久化）操作，包括集成存储过程，集成SQL语句，或集成现代ORM组件的形式，实现实现数据的存储。  

![图片](https://uploader.shimo.im/f/6J4vUgMZ1T8BvhSF.png!thumbnail)

## 三层架构的应用

遇到项目，先从实体关系建模开始，使用PowerDesign或其他数据库设计软件分析业务与业务之间的关系，是一对多，还是一对一，还是多对多，绘制实体关系图。

![图片](https://uploader.shimo.im/f/XcWg36URN8u9swFi.png!thumbnail)

在进行软件开发时，根据数据需求，定制想要的数据接口，从而实现以数据为核心的业务功能开发。

于是，在业务层次上，这种三层架构，进一步可以表示为如下分层结构：

![图片](https://uploader.shimo.im/f/mQXxSnwF2UYev8fT.png!thumbnail)

在三层架构中，实体是业务的核心，所有的业务代码，都是围绕实体展开，而左侧三个功能层，其主要目的都是为了实现对实体的“增删改查”操作。

以下代码简述了一个订单对象提交的全过程。（模型和代码仅供参考，不能直接运行）

![图片](https://uploader.shimo.im/f/KYBDBV3sYQFwoscd.png!thumbnail)

```c#
/// <summary>
/// UI控制器
/// </summary>
public class OrderController
{
  private OrderBll OrderBll = new OrderBll();
  /// <summary>
  /// 新增订单
  /// </summary>
  /// <param name="userId"></param>
  /// <param name="productId"></param>
  /// <param name="count"></param>
  public void AddOrder(int userId, int productId, int count)
  {
      OrderBll.AddOrder(userId, productId, count);
  }
}

/// <summary>
/// 业务逻辑层
/// </summary>
public class OrderBll
{
  private UserInfoDal userInfoDal = new UserInfoDal();
  private ProductInfoDal productInfoDal = new ProductInfoDal();
  private OrderDal orderDal = new OrderDal();
  /// <summary>
  /// 新增订单
  /// </summary>
  /// <param name="userId"></param>
  /// <param name="productId"></param>
  /// <param name="count"></param>
  public void CreateOrder(int userId, int productId, int count)
  {
      UserInfo userInfo = userInfoDal.Get(userId);
      ProductInfo productInfo = productInfoDal.Get(productId);
      //新订单
      Order order = new Order();
      order.Address = userInfo.Address;
      order.UserId = userId;
      order.TotalPrice = productInfo.Price * count;
      order.ProductId = productId;
      orderDal.Insert(order);
  }
}
/// <summary>
/// 数据访问层
/// </summary>
public class OrderDal
{
    /// <summary>
    /// 插入数据
    /// </summary>
    /// <param name="order"></param>
    public void Insert(Order order)
    { 
    }
}
```
这种基于实体驱动建模的三层架构，变成了以数据为核心的“表模块模式”。
参见《企业应用架构模式》第87页中关于表模块的介绍：

>表模块以一个类对应数据库中的一个表来组织领域逻辑，而且使用单一的类实体来包含将对数据进行的各种操作程序。
>通常，表模块会与面向表的后端数据结构一起使用。以列表形式排列的数据通常是某个SQL调用的结果，它们被至于一个记录集中，用于某一个SQL表。表模块提供了一个明确的基于方法的接口对数据进行操作。
>要进行一些实际的操作，一般需要多个表模块的行为。
>表模块中的“表”一词，暗示你数据库中的每一个表对应一个表模块。虽然大多数情况下都是如此，但也并非绝对。对于通用的视图或其他查询，建立一个表模块也是有用的。事实上，表模块的结构并非真的取决于数据库表的结构，更多的是由应用程序能识别的虚拟表所标识，例如视图或查询。

在《Microsoft.NET企业级架构设计》一书中，作者认为“多数.NET开发者在成长的过程中都受到了表模块模式的影响”。而相比之下，多数Java开发者则“深陷事务脚本的泥足”。

## 三层架构的优缺点

#### 优点：

软件分层架构的目的是为了分离关注点，三层架构也同样如此，简简单单的三层代码+ER图，就能设计出一个良好结构的软件系统。

这种模式，建立了以数据库表为核心的开发模式，使得开发者能够很便捷的对业务进行分析，进而驱动软件功能的快速开发。

在应对简单业务变迁过程中，由于能够快速完成代码的堆积，也使得开发者只需关注数据库表的拼凑，就能快速的完成代码开发，为开发项目带来了不少便利。   

除了简单业务普遍采用三层，事实上许多复杂项目也会同样采用，大概是由于三层架构的思想已经深入人心，许多资深开发者都形成的只要有表就能完成项目的开发的思维定势。

#### 缺点：

还是使用上述示例代码，我们假设需求发生了变化，要求减少订单的数量或增加订单，我们会怎么做？也许，我们很容易就写出了下面的代码：

>（当然，实际项目中，如果订单已经提交，很少会直接对订单数量进行修改的，往往会重新发起新订单，但为了演示方便，我们先设定有这么一个奇怪的需求吧。）
```plain
/// <summary>
/// 减少订单数量
/// </summary>
/// <param name="orderId"></param>
/// <param name="minusCount"></param>
public void MinusOrder(int orderId, int minusCount)
{
    Order order = orderDal.Get(orderId);
    order.Count -= minusCount;
    order.TotalPrice -= order.Price * minusCount;
    orderDal.Update(order);
}
/// <summary>
/// 增加订单数量
/// </summary>
/// <param name="orderId"></param>
/// <param name="minusCount"></param>
public void AddOrder(int orderId, int addCount)
{
    Order order = orderDal.Get(orderId);
    order.Count += addCount;
    order.TotalPrice += order.Price * addCount;
    orderDal.Update(order);
}
```
这个代码写起来非常快，因为只是新增了两小段代码逻辑，而从减少订单，到新增订单，只是加法和减法的区别，自然而然就更快了。
但是，速度快，一定是优点么？如果需求继续持续不断的累积呢。

例如，我们要修改订单收货人，收货地址，修改订单价格，是不是我们这种代码逻辑会越来越多，而且不同的业务逻辑互相搅合，使得后期的维护变得越来越困难？

所以，笔者认为，三层架构的缺点，就是前期开发速度太快，由于缺乏设计思想和设计模式的参与，太容易导致异味、垃圾代码、重复代码等问题产生。

所有这些问题，最终都被归类于“技术债”的范畴。

>详见维基百科。
>技术债：指开发人员为了加速软件开发，在应该采用最佳方案时进行了妥协，改用了短期内能加速软件开发的方案，从而在未来给自己带来的额外开发负担。
>这种技术上的选择，就像一笔债务一样，虽然眼前看起来可以得到好处，但必须在未来偿还。
>软件工程师必须付出额外的时间和精力持续修复之前的妥协所造成的问题及副作用，或是进行重构，把架构改善为最佳实现方式。
# 回顾领域驱动设计

## 领域驱动设计简介

领域驱动设计思想来源于埃里克埃文斯在2002年前后出版的技术书籍《领域驱动设计·软件系统复杂性核心应对之道》，在这本书中，作者介绍了领域驱动设计相关的核心模式，例如：统一语言，模型驱动设计，领域实体，聚合，值对象，仓储，限界上下文等模式。 

随着微服务的不断兴起，领域驱动设计也越来越受到互联网人的广泛追捧，在许多不同的行业应用实践过程中，已经逐渐扮演了非常基础的作用。无论是微服务架构下的服务粒度拆分，或者甚至是中台应用，以及传统的单体应用，都可以利用领域驱动设计思想下提供的模式，为应用程序的开发插上想象的翅膀。

## 领域驱动设计的分层逻辑

在上一篇博客中，我们也介绍了领域驱动设计思想分层逻辑结构，共划分为如下四个层次：

![图片](https://uploader.shimo.im/f/MvFZ7RdoHuzyB5o4.png!thumbnail)

* 用户界面层（或者表示层）：负责向用户显示信息和解释用户指令。这里的用户，既可以是使用用户界面的人，也可以是另外一个计算机系统。
* 应用层：定义软件要完成的任务，并且只会表达领域概念的对象来解决问题。这一层实际上负责的是系统与应用层进行交互的必要渠道。
* 领域层：负责表达业务概念、业务状态信息以及业务规则。尽管技术细节由基础设施层实现，但业务情况状态的反映则需要有领域层进行控制。领域层是业务软件的核心。
* 基础设施层：为上面各层提供通用的技术能力：为应用层传递消息，为领域层提供持久化机制，为用户界面层绘制屏幕组件等等。基础设施层还能够通过架构框架来支持4个层次间的交互模式。
## 领域驱动设计的应用步骤

### 1）形成统一语言

统一语言是围绕产品展开的一系列流程，方案，术语和名词解释及匹配的注释。在领域驱动设计为每个应用设计成体系的【统一语言】是核心要点。

统一语言的形成是团队成员协同参与，围绕不同的需求，达成一致性理解的过程。

形成统一语言有时需要领域专家的参与，但有时可能难以达到这个条件，用需求代言人也同样能够满足这个条件。

### 2）使用UML建模和画图

1. 建模的必要性

在我们工作过程中模型无处不在，不管是在纸上绘制的简单模型，或者使用专业软件绘制的各种模型，都是模型。领域驱动设计本身，依然依赖于模型驱动设计。

学会建模对于广大开发者来说，都是一项基本技能，当然也是众多最弱技能中的一种，因为广泛依赖于实体关系建模的思维模式，使得开发者已经很难形成有效的模型设计思想，代码也越来越趋于【过程化】。 

有时开发者甚至连实体关系建模这个步骤都会省略，直接使用Code First或甚至数据库开始建表，这样看起来速度非常快，但是太容易翻车了。

在团队协作项目中，没有良好的模型，仅凭高级开发者或有经验开发者的“”一面之词”进行设计，几乎很难完成一个复杂项目。

而uml统一建模语言也是这样的良好工具。

2. 使用哪些模型

笔者曾经有幸请教国内.NET技术圈拥有多年DDD实践经验的阿里技术专家，汤雪华老师，他指出：

>采用实体关系建模很容易看出对象与对象的关系，但仅此而已。数据并非对象，数据也无法看出行为，如果要依托实体关系建模来构建系统，往往需要开发者发挥自己的主观抽象思维，根据客户提供的资料或可用的原型，自行思考代码的逻辑实现。
>但显然，具备优秀逻辑思维能力和设计思想的开发者凤毛麟角，仅凭ER图，代码写出来往往很糟糕。

他认为，采用领域驱动设计，产品架构图，系统架构图，领域模型图，类图，关键业务场景的交互时序图，这些是必不可少的。

* 产品架构图：列出产品功能，表现出产品模块间的相关性。

![图片](https://uploader.shimo.im/f/cgPXlNojcXUjifDA.png!thumbnail)

图来自[http://www.woshipm.com/pmd/1065960.html](http://www.woshipm.com/pmd/1065960.html)

* 系统架构图：从技术层面列出系统模块组成关系。

![图片](https://uploader.shimo.im/f/jq0L1Ukk68gw8PuH.png!thumbnail)

原图来自互联网

* 领域模型关系图：反映出各领域模型间的相关性，限界上下文，聚合，和聚合根。

![图片](https://uploader.shimo.im/f/7okL3aEpRznm9y7S.png!thumbnail)

来自[https://102.alibaba.com/detail?id=174](https://102.alibaba.com/detail?id=174)

3. 如何建模？

如果说代码语言是为了与其他开发者进行沟通交流，那我们建立的各种软件设计模型将极大的方便不同领域的人员进行交流。建模也可以称之为语言的一部分。利用uml建立类图，是一种可以比较易于接受的方式。我们可以采用以下手段来建立领域模型。 

1)建立一个与实现绑定的模型。初版的模型也许很简陋，但是它可以成为一个基础，然后在后期逐渐完善。 

2)建立一种基于模型的通用语言或表达形式和机制。通过通用语言让参与项目的所有人理解模型。 

3)开发一个蕴含丰富知识的模型。模型不是单纯的数据结构，它更是各类知识的聚合体。 

4)提炼模型，模型应该能在项目过程中动态改变，发现新的概念就加进来，过时的概念就适时移除，避免臃肿。 

5)头脑风暴和实验。模型在于实践和应用，它需要项目参与者共同的努力，而头脑风暴是发挥集体智慧的良好方式。对模型进行实验或者进行场景的模拟，有利于让模型更符合需求。 

当然，对于领域专家而言，不同类型的模型也许无法理解，例如类图可能过于复杂，可以使用画图的形式，通过解释性的图形，甚至纸面上的图，更能直观的表现出领域的逻辑层次。

![图片](https://uploader.shimo.im/f/6FguSRNf353OrNWy.png!thumbnail)

这张来自TW分享的一张图，就是一个基于.NET MVC的产品设计UML设计图。

建模也并非这篇博客所能讲清楚的，包括笔者自己，也只是偶尔设计过用例图，时序图和类图，可能需要在后期系统的学习一下。

### 3）代码实现

回到最开始的那个三层架构下的代码示例，如果采用领域驱动设计，大概如下图所示：

# ![图片](https://uploader.shimo.im/f/Wj3fmmFuREdF29uZ.png!thumbnail)

回到开始那个示例代码，如果采用DDD的代码实现，大概是这样的：

```plain
/// <summary>
/// 应用服务层
/// </summary>
public class OrderAppService
{
    private OrderRepository _orderRepository;
    private UserInfoRepository _userInfoRepository;
    private ProductInfoRepository _productInfoRepository;
    public OrderAppService(OrderRepository orderRepository, UserInfoRepository userInfoRepository, ProductInfoRepository productInfoRepository)
    {
        _orderRepository = orderRepository;
        _userInfoRepository = userInfoRepository;
        _productInfoRepository = productInfoRepository; 
    }
  /// <summary>
  /// 新增订单
        /// </summary>
        /// <param name="userId"></param>
        /// <param name="productId"></param>
        /// <param name="count"></param>
        public void CreateOrder(int userId, int productId, int count)
        {
              UserInfo userInfo = _userInfoRepository.Get(userId);
            ProductInfo productInfo = _productInfoRepository.Get(productId);
            if (userInfo != null && productInfo != null)
            {
                //新订单
                Order order = Order.CreateOrder(productId, userInfo.Address, userId, productInfo.Price, count);
                _orderRepository.Insert(order);
            }
        }
        /// <summary>
        /// 减少订单数量
        /// </summary>
        /// <param name="orderId"></param>
        /// <param name="minusCount"></param>
        public void MinusOrder(int orderId, int minusCount)
        {
            Order order = _orderRepository.Get(orderId);
            order.Minus(minusCount);
            _orderRepository.Update(order);
        }
        /// <summary>
        /// 增加订单数量
        /// </summary>
        /// <param name="orderId"></param>
        /// <param name="minusCount"></param>
        public void AddOrder(int orderId, int addCount)
        {
            Order order = _orderRepository.Get(orderId);
            order.Add(addCount);
            _orderRepository.Update(order);
        }
}
    /// <summary>
    ///订单对象 
    /// </summary>
    public class Order
    {
        /// <summary>
        /// 主键
        /// </summary>
        public int Id { get; protected set; }
        /// <summary>
        /// 地址
        /// </summary>
        public string Address { get; protected set; }
        /// <summary>
        /// 用户id
        /// </summary>
        public int UserId { get; protected set; }
        /// <summary>
        /// 产品id
        /// </summary>
        public int ProductId { get; protected set; }
        /// <summary>
        /// 数量
        /// </summary>
        public int Count { get; protected set; }

        /// <summary>
        /// 单价
        /// </summary>
        public double Price { get; protected set; }
        /// <summary>
        /// 总价
        /// </summary>
        public double TotalPrice { get; protected set; }
        /// <summary>
        /// 创建订单
        /// </summary>
        /// <param name="productId"></param>
        /// <param name="address"></param>
        /// <param name="userId"></param>
        /// <param name="price"></param>
        /// <param name="count"></param>
        /// <returns></returns>
        public static Order CreateOrder(int productId, string address, int userId, double price, int count)
        {
            return new Order()
            {
                Address = address,
                UserId = userId,
                TotalPrice = price * count,
                ProductId = productId,
            };
        }
        /// <summary>
        /// 新增
        /// </summary>
        /// <param name="count"></param>
        public void Add(int count)
        {
        }
        /// <summary>
        /// 减少
        /// </summary>
        /// <param name="count"></param>
        public void Minus(int count)
        {
        }

    }
```
这段代码，最主要的变化是如下几点：
1. 引入领域模型，在三层架构的示例代码中，我们建立了如下模型：

![图片](https://uploader.shimo.im/f/NuDIoVfptngGcZcf.png!thumbnail)

这个模型是当我们Entity Framework脚本时生成的实体模型，在业内通常称其为“贫血模型”。对人类来说，红细胞负责把氧气输送到组织细胞，然后新陈代谢，产生ATP，产生动力。

而“贫血模型”这个术语恰如其份的表现出这类模型虽然还能有效的工作，但是需要由其他对象来驱动其完成动作的含义。

领域模型与贫血模型相比，更关注对象的行为，而关注行为的目的是创建带有公共接口并与在现实世界观察到的实体相似的对象，使得依照统一语言的名字和规则进行建模变得更加容易。

2. 将原来的Order对象抽象化建模为一个DDD实体。DDD实体是一个包含数据（属性）和行为（方法）的POCO对象。

在《Microsoft .NET企业级应用架构实战》书第9.2.2中指出了领域实体的特点：

>定义明确的身份标识。
>通过公共和非公共方法表示行为。
>通过只读属性暴露状态。
>限制基元类型的使用，使用值对象代替。
>工厂方法优于多个构造函数。
3. 私有set或protected set：

在示例代码中将Order中的所有属性设置为

```plain
public double TotalPrice { get; protected set; }
```
这样的目的是为了避免对该属性的随意更改，使得开发者在对属性进行操作过程中，多了一个环节，即需要谨慎思考这样的代码修改，从行为角度来分析是否符合业务需要。
在设计时实体时，开放set可能会带来严重的副作用，例如影响实体的状态。

在张逸老师的《[领域驱动设计实战，战术篇第15课](https://gitbook.cn/gitchat/column/5cbed2f6f00736695f3a8699/topic/5cd3fe04e30c87051ad3d110)》中，作者指出：

>对象之间若要默契配合，形成良好的协作关系，就需要通过行为进行协作，而不是让参与协作的对象成为数据的提供者。
>《ThoughtWorks 软件开发沉思录》中的“对象健身操”提出了优秀软件设计的九条规则，其中最后一条提出：不使用任何 Getter/Setter/Property。
>作者 Jeff Bay 认为：“如果可以从对象之外随便询问实例变量的值，那么行为与数据就不可能被封装到一处。在严格的封装边界背后，真正的动机是迫使程序员在完成编码之后，一定有为这段代码的行为找到一个适合的位置，确保它在对象模型中的唯一性。”

当然，在实际开发过程中，有时并不一定把get方法也设置为protected，毕竟有时候还需要有所妥协。

4. 将创建方法从业务逻辑层，移动到了领域对象Order中的静态工厂方法。
```plain
/// <summary>
/// 创建订单
/// </summary>
/// <param name="productId"></param>
/// <param name="address"></param>
/// <param name="userId"></param>
/// <param name="price"></param>
/// <param name="count"></param>
/// <returns></returns>
public static Order CreateOrder(int productId, string address, int userId, double price, int count)
{        
    return new Order()
    {
        Address = address,
        UserId = userId,
        TotalPrice = price * count,
        ProductId = productId,
    };
}
```
创建过程应该是一个非常严谨的过程，而原来在业务逻辑层中初始化对象的方法，随意性比较高，很容易就出现开发者在创建过程中将无关属性赋值的现象。
但如果把创建过程改成使用构造方法，又可能会造成可读性问题，而使用工厂方法，并创建一个受保护的构造方法则不会造成这个担忧。

5. 将订单新增内容和减少内容从业务逻辑层移动到了领域对象上，并封装为方法。采用迪米卡法则，只暴露最小的参数，每次只对最该赋值的属性进行操作，也容易约束开发者的操作。
```plain
/// <summary>
/// 新增
/// </summary>
/// <param name="count"></param>
public void Add(int count)
{
}

/// <summary>
/// 减少
/// </summary>
/// <param name="count"></param>
public void Minus(int count)
{

}
```
大概修改过程是最容易造成领域知识丢失的地方，而通过封装为方法，使得这个过程得以以受控的形式进行，有助于让其他开发者通过暴露的方法。
但这样做要确保所使用的命名规范符合统一语言，否则会重蹈贫血模型的覆辙。当然，在领域设计中，经常会纠结于哪些行为应该放在领域对象中，可以参考这样的规则：

* 如果方法只处理实体的成员，它可能属于这个实体。
* 如果方法访问相同聚合的其他实体或值对象，它可能属于聚合根。
* 如何方法里的代码需要查询或更新持久层，或者需要用到实体（或聚合）边界以外的引用，它属于领域服务方法。
# 对比分析

## 二者的对比

笔者整理了一个简单的图表来表现二者的对比关系。显然，三层架构并非毫无优势，领域驱动设计也并非银弹。  

  

|     | 三层架构   | 领域驱动设计   | 
|:----|:----|:----|
| 业务识别方法   | 结合瀑布模型，通过需求分析，形成数据字典，指导数据库设计。   | 团队协作形成统一语言，并从统一语言中提取术语，指导类、流程，变量，行为定义等。   | 
| 业务参与者   | 具备IT知识的开发人员，业务人员只能提供需求，往往不能参与设计过程。   | 由需求提供者或客户、开发者、测试、产品经理等组成的跨职能团队全力参与。   | 
| 建模方法   | 实体关系建模为主，有时可以用UML   | 以UML方法为主，画图为辅   | 
| 业务代码分层   | 业务代码理论上应该在业务逻辑层，但有时游离在控制器、业务逻辑层或数据访问层，甚至受依赖的其他业务逻辑中   | 业务代码在领域层，有时在领域对象上，有时在领域服务中。   | 
| 修改代码的难易程度   | 随时随地想改就改   | 需要遵循一定的设计原则或步骤、流程   | 
| 可维护性   | 项目简单时，易于维护；复杂时，难于维护。   | 掌握方法时，维护难度比较平滑。   | 
| 数据持久化   | 在数据访问层中完成，有时可以适当复用；也有开发者将数据访问层提取出仓储的模板方法进行复用。   | 一般在仓储层中实现，且仓储一般是基础设施，意味着除特定场景外，基础设施不会依赖于领域而二外定制行为。   | 
| 多业务逻辑的整合   | 一般在业务逻辑层中实现   | 一般在应用服务层实现。   | 
| 可测试性   | 比较难以加入测试代码   | 易于加入测试代码；也可以根据UML使用TDD来进行开发。   | 

  

## 该如何取舍？

下图这种流传已久，同样来自马丁弗勒老爷子《企业架构应用模式》。

表现了随着软件复杂度的逐渐提升，数据驱动设计和领域驱动设计模式两种不同类型的设计模式的开发效率（时间）对比曲线。

![图片](https://uploader.shimo.im/f/06wlyhAKuoHh8JbJ.png!thumbnail)

* 数据驱动设计建立了一个比较平滑的发展轨迹，但是随着拐点的到来，将变得越来越为难以维护，最终造出一个难以维护的“意大利面”。

![图片](https://uploader.shimo.im/f/7wZlq6CXgdjXacCH.png!thumbnail)

* 领域驱动设计，前期的起点确实比数据驱动设计要高很多，而且甚至在刚刚使用一段时间后，由于业务复杂度的提升，会迎来一个拐点。这个拐点有点像“邓宁·克鲁格效应”中遇到的“绝望之谷”，让开发者和管理层感觉有点力不从心，不少企业最终又拆掉了他们的领域驱动设计搭建的软件；
* 使用领域驱动设计，随着复杂度的逐渐推移，软件开发人员的信心越来越足，代码自然也能够不断演进，平滑发展，。

![图片](https://uploader.shimo.im/f/2sgCQ0cuflkIy50c.png!thumbnail)

* 从长期来看，领域驱动建模将给复杂系统带来更加高效的维护效能。
# 总结

本文介绍了三层架构和领域驱动设计两种不同的设计思想中如何实现业务逻辑代码的过程，并对针对代码的维护性问题进行了分析。由于时间仓促，部分观点、设计图、代码可能还不够成熟，还请大家批评指正。

下一篇将介绍ABP框架开发中的具体实践步骤。

