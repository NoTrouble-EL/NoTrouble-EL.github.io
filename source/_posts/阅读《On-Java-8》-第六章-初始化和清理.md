---
title: 阅读《On Java 8》--第六章 初始化和清理
date: 2021-04-27 15:53:26
tags:
- JAVASE
- 《On Java 8》
categories: JAVASE
mathjax: true
---

​	“不安全”的编程是造成编程代价昂贵的罪魁祸首之一。有两个安全性问题：初始化和清理。

​		Java采用了构造器的概念，另外还使用了垃圾收集器(Garbage Collector，GC)去自动回收不再被使用的对象所占的资源。这一章将讨论初始化和清理的问题，以及在Java中对它们的支持。

 <!-- more --> 

## 利用构造器保证初始化

​		在Java中，类的设计者通过构造器保证每个对象的初始化。如果一个类有构造器，那么Java会在用户使用对象之前自动调用对象的构造器方法，从而保证初始化。下一个挑战是如何命名构造器方法。在Java中使用了与C++同样的方式：构造器名称和类名相同。在初始化过程中自动调用构造器方法是有意义的。

```java
// housekeeping/SimpleConstructor.java
// Demonstration of a simple constructor

class Rock {
    Rock() { // 这是一个构造器
        System.out.print("Rock ");
    }
}

public class SimpleConstructor {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Rock();
        }
    }
}
//Rock Rock Rock Rock Rock Rock Rock Rock Rock Rock 
```

​		现在，当创建一个对象时：new Rock()，内存被分配，构造器被调用。构造器保证了对象在你使用之前进行了正确的初始化。

​		有一点需要注意，构造器方法名与类名相同，不需要符合首字母小写的风格。

​		跟其他方法一样，构造器方法也可以传入参数来定义如何创建一个对象：

```java
// housekeeping/SimpleConstructor2.java
// Constructors can have arguments

class Rock2 {
    Rock2(int i) {
        System.out.print("Rock " + i + " ");
    }
}

public class SimpleConstructor2 {
    public static void main(String[] args) {
        for (int i = 0; i < 8; i++) {
            new Rock2(i);
        }
    }
}
//Rock 0 Rock 1 Rock 2 Rock 3 Rock 4 Rock 5 Rock 6 Rock 7
```

​		如果类Tree有一个构造方法，只接收一个参数用来表示树的高度，那么你可以像下面这样创建一棵树：

```java
Tree t = new Tree(12); // 12-foot 树
```

​		如果Tree(int)是唯一的构造器，那么编译器就不允许你以其他任何方式创建Tree类型的对象。

​		构造器消除了一类重要的问题，使得代码更易读。例如：在上面的代码中，你看不到initialize()方法的显示调用，而从概念上来看，initialize()方法应该与对象的创建分离。在Java中，对象的创建和初始化对象是统一的概念，二者不可分割。

​		构造器没有返回值，它是一种特殊的方式。但它和返回类型为void的普通方法不同，普通方法可以返回空值，你还能选择让它返回别的类型；而构造器没有返回值，却同时也没有给你选择的余地(new表达式虽然返回了刚创建的对象的引用，但构造器本身却没有任何返回值)。如果它有返回值，并且你也可以自己选择让它返回什么，那么编译器就还得知道接下来该怎么处理那个返回值(这个返回值没有接收者)。

## 方法重载

​		在Java中，还有一个因素也促成了必须使用方法重载：构造器。因为构造器方法肯定与类名相同，所以一个类中只有一个构造器名。那么你怎么通过不同的方式创建一个对象呢？例如，你想创建一个类，这个类的初始化有两种方式：一种是标准化方式，另一种是从文件中读取信息的方式。你需要两个构造器：无参构造器和有一个String类型参数的构造器，该参数传入文件名。两个构造器具有相同的名字——与类名相同。因此，方法重载是必要的，它允许方法具有相同的方法名但接收的参数不同。尽管方法重载对于构造器是重要的，但是也可以很方便对其他任何方法进行重载。

```java
// housekeeping/Overloading.java
// Both constructor and ordinary method overloading

class Tree {
    int height;
    Tree() {
        System.out.println("Planting a seedling");
        height = 0;
    }
    Tree(int initialHeight) {
        height = initialHeight;
        System.out.println("Creating new Tree that is " + height + " feet tall");
    }
    void info() {
        System.out.println("Tree is " + height + " feet tall");
    }
    void info(String s) {
        System.out.println(s + ": Tree is " + height + " feet tall");
    }
}
public class Overloading {
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            Tree t = new Tree(i);
            t.info();
            t.info("overloaded method");
        }
        new Tree(); 
    }
}
//Creating new Tree that is 0 feet tall
//Tree is 0 feet tall
//overloaded method: Tree is 0 feet tall
//Creating new Tree that is 1 feet tall
//Tree is 1 feet tall
//overloaded method: Tree is 1 feet tall
//Creating new Tree that is 2 feet tall
//Tree is 2 feet tall
//overloaded method: Tree is 2 feet tall
//Creating new Tree that is 3 feet tall
//Tree is 3 feet tall
//overloaded method: Tree is 3 feet tall
//Creating new Tree that is 4 feet tall
//Tree is 4 feet tall
//overloaded method: Tree is 4 feet tall
//Planting a seedling
```

### 区分方法重载

​		如果两个方法命名相同，Java是怎么知道你调用的是哪个呢？有一条简单的规则：每个被重载的方法必须有独一无二的参数列表。你甚至可以根据参数列表中的参数顺序来区分不同的方法，尽管这会造成代码难以维护：

```java
// housekeeping/OverloadingOrder.java
// Overloading based on the order of the arguments

public class OverloadingOrder {
    static void f(String s, int i) {
        System.out.println("String: " + s + ", int: " + i);
    }

    static void f(int i, String s) {
        System.out.println("int: " + i + ", String: " + s);
    }

    public static void main(String[] args) {
        f("String first", 1);
        f(99, "Int first");
    }
}
//String: String first, int: 1
//int: 99, String: Int first
```

​		两个f()方法具有相同的参数，但是参数顺序不同，根据这个就可以区分它们。

### 重载与基本类型

​		基本类型可以自动从较小的类型转型为较大的类型。当这与重载结合时，这会令人有点困惑：

