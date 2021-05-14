---
title: 阅读《On Java 8》-- 第十一章 内部类
date: 2021-05-14 15:21:54
tags:
- JAVASE
- 《On Java 8》
categories: JAVASE
mathjax: true
---

一个定义在另一个类中的类，叫作内部类。

​		内部类是一种非常有用的特性，因为它允许你把一些逻辑相关的类组织在一起，并控制位于内部的类的可见性。然而必须要了解，内部类与组合是完全不同的概念，这一点很重要。在最初，内部类看起来就像是一种代码隐藏机制：将类置于其他类的内部。但是，你将会了解到，内部类远不止如此，它了解外部类，并能与之通信，而且你用内部类写出的代码更加优雅而清晰，尽管并不总是这样(而且Java8的Lambda表达式和方法引用减少了编写内部类的需求)。

​		最初，内部类可能看起来有些奇怪，而且要花些时间才能在设计中轻松地使用它们。对内部类的需求并非总是很明显的，但是在描述完内部类的基本语法与语义之后，就能明白使用内部类的好处了。

 <!-- more --> 

## 创建内部类

​		创建内部类的方式就如同你想的一样——把类的定义置于外部类的里面：

```java
// innerclasses/Parcel1.java
// Creating inner classes
public class Parcel1 {
    class Contents {
        private int i = 11;
      
        public int value() { return i; }
    }
  
    class Destination {
        private String label;
      
        Destination(String whereTo) {
            label = whereTo;
        }
      
        String readLabel() { return label; }
    }
    // Using inner classes looks just like
    // using any other class, within Parcel1:
    public void ship(String dest) {
        Contents c = new Contents();
        Destination d = new Destination(dest);
        System.out.println(d.readLabel());
    }
  
    public static void main(String[] args) {
        Parcel1 p = new Parcel1();
        p.ship("Tasmania");
    }
}
//Tasmania
```

​		当我们在ship()方法里面使用内部类的时候，与使用普通类没什么不同。在这里，明显的区别只是内部类的名字是嵌套在Parcel1里面的。

​		更典型的情况是，外部类将有一个方法，该方法返回了一个指向内部类的引用，就像to()和contents()方法中看到的那样：

```java
// innerclasses/Parcel2.java
// Returning a reference to an inner class
public class Parcel2 {
    class Contents {
        private int i = 11;
      
        public int value() { return i; }
    }
  
    class Destination {
        private String label;
      
        Destination(String whereTo) {
            label = whereTo;
        }
      
        String readLabel() { return label; }
    }
  
    public Destination to(String s) {
        return new Destination(s);
    }
  
    public Contents contents() {
        return new Contents();
    }
  
    public void ship(String dest) {
        Contents c = contents();
        Destination d = to(dest);
        System.out.println(d.readLabel());
    }
  
    public static void main(String[] args) {
        Parcel2 p = new Parcel2();
        p.ship("Tasmania");
        Parcel2 q = new Parcel2();
        // Defining references to inner classes:
        Parcel2.Contents c = q.contents();
        Parcel2.Destination d = q.to("Borneo");
    }
}
//Tasmania
```

​		如果想从外部类的非静态方法之外的任意位置创建某个内部类的对象，那么必须像在main()方法中那样，具体地指明这个对象的类型：OuterClassName.InnerClassName。在外部类的静态方法中也可以直接指明类型InnerClassName，在其他类中需要指明OuterClassName.InnerClassName。

## 链接外部类

​		到目前为止，内部类似乎还只是一种名字隐藏和组织代码的模式。这些是很有用的，但还不是最引人注目的，它还有其他的用途。当生成一个内部类的对象时，此对象与制造它的外部对象(enclosing object)之间就有了一种联系，所以它能访问其他外部对象的所有成员，而不需要任何特殊条件。此外，内部类还拥有外部类的所有元素的访问权。

```java
// innerclasses/Sequence.java
// Holds a sequence of Objects
interface Selector {
    boolean end();
    Object current();
    void next();
}
public class Sequence {
    private Object[] items;
    private int next = 0;
    public Sequence(int size) {
        items = new Object[size];
    }
    public void add(Object x) {
        if(next < items.length)
            items[next++] = x;
    }
    private class SequenceSelector implements Selector {
        private int i = 0;
        @Override
        public boolean end() { return i == items.length; }
        @Override
        public Object current() { return items[i]; }
        @Override
        public void next() { if(i < items.length) i++; }
    }
    public Selector selector() {
        return new SequenceSelector();
    }
    public static void main(String[] args) {
        Sequence sequence = new Sequence(10);
        for(int i = 0; i < 10; i++)
            sequence.add(Integer.toString(i));
        Selector selector = sequence.selector();
        while(!selector.end()) {
            System.out.print(selector.current() + " ");
            selector.next();
        }
    }
}
//0 1 2 3 4 5 6 7 8 9
```

