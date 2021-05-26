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

### Runnable接口

​		Runnable接口自1.0版以来一直在Java中，因此不需要导入。它符合特殊的单方法接口格式：它的方法run()不带参数，也没有返回值。因此，我们可以使用Lambda表达式和方法引用作为Runnable：

```java
// functional/RunnableMethodReference.java

// 方法引用与 Runnable 接口的结合使用

class Go {
  static void go() {
    System.out.println("Go::go()");
  }
}

public class RunnableMethodReference {
  public static void main(String[] args) {

    new Thread(new Runnable() {
      public void run() {
        System.out.println("Anonymous");
      }
    }).start();

    new Thread(
      () -> System.out.println("lambda")
    ).start();

    new Thread(Go::go).start();
  }
}
//Anonymous
//lambda
//Go::go()
```

​		Thread对象将Runnable作为其构造函数参数，并具有会调用run()的方法start()。注意这里只有匿名内部类才要求显示声明run()方法。

### 未绑定的方法引用

​		未绑定的方法引用是指没有关联对象的普通(非静态)方法。使用未绑定的引用，我们必须先提供对象：

```java
// functional/UnboundMethodReference.java

// 没有方法引用的对象

class X {
  String f() { return "X::f()"; }
}

interface MakeString {
  String make();
}

interface TransformX {
  String transform(X x);
}

public class UnboundMethodReference {
  public static void main(String[] args) {
    // MakeString ms = X::f; // [1]
    TransformX sp = X::f;
    X x = new X();
    System.out.println(sp.transform(x)); // [2]
    System.out.println(x.f()); // 同等效果
  }
}
//X::f()
//X::f()
```

​		到目前为止，我们已经见过了方法引用和对应接口的签名(参数类型和返回类型)一致的几个赋值例子。在[1]中，我们尝试同样的做法，把X的f()方法引用赋值给MakeString。结果即使make()与f()具有相同的签名，编译也会报“invalid method reference”(无效方法引用)错误。问题在于，这里其实还需要另一个隐藏参数参与：我们的老朋友this。你不能在没有X对象的前提下调用f()。因此，X::f表示未绑定的方法引用，因为它尚未“绑定”到对象。

​		要解决这个问题，我们需要一个X对象，因此我们接口实际上需要一个额外的参数，正如TransformX中看到的那样。如果将X::f赋值给TransformX，在Java中是允许的。我们必须做第二个心理调整——使用未绑定的引用时，函数式方法的签名(接口中的单个方法)不再与方法引用的签名完全匹配。原因是：你需要一个对象来调用方法。

​		[2]的结果有点像脑经急转弯。我拿到未绑定的方法引用，并且调用它的transform()方法，将一个x类的对象传递给它，最后使得x.f()以某种方式被调用。Java知道它必须拿第一个参数，该参数实际就是this对象，然后对此调用方法。

​		如果你的方法有更多个参数，就以第一个参数接受this的模式来处理。

```java
// functional/MultiUnbound.java

// 未绑定的方法与多参数的结合运用

class This {
  void two(int i, double d) {}
  void three(int i, double d, String s) {}
  void four(int i, double d, String s, char c) {}
}

interface TwoArgs {
  void call2(This athis, int i, double d);
}

interface ThreeArgs {
  void call3(This athis, int i, double d, String s);
}

interface FourArgs {
  void call4(
    This athis, int i, double d, String s, char c);
}

public class MultiUnbound {
  public static void main(String[] args) {
    TwoArgs twoargs = This::two;
    ThreeArgs threeargs = This::three;
    FourArgs fourargs = This::four;
    This athis = new This();
    twoargs.call2(athis, 11, 3.14);
    threeargs.call3(athis, 11, 3.14, "Three");
    fourargs.call4(athis, 11, 3.14, "Four", 'Z');
  }
}
```

​		需要指出的是，我将类命名为This，并将函数式方法的第一个参数命名为athis，但你在生产代码中应该使用其他名字，以防止混淆。

### 构造函数引用

​		你还可以捕获构造函数的引用，然后通过引用调用该构造函数。

