---
title: 阅读《On-Java-8》-第七章-封装
date: 2021-05-07 21:39:54
tags:
- JAVASE
- 《On Java 8》
categories: JAVASE
mathjax: true
---

​		Java提供了访问修饰符(access specifier)供类库开发者指明哪些对于客户端程序员是可用的，哪些是不可用的。访问控制权限的等级，从“最大权限”到“最小权限”依次是：public、protected，包访问权限(package access)(没有关键字)和private。

 <!-- more --> 

## 包的概念

​		包内包含一组类，它们被组织在一个单独的命名空间(namespace)下。

​		例如，标准Java发布中有一个工具库，它被组织在java.util命名空间下。java.util中含有一个类，叫做ArrayList。使用ArrayList的一种方式是用其全名java.util.ArrayList。

```java
// hiding/FullQualification.java

public class FullQualification {
    public static void main(String[] args) {
        java.util.ArrayList list = new java.util.ArrayList();
    }
}
```

​		这种方式使得程序冗长乏味，因此你可以换一种方式，使用import关键字。如果需要导入某个类，就需要在import语句中声明：

```java
// hiding/SingleImport.java
import java.util.ArrayList;

public class SingleImport {
    public static void main(String[] args) {
        ArrayList list = new ArrayList();
    }
}
```

​		现在你就可以不加限定词，直接使用ArrayList了。但是对于java.util包下的其他类，你还是不能用。要导入其中所有的类，只需使用\*：

```java
import java.util.*
```

​		之所以使用导入，是为了提供一种管理命名空间的机制。所有类名之间都是相互隔离的。类A中的方法f()不会与类B中具有相同签名的方法f()冲突。但是如果类名冲突呢？为了解决冲突，我们为每个类创建一个唯一标识符组合。

​		到目前为止的大部分示例都存在单个文件，并为本地使用的，所以尚未受到包名的干扰。但是，这些示例其实已经位于包中了，叫做“未命名”包或默认包(default package)。但是，如果你打算为相同机器上的其他Java程序创建友好的类库或程序时，就必须仔细考虑以防类名冲突。

​		一个Java源代码文件称为一个编译单元(compilation unit)(有时也称翻译单元(translation unit))。每个编译单元的文件后缀名必须是.java。在编译单元中可以有一个public类，它的类名必须与文件同名(包括大小写，但不包括后缀名.java)。每个编译单元中只能有一个public类，否则编译器不接受。如果这个编译单元中还有其他类，那么在包之外是无法访问到这些类的，因为它们不是public类，此时它们为主public类提供“支持”类。

### 代码组织

​		当编译一个.java文件时，.java文件的每个类都会有一个输出文件。每个输出文件的文件名和.java文件中每个类的类名相同，只是后缀名是.class。因此，在编译少量的.java文件后，会得到大量的.class文件。如果你使用过编译型语言，那么你可能习惯编译后产生的一个中间文件(通常称为“obj”文件)，然后与使用链接器(创建可执行文件)或类库生成器(创建类库)产生的其他同类文件打包到一起的情况。这不是Java工作的方式。在Java中，可运行程序是一组.class文件，它们可以打包压缩成一个Java文档文件(JAR，使用jar文档生成器)。Java解释器负责查找、加载和解释这些文件。

​		类库是一组类文件。每个源文件通常都含有一个public类和任意数量的非public类，因此每个文件都有一个public组件。如果把这些组件集中在一起，就需要使用关键字package。

​		如果你使用了package语句，它必须是文件中除了注释之外的第一行代码：

```java
package hiding;
```

​		意味着这个编译单元是一个名为hiding类库的一部分。换句话说，你正在生命的编译单元中的public类名称位于名为hiding的保护伞下。任何人想要使用该名称，必须指明完整的类名或者使用import关键字导入hiding。

​		例如，假设文件名是MyClass.java，这意味着文件中只能有一个public类，且类名必须是MyClass：

