---
title:  .TDD学习笔记（二）单元测试
date: 2020-4-30 19:07
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# 单元测试
## 定义
单元测试最早来源于Kent Beck，他在开发SmallTalk中引入了这个概念，随着软件工程学的不断发展，使得单元测试已经成为软件编程中一项非常有用的实践。

在维基百科中，“单元测试”是这样定义的：

>一个单元测试是一段代码（通常是一个方法），这段代码调用另一段代码，然后检验某些假设的正确性。如果这些假设是错误的，单元测试就失败了。一个单元可以是一个方法或一个函数。

  而《单元测试的艺术》作者Roy Osherove则认为，一个单元不仅仅是一个方法，也有可能是包括实现某个功能的多个类和函数。

## 什么是好的单元测试
Roy Osherove同时也认为，一个单元测试应该具备以下特征：

1. 它应该是自动化的，可重复执行。
2. 它应该很容易实现；
3. 它应该第二天还有意义；
4. 任何人都应该能一键运行它；
5. 它应该运行很快；
6. 它的结果应该是稳定的（如果运行之间没有进行修改的话，多次运行一个测试应该总是能够返回同样的结果）
7. 它应该能完全控制被测试的单元；
8. 它应该是完全隔离的（独立于其他测试的运行）；
9. 如果它失败了，我们应该很容易发现什么是期待的结果，进而定位问题所在。
# 几种概念
在[这篇博客](https://blog.pragmatists.com/test-doubles-fakes-mocks-and-stubs-1a7491dfa3da)中，作者对Fake、Mock、Stub进行了对比。

## Fakes（伪造）
![图片](https://uploader.shimo.im/f/9bhjo93eYX4y1Mbt.png!thumbnail)

Fake创建的对象，看似跟原对象一致，但是简化了原来对象的某些行为，使得我们在进行代码过程中，无需通过启动数据库或其他外部组件，即可对服务进行集成测试。

## Mock（模拟）
![图片](https://uploader.shimo.im/f/OLf5e8HV6yUrw7uC.png!thumbnail)

mock是在调用方法中，注入“模拟”的完整的被调用者对象，并在Test方法中，通过注入的这个模拟对象来执行对应的操作。

## Stub（打桩）
![图片](https://uploader.shimo.im/f/X7xnMFmBKoQQXb7C.png!thumbnail)

存根是预先定义一个方法的返回值，以便我们在调用该方法时，返回存根对象，这样使我们的代码不会以不改变原方法、或对原方法产生副作用的情况下，实现某方法。

## 横切关注点
>部分关注点「横切」程序代码中的数个模块，即在多个模块中都有出现，它们即被称作「横切关注点（Cross-cutting concerns, Horizontal concerns）」。

横切关注点也是面向对象编程中的概念，我们通俗意义上理解的AOP框架，可以理解为解决横切关注点问题的一种框架。

日志、异常处理、服务调用、方法调用链路都是大家会遇到的一类关注点问题，而而在《单元测试的艺术》这本书中，作者也指出“时间”（DateTime）也同样是一种问题。例如，如果我们在代码中普遍使用了系统默认的DateTime.Now，那么假设我们要测试代码在元旦和非元旦日期中的不同行为时，是不是手动把系统时间修改为指定的时间？这显然是的代码不利于维护，也不利于代码的可测试性。

通过定义了一个SystemTime 对象来解决这个问题，确实是一种非常不错的方法。

```
 [TestFixture]
    public class TimeLoggerTests
    {
        [Test]
        public void SettingingSystemTime_Always_ChangesTime()
        {
            SystemTime.Set(new DateTime(2000, 1, 1));
            string output = TimeLogger.CreateMessage("a");
            StringAssert.Contains("01.01.2000", output);
        }
    }
 public class SystemTime
    {
        private static DateTime _dateTime;
        public static void Set(DateTime custom)
        {
            _dateTime = custom;
        }
        public static void Reset()
        {
            _dateTime = DateTime.MinValue;
        }
        public static DateTime Now
        {
            get
            {
                if (_dateTime != DateTime.MinValue)
                {
                    return _dateTime;
                }
                return DateTime.Now;
            }
        }
        
    }
```
# 测试框架
测试框架是用来辅助开发者进行单元测试的代码库。在.NET开发环境下，我们常见的的测试框架可以分成以下两种类型：

## 单元测试框架
单元测试框架框架是帮助开发者进行单元测试的代码库和模块，它也可以作为自动编译过程的一个步骤运行测试。单元测试的框架[如此之多](https://en.wikipedia.org/wiki/List_of_unit_testing_frameworks)，而在.NET中，常见的主要包括这几种：

1、MSTest:这是Visual Studio中最常见的测试框架，在除Visual Studio2019以前的版本中，创建的单元测试项目自带的就是这种测试框架。

2、XUnit:XUnit是一个大家族，在Java、.NET、等多种技术语言下都有XUnit的身影。

3、NUnit:在许多介绍单元测试的书籍中，都会采用NUnit作为示例，在本文中，也主要介绍这种框架。

## 隔离（模拟）框架   
隔离（模拟）是一种可编程的API，使用这种API可以使得创建为对象比手工编写简便、快速和容易。常见的隔离（模拟）框架包括以下几种：

1、Moq：在.NET中常见的Mock框架。

2、NSubstitute：在《单元测试的艺术》一书中，作者Roy Osherove着重介绍过这种测试隔离框架，也经常和Moq框架一起进行[比较](https://itenium.be/blog/dotnet/nsubstitute-vs-moq/)。

3、Microsoft Fakes:也是一种模拟框架，经常被用于和上述模拟框架[对比](https://saucelabs.com/blog/mock-frameworks-vs-microsoft-fakes)。

4、FakeItEasy、EasyMoq、JustMock框架：其他模拟框架。 

# 编写良好测试代码中常见的问题
## 如何给测试方法命名
方法的命名一直是困扰开发者的难题，尤其是单元测试方法。我们该如何给单元测试方法命名呢？目前我了解到两种不同的命名方法：

假设，现有一个新增方法为：

```
public int Add(int x,int y)
```
一种是Should开头的单元测试命名方法，可以命名为
```
Should_Returns_Sum_When_Add_Numbers();
```
另外一种是在《单元测试的艺术》这本书中作者用到的命名方法，作者将单元测试命名为三个部分，分别为：被测试方法名，测试场景，预期行为，将三个部分用下划线“_”分开，例如MethodUnderTest_Scenario_Behavior()。按照这个命名方法，上述方法可以被命名为：
```
Add_Nums_Returns_ResultsOfInteger();
```
## 静态类或单例如何进行单元测试
### 静态类
在.NET Framework中经常互相静态类和静态对象，这无形中给我们的单元测试过程带来了不少困扰。我们可以采取以下方式对这些静态类进行测试。

1、静态类应该只限于静态的方法，例如像StringExtension这样的扩展方法，这种方式是可以直接进行测试的。

2、对于历史代码中为包含不少静态成员的“静态”对象，应该将其改成有IoC框架注入的单例对象，这样就能使用mock的方式进行单元测试。

3、对于无法修改的静态对象，我们可以考虑将其隔离。

### 单例
而对于单例代码，则可以采用将单例逻辑和单例持有者分开的方式，让代码更易于测试。

```
public class MySingleton
{
       private static MySingleton _instance;
       public static MySingleton Instance
       {
           get
           {
               if (_instance == null)
               {
                   _instance = new MySingleton();
               }
               return _instance;
           }
       }
       public void Foo()
       {


       }
}
```
修改后：

```
 public class MySingletonLogic
    { 
        public void Foo()
        {


        }
    }
    public class MySingletonHolder
    {
        private static MySingletonLogic _instance;
        public static MySingletonLogic Instance
        {
            get
            {
                if (_instance == null)
                {
                    _instance = new MySingletonLogic();
                }
                return _instance;
            }
        } 
    }
```
通过这种方式的改造，使得我们能够非常方便的对Foo方法进行测试了。
### 何时开始进行单元测试？
最好的时机就是当下，当你需要键入一行逻辑代码时，先写一个测试方法，按照TDD的流程进行开发，将有利于你的代码开发过程处于“自信满满”的状态，而且还能减少代码调试的时间，进而提高代码开发的效率。

