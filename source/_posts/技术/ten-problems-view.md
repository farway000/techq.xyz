---
title:  15个基本的C＃面试问题
date: 2020-03-26 08:54
tags: 技术
author: 译者：笑语
categories:
  - 技术
---


1、给定一个int数组，编写方法以统计所有偶数的值。

有很多方法可以做到这一点，但是最直接的两种方法是：

```
static long TotalAllEvenNumbers(int[] intArray) {
  return intArray.Where(i => i % 2 == 0).Sum(i => (long)i);
}
```
还有就是
```
static long TotalAllEvenNumbers(int[] intArray) {
  return (from i in intArray where i % 2 == 0 select (long)i).Sum();
}
```
当然，你还需要注意以下关键：
1. 你是否利用 C＃语言特性 一行就解决问题。（即，不是使用包含循环，条件语句和累加器的更长篇幅的解决方案）
2. 你是否考虑过溢出的可能性。例如，诸如 

     return intArray.Where(i => i % 2 == 0).Sum()（与函数的返回类型无关）

这可能一个很"明显"的单行，但这样溢出的可能性很高。虽然上面的答案中使用的转换为long的方法并没有消除这种可能性，但是它使得发生溢出异常的可能性非常小。但请注意，如果你写答案的时候询问数组的预期大小及其成员的大小，则显然你在做这道题目的时候在考虑此溢出问题，这很棒。

# 2、下面的代码的输出是什么？解释你的答案。
```
class Program {
  static String location;
  static DateTime time;
 
  static void Main() {
    Console.WriteLine(location == null ? "location is null" : location);
    Console.WriteLine(time == null ? "time is null" : time.ToString());
  }
}
```
输出将是：

```
location is null
1/1/0001 12:00:00 AM
```
下面的简短程序的输出是什么？解释你的答案。简短程序的输出是什么？解释你的答案。
尽管两个变量都未初始化，但是String是引用类型 、DateTime 是值类型。作为值类型，单位化DateTime变量设置为默认值  公元1年晚上12点，*而不是* null 

# 3、下面语句中 time 和null 的比较是有效还是无效的?
```
static DateTime time;
/* ... */
if (time == null)
{
	/* do something */
}
```
有人可能会认为，由于变量永远不可能为null (它被自动初始化为1月1日的值)，所以编译器在比较某个变量时就会报错。具体来说，操作符将其操作数强制转换为不同的允许类型，以便在两边都得到一个通用类型，然后可以对其进行比较。这就是为什么像这样的东西会给你期望的结果(而不是失败或意外的行为，因为操作数是不同的类型):

```
double x = 5.0;
int y = 5;
Console.WriteLine(x == y);  // outputs true
```
然而，这有时会导致意外的行为，例如DateTime变量和null的比较。在这种情况下，DateTime变量和null文字都可以转换为可空的。因此，比较这两个值是合法的，即使结果总是假的。
# 4、给定circle以下类的实例：
```
public sealed class Circle {
  private double radius;
  
  public double Calculate(Func<double, double> op) {
    return op(radius);
  }
}
```
 简编写代码以计算圆的周长，而无需修改Circle类本身。
首选的答案如下:

```
circle.Calculate(r => 2 * Math.PI * r);
```
由于我们不能访问对象的私有半径字段，所以我们通过内联传递计算函数，让对象本身计算周长。

许多c#程序员回避(或不理解)函数值参数。虽然在这种情况下，这个例子有点做作，但其目的是看看申请人是否了解如何制定一个调用来计算哪个与方法的定义相匹配。

另外，一个有效的(虽然不那么优雅的)解决方案是从对象中检索半径值本身，然后执行计算结果:

```
var radius = circle.Calculate(r => r);
var circumference = 2 * Math.PI * radius;
```
无论哪种方式。我们在这里主要寻找的是面试者是否熟悉并理解如何调用Calculate方法。


# 5、下面程序的输出是什么?解释你的答案。
```
class Program {
  private static string result;
 
  static void Main() {
    SaySomething();
    Console.WriteLine(result);
  }
 
  static async Task<string> SaySomething() {
    await Task.Delay(5);
    result = "Hello world!";
    return “Something”;
  }
```
下面
此外，如果我们替换wait task，答案会改变吗? 比如 thread . sleep (5) ? 为什么?的简短