```java
// hiding/mypackage/MyClass.java
package hiding.mypackage;

public class MyClass {
    // ...
}
```

​		现在，如果有人想使用MyClass或hiding.mypackage中的其他public类，就必须使用关键字import来使hiding.mypackage中的名称可用。还有一种选择是使用完整的名称：

```java
// hiding/QualifiedMyClass.java

public class QualifiedMyClass {
    public static void main(String[] args) {
        hiding.mypackage.MyClass m = new hiding.mypackage.MyClass();
    }
}
```

​		关键字**import**使之更加简洁：

```java
// hiding/ImportedMyClass.java
import hiding.mypackage.*;

public class ImportedMyClass {
    public static void main(String[] args) {
        MyClass m = new MyClass();
    }
}    
```

​		package和import这两个关键字将单一的全局命名空间分隔开，从而避免名称冲突。

### 创建独一无二的包名

​		你可能注意到，一个包从未真正被打包成单一的文件，它可以由很多.class文件构成，因而事情就变得有点复杂了。为了避免这种情况，一种合乎逻辑的做法是将特定包下的所有.class文件都放在一个子目录下。也就是说，利用操作系统的文件结构的层次性。这是Java解决混乱问题的一种方式。

​		将所有的文件放在一个子目录还解决了其他的两个问题：创建独一无二的包名和查找可能隐藏于目录结构某处的类。这是通过将.class文件所在的路径位置编码成package名称来实现的。按照惯例，package名称是类的创建者的反顺序的Internet域名。如果你遵循惯例，因为Integer域名是独一无二的，所以你的package名称也应该是独一无二的，不会发生名称冲突。如果你没有自己的域名，你就得构造一组不大可能与他人重复的组合，来创建独一无二的package名称。

​		此技巧的第二部分是把package名称分解成你机器上的一个目录，所以当Java解释器必须要加载一个.class文件时，它能定位到.class文件所在的位置。首先，它找出环境变量CLASSPATH(通过操作系统设置，有时也能通过Java的安装程序或基于Java的工具设置)。CLASSPATH包含一个或多个目录，用作查找.class文件的根目录。从根目录开始，Java解释器获取包名并将每个句点替换成反斜杠，生成一个基于根目录的路径名(取决于你的操作系统，包名foo.bar.baz变成foo\bar\baz或foo/bar/baz或其它)。然后这个路径与CLASSPATH的不同项连接，解释器就在这些目录中查找与你所创建的类名称相关的.class文件。

​		为了理解这点，例如域名为：MindviewInc.com，将之反转并全部改为小写后就是com.mindviewinc，这将作为创建一个独一无二的全局名称。然后再创建一个名为simple的类库，从而细分名称：

```java
package com.mindviewinc.simple;
```

​		这个包名可以用作下面两个文件的命名空间保护伞：

```java
// com/mindviewinc/simple/Vector.java
// Creating a package
package com.mindviewinc.simple;

public class Vector {
    public Vector() {
        System.out.println("com.mindviewinc.simple.Vector");
    }
}
```

​		如前所述，package语句必须是文件的第一行非注释代码。第二个文件看上去差不多：

```java
// com/mindviewinc/simple/List.java
// Creating a package
package com.mindviewinc.simple;

public class List {
    System.out.println("com.mindview.simple.List");
}
```

​		这两个文件都位于我机器中的子目录下：

```tex
C:\DOC\Java\com\mindviewinc\simple
```

​		如果你回头看这个路径，会看到com.mindviewinc.simple，但是路径的第一部分呢？CLASSPATH环境变量会处理它。我的机器上的环境变量部分如下：

```tex
CLASSPATH=.;D:\JAVA\LIB;C:\DOC\Java
```

​		CLASSPATH可以包含多个不同的搜索路径。

​		但是在使用JAR文件时，有点不一样。你必须在类路径写清楚JAR文件的实际名称，不能仅仅是JAR文件所在的目录。因此，对于一个名为grape.jar的JAR文件，类路径应包括：