​		Sequence类只是一个固定大小的Object的数组，以类的形式包装了起来。可以调用add()在序列末尾增加新的Object(只要还有空间)，要获取Sequence中的每一个对象，可以使用Selector接口。这是“迭代器”设计模式的一个例子。Selector允许你检查序列是否到末尾了(end())，访问当前对象(current())，以及移到序列中的下一个对象(next())。因为Selector是一个接口，所以别的类可以按它们自己的方式来实现这个接口，并且其他方法能以此接口为参数，来生成更加通用的代码。

​		这里，SequenceSelector是提供Selector功能的private类。可以看到，在main()中创建一个Sequence，并向其中添加了一些String对象。然后通过调用selector()获取一个Selector，并用它在Sequence中移动和选择每一个元素。最初看到SequenceSelector，可能觉得它只是一个内部类罢了。但请仔细观察它，注意方法end()，current()和next()都用到了items，这是一个引用，它并不是SequenceSelector的一部分，而是外部类的一个private字段。然而内部类可以访问其他外部类的方法和字段，就像自己拥有它们似的，这带来了很大的方便，就如前面的例子所示。

​		所以内部类自动拥有对其外部类所有成员的访问权。这是如何做到的呢？当某个外部类的对象创建了一个内部类对象时，此内部类对象必定会秘密地捕获一个指向那个外部类对象的引用。然后，在你访问此外部类的成员时，就用那个引用来选择外部类的成员。幸运的是，编译器会帮你处理所有的细节，但你现在可以看到：内部类的对象只能在与其外部类的对象相关联的情况下才能被创建(就像你应该看到的，内部类是非static类时)。构建内部类对象时，需要一个指向其外部类对象的引用，如果编译器访问不到这个引用就会报错。不过绝大多数时候这都无需程序员操心。

## 使用.this和.new

​		如果你需要生成对外部类对象的引用，可以使用外部类的名字后面紧跟圆点和this。这样产生的引用自动地具有正确的类型，这一点在编译期就知晓并受到检查，因此没有任何运行时开销。下面的示例展示了如何使用.this：

```java
// innerclasses/DotThis.java
// Accessing the outer-class object
public class DotThis {
    void f() { System.out.println("DotThis.f()"); }
  
    public class Inner {
        public DotThis outer() {
            return DotThis.this;
            // A plain "this" would be Inner's "this"
        }
    }
  
    public Inner inner() { return new Inner(); }
  
    public static void main(String[] args) {
        DotThis dt = new DotThis();
        DotThis.Inner dti = dt.inner();
        dti.outer().f();
    }
}
//DotThis.f()
```

​		有时你可能想要告知某些其他对象，去创建某个内部类的对象。要实现此目的，你必须在new表达式中提供对其他外部类对象的引用，这是需要使用.new语法，就像下面这样：

```java
// innerclasses/DotNew.java
// Creating an inner class directly using .new syntax
public class DotNew {
    public class Inner {}
    public static void main(String[] args) {
        DotNew dn = new DotNew();
        DotNew.Inner dni = dn.new Inner();
    }
}
```

​		想要直接创建内部类的对象，你不能按照你想象的方式，去引用外部类的名字DotNew，而是必须使用外部类的对象来创建该内部类对象，就像在上面的程序中所看到的那样。这也解决了内部类名字作用域的问题，因此你不必声明(实际上你不能声明)dn.newDotNew.Inner。

​		下面你可以看到将.new应用于Parcel的示例：

```java
// innerclasses/Parcel3.java
// Using .new to create instances of inner classes
public class Parcel3 {
    class Contents {
        private int i = 11;
        public int value() { return i; }
    }
    class Destination {
        private String label;
        Destination(String whereTo) { label = whereTo; }
        String readLabel() { return label; }
    }
    public static void main(String[] args) {
        Parcel3 p = new Parcel3();
        // Must use instance of outer class
        // to create an instance of the inner class:
        Parcel3.Contents c = p.new Contents();
        Parcel3.Destination d =
                p.new Destination("Tasmania");
    }
}
```

​		在拥有外部类对象之前是不可能创建内部类对象的。这是因为内部类对象会暗暗地连接到建它的外部类对象上。但是，如果你创建的是嵌套类(静态内部类)，那么它就不需要对外部对象的引用。

## 内部类与向上转型

​		当将内部类向上转型为其基类，尤其是转型为一个接口的时候，内部类就有了用武之地。(从实现了某个接口的对象，得到对此接口的引用，与向上转型为这个对象的基类，实质上效果是一样的。)这是因为此内部类-某个接口的实现-能够完全不可见，并且不可用。所得到的只是指向基类或接口的引用，所以能很方便地隐藏实现细节。

​		我们可以创建前一个示例的接口：

```java
// innerclasses/Destination.java
public interface Destination {
    String readLabel();
}
```

```java
// innerclasses/Contents.java
public interface Contents {
    int value();
}
```

