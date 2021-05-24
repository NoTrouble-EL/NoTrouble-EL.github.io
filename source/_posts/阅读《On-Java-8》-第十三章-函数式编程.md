---
title: 阅读《On Java 8》-- 第十三章 函数式编程
date: 2021-05-23 11:42:53
tags:
- JAVASE
- 《On Java 8》
categories: JAVASE
mathjax: true
---

在计算机时代早期，内存是稀缺和昂贵。几乎每个人都用汇编语言。人们虽然知道编译器，但编译器生成的代码很低效，比手工编码的汇编程序多很多字节，仅仅想到这一点，人们还是选择汇编语言。

​		通常，为了使程序能在有限的内存上运行，在程序运行时，程序员通过修改内存中的代码，使程序可以执行不同的操作，用这种方式来节省代码空间。这种技术被称为**自修改代码**(self-modifying code)只要程序小到几个人就能维护所有棘手和难懂的汇编代码，你就能让程序运行起来。

​		随着内存和处理器变得更便宜、更快。C语言出现并被大多数汇编语言认为更“高级”。人们发现使用c可以显著提高生产力。同时，使用C创建自修改代码仍然不难。

​		随着硬件越来越便宜，程序的规模和复杂性都在增长。这一切只是让程序工作变得困难。我们想方设法使代码更加一致和易懂。使用纯粹的自修改代码造成的结果就是：我们很难确定程序在做什么。它也难以测试：除非你想一点点测试输出，代码转换和修改等等过程？

​		然而，使用代码以某种方式操作其他代码的想法也很有趣，只要能保证它更安全。从代码创建，维护和可靠性的角度来看，这个想法非常吸引人。我们不用从头开始编写大量代码，而是从易于理解、充分测试及可靠的现有小块开始，最后将它们组合在一起以创建新代码。难道这不会让我们更有效率，同时创造更健壮的代码吗？

​		这就是**函数式编程**(FP)的意义所在。通过合并现有代码来生成新功能而不是从头开始编写所有内容，我们可以更快地获得更可靠的代码。至少在某些情况下，这套理论似乎很有用。在这一过程中，函数式语言已经产生了优雅的语法，这些语法对于非函数式语言也适用。

​		你也可以这样想：OO(object oriented，面向对象)是抽象数据，FP(functional programming，函数式编程)是抽象行为。

​		纯粹的函数式语言在安全性方面更进一步。它强加了额外的约束，即所有数据必须是不可变的：设置一次，永不改变。将值传递给函数，该函数然后生成新值但从不修改自身外部的任何东西(包括其参数或该函数范围之外的元素)。当强制执行此操作时，你知道任何错误都不是由所谓的副作用引起的，因为该函数仅创建并返回结果，而不是其他任何错误。

​		更好的是，“不可变对象和无副作用”范式解决了并发编程中最基本和最棘手的问题之一(当程序的某些部分同时在多个处理器上运行时)。这是可变共享状态的问题，这意味着代码的不同部分(在不同的处理器上运行)可以尝试同时修改同一块内存(谁赢了？没人知道)。如果函数永远不会修改现有值但只生成新值，则不会对内存产生竞争，这是纯函数式语言的定义。因此，经常提出纯函数式语言作为并发编程的解决方案(还有其他可行的解决方案)。

​		需要提醒大家的是，函数式语言背后有很多动机，这意味着描述它们可能会有些混淆。它通常取决于各种观点：为“并发编程”，“代码可靠性”和“代码创建和库复用”。关于函数式编程能高效创建更健壮的代码这一观点仍存在部分争议。虽然已有一些好的范例，但不足以证明纯函数式语言就是解决编程问题的最佳方法。

​		FP思想值得融入非FP语言，如Python。Java8也从中吸收并支持了FP。

 <!-- more --> 

## 新旧对比

​		通常，传递给方法的数据不同，结果不同。如果我们希望方法在调用时行为不同，该怎么做呢？结论是：只要将代码传递给方法，我们就可以控制它的行为。为此，我们通过在方法中创建包含所需行为的对象，然后该对象传递给我们想要控制的方法来完成此操作。下面我们用传统形式和Java8的方法引用、Lambda表达式分别演示：

