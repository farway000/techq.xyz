---
title:   常见的C＃异常及其修复方法
date: 2020-7-20 21:05
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# 常见的C＃异常及其修复方法

如果您今天是依靠编写的软件来谋生，那么您可能至少对异常的概念很熟悉。

Jeff Atwood曾经称它们为“现代编程语言的基础”。[异常](https://raygun.com/blog/java-exceptions-terminology/)是现代软件开发中常见且有用的结构，但有时它们也可能造成混乱。

那么什么*是*异常？更具体地说，C＃异常的主要类型是什么，以及如何使用它们？

今天的帖子将回答上述问题以及更多问题。我们将从“异常”的简要定义开始，然后继续解释该结构的组成部分。最后，我们将概述最常见的C＃异常以及如何处理它们。

让我们开始吧。

## 有什么异常？

异常是一种可用于处理错误的机制。就这么简单。在某些情况下，例如，C程序员将返回错误代码，而[Java](https://raygun.com/blog/java-exceptions-terminology/)或C＃程序员都很有可能引发异常。

异常表示执行流程的突然中断。一旦引发异常，执行就会停止。如果未处理异常，则应用程序崩溃。

但是，实际上您是如何做到的呢？您如何引发或捕获异常？所有这些甚至意味着什么？

这就是我们将在下一部分中详细介绍的内容。

## C＃异常剖析

现在，我们将简要介绍C＃异常的情况。您将了解应该使用的主要关键字，这些关键字不仅可以捕获和处理异常，还可以抛出自己的异常。我们列表上的第一个是`try`关键字和块。

### try

异常处理解剖的第一部分是`try`块。您可以使用它来尝试执行一些可能引发异常的代码。考虑以下代码摘录：

```objective-c
string content = string.Empty;
try
{
    content = System.IO.File.ReadAllText(@"C:\file.txt");
}
```

上面的代码声明了一个变量，并为其分配了空字符串。然后，我们有了`try`块。块中的单行代码`try`使用该类中的`ReadAllText`静态方法`System.IO.File`。我们担心该路径表示的文件可能不存在，在这种情况下会引发异常。但是，当然，这还不够。我们的代码不执行任何处理异常的操作。它甚至目前还没有编译！这是来自编译器的消息：

```objective-c
Expected catch or finally
```

该消息是不言自明的。我们需要 `catch`或`finally`代码块，因为尝试处理异常然后忘记执行处理部分没有任何意义。让我们开始吧。

### catch

`catch`代码块使我们能够实际处理异常。我们将使用更多代码扩展前面的示例。

看看这个：

```objective-c
        static void Main(string[] args)
        {
            string content = string.Empty;

            try
            {
                Console.WriteLine("First message inside try block.");
                Console.WriteLine("Second message inside try block.");
                content = System.IO.File.ReadAllText(@"C:\file.txt");
                Console.WriteLine("Third message inside try block. This shouldn't be printed.");
            }
            catch
            {
                Console.WriteLine("First message inside the catch block.");
                Console.WriteLine("Second message inside the catch block.");
            }

            Console.WriteLine("Outside the try-catch.");
            Console.ReadLine();
        }
```

如果运行上面的代码，则应看到以下内容：

```objective-c
First message inside try block.
Second message inside try block.
First message inside the catch block.
Second message inside the catch block.
Outside try-catch.
```

该路径处的文件不存在，因此将引发`System.FileNotFoundException`。发生这种情况时，执行流程将立即中断。因此，将在try块中打印“第三条消息”的行。这不应该被打印。” 永远不会被执行。

执行流程由`catch`块执行。完成后，将控制权交还给main方法，然后打印`Outside try-catch`。并等待用户输入。

### 最后

在了解了`try`和之后`catch`，我们终于（没有双关语）准备好使用这个非常有用但经常被误解的异常处理机制的一部分。那么，什么是`finally`？

该`finally`块是一种确保将执行给定代码段的方式，无论是否引发异常。考虑下面的代码：

```objective-c
try
{
    content = System.IO.File.ReadAllText(path);
    Console.WriteLine("If you're reading this, no exception was thrown.");
}
catch (System.IO.FileNotFoundException e)
{
    Console.WriteLine("If you're reading this, an exception was thrown.");
    Console.WriteLine("Message: " + e.Message);
}
finally
{
    Console.WriteLine("This will be printed, no matter what.");
}
```

那仍然是相同的示例，但是现在要简单得多。注意最后的`finally`块。如果运行此代码，则应该看到以下消息：

```objective-c
If you're reading this, an exception was thrown.
Message: Could not find file 'C:\file.txt'.
This will be printed, no matter what.
Outside the try-catch.
```

现在让我们模拟文件的存在。注释掉试图从文件中读取的行，然后再次运行该应用程序。这是您现在应该看到的：

```objective-c
If you're reading this, no exception was thrown.
This will be printed, no matter what.
Outside the try-catch.
```

如您所见，在最近的场景中，没有引发异常，因此该`catch`块中没有执行任何行。另一方面，`finally`在两种情况下都执行了该块。

### throw

当涉及到异常时，您将不会总是从别人那里处理它们。您也可以自己抛出异常。为此，您将使用`throw`关键字，然后是要引发的异常的类的实例化。以下代码举例说明了这一点：

```objective-c
public ProductService(IProductRepository repository)
{
    if (repository == null)
        throw new ArgumentNullException();

    this.repository = repository;
}
```

## 常见的.NET异常

以下是常见的.NET异常列表：

### System.NullReferenceException

这是最著名的（甚至是臭名昭著的）异常之一。当您尝试调用方法/属性/索引器/等时，抛出此异常。在包含空引用的变量（即，它不指向任何对象）。下面的代码将导致空引用异常：

```objective-c
Person p = people.Where(x => x.SSN == verifySsn).FirstOrDefault();
string name = p.Name;
```

在上面的示例中，我们过滤了将每个项目的SSN属性与`verifySsnvariable`变量进行比较的序列。然后，我们使用该`FirstOrDefault()`方法从序列中仅提取第一项。如果序列不产生任何项目，则它将返回该类型的默认值。由于`Person`是引用类型，因此其返回值为null。

在下一行，我们尝试`Name`在空引用上取消引用属性。*繁荣！*空引用异常。

这是通常不抛出也不捕获的异常。您不要扔它，因为它毫无意义。如果您想与代码的客户交流，给定方法不接受null作为其参数的有效值，则使用的正确异常是`System.ArgumentNullException`。

您如何“修复”此异常？简而言之，您必须对可为空性小心谨慎。如果您正在编写将由第三方使用的任何代码（即使这些第三方是您的同事），则请认真考虑是否接受空引用作为有效值。

无论您做出什么决定，都必须使该决定非常明确并记录在案。您可以通过抛出`System.ArgumentNullException`，例如，并在方法上使用XML文档标题来实现。

### System.IndexOutOfRangeException

在应用程序代码通常不会抛出或捕获该异常的意义上，该异常与上一个异常类似。

那为什么呢？

好吧，当您尝试使用无效的索引值访问数组，列表或任何可索引序列中的元素时，将引发此异常。一个简单的例子：

```objective-c
public static void PrintUrlSufix(string url)
{
    var parts = url.Split('.');
    Console.WriteLine(parts[2]);
}
```

上面的代码显然希望使用**www.acme.com**格式的URL 。但是，如果只是获得**acme.com**怎么办？为此，如果得到**Hakuna Matata**怎么办？是的，没错：`System.IndexOutOfBoundException`是的。

您如何避免遇到此异常？永远不要把事情视为理所当然。永远不要仅仅假设数据将采用正确的格式。做您的尽职调查和检查的东西。

### System.IO.IOException

此C＃异常具有一个不言自明的名称。这正是您的想法：这是IO操作期间发生错误时引发的异常。与前两个异常不同，您可能会发现自己不时捕捉或抛出其中一个。

本`IOException`类实际上是一些更具体的异常，例如：

- DirectoryNotFoundException
- EndOfStreamException
- FileNotFoundException
- FileLoadException
- PathTooLongException

有关的文档，`IOException`建议您尽可能使用更具体的异常，而不是更一般的异常。

### System.Net.WebException

此异常与网络有关。如果使用[可插拔协议](https://docs.microsoft.com/dotnet/framework/network-programming/introducing-pluggable-protocols)访问网络时发生错误，则抛出该错误[。](https://docs.microsoft.com/dotnet/framework/network-programming/introducing-pluggable-protocols)处理此异常时，请记住验证该`Response`属性，该属性将包含远程主机返回的响应。

### System.Data.SqlClient.SqlException

此异常与数据库（特别是SQL Server）有关。SQL Server返回错误或警告时将引发该错误。该类具有一个称为的属性`Errors`，该属性是一个包含`SqlError`该类的一个或多个实例的集合。依次包含有关发生的错误的详细信息。

### System.StackOverflowException

当执行堆栈溢出时，抛出此异常，这通常意味着递归出错。该代码有太多的嵌套方法调用。事情就是这样：这个异常是无法捕获的-至少从.NET 2.0起就没有-这意味着当抛出该异常时，您几乎没有其他选择。默认情况下，您的过程将被终止。

您应该做的而不是捕获此异常的方法是编写代码，以防止它首先发生。

### System.OutOfMemoryException

可以说这是[最令人困惑的C＃异常之一](https://stackoverflow.com/questions/1153702/system-outofmemoryexception-was-thrown-when-there-is-still-plenty-of-memory-fr)。网上有很多资源[可以很好地阐明问题](https://blogs.msdn.microsoft.com/ericlippert/2009/06/08/out-of-memory-does-not-refer-to-physical-memory/)，但是我将在此处提供一个简短的版本。发生的情况是此异常不涉及可用的物理内存。

那么，什么时候抛出此异常？

您知道何时要停车吗，因为其他驾驶员未正确停车而不能停车吗？如果您只需在停放的汽车之间添加所有可用空间，就足以容纳您的车辆。但是目前不可能，因为它不是连续的区域。

当您获得此内存时，这差不多发生了什么。可能有很多可用的总内存，但是没有连续的部分可以满足所需的分配。实际上，例如，如果您尝试将 `StringBuilder`的`MaxCapacity`属性扩展到该属性之外，则会发生此异常。

### System.InvalidCastException

此异常也具有不言自明的名称。当代码由于未定义强制类型转换而无法从一种类型转换为另一种类型时，将引发该错误。以下代码将引发此类型的异常：

```objective-c
object o = "10";
int x = (int)o;
```

这是您通常不会捕获的异常。相反，您将以不会发生的方式编写代码。例如，以下代码在尝试强制类型转换之前进行类型检查：

```objective-c
public override bool Equals (object obj )
{
   if (!obj is Foo)
       return false;

   Foo other = (Foo)obj;
   return this.bar == other.bar;
}
```

上面的代码可以使用简化的[`as`操作符](https://docs.microsoft.com/dotnet/csharp/language-reference/keywords/as)，在这里我没有进一步具体说明，作为一个练习留给读者。无论如何，这就是问题：您实际上应该尝试做的是避免问题，而不是首先进行转换。利用泛型来防止陷入需要强制转换的情况。

### System.InvalidOperationException

像之前的许多其他异常一样，这种异常通常是您不会发现的。相反，您应该做的是编写不会发生的代码。考虑以下示例：

```objective-c
var numbers = new List<int> { 1, 3, 5 };
var firstGreaterThanFive = numbers.Where(x => x > 5).First();
```

当序列不产生任何结果时，将抛出First LINQ扩展方法。如果您知道该序列有时可能不会产生结果-没关系-您应该改用FirstOrDefault。在空序列上调用此方法时，将返回序列类型的默认值，而不是抛出异常。

### System.ObjectDisposedException

我们将在这篇文章中介绍的最后一个C＃异常也属于“不应该处理，请修复代码”类别。换句话说，这是开发人员错误。当您尝试使用已处理的[IDisposable](https://docs.microsoft.com/dotnet/api/system.idisposable?view=netframework-4.7.2)进行操作时，将引发此异常。

此当通常发生在开发者调用`Dispose`，`Close`或其它类似方法和后来试图访问该对象的一个成员时。

## 如何成功的进行异常处理？

错误处理是软件开发教学中经常被忽略的话题，出现异常有时非常不幸。如果没有可靠的[错误处理策略](https://raygun.com/blog/errors-and-exceptions/)，您的应用程序将永远是劣等的。

通过本文，我们希望通过定义异常的概念并对C＃异常的主要类型进行快速概述，以帮助解决该问题。但本文并没有涵盖异常处理的全部。恰好相反，我认为这是一个机会，可以开始引导您对该主题的学习，并且永不停止学习和练习。

祝您能想出一种对您的应用程序有效的策略！