```tex
CLASSPATH=.;D\JAVA\LIB;C:\flavors\grape.jar
```

​		一旦设置好类路径，下面的文件就可以放在任意目录：

```java
// hiding/LibTest.java
// Uses the library
import com.mindviewinc.simple.*;

public class LibTest {
    public static void main(String[] args) {
        Vector v = new Vector();
        List l = new List();
    }
}
//com.mindviewinc.simple.Vector
//com.mindviewinc.simple.List
```

​		当编译器遇到导入simple库的import语句时，它首先会在CLASSPATH指定的目录中查找子目录com/mindviewinc/simple，然后从已编译的文件中找出名称相符者。注意，这两个类和其中要访问的方法都必须是public修饰的。

### 冲突

​		若通过\*导入了两个包含相同名字类名的类库，会发生什么?

```java
import com.mindviewinc.simple.*;
import java.util.*;
```

​		因为java.util也包含了Vector类，这就存在潜在的冲突。但是你只要不写导致冲突的代码，就不会有问题——这样很好，否则就得做很多类型检查工作来防止那些根本不会出现的冲突。

​		现在如果要创建一个Vector类，就会出现冲突：

```java
Vector v = new Vector();
```

​		这里的Vector类指的是谁呢？编译器不知道，读者也不知道。所以编译器报错，强制你明确指明。对于标准的Java类Vector，你可以这么写：

```java
java.util.Vector v = new java.util.Vector();
```

​		这种写法完全指明了Vector类的位置，那么就没有必要写import java.util语句，除非使用其他来自java.util中的类。或者，可以导入单个类以防冲突——只要不在同一个程序中使用有冲突的名字(若使用了有冲突的名字，必须要明确指明全名)。

### 定制工具库

​		一般来说，将会使用反转后的域名来命名要创建的工具包，例如：com.mindviewinc.util，但为了简化，这里将工具包命名为onjava。例如，下面是“控制流”一章中使用到的range()方法，采用了for-in语法进行简单的遍历：

```java
// onjava/Range.java
// Array creation methods that can be used without
// qualifiers, using static imports:
package onjava;

public class Range {
    // Produce a sequence [0,n)
    public static int[] range(int n) {
        int[] result = new int[n];
        for (int i = 0; i < n; i++) {
            result[i] = i;
        }
        return result;
    }
    // Produce a sequence [start..end)
    public static int[] range(int start, int end) {
        int sz = end - start;
        int[] result = new int[sz];
        for (int i = 0; i < sz; i++) {
            result[i] = start + i;
        }
        return result;
    }
    // Produce sequence [start..end) incrementing by step
    public static int[] range(int start, int end, int step) {
        int sz = (end - start) / step;
        int[] result = new int[sz];
        for (int i = 0; i < sz; i++) {
            result[i] = start + (i * step);
        }
        return result;
    }
}

```

​		这个文件的位置一定是在某个以一个CLASSPATH位置开始，然后接着是onjava的目录下。编译完之后，就可以在系统的任何地方使用import onjava语句来使用这些方法了。从现在开始，无论何时你创建了有用的新工具，都可以把它加入到自己的类库中。

### 使用import改变行为

​		Java没有C的条件编译(conditional compilation)功能，该功能使你不必更改任何程序代码而能切换开关产生不同的行为。Java之所以去掉此功能，可能是因为C在绝大多数情况下使用该功能解决跨平台问题：程序代码的不同部分要根据不同的平台来编译。而Java自身就是跨平台设计的，这个功能就没有必要了。

​		但是，条件编译还有其他的用途。调试是一个很常见的用途，调式功能在开发过程中是开启的，在发布的产品中是禁用的。可以通过改变导入的package来实现这一目的，修改的方法是将程序中的代码从调试版改为发布版。这个技术可用于任何种类的条件代码。

### 使用包的忠告

