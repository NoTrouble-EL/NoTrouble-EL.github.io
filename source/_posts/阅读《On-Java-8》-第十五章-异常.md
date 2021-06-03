---
title: 阅读《On Java 8》-- 第十五章 异常
date: 2021-06-02 22:43:09
tags:
- JAVASE
- 《On Java 8》
categories: JAVASE
mathjax: true
---

​		改进的错误恢复机制是提高代码健壮性的最强有力的方式。错误恢复在我们所编写的每一个程序中都是基本的要素，但是在Java中它显得格外重要，因为Java的主要目标之一就是创建供他人使用的程序构件。

​		发现错误的理想时机是在编译阶段，也就是在你试图运行程序之前。然而，编译期间并不能找出所有错误，余下的问题必须在运行期间解决。这就需要错误源能通过某种方式，把适当的信息传递给某个接收者——该接收者将知道如何正确处理这个问题。

​		Java使用异常来提供一致的错误报告模型，使得构建能够与客户端代码可靠地沟通问题。

​		Java中的异常处理的目的在于通过使用少于目前数量的代码来简化大型、可靠的程序的生成，并且通过这种方式可以使你更加确信：你的应用中没有未处理的错误。

​		因为异常处理是Java中唯一官方的错误报告机制，并且通过编译器强制执行。

 <!-- more --> 

## 异常概念

​		C以及其他早期语言常常具有多种错误处理模式，这些模式往往建立在约定俗成的基础之上，而并不属于语言的一部分。通常会返回某个特殊值或设置某个标志，并且假定接收者将对这个返回值或标志进行检查，以判定是否发生了错误。然而，随着时间的推移，人们发现，高傲的程序员在使用程序库的时候更倾向于认为：“对，错误也许会发生，但那是别人造成的，不关我的事”。所以，程序员不去检察错误情形也就不足为奇了(何况对某些错误情形的检查确实很无聊)。如果的确在每次调用方法的时候都彻底进行错误检查，代码很可能会变得难以阅读。正是由于程序员还仍然用这些方式拼凑系统，所以他们拒绝承认这样一个事实：对于构造大型、健壮、可维护的程序而言，这种错误处理模式已经成为了主要障碍。

​		解决的办法是，用强制规定的形式来消除错误处理过程中随心所欲的因素。这种做法由来已久，对异常处理的实现可以追溯到20世纪60年代的操作系统，甚至于BASIC语言中的“one error goto”语句。而C++的异常处理机制基于Ada，Java中的异常处理机制则建立在C++的基础之上。

​		“异常”这个词有“我对此感到意外”的意思。问题出现了，你也许不清楚该如何处理，但你的确知道不应该置之不理，你要停下来，看看是不是有别人或在别的地方，能够处理这个问题。只是在当前的环境中还没有足够的信息来解决这个问题，所以就把这个问题提交到一个更高级别的环境中，在那里将作出正确的决定。

​		异常往往能够降低错误处理代码的复杂度。如果不使用异常，那么就必须检查特定的错误吗，并在程序中的许多地方去处理它。而如果使用异常，那就不必在方法调用处进行检查，因为异常机制将保证能够捕获这个错误。理想情况下，只需要在一个地方处理错误，即所谓的异常处理程序中。这种方式不仅节省代码，而且把“描述在正常执行过程中做什么事”的代码和“除了问题怎么办”的代码相分离。总之，与之前的错误处理方法相比，异常机制使代码的阅读、编写和调试工作更加井井有条。

## 基本异常

​		异常情形(exceptional condition)是指阻止当前方法或作用域继续执行的问题。把异常情形和普通问题相区分很重要，所谓的普通问题是指，在当前环境下能得到足够的信息，总能处理这个错误。而对于异常情形，就不能继续下去了，因为在当前环境中无法获取必要的信息来解决问题。你所能做的就是从当前环境跳出，并且把问题提交给上一级环境。这就是抛出异常时所发生的事情。

