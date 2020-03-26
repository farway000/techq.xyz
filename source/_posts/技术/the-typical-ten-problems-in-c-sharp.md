---
title:  C＃编程中最常见的10个错误
date: 2020-03-26 08:55
tags: 技术
author:  译者：邹溪源
categories:
  - 技术
---
原文来自：[https://www.toptal.com/c-sharp/top-10-mistakes-that-c-sharp-programmers-make](https://www.toptal.com/c-sharp/top-10-mistakes-that-c-sharp-programmers-make)

帕特里克·赖德（PATRICK RYDER）在Microsoft工作期间帮助创建了VB 1.0和更高版本的.NET平台。自2000年以来，他专注于全栈项目。

C＃是针对Microsoft 公共语言运行库（CLR）的几种语言之一。面向CLR的语言受益于多种功能，例如跨语言集成和异常处理，增强的安全性，简化的组件交互模型以及调试和性能分析服务。在当今的CLR语言中，C＃被广泛用于针对Windows台式机，移动或服务器环境的复杂，专业的开发项目中。(译者注，目前已支持各类跨平台的操作系统环境）

C＃是一种面向对象的强类型语言。在编译和运行时，C＃中严格的类型检查会导致尽早报告大多数典型的C＃编程错误，并准确定位其位置。这可以在C Sharp编程中节省大量时间，相比之下，在更自由地执行类型安全的语言中，跟踪令人困惑的错误的原因可能会在违规操作发生很久之后才发生。但是，许多C＃编码人员无意间（或不小心）放弃了这种检测的好处，这导致了本C＃教程中讨论的一些问题。

## 关于本C Sharp编程教程
**本教程描述了C＃程序员犯下的10种最常见的C＃编程错误或应避免的问题，并为他们提供了帮助。**

尽管本文中讨论的大多数错误都是C＃特定的，但有些错误也与其他以CLR为目标或使用框架类库（FCL）的语言有关。

## 常见的C＃编程错误＃1：使用值类型与引用相等，或反过来
C ++和许多其他语言的程序员习惯于控制他们分配给变量的值是简单的值还是对现有对象的引用。但是，在C Sharp编程中，该决定由编写对象的程序员决定，而不是由实例化该对象并将其分配给变量的程序员做出。对于那些试图学习C＃编程的人来说，这是一个常见的“陷阱”。

如果您不知道所使用的对象是值类型还是引用类型，则可能会遇到一些意外。例如：

```
      Point point1 = new Point(20, 30);
      Point point2 = point1;
      point2.X = 50;
      Console.WriteLine(point1.X);       // 20 (does this surprise you?)
      Console.WriteLine(point2.X);       // 50
      
      Pen pen1 = new Pen(Color.Black);
      Pen pen2 = pen1;
      pen2.Color = Color.Blue;
      Console.WriteLine(pen1.Color);     // Blue (or does this surprise you?)
      Console.WriteLine(pen2.Color);     // Blue
```
正如你所看到的，都Point和Pen对象创建方式不尽相同，但值point1保持不变，当一个新的X坐标值被分配到point2，而价值pen1 *是*当一个新的颜色被分配到修改pen2。因此，我们可以*推断出*，point1并且point2每个Point对象都包含自己的对象副本，而pen1和pen2都包含对同一Pen对象的引用。*但是，如果不进行此实验，我们怎么知道呢？*
答案是查看对象类型的定义（您可以在Visual Studio中通过将光标置于对象类型的名称上并按F12轻松地完成此操作）：

```
      public struct Point { ... }     // defines a “value” type
      public class Pen { ... }        // defines a “reference” type
```
如上所示，在C＃编程中，struct关键字用于定义值类型，而class关键字用于定义引用类型。*对于那些具有C ++背景的人，由于C ++和C＃关键字之间的许多相似之处而陷入一种错误的安全感，这种行为可能会让人感到意外，您可能会从C＃教程中寻求帮助。*
如果您要依赖值和引用类型之间不同的某些行为（例如，将对象作为方法参数传递并让该方法更改对象状态的能力），请确保您正在处理正确的对象类型，以避免C＃编程问题。

## 常见的C＃编程错误＃2：误解了未初始化变量的默认值
在C＃中，值类型不能为null。根据定义，值类型具有值，甚至值类型的未初始化变量也必须具有值。这称为该类型的默认值。当检查变量是否未初始化时，这会导致以下结果，通常是意外的结果：

```
      class Program {
          static Point point1;
          static Pen pen1;
          static void Main(string[] args) {
              Console.WriteLine(pen1 == null);      // True
              Console.WriteLine(point1 == null);    // False (huh?)
          }
      }
```
为什么point1不为空？答案是Point是值类型，它的默认值为Point（0,0），而不是null。未能意识到这一点是在C＃中非常容易（也是常见）的错误。
许多（但不是全部）值类型都有一个IsEmpty属性，您可以检查该属性是否等于其默认值：

```
Console.WriteLine(point1.IsEmpty);        // True
```
当您检查变量是否已初始化时，请确保您知道该类型的未初始化变量在默认情况下将具有什么值，并且不要依赖于它为null。
## 常见的C＃编程错误＃3：使用不正确或未指定的字符串比较方法
比较C＃中的字符串有很多不同的方法。

尽管许多程序员使用==运算符进行字符串比较，但这实际上是最不希望采用的方法之一，主要是因为它没有在代码中明确指定需要哪种类型的比较。

相反，在C＃编程中测试字符串相等性的首选方法是使用以下Equals方法：

      public bool Equals(string value);

      public bool Equals(string value, StringComparison comparisonType);

第一个方法签名（即不带comparisonType参数）实际上与使用==运算符相同，但是具有显式应用于字符串的好处。它执行字符串的序数比较，基本上是逐字节比较。在很多情况下，这正是您想要的比较类型，尤其是在比较以编程方式设置值的字符串（例如文件名，环境变量，属性等）时。在这些情况下，只要序数比较确实是正确的类型这种情况下的比较，使用Equals方法不带 comparisonType参数的唯一缺点是，阅读代码的人可能不知道您要进行哪种类型的比较。

但是，使用Equals每次比较字符串包含comparisonType的方法签名，不仅可以使代码更清晰，还可以使您明确考虑需要进行哪种类型的比较。这是一件值得做的事情，因为即使英语在序数比较和对文化敏感的比较之间不能提供很多差异，其他语言也可以提供很多好处，而忽略其他语言的可能性正在为您提供巨大的潜力错误的道路。例如：

```
      string s = "strasse";
      
      // outputs False:
      Console.WriteLine(s == "straße");
      Console.WriteLine(s.Equals("straße"));
      Console.WriteLine(s.Equals("straße", StringComparison.Ordinal));
      Console.WriteLine(s.Equals("Straße", StringComparison.CurrentCulture));        
      Console.WriteLine(s.Equals("straße", StringComparison.OrdinalIgnoreCase));
      
      // outputs True:
      Console.WriteLine(s.Equals("straße", StringComparison.CurrentCulture));
      Console.WriteLine(s.Equals("Straße", StringComparison.CurrentCultureIgnoreCase));
```
最安全的做法是始终为该Equals方法提供comparisonType参数。以下是一些基本准则：
* 在比较用户输入的字符串或要显示给用户的字符串时，请使用区分区域性的比较（CurrentCulture或CurrentCultureIgnoreCase）。
* 比较程序字符串时，请使用序数比较（Ordinal或OrdinalIgnoreCase）。
* InvariantCulture和InvariantCultureIgnoreCase一般不被除了在非常有限的情况下使用，因为顺序比较是更有效的。如果需要进行文化意识比较，则通常应针对当前文化或其他特定文化进行比较。

除了Equals方法之外，字符串还提供了 Compare方法，该方法为您提供有关字符串相对顺序的信息，而不仅仅是进行相等性测试。此方法是优选的<，<=，>和>=运算符，对于上述的为讨论避免C＃的问题同样的原因。

## 常见的C＃编程错误＃4：使用迭代（而不是声明性）语句来操作集合
在C＃3.0中，向[语言](http://msdn.microsoft.com/en-us/library/bb308959.aspx)添加[语言集成查询](http://msdn.microsoft.com/en-us/library/bb308959.aspx)（LINQ）永远改变了查询和操作集合的方式。从那时起，如果您使用迭代语句来操作集合，那么您本来应该使用LINQ。

一些C＃程序员甚至不知道LINQ的存在，但是幸运的是，这个数目正在变得越来越小。但是，许多人仍然认为，由于LINQ关键字和SQL语句之间的相似性，它的唯一用途是在查询数据库的代码中。

尽管数据库查询是LINQ语句的一种非常普遍的用法，但它们实际上是在任何可枚举的集合（即，实现IEnumerable接口的任何对象）上工作的。因此，例如，如果您有一个Accounts数组，而不是为每个each编写一个C＃List：

```
      decimal total = 0;
      foreach (Account account in myAccounts) {
        if (account.Status == "active") {
          total += account.Balance;
        }
      }
你可以这样写：
      decimal total = (from account in myAccounts
                       where account.Status == "active"
                       select account.Balance).Sum();
```
尽管这是一个非常简单的示例，说明如何避免这种常见的C＃编程问题，但在某些情况下，单个LINQ语句可以轻松替换代码中的迭代循环（或嵌套循环）中的数十个语句。更少的通用代码意味着更少的引入错误的机会。但是请记住，在性能方面可能会有所取舍。在对性能有严格要求的情况下，尤其是在迭代代码能够对LINQ无法进行的集合进行假设的情况下，请确保在这两种方法之间进行性能比较。
## 常见的C＃编程错误＃5：无法考虑LINQ语句中的基础对象
LINQ非常适合抽象处理集合的任务，无论它们是内存中对象，数据库表还是XML文档。在理想环境中，您不需要知道底层对象是什么。但是这里的错误是假设我们生活在一个完美的世界中。实际上，如果相同的LINQ语句恰好采用不同的格式，则当它们对完全相同的数据执行时，它们可以返回不同的结果。

例如，考虑以下语句：

```
      decimal total = (from account in myAccounts
                       where account.Status == "active"
                       select account.Balance).Sum();
```
如果对象的其中一个account.Status等于“活动”（请注意大写字母A）会怎样？好吧，如果myAccounts是一个DbSet对象（使用默认的不区分大小写的默认配置设置），则where表达式仍会匹配该元素。但是，如果myAccounts位于内存阵列中，则它将不匹配，因此将产生总计不同的结果。
等一下 在前面讨论字符串比较时，我们看到==运算符对字符串进行了序数比较。那么，为什么在这种情况下==操作员执行不区分大小写的比较？

*答案是，当LINQ语句中的基础对象是对SQL表数据的引用时（如本示例中的Entity Framework DbSet对象一样），该语句将转换为T-SQL语句。然后，操作员将遵循T-SQL编程规则，而不是C＃编程规则，因此，上述情况下的比较最终不区分大小写。*

通常，即使LINQ是查询对象集合的有用且一致的方式，实际上，您仍然需要知道您的语句是否将转换为C＃以外的其他内容，以确保代码的行为能够在运行时达到预期。

## 常见的C＃编程错误＃6：扩展方法使您感到困惑或冒充
如前所述，LINQ语句可在实现IEnumerable的任何对象上工作。例如，以下简单功能将在任何帐户集合上累加余额：

```
      public decimal SumAccounts(IEnumerable<Account> myAccounts) {
          return myAccounts.Sum(a => a.Balance);
      }
```
在上面的代码中，myAccounts参数的类型声明为 IEnumerable<Account>。由于myAccounts引用Sum方法（C＃使用熟悉的“点符号”来引用类或接口上的方法），因此我们希望看到Sum()在IEnumerable<T>接口定义上调用的方法。但是，定义 IEnumerable<T>未引用任何Sum方法，而只是这样：
```
      public interface IEnumerable<out T> : IEnumerable {
          IEnumerator<T> GetEnumerator();
      }
```
那么该Sum()方法在哪里定义？C＃是强类型的，因此，如果对该Sum方法的引用无效，则C＃编译器肯定会将其标记为错误。因此，我们知道它必须存在，但是在哪里？此外，LINQ为查询或汇总这些集合提供的所有其他方法的定义在哪里？
答案是这Sum()不是IEnumerable接口上定义的方法 。相反，它是在System.Linq.Enumerable类上定义的静态方法（称为“扩展方法”）：

```
      namespace System.Linq {
        public static class Enumerable {
          ...
          // the reference here to “this IEnumerable<TSource> source” is
          // the magic sauce that provides access to the extension method Sum
          public static decimal Sum<TSource>(this IEnumerable<TSource> source,
                                             Func<TSource, decimal> selector);
          ...
        }
      }
```
那么，什么使扩展方法与任何其他静态方法不同，又使我们能够在其他类中访问它呢？
扩展方法的显着特征是this其第一个参数上的 修饰符。这是“魔术”，可以将其标识为编译器的扩展方法。它修改的参数的类型（在本例中为IEnumerable<TSource>）表示将要实现此方法的类或接口。

（另一方面，IEnumerable接口名称和Enumerable定义扩展方法的类的名称 之间的相似性并没有什么神奇的。这种相似性只是一个任意的样式选择。）

有了这种理解，我们还可以看到sumAccounts上面介绍的功能可以改为如下实现：

      public decimal SumAccounts(IEnumerable<Account> myAccounts) {

          return Enumerable.Sum(myAccounts, a => a.Balance);

      }

我们本可以以这种方式实现它的事实反而引起了一个问题，为什么根本没有扩展方法？ [扩展方法](http://msdn.microsoft.com/en-us/library/bb383977.aspx)本质上是C＃编程语言的一种便利，它使您可以将方法“添加”到现有类型，而无需创建新的派生类型，重新编译或修改原始类型。

通过using [namespace];在文件顶部包含一条语句，可将扩展方法纳入范围。您需要知道哪个C＃名称空间包含要查找的扩展方法，但是一旦知道要查找的内容，就很容易确定。

当C＃编译器在对象的实例上遇到方法调用，但未找到在引用的对象类上定义的方法时，它将查看范围内的所有扩展方法，以尝试查找与所需方法匹配的扩展方法。签名和类。如果找到一个，它将实例引用作为该扩展方法的第一个参数传递，然后其余参数（如果有）将作为后续参数传递给扩展方法。（如果C＃编译器在范围内找不到任何相应的扩展方法，它将抛出错误。）

扩展方法是C＃编译器中“语法糖”的一个示例，它使我们能够编写（通常）更清晰，更可维护的代码。更清楚的是，如果您知道它们的用法。否则，可能会有些混乱，尤其是在开始时。

尽管使用扩展方法当然具有优势，但它们可能会引起问题，并且对于那些不了解它们或不正确理解它们的开发人员，C＃编程帮助会大声疾呼。当在线查看代码示例或任何其他预先编写的代码时，尤其如此。当此类代码产生编译器错误时（因为它调用的类显然没有定义方法），人们倾向于认为该代码适用于该库的不同版本，或完全适用于不同的库。可能会花费大量时间搜索不存在的新版本或幻影“缺少库”。

当对象上存在具有相同名称的方法时，即使熟悉扩展方法的开发人员仍然偶尔会被捕获，但是其方法签名与扩展方法的方法签名之间存在细微的差异。寻找错别字或错误可能会浪费很多时间。

在C＃库中使用扩展方法变得越来越普遍。除LINQ之外，[Unity Application Block](http://msdn.microsoft.com/en-us/library/ff648512.aspx)和[Web API框架](http://msdn.microsoft.com/en-us/library/hh833994%28v=vs.108%29.aspx)是Microsoft经常使用的两个现代库的示例，它们也使用扩展方法，并且还有许多其他方法。框架越现代，就越有可能包含扩展方法。

当然，您也可以编写自己的扩展方法。请意识到，尽管扩展方法看起来像常规实例方法一样被调用，但这实际上只是一种幻想。特别是，您的扩展方法不能引用它们正在扩展的类的私有成员或受保护成员，因此不能完全替代更传统的类继承。

## 常见的C＃编程错误＃7：为当前任务使用错误的集合类型
C＃提供了大量的各种对象集合，具有以下仅为部分清单：

Array，ArrayList，BitArray，BitVector32，Dictionary<K,V>，HashTable，HybridDictionary，List<T>，NameValueCollection，OrderedDictionary，Queue, Queue<T>，SortedList，Stack, Stack<T>，StringCollection，StringDictionary。

尽管在某些情况下，太多的选择和不足的选择一样糟糕，但对于集合对象却并非如此。可用的选项数量肯定可以使您受益。预先花一些时间进行研究，然后为您的目的选择最佳的收集类型。这可能会导致更好的性能和更少的错误空间。

如果有一种收集类型专门针对您拥有的元素类型（例如字符串或位），则倾向于首先使用该元素。当针对特定类型的元素时，实现通常会更高效。

为了利用C＃的类型安全性，通常应首选使用通用接口而不是非通用接口。泛型接口的元素是您在声明对象时指定的类型，而非泛型接口的元素则是object类型。使用非泛型接口时，C＃编译器无法对您的代码进行类型检查。同样，在处理原始值类型的集合时，使用非泛型集合将导致这些类型的重复 [装箱/拆箱](http://msdn.microsoft.com/en-us/library/yz2be5wk.aspx)，与适当类型的泛型集合相比，可能会对性能产生重大的负面影响。

另一个常见的C＃问题是编写您自己的集合对象。但这并不是说它永远不合适，但是通过提供.NET提供的广泛选择，您可以通过使用或扩展已经存在的扩展而不是重新发明轮子来节省大量时间。特别是，用于C＃和CLI的C5通用集合库“开箱即用”提供了各种各样的附加集合，例如持久树数据结构，基于堆的优先级队列，哈希索引数组列表，链接列表等等。

## 常见的C＃编程错误＃8：忽略释放资源
CLR环境使用垃圾回收器，因此您无需显式释放为任何对象创建的内存。实际上，您不能。没有C ++ delete运算符或free()这样的函数。但这并不意味着您在使用完所有对象后就可以忘记所有对象。许多类型的对象封装了其他类型的系统资源（例如，磁盘文件，数据库连接，网络套接字等）。保持这些资源开放状态会迅速耗尽系统资源的总数，从而降低性能并最终导致程序错误。

尽管可以在任何C＃类上定义析构函数方法，但析构函数（在C＃中也称为终结器）存在的问题是，您不确定是否会调用它们。它们在将来的不确定时间内被垃圾收集器调用（在单独的线程上，这可能会导致其他问题）。尝试通过强制使用垃圾回收来克服这些限制 GC.Collect()不是[C＃最佳实践](https://orcharddojo.net/orchard-resources/Library/DevelopmentGuidelines/BestPractices/CSharp)，因为这将在线程收集所有符合收集条件的对象时在未知时间内阻塞线程。

这并不是说终结器没有很好的用途，但是以确定性方式释放资源并不是其中之一。相反，当您在文件，网络或数据库连接上进行操作时，您希望在完成使用后立即显式释放基础资源。

在几乎[所有环境中，](https://www.toptal.com/c-sharp/how-to-make-an-android-and-ios-app-in-c-on-a-mac)资源泄漏都是一个问题。但是，C＃提供了一种健壮且易于使用的机制，如果使用该机制，则使泄漏的情况更加罕见。.NET框架定义了IDisposable仅由Dispose()方法组成的接口 。任何实现的对象都IDisposable希望在对象的使用者完成对它的操作后才调用该方法。这导致显式，确定性的资源释放。

如果要在单个代码块的上下文中创建和处理对象，则忘记调用基本上是不可原谅的 Dispose()，因为C＃提供了一条using语句， Dispose()无论代码块如何退出（无论它是例外，return陈述式，或是干脆关闭区块）。是的，这与using前面提到的语句相同，该语句用于在文件顶部包含C＃名称空间。它有第二个完全不相关的目的，许多C＃开发人员都不知道。即，确保Dispose()在退出代码块时对对象进行调用：

```
      using (FileStream myFile = File.OpenRead("foo.txt")) {
        myFile.Read(buffer, 0, 100);
      }
```
通过using 在上面的示例中创建一个块，您可以确定 myFile.Dispose()在处理完文件后立即调用该块，无论是否Read()引发异常。
## 常见的C＃编程错误＃9：回避异常
C＃将其类型安全性强制实施到运行时。这使您能够比在C ++等语言中更快地查明C＃中的许多类型的错误，在C＃中错误的类型转换可能导致将任意值分配给对象的字段。但是，程序员再次可以浪费这一强大功能，从而导致C＃问题。之所以陷入这种陷阱，是因为C＃提供了两种不同的处理方式，一种可以引发异常，而另一种则不能。有些人会回避异常路由，认为不必编写try / catch块可以节省一些代码。

例如，以下两种方法可以在C＃中执行显式类型转换：

```
      // METHOD 1:
      // Throws an exception if account can't be cast to SavingsAccount
      SavingsAccount savingsAccount = (SavingsAccount)account;
      
      // METHOD 2:
      // Does NOT throw an exception if account can't be cast to
      // SavingsAccount; will just set savingsAccount to null instead
      SavingsAccount savingsAccount = account as SavingsAccount;
```
使用方法2可能发生的最明显的错误是无法检查返回值。这可能会导致最终的NullReferenceException，该异常可能会在更晚的时间浮出水面，从而更加难以找到问题的根源。相反，方法1会立即抛出一个 InvalidCastException问题，使问题的根源更加明显。
而且，即使您记得在方法2中检查过返回值，如果发现它为空，您将怎么办？您编写的方法是否适合报告错误？如果强制转换失败，您还可以尝试其他方法吗？如果不是，那么抛出异常是正确的事，因此您最好让它尽可能地靠近问题的根源。

这是其他两个常见方法对的两个示例，其中一个抛出异常而另一个不抛出异常：

```
      int.Parse();     // throws exception if argument can’t be parsed
      int.TryParse();  // returns a bool to denote whether parse succeeded
      
      IEnumerable.First();           // throws exception if sequence is empty
      IEnumerable.FirstOrDefault();  // returns null/default value if sequence is empty
```
一些C＃开发人员是如此“异常不利”，以至于他们自动认为不抛出异常的方法是更好的。尽管在某些特定情况下这可能是正确的，但作为概括，它根本不正确。
作为一个特定的示例，如果您有替代的合法（例如，默认）操作要发生，那么将产生异常，那么非异常方法可能是一个合法的选择。在这种情况下，写这样的东西确实更好：

```
      if (int.TryParse(myString, out myInt)) {
        // use myInt
      } else {
        // use default value
      }
```
代替：
```
      try {
        myInt = int.Parse(myString);
        // use myInt
      } catch (FormatException) {
        // use default value
      }
```
但是，认为TryParse必然是“更好”的方法是不正确的。有时候是这种情况，有时候不是。这就是为什么有两种方法可以做到这一点。在您所处的环境中使用正确的方法，请记住，作为开发人员，异常肯定可以成为您的朋友。
## 常见的C＃编程错误＃10：允许编译器警告累积
尽管此问题绝对不是C＃特有的，但由于放弃了C＃编译器提供的严格类型检查的优点，因此在C＃编程中尤为突出。

产生警告是有原因的。尽管所有C＃编译器错误都表明您的代码有缺陷，但许多警告也是如此。两者的区别在于，在出现警告的情况下，编译器在发出代码所表示的指令时没有问题。即使这样，它也会发现您的代码有些混乱，并且您的代码有可能无法准确反映您的意图。

就本C＃编程教程而言，一个常见的简单示例是，当您修改算法以消除对正在使用的变量的使用时，却忘记了删除变量声明。该程序将完美运行，但编译器将标记无用的变量声明。程序运行完美的事实导致程序员忽略了修复警告原因的方法。此外，编码人员还利用了Visual Studio功能，该功能使他们可以轻松地将警告隐藏在“错误列表”窗口中，从而使他们只能专注于错误。很快就出现了数十种警告，所有这些警告都被幸福地忽略了（或更糟的是隐藏了）。

但是，如果您迟早忽略这种类型的警告，则类似这样的内容很可能会在您的代码中找到：

```
      class Account {
      
          int myId;
          int Id;   // compiler warned you about this, but you didn’t listen!
  
          // Constructor
          Account(int id) {
              this.myId = Id;     // OOPS!
          }
  
      }
```
而且，以Intellisense允许我们编写代码的速度，此错误并不像看起来那样不可能。
现在，您的程序中出现了严重错误（尽管出于已经说明的原因，编译器仅将其标记为警告），并且根据程序的复杂程度，您可能会浪费大量时间来跟踪该程序。如果您首先注意了此警告，则只需五秒钟即可解决此问题。

**记住，如果您正在侦听，C Sharp编译器会为您提供有关代码健壮性的许多有用信息。不要忽略警告。**通常，它们只需要花费几秒钟的时间进行修复，而在发生新问题时修复它们可以节省您的时间。训练自己，使Visual Studio“错误列表”窗口显示“ 0错误，0警告”，以便所有警告使您感到不舒服，无法立即解决它们。

当然，每个规则都有例外。因此，有时您的代码对编译器来说似乎有些混乱，即使这正是您的预期。在极少数情况下，请#pragma warning disable [warning id]仅在周围使用触发警告的代码，并仅使用其触发的警告ID。这将取消该警告，并且仅禁止该警告，因此您仍然可以保持警惕以防出现新的警告。

## 结论
C＃是一种功能强大且灵活的语言，具有许多可以极大地提高生产率的机制和范例。但是，就像使用任何软件工具或语言一样，对其功能的有限了解或欣赏有时可能更多的是障碍而不是收益，可能会导致生产环境代码的问题频发。为此，我们需要更多的了解C#语言中那些常见的错误，并不断的持续优化，确保每一行代码都处于可控的状态。

在你的日常开发过程中，你是否也曾经遇到过这些常见错误？赶紧跟你身边的伙伴一起分享吧~



