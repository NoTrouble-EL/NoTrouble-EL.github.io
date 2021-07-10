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

### 精确的重新抛出异常

​		在Java7之前，如果捕捉到一个异常，重新抛出异常只能与原异常完全相同。这导致代码不精确，Java7修复了这个问题。所以在Java7之前，这无法编译：

```java
class BaseException extends Exception {}
class DerivedException extends BaseException {}

public class PreciseRethrow {
    void catcher() throws DerivedException {
        try {
            throw new DerivedException();
        } catch(BaseException e) {
            throw e;
        }
    }
}
```

​		因为catch捕获了一个BaseException，编译器强迫你声明catcher()抛出BaseException，即使它实际上抛出了更具体的DerivedException。从Java7开始，这段代码就可以编译，这是一个很小但很有用的修复。

### 异常链

​		常常会想要在捕获一个异常后抛出另一个异常，并且希望把原始异常的信息保存下来，这被称为异常链。在JDK1.4以前，程序员必须自己编写代码来保存原始异常信息。现在所有Throwable的子类在构造器中都可以接受一个cause对象作为参数。这个cause就用来表示原始异常，这样通过把原始异常传递给新的异常，使得即使在当前位置创建并抛出了新的异常，也能通过这个异常链追踪到异常最初发生的位置。

​		有趣的是，在Throwable的子类中，只有三种基本的异常类提供了带cause参数的构造器。它们是Error、Exception以及RuntimeException。如果要把其他类型的异常链接起来，应该使用initCause()方法而不是构造器。

​		下面的例子能让你在运行时动态地向DynamicFields对象添加字段：

```java
// exceptions/DynamicFields.java
// A Class that dynamically adds fields to itself to
// demonstrate exception chaining
class DynamicFieldsException extends Exception {}
public class DynamicFields {
    private Object[][] fields;
    public DynamicFields(int initialSize) {
        fields = new Object[initialSize][2];
        for(int i = 0; i < initialSize; i++)
            fields[i] = new Object[] { null, null };
    }
    @Override
    public String toString() {
        StringBuilder result = new StringBuilder();
        for(Object[] obj : fields) {
            result.append(obj[0]);
            result.append(": ");
            result.append(obj[1]);
            result.append("\n");
        }
        return result.toString();
    }
    private int hasField(String id) {
        for(int i = 0; i < fields.length; i++)
            if(id.equals(fields[i][0]))
                return i;
        return -1;
    }
    private int getFieldNumber(String id)
            throws NoSuchFieldException {
        int fieldNum = hasField(id);
        if(fieldNum == -1)
            throw new NoSuchFieldException();
        return fieldNum;
    }
    private int makeField(String id) {
        for(int i = 0; i < fields.length; i++)
            if(fields[i][0] == null) {
                fields[i][0] = id;
                return i;
            }
// No empty fields. Add one:
        Object[][] tmp = new Object[fields.length + 1][2];
        for(int i = 0; i < fields.length; i++)
            tmp[i] = fields[i];
        for(int i = fields.length; i < tmp.length; i++)
            tmp[i] = new Object[] { null, null };
        fields = tmp;
// Recursive call with expanded fields:
        return makeField(id);
    }
    public Object
    getField(String id) throws NoSuchFieldException {
        return fields[getFieldNumber(id)][1];
    }
    public Object setField(String id, Object value)
            throws DynamicFieldsException {
        if(value == null) {
// Most exceptions don't have a "cause"
// constructor. In these cases you must use
// initCause(), available in all
// Throwable subclasses.
            DynamicFieldsException dfe =
                    new DynamicFieldsException();
            dfe.initCause(new NullPointerException());
            throw dfe;
        }
        int fieldNumber = hasField(id);
        if(fieldNumber == -1)
            fieldNumber = makeField(id);
        Object result = null;
        try {
            result = getField(id); // Get old value
        } catch(NoSuchFieldException e) {
// Use constructor that takes "cause":
            throw new RuntimeException(e);
        }
        fields[fieldNumber][1] = value;
        return result;
    }
    public static void main(String[] args) {
        DynamicFields df = new DynamicFields(3);
        System.out.println(df);
        try {
            df.setField("d", "A value for d");
            df.setField("number", 47);
            df.setField("number2", 48);
            System.out.println(df);
            df.setField("d", "A new value for d");
            df.setField("number3", 11);
            System.out.println("df: " + df);
            System.out.println("df.getField(\"d\") : "
                    + df.getField("d"));
            Object field =
                    df.setField("d", null); // Exception
        } catch(NoSuchFieldException |
                DynamicFieldsException e) {
            e.printStackTrace(System.out);
        }
    }
}
/**
null: null
null: null
null: null
d: A value for d
number: 47
number2: 48
df: d: A new value for d
number: 47
number2: 48
number3: 11

df.getField("d") : A new value for d
DynamicFieldsException
at DynamicFields.setField(DynamicFields.java:65)
at DynamicFields.
Caused by: java.lang.NullPointerException
at DynamicFields.setField(DynamicFields.java:67)
... 1 more
*/
```

​		每个DynamicFields对象都含有一个数组，其元素是“成对的对象”。第一个对象表示字段标识符(一个字符串)，第二个表示字段值，值的类型可以是基本类型外的任意类型。当创建对象的时候，要合理估计一下需要多少字段。当调用setField()方法的时候，它将试图通过标识符修改已有字段，否则就建一个新的字段，并把值放入。如果空间不够了，将建立一个更长的数组，并把原来数组的元素复制进去。如果你试图为字段设置一个空值，将抛出一个DynamicFieldsException异常，它是通过使用initCause()方法把NullPointerException对象插入而建立的。