​		除法就是一个简单的例子。除数有可能为0，所以先进行检查很有必要。但除数为0代表的究竟是什么意思呢？通过当前正在解决的问题环境，或许能知道该如何处理除数为0的情况。但如果这是一个意料之外的值，你也不清楚该如何处理，那么就要抛出异常，而不是顺着原来的路径继续执行下去。

​		当抛出异常后，有几件事会随之发生。首先，同Java中其他对象的创建一样，将使用new在堆上创建异常对象。然后，当前的执行路径被终止，并且从当前环境中弹出对异常对象的引用。此时，异常处理机制接管程序，并开始寻找一个恰当的地方来继续执行程序。这个恰当的地方就是异常处理程序，它的任务就是将程序从错误状态中恢复，以使程序能要么换一种方式运行，要么继续运行下去。

​		举一个抛出异常的简单例子。对于引用t，传给你的时候可能尚未被初始化。所以在使用这个对象引用调用其方法之前，会先对引用进行检查。可以创建一个代表错误信息的对象，并且将它从当前环境中“抛出”，这样就把错误信息传播到了“更大”的环境中。这被称为抛出一个异常，看起来像这样：

```java
if(t == null)
    throw new NullPointerException();
```

​		这就是抛出了异常，于是在当前环境下就不必再为这个问题操心了，它将在别的地方得到处理。

​		异常允许你将做的每件事都当作一个事务来考虑，而异常守护着这些事务：“事务的基本保障是，我们需要的分布式计算的异常处理机制。事务相当于计算机中的合同法。如果任何事出现了错误，我们只需要丢弃整个计算。”你也可以将异常看作一种内建的“恢复”(undo)系统，因为你在使用程序中可以有各种恢复点。一旦程序的一个部分失败了，异常将“恢复”到一个已知的稳定点上。

### 异常参数

​		与使用Java中的其他对象一样，我们总是用new在堆上创建异常，这也伴着存储空间的分配和构造器的调用。所有标准异常类都有两个构造器：一个是无参构造器；另一个是接受字符串作为参数，以便能把相关信息放入异常对象的构造器：

```java
throw new NullPointerException("t = null");
```

​		关键字throw将产生许多有趣的结果。在使用new创建了异常对象之后，此对象的引用将传给throw。尽管异常对象的类型通常与方法设计的返回类型不同，但从效果上看，它就像是从方法“返回”的。可以简单地把异常处理看成一种不同的返回机制，当然若过分强调这种类比的话，就会有麻烦了。另外还能用抛出异常的方式从当前的作用域退出。在这两种情况下，将会返回一个异常对象，然后退出方法或作用域。

​		抛出异常与方法正常返回的相似之处到此为止。因为异常返回的“地点”与普通方法调用返回的“地点”完全不同。(异常将一个恰当的异常处理程序中得到解决，它的位置可能离异常被抛出的地方很远，也可能会跨越方法调用栈的许多层级。)

​		此外，能够抛出任意类型的Throwable对象，它是异常类型的根类。通常，对于不同类型的错误，要抛出相应的异常。错误信息可以保存在异常对象内部或者用异常类的名称来暗示。上一层环境通过这些信息来决定如何处理异常。

## 异常捕获

​		要明白异常是如何被捕获的，必须首先理解监控区域(guarded region)的概念。它是一段可能产生异常的代码，并且后面跟着处理这些异常的代码。

### try语句块

​		如果在方法内部抛出了异常(或者在方法内部调用的的其他方法抛出了异常)，这个方法将在抛出异常的过程中结束。要是不希望方法就此结束，可以在方法内设置一个特殊的块来捕获异常。因为在这个块里“尝试”各种(可能产生异常的)方法调用，所以称为try块。它是跟在try关键字之后的普通程序块：

```java
try {
    // Code that might generate exceptions
}
```