```java
// housekeeping/PrimitiveOverloading.java
// Promotion of primitives and overloading

public class PrimitiveOverloading {
    void f1(char x) {
        System.out.print("f1(char)");
    }
    void f1(byte x) {
        System.out.print("f1(byte)");
    }
    void f1(short x) {
        System.out.print("f1(short)");
    }
    void f1(int x) {
        System.out.print("f1(int)");
    }
    void f1(long x) {
        System.out.print("f1(long)");
    }
    void f1(float x) {
        System.out.print("f1(float)");
    }
    void f1(double x) {
        System.out.print("f1(double)");
    }
    void f2(byte x) {
        System.out.print("f2(byte)");
    }
    void f2(short x) {
        System.out.print("f2(short)");
    }
    void f2(int x) {
        System.out.print("f2(int)");
    }
    void f2(long x) {
        System.out.print("f2(long)");
    }
    void f2(float x) {
        System.out.print("f2(float)");
    }
    void f2(double x) {
        System.out.print("f2(double)");
    }
    void f3(short x) {
        System.out.print("f3(short)");
    }
    void f3(int x) {
        System.out.print("f3(int)");
    }
    void f3(long x) {
        System.out.print("f3(long)");
    }
    void f3(float x) {
        System.out.print("f3(float)");
    }
    void f3(double x) {
        System.out.print("f3(double)");
    }
    void f4(int x) {
        System.out.print("f4(int)");
    }
    void f4(long x) {
        System.out.print("f4(long)");
    }
    void f4(float x) {
        System.out.print("f4(float)");
    }
    void f4(double x) {
        System.out.print("f4(double)");
    }
    void f5(long x) {
        System.out.print("f5(long)");
    }
    void f5(float x) {
        System.out.print("f5(float)");
    }
    void f5(double x) {
        System.out.print("f5(double)");
    }
    void f6(float x) {
        System.out.print("f6(float)");
    }
    void f6(double x) {
        System.out.print("f6(double)");
    }
    void f7(double x) {
        System.out.print("f7(double)");
    }
    void testConstVal() {
        System.out.print("5: ");
        f1(5);f2(5);f3(5);f4(5);f5(5);f6(5);f7(5);
        System.out.println();
    }
    void testChar() {
        char x = 'x';
        System.out.print("char: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testByte() {
        byte x = 0;
        System.out.print("byte: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testShort() {
        short x = 0;
        System.out.print("short: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testInt() {
        int x = 0;
        System.out.print("int: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testLong() {
        long x = 0;
        System.out.print("long: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testFloat() {
        float x = 0;
        System.out.print("float: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testDouble() {
        double x = 0;
        System.out.print("double: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }

    public static void main(String[] args) {
        PrimitiveOverloading p = new PrimitiveOverloading();
        p.testConstVal();
        p.testChar();
        p.testByte();
        p.testShort();
        p.testInt();
        p.testLong();
        p.testFloat();
        p.testDouble();
    }
}
//5: f1(int)f2(int)f3(int)f4(int)f5(long)f6(float)f7(double)
//char: f1(char)f2(int)f3(int)f4(int)f5(long)f6(float)f7(double)
//byte: f1(byte)f2(byte)f3(short)f4(int)f5(long)f6(float)f7(double)
//short: f1(short)f2(short)f3(short)f4(int)f5(long)f6(float)f7(double)
//int: f1(int)f2(int)f3(int)f4(int)f5(long)f6(float)f7(double)
//long: f1(long)f2(long)f3(long)f4(long)f5(long)f6(float)f7(double)
//float: f1(float)f2(float)f3(float)f4(float)f5(float)f6(float)f7(double)
//double: f1(double)f2(double)f3(double)f4(double)f5(double)f6(double)f7(double)
```

​		如果传入的参数类型大于方法期望接收的参数类型，你必须首先做下转换，如果你不做的话，编译器就会报错。

### 返回值重载

​		经常有人会困惑，“为什么只能通过方法名和参数列表，不能通过方法名和返回值区分方法呢？”。例如：

```java
void f(){}
int f() {return 1;}
```

​		有些情况下，编译器很容易就可以从上下文准确推断出该调用哪个方法，如int x = f()。

​		但是，你可以调用一个方法且忽略返回值。这叫做调用一个函数的副作用，因为你不在乎返回值，只是想利用方法做些事。所以如果你直接调用f()，Java编译器就不知道你想调用哪个方法，阅读者也不明所以。因为这个原因，所以你不能根据返回值类型区分重载的方法。

## 无参构造器

​		一个无参构造器就是不接收参数的构造器，用来创建一个“默认的对象”。如果你创建一个类，类中没有构造器，那么编译器就会自动为你创建一个无参构造器：

```java
// housekeeping/DefaultConstructor.java
class Bird {}
public class DefaultConstructor {
    public static void main(String[] args) {
        Bird bird = new Bird(); // 默认的
    }
}
```

​		表达式new Bird()创建了一个新对象，调用了无参构造器，尽管在Bird类中并没有显示的定义无参构造器。试想一下若果没有构造器，我们如何创建一个对象呢。但是，一旦你显示地定义了构造器(无论有参还是无参)，编译器就不会自动为你创建无参构造器：

```java
// housekeeping/NoSynthesis.java
class Bird2 {
    Bird2(int i) {}
    Bird2(double d) {}
}
public class NoSynthesis {
    public static void main(String[] args) {
        //- Bird2 b = new Bird2(); // No default
        Bird2 b2 = new Bird2(1);
        Bird2 b3 = new Bird2(1.0);
    }
}
```

​		如果你调用了new Bird2()，编译器会提示找不到匹配的构造器。当类中没有构造器时，编译器会说“你一定需要构造器，那么让我为你创建一个吧”。但是如果类中有构造器，编译器会说“你已经写了构造器了，所以肯定知道你在做什么，如果你没有创建默认的构造器，说明你本来就不需要”。

## this关键字

​		对于两个相同类型的对象a和b，你可能在想如何调用这两个对象的peel()方法：

```java
// housekeeping/BananaPeel.java

class Banana {
    void peel(int i) {
        /*...*/
    }
}
public class BananaPeel {
    public static void main(String[] args) {
        Banana a = new Banana(), b = new Banana();
        a.peel(1);
        b.peel(2);
    }
}
```

​		如果只有一个方法peel()，那么怎么知道调用的是对象a的peel()方法还是对象b的peel()方法呢？编译器做了一些底层工作，所以你可以像这样编写代码。peel()方法中第一个参数隐秘地传入了一个指向操作对象的引用，因此，上述例子中的方法调用如下：

```java
Banana.peel(a, 1)
Banana.peel(b, 2)
```

​		这是在内部实现的，你不可以直接这么编写代码，编译器是不会接受，但能说明到地方生了什么。假设现在在方法内部，你想获得当前对象的引用。但是，对象引用是被秘密地传达给编译器——并不在参数列表中。方便的是，有一个关键字：this。this关键字只能在非静态方法内部使用。当你调用一个对象的方法时，this生成了一个对象引用。你可以像对待其他引用一样对待这个引用。如果你在一个类的方法里调用该类的其他方法，不要使用this，直接调用即可，this自动地应用于其他方法上了。因此你可以像这样：

```java
// housekeeping/Apricot.java

public class Apricot {
    void pick() {
        /* ... */
    }

    void pit() {
        pick();
        /* ... */
    }
}
```

​		在pit()方法中，你可以使用this.pick()，但是没有必要。编译器自动为你做了这些。this关键字只用在一些必须显式使用当前对象引用的特殊场合。例如，用在return语句中返回当前对象的引用。

```java
// housekeeping/Leaf.java
// Simple use of the "this" keyword

public class Leaf {

    int i = 0;

    Leaf increment() {
        i++;
        return this;
    }

    void print() {
        System.out.println("i = " + i);
    }

    public static void main(String[] args) {
        Leaf x = new Leaf();
        x.increment().increment().increment().print();
    }
}
//i = 3
```

​		因为increment()通过this关键字返回当前对象的引用，因此在相同的对象上可以轻易地执行多次操作。

​		this关键字在向其他方法传递当前对象时也很有用：

```java
// housekeeping/PassingThis.java

class Person {
    public void eat(Apple apple) {
        Apple peeled = apple.getPeeled();
        System.out.println("Yummy");
    }
}

public class Peeler {
    static Apple peel(Apple apple) {
        // ... remove peel
        return apple; // Peeled
    }
}

public class Apple {
    Apple getPeeled() {
        return Peeler.peel(this);
    }
}

public class PassingThis {
    public static void main(String[] args) {
        new Person().eat(new Apple());
    }
}
//Yummy
```

​		Apple因为某些原因，必须调用一个外部工具方法Peeler.peel()做一些行为。必须使用this才能将自身传递给外部方法。