​		至于返回值，setField()将用getField()方法把此位置的旧值取出，这个操作可能会抛出NoSuchFieldException异常。如果客户端程序员调用了getField()方法，那么他就有责任处理这个可能抛出异常的NoSuchFieldException异常，但如果异常是从setField()方法里抛出的，这种情况将被视为编程错误，所以就使用接受cause参数的构造器把NoSuchFieldException异常转换为RuntimeException异常。

​		你会注意到，toString()方法使用了一个StringBuilder来创建其结果。

​		main()方法中的catch子句看起来不同——它使用相同的子句处理两种不同类型的异常，这两种不同的异常通过“或(|)”符号结合起来。Java7的这项功能有助于减少代码重复，并使你更容易指定要捕获的确切类型，而不是简单地捕获一个基类性。你可以通过这种方式组合多种异常类型。

## Java标准异常

​		Throwable这个Java类被用来表示任何可以作为异常被抛出的类。Throwable对象可分别为两种类型(指从Throwable继承而得到的类型)：Error用来表示编译时和系统错误(除特殊情况外，一般你不用关心)；Exception是可以被抛出的基本类型，在Java类库、用户方法以及运行时故障中都可能抛出Exception型异常。所以Java程序员关心的基类型通常是Exception。但很快你就会发现，这些异常除了名称外其实都差不多。同时，Java中异常的数目在持续增加，所以在书中简单罗列它们毫无意义。所使用的第三方类库也可能会有自己的异常。对异常来说，关键是理解概念以及如何使用。

​		基本理念是用异常的名称代表发生的问题。异常的名称应该可以望文知意。异常并非全是在java.lang包中定义的；有些异常是用来支持其他像util、net和io这样的程序包，这些异常可以通过它们的完整名称或者从它们的父类中看出端倪。比如，所有的输入/输出异常都是从java.io.IOException继承而来的。

### 特例：RuntimeException

​		在本章中第一个例子中：

```java
if(t == null)
    throw new NullPointerException();
```

​		如果必须对传递给方法的每一个引用都检查其是否为null(因为无法确定调用者是否传入了非法引用)，这听起来着实吓人。幸运的是，这不必由你亲自来做，它属于Java的标准运行时检测的一部分。如果对null引用进行调用，Java会自动抛出NullPointerException异常，所以上述代码是多余的，尽管你也许想要执行其他的检查以确保NullPointerException不会出现。

​		属于运行时异常的类型有很多，它们被Java自动抛出，所以不必在异常说明中把它们列出来。非常方便的是，通过这些异常设置为RuntimeException的子类而把它们归类起来，这是继承的一个绝佳例子：建立具有相同特征和行为的一组类型。

​		RuntimeException代表的是编程错误：

* 1.无法预料的错误。比如从你控制范围之外传递进来的null引用。
* 作为程序员，应该在代码中进行错误检查。(比如对于ArrayIndexOutOfBoundsException，就得注意一下数组的大小了。)在一个地方发生的异常，常常会在另一个地方导致错误。



​		在这些情况下使用异常很有好处，它们能给调试带来便利。

​		如果不捕获这种类型的异常会发生什么事呢？因为编译器没有在这个问题上对异常说明进行强制检查，RuntimeException类型的异常也许会穿越所有的执行路径直达main()方法，而不会被捕获。要明白到底发生了什么，可以试试下面的例子：

```java
// exceptions/NeverCaught.java
// Ignoring RuntimeExceptions
// {ThrowsException}
public class NeverCaught {
    static void f() {
        throw new RuntimeException("From f()");
    }
    static void g() {
        f();
    }
    public static void main(String[] args) {
        g();
    }
}
/**
___[ Error Output ]___
Exception in thread "main" java.lang.RuntimeException:
From f()
at NeverCaught.f(NeverCaught.java:7)
at NeverCaught.g(NeverCaught.java:10)
at NeverCaught.main(NeverCaught.java:13)
*/
```

​		如果RuntimeException没有被捕获而直达main()，那么在程序退出前将调用异常的printStackTrace()方法。

​		你会发现，RuntimeException(或任何从它继承的异常)是一个特例。对于这种异常类型，编译器不需要异常说明，其输出被报告给了System.err。

​		请务必记住：代码中只有RuntimeException(及其子类)类型的异常可以被忽略，因为编译器强制要求处理所有受检查类型的异常。

​		值得注意的是：不应把Java异常处理机制当成是单一用途的工具。是的，它被设计用来处理一些烦人的运行时错误，这些错误往往是由代码控制能力之外的因素导致的；然而，它对于发现某些编译器无法检测到的编程错误，也是非常重要的。

## 使用finally进行清理

​		有一些代码片段，可能会希望无论try块中的异常是否抛出，它们都能得到执行。这通常适用于内存回收之外的情况(因为回收由垃圾回收器完成)，为了达到这个效果。可以在异常处理程序后面加上finally子句。完整的异常处理程序看起来像这样：

```java
try {
// The guarded region: Dangerous activities
// that might throw A, B, or C
} catch(A a1) {
// Handler for situation A
} catch(B b1) {
// Handler for situation B
} catch(C c1) {
// Handler for situation C
} finally {
// Activities that happen every time
}
```

​		为了证明finally子句总能运行，可以试试下面这个程序：

```java
// exceptions/FinallyWorks.java
// The finally clause is always executed
class ThreeException extends Exception {}
public class FinallyWorks {
    static int count = 0;
    public static void main(String[] args) {
        while(true) {
            try {
                // Post-increment is zero first time:
                if(count++ == 0)
                    throw new ThreeException();
                System.out.println("No exception");
            } catch(ThreeException e) {
                System.out.println("ThreeException");
            } finally {
                System.out.println("In finally clause");
                if(count == 2) break; // out of "while"
            }
        }
    }
}
/**
ThreeException
In finally clause
No exception
In finally clause
*/
```