​		对于不支持异常处理的程序语言，要想仔细检查错误，就得在每个方法调用的前后加上设置和错误检查的代码，设置在每次调用同一方法时也得这么做。有了异常处理机制，可以把所有动作都放在try块里，然后只需要在一个地方就可以捕获所有异常。这意味着你的代码将更容易编写和阅读，因为代码的意图和错误检查不是混淆在一起的。

### 异常处理程序

​		当然，抛出的异常必须在某处得到处理。这个“地点”就是异常处理程序，而且针对每个要捕获的异常，得准备相应的处理程序。异常处理程序紧跟在try块之后，以关键字catch表示：

```java
try {
    // Code that might generate exceptions
} catch(Type1 id1) {
    // Handle exceptions of Type1
} catch(Type2 id2) {
    // Handle exceptions of Type2
} catch(Type3 id3) {
    // Handle exceptions of Type3
}
// etc.
```

​		每个catch子句(异常处理程序)看起来就像是接收且仅接收一个特殊类型的参数的方法。可以在处理程序的内部使用标识符(id1, id2等等)，这与方法参数的使用很相似。有时可能用不到标识符，因为异常的类型已经给了你足够的信息来对异常进行处理，但标识符并不可以省略。

​		异常处理程序必须紧跟在try块之后。当异常被抛出时，异常处理机制将负责搜寻参数与异常类型相匹配的第一个处理程序。然后进入catch子句执行，此时认为异常得到了处理。一旦catch子句结束，则处理程序的查找过程结束。注意，只有匹配catch子句才能得到执行；这与switch语句不同，switch语句需要在每一个case后面跟一个break，以避免执行后续的case子句。

​		注意在try块内部，许多不同的方法调用可能产生类型相同的异常，而你只需要提供一个针对此类型的异常处理程序。

### 终止与恢复

​		异常处理理论上有两种基本模型。Java支持终止模式(它是Java和C++所支持的模型)。在这种模式中，将假设错误非常严重，以至于程序无法返回到异常发生的地方继续执行。一旦异常被抛出，就表明错误已无法挽回，也不能回来继续执行。

​		另一种称为恢复模型。意思是异常处理程序的工作是修正错误，然后重新尝试调用出问题的方法，并认为第二次能成功。对于恢复模型，通常希望异常被处理之后能继续执行程序。如果想要用Java实现类似恢复的行为，那么在遇见错误时就不能抛出异常，而是调用方法来修正该错误。或者，把try块放在while循环里，这样就不断地进入try块，直到得到满意的结果。

​		在过去，使用支持恢复模型异常处理的操作系统的程序员最终还是要转向使用类似“终止模型”的代码，并且忽略恢复行为。所以虽然恢复模型开始显得很吸引人，但不是很实用。其中的主要原因可能是它所导致的耦合：恢复性的处理程序需要了解异常抛出的地点，这势必要包含依赖于抛出位置的非通用性代码。这增加了代码编写和维护的困难，对于异常可能会从许多地方抛出的大型程序来说，跟是如此。

## 自定义异常

​		不必拘泥于Java已有的异常类型。Java异常体系不可能预见你将报告的所有错误，所以你可以创建自己的异常类，来表示你的程序中可能遇到的问题。

​		要自己定义异常类，必须从已有的异常类继承，最好是选择意思相近的异常类继承(不过这样的异常并不容易找)。建立新的异常类型的最简单的方法就是编译器为你产生无参构造器，所以这几乎不用写多少代码：

```java
// exceptions/InheritingExceptions.java
// Creating your own exceptions
class SimpleException extends Exception {}

public class InheritingExceptions {
    public void f() throws SimpleException {
        System.out.println(
                "Throw SimpleException from f()");
        throw new SimpleException();
    }
    public static void main(String[] args) {
        InheritingExceptions sed =
                new InheritingExceptions();
        try {
            sed.f();
        } catch(SimpleException e) {
            System.out.println("Caught it!");
        }
    }
}
//Throw SimpleException from f()
//Caught it!
```