### 在构造器中调用构造器

​		当你在一个类中写了多个构造器，有时你想在一个构造器中调用另一个构造器来避免代码重复。你通过this关键字实现这样的调用。

​		通常当你说this，意味着“这个对象”或“当前对象”，它本身生成对当前对象的引用。在一个构造器中，当你给this一个参数列表时，它是另一层意思。它通过最直接的方式显示地调用匹配参数列表的构造器：

```java
// housekeeping/Flower.java
// Calling constructors with "this"

public class Flower {
    int petalCount = 0;
    String s = "initial value";

    Flower(int petals) {
        petalCount = petals;
        System.out.println("Constructor w/ int arg only, petalCount = " + petalCount);
    }

    Flower(String ss) {
        System.out.println("Constructor w/ string arg only, s = " + ss);
        s = ss;
    }

    Flower(String s, int petals) {
        this(petals);
        //- this(s); // Can't call two!
        this.s = s; // Another use of "this"
        System.out.println("String & int args");
    }

    Flower() {
        this("hi", 47);
        System.out.println("no-arg constructor");
    }

    void printPetalCount() {
        //- this(11); // Not inside constructor!
        System.out.println("petalCount = " + petalCount + " s = " + s);
    }

    public static void main(String[] args) {
        Flower x = new Flower();
        x.printPetalCount();
    }
}
//Constructor w/ int arg only, petalCount = 47
//String & int args
//no-arg constructor
//petalCount = 47 s = hi
```

​		从构造器Flower(String s, int petals)可以看出，其中只能通过this调用一次构造器。另外，必须首先调用构造器，否则编译器会报错。这个例子同样展示了this的另一个用法。参数列表中的变量名s和成员变量s相同，会引起混淆。你可以通过this.s表名你指的是成员变量s，从而避免重复。在printPetalConut()方法中，编译器不允许你在一个构造器之外的方法里调用构造器。

### static的含义

​		记住了this关键字的内容，你会对static修饰的方法更加具有深入的理解：static方法中不会存在this。你不能在静态方法中调用非静态方法(反之可以)。静态方法是为类而创建的，不需要任何对象。事实上，这就是静态方法的主要目的，静态方法看起来就像全局方法一样，但Java中不允许全局方法，一个类中的静态方法可以访问其他静态方法和静态属性。一些人认为静态方法不是面向对象的，因为它们的确具有全局方法的语义。使用静态方法，因为不存在this，所以你没有向一个对象发消息。的确，如果你发现代码中出现了大量的static方法，就该重新考虑自己的设计了。然而，static的概念很实用，许多时候都要用到它。

## 垃圾回收器

​		Java中有垃圾回收器回收无用对象占用的内存。但现在考虑一种特殊情况：你创建的对象不是通过new来分配内存的，而垃圾回收器只知道如何释放new创建的对象的内存，所以它不知道如何回收不是new分配的内存。为了处理这种情况，Java允许在类中定义一个名为finalize()的方法。

​		它的工作原理“假定”是这样的：当垃圾回收器准备回收对象的内存时，首先会调用其finalize()方法，并在下一轮的垃圾回收动作发生时，才会真正回收对象占用的内存。所以如果你打算使用finalize()，就能在垃圾回收时做一些重要的清理工作。finalize()是一个潜在的编程陷阱，因为一些程序员会一开始把它误认为是C++中的析构函数(C++在销毁对象时会调用这个函数)。所以有必要明确区分一下：在C++中，对象总是被销毁的，而在Java中，对象并非总是被垃圾回收，或者换句话说：

* 1.对象可能不被垃圾回收
* 2.垃圾回收不等同于析构



​		这意味着在你不需要某个对象之前，如果必须执行某些动作，你得自己去做。Java没有析构器或类似的概念，所以你必须得自己创建一个普通的方法完成这项清理工作。例如，对象在创建的过程中将自己绘制到屏幕上。如果不是明确地从屏幕上将其擦除，它可能永远得不到清理。如果在finalize()方法中加入某种擦除功能，那么当垃圾回收发生时，finalize()方法被调用(不保证一定会发生)图像就会被擦除，要是“垃圾回收”没有发生，图像则仍会保留下来。

​		也许你会发现，只要程序没有濒临内存用完的那一刻，对象占用的空间就总也得不到释放。如果程序执行结束，而垃圾回收器一直没有释放你创建的任何对象的内存，则当程序退出时，那些资源会全部交还给操作系统。这个策略是恰当的，因为垃圾回收本身也有开销，要是不使用它，那就不用支付这部分开销了。

### finalize()的用途

​		如果你不能将finalize()方法作为通用的清理方法，那么这个方法有什么用呢？这引入了要记住的第3点：3.垃圾回收只与内存有关。

​		也就是说，使用垃圾回收的唯一原因就是为了回收程序不再使用的内存。所以对于与垃圾回收有关的任何行为来说(尤其是finalize()方法)，它们也必须同内存及其回收有关。

​		但这是否意味着如果对象中包括其他对象，finalize()方法就应该明确释放那些对象？不是，无论对象是如何创建的，垃圾回收器都会负责释放对象所占用的所有内存。这就将对finalize()的需求限制到一种特殊情况，即通过某种创建对象方式之外的方式为对象分配了存储空间。不过，你可能会想，Java中万物皆对象，这种情况怎么可能发生？

​		看起来之所以有finalize()方法，是因为在分配内存时可能采用了类似C语言中的做法，而非Java中的通常做法。这种情况主要发生在使用“本地方法”的情况下，本地方法是一种Java语言调用非Java语言代码的形式。本地方法目前只支持C和C++，但是它们可以调用其他语言写的代码，所以实际上可以调用任何代码。在非Java代码中，也许会调用C的malloc()函数系列来分配存储空间，而且除非调用free()函数，不然存储空间永远得不到释放，造成内存泄露。但是，free()是C和C++中的函数，所以你需要在finalize()方法里用本地方法调用它。

### 你必须实施清理

​		要清理一个对象，用户必须在需要清理的时候执行清理动作的方法。在C++中，所有对象都会被销毁，或者说应该被销毁。如果在C++创建了一个局部对象，此时的销毁动作发生在以“右花括号”为边界的、此对象作用域的末尾处。如果对象是用new创建的，那么程序员调用C++的delete操作符，就会调用相应的析构函数。如果程序员忘记调用delete，那么永远不会调用析构函数，这样就导致内存泄漏，对象的其他部分也不会得到清理。相反。在Java中，没有用于释放对象的delete，因为垃圾回收器会帮助你释放存储空间。设置可以肤浅地认为，正是由于垃圾回收的存在，使得Java没有析构函数。然而，随着学习的深入，你会明白垃圾回收器的存在并不能完全替代析构函数(而且绝不能直接调用finalize()，所以这也不是一种解决方案)。如果希望进行除释放存储空间之外的清理工作，还是得明确用某个恰当的Java方法：这就等同于使用了析构函数，只是没有它方便。

​		记住，无论是“垃圾回收”还是“终结”，都不保证一定会发生。如果JVM并未面临内存耗尽的情形，它可能不会浪费时间执行垃圾回收以恢复内存。

### 终结条件

​		通常，不能指望finalize()，你必须创建其他的“清理”方法，并明确地调用它们。所以看起来，finalize()只对大部分程序员很难用到的一些晦涩内存清理有用了。但是，finalize()还有一个有趣的用法，它不依赖于每次都要对finalize()进行调用，这就是对象终结条件的验证。