​		从输出中发现，无论异常是否被抛出，finally子句总能被执行。这也为解决Java不允许我们回到异常抛出点这一问题，提供了一个思路。如果try块放在循环里，就可以设置一种在程序执行前一定会遇到的异常状况。还可以加入一个static类型的计数器或者别的装置，使循环在结束以前能尝试一定的次数。这将使程序的健壮性更上一个台阶。

### finally用来做什么？

​		对于没有垃圾回收和析构函数自动调用机制的语言来说，finally非常重要。它能使程序员保证：无论try块里发生了什么，内存总能得到释放。但Java有垃圾回收机制，所以内存释放不再是问题。而且，Java也没有析构函数可供调用。那么，Java在什么情况下才能用到finally呢？

​		当要把除内存之外的资源恢复到它们的初始状态时，就要用到finally子句。这种需要清理的资源包括：已经打开的文件或网络连接，在屏幕上画的图形，甚至是可以是外部世界的某个开关，如下面例子所示：

```java
// exceptions/Switch.java
public class Switch {
    private boolean state = false;
    public boolean read() { return state; }
    public void on() {
        state = true;
        System.out.println(this);
    }
    public void off() {
        state = false;
        System.out.println(this);
    }
    @Override
    public String toString() {
        return state ? "on" : "off";
    }
}
// exceptions/OnOffException1.java
public class OnOffException1 extends Exception {}
// exceptions/OnOffException2.java
public class OnOffException2 extends Exception {}
// exceptions/OnOffSwitch.java
// Why use finally?
public class OnOffSwitch {
    private static Switch sw = new Switch();
    public static void f()
            throws OnOffException1, OnOffException2 {}
    public static void main(String[] args) {
        try {
            sw.on();
            // Code that can throw exceptions...
            f();
            sw.off();
        } catch(OnOffException1 e) {
            System.out.println("OnOffException1");
            sw.off();
        } catch(OnOffException2 e) {
            System.out.println("OnOffException2");
            sw.off();
        }
    }
}
//on
//off
```

​		程序的目的是要确保main()结束的时候开关必须是关闭的，所以在每个try块和异常处理程序的末尾都加入了对sw.off()方法的调用。但也可能有这种情况：异常被抛出，但没被处理程序捕获，这时sw.off()就得不到调用。但是有了finally，只要把try块中的清理代码移放在一处即可：

```java
// exceptions/WithFinally.java
// Finally Guarantees cleanup
public class WithFinally {
    static Switch sw = new Switch();
    public static void main(String[] args) {
        try {
            sw.on();
            // Code that can throw exceptions...
            OnOffSwitch.f();
        } catch(OnOffException1 e) {
            System.out.println("OnOffException1");
        } catch(OnOffException2 e) {
            System.out.println("OnOffException2");
        } finally {
            sw.off();
        }
    }
}
/**
on
off
*/
```

​		这里sw.off()被移到一处，并且保证在任何情况下都能得到执行。

​		甚至在异常没有被当前的异常处理程序捕获的情况下，异常处理机制也会在跳到更高一层的异常处理程序之前，执行finally子句：

```java
// exceptions/AlwaysFinally.java
// Finally is always executed
class FourException extends Exception {}
public class AlwaysFinally {
    public static void main(String[] args) {
        System.out.println("Entering first try block");
        try {
            System.out.println("Entering second try block");
            try {
                throw new FourException();
            } finally {
                System.out.println("finally in 2nd try block");
            }
        } catch(FourException e) {
            System.out.println(
                    "Caught FourException in 1st try block");
        } finally {
            System.out.println("finally in 1st try block");
        }
    }
}
/**
Entering first try block
Entering second try block
finally in 2nd try block
Caught FourException in 1st try block
finally in 1st try block
*/
```

​		当涉及break和continue语句的时候，finally子句也会得到执行。请注意，如果把finally子句和带标签的break及continue配合使用，在Java里就没必要使用goto语句了。

### 在return中使用finally

​		因为finally子句总是会执行，所以可以从一个方法内的多个点返回，仍然能保证重要的清理工作会执行：

```java
// exceptions/MultipleReturns.java
public class MultipleReturns {
    public static void f(int i) {
        System.out.println(
                "Initialization that requires cleanup");
        try {
            System.out.println("Point 1");
            if(i == 1) return;
            System.out.println("Point 2");
            if(i == 2) return;
            System.out.println("Point 3");
            if(i == 3) return;
            System.out.println("End");
            return;
        } finally {
            System.out.println("Performing cleanup");
        }
    }
    public static void main(String[] args) {
        for(int i = 1; i <= 4; i++)
            f(i);
    }
}
/**
Initialization that requires cleanup
Point 1
Performing cleanup
Initialization that requires cleanup
Point 1
Point 2
Performing cleanup
Initialization that requires cleanup
Point 1
Point 2
Point 3
Performing cleanup
Initialization that requires cleanup
Point 1
Point 2
Point 3
End
Performing cleanup
*/
```

​		从输出中可以看出，从何处返回无关紧要，finally子句永远会执行。

### 缺憾：异常丢失

​		遗憾的是，Java的异常实现也有瑕疵。异常作为程序出错的标志，决不应该被忽略，但它还是有可能被轻易地忽略。用某些特殊的方式使用finally子句，就会发生这种情况：