```java
// functional/CtorReference.java

class Dog {
  String name;
  int age = -1; // For "unknown"
  Dog() { name = "stray"; }
  Dog(String nm) { name = nm; }
  Dog(String nm, int yrs) { name = nm; age = yrs; }
}

interface MakeNoArgs {
  Dog make();
}

interface Make1Arg {
  Dog make(String nm);
}

interface Make2Args {
  Dog make(String nm, int age);
}

public class CtorReference {
  public static void main(String[] args) {
    MakeNoArgs mna = Dog::new; // [1]
    Make1Arg m1a = Dog::new;   // [2]
    Make2Args m2a = Dog::new;  // [3]

    Dog dn = mna.make();
    Dog d1 = m1a.make("Comet");
    Dog d2 = m2a.make("Ralph", 4);
  }
}
```

​		Dog有三个构造函数，函数式接口的make()方法反映了构造函数参数列表(make()方法名称可以不同)。

​		注意我们如何对[1]，[2]和[3]中的每一个使用Dog::new。这三个构造函数只有一个相同名称：::new，但在每种情况下赋值给不同的接口，编译器可以从中知道具体使用哪个构造函数。

​		编译器知道调用函数式方法(本例中为make())就相当于调用构造函数。

## 函数式接口

​		方法引用和Lambda表达式都必须被赋值，同时赋值需要类型信息才能使编译器保证类型的正确性。尤其是Lambda表达式，它引入了新的要求。代码示例：

```java
x -> x.toString()
```

​		我们清楚这里返回类型必须是String，但x是什么类型呢？

​		Lambda表达式包含**类型推导**(编译器会自动推导出类型信息，避免了程序员显式地声明)。编译器必须能够以某种方式推导出x的类型。

​		下面是第二个代码实例：

```java
(x, y) -> x + y
```

​		现在x和y可以是任何支持+运算符连接的数据类型，可以是两个不同的数值类型或者是一个String加任意一种可自动转换为String的数据类型(这包括了大多数类型)。但是，当Lambda表达式被赋值时，编译器必须确定x和y的确切类型以生成正确的代码。

​		该问题也适用于方法引用。假设你要传递System.out::println到你正在编写的方法，你怎么知道传递给方法的参数的类型？

​		为了解决这个问题，Java8引入了java.util.function包。它包含一组接口，这些接口是Lambda表达式和方法引用的目标类型。每个接口只包含一个抽象方法，称为**函数式方法**。

​		在编写接口时，可以使用@FunctionalInterface注解强制执行此“函数式方法”模式：

```java
// functional/FunctionalAnnotation.java

@FunctionalInterface
interface Functional {
  String goodbye(String arg);
}

interface FunctionalNoAnn {
  String goodbye(String arg);
}

/*
@FunctionalInterface
interface NotFunctional {
  String goodbye(String arg);
  String hello(String arg);
}
产生错误信息:
NotFunctional is not a functional interface
multiple non-overriding abstract methods
found in interface NotFunctional
*/

public class FunctionalAnnotation {
  public String goodbye(String arg) {
    return "Goodbye, " + arg;
  }
  public static void main(String[] args) {
    FunctionalAnnotation fa =
      new FunctionalAnnotation();
    Functional f = fa::goodbye;
    FunctionalNoAnn fna = fa::goodbye;
    // Functional fac = fa; // Incompatible
    Functional fl = a -> "Goodbye, " + a;
    FunctionalNoAnn fnal = a -> "Goodbye, " + a;
  }
}
```

​		@FunctionalInterface注解是可选的；Java会在main()中把Functional和FunctionalNoAnn都当作函数式接口来看待。在NotFunctional的定义中可看出@FunctionalInterface的作用：当接口中抽象方法多于一个时产生编译期错误。

​		仔细观察在定义f和fna时发生了什么。Functional和FunctionalNoAnn声明了是接口，然而被赋值的只是方法goodbye()。首先，这只是一个方法而不是类；其次，它甚至都不是实现了该接口的类中的方法。这是添加到Java8中的一点小魔法：如果将方法引用或Lambda表达式赋值给函数式接口(类型需要匹配)，Java会适配你的赋值到目标接口。编译器会在后台把方法引用或Lambda表达式包装进实现目标接口的类的实例中。

​		虽然FunctionalAnnotation确实符合Functional模型，但是Java不允许我们像fac定义的那样，将FunctionalAnnotation直接赋值给Functional，因为FunctionalAnnotation并没有显示地去实现Functional接口。唯一的惊喜是，Java8允许我们将函数赋值给接口，这样的语法更加简单漂亮。