​		当对某个对象不感兴趣时——也就是它将被清理了，这个对象应该处于某种状态，这种状态下它占用的内存可以被安全地释放掉。例如，如果对象代表了一个打开的文件，在对象被垃圾回收之前程序员应该关闭这个文件。只要对象中存在没有被适当清理的部分，程序就存在很隐晦的bug。finalize()可以用来最终发现这个情况。尽管它并不是总被调用。如果某次finalize()的动作使得bug被发现，那么就可以据此找出问题所在——这才是人们真正关心的：

```java
// housekeeping/TerminationCondition.java
// Using finalize() to detect a object that
// hasn't been properly cleaned up

import onjava.*;

class Book {
    boolean checkedOut = false;

    Book(boolean checkOut) {
        checkedOut = checkOut;
    }

    void checkIn() {
        checkedOut = false;
    }

    @Override
    protected void finalize() throws Throwable {
        if (checkedOut) {
            System.out.println("Error: checked out");
        }
        // Normally, you'll also do this:
        // super.finalize(); // Call the base-class version
    }
}

public class TerminationCondition {

    public static void main(String[] args) {
        Book novel = new Book(true);
        // Proper cleanup:
        novel.checkIn();
        // Drop the reference, forget to clean up:
        new Book(true);
        // Force garbage collection & finalization:
        System.gc();
        new Nap(1); // One second delay
    }

}
//Error: checked out
```

​		本例的终结条件是：所有Book对象在被垃圾回收之前必须被登记。在main()方法中，有一本书没有登记。要是没有finalize()方法来验证终结条件，将会很难发现这个bug。

​		你可能会注意到使用了@Override。@意味着这是一个注解，注解是关于代码的额外信息。在这里，该注解告诉编译器这不是偶然地重新定义在每个对象中都存在的finalize()方法——程序员知道自己在做什么。编译器确保你没有拼错方法名，而且确保那个方法存在于基类中。

​		注意，System.gc()用于强制进行终结动作。但是即使不这么做，只要重复地执行程序，最终也能找出错误的Book对象。

​		你应该总是假设基类版本的finalize()也要做一些重要的事情，使用super调用它，就像Book.finalize()中看到的那样。

### 垃圾回收器如何工作

​		你可以把C++里的堆想象成一个院子，里面每个对象都负责管理自己的地盘。一段时间后，对象可能被销毁，但地盘必须复用。在某些Java虚拟机中，堆的实现既然不同：它更像一个传送带，每分配一个新对象，它就向前移动一格。这意味着对象存储空间的分配速度特别快。Java的“堆指针”只是简单地移动到尚未分配的区域，所以它的效率与C++在栈上分配空间的效率相当。当然实际过程中，在薄记工作方面还有少量额外的开销，但是这部分开销比不上查找可用空间开销大。

​		你可能意识到了，Java中的堆并非完全像传送带那样工作。要是那样的话，势必会导致频繁的内存页面调度——将其移进移出硬盘，因此会显得需要拥有比实际需要更多的内存。页面调度会显著影响性能。最终，在创建了足够多的对象后，内存资源被耗尽。其中的秘密在于垃圾回收器的介入。当它工作时，一边回收内存，一边使堆中的对象紧凑排列，这样“堆指针”就可以很容易地移动到更靠近传送带的开始处，也就尽量避免了页面错误。垃圾回收器通过重新排列对象，实现了一种高速的，有无限空间可分配的堆模型。

​		要想理解Java中的垃圾回收，先了解其它系统中的垃圾回收机制将会很有帮助。一种简单但速度很慢的垃圾回收机制叫作引用计数。每个对象中含有一个引用计数器，每当有引用指向该对象时，引用计数加1.当引用离开作用域或被置为null时，引用计数减1.因此管理引用计数是一个开销不大但是在程序的整个生命周期频繁发生的负担。垃圾回收器会遍历含有全部对象的列表，当发现某个对象的引用计数为0时，就释放其占用空间。这个机制存在一个缺点：如果对象之间存在循环引用，那么它们的引用计数都不为0，就会出现应该被回收但无法被回收的情况。对垃圾回收器而言，定位这样的循环引用所需要的工作量极大。引用计数常用来说明垃圾回收的工作方式，但似乎从未被应用于任何一种JVM实现中。

​		在更快的策略中，垃圾回收器并非基于引用计数。它们的依据是：对于任意“活”的对象，一定能最终追溯到其存活在栈或静态存储区中的引用。这个引用链条可能会穿过数个对象层次，由此，如果从栈或静态存储区出发，遍历所有的引用，你将会发现所有“活”的对象。对于发现的每个引用，必须追踪它所引用的对象，然后是该对象包含的所有引用，如此反复进行，知道访问完“根源于栈活静态存储区的引用”所形成的整个网络。你所访问过的对象一定是“活”的。注意，这解决了对象间循环引用的问题，这些对象不会被发现，因此也就被自动回收了。

​		在这种方式下，JVM采用了一种自适应的垃圾回收技术。至于如何处理找到的存活对象，取决于不同的JVM实现。其中，有一种叫做停止-复制(stop-and-copy)。顾名思义，这需要先暂停程序的运行，然后将所有存活的对象从当前堆复制到另一个堆，没有复制的就是需要被垃圾回收的。另外，当对象被复制到新堆时，它们是一个挨着一个紧凑排列，然后就可以按照前面描述的那样简单、直接地分配新空间了。

​		当对象从一处复制到另一处，所有指向它的引用都必须修正。位于栈或静态存储区的引用可以直接被修正，但可能还有其他指向这些对象的引用，它们在遍历的过程中才能被找到。

​		这种所谓的“复制回收器”效率低下主要因为两个原因。其一：得有两个堆，然后在这两个分离的堆之间来回折腾，得围护比实际需要多一倍的空间。某些Java虚拟机对此问题的处理方式是，按需从堆中分配几块较大的内存，复制动作发生在这些大块内存之间。

​		其二在于复制本身。一旦程序进入稳定状态之后，可能只产生少量垃圾，甚至没有垃圾。尽管如此，复制回收器仍然会将所有内存从一处复制到另一处，这很浪费。为了避免这种状态，一些JVM会进行检查：要是没有新垃圾产生，就会转换到另一种模式(即“自适应”)。这种模式称为标记-清扫(mark-and-sweep)对一般用途而言，“标记-清扫”方式速度相当慢，但是当你知道程序只会产生少量垃圾甚至不产生垃圾时，它的速度就很快了。

​		“标记-清扫”所依据的思路仍然是从栈和静态存储区出发，遍历所有的引用，找出所有存活的对象。但是，每当找到一个存活对象，就给对象设一个标记，并不回收它。只有当标记过程完成后，清理动作才开始。在清理过程中，没有标记的对象将被释放，不会发生任何复制动作。“标记-清扫”后剩下的堆空间是不连续的，垃圾回收器要是希望得到连续空间的话，就需要重新整理剩下的对象。

​		“停止-复制”指的是这种垃圾回收动作不是在后台进行的；相反，垃圾回收动作发生的同时，程序将会暂停。许多参考文档将垃圾回收视为低优先级的后台进程，但是早期版本的JVM并不是这么实现垃圾回收器的。当可用内存较低时，垃圾回收器会暂停程序。同样，“标记-清扫”工作也必须在程序暂停的情况下才能进行。