```java
// exceptions/LostMessage.java
// How an exception can be lost
class VeryImportantException extends Exception {
    @Override
    public String toString() {
        return "A very important exception!";
    }
}
class HoHumException extends Exception {
    @Override
    public String toString() {
        return "A trivial exception";
    }
}
public class LostMessage {
    void f() throws VeryImportantException {
        throw new VeryImportantException();
    }
    void dispose() throws HoHumException {
        throw new HoHumException();
    }
    public static void main(String[] args) {
        try {
            LostMessage lm = new LostMessage();
            try {
                lm.f();
            } finally {
                lm.dispose();
            }
        } catch(VeryImportantException | HoHumException e) {
            System.out.println(e);
        }
    }
}
//A trivial exception
```

​		从输出中可以看到，VeryImportantException不见了，它被finally子句的HoHumException所取代。这是相当严重的缺陷，因为异常可能会以一种比前面例子所示更微妙和难以察觉的方式完全丢失。相比之下，C++把“前一个异常还没处理就抛出下一异常”的情形看成是糟糕的编程错误。也许在Java的未来版本中会修正这个问题(另一方面，要把所有抛出异常的方法，如上例中的dispose()方法，全部打包放到try-catch子句里面)。

​		一种更简单的丢失异常的方式是从finally子句中返回：

```java
// exceptions/ExceptionSilencer.java
public class ExceptionSilencer {
    public static void main(String[] args) {
        try {
            throw new RuntimeException();
        } finally {
            // Using 'return' inside the finally block
            // will silence any thrown exception.
            return;
        }
    }
}
```

​		如果运行这个程序，就会看到即使方法里抛出了异常，它也不会产生任何输出。

## 异常限制

​		当覆盖方法的时候，只能抛出在其基类方法的异常说明里列出的那些异常。这个限制很有用，因为这意味着与基类一起工作的代码，也能和导出类一起正常工作(这是面向对象的基本概念)，异常也不例外。

​		下面例子演示了这种(在编译时)施加在异常上面的限制：

```java
// exceptions/StormyInning.java
// Overridden methods can throw only the exceptions
// specified in their base-class versions, or exceptions
// derived from the base-class exceptions
class BaseballException extends Exception {}
class Foul extends BaseballException {}
class Strike extends BaseballException {}
abstract class Inning {
    Inning() throws BaseballException {}
    public void event() throws BaseballException {
// Doesn't actually have to throw anything
    }
    public abstract void atBat() throws Strike, Foul;
    public void walk() {} // Throws no checked exceptions
}
class StormException extends Exception {}
class RainedOut extends StormException {}
class PopFoul extends Foul {}
interface Storm {
    void event() throws RainedOut;
    void rainHard() throws RainedOut;
}
public class StormyInning extends Inning implements Storm {
    // OK to add new exceptions for constructors, but you
// must deal with the base constructor exceptions:
    public StormyInning()
            throws RainedOut, BaseballException {}
    public StormyInning(String s)
            throws BaseballException {}
    // Regular methods must conform to base class:
//- void walk() throws PopFoul {} //Compile error
// Interface CANNOT add exceptions to existing
// methods from the base class:
//- public void event() throws RainedOut {}
// If the method doesn't already exist in the
// base class, the exception is OK:
    @Override
    public void rainHard() throws RainedOut {}
    // You can choose to not throw any exceptions,
// even if the base version does:
    @Override
    public void event() {}
    // Overridden methods can throw inherited exceptions:
    @Override
    public void atBat() throws PopFoul {}
    public static void main(String[] args) {
        try {
            StormyInning si = new StormyInning();
            si.atBat();
        } catch(PopFoul e) {
            System.out.println("Pop foul");
        } catch(RainedOut e) {
            System.out.println("Rained out");
        } catch(BaseballException e) {
            System.out.println("Generic baseball exception");
        }
// Strike not thrown in derived version.
        try {
// What happens if you upcast?
            Inning i = new StormyInning();
            i.atBat();
// You must catch the exceptions from the
// base-class version of the method:
        } catch(Strike e) {
            System.out.println("Strike");
        } catch(Foul e) {
            System.out.println("Foul");
        } catch(RainedOut e) {
            System.out.println("Rained out");
        } catch(BaseballException e) {
            System.out.println("Generic baseball exception");
        }
    }
}

```

​		在Inning类中，可以看到构造器和event()方法都声明将异常抛出，但实际上没有抛出。这种方式使你能强制用户去捕获可能在覆盖后的event()版本中增加的异常，所以它很合理。这对于抽象方法同样成立，比如atBat()。

​		接口Storm包含了一个在Inning中定义的方法event()和一个不在Inning中定义的方法rainHard()。这两个方法都抛出新的异常RainedOut，如果StormyInning类在扩展Inning类的同时又实现了Storm接口，那么Strom里的event()方法就不能改变在Inning中的event()方法的异常接口。否则的话，在使用基类的时候就不能判断是否捕获了正确的异常，所以这也很合理。当然，如果接口里定义的方法不是来自于基类，比如rainHard()，那么此方法抛出什么样的异常都没有问题。

​		异常限制对构造器不起作用。你会发现StormyInning的构造器可以抛出任何异常，而不必理会基类构造器所抛出的异常。然而，因为基类构造器必须以这样或那样的方式被调用(这里默认构造器将自动被调用)，派生类构造器的异常说明必须包含基类构造器的异常说明。

​		派生类构造器不能捕获基类构造器抛出的异常。

​		StormyInning.walk()不能通过编译是因为它抛出了一个Inning.walk()中没有声明的异常。如果编译器允许这么做的话，就可以编写调用Inning.walk()却不处理任何异常的代码。但是，当使用Inning派生类的对象时，就会抛出异常，从而导致程序出问题。通过强制派生类遵守基类方法的异常说明，对象的可替换性得到了保证。