​		当你创建一个包时，包名就隐含了目录结构。这个包必须位于包名指定的目录中，该目录必须以CLASSPATH开始的目录中可以查询到。最初使用关键字package可能会有点不顺，因为除非遵守“包名对应目录路径”的规则，否则会收到很多意外的运行时错误信息如找不到特定的类，即使这个类就位于同一目录中。如果你收到类似信息，尝试把package语句注释掉，如果程序能够运行的话，你就知道问题出现在哪里了。

​		注意，编译过的代码通常位于与源代码的不同目录。这是很多工程的标准，而集成开发环境(IDE)通常会自动为我们做这些。必须保证JVM通过CLASSPATH能找到编译后的代码。

## 访问权限修饰符

​		Java访问修饰符public、protected和private位于定义的类名，属性名和方法名之前。每个访问权限修饰符只能控制它所修饰的对象。

​		若不提供访问修饰符，就意味着“包访问权限”。所以无论如何，万物都有某种形式的访问控制权。

### 包访问权限

​		默认访问权限没有关键字，通常被称为包访问权限(package access)。这意味着当前包中的所有其它类都可以访问那个成员。对于这个包之外的类，这个成员看上去是private的。由于一个编译单元只能隶属于一个包，所以通过包访问权限，位于同一编译单元中的所有类彼此之间都是可访问的。

​		包访问权限可以把相关类聚到一个包下，以便它们能轻易地相互访问。包里的类赋予它们包访问权限的成员相互访问的权限，所以你“拥有”了包内的程序代码。只能通过你所拥有的代码去访问你所拥有的其他代码，这样规定很有意义。构建包访问权限机制是将类聚焦在包中的重要原因之一。

​		类控制着哪些代码有权限访问自己的成员。取得对成员的访问权的唯一方式是：

* 使成员成为public。那么无论是谁，无论在哪，都可以访问它。
* 赋予成员默认包访问权限，不用加任何访问修饰符，然后将其他类放在相同的包内。这样，其它类就可以访问该成员。
* 在“复用”这一章你将看到，继承的类既可以访问public成员，也可以访问protected成员(但不能访问private成员)。只有当两个类处于同一个包内，它才可以访问包访问权限的成员。但现在不用担心继承和protected。
* 提供访问器(accessor)和修改器(mutator)(有时也称为“get/set”方法)，从而读取和改变值。

### public:接口访问权限

​		当你使用关键字public，就意味着紧随public后声明的成员对于每个人都是可用的。假设定义了一个包含下面编译单元的dessert包：

```java
// hiding/dessert/Cookie.java
// Creates a library
package hiding.dessert;

public class Cookie {
    public Cookie() {
        System.out.println("Cookie constructor");
    }
    
    void bite() {
        System.out.println("bite");
    }
}
```

​		记住，Cookie.java文件产生的类文件必须位于名为dessert的子目录中，该子目录在hiding下，它必须在CLASSPATH的几个目录下。不要错误地认为Java总是会将当前目录视作查找行为的起点之一。如果你的CLASSPATH中没有.，Java就不会查找当前目录。

​		现在，使用Cookie创建一个程序：

```java
// hiding/Dinner.java
// Uses the library
import hiding.dessert.*;

public class Dinner {
    public static void main(String[] args) {
        Cookie x = new Cookie();
        // -x.bite(); // Can't access
    }
}
//Cookie constructor
```

​		你可以创建一个Cookie对象，因为它构造器和类都是public的。但是，在Dinner.java中无法访问到Cookie对象中的bite()方法，因为bite()只提供了包访问权限，因而在dessert包之外无法访问，编译器禁止你使用它。

### 默认包

​		你可能惊讶地发现，一下代码尽管看上去破坏了规则，但是仍然可以编译：

```java
// hiding/Cake.java
// Accesses a class in a separate compilation unit
class Cake {
    public static void main(String[] args) {
        Pie x = new Pie();
        x.f();
    }
}
//Pie.f()
```

​		同一目录下的第二个文件：

```java
// hiding/Pie.java
// The other class
class Pie {
    void f() {
        System.out.println("Pie.f()");
    }
}
```