​		这里讨论的JVM中，内存分配以较大的“块”为单位。如果对象较大，它会占用单独的块。严格来说，“停止-复制”要求在释放旧对象之前，必须先将所有存活对象从旧堆复制到新堆，这导致了大量的内存复制行为。有了块，垃圾回收器就可以把对象复制到废弃的块。每个块都有年代数来记录自己是否存活。通常，如果块在某处被引用，其年代数加1，垃圾回收器会对上次回收动作之后新分配的块进行整理。这对处理大量短命的临时对象很有帮助。垃圾回收器会定期进行完整的清理动作——大型对象仍然不会复制(只是年代数增加)，含有小型对象的那些块则被复制并整理。JVM会监视，如果所有对象都很稳定，垃圾回收的效率降低的话，就切换到“标记-清扫”方式。同样，JVM会跟踪“标记-清扫”的效果。如果堆空间出现很多碎片，就会切回“停止-复制”方式。这就是“自适应”的由来，你可以给它个啰嗦的称呼：“自适应的、分代的、停止-复制、标记-清扫”式的垃圾回收器。

​		JVM中有许多附加技术用来提升速度。尤其是加载器操作有关的，被称为“即时”(Just-In-time，JIT)编译器的技术。这种技术可以把程序全部或部分翻译成本地机器码，所以不需要JVM来进行翻译，因此运行得更快。当需要装载某个类时，编译器先会找到其.class文件，然后将该类得字节码装入内存。你可以让即时编译器编译所有代码，但这种做法有两个缺点：一是这种加载动作贯穿整个程序生民周期内，累加起来需要花更多时间；二是会增加可执行代码得长度，这会导致页面调度，从而一定降低程序速度。另一种做法称为惰性评估，意味着即时编译器只有在必要的时候才编译代码。这样，从未被执行的代码也许就压根不会被JIT编译。新版JDK中的Java Ho'tSpot技术就采用了类似的做法，代码每被执行一次就优化一些，所执行次数越多，它的速度就越快。

## 成员初始化

​		Java尽量保证所有变量在使用前都能得到恰当的初始化。对于方法的局部变量，这种保证会以编译错误的方式呈现：

```java
void f() {
    int i;
    i++;
}
```

​		你会得到一条错误，告诉你i可能尚未初始化。编译器可以为i赋一个默认值，但是未初始化的局部变量更有可能是程序员的疏忽，所以采用默认值反而会掩盖这种失误。强制程序员提供一个初始值，往往能帮助找出程序里的bug。

​		若类的成员变量是基本类型，情况会变得有些不同。类的每个基本类型数据成员保证都会有一个初始值：

```java
// housekeeping/InitialValues.java
// Shows default initial values

public class InitialValues {
    boolean t;
    char c;
    byte b;
    short s;
    int i;
    long l;
    float f;
    double d;
    InitialValues reference;

    void printInitialValues() {
        System.out.println("Data type Initial value");
        System.out.println("boolean " + t);
        System.out.println("char[" + c + "]");
        System.out.println("byte " + b);
        System.out.println("short " + s);
        System.out.println("int " + i);
        System.out.println("long " + l);
        System.out.println("float " + f);
        System.out.println("double " + d);
        System.out.println("reference " + reference);
    }

    public static void main(String[] args) {
        new InitialValues().printInitialValues();
    }
}
//Data type Initial value
//boolean false
//char[NUL]
//byte 0
//short 0
//int 0
//long 0
//float 0.0
//double 0.0
//reference null
```

​		可见尽管数据成员的初始值还没有给出，但它们确实有初值(char值为0，所以显示为空白)。所以这样至少不会出现“未初始化变量”的风险了。

​		在类里定义一个对象引用时，如果不将其初始化，那么引用就会被赋值为null。

### 指定初始化

​		怎么给一个变量赋初值呢？一种很直接的方法是在定义类成员变量的地方为其赋值。以下代码修改了InitialValues类成员变量的定义，直接提供了初值：

```java
// housekeeping/InitialValues2.java
// Providing explicit initial values

public class InitialValues2 {
    boolean bool = true;
    char ch = 'x';
    byte b = 47;
    short s = 0xff;
    int i = 999;
    long lng = 1;
    float f = 3.14f;
    double d = 3.14159;
}
```

​		你也可以用同样的方式初始化非基本类型的对象。如果Depth是一个类，那么可以像下面这样创建一个对象并初始化它：

```java
// housekeeping/Measurement.java

class Depth {}

public class Measurement {
    Depth d = new Depth();
    // ...
}

```

​		如果没有为d赋予初值就尝试使用它，就会出现运行时错误，告诉你产生了一个异常。

​		你也可以通过调用某个方法来提供初值：

```java
// housekeeping/MethodInit.java

public class MethodInit {
    int i = f();
    
    int f() {
        return 11;
    }
    
}

```

​		这个方法可以带有参数，但这参数不能是未初始化的类成员变量，因此，可以这么写：

```java
// housekeeping/MethodInit2.java

public class MethodInit2 {
    int i = f();
    int j = g(i);
    
    int f() {
        return 11;
    }
    
    int g(int n) {
        return n * 10;
    }
}
```

​		但你不能这么写：

```java
// housekeeping/MethodInit3.java

public class MethodInit3 {
    //- int j = g(i); // Illegal forward reference
    int i = f();

    int f() {
        return 11;
    }

    int g(int n) {
        return n * 10;
    }
}
```

​		显然，上述程序的正确性取决于初始化的顺序，而与其编译方式无关。所以，编译器恰当地对“向前引用”发出了警告。

​		这种初始化方式简单，但有个限制：类InitialValues的每个对象都有相同的初值，有时这的确是我们需要的，但有时却需要更大的灵活性。

## 构造器初始化

​		可以用构造器进行初始化，这种方式给了你更大的灵活性，因为你可以在运行时调用方法进行初始化。但是，这无法阻止自动初始化的进行，它会在构造器被调用之前发生。因此，如果使用如下代码：

```java
// housekeeping/Counter.java

public class Counter {
    int i;
    
    Counter() {
        i = 7;
    }
    // ...
}
```

​		i首先会被初始化为0，然后变为7。对于所有的基本类型和引用，包括在定义时已明确指定初值的变量，这种情况都是成立的。因此，编译器不会强制你一定要在构造器的某个地方或在使用它们之前初始化元素——初始化早已得到了保证。

### 初始化顺序

​		在类中变量定义的顺序决定了它们初始化的顺序。即变量定义散布在方法定义之间，它们仍会在任何方法(包括构造器)被调用之前得到初始化：

```java
// housekeeping/OrderOfInitialization.java
// Demonstrates initialization order
// When the constructor is called to create a
// Window object, you'll see a message:

class Window {
    Window(int marker) {
        System.out.println("Window(" + marker + ")");
    }
}

class House {
    Window w1 = new Window(1); // Before constructor

    House() {
        // Show that we're in the constructor:
        System.out.println("House()");
        w3 = new Window(33); // Reinitialize w3
    }

    Window w2 = new Window(2); // After constructor

    void f() {
        System.out.println("f()");
    }

    Window w3 = new Window(3); // At end
}

public class OrderOfInitialization {
    public static void main(String[] args) {
        House h = new House();
        h.f(); // Shows that construction is done
    }
}
//Window(1)
//Window(2)
//Window(3)
//House()
//Window(33)
//f()
```

​		在House类中，故意把几个Window对象定义散布各处，以证明它们全都会在调用构造器或其他方法之前得到初始化。此外，w3在构造器中被再次赋值。