​		java.util.function包旨在创建一组完整的目标接口，使得我们一般情况下不需要再定义自己的接口。主要因为基本类型的存在，导致预定义的接口数量有少许增加。如果你了解命名模式，顾名思义就能知道特定接口的作用。

​		以下是基本命名准则：

* 1.如果只处理对象而非基本类型，名称则为Function、Consumer、Predicate等。参数类型通过泛型添加。
* 2.如果接受的参数是基本类型，则由名称的第一部分表示，如：LongConsumer、DoubleFunction、IntPredicate等，但返回基本类型的Supplier接口例外。
* 3.如果返回值为基本类型，则用To表示，如：ToLongFunction\<T\>和IntToLongFunction。
* 4.如果返回值类型与参数类型相同，则是一个Operator：单个参数使用UnaryOperator，两个参数使用BinaryOperator。
* 5.如果接收参数并返回一个布尔值，则是一个**谓词**(Predicate)。
* 6.如果接受的两个参数类型不同，则名称中有一个Bi。



​		例如，为什么没有IntComparator，LongComparator和DoubleComparator呢？有BooleanSupplier却没有其他表示Boolean的接口；有通用的BiConsumer却没有用于int，long和double的BiConsumers变体(我理解他们为什么放弃这些接口)。这到底是疏忽还是有人认为其他组合使用得很少呢？

​		你还可以看到基本类型给Java添加了多少复杂性。该语言的第一版中就包含了基本类型，原因是考虑效率问题。现在，在语言的生命周期里，我们一直忍受语言设计的糟糕选择所带来的影响。

​		下面枚举了基于Lambda表达式的所有不同Function变体的示例：

```java
// functional/FunctionVariants.java

import java.util.function.*;

class Foo {}

class Bar {
  Foo f;
  Bar(Foo f) { this.f = f; }
}

class IBaz {
  int i;
  IBaz(int i) {
    this.i = i;
  }
}

class LBaz {
  long l;
  LBaz(long l) {
    this.l = l;
  }
}

class DBaz {
  double d;
  DBaz(double d) {
    this.d = d;
  }
}

public class FunctionVariants {
  static Function<Foo,Bar> f1 = f -> new Bar(f);
  static IntFunction<IBaz> f2 = i -> new IBaz(i);
  static LongFunction<LBaz> f3 = l -> new LBaz(l);
  static DoubleFunction<DBaz> f4 = d -> new DBaz(d);
  static ToIntFunction<IBaz> f5 = ib -> ib.i;
  static ToLongFunction<LBaz> f6 = lb -> lb.l;
  static ToDoubleFunction<DBaz> f7 = db -> db.d;
  static IntToLongFunction f8 = i -> i;
  static IntToDoubleFunction f9 = i -> i;
  static LongToIntFunction f10 = l -> (int)l;
  static LongToDoubleFunction f11 = l -> l;
  static DoubleToIntFunction f12 = d -> (int)d;
  static DoubleToLongFunction f13 = d -> (long)d;

  public static void main(String[] args) {
    Bar b = f1.apply(new Foo());
    IBaz ib = f2.apply(11);
    LBaz lb = f3.apply(11);
    DBaz db = f4.apply(11);
    int i = f5.applyAsInt(ib);
    long l = f6.applyAsLong(lb);
    double d = f7.applyAsDouble(db);
    l = f8.applyAsLong(12);
    d = f9.applyAsDouble(12);
    i = f10.applyAsInt(12);
    d = f11.applyAsDouble(12);
    i = f12.applyAsInt(13.0);
    l = f13.applyAsLong(13.0);
  }
}
```

​		这些Lambda表达式尝试生成适合函数签名的最简代码。在某些情况下有必要进行强制类型转换，否则编译器会报截断错误。

​		main()中的每个测试都显示了Function接口中不同类型的apply()方法。每个都产生一个与其关联的Lambda表达式的调用。

​		方法引用有自己的小魔法：

```java
/ functional/MethodConversion.java

import java.util.function.*;

class In1 {}
class In2 {}

public class MethodConversion {
  static void accept(In1 i1, In2 i2) {
    System.out.println("accept()");
  }
  static void someOtherName(In1 i1, In2 i2) {
    System.out.println("someOtherName()");
  }
  public static void main(String[] args) {
    BiConsumer<In1,In2> bic;

    bic = MethodConversion::accept;
    bic.accept(new In1(), new In2());

    bic = MethodConversion::someOtherName;
    // bic.someOtherName(new In1(), new In2()); // Nope
    bic.accept(new In1(), new In2());
  }
}
//accept()
//someOtherName()
```