​		现在Contents和Destination表示客户端程序员可用的接口。记住，接口的所有成员自动被设置为public。

​		当取得了一个指向基类或接口的引用，甚至可能无法找出它确切的类型，看下面的例子：

```java
// innerclasses/TestParcel.java
class Parcel4 {
    private class PContents implements Contents {
        private int i = 11;
        @Override
        public int value() { return i; }
    }
    protected final class PDestination implements Destination {
        private String label;
        private PDestination(String whereTo) {
            label = whereTo;
        }
        @Override
        public String readLabel() { return label; }
    }
    public Destination destination(String s) {
        return new PDestination(s);
    }
    public Contents contents() {
        return new PContents();
    }
}
public class TestParcel {
    public static void main(String[] args) {
        Parcel4 p = new Parcel4();
        Contents c = p.contents();
        Destination d = p.destination("Tasmania");
        // Illegal -- can't access private class:
        //- Parcel4.PContents pc = p.new PContents();
    }
}
```

​		在Parcel4中，内部类PContents是private，所以除了Parcel4，没有人能访问它。普通(非内部)类的访问权限不能被设为private或者protected；它们只能设置为public或package访问权限。

​		PDestination是protected，所以只有Parcel4及其子类、还有与Parcel4同一个包中的类能访问PDestination，其它类都不能访问PDestination，这意味着，如果客户端程序员想了解或访问这些成员，那是要受到限制的。实际上，甚至不能向下转型成private内部类，因为不能访问其名字，就像在TestParcel类中看到的那样。

​		private内部类给类的设计者提供了一种途径，通过这种方式可以完全阻止任何依赖于类型的编码，完全隐藏了实现的细节。此外，从客户端程序员的角度来看，由于不能访问任何新增的、原本不属于公共接口的方法，所以扩展接口是没有价值的。这也给Java编译器提供了生成高效代码的机会。

## 内部类方法和作用域

​		到目前为止，读者所看到的只是内部类的典型用途。通常，如果所读、写的代码包含了内部类，那么它们都是“平凡的”内部类。简单并且容易理解。然而，内部类的语法重写了大量其他的更加难以理解的技术。例如，可以在一个方法里面或者在任意的作用域内定义内部类。

​		这么做有两个理由：

* 1.如前所示，你实现了某些类型的接口，于是可以创建并返回对其的引用。
* 2.你要解决一个复杂的问题，想创建一个类来辅助你的解决方案，但是又不希望这个类是公共可用的。



​		在后面的例子中，先前的代码将被修改，以用来实现：

* 一个定义在方法中的类。
* 一个定义在作用域内的类，此作用域在方法的内部。
* 一个实现了接口的匿名类。
* 一个匿名类，它扩展了没有默认构造器的类。
* 一个匿名类，它执行字段初始化。
* 一个匿名类，它通过实例初始化实现构造(匿名内部类不可能有构造器)。



​		第一个例子展示了在方法的作用域(而不是在其他类的作用域内)创建一个完整的类。这被称作局部内部类：

```java
// innerclasses/Parcel5.java
// Nesting a class within a method
public class Parcel5 {
    public Destination destination(String s) {
        final class PDestination implements Destination {
            private String label;
          
            private PDestination(String whereTo) {
                label = whereTo;
            }
          
            @Override
            public String readLabel() { return label; }
        }
        return new PDestination(s);
    }
  
    public static void main(String[] args) {
        Parcel5 p = new Parcel5();
        Destination d = p.destination("Tasmania");
    }
}
```

​		PDestination类是destination()方法的一部分，而不是Parcel5的一部分。所以，在destination()之外不能访问PDestination，注意出现在return语句中的向上转型-返回的是Destination的引用，它是PDestination的基类。当然，在destination()中定义了内部类PDestination，并不意味着一旦destination()方法执行完毕，PDestination就不可用了。

​		你可以在同一个子目录下的任意类中对某个内部类使用类标识符PDestination，这并不会有命名冲突。

​		下面的例子展示了如何在任意的作用域内嵌入一个内部类：

```java
// innerclasses/Parcel6.java
// Nesting a class within a scope
public class Parcel6 {
    private void internalTracking(boolean b) {
        if(b) {
            class TrackingSlip {
                private String id;
                TrackingSlip(String s) {
                    id = s;
                }
                String getSlip() { return id; }
            }
            TrackingSlip ts = new TrackingSlip("slip");
            String s = ts.getSlip();
        }
        // Can't use it here! Out of scope:
        //- TrackingSlip ts = new TrackingSlip("x");
    }
    public void track() { internalTracking(true); }
    public static void main(String[] args) {
        Parcel6 p = new Parcel6();
        p.track();
    }
}

```

​		TrackingSlip类被嵌入在if语句的作用域内，这并不是说该类的创建是有条件的，它其实与别的类一起编译过了。然而，在定义TrackingSlip的作用域之外，它是不可用的，除此之外，它与普通的类一样。