​		编译器创建了无参构造器，它将自动调用基类的无参构造器。本例中不会得到像SimpleException(String)这样的构造器，这种构造器也不实用。你将看到，对异常来说，最重要的部分就是类名，所以本例中建立的异常类在大多数情况下已经够用了。

​		本例的结果被显示在控制台。你也可以通过写入System.err而将错误发送给标准错误流。通常这比把错误信息输出到System.out要好，因为System.out也许会被重定向。如果把结果送到System.err，它就不会随System.out一起被重定向，所以用户就更容易注意到它。

​		你也可以为异常类创建一个接受字符串参数的构造器：

```java
// exceptions/FullConstructors.java
class MyException extends Exception {
    MyException() {}
    MyException(String msg) { super(msg); }
}
public class FullConstructors {
    public static void f() throws MyException {
        System.out.println("Throwing MyException from f()");
        throw new MyException();
    }
    public static void g() throws MyException {
        System.out.println("Throwing MyException from g()");
        throw new MyException("Originated in g()");
    }
    public static void main(String[] args) {
        try {
            f();
        } catch(MyException e) {
            e.printStackTrace(System.out);
        }
        try {
            g();
        } catch(MyException e) {
            e.printStackTrace(System.out);
        }
    }
}
/**
Throwing MyException from f()
MyException
    at FullConstructors.f(FullConstructors.java:11)
    at FullConstructors.main(FullConstructors.java:19)
Throwing MyException from g()
MyException: Originated in g()
    at FullConstructors.g(FullConstructors.java:15)
    at FullConstructors.main(FullConstructors.java:24)
*/
```

​		新增的代码非常简短：两个构造器定义了MyException类型对象的创建方式。对于第二个构造器，使用super关键字明确调用了其基类构造器，它接受一个字符串作为参数。

​		在异常处理程序中，调用了在Throwable类声明的printStackTrace()方法。就像从输出中看到的，它将打印“从方法调用处直到异常抛出处”的方法调用序列。这里，信息被发送到了System.out，并自动地被捕获和显示在输出中。但是，如果调用默认版本：

```java
e.printStackTrace();
```

​		信息就会被输出到标准错误流。

### 异常与记录日志

​		你可能还想使用java.util.logging工具将输出记录到日志中。基本的日志记录功能还是相当简单易懂的：

```java
// exceptions/LoggingExceptions.java
// An exception that reports through a Logger
// {ErrorOutputExpected}
import java.util.logging.*;
import java.io.*;
class LoggingException extends Exception {
    private static Logger logger =
            Logger.getLogger("LoggingException");
    LoggingException() {
        StringWriter trace = new StringWriter();
        printStackTrace(new PrintWriter(trace));
        logger.severe(trace.toString());
    }
}
public class LoggingExceptions {
    public static void main(String[] args) {
        try {
            throw new LoggingException();
        } catch(LoggingException e) {
            System.err.println("Caught " + e);
        }
        try {
            throw new LoggingException();
        } catch(LoggingException e) {
            System.err.println("Caught " + e);
        }
    }
}
/**
___[ Error Output ]___
May 09, 2017 6:07:17 AM LoggingException <init>
SEVERE: LoggingException
at
LoggingExceptions.main(LoggingExceptions.java:20)
Caught LoggingException
May 09, 2017 6:07:17 AM LoggingException <init>
SEVERE: LoggingException
at
LoggingExceptions.main(LoggingExceptions.java:25)
Caught LoggingException
*/
```

​		静态的Logger.getLogger()方法创建一个String参数相关联的Logger对象，这个Logger对象会将其输出发送到System.err。向Logger写入的最简单方式就是直接调用与日志记录消息的级别相关联的方法，这里使用的是severe()。为了产生日志记录消息，我们欲获取异常抛出处的栈轨迹，但是printStacktrace()不会默认地产生字符串。为了获取字符串，我们需要使用重载的printStackTrace()方法，它接受一个java.io.PrintWriter对象作为参数。如果我们将一个java.io.StringWrite对象传递给这个传递给这个PrintWrite的构造器，那么通过toString()方法，就可以将输出抽取为一个String。