​		最初看上去这两个文件毫不相关，但在Cake中可以创建一个Pie对象并调用它的f()方法。(注意，你的CLASSPATH中一定得有.，这样文件才能编译)通常会认为Pie和f()具有包访问权限，因此不能被Cake访问。它们的确具有包访问权限，这是部分正确的。Cake.java可以访问它们是因为它们在相同的目录中且没有给自己设定明确的包名。Java把这样的文件看作隶属于该目录的默认包中，因此它们为该目录中所有的其他文件都提供了包访问权限。

### private:你无法访问

​		关键字private意味着除了包含该成员的类，其他任何类都无法访问这个成员。同一个包中的其他类无法访问private成员，因此这等于说自己隔离自己。另一方面 ，让许多人合作创建一个包也是有可能的。使用private，你可以自由地修改那个被修饰的成员，无需担心会影响同一包下的其它类。

​		默认的包访问权限通常提供了足够的隐藏措施；记住，使用类的客户端成员无法访问包访问权限成员。这样做很好，因为默认访问权限是一种我们常用的权限。因此，通常考虑的是把哪些成员声明为public供客户端程序员使用。所以，最初不常使用关键字private，因为程序没有它也可以照常工作。然而，使用private是非常重要的，尤其是在多线程环境中。

​		以下是一个使用private的例子：

```java
// hiding/IceCream.java
// Demonstrates "private" keyword

class Sundae {
    private Sundae() {}
    static Sundae makeASundae() {
        return new Sundae();
    }
}

public class IceCream {
    public static void main(String[] args) {
        //- Sundae x = new Sundae();
        Sundae x = Sundae.makeASundae();
    }
}
```

​		以上展示了private的用武之地：控制如何创建对象，防止别人直接访问某个特定的构造器。例子中，你无法通过构造器创建一个Sundae对象，而必须调用makeASunda()方法创建对象。

​		任何可以肯定只是该类的“助手”方法，都可以声明为private，以确保不会在包中的其他地方误用它，也防止了你会改变或删除它。将方法声明为private确保了你拥有这种选择权。

​		对于类中的private属性也是一样。除非必须公开底层实现，否则就将属性声明为private。然而，不能因为类中某个对象的引用是private，就认为其他对象也无法拥有该对象的public引用。

### protected:继承访问权限

​		关键字protected处理的是继承的概念，通过继承可以利用一个现有的类——基类，然后添加新成员到现有类中而不必碰现有类。我们还可以改变类的现有成员的行为。为了从一个类中继承，需要声明新类extends一个现有类：

```java
class Foo extends Bar {}
```

​		类定义的其他部分看起来是一样的。

​		如果你创建了一个新包，并从另一个包继承类，那么唯一能访问的就是被继承类的public成员。有时，基类的创建者会希望某个特定成员能被继承类访问，但不能被其他类访问。这时就需要使用protected。protected也提供包访问权限，也就是说，相同包内的其他类可以访问protected元素。

​		下面的类不能调用包访问权限的方法bite()：

```java
// hiding/ChocolateChip.java
// Can't use package-access member from another package
import hiding.dessert.*;

public class ChocolateChip extends Cookie {
    public ChocolateChip() {
        System.out.println("ChocolateChip constructor");
    } 
    
    public void chomp() {
        //- bite(); // Can't access bite
    }
    
    public static void main(String[] args) {
        ChocolateChip x = new ChocolateChip();
        x.chomp();
    }
}
//Cookie constructor
//ChocolateChip constructor
```

​		如果类Cookie中存在一个方法bite()，那么它的任何子类中都存在bite()。但是因为bite()具有包访问权限并且位于另一个包中，所以我们这个包中无法使用它。你可以把它声明为public，但这样一来每个人都能访问它，这可能也不是你想要的。如果你将Cookie改成如下这样：

```java
// hiding/cookie2/Cookie.java
package hiding.cookie2;

public class Cookie {
    public Cookie() {
        System.out.println("Cookie constructor");
    }
    
    protected void bite() {
        System.out.println("bite");
    }
}
```