​		查看BiConsumer的文档，你会看到它的函数方法为accept()。的确，如果我们将方法命名为accept()，它就可以作为方法引用。但是我们也可用不同的名称，比如someOtherName()。只要参数类型、返沪类型与BiConsumer的accpt()相同即可。

​		因此，在使用函数接口时，名称无关紧要——只要参数类型和返回类型相同。Java会将你的方法映射到接口方法。要调用方法，可以调用接口的函数式方法名，而不是你的方法名。

​		现在我们来看看，将方法引用应用于基于类的函数式接口(即那些不包含基本类型的函数式接口)。下面的例子中，创建了适合函数式方法签名的最简单的方法：

```java
// functional/ClassFunctionals.java

import java.util.*;
import java.util.function.*;

class AA {}
class BB {}
class CC {}

public class ClassFunctionals {
  static AA f1() { return new AA(); }
  static int f2(AA aa1, AA aa2) { return 1; }
  static void f3(AA aa) {}
  static void f4(AA aa, BB bb) {}
  static CC f5(AA aa) { return new CC(); }
  static CC f6(AA aa, BB bb) { return new CC(); }
  static boolean f7(AA aa) { return true; }
  static boolean f8(AA aa, BB bb) { return true; }
  static AA f9(AA aa) { return new AA(); }
  static AA f10(AA aa1, AA aa2) { return new AA(); }
  public static void main(String[] args) {
    Supplier<AA> s = ClassFunctionals::f1;
    s.get();
    Comparator<AA> c = ClassFunctionals::f2;
    c.compare(new AA(), new AA());
    Consumer<AA> cons = ClassFunctionals::f3;
    cons.accept(new AA());
    BiConsumer<AA,BB> bicons = ClassFunctionals::f4;
    bicons.accept(new AA(), new BB());
    Function<AA,CC> f = ClassFunctionals::f5;
    CC cc = f.apply(new AA());
    BiFunction<AA,BB,CC> bif = ClassFunctionals::f6;
    cc = bif.apply(new AA(), new BB());
    Predicate<AA> p = ClassFunctionals::f7;
    boolean result = p.test(new AA());
    BiPredicate<AA,BB> bip = ClassFunctionals::f8;
    result = bip.test(new AA(), new BB());
    UnaryOperator<AA> uo = ClassFunctionals::f9;
    AA aa = uo.apply(new AA());
    BinaryOperator<AA> bo = ClassFunctionals::f10;
    aa = bo.apply(new AA(), new AA());
  }
}
```

​		请注意，每个方法名称都是随意的(如f1()，f2()等)。正如你刚才看到的，一旦将方法引用赋值给函数接口，我们就可以调用与该接口关联的函数方法。在此示例中为get()、compare()、accept()、apply()和test()。

### 多参数函数式接口

​		java.util.functional中的接口是有限的。比如有BiFunction，但也仅此而已。如果需要三参数函数的接口怎么办？其实这些接口非常简单，很容易查看Java库源代码并自行创建。代码示例：

```java
// functional/TriFunction.java

@FunctionalInterface
public interface TriFunction<T, U, V, R> {
    R apply(T t, U u, V v);
}
```

​		简单测试，验证它是否有效：

```java
// functional/TriFunctionTest.java

public class TriFunctionTest {
  static int f(int i, long l, double d) { return 99; }
  public static void main(String[] args) {
    TriFunction<Integer, Long, Double, Integer> tf =
      TriFunctionTest::f;
    tf = (i, l, d) -> 12;
  }
}
```

​			这里我们同时测试了方法引用和Lambda表达式。

### 缺少基本类型的函数

​		让我们重温一下BiConsumer，看看我们将如何创建各种缺失的预定义组合，涉及int，long和double：