​		尽管由于LoggingException将所有记录日志的基础设施都构建在异常自身中，使得它所使用的方式非常方便，并因此不需要客户端程序员的干预就可以自动运行，但是更常见的情形是我们需要捕获和记录其他人编写的异常，因此我们必须在异常处理程序中生成日志消息；

```java
// exceptions/LoggingExceptions2.java
// Logging caught exceptions
// {ErrorOutputExpected}
import java.util.logging.*;
import java.io.*; 
public class LoggingExceptions2 {
    private static Logger logger =
            Logger.getLogger("LoggingExceptions2");
    static void logException(Exception e) {
        StringWriter trace = new StringWriter();
        e.printStackTrace(new PrintWriter(trace));
        logger.severe(trace.toString());
    }
    public static void main(String[] args) {
        try {
            throw new NullPointerException();
        } catch(NullPointerException e) {
            logException(e);
        }
    }
}
/**
___[ Error Output ]___
May 09, 2017 6:07:17 AM LoggingExceptions2 logException
SEVERE: java.lang.NullPointerException
at
LoggingExceptions2.main(LoggingExceptions2.java:17)
*/
```

​		还可以更进一步自定义异常，比如加入额外的构造器和成员：

```java
// exceptions/ExtraFeatures.java
// Further embellishment of exception classes
class MyException2 extends Exception {
    private int x;
    MyException2() {}
    MyException2(String msg) { super(msg); }
    MyException2(String msg, int x) {
        super(msg);
        this.x = x;
    }
    public int val() { return x; }
    @Override
    public String getMessage() {
        return "Detail Message: "+ x
                + " "+ super.getMessage();
    }
}
public class ExtraFeatures {
    public static void f() throws MyException2 {
        System.out.println(
                "Throwing MyException2 from f()");
        throw new MyException2();
    }
    public static void g() throws MyException2 {
        System.out.println(
                "Throwing MyException2 from g()");
        throw new MyException2("Originated in g()");
    }
    public static void h() throws MyException2 {
        System.out.println(
                "Throwing MyException2 from h()");
        throw new MyException2("Originated in h()", 47);
    }
    public static void main(String[] args) {
        try {
            f();
        } catch(MyException2 e) {
            e.printStackTrace(System.out);
        }
        try {
            g();
        } catch(MyException2 e) {
            e.printStackTrace(System.out);
        }
        try {
            h();
        } catch(MyException2 e) {
            e.printStackTrace(System.out);
            System.out.println("e.val() = " + e.val());
        }
    }
}
/**
Throwing MyException2 from f()
MyException2: Detail Message: 0 null
at ExtraFeatures.f(ExtraFeatures.java:24)
at ExtraFeatures.main(ExtraFeatures.java:38)
Throwing MyException2 from g()
MyException2: Detail Message: 0 Originated in g()
at ExtraFeatures.g(ExtraFeatures.java:29)
at ExtraFeatures.main(ExtraFeatures.java:43)
Throwing MyException2 from h()
MyException2: Detail Message: 47 Originated in h()
at ExtraFeatures.h(ExtraFeatures.java:34)
at ExtraFeatures.main(ExtraFeatures.java:48)
e.val() = 47
*/
```

​		新的异常添加了字段x以及设定x值的构造器和读取数据的方法。此外，还覆盖了Throwable.getMessage()方法，以产生更详细的信息。对于异常类来说，getMessage()方法有点类似于toString()方法。

​		既然异常也是对象的一种，所以可以继续修改这个异常类，以得到更强大的功能。但要记住，使用程序包的客户端程序员可能仅仅只是查看一下抛出的异常类型，其他的就不管了，所以对异常所添加的其他功能也许根本用不上。

## 异常声明