​		这样，bite()对于所有继承Cookie的类，都是可访问的：

```java
// hiding/ChocolateChip2.java
import hiding.cookie2.*;

public class ChocolateChip2 extends Cookie {
    public ChocoalteChip2() {
        System.out.println("ChocolateChip2 constructor");
    }
    
    public void chomp() {
        bite(); // Protected method
    }
    
    public static void main(String[] args) {
        ChocolateChip2 x = new ChocolateChip2();
        x.chomp();
    }
}
//Cookie constructor
//ChocolateChip2 constructor
//bite
```

​		尽管bite()也具有包访问权限，但它不是public的。

### 包访问权限VsPublic构造器

​		当你定义一个具有包访问权限的类，你可以在类中定义一个public构造器，编译器不会报错：

```java
// hiding/packageaccess/PublicConstructor.java
package hiding.packageaccess;

class PublicConstructor {
    public PublicConstructor() {}
}
```

​		有一个Checkstyle工具，你可以运行命令gradlew hiding:checkstyleMain使用它，它会指出这种写法是虚假的，而且从技术上来说是错误的。实际上你不能从包外访问到这个public构造器：

```java
// hiding/CreatePackageAccessObject.java
// {WillNotCompile}
import hiding.packageaccess.*;

public class CreatePackageAcessObject {
    public static void main(String[] args) {
        new PublicConstructor();
    }
}
```

​		如果你编译下这个类，会得到编译错误信息：

```tex
CreatePackageAccessObject.java:6:error:
PublicConstructor is not public in hiding.packageaccess;
cannot be accessed from outside package
new PublicConstructor();
^
1 error
```

​		因此，在一个具有包访问权限的类中定义一个public的构造器并不能真的使这个构造器成为public，在声明的时候就应该标记为编译时错误。

## 接口和实现

​		访问控制通常被称为隐藏实现(implementation hiding)。将数据和方法包装进类中并把具体实现隐藏被称作是封装(encapsulation)。其结果就是一个同时带有特征和行为的数据类型。

​		出于两个重要的原因，访问控制在数据类型内部划定了边界。第一个原因是确立客户端程序员可以使用和不能使用的边界。可以在结构中建立自己的内部机制而不必担心客户端程序员偶尔将内部实现作为他们可以使用的接口的一部分。

​		这直接引出了第二个原因：将接口与实现分离。如果在一组程序中使用接口，而客户端程序员只能向public接口发送消息的话，那么就可以自由地修改任何不是public的事物，却不会破坏客户端代码。

​		为了清晰起见，你可以采用一种创建类的风格：public成员放在类的开头，接着是protected成员，包访问权限成员，最后是private成员。这么做的好处是类的使用者可以从头读起，首先会看到对他们而言最重要的部分，直到遇到非public成员时停止阅读：

```java
// hiding/OrganizedByAccess.java

public class OrganizedByAccess {
    public void pub1() {/* ... */}
    public void pub2() {/* ... */}
    public void pub3() {/* ... */}
    private void priv1() {/* ... */}
    private void priv2() {/* ... */}
    private void priv3() {/* ... */}
    private int i;
    // ...
}
```

​		这么做只能是程序阅读起来稍微容易一些，因为实现和接口还是混合一起。也就是说，你仍能看到源代码——部分实现，因为它就在类中。另外，javadoc提供的注释文档功能降低了程序代码的可读性对客户端程序员的重要性。将接口展现给类的使用者实际上是类浏览器的任务，类浏览器会展示给所有可用的类，并告诉你如何使用它们。在Java中，JDK文档起到了类浏览器的作用。

## 类访问权限

​		访问权限修饰符也可以用于确定类库中的哪些类对于类库的使用者是可用的。如果希望某个类可以被客户端程序员使用，就把关键字public作用于整个类的定义。这甚至控制着客户端程序员能否创建类的对象。

