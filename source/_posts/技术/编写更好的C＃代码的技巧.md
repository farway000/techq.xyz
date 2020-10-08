---
title:  编写更好的C＃代码的技巧
date: 2020-10-8 18:07
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# 编写更好的C＃代码的技巧

##### 编者导语

*本文来自https://www.pluralsight.com，原文包含以下三篇文章：*

*《编写更好的C＃代码简介》https://www.pluralsight.com/guides/introduction-to-writing-better-csharp-code*

*《编写更好的C＃代码的技巧》https://www.pluralsight.com/guides/tips-for-writing-better-c-code*

*《有关编写更好的C＃代码的更多技巧》https://www.pluralsight.com/guides/more-tips-for-writing-better-csharp-code*

*虽然本文仅介绍了C#6.0语言特性，而现在最新的C#已经到了9.0，但这些内容已经仍然常读常新。*

# 一、简介

C＃已从C＃5更改为C＃6，为使项目更具可读性，基于最佳标准的实践也得到了发展，。

本指南系列的目的是帮助您为在团队环境中运行的C＃项目和.NET Framework应用程序编写更简洁的代码。在团队环境下，编写好的代码对开发人员可能更容易，因为编写的代码将由团队中其他开发人员使用，管理和更新，而代码质量往往取决于您个人团队的“哲学”和开发人员的编码实践。在这种情况下，最好的方法是遵循编码团队的准则，并为应用程序项目中的C＃程序添加设计和风格，以使它们对读者更好。请注意，C＃编译器并不关心您放入代码中的风格。但我们以一种使C＃应用程序对读者来说看起来更简单，更清洁，将更容易的方式更深入地进行编程，同时保持代码开发的性能和效率。

在阅读本指南之前，您应该了解以下几点：

1. 第6版对C＃的改进
2. .NET框架中的LINQ
3. `Task`C＃中的异步编程和对象
4. 使用C＃进行的不安全编程，使您无法正常的使用内存管理

##### 不专注于性能

应该注意的是，我不会谈论改变程序性能，提高效率或减少程序运行所花费的时间。通过编写简洁的C＃代码，您可以在几秒钟内提高程序性能，但是以下技巧并不能保证您的代码性能更好。

## 为什么要编写整洁的代码？

您编写代码，编译器编译时没有警告也没有错误，代码很好。但是，如果其他人想读出该代码怎么办？如果有人后来需要为您或您所在的公司升级代码，该怎么办？看下面的代码：

```C#
public static void Main(string[] args) {
  int x = 0;
  x = Console.Read();

  Console.WriteLine(x * 1.5);
}
```

该程序运行良好，系统中没有错误，应用程序也可以正常工作。但是您能告诉我该程序在现实生活中做什么吗？以下是可以做出的一些假设：

1. 它只是乘以价值
2. 就像奖金一样，它正在增加价值
3. 是个人银行存款总额的利率
4. 等等。

哪一个是真实的？没有人会知道。在这种情况下，最好编写出良好的代码，并记住遵循编程的基础。看下面的代码：

```c#
public static void Main(string[] args) {
  int salary = 0;
  salary = Console.Read();

  Console.WriteLine(salary * 1.5);
}
```

这比以前的代码有意义吗？我们可以很容易地说这个代码将增加薪水的价值。请注意，仅通过改进代码，我们就能确保其他人可以比以前更快地理解它。

在本指南中，我不会向您展示如何遵循最佳原则。相反，我将以您已有的知识为基础，并教您如何充分利用C＃程序。我将重点介绍如何在应用程序中编写良好的C＃逻辑，因此您将看到通过以这种方式和结构编写程序，可以从应用程序中获得很多好处。

因此，让我们开始吧。

## 对象初始化

C＃是一种面向对象的编程语言。如果对象本身没有分块，那么写一组提示有什么好处？本节将重点介绍在前进并`new Object()`在应用程序中编写代码之前应考虑的事项。您必须了解如何创建C＃类以及事物如何协作以在系统中启动一个小程序。

例如，看下面的代码：

```C#
class Person {
  public int ID { get; set; }
  public string Name { get; set; }
  public DateTime DateOfBirth { get; set; }
  public bool Gender { get; set; }
}
```

您可能想要创建默认情况下设置值的程序，或者让它们来自模型或诸如此源代码的任何其他面向数据库的数据源，这些程序简化了在对象时输入默认值的方式正在创建。

```c#
var person = new Person { ID = 1, Name = "Afzaal Ahmad Zeeshan", DateOfBirth = new DateTime(1995, 08, 29), Gender = true };
```