​		由输出可见，引用w3被初始化了两次：一次在调用构造器前，一次在构造器调用期间(第一次引用的对象将被丢弃，并作为垃圾回收)。这咋一看可能觉得效率不高，但保证了正确的初始化。试想，如果定义了一个重载构造器，在其中没有初始化w3，同时在定义w3时没有赋予初值，那么会产生怎样的后果呢？

### 静态数据的初始化

​		无论创建多少个对象，静态数据都只占一份存储区域。static关键字不能应用于局部变量，所以只能作用于(字段、域)。如果一个字段是静态的基本类型，你没有初始化它，那么它就会获得基本类型的标准初值。如果它是对象引用，那么它的默认值就是null。

​		如果在定义时进行初始化，那么静态变量看起来就跟非静态变量一样。

​		下面例子显示了静态存储区是何时初始化的：

```java
// housekeeping/StaticInitialization.java
// Specifying initial values in a class definition

class Bowl {
    Bowl(int marker) {
        System.out.println("Bowl(" + marker + ")");
    }
    
    void f1(int marker) {
        System.out.println("f1(" + marker + ")");
    }
}

class Table {
    static Bowl bowl1 = new Bowl(1);
    
    Table() {
        System.out.println("Table()");
        bowl2.f1(1);
    }
    
    void f2(int marker) {
        System.out.println("f2(" + marker + ")");
    }
    
    static Bowl bowl2 = new Bowl(2);
}

class Cupboard {
    Bowl bowl3 = new Bowl(3);
    static Bowl bowl4 = new Bowl(4);
    
    Cupboard() {
        System.out.println("Cupboard()");
        bowl4.f1(2);
    }
    
    void f3(int marker) {
        System.out.println("f3(" + marker + ")");
    }
    
    static Bowl bowl5 = new Bowl(5);
}

public class StaticInitialization {
    public static void main(String[] args) {
        System.out.println("main creating new Cupboard()");
        new Cupboard();
        System.out.println("main creating new Cupboard()");
        new Cupboard();
        table.f2(1);
        cupboard.f3(1);
    }
    
    static Table table = new Table();
    static Cupboard cupboard = new Cupboard();
}
//Bowl(1)
//Bowl(2)
//Table()
//f1(1)
//Bowl(4)
//Bowl(5)
//Bowl(3)
//Cupboard()
//f1(2)
//main creating new Cupboard()
//Bowl(3)
//Cupboard()
//f1(2)
//main creating new Cupboard()
//Bowl(3)
//Cupboard()
//f1(2)
//f2(1)
//f3(1)
```

​		Bowl类展示类的创建，而Table和Cupboard在它们的类定义中包含Bowl类型的静态数据成员。注意，在静态数据成员定义之前，Cupboard类中先定义了一个Bowl类型的非静态成员b3。

​		由输出可见，静态初始化只有在必要时刻才会进行。如果不创建Table对象，也不引用Table.bowl1和Table.bowl2，那么静态的Bowl类对象bowl1和bowl2永远不会被创建。只有在第一个Table对象被创建时，它们才会被初始化。此后，静态对象不会再次被初始化。

​		初始化的顺序先是静态对象，然后是非静态对象，从输出可以看出。要执行main()方法，必须加载StaticInitialization类，它的静态属性table和cupboard随后被初始化，这会导致它们对应的类也被加载，而由于它们都包含静态的Bowl对象，所以Bowl类也会被加载。因此，在这个特殊的程序中，所有类都会在main()方法之前被加载。实际情况通常并非如此，因为在典型的程序中，不会像本例中所示的那样，将所有事物通过static联系起来。

​		概括一下创建对象的过程，假设有个名为Dog的类：

* 即使没有显示地使用static关键字，构造器实际上也是静态方法。所以，当首次创建Dog类型的对象或首次访问Dog类的静态方法或属性时，Java解释器必须在类路径中查找，以定位Dog.class。
* 当加载完Dog.class后，有关静态初始化的所有动作都会执行。因此，静态初始化只会在首次加载Class对象时初始化一次。
* 当用new Dog()创建对象时，首先会在堆上为Dog对象分配足够的存储空间。
* 分配的存储空间首先会被清零，即会将Dog对象中的所有基本数据设置为默认值，引用被置为null。
* 执行所有出现在字段定义处的初始化动作。
* 执行构造器。

### 显示的静态初始化

​		你可以将一组静态初始化动作放在类里面的一个特殊的“静态子句”(有时叫做静态块)中：

```java
// housekeeping/Spoon.java

public class Spoon {
    static int i;
    
    static {
        i = 47;
    }
}
```

​		这看起来像个方法，但实际上它只是一段跟在static关键字后面的代码块。与其他静态初始化一样，这段代码仅执行一次：当首次创建这个类的对象或首次访问这个类的静态成员(甚至不需要创建该类的对象)时：

```java
// housekeeping/ExplicitStatic.java
// Explicit static initialization with "static" clause

class Cup {
    Cup(int marker) {
        System.out.println("Cup(" + marker + ")");
    }
    
    void f(int marker) {
        System.out.println("f(" + marker + ")");
    }
}

class Cups {
    static Cup cup1;
    static Cup cup2;
    
    static {
        cup1 = new Cup(1);
        cup2 = new Cup(2);
    }
    
    Cups() {
        System.out.println("Cups()");
    }
}

public class ExplicitStatic {
    public static void main(String[] args) {
        System.out.println("Inside main()");
        Cups.cup1.f(99); // [1]
    }
    
    // static Cups cups1 = new Cups(); // [2]
    // static Cups cups2 = new Cups(); // [2]
}
//Inside main()
//Cup(1)
//Cup(2)
//f(99)
```

### 非静态实例初始化

​		Java提供了被称为实例初始化的类似语法，用来初始化每个对象的非静态变量：

```java
// housekeeping/Mugs.java
// Instance initialization

class Mug {
    Mug(int marker) {
        System.out.println("Mug(" + marker + ")");
    }
}

public class Mugs {
    Mug mug1;
    Mug mug2;
    { // [1]
        mug1 = new Mug(1);
        mug2 = new Mug(2);
        System.out.println("mug1 & mug2 initialized");
    }
    
    Mugs() {
        System.out.println("Mugs()");
    }
    
    Mugs(int i) {
        System.out.println("Mugs(int)");
    }
    
    public static void main(String[] args) {
        System.out.println("Inside main()");
        new Mugs();
        System.out.println("new Mugs() completed");
        new Mugs(1);
        System.out.println("new Mugs(1) completed");
    }
}
//Inside main()
//Mug(1)
//Mug(2)
//mug1 & mug2 initialized
//Mugs()
//new Mugs() completed
//Mug(1)
//Mug(2)
//mug1 & mug2 initialized
//Mugs(int)
//new Mugs(1) completed
```

​		看起来它很像静态代码块，只不过少了static关键字。这种语法对于支持“匿名内部类”的初始化是必须的，但是你也可以使用它保证某些操作一定会发生，而不管哪个构造器被调用。从输出看出，实例初始化子句是在两个构造器之前执行的。

## 数组初始化

​		数组是相同类型的、用一个标识符名称封装到一起的一个对象序列或基本数据类型。数组是通过方括号操作符[]来定义和使用的。要定义一个数组引用，只需要在类型名加上方括号：

```java
int[] a1;
```

​		方括号也可以放在标识符的后面，两者含义是一样的：

```java
int a1[];
```

​		编译器不允许指定数组的大小。这又把我们带回有关“引用”的问题上。你所拥有的只是对数组的一个引用，但是还没有给数组对象本身分配任何空间。为了给数组创建相应的存储空间，必须写初始化表达式，它必须在创建数组的地方出现。这种特殊的初始化是由一对花括号括起来的值组成。这种情况下，存储空间的分配将由编译器负责：