​		为了控制一个类的访问权限，修饰符必须出现在关键字class之前：

```java
public class Widget {
```

​		如果你类库名是hiding，那么任何客户端程序员都可以通过如下声明访问Widget：

```java
import hiding.Widget;
```

​		或者：

```java
import hiding.*;
```

​		这里有一些额外的限制：

* 每个编译单元中只能有一个public类。这表示，每个编译单元有一个公共的接口用public类表示。该接口可以包含许多支持包访问权限的类。一旦一个编译单元中出现一个以上的public类，编译就会报错。
* public类的名称必须与含有编译单元的文件名相同，包括大小写。所以对于Widget来说，文件名必须是Widget.java，不能是widget.java或WIDGET.java。再次强调，如果名字不匹配，编译器会报错。
* 虽然不是很常见，但是编译单元内没有public类也是可能的。这时可以随意命名文件



​		如果是一个在hiding包中的类，只用来完成Widget或hiding包下一些其他public类所要执行的任务，怎么设置它的访问权限呢？为了保留此灵活性，需要确保客户端程序员不依赖隐藏在hiding中的任何特定细节，那么把public关键字从类中去掉，给予它包访问权限，就可以了。

​		当你创建了一个包访问权限的类，把类中的属性声明为private仍然是有意义的——应该尽可能将所有属性都声明为private，但是通常把方法声明成与类相同的访问权限也是合理的。一个包访问权限的类只能被用于包内，除非强制将某些方法声明为public，这种情况下，编译器会告诉你。

​		注意，类既不能是private的，也不能是protected的。所以对于类的访问权限只有两种选择：包访问权限或public。为了防止类被外界访问，可以将所有的构造器声明为private，这样只有你自己能创建对象(在类的static成员中)。

```java
// hiding/Lunch.java
// Demonstrates class access specifiers. Make a class
// effectively private with private constructors:

class Soup1 {
    private Soup1() {}
    
    public static Soup1 makeSoup() { // [1]
        return new Soup1();
    }
}

class Soup2 {
    private Soup2() {}
    
    private static Soup2 ps1 = new Soup2(); // [2]
    
    public static Soup2 access() {
        return ps1;
    }
    
    public void f() {}
}
// Only one public class allowed per file:
public class Lunch {
    void testPrivate() {
        // Can't do this! Private constructor:
        //- Soup1 soup = new Soup1();
    }
    
    void testStatic() {
        Soup1 soup = Soup1.makeSoup();
    }
    
    void testSingleton() {
        Soup2.access().f();
    }
}
```

​		可以像[1]那样通过static方法创建对象，也可以像[2]那样先创建一个静态对象，当用户需要访问它时返回对象的引用即可。

​		到目前为止，大部分的方法要么返回void，要么返回基本类型，所以[1]处的定义乍看之下会有困惑。方法名(makeSoup)前面的Soup1表明了方法返回的类型。到目前为止，这里经常是void，即不返回任何东西。然而也可以返回对象的引用，就像这里一样。这个方法返回了对Soup1类对象的引用。

​		Soup1和Soup2展示了如何通过将你所有的构造器声明为private的方式防止直接创建某个类的对象。记住，如果你不显示地创建构造器，编译器会自动为你创建一个无参构造器。如果我们编写了无参构造器，那么编译器就不会自动创建构造器了。将构造器声明为private，那么谁也无法创建该类的对象了。但是现在别人该怎么使用这个类呢？上述例子给出了两个选择。在Soup1中，有一个static方法，它的作用是创建一个新的Soup1对象并返回对象的引用。如果想要在返回引用之前在Soup1上做一些额外的操作，或是记录创建了多少个Soup1对象(可以用来限制数量)，这种做法是有用的。

​		Soup2用到了所谓的设计模式(design pattern)。这种模式叫做单例模式(singleton)，因为它只允许创建类的一个对象。Soup2类的对象是作为Soup2的static private成员而创建的，所以有且只有一个，你只能通过public修饰的access()方法访问到这个对象。