程序的输出是什么？解释你的答案。序的输出是什么？解释你的答案。

回答：

问题第一部分（即带有的代码版本await Task.Delay(5);）的答案是该程序将仅输出一个空行（而不是 “ Hello world！”）。这是因为调用result时仍将未初始化Console.WriteLine。

大多数程序和面向对象的程序员都希望函数return在返回调用函数之前从头到尾执行，或者从语句执行。C＃async函数不是这种情况。它们只执行到第一个await语句，然后返回到调用方。由await（在此例中为Task.Delay）调用的函数是异步执行的，并且该await语句之后的行直到Task.Delay完成（在5毫秒内）之前都不会发出信号。但是，在这段时间内，控制权已经返回给调用者，该调用者Console.WriteLine对尚未初始化的字符串执行该语句。

调用await Task.Delay(5) 可让当前线程继续其正在执行的操作，如果已完成（等待任何等待），则将其返回到线程池。这是异步/等待机制的主要好处。它允许CLR使用线程池中的更少线程来服务更多请求。

异步编程已经变得越来越普遍，因为执行许多活动的网络服务请求或数据库请求的设备越来越普遍。C＃具有一些出色的编程结构，可以极大地简化异步方法的编程任务，并且意识到它们的程序员将产生更好的程序。

关于问题的第二部分，如果将await Task.Delay(5);其替换为Thread.Sleep(5)，则程序将输出Hello world!。一种没有至少一个语句的async方法，其操作就像同步方法一样。也就是说，它将从头到尾执行，或者直到遇到一条语句为止。调用只是阻塞了当前正在运行的线程，因此调用仅将方法的执行时间增加了5毫秒。awaitreturnThread.Sleep()Thread.Sleep(5)SaySomething()


# 6、下面的程序输出是什么？解释你的答案。
```
delegate void Printer();

static void Main()
{
        List<Printer> printers = new List<Printer>();
        int i=0;
        for(; i < 10; i++)
        {
            printers.Add(delegate { Console.WriteLine(i); });
        }

        foreach (var printer in printers)
        {
            printer();
        }
}
```
这个程序将把数字10输出十次。

原因如下: 委托被添加到 for循环中l了，而 “引用” (或者“指针”)被存储到i中，而不是值本身。因此，在我们退出循环之后，变量i被设置为10，所以到调用每个委托时，传递给它们的值都是10。

# 7、是否可以将混合数据类型（例如int，string，float，char）全部存储在一个数组中？
是! 之所以可以这样做，是因为数组的类型object不仅可以存储任何数据类型，还可以存储类的对象，如下所示：

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace ConsoleApplication8
{
    class Program
    {
        class Customer
        {
            public int ID { get; set; }
            public string Name { get; set; }
            public override string ToString()
            {
                return this.Name;
            }
        }
        static void Main(string[] args)
        {
            object[] array = new object[3];
            array[0] = 101;
            array[1] = "C#";
            Customer c = new Customer();
            c.ID = 55;
            c.Name = "Manish";
            array[2] = c;
            foreach (object obj in array)
            {
                Console.WriteLine(obj);
            }
            Console.ReadLine();
        }
    }
}
```
# 8、比较C＃中的结构和类。他们有什么共同点？它们有何不同？
C＃中的类和结构确实有一些共同点，即：

他们都是

是复合数据类型

可以包含方法和事件

可以支持接口

但是有许多差异。比较一下：

**类：**

支持继承

是引用（指针）类型

引用可以为空

每个新实例都有内存开销

**结构：**

不支持继承

是值类型

按值传递（如整数）

不能有空引用（除非使用了Nullable）

每个新实例没有内存开销（除非“装箱”）

# 9、这里有一个包含一个或多个$符号的字串，例如:
```
"foo bar foo $ bar $ foo bar $ "
```
问题：如何$从给定的字符串中删除第二和第三次出现的？
答案：

使用如下正则表达式：

```
string s = "like for example $  you don't have $  network $  access";       
Regex rgx = new Regex("\\$\\s+");
s = Regex.Replace(s, @"(\$\s+.*?)\$\s+", "$1$$");
Console.WriteLine("string is: {0}",s);
```
说明：
* (\$\s+.*?)-第1组，捕获一个文字$，一个或多个空格字符，然后捕获除换行符以外的任意数量的字符，并尽可能少地捕获到下一个最接近的匹配项
* \$\s+—单个$符号和一个或多个空格字符
* $1引用组1的值，它只是将其插入被替换的字符串中，$$代表替换模式中的$符号。
# 10、下面的程序输出是什么？
```
public class TestStatic
    {
        public static int TestValue;

        public TestStatic()
        {
            if (TestValue == 0)
            {
                TestValue = 5;
            }
        }
        static TestStatic()
        {
            if (TestValue == 0)
            {
                TestValue = 10;
            }
        }

        public void Print()
        {
            if (TestValue == 5)
            {
                TestValue = 6;                
            }
            Console.WriteLine("TestValue : " + TestValue);

        } 
    }

 public void Main(string[] args)
        {
            TestStatic t = new TestStatic();
            t.Print();
        }