​		覆盖后的event()方法表明，派生类版的方法可以不抛出任何异常，即使基类版的方法抛出了异常。因为这样做不会破坏那些假定基类版的方法会抛出异常的代码。类似的情况出现在atBat()上，它抛出的异常PopFoul是由基类版atBat()抛出的Foul异常派生而来。如果你写的代码同Inning一起工作，并且调用了atBat()的话，那么肯定能捕获Foul。又因为PopFoul是由Foul派生而来，因此异常处理程序也捕获PopFoul。

​		最后一个有趣的地方在main()。如果处理的刚好是StormyInning对象的话，编译器只要求捕获这个类所抛出的异常。但如果将它向上转型成基类性，那么编译器就会准确地要求捕获基类的异常。所以这些限制都是为了能产生更为健壮的异常处理代码。

​		尽管在继承过程中，编译器会对异常说明做强制要求，但异常说明本身并不属于方法类型的一部分，方法类型是由方法的名字与参数的类型组成的。因此，不能基于异常说明来重载方法。此外，一个出现在基类方法的异常说明中的异常，不一定会出现在派生类的异常说明里。这点同继承的规则明显不同，在继承中，基类的方法必须出现在派生类里，换句话说，在继承和覆盖的过程中，某个特定方法的“异常说明的接口”不是变大了而是变小了——这恰好和类接口在继承时情形相反。

## 构造器

​		这一点很重要，即你要时刻询问自己“如果异常发生了，所有东西都能被正确的清理吗？”尽管大多数情况下是非常安全的，但涉及构造器时，问题就出现了。构造器会把对象设置成安全的初始状态，但还会有别的动作，比如打开一个文件，这样的动作只有在对象使用完毕并且用户调用了特殊的清理方法之后才能得以清理。如果在构造器内抛出了异常，这些清理行为也许就不能正常工作了。这意味着在编写构造器时要格外小心。

​		你也许会认为使用finally就可以解决问题。但问题并非如此简单，因为finally会每次都执行清理代码。如果构造器在其执行过程中半途而废，也许该对象的某些部分还没有被成功创建，而这部分在finally子句中却是要被清理的。

​		下面的例子中，建立了一个InputFile类在，它能打开一个文件并且每次读取其中一行。这里使用了Java标准输入/输出库中的FileReader和BufferedReader类，这些类的基本用法很简单，你应该很容易明白：

```java
// exceptions/InputFile.java
// Paying attention to exceptions in constructors
import java.io.*;
public class InputFile {
    private BufferedReader in;
    public InputFile(String fname) throws Exception {
        try {
            in = new BufferedReader(new FileReader(fname));
            // Other code that might throw exceptions
        } catch(FileNotFoundException e) {
            System.out.println("Could not open " + fname);
            // Wasn't open, so don't close it
            throw e;
        } catch(Exception e) {
            // All other exceptions must close it
            try {
                in.close();
            } catch(IOException e2) {
                System.out.println("in.close() unsuccessful");
            }
            throw e; // Rethrow
        } finally {
        // Don't close it here!!!
        }
    }
    public String getLine() {
        String s;
        try {
            s = in.readLine();
        } catch(IOException e) {
            throw new RuntimeException("readLine() failed");
        }
        return s;
    }
    public void dispose() {
        try {
            in.close();
            System.out.println("dispose() successful");
        } catch(IOException e2) {
            throw new RuntimeException("in.close() failed");
        }
    }
}
```

​		InputFile的构造器接受字符串作为参数，该字符串表示所要打开的文件名。在try块中，会使用此文件名建立FileReader对象。FileReader对象本身用处并不大，但可以用它来建立BufferedReader对象。注意，使用InputFile的好处之一是把两步操作合二为一。

​		如果FileReader的构造器失败了，将抛出FileNotFoundException异常。对于这个异常，并不需要关闭文件，因为这个文件还没有被打开。而任何其他捕获异常的catch子句必须关闭文件，因为在它们捕获到异常之时，文件已经打开了，close()方法也可能会抛出异常，所以尽管它已经在另一个catch子句块里了，还是要再用一层try-catch，这对Java编译器而言只不过是多了一对花括号。在本地做完处理之后，异常被重新抛出，对于构造器而言这么做是很合适的，因为你总不希望去误导调用方，让他认为“这个对象已经创建完毕，可以使用了”。

​		在本例中，由于finally会在每次完成构造器之后都执行一遍，因此它实在不该是调用close()关闭文件的地方。我们希望文件在InputFile对象的整个生命周期内都处于打开状态。

​		getLine()方法会返回表示文件下一行内容的字符串。它调用了能抛出异常的readLine()，但是这个异常已在方法内得到处理，因此getLine()不会抛出任何异常。在设计异常时有一个问题：应该把异常全部放在这一层处理；还是先处理一部分，然后向上层抛出相同的异常；又或者是不做任何处理直接向上层抛出。如果用法恰当的话，直接向上层抛出的确能简化编程。在这里，getLine()方法将异常转换为RuntimeException，表示一个编程错误。

​		用户在不需要InputFile对象时，就必须调用dispose()方法，这将释放BufferedReader或FileReader对象所占用的系统资源，在使用完InputFile对象之前不会调用它的。可能你会考虑把上述功能放到finalize()里面，在封装中讲过，你不知道finalize()会不会被调用，这也是Java的缺陷：除了内存的清理之外，所有的清理都不会自动发生。所以必须告诉客户端程序员，这是他们的责任。

​		对于构造阶段可能会抛出异常，并且要求清理的类，最安全的使用方式是使用嵌套的try子句：