```java
int[] a1 = {1, 2, 3, 4, 5};
```

​		那么为什么在还没有数组的时候定义一个数组引用呢？

```java
int[] a2;
```

​		在Java中可以将一个数组赋值给另一个数组，所以可以这样：

```java
a2 = a1;
```

​		其实真正做的只是复制了一个引用，就像下面演示的这样：

```java
// housekeeping/ArraysOfPrimitives.java

public class ArraysOfPrimitives {
    public static void main(String[] args) {
        int[] a1 = {1, 2, 3, 4, 5};
        int[] a2;
        a2 = a1;
        for (int i = 0; i < a2.length; i++) {
            a2[i] += 1;
        }
        for (int i = 0; i < a1.length; i++) {
            System.out.println("a1[" + i + "] = " + a1[i]);
        }
    }
}
//a1[0] = 2;
//a1[1] = 3;
//a1[2] = 4;
//a1[3] = 5;
//a1[4] = 6;
```

​		a1初始化了，但是a2没有；这里a2在后面被赋给另一个数组。由于a1和a2是相同数组的别名，因此通过a2所做的修改在a1中也能看到。

​		所有的数组都有一个固定成员length，告诉你这个数组有多少个元素，你不能对其修改。Java数组计数也是从0开始的，所能使用的最大下标数是length-1。在你访问超出这个边界时，会报运行时错误(异常)，从而避免此类问题。

### 动态数组创建

​		若果在编写程序时，不确定数组中需要多少个元素，可以使用new在数组中创建元素。如下例所示，使用new创建基本类型数组。new不能创建非数组以外的基本类型数据：

```java
// housekeeping/ArrayNew.java
// Creating arrays with new
import java.util.*;

public class ArrayNew {
    public static void main(String[] args) {
        int[] a;
        Random rand = new Random(47);
        a = new int[rand.nextInt(20)];
        System.out.println("length of a = " + a.length);
        System.out.println(Arrays.toString(a));
    } 
}
//length of a = 18
//[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```

​		数组的大小是通过Random.nextInt()随机确定的，这个方法会返回0到输入参数之间的一个值。由于随机性，很明显数组的创建确实是在运行时进行的。此外，程序输出表名，数组元素中的基本数据类型会自动初始化为默认值(对于数字和字符是0，对于布尔类型是false)。Arrays.toString()是java.util标准库中的方法，会产生一维数组的可打印版本。

​		本例中，数组也可以在定义的同时进行初始化：

```java
int[] a = new int[rand.nextInt(20)];
```

​		如果可能的话，应该尽量这么做。如果你创建了一个非基本类型的数组，那么你创建的是一个引用数组。以整数的包装类型Integer为例，它是一个类而非基本类：

```java
// housekeeping/ArrayClassObj.java
// Creating an array of nonprimitive objects

import java.util.*;

public class ArrayClassObj {
    public static void main(String[] args) {
        Random rand = new Random(47);
        Integer[] a = new Integer[rand.nextInt(20)];
        System.out.println("length of a = " + a.length);
        for (int i = 0; i < a.length; i++) {
            a[i] = rand.nextInt(500); // Autoboxing
        }
        System.out.println(Arrays.toString(a));
    }
}
length of a = 18
[55, 193, 361, 461, 429, 368, 200, 22, 207, 288, 128, 51, 89, 309, 278, 498, 361, 20]
```

​		这里，即使使用new创建之后：

```java
Integer[] a = new Integer[rand.nextInt(20)];    
```

​		它只是一个引用数组，直到通过创建新的Integer对象，并把对象赋值给引用，初始化才算结束：

```java
a[i] = rand.nextInt(500);
```

​		如果忘记了创建对象，但试图使用数组中的空引用，就会在运行时产生异常。

​		也可以用花括号括起来的列表来初始化数组，有两种形式：

```java
// housekeeping/ArrayInit.java
// Array initialization
import java.util.*;

public class ArrayInit {
    public static void main(String[] args) {
        Integer[] a = {
                1, 2,
                3, // Autoboxing
        };
        Integer[] b = new Integer[] {
                1, 2,
                3, // Autoboxing
        };
        System.out.println(Arrays.toString(a));
        System.out.println(Arrays.toString(b));

    }
}
[1, 2, 3]
[1, 2, 3]
```

​		在这两种形式，初始化列表的最后一个逗号是可选的。

​		尽管第一种形式很有用，但是它更加受限，因为它只能用于数组定义处。第二种形式可以用在任何地方，甚至用在方法的内部。例如，你创建了一个String数组，将其传递给另一个类的main()方法，如下：

```java
// housekeeping/DynamicArray.java
// Array initialization

public class DynamicArray {
    public static void main(String[] args) {
        Other.main(new String[] {"fiddle", "de", "dum"});
    }
}

class Other {
    public static void main(String[] args) {
        for (String s: args) {
            System.out.print(s + " ");
        }
    }
}
//fiddle de dum 
```

​		Other.main()的参数是在调用处创建的，因此你甚至可以在方法的调用处提供可替换的参数。

### 可变参数列表

​		你可以以一种类似C语言中的可变参数列表来创建和调用方法。这可以应用在参数个数或类型未知的场合。由于所有的类都最后继承于Object类，所以你可以创建一个以Object数组为参数的方法：

```java
// housekeeping/VarArgs.java
// Using array syntax to create variable argument lists

class A {}

public class VarArgs {
    static void printArray(Object[] args) {
        for (Object obj: args) {
            System.out.print(obj + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        printArray(new Object[] {47, (float) 3.14, 11.11});
        printArray(new Object[] {"one", "two", "three"});
        printArray(new Object[] {new A(), new A(), new A()});
    }
}
//47 3.14 11.11 
//one two three 
//A@15db9742 A@6d06d69c A@7852e922
```

​		printArray()参数是Object数组，使用for-in语法遍历和打印数组的每一项。标准Java库能输出有意义的内容，但这里创建的是类的对象，打印出的内容是类名，后面跟着一个@符号以及多个十六进制数字。因而，默认行为(如果没有定义toString()方法的话)就是打印类名和对象的地址。

```java
// housekeeping/NewVarArgs.java
// Using array syntax to create variable argument lists

public class NewVarArgs {
    static void printArray(Object... args) {
        for (Object obj: args) {
            System.out.print(obj + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        // Can take individual elements:
        printArray(47, (float) 3.14, 11.11);
        printArray(47, 3.14F, 11.11);
        printArray("one", "two", "three");
        printArray(new A(), new A(), new A());
        // Or an array:
        printArray((Object[]) new Integer[] {1, 2, 3, 4});
        printArray(); // Empty list is OK
    }
}
//47 3.14 11.11 
//47 3.14 11.11 
//one two three 
//A@15db9742 A@6d06d69c A@7852e922 
//1 2 3 4 
```

​		有了可变参数，你就再也不用显式地编写数组语法了，当你指定参数时，编译器实际上会为你填充数组。你获取的仍然是一个数组，这就是为什么printArray()可以使用for-in迭代数组的原因。但是，这不仅仅只是从元素列表到数组的自动转换。注意程序的倒数第二行，一个Integer数组被转型为一个Object数组，并且传递给了printArray()。显然，编译器会发现这是一个数组，不会执行转换。因此，如果你有一组事物，可以把它们当作列表传递，而如果你已经有了一个数组，该方法会把它们当作可变参数列表来接受。