```
TestValue : 10

在创建该类的任何实例之前，将调用该类的静态构造函数。此处调用的静态构造函数TestValue首先将变量初始化。

# 11、有没有一种方法可以修改ClassA、以便您可以在调用Main方法时使用参数调用构造函数，而无需创建任何其他新实例ClassA？
```
class ClassA
{
  public ClassA() { }

  public ClassA(int pValue) {  }
}
```
启动类
```
class Program
{
  static void Main(string[] args)
  {
    ClassA refA = new ClassA();
  }
}
```
回答：

所述this关键字被用于调用其他构造，初始化该类对象。下面是实现：

```
class ClassA
{
  public ClassA() : this(10)
  { }

  public ClassA(int pValue)
  {  }
}
```
# 12、以下代码输出什么？
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace main1
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                Console.WriteLine("Hello");
            }
            catch (ArgumentNullException)
            {
                Console.WriteLine("A");
            }
            catch (Exception)
            {
                Console.WriteLine("B");
            }
            finally
            {
                Console.WriteLine("C");
            }
            Console.ReadKey();
        }
    }
}
```
答案：

```
Hello
C
```
# 13、描述依赖注入。
依赖注入是一种使紧密链接的类分离的方式，从而减少了类之间的直接依赖。有多种方法可以实现依赖项注入：

1. 构造函数依赖
2. 属性依赖
3. 方法依赖
# 14、编写一个C＃程序，该程序接受以千米为单位的距离，将其转换为米，然后显示结果。
```
using system; 

class abc   

{   
    public static Void Main()   
    
    {
      
            int ndistance, nresult;  
            
        Console.WriteLine("Enter the distance in kilometers");  
        
        ndistance = convert.ToInt32(Console.ReadLine());  
        
        nresult = ndistance * 1000;
          
        Console.WriteLine("Distance in meters: " + nresult);  
        
        Console.ReadLine();  
        
    }  
    
}  
```

# 15、描述装箱和拆箱。并写一个例子。
装箱是将值类型隐式转换为该类型object或该值类型实现的任何接口类型。将值类型装箱会创建一个包含该值的对象实例，并将其存储在堆中。

例：

```
int x = 101;
object o = x;  // boxing value of x into object o

o = 999;
x = (int)o;    // unboxing value of o into integer x
```
# 最后：
面试不仅要基础扎实，更重要的是能解决棘手的技术问题，所以以上这些内容仅供参考。并非每个值得招聘的优秀候选人都能够回答所有问题，也不能确定能够全部回答，就能保证他是一个优秀候选人。归根结底，招聘仍然是一门艺术，一门科学以及许多工作。

如果你有招聘的要求，也欢迎和我们公众号联系，我们有12万的粉丝，相信能在其中找到适合您公司的 .net 候选人。

恭喜你！全部看完，看来您高手只有一步之遥，赶紧转发朋友圈吧！让其他.net 新手也来瞻仰瞻仰。 