```java
// exceptions/Cleanup.java
// Guaranteeing proper cleanup of a resource
public class Cleanup {
    public static void main(String[] args) {
        try {
            InputFile in = new InputFile("Cleanup.java");
            try {
                String s;
                int i = 1;
                while((s = in.getLine()) != null)
                    ; // Perform line-by-line processing here...
            } catch(Exception e) {
                System.out.println("Caught Exception in main");
                e.printStackTrace(System.out);
            } finally {
                in.dispose();
            }
        } catch(Exception e) {
            System.out.println(
                    "InputFile construction failed");
        }
    }
}
//dispose() successful
```

​		请仔细观察这里的逻辑：对InputFile对象的构造在其自己的try语句块中有效，如果构造失败，将进入外部的catch子句，而dispose()方法不会被调用。但是，如果构造成功，我们肯定想确保对象能够被清理，因此在构造之后立即创建一个新的try语句块。执行清理的finally与内部的try语句块相关联。在这种方式中，finally子句在构造失败时是不会执行的，而在构造成功时将总是执行。

​		这种通用的清理惯用法在构造器不抛出任何异常时也应该运用，其基本规则是：在创建需要清理的对象之后，立即进入一个try-finally语句块：

```java
// exceptions/CleanupIdiom.java
// Disposable objects must be followed by a try-finally
class NeedsCleanup { // Construction can't fail
    private static long counter = 1;
    private final long id = counter++;
    public void dispose() {
        System.out.println(
                "NeedsCleanup " + id + " disposed");
    }
}
class ConstructionException extends Exception {}
class NeedsCleanup2 extends NeedsCleanup {
    // Construction can fail:
    NeedsCleanup2() throws ConstructionException {}
}
public class CleanupIdiom {
    public static void main(String[] args) {
        // [1]:
        NeedsCleanup nc1 = new NeedsCleanup();
        try {
        // ...
        } finally {
            nc1.dispose();
        }
        // [2]:
        // If construction cannot fail,
        // you can group objects:
        NeedsCleanup nc2 = new NeedsCleanup();
        NeedsCleanup nc3 = new NeedsCleanup();
        try {
        // ...
        } finally {
            nc3.dispose(); // Reverse order of construction
            nc2.dispose();
        }
        // [3]:
        // If construction can fail you must guard each one:
        try {
            NeedsCleanup2 nc4 = new NeedsCleanup2();
            try {
                NeedsCleanup2 nc5 = new NeedsCleanup2();
                try {
                // ...
                } finally {
                    nc5.dispose();
                }
            } catch(ConstructionException e) { // nc5 const.
                System.out.println(e);
            } finally {
                nc4.dispose();
            }
        } catch(ConstructionException e) { // nc4 const.
            System.out.println(e);
        }
    }
}
/**
NeedsCleanup 1 disposed
NeedsCleanup 3 disposed
NeedsCleanup 2 disposed
NeedsCleanup 5 disposed
NeedsCleanup 4 disposed
*/
```

* [1]相当简单，遵循了在可去除对象之后紧跟try-finally的原则。如果对象构造不会失败，就不需要任何catch。
* [2]为了构造和清理，可以看到将具有不能失败的构造器的对象分组在一起。
* [3]展示了如何处理那些具有可以失败的构造器，且需要清理的对象。为了正确处理这种情况，事情变得很棘手，因为对于每一个构造器，都必须包含在其自己的try-finally语句块中，并且每一个对象构造必须都跟随一个try-finally语句块以确保清理。



​		本例中异常处理的混乱情形，有力的论证了应该创建不会抛出异常的构造器，尽管这并不总会实现。

​		注意，如果dispose()可以抛出异常，那么你可能需要额外的try语句块。基本上，你应该仔细考虑所有可能性，并确保正确处理每一种情况。

## Try-With-Resources 用法

​		上一节的内容可能让你有些头疼。在考虑所有可能失败的方法时，找出放置所有try-catch-finally块的位置变得令人生畏。确保没有任何故障路径，使系统远离不稳定状态，这非常具有挑战性。

​		InputFile.java是一个特别棘手的情况，因为文件被打开，然后它在对象的生命周期中保持打开状态。每次调用getLine()都可能导致异常，而且dispose()也是这种情况。这个例子只是好在它显示了事情可以混乱到什么地步。它还表名了你应该尽量不要那样设计代码。

​		InputFile.java一个更好的实现方式是如果构造函数读取文件并在内部缓冲它——这样，文件的打开，读取和关闭都发生在构造函数中。或者，如果读取和存储文件不切实际，你可以改为生成Stream。理想情况下，你可以设计如下的样子：

```java
// exceptions/InputFile2.java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;
public class InputFile2 {
    private String fname;

    public InputFile2(String fname) {
        this.fname = fname;
    }

    public Stream<String> getLines() throws IOException {
        return Files.lines(Paths.get(fname));
    }

    public static void
    main(String[] args) throws IOException {
        new InputFile2("InputFile2.java").getLines()
                .skip(15)
                .limit(1)
                .forEach(System.out::println);
    }
}
//main(String[] args) throws IOException {
```

​		现在，getLines()全权负责打开文件并创建Stream。

​		你不能总是轻易地回避这个问题。有时会有以下问题：

* 需要资源清理
* 需要在特定的时刻进行资源清理，比如你离开作用域的时候(在通常情况下意味着通过异常进行清理)。



​		一个常见的例子是java.io.FileInputStrea。要正确使用它，你必须编写一些棘手的样板代码：

```java
// exceptions/MessyExceptions.java
import java.io.*;
public class MessyExceptions {
    public static void main(String[] args) {
        InputStream in = null;
        try {
            in = new FileInputStream(
                    new File("MessyExceptions.java"));
            int contents = in.read();
            // Process contents
        } catch(IOException e) {
            // Handle the error
        } finally {
            if(in != null) {
                try {
                    in.close();
                } catch(IOException e) {
                    // Handle the close() error
                }
            }
        }
    }
}
```