相反，请尝试通过以下方式编写相同的代码：

```c#
var person = new Person();
person.ID = 1;
person.Name = "Afzaal Ahmad Zeeshan";
// So on.
```

这里的代码没有明显的性能改进，但是可以真正提高代码的可读性。如果您喜欢*缩进*，请在这里查看：

```c#
var person = new Person
{
  ID = 1,
  Name = "Afzaal Ahmad Zeeshan",
  DateOfBirth = new DateTime(1995, 08, 29),
  Gender = true
};
```

这也有缩进，但是它为您的C＃代码的可读性添加了更多的说明。尽管前面的代码可以实现相同的功能，但是建议的代码可以使代码更易读和简洁。

# 二、技巧

## 空检查

`NullReferenceException`当缺少初始化的对象再次抛出异常时，您是否曾经对感到恼火？在程序中进行空检查有很多好处，不仅可以提高可读性，而且可以确保程序不会由于内存问题而终止（例如，内存中不存在变量时）。这些可能与程序的安全性以及团队具有的良好UI和UX准则相抵触。大多数情况下，由于以下原因会引发空异常：

```c#
string name = null;

Console.WriteLine(name);
```

在大多数情况下，除非您解决此问题，否则编译器本身不会继续运行，但是如果您设法以某种方式诱使编译器认为变量具有值，但在运行时没有变量，则会出现空引用异常。为了克服这个问题，您可以执行以下操作：

```c#

string name = null;

// Try to enter the value, from somewhere
if(name != null) {
   Console.WriteLine(name);
}
```

此安全检查将确保在调用此变量时该值可用。否则，它将影响您代码的路径。但是，在C＃6中，还有另一种方法可以克服此错误。考虑以下情形：建立数据库，建立数据表，找到您的人员但找不到他们的就业详细信息。你能找到他们工作的公司吗？

```C#
var company = DbHelper.PeopleTable.Find(x => x.id == id).FirstOrDefault().EmploymentHistory.CompanyName; // Error
```

如果您这样做，将会出现错误，因为我们只能在这些值的列表中进行简单几步的对象筛选。然后我们将碰到一个空值，一切都丢失了。C＃6提出了一种克服这些情况的新方法，方法是在值和字段可以为null的后面使用安全的导航运算符。`?.`。像这样：

```c#
var company = DbHelper?.PeopleTable?.Find(x => x.id == id)?.FirstOrDefault()?.EmploymentHistory?.CompanyName; // Works
```

如果前一个不为null，则此代码仅检查下一个值。如果先前的值为null，它将返回null并将null保存为的值`company`，而不是引发错误。将检查留给框架本身可以很方便，但是，尽管如此，您仍然必须在最后检查其余值是否为null。

```c#
var company = DbHelper.PeopleTable?.Find(x => x.id == id)?.FirstOrDefault()?.EmploymentHistory?.CompanyName;

if(company != null) {
   // Final process
}
```

但是您明白了这一点，而不是编写代码并检查所有内容是否为空，而是可以执行简单的检查并执行程序中想要的操作和逻辑。否则，将需要`try...catch`包装器或多个`if...else`块来控制程序在系统中的导航方式。

## 异步编程模式

如果您正在使用C＃5进行编程，那么您已经在使用async / await关键字为您的应用程序带来改进。如果不是这种情况，那么我建议您在应用程序的源代码中使用异步编程模式。这不仅可以提高对程序的响应速度，还可以提高应用程序的可读性。在源代码中具有异步模式的一些好处是：

1. 代码路径开始变得更加有意义。如果有一个进程在后台开始运行，那么程序员可以了解程序应该在哪里。
2. 应用程序挂起问题将消失。大多数与应用程序阻塞相关的问题直接来自代码。当UI线程无法更新UI时，用户会认为该应用程序正在挂起并且没有响应，而事实并非如此。异步方法确实可以帮上大忙。
3. 基于Windows运行时的应用程序完全基于此方法。您将（**并且必须是！**）在您的Windows Runtime应用程序中使用这种方法来解决诸如挂起应用程序或不良的编程习惯之类的问题。

自从线程化以来，代码执行的并行化就已经存在。异步已经成为程序和应用程序的重要组成部分，因此您更应该考虑使用它。

## C＃字符串构建

字符串是当今应用程序的重要组成部分，构建字符串可能会花费很多时间，并且还会导致应用程序性能下降。您可以通过多种方式在C＃程序中构建字符串。以下是其中几种方式：