```java
// functional/BiConsumerPermutations.java

import java.util.function.*;

public class BiConsumerPermutations {
  static BiConsumer<Integer, Double> bicid = (i, d) ->
    System.out.format("%d, %f%n", i, d);
  static BiConsumer<Double, Integer> bicdi = (d, i) ->
    System.out.format("%d, %f%n", i, d);
  static BiConsumer<Integer, Long> bicil = (i, l) ->
    System.out.format("%d, %d%n", i, l);
  public static void main(String[] args) {
    bicid.accept(47, 11.34);
    bicdi.accept(22.45, 92);
    bicil.accept(1, 11L);
  }
}
//47, 11.340000
//92, 22.450000
//1, 11
```

​		这里使用System.out.format()来显示。它类似于System.out.println()但提供了更多的显示选项。这里，%f表示我将n作为浮点值给出，%d表示n是一个整数值。这其中可以包含空格，输入%n会换行——当然使用传统的\n也能换行，但%n是自动跨平台的，这是使用format()的另一个原因。

​		上例只是简单使用了合适的包装类型，而装箱和拆箱负责它与基本类型之间的来回转换。又比如，我们可以将包装类型和Function一起使用，而不去用各种针对基本类型的预定义接口。代码示例：

```java
// functional/FunctionWithWrapped.java

import java.util.function.*;

public class FunctionWithWrapped {
  public static void main(String[] args) {
    Function<Integer, Double> fid = i -> (double)i;
    IntToDoubleFunction fid2 = i -> i;
  }
}
```

​		如果没有强制转换，则会收到错误消息：“Integer cannot be converted to Double”，而使用IntToDoubleFunction就没有此类问题。IntToDoubleFunction接口的源代码是这样的：

```java
@FunctionalInterface 
public interface IntToDoubleFunction { 
  double applyAsDouble(int value); 
}
```

​		因为我们可以简单地写Function\<Integer，Double\>并产生正常的结果，所以用基本类型(IntToDoubleFunction)的唯一理由是可以避免传递参数和返回结果过程中的自动拆装箱，进而提升性能。

​		似乎是考虑到使用频率，某些函数类型并没有预定义。

​		当然，如果因为缺少针对基本类型的函数式接口造成性能问题，你可以轻松编写自己的接口(参考Java源代码)——尽管这里出现性能瓶颈的可能性不大。

## 高阶函数

​		这个名字可能听起来令人生畏，但是：高阶函数(Higher-order Function)只是一个消费或产生函数的函数。

​		我们先来看看如何产生一个函数：

```java
// functional/ProduceFunction.java

import java.util.function.*;

interface
FuncSS extends Function<String, String> {} // [1]

public class ProduceFunction {
  static FuncSS produce() {
    return s -> s.toLowerCase(); // [2]
  }
  public static void main(String[] args) {
    FuncSS f = produce();
    System.out.println(f.apply("YELLING"));
  }
}
//yelling
```

​		这里，produce()是高阶函数。

[1]使用继承，可以轻松地为专用接口创建别名。

[2]使用Lambda表达式，可以轻松地在方法中创建和返回一个函数。

​		要消费一个函数，消费函数需要在参数列表正确地描述函数类型。代码示例：

```java
// functional/ConsumeFunction.java

import java.util.function.*;

class One {}
class Two {}

public class ConsumeFunction {
  static Two consume(Function<One,Two> onetwo) {
    return onetwo.apply(new One());
  }
  public static void main(String[] args) {
    Two two = consume(one -> new Two());
  }
}
```

​		当基于消费函数生成新函数时，事情就变得相当有趣了。代码示例如下：

```java
// functional/TransformFunction.java

import java.util.function.*;

class I {
  @Override
  public String toString() { return "I"; }
}

class O {
  @Override
  public String toString() { return "O"; }
}

public class TransformFunction {
  static Function<I,O> transform(Function<I,O> in) {
    return in.andThen(o -> {
      System.out.println(o);
      return o;
    });
  }
  public static void main(String[] args) {
    Function<I,O> f2 = transform(i -> {
      System.out.println(i);
      return new O();
    });
    O o = f2.apply(new I());
  }
}
//I
//O
```

​		在这里，transform()生成一个与传入的函数具有相同签名的函数，但是你可以生成任何你想要的类型。

​		这里使用了Function接口中名为andThen()的默认方法，该方法专门用于操作函数。顾名思义，在调用in函数之后调用andThen()(还有个compose()方法，它在in函数之前应用新的函数)。要附加一个andThen()函数，我们只需要将该函数作为参数传递。transform()产生的是一个新函数，它将in动作与andThen()参数的动作结合起来。

## 闭包