```java
// functional/Strategize.java

interface Strategy {
  String approach(String msg);
}

class Soft implements Strategy {
  public String approach(String msg) {
    return msg.toLowerCase() + "?";
  }
}

class Unrelated {
  static String twice(String msg) {
    return msg + " " + msg;
  }
}

public class Strategize {
  Strategy strategy;
  String msg;
  Strategize(String msg) {
    strategy = new Soft(); // [1]
    this.msg = msg;
  }

  void communicate() {
    System.out.println(strategy.approach(msg));
  }

  void changeStrategy(Strategy strategy) {
    this.strategy = strategy;
  }

  public static void main(String[] args) {
    Strategy[] strategies = {
      new Strategy() { // [2]
        public String approach(String msg) {
          return msg.toUpperCase() + "!";
        }
      },
      msg -> msg.substring(0, 5), // [3]
      Unrelated::twice // [4]
    };
    Strategize s = new Strategize("Hello there");
    s.communicate();
    for(Strategy newStrategy : strategies) {
      s.changeStrategy(newStrategy); // [5]
      s.communicate(); // [6]
    }
  }
}
//hello there?
//HELLO THERE!
//Hello
//Hello there Hello there
```

​		Strategy接口提供了单一的approach()方法来承载函数式功能。通过创建不同的Strategy对象，我们可以创建不同的行为。

​		我们一般通过创建一个实现Strategy接口的类来实现这种行为，正如在Soft里所做的。

[1]在Strategize中，你可以看到Soft作为默认策略，在构造器中赋值。

[2]一种较为简洁且更加自然的方式是创建一个匿名内部类。即便如此，仍有相当数量的冗余代码。你总需要仔细观察后才会发现：“哦， 我明白了，原来这里使用了匿名内部类。”

[3]Java8的Lambda表达式，其参数和函数体被箭头->分隔开。箭头右侧是从Lambda返回的表达式。它与单独定义类和采用匿名内部类是等价的，但代码少得多。

[4]Java8的方法引用，它以::为特征。::的左边是类或对象的名称，但是没有参数列表。

[5]在使用默认的Soft策略之后，我们逐步遍历数组中的所有Strategy，并通过调用changeStrategy()方法将每个Strategy传入变量s中。

[6]现在，每次调用communicate()都会产生不同的行为，具体取决于此刻正在使用的策略代码对象。我们传递的是行为，而并不仅仅是数据。

​		在Java8之前，我们能够通过[1]和[2]的方式传递功能。然而，这种语法的读写非常笨拙，并且我们别无选择。方法引用和Lambda表达式的出现让我们可以在需要时传递功能，而不是仅在必要时才这么做。

## Lambda表达式

​		Lambda表达式是使用最小可能语法编写的函数定义：

* 1.Lambda表达式产生函数，而不是类。虽然在JVM(Java Virtual Machine，Java虚拟机)上，一切都是类，但是幕后有各种操作执行让Lambda看起来像函数——作为程序员，你可以高兴地假装它们“就是函数”。
* 2.Lambda语法尽可能少，这正是为了使Lambda易于编写和使用。



​		我们在Strategize.java中看到了一个Lambda表达式，但还有其他语法变体：

```java
// functional/LambdaExpressions.java

interface Description {
  String brief();
}

interface Body {
  String detailed(String head);
}

interface Multi {
  String twoArg(String head, Double d);
}

public class LambdaExpressions {

  static Body bod = h -> h + " No Parens!"; // [1]

  static Body bod2 = (h) -> h + " More details"; // [2]

  static Description desc = () -> "Short info"; // [3]

  static Multi mult = (h, n) -> h + n; // [4]

  static Description moreLines = () -> { // [5]
    System.out.println("moreLines()");
    return "from moreLines()";
  };

  public static void main(String[] args) {
    System.out.println(bod.detailed("Oh!"));
    System.out.println(bod2.detailed("Hi!"));
    System.out.println(desc.brief());
    System.out.println(mult.twoArg("Pi! ", 3.14159));
    System.out.println(moreLines.brief());
  }
}
//Oh! No Parens!
//Hi! More details
//Short info
//Pi! 3.14159
//moreLines()
//from moreLines()
```

​		我们从三个接口开始，每个接口都有一个单独的方法。但是，每个方法都有不同数量的参数，以便演示Lambda表达式语法。

​		任何Lambda表达式的基本语法是：

* 1.参数。
* 2.接着->，可视为“产出”。
* 3.->之后的内容都是方法体。

[1]当只有一个参数，可以不需要括号()。然而，这是一个特例。

[2]正常情况使用括号()包裹参数。为了保持一致性，也可以使用括号()包裹单个参数，虽然这种情况并不常见。