​		Java鼓励人们把方法可能抛出的异常告知使用此方法的客户端程序员。这是种优雅的做法，它使得调用者能确切知道写什么样的代码可以捕获所有潜在的异常。当然，如果提供了源代码，客户端程序员可以在源代码中查找throw语句来获知相关信息，然而程序库通常并不与源代码一起发布。为了预防这样的问题，java提供了相应的语法(并强制使用这个语法)，使你能以礼貌的方式告知客户端程序员某个方法可能会抛出的异常类型，然后客户端程序员就可以进行相应的处理。这就是异常说明，它属于方法声明的一部分，紧跟在形式参数列表之后。

​		异常说明使用了附加的关键字throws，后面接一个所有潜在的异常类型的列表，所以方法定义可能看起来像这样：

```java
void f() throws TooBig, TooSmall, DivZero { // ...
```

​		但是，要是这样写：

```java
void f() { // ...
```

​		就表示此方法不会抛出任何异常(除了从RuntimeException继承的异常，它们可以在没有异常说明的情况下被抛出)。

​		代码必须与异常说明保持一致。如果方法里的代码产生了异常却没有进行处理，编译器会发现这个问题并提醒你：要么处理这个异常，要么就在异常说明中表名此方法将产生异常。通过这种自顶向下强制执行的异常说明机制，Java在编译时就可以保证一定水平的异常正确性。

​		不过还是有个能“作弊”的地方：可以在声明方法将抛出异常，实际上却不抛出。编译器相信了这个声明，并强制此方法的用户像真的抛出异常那样使用这个方法。这样做的好处是，为异常先占个位子，以后就可以抛出这种异常而不用修改已有代码。在定义抽象基类和接口时这种能力很重要，这样派生类或接口实现就能够抛出这些预先声明的异常。

​		这种在编译时被强制检查的异常称为被检查的异常。

## 捕获所有异常

​		可以只写一个异常处理程序来捕获所有类型的异常。通过捕获异常类型的基类Exception，就可以做到这一点(事实上还有其他的基类，但Exception是所有编程行为相关的基类)：

```java
catch(Exception e) {
    System.out.println("Caught an exception");
}
```

​		这将捕获所有异常，所以最好把它放在处理程序列表的末尾，以防它抢在其他处理程序之前先把异常捕获了。

​		因为Exception是与编程有关的所有异常类的基类，所以它不会含有太多具体的信息，不过可以调用它从其基类Throwable继承的方法：

```java
String getMessage()
String getLocalizedMessage()
```

​		用来获取详细信息，或用本地语言表示的详细信息。

```java
String toString()
```

​		返回对Throwable的简单描述，要是有详细信息的话，也会把它包含在内。

```java
void printStackTrace()
void printStackTrace(PrintStream)
void printStackTrace(java.io.PrintWriter)
```

​		打印Throwable和Throwable的调用栈轨迹。调用栈显示了“把你带到异常抛出地点”的方法调用序列。其中第一个版本输出到标准错误，后两个版本允许选择要输出的流。

```java
Throwable fillInStackTrace()
```

​		用于在Throwable对象的内部记录栈帧的当前状态。这在程序重新抛出错误或异常时很有用。

​		此外，也可以使用Throwable从其基类Object继承的方法。对于异常来说，getClass()也许是个很好用的方法，它将返回一个表示此对象类型的对象。然后可以使用getName()方法查询到这个Class对象包含包信息的名称，或者使用只产生类名称的getSimpleName()方法。

​		下面的例子演示了如何使用Exception类型的方法：

```java
// exceptions/ExceptionMethods.java
// Demonstrating the Exception Methods
public class ExceptionMethods {
    public static void main(String[] args) {
        try {
            throw new Exception("My Exception");
        } catch(Exception e) {
            System.out.println("Caught Exception");
            System.out.println(
                    "getMessage():" + e.getMessage());
            System.out.println("getLocalizedMessage():" +
                    e.getLocalizedMessage());
            System.out.println("toString():" + e);
            System.out.println("printStackTrace():");
            e.printStackTrace(System.out);
        }
    }
}
/**
Caught Exception
getMessage():My Exception
getLocalizedMessage():My Exception
toString():java.lang.Exception: My Exception
printStackTrace():
java.lang.Exception: My Exception
at ExceptionMethods.main(ExceptionMethods.java:7)
*/
```

