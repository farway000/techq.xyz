# **LINQ：最终统治了所有的语言！**

 **让我们看看LINQ如何彻底改变了.NET中访问数据的方式** 

NETQ与其他技术堆栈的不同之处之一绝对是LINQ，它是Language Integrated Query的首字母缩写。实际上，它是随.NET Framework 3.5和Visual Studio 2008引入的，它是第一个独立于体系结构并集成在C＃和Visual Basic语言中的框架。

借助LINQ，我们可以使用独立于各种源的单个编程模型来查询和操作数据。为了更好地理解它是什么，我们必须跳过去。

在C＃的第一个版本中，我们必须使用*for*或*foreach*循环来遍历一个集合，正如我们所知，该集合实现*IEnumerable*接口，例如，在其中找到一个特定的对象。以下代码返回公司年龄在19至36岁（20至35岁）之间的所有客户：

```c#
class Customer
{ 
   public int CustomerID { get; set; } 
   public String CustomerName { get; set; } 
   public int Age { get; set; } 
} 
 
class Program 
{ 
   static void Main(string[] args) 
   { 
       Customer[] customerArray = { 
          new Customer() { CustomerID = 1, CustomerName = "Joy", Age = 22 }, 
          new Customer() { CustomerID = 2, CustomerName = "Bob", Age = 45 }, 
          new Customer() { CustomerID = 3, CustomerName = "Curt", Age = 25 },
       };
        
       Customer[] customers = new Customer[10]; 
        
       int i = 0; 
        
       foreach (Customer cst in customerArray) 
       { 
           if (cst.Age > 19 && cst.Age < 36) 
           { 
               customers[i] = cst; 
               i++; 
           }
       } 
   } 
}
```

 有什么不同的方法？让我们尝试从“ *委托”*的概念开始逐步进行开发。甲*代表*是表示与相同的参数和返回的类型的方法的引用类型。它“委托”它旨在执行代码的方法，我们可以用这种方式声明它：  

```
public delegate bool Operations(int number);
```

 此*委托*可以指向所有接受输入整数并返回布尔值的方法。例如，假设在*CustomerOperations*类中有一个方法： 

```
public bool CustomerAgeRangeCheck(int number)
{
    return number > 19 && number < 36; 
}
```

 我们可以注册一个或多个将在执行*委托时*执行的方法： 

```
Operations op = new Operations(CustomerOperations.CustomerAgeRangeCheck);
```

 或者简单地： 

```
 Operations op = CustomerOperations.CustomerAgeRangeCheck; 
```

 因此，我们可以使用*委托*，在这种情况下，它将返回*true*： 

```
 op(22); 
```

*委托*用于将方法作为参数传递给其他方法：*事件处理程序*和*回调*是通过*委托*调用的方法的示例。

C＃2.0引入了*匿名委托*，您现在可以使用匿名方法来声明和初始化*委托*。例如，我们可以这样写：

```
delegate bool CustomerFilters(Customer customer);
  
class CustomerOperations
{
    public static Customer[] FindWhere(Customer[] customers, CustomerFilters  customerFiltersDelegate)
    {
        int i = 0;
        Customer[] result = new Customer[6];
        foreach (Customer customer in customers)
           if (customerFiltersDelegate(customer))
           {
               result[i] = customer;
                i++;
           }
    
        return result;
    }
}
  
class Program
{
    static void Main(string[] args)
    {
        Customer[] customers = {
           new Customer() { CustomerID = 1, CustomerName = "Joy", Age = 22 },
           new Customer() { CustomerID = 2, CustomerName = "Bob", Age = 45 },
           new Customer() { CustomerID = 3, CustomerName = "Curt", Age = 25 },
        };
 
        Customer[] filteredCustomersAge = CustomerOperations.FindWhere(customers, delegate (Customer customer)  //Using anonimous delegate
           {
              return customer.Age > 19 && customer.Age < 36;
           });
    }
}
```

 使用C＃2.0，我们的优势是可以使用*匿名委托*在不同条件下进行搜索，而无需使用*for*或*foreach*循环。例如，我们可以使用上一个示例中相同的委托函数来查找“ CustomerID”为3或名称为“ Bob”的客户： 

```
Customer[] filteredCustomersId = CustomerOperations.FindWhere(customers, delegate (Customer customer)  
          {
             return customer.CustomerID == 3;
          });
 
Customer[] filteredCustomersName = CustomerOperations.FindWhere(customers, delegate (Customer customer)  
          {
             return customer.CustomerName == "Bob";
          });
```

随着C＃的发展，从3.x版本开始，Microsoft团队引入了新功能，使代码更加紧凑和易读。这些直接支持LINQ来查询不同类型的数据源并获得产生单个指令的元素。