​		当finally子句有自己的try块时，感觉事情变得过于复杂。

​		幸运的是，Java7引入try-with-resources语法，它可以非常清除地简化上面的代码：

```java
// exceptions/TryWithResources.java
import java.io.*;
public class TryWithResources {
    public static void main(String[] args) {
        try(
                InputStream in = new FileInputStream(
                        new File("TryWithResources.java"))
        ) {
            int contents = in.read();
            // Process contents
        } catch(IOException e) {
            // Handle the error
        }
    }
}
```

​		在Java7之前，try后面总是跟着一个\{，但是现在可以跟一个带括号的定义——这里是我们创建的FileInputStream对象。括号内的部分称为资源规范头(resource specification header)。现在in在整个try块的其余部分都是可用的。更重要的是，无论你如何退出try块(正常或通过异常)，和以前的finally子句等级的代码都会被执行，并且不用编写那些杂乱而棘手的代码。这是一项重要的改进。

​		它是如何工作的？try-with-resources定义子句中创建的对象必须实现java.lang.AutoCloseable接口，这个接口只有一个方法：close()。当在Java7中引入AutoCloseable时，许多接口和类被修改以实现它；

```java
// exceptions/StreamsAreAutoCloseable.java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;
public class StreamsAreAutoCloseable {
    public static void
    main(String[] args) throws IOException{
        try(
                Stream<String> in = Files.lines(
                        Paths.get("StreamsAreAutoCloseable.java"));
                PrintWriter outfile = new PrintWriter(
                        "Results.txt"); // [1]
        ) {
            in.skip(5)
                    .limit(1)
                    .map(String::toLowerCase)
                    .forEachOrdered(outfile::println);
        } // [2]
    }
}
```

* [1]你在这里可以看到其他的特性：资源规范头中可以包含多个定义，并且通过分号进行分割(最后一个分号是可选的)。规范头中定义的每个对象都会在try语句块运行结束之后调用close()方法。
* [2]try-with-resources里面的try语句块可以不包含catch或者finally语句独立存在。在这里，IOException被main()方法抛出，所以这里并不需要在try后面跟着一个catch语句块。



​		Java5中的Closeable已经被修改，修改之后的接口继承了AutoCloseable接口。所以所有实现了Closeable接口的对象，都支持try-with-resources特性。

### 揭示细节

​		为了研究try-with-resources的基本机制，我们将创建自己的AutoCloseable类：

```java
// exceptions/AutoCloseableDetails.java
class Reporter implements AutoCloseable {
    String name = getClass().getSimpleName();
    Reporter() {
        System.out.println("Creating " + name);
    }
    public void close() {
        System.out.println("Closing " + name);
    }
}
class First extends Reporter {}
class Second extends Reporter {}
public class AutoCloseableDetails {
    public static void main(String[] args) {
        try(
                First f = new First();
                Second s = new Second()
        ) {
        }
    }
}
/**
Creating First
Creating Second
Closing Second
Closing First
*/
```

​		退出try块会调用两个对象的close()方法，并以与创建顺序相反的顺序关闭它们。顺序很重要，因为在这种情况下，Second对象可能依赖于First对象，因此如果First在第Second关闭时已经关闭。Second的close()方法可能会尝试访问First中不再可用的某些功能。

​		假设我们在资源规范头中定义了一个不是AutoCloseable对象

```java
// exceptions/TryAnything.java
// {WillNotCompile}
class Anything {}
public class TryAnything {
    public static void main(String[] args) {
        try(
                Anything a = new Anything()
        ) {
        }
    }
}
```

​		正如我们所希望和期望的那样，Java不会让我们这样做，并且出现编译时错误。

​		如果其中一个构造函数抛出异常怎么办？

```java
// exceptions/ConstructorException.java
class CE extends Exception {}
class SecondExcept extends Reporter {
    SecondExcept() throws CE {
        super();
        throw new CE();
    }
}
public class ConstructorException {
    public static void main(String[] args) {
        try(
                First f = new First();
                SecondExcept s = new SecondExcept();
                Second s2 = new Second()
        ) {
            System.out.println("In body");
        } catch(CE e) {
            System.out.println("Caught: " + e);
        }
    }
}
/**
Creating First
Creating SecondExcept
Closing First
Caught: CE
*/
```

​		现在资源规范头中定义了3个对象，中间的对象抛出异常。因此，编译器强制我们使用catch子句来捕获构造函数。这意味着资源规范头实际上被try块包围。

​		正如预期的那样，First创建时没有发生意外，SecondExcept在创建期间抛出异常。请注意，不会为SecondExcept调用close()，因为如果构造函数失败，则无法假设你可以安全地对该对象执行任何操作，包括关闭它。由于SecondExcept的异常，Second对象实例s2不会被创建，因此也不会有清除事件发生。

​		如果没有构造函数抛出异常，但在try的主体中可能抛出异常，那么你将再次被强制要求提供一个catch子句：

```java
// exceptions/BodyException.java
class Third extends Reporter {}
public class BodyException {
    public static void main(String[] args) {
        try(
                First f = new First();
                Second s2 = new Second()
        ) {
            System.out.println("In body");
            Third t = new Third();
            new SecondExcept();
            System.out.println("End of body");
        } catch(CE e) {
            System.out.println("Caught: " + e);
        }
    }
}
/**
Creating First
Creating Second
In body
Creating Third
Creating SecondExcept
Closing Second
Closing First
Caught: CE
*/
```

​		请注意，第3个对象永远不会被清除。那是因为它不是在资源规范头中创建的，所以它没有被保护。这很重要，因为Java在这里没有以警告或错误的形式提供指导，因此像这样的错误很容易漏掉。实际上，如果依赖某些集成开发环境来自动重写代码，以使用try-with-resources特性，那么它们通常只会保护它们遇到的第一个对象，而忽略其余的对象。

​		最后，让我们看一下抛出异常的close()方法：

```java
// exceptions/CloseExceptions.java
class CloseException extends Exception {}
class Reporter2 implements AutoCloseable {
    String name = getClass().getSimpleName();
    Reporter2() {
        System.out.println("Creating " + name);
    }
    public void close() throws CloseException {
        System.out.println("Closing " + name);
    }
}
class Closer extends Reporter2 {
    @Override
    public void close() throws CloseException {
        super.close();
        throw new CloseException();
    }
}
public class CloseExceptions {
    public static void main(String[] args) {
        try(
                First f = new First();
                Closer c = new Closer();
                Second s = new Second()
        ) {
            System.out.println("In body");
        } catch(CloseException e) {
            System.out.println("Caught: " + e);
        }
    }
}
/**
Creating First
Creating Closer
Creating Second
In body
Closing Second
Closing Closer
Closing First
Caught: CloseException
*/
```

​		从技术上讲，我们并没有被迫在这里提供一个catch子句；你可以通过main()throws CloseException的方式来报告异常。但catch子句是放置错误处理代码的典型位置。

​		请注意，因为所有三个对象都已创建，所以它们都以相反的顺序关闭-即时Close.close()抛出异常也是如此。仔细想想，这就是你想要的结果。但如果你必须亲手编写所有的逻辑，或许会丢失一些东西并使得逻辑出错。想想那些程序员没有考虑Clean up的所有影响并且出错的代码。因此，如果可以，你应当始终使用try-with-resources。这个特性有助于生成更简洁，更易于理解的代码。

## 异常匹配

​		抛出异常的时候，异常处理系统会按照代码的书写顺序找出“最近”的处理程序。找到匹配的处理程序之后，它就认为异常将得到处理，然后就不再继续查找。

​		查找的时候并不要求抛出的异常同处理程序所声明的异常完全匹配。派生类的对象也可以匹配其基类的处理程序，就像这样：

```java
// exceptions/Human.java
// Catching exception hierarchies
class Annoyance extends Exception {}
class Sneeze extends Annoyance {}
public class Human {
    public static void main(String[] args) {
        // Catch the exact type:
        try {
            throw new Sneeze();
        } catch(Sneeze s) {
            System.out.println("Caught Sneeze");
        } catch(Annoyance a) {
            System.out.println("Caught Annoyance");
        }
        // Catch the base type:
        try {
            throw new Sneeze();
        } catch(Annoyance a) {
            System.out.println("Caught Annoyance");
        }
    }
}
//Caught Sneeze
//Caught Annoyance
```

​		Sneeze异常会被第一个匹配的catch子句捕获，也就是程序里的第一个。然而如果将这个catch子句删掉，只留下Annoyance的catch子句，该程序仍然能运行，因为这次捕获的是Sneeze的基类。换句话说，catch(Annoyance a)会捕获Annoyance以及所有从它派生的异常。这一点非常有用，因为如果决定在方法里加上更多派生异常的话，只要客户程序员捕获的是基类异常，那么它的代码就无需更改。

​		如果把捕获基类的catch子句放在最前面，以此想把派生类的异常全给“屏蔽”掉，就像这样：

```java
try {
    throw new Sneeze();
} catch(Annoyance a) {
    // ...
} catch(Sneeze s) {
    // ...
}
```

​		此时，编译器会发现Sneeze的catch子句永远得不到执行，因此它会向你报告错误。

## 其他可选方式

​		异常处理系统就像一个活门(trap door)，使你能放弃程序的正常执行序列。当“异常情形”发生的时候，正常的执行已变得不可能或者不需要了，这时就要用到这个“活门”。异常代表了当前方法不能继续执行的情形。开发异常处理系统的原因是，如果为每个方法所有可能发生的错误都进行处理的话，任务就显得过于繁重了，程序员也不愿意这么做。结果常常是将错误忽略。应该注意到，开发异常处理的初衷是为了方便程序员处理错误。

​		异常处理的一个重要原则是“只有在你知道如何处理的情况下才捕获异常”。实际上，异常处理的一个重要目标就是把处理错误的代码同错误发生的地点相分离。这使你能在一段代码中专注于要完成的事情，至于如何处理错误，则放在另一段代码中完成。这样一来，主要代码就不会与错误处理逻辑混在一起，也更容易理解和维护。通过允许一个处理程序去处理多个出错点，异常处理还使得错误处理代码的数量趋于减少。

​		“被检查的异常”使这个问题变得有些复杂，因为它们强制你在可能还没准备好处理错误的时候被迫加上catch子句，这就导致了吞食则有害(harmful if swallowed)的问题：

```java
try {
    // ... to do something useful
} catch(ObligatoryException e) {} // Gulp!
```

### 历史

### 观点

### 把异常传递给控制台

### 把“被检查异常”转换为“不检查异常”

## 异常指南

​		应该在下列情况下使用异常：

* 1.尽可能使用try-with-resources
* 2.在恰当的级别处理问题。
* 3.解决问题并且重新调用产生异常的方法。
* 4.进行少许修补，然后绕过异常发生的地方继续执行。
* 5.用别的数据进行计算，以代替方法预计会返回的值。
* 6.把当前运行环境下能做的事情尽量做完，然后把相同的异常重新抛到更高层。
* 7.把当前运行环境下能做的事情尽量做完，然后把不同的异常抛到更高层。
* 8.终止程序。
* 9.进行简化。
* 10.让类库和程序更安全。