​		程序的最后一行表名，可变参数的个数可以为0。当具有可选的尾随参数，这一特性会有帮助：

```java
// housekeeping/OptionalTrailingArguments.java

public class OptionalTrailingArguments {
    static void f(int required, String... trailing) {
        System.out.print("required: " + required + " ");
        for (String s: trailing) {
            System.out.print(s + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        f(1, "one");
        f(2, "two", "three");
        f(0);
    }
}
//required: 1 one 
//required: 2 two three 
//required: 0 
```

​		这段程序展示了如何使用Object类之外类型的可变参数列表。这里，所有的可变参数都是String对象。可变参数列表中可以使用任何类型的参数，包括基本类型。下面例子展示了可变参数列表为数组的情形，并且如果列表中没有任何元素，那么转变大小为0的数组：

```java
// housekeeping/VarargType.java

public class VarargType {
    static void f(Character... args) {
        System.out.print(args.getClass());
        System.out.println(" length " + args.length);
    }
    
    static void g(int... args) {
        System.out.print(args.getClass());
        System.out.println(" length " + args.length)
    }
    
    public static void main(String[] args) {
        f('a');
        f();
        g(1);
        g();
        System.out.println("int[]: "+ new int[0].getClass());
    }
}
//class [Ljava.lang.Character; length 1
//class [Ljava.lang.Character; length 0
//class [I length 1
//class [I length 0
//int[]: class [I
```

​		getClass()方法属于Object类。它会产生对象的类，并在打印该类时，看到表示该类类型的编码字符串。前导[代表这是一个后面紧随的类型的数组，I表示基本类型int；为了进行双重检查，我在最后一行创建了一个int数组，打印了其类型。这样也验证了使用可变参数列表不依赖于自动装箱，而使用的是基本类型。

​		然而，可变参数列表与自动装箱可以和谐共处：

```java
// housekeeping/AutoboxingVarargs.java

public class AutoboxingVarargs {
    public static void f(Integer... args) {
        for (Integer i: args) {
            System.out.print(i + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        f(1, 2);
        f(4, 5, 6, 7, 8, 9);
        f(10, 11, 12);
        
    }
}
//1 2
//4 5 6 7 8 9
//10 11 12
```

​		注意吗，你可以在单个参数列表中将类型混合在一起，自动装箱机制会有选择地把int类型提升为Integer。

​		可变参数列表使得方法重载更加复杂了，尽管乍看之下似乎足够安全：

```java
// housekeeping/OverloadingVarargs.java

public class OverloadingVarargs {
    static void f(Character... args) {
        System.out.print("first");
        for (Character c: args) {
            System.out.print(" " + c);
        }
        System.out.println();
    }
    
    static void f(Integer... args) {
        System.out.print("second");
        for (Integer i: args) {
            System.out.print(" " + i);
        }
        System.out.println();
    }
    
    static void f(Long... args) {
        System.out.println("third");
    }
    
    public static void main(String[] args) {
        f('a', 'b', 'c');
        f(1);
        f(2, 1);
        f(0);
        f(0L);
        //- f(); // Won's compile -- ambiguous
    }
}
//first a b c
//second 1
//second 2 1
//second 0
//third
```

​		在每种情况下，编译器都会使用自动装箱来匹配重载的方法，然后调用最明确的方法。

​		但是如果调用不含参数的f()，编译器就无法直到应该调用哪个方法了。尽管这个错误可以弄清楚，但是它可能会使客户端程序员感到意外。

​		你可能会通过某个方法中增加一个非可变参数解决这个问题：

```java
// housekeeping/OverloadingVarargs2.java
// {WillNotCompile}

public class OverloadingVarargs2 {
    static void f(float i, Character... args) {
        System.out.println("first");
    }
    
    static void f(Character... args) {
        System.out.println("second");
    }
    
    public static void main(String[] args) {
        f(1, 'a');
        f('a', 'b');
    }
}

```

```java
// housekeeping/OverloadingVarargs3

public class OverloadingVarargs3 {
    static void f(float i, Character... args) {
        System.out.println("first");
    }
    
    static void f(char c, Character... args) {
        System.out.println("second");
    }
    
    public static void main(String[] args) {
        f(1, 'a');
        f('a', 'b');
    }
}
//first
//second
```

​		你应该总在重载方法的一个版本上使用可变参数列表，或者压根不用它。

## 枚举类型

​		Java5中添加了一个看似很小的特性enum关键字，它使得我们在需要群组并使用枚举类型集时，可以很方便地处理。

```java
// housekeeping/Spiciness.java

public enum Spiciness {
    NOT, MILD, MEDIUM, HOT, FLAMING
}
```

​		这里创建了一个名为Spiciness的枚举类型，它有5个值。由于枚举类型的实例是常量，因此按照命名惯例，它们都用大写字母表示。要使用enum，需要创建一个该类型的引用，然后将其赋值给某个实例：

```java
// housekeeping/SimpleEnumUse.java

public class SimpleEnumUse {
    public static void main(String[] args) {
        Spiciness howHot = Spiciness.MEDIUM;
        System.out.println(howHot);
    }
}
//MEDIUM
```

​		在你创建enum时，编译器会自己添加一些有用的特性。例如，它会创建toString()方法，以便你方便地显示某个enum实例的名称，这从上面例子中的输出可以看出。编译器还会创建ordonal()方法表示某个特定enum常量的声明顺序，static values()方法按照enum常量的声明顺序，生成这些常量值构成的数组：

```java
// housekeeping/EnumOrder.java

public class EnumOrder {
    public static void main(String[] args) {
        for (Spiciness s: Spiciness.values()) {
            System.out.println(s + ", ordinal " + s.ordinal());
        }
    }
}
//NOT, ordinal 0
//MILD, ordinal 1
//MEDIUM, ordinal 2
//HOT, ordinal 3
//FLAMING, ordinal 4
```

​		尽管enum看起来是一种新的数据类型，但是这个关键字只是在生成enum的类时，产生了某些编译行为，因此在很大程度上你可以将enum当作其他任何类。事实上，enum确实是类，并且具有自己的方法。

​		enum有一个很实用的特性，就是在switch语句中使用：

```java
// housekeeping/Burrito.java

public class Burrito {
    Spiciness degree;
    
    public Burrito(Spiciness degree) {
        this.degree = degree;
    }
    
    public void describe() {
        System.out.print("This burrito is ");
        switch(degree) {
            case NOT:
                System.out.println("not spicy at all.");
                break;
            case MILD:
            case MEDIUM:
                System.out.println("a little hot.");
                break;
            case HOT:
            case FLAMING:
            default:
                System.out.println("maybe too hot");
        }
    }
    
    public static void main(String[] args) {
        Burrito plain = new Burrito(Spiciness.NOT),
        greenChile = new Burrito(Spiciness.MEDIUM),
        jalapeno = new Burrito(Spiciness.HOT);
        plain.describe();
        greenChile.describe();
        jalapeno.describe();
    }
}
//This burrito is not spicy at all.
//This burrito is a little hot.
//This burrito is maybe too hot.
```

​		由于switch是在有限的可能值集合中选择，因此enum是绝佳的组合。注意，enum的名称是如何能够倍加清楚地表名程序的目的的。

​		通常，你可以将enum用作另一种创建数据类型的方式，然后使用所得到的类型。这正是关键所在，所以你不用过多地考虑它们。在enum被引入之前，你必须花费大量的精力去创建一个等同的枚举类型，并是安全可用的。