这些功能是：

  -在*VAR*结构，一个隐式类型的局部变量。它是强类型化的，因为已经声明了类型本身，但是由编译器根据分配给它的值使用类型推断来确定类型。以下两个语句在功能上等效：

```
var customerAge = 30; // Implicitly typed.
int customerAge = 30; // Explicitly typed.
```

  -使用 *对象初始化程序，* 您可以在对象创建期间将值分配给*对象的*全部或某些属性，而无需在分配指令行之后调用构造函数。 

```
Customer customer = new Customer { Age = 30, CustomerName = "Adolfo" };
```

 与以下代码不同，在前一种情况下，所有内容都被视为单个操作。 

```
Customer customer = new Customer();
customer.Age = 30; 
customer.CustomerName = "Adolfo";
```

  \- *匿名类型*，由编译器构建的只读类型，只有编译器知道它。但是，如果*程序*集中的两个或多个匿名*对象初始化*程序具有相同顺序的属性序列，并且具有相同的名称和类型，则编译器会将这些对象视为相同类型的实例。*匿名类型*是将查询结果中的一组属性临时分组的好方法，而不必定义单独的命名类型。 

```
var customer = new { YearsOfFidelity = 10, Name = "Francesco"}; 
```

   -扩展方法，使您可以将方法“添加”到现有类型，而无需创建新的派生类型，重新编译或修改原始类型。扩展方法是静态方法，但由于引入了语法糖，因此被称为，因为它们是扩展类型上的实例方法。 

```
public static class StringExtensionMethods
{
    public static string ReverseString(this string input)
    {
        if (string.IsNullOrEmpty(input)) return "";
        return new string(input.ToCharArray().Reverse().ToArray());
    }
}
```

 *扩展方法*必须在静态类中定义。第一个参数表示要扩展的类型，并且必须以关键字*this*开头，其他参数则不需要它。 

```
Console.WriteLine("Hello".ReverseString());   //olleH
```

 请注意，在方法调用中不得指定第一个参数，该参数以*this*修饰符开头。 

  \- *Lambda表达式*，可作为可变的或作为在一方法调用中的参数被传递匿名函数。 

```
customer => customer.Age > 19 && customer.Age < 36;
```

=>运算符称为*lambda运算符*，而*customer*是函数的输入参数。*lambda运算符*右侧的部分代表函数的主体及其返回的值，在这种情况下为布尔值。

在LINQ的引入中，我们终于有了C＃3.5版本。

简而言之，我们可以说LINQ是IEnumerable <T>和IQueryable <T>接口的*扩展方法*库，它使我们能够执行各种操作，如过滤，进行投影，聚合和排序。

我们有几种可用的LINQ实现：

- LINQ到对象（内存中对象集合）
- LINQ到实体（实体框架）
- LINQ to SQL（SQL数据库）
- LINQ to XML（XML文档）
- LINQ到数据集（ADO.Net数据集）
- 通过实现IQueryable接口（其他数据源）

在前面的示例中，数组用作数据源，因此隐式支持通用接口IEnumerable <T <。支持IEnumerable <T>或其派生接口的类型（例如通用IQueryable <T>接口）称为*可查询类型*，使我们可以直接执行LINQ查询。如果数据源尚未以*可查询类型*存储在内存中，则LINQ提供程序必须将其表示为*可查询类型*。

正如我们所说，LINQ查询主要基于.NET Framework 2.0版中引入的通用类型。这意味着，例如，如果尝试将Customer对象添加到List <string>对象，则在编译时将生成错误。使用通用集合很容易，因为不需要在运行时强制转换类型。
如果愿意，可以使用前面提到的*var*关键字来避免通用语法，在下面的示例中，该关键字要求编译器通过检查*from*子句中指定的数据源来推断查询变量的类型。
因此，让我们看看如何能达到同样的效果，在前面的代码中我们获得了使用*匿名委托*，使用LINQ到对象查询，该*变种*构造和*lambda表达式*：

```
var filteredCustomersAge = customers.Where(c => c.Age > 19 && c.Age < 36);
```

 这种语法称为方法语法。
在下一个示例中，我们将使用查询语法（Query Syntax），该语法是为那些已经了解SQL语言并且因此会喜欢这种方法的人引入的： 

```
var filteredCustomersAge =
    from customer in customers
    where customer.Age > 19 && customer.Age < 36
    select customer;
```

查询语法和方法语法在语义上是相同的，许多人发现查询的语法更简单，更易于阅读。
在查询语法中，LINQ查询运算符在编译时转换为对相关LINQ *扩展方法的*调用。

在下一篇文章中，我们将继续讨论LINQ！
我们将讨论IQueryable <T>接口，其相关的LINQ扩展方法以及与IEnumerable <T>接口的区别。
与远程数据库一样，我们还将看到LINQ与内存外集合的数据源一起使用。