​		可以发现每个方法都比前一个提供了更多的信息——实际上它们每一个都是前一个的超集。

### 多重捕获

​		如果有一组具有相同基类的异常，你想使用同一方式进行捕获，那你直接catch它们的基类性。但是，如果这些异常没有共同的基类性，在Java7之前，你必须为每一个类型编写一个catch：

```java
// exceptions/SameHandler.java
class EBase1 extends Exception {}
class Except1 extends EBase1 {}
class EBase2 extends Exception {}
class Except2 extends EBase2 {}
class EBase3 extends Exception {}
class Except3 extends EBase3 {}
class EBase4 extends Exception {}
class Except4 extends EBase4 {}

public class SameHandler {
    void x() throws Except1, Except2, Except3, Except4 {}
    void process() {}
    void f() {
        try {
            x();
        } catch(Except1 e) {
            process();
        } catch(Except2 e) {
            process();
        } catch(Except3 e) {
            process();
        } catch(Except4 e) {
            process();
        }
    }
}
```

​		通过Java7的多重捕获机制，你可以使用“或”将不同类型的异常组合起来，只需要一行catch语句：

```java
// exceptions/MultiCatch.java
public class MultiCatch {
    void x() throws Except1, Except2, Except3, Except4 {}
    void process() {}
    void f() {
        try {
            x();
        } catch(Except1 | Except2 | Except3 | Except4 e) {
            process();
        }
    }
}
```

​			或者以其他的组合方式：

```java
// exceptions/MultiCatch2.java
public class MultiCatch2 {
    void x() throws Except1, Except2, Except3, Except4 {}
    void process1() {}
    void process2() {}
    void f() {
        try {
            x();
        } catch(Except1 | Except2 e) {
            process1();
        } catch(Except3 | Except4 e) {
            process2();
        }
    }
}
```

​		这对书写整洁的代码很有帮助。

### 栈轨迹

​		printStackTrace()方法所提供的信息可以通过getStackTrace()方法直接来访问，这个方法将返回一个由栈轨迹中的元素所构成的数组，其中每一个元素都表示栈中的一帧。元素0是栈顶元素，并且是调用序列中的最后一个方法调用。数组中的最后一个元素和栈底是调用序列中的第一个方法调用。下面的程序是一个简单的演示示例：

```java
// exceptions/WhoCalled.java
// Programmatic access to stack trace information
public class WhoCalled {
    static void f() {
// Generate an exception to fill in the stack trace
        try {
            throw new Exception();
        } catch(Exception e) {
            for(StackTraceElement ste : e.getStackTrace())
                System.out.println(ste.getMethodName());
        }
    }
    static void g() { f(); }
    static void h() { g(); }
    public static void main(String[] args) {
        f();
        System.out.println("*******");
        g();
        System.out.println("*******");
        h();
    }
}
/**
f
main
*******
f
g
main
*******
f
g
h
main
*/
```

​		这里，我们只打印了方法名，但实际上还可以打印整个StackTranceElement，它包含其他附加的信息。

### 重新抛出异常

​		有时希望把刚捕获的异常重新抛出，尤其是在使用Exception捕获所有异常的时候。既然已经得到了对当前异常对象的引用，可以直接把它重新抛出：

```java
catch(Exception e) {
    System.out.println("An exception was thrown");
    throw e;
}
```

​		重抛异常会把异常抛给上一级环境中的异常处理程序，同一个try块的后续catch子句将被忽略。此外，异常对象的所有信息都得以保持，所以高一级环境中捕获此异常的处理程序可以从这个异常对象中得到所有信息。