```c#
string str = ""; // Setting it to null would cause additional problems.

// Way 1
str = "Name: " + name + ", Age: " + age;

// Way 2
str = string.Format("Name: {0}, Age: {1}", name, age);

// Way 3
var builder = new StringBuilder();
builder.Append("Name: ");
builder.Append(name);
builder.Append(", Age: ");
builder.Append(age);
str = builder.ToString();
```

请注意，C＃中的字符串是不可变的。这意味着，如果您尝试更新它们的值，则会重新创建它们，并从内存中删除以前的句柄。这就是为什么方式1看起来是最好的方式，但经过进一步思考，事实并非如此。最好的方法是方法3，它使您可以构建字符串而不必在内存中重新创建对象。同时，C＃6引入了一种全新的方式在C＃中构建字符串，该方式比您以前想象的要好得多。新的*字符串插值*运算符`$`为您提供了以最佳方式执行字符串构建的功能。字符串插值如下所示：

```C#
static void Main(string[] args)
{
   // Just arbitrary variables
   string name = "";
   int age = 0;

   // Our interest
   string str = $"Name: {name}, Age: {age}";
}
```

只需一行代码，编译器就会自动将其转换为`string.Format()`版本。为了证明这一点，将详细说明此C＃程序已生成的字节码，并向您展示如何自动更改语法以读取字符串格式。

```C#
IL_0000:  nop
IL_0001:  ldstr       ""
IL_0006:  stloc.0     // name
IL_0007:  ldc.i4.0
IL_0008:  stloc.1     // age
IL_0009:  ldstr       "Name: {0}, Age: {1}"
IL_000E:  ldloc.0     // name
IL_000F:  ldloc.1     // age
IL_0010:  box         System.Int32
IL_0015:  call        System.String.Format
IL_001A:  stloc.2     // str
IL_001B:  ret
```

可以看出，这显示了如何将语法更改回我们已经看到的语法。有关**`IL_0009`**更多信息，请参见。当其他人正在读取程序时，这可以使您的程序外观更简洁，并且如果要构建的字符串较小，则可以提高性能。如果字符串较大，请使用`StringBuilder`。

# 三、更多技巧

## 遍历数据

如果不对一组数据进行循环和迭代，那么应用程序有什么用？在这种情况下，有时您将不得不查找值，查找节点，查找记录或对集合进行任何其他遍历。在这种情况下，您确实需要确保编写干净的代码，因为这是性能和可读性都非常重要且相互关联的领域。有了一些经验，我就克服了编写用于读取和遍历数据的错误代码的方式。这正是LINQ应该加入的地方，LINQ允许您编写使用最佳.NET框架为用户和客户提供最佳编码体验和最佳体验的程序。

以前，您可能已经做过以下一些事情：

```c#
6
// A function to search for people
Person FindPerson(int id) {
   var people = DbContext.GetPeople(); // Returns List<Person>

   foreach (var person in people) {
      if(person.ID == id) {
         return person;
      }
   }

   // No person found.
   return null;
}

// Then do this
var person = FindPerson(123);
```

对于任何想接手您代码的人来说，这都是一段易读的代码。但是，使用C＃中的LINQ查询可以使代码更加简单和整洁。您可以通过两种方式执行此操作。一个有点像SQL，另一个是通过`Where`在集合上使用该函数并传递我们的要求。

```c#
// A function to search for people
Person FindPerson(int id) {
   var people = DbContext.GetPeople(); // Returns List<Person>

   return (from person in people
          where person.ID == id
          select person).ToList().FirstOrDefault();
}

// Then do this
var person = FindPerson(123);
```

该代码看起来有点像SQL，可以增强代码的可读性和性能。该函数相似，但是，该`Where`函数的读取效果更好，并使所有迭代都针对.NET框架本身，而.NET框架将为应用程序提供最佳性能。

现在，让我们看看用相同的C＃代码编写此查询的另一种方式：

```c#
// A function to search for people
Person FindPerson(int id) {
   var people = DbContext.GetPeople(); // Returns List<Person>

   return people.FirstOrDefault(x => x.ID == id);
}

// Then do this
var person = FindPerson(123);
```

请注意，`null`如果没有找到匹配项，则返回第一个代码。这段代码也做同样的事情。唯一的第一个代码更糟糕的是它必须对集合本身执行迭代。

该本地变量`return person;`将允许程序返回控件，但是如果数据位于最后一个位置会发生什么呢？此数据搜索算法的复杂度仍为O（n）。

## 避免unsafe上下文