[3]如果没有参数，则必须使用括号()表示空参数列表。

[4]对于多个参数，将参数列表放在括号()中。

​		到目前为止，所有Lambda表达式方法体都是单行。该表达式的结果自动成为Lambda表达式的返回值，在此处使用return关键字是非法的。这是Lambda表达式简化相应语法的另一种方式。

[5]如果在Lambda表达式中确实需要多行，则必须将这些方法花括号中。在这种情况下，就需要return。

​		Lambda表达式通常比匿名内部类产生更易读的代码。

### 递归

​		递归函数是一个自我调用的函数。可以编写递归的Lambda表达式，但需要注意：递归方法必须是实例变量或静态变量，否则会出现编译时错误。我们将为每个案例创建一个示例：

​		这两个示例都需要一个接受int型参数并生成int的接口：

```java
// functional/IntCall.java

interface IntCall {
  int call(int arg);
}
```

​		整数n的阶乘将所有小于或等于n的正整数相乘。阶乘函数是一个常见的递归示例：

```java
// functional/RecursiveFactorial.java

public class RecursiveFactorial {
  static IntCall fact;
  public static void main(String[] args) {
    fact = n -> n == 0 ? 1 : n * fact.call(n - 1);
    for(int i = 0; i <= 10; i++)
      System.out.println(fact.call(i));
  }
}
//1
//1
//2
//6
//24
//120
//720
//5040
//40320
//362880
//3628800
```

​		这里，fact是一个静态变量。注意使用三元**if-else**。递归函数将一直调用自己，直到i==0。所有递归函数都有“停止条件”，否则将无线递归并产生异常。

​		我们可以将Fibonacci序列用递归的Lambda表达式来实现，这次使用实例变量：

```java
// functional/RecursiveFibonacci.java

public class RecursiveFibonacci {
  IntCall fib;

  RecursiveFibonacci() {
    fib = n -> n == 0 ? 0 :
               n == 1 ? 1 :
               fib.call(n - 1) + fib.call(n - 2);
  }
  
  int fibonacci(int n) { return fib.call(n); }

  public static void main(String[] args) {
    RecursiveFibonacci rf = new RecursiveFibonacci();
    for(int i = 0; i <= 10; i++)
      System.out.println(rf.fibonacci(i));
  }
}
//0
//1
//1
//2
//3
//5
//8
//13
//21
//34
//55
```

​		将Fibonacci序列中的最后两个元素求和来产生下一个元素。

## 方法引用

​		Java8方法引用没有历史包袱。方法引用组成：类名或对象名，后面跟::，然后跟方法名称。

```java
// functional/MethodReferences.java

import java.util.*;

interface Callable { // [1]
  void call(String s);
}

class Describe {
  void show(String msg) { // [2]
    System.out.println(msg);
  }
}

public class MethodReferences {
  static void hello(String name) { // [3]
    System.out.println("Hello, " + name);
  }
  static class Description {
    String about;
    Description(String desc) { about = desc; }
    void help(String msg) { // [4]
      System.out.println(about + " " + msg);
    }
  }
  static class Helper {
    static void assist(String msg) { // [5]
      System.out.println(msg);
    }
  }
  public static void main(String[] args) {
    Describe d = new Describe();
    Callable c = d::show; // [6]
    c.call("call()"); // [7]

    c = MethodReferences::hello; // [8]
    c.call("Bob");

    c = new Description("valuable")::help; // [9]
    c.call("information");

    c = Helper::assist; // [10]
    c.call("Help!");
  }
}
//call()
//Hello, Bob
//valuable information
//Help!
```

[1]我们从单一方法接口开始。

[2]show()的签名(参数类型和返回类型)符合Callable的call()的签名。

[3]hello()也符合call()的签名。

[4]help()也符合，它是静态内部类中的静态方法。

[5]assist()是静态内部类中的静态方法。

[6]我们将Describe对象的方法引用赋值给Callable，它没有show()方法，而是call()方法。但是，Java似乎接受用这个看似奇怪的赋值，因为方法引用符合Callable的call()方法的签名。

[7]我们现在可以通过调用call()来调用show，因为Java将call()映射到show()。

[8]这是一个静态方法引用。

[9]这是[6]的另一个版本：对已实例化对象的方法的引用，有时称为**绑定方法引用**。

[10]最后，获取静态内部类中静态方法的引用与[8]中通过外部类引用相似。

​		上例只是简短的介绍，我们很快就能看到方法引用的所有不同形式。