​		如果只是把当前异常对象重新抛出，那么printStackTrace()方法显示的将是原来异常抛出点的调用栈信息，而并非重新抛出点的信息。想要跟新这个信息，可以调用fillInStackTrace()方法，这将返回一个Throwable对象，它是通过把当前调用栈信息填入原来那个异常对象而建立的，就像这样：

```java
// exceptions/Rethrowing.java
// Demonstrating fillInStackTrace()
public class Rethrowing {
    public static void f() throws Exception {
        System.out.println(
                "originating the exception in f()");
        throw new Exception("thrown from f()");
    }
    public static void g() throws Exception {
        try {
            f();
        } catch(Exception e) {
            System.out.println(
                    "Inside g(), e.printStackTrace()");
            e.printStackTrace(System.out);
            throw e;
        }
    }
    public static void h() throws Exception {
        try {
            f();
        } catch(Exception e) {
            System.out.println(
                    "Inside h(), e.printStackTrace()");
            e.printStackTrace(System.out);
            throw (Exception)e.fillInStackTrace();
        }
    }
    public static void main(String[] args) {
        try {
            g();
        } catch(Exception e) {
            System.out.println("main: printStackTrace()");
            e.printStackTrace(System.out);
        }
        try {
            h();
        } catch(Exception e) {
            System.out.println("main: printStackTrace()");
            e.printStackTrace(System.out);
        }
    }
}
/**
originating the exception in f()
Inside g(), e.printStackTrace()
java.lang.Exception: thrown from f()
at Rethrowing.f(Rethrowing.java:8)
at Rethrowing.g(Rethrowing.java:12)
at Rethrowing.main(Rethrowing.java:32)
main: printStackTrace()
java.lang.Exception: thrown from f()
at Rethrowing.f(Rethrowing.java:8)
at Rethrowing.g(Rethrowing.java:12)
at Rethrowing.main(Rethrowing.java:32)

originating the exception in f()
Inside h(), e.printStackTrace()
java.lang.Exception: thrown from f()
at Rethrowing.f(Rethrowing.java:8)
at Rethrowing.h(Rethrowing.java:22)
at Rethrowing.main(Rethrowing.java:38)
main: printStackTrace()
java.lang.Exception: thrown from f()
at Rethrowing.h(Rethrowing.java:27)
at Rethrowing.main(Rethrowing.java:38)
*/
```

​		调用fillInStackTrace()那一行就成了异常的新发生地了。

​		有可能在捕获异常之后抛出另一种异常。这么做的话，得到的效果类似于使用fillInStackTrace()，有关原来异常发生点的信息会丢失，剩下的是与新的抛出点有关的信息：

```java
// exceptions/RethrowNew.java
// Rethrow a different object from the one you caught
class OneException extends Exception {
    OneException(String s) { super(s); }
}
class TwoException extends Exception {
    TwoException(String s) { super(s); }
}
public class RethrowNew {
    public static void f() throws OneException {
        System.out.println(
                "originating the exception in f()");
        throw new OneException("thrown from f()");
    }
    public static void main(String[] args) {
        try {
            try {
                f();
            } catch(OneException e) {
                System.out.println(
                        "Caught in inner try, e.printStackTrace()");
                e.printStackTrace(System.out);
                throw new TwoException("from inner try");
            }
        } catch(TwoException e) {
            System.out.println(
                    "Caught in outer try, e.printStackTrace()");
            e.printStackTrace(System.out);
        }
    }
}
/**
originating the exception in f()
Caught in inner try, e.printStackTrace()
OneException: thrown from f()
at RethrowNew.f(RethrowNew.java:16)
at RethrowNew.main(RethrowNew.java:21)
Caught in outer try, e.printStackTrace()
TwoException: from inner try
at RethrowNew.main(RethrowNew.java:26)
*/
```

​		最后那个异常仅知道自己来自main()，而对f()一无所知。

​		永远不必为清理前一个异常对象而担心，或者说为异常对象的清理而担心。它们都是用new在堆上创建的对象，所以垃圾回收器会自动把它们清理掉。

​		