在您必须亲自处理内存时，C＃还支持手动内存管理。C＃中的不安全上下文允许您操作内存，执行指针算术，在可能无法访问的内存位置读取和写入数据，等等。但是，.NET框架可以做很多事情来克服内存问题，延迟和磁盘上其他问题。这也使.NET框架完全无需实际执行任何内存管理，.NET框架将为您做到这一点。

使用不安全的上下文有很多好处，例如，当您要围绕本机C ++库编写包装器时。Emgu CV就是这样一个示例，您将在其中编写一些代码来处理如何管理本机代码，并以更简单的方式来处理内存中的错误。在这种情况下，您可以：

1. 使用指针管理和指针算术。您不能在此上下文之外的任何地址上执行任何操作，这是.NET规则所处的位置。
2. 使用内存管理来操作内存中的对象。
3. 使用C ++风格的编程，这正是C＃设计的目的。

这几乎没有好处，如果您应该在应用程序中考虑这一点，请明智地考虑。

#### 关于`Unsafe`纯属个人观点

我还想指出，关于“不安全”的利弊，我所说的一切都是我个人的看法。我不经常在程序中使用`unsafe`上下文，因为没有理由不考虑在应用程序中使用上下文。但是，如果您的应用程序需要本机内存管理，则可以使用此上下文。

## 尽可能使用Lambda表达式

Lambda来自函数式编程领域，在C＃中已广泛使用，从内联函数一直到C＃6中的getter only属性。我将展示C＃中的两种用法，它们构成的程序，不仅看起来更清爽，而且性能指标也更高。

为此，我将向您显示该C＃代码的IL。我个人喜欢在许多领域使用lambda，尤其是当我不得不用C＃编写内联函数时。自从可以使用此概念编写仅用于getter的属性以来，我一直在使用它们，并且我个人认为它比以前做同一件事的方法更好。

### 1.将Lambda用于内联函数

您应该知道一些C＃编程的示例，使用这种写法的代码很多。

例如在应用程序中进行事件处理的情况下，对于事件处理，您可以像下面这样编写当前函数：

```c#

// Without lamdbas
myBtn.Click += Btn_Click;

public void Btn_Click (object sender, EventArgs e) {
   // Code to handle the event
}

// With the help of lambdas
myBtn.Click += (sender, e) =>
               {
                  // Code to handle the event.
               }
```

请注意，编译器将自动将对象映射到其类型。这在许多方面都很方便，因为它允许您用C＃编写仅与对象一起保留的内联函数，除非您也想在其他任何地方使用它们。但是，这种处理事件的方法有一个缺点：一旦附加了事件处理程序，便无法删除它。在C＃中可以，`-+`。

但是由于我们没有删除事件的参考，因此只能使用单独的函数。**但是**，如果不必删除处理程序，则应始终考虑在程序中使用这种事件处理方式。

### 2.将Lambda用于仅Getter的属性

在C＃中，有一个使用属性而不是字段的概念。您可以控制如何设置值以及如何从字段中捕获值。将其视为Java编程语言的getter和setter方法的替代方法（或类似方法）。唯一的区别是您不必在某个地方分别编写它们，它们直接写在字段本身的前面。然后，C＃程序编译器将创建自己的后备字段，用于存储值。

基本上，您必须编写如下这样的属性：

```c#
public string Name { get; }
```

请注意，这些属性是恒定的，设置后就无法更改。它们是在构造函数中设置的，或者（从C＃6开始）在它们的前面设置。像这样：

```c#
public string Name { get; } = "Afzaal Ahmad Zeeshan";
```

但是，由于我们已经知道这是一个常量字段，您不能修改它，那么为什么不创建一个简单的常量属性呢？事情变得有些棘手。甚至一个属性也必须由字段来备份。在这种情况下，这将为我们解决问题：

```c#c#
public string Name => "Afzaal Ahmad Zeeshan";
```

这等效于编写以下内容：

```c#
public string Name { get { return "Afzaal Ahmad Zeeshan"; } }
```

但是由于编译期将getter字段转换为常量字段，并且在必须调用此属性的时候才会在程序中使用该字段，因此性能要好得多。

## 最后的话

本指南系列的目的是使您了解一些使程序更易于阅读和更好执行的方法。C＃编译器本身会尽最大努力提高代码的质量和效率，而这程序员带来便利，同时也将使程序更好地工作。

除了上面提到的方法，还有许多其他提高可读性的方法，其中许多方法适合公司团队协作的形式编写程序，因为大多数团队往往都要求程序员遵循自己的编程方法和方式。

