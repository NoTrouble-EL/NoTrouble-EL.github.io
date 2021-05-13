---
title: 阅读《On-Java-8》--第四章 运算符
date: 2021-04-21 13:41:38
tags:
- JAVASE
- 《On Java 8》
categories: JAVASE
mathjax: true
---

## 开始使用

​		运算符接收一个或多个参数并生成新值。这个参数与普通方法调用的形式不同，但效果相同。所有运算符都能根据自己的运算对象生成一个值。除此之外，一些运算符可以改变运算对象的值，这叫作“副作用”(Side Effect)。运算符最常见的用途就是修改自己的运算对象，从而产生副作用。但要注意生成的值亦可由没有副作用的运算符生成。

​		几乎所有的运算符只能操作基本类型(Primitives)。唯一例外的是=、==、!=，它能操作所有对象。除此之外String类支持“+”和“+=”。

 <!-- more --> 

## 优先级

​		运算符的优先级决定了存在多个运算符时一个表达式各部分的运算顺序。Java对运算顺序作出了特别的规定。其中，最简单的就是乘法和除法在加法和减法之前完成。程序员经常都会忘记其他优先规则，所以应该用括号明确规定运算顺序。

```java
public class Precedence{
    public static void main(String[] args){
        int x = 1, y = 2, z = 3;
        int a = x + y - 2 / 2 + z;//5
        int b = x + (y - 2) / (2 + z)//1
        System.out.println("a = " + a);
        System.out.println("b = " + b);
    }
}
```

​		这些语句看起来大致相同，但从输出中可以看出它们具有非常不同的含义，具体取决于括号的使用。注意到，在System.out.println()语句中使用了+运算符。但在这里+代表的意思是字符串连接符。编译器会将+连接的非字符串尝试转化为字符串。上例中的输出结果说明了a和b都已经被转化成了字符串。

## 赋值

​		运算符的赋值是由符号=完成的。它代表着获取=右边的值并赋给左边的变量。右边可以是任何常量、变量或者可产生一个返回值的表达式。但左边必须是一个明确的、已命名的变量。也就是说，必须要有一个物理空间来存放右边的值。举个例子，可以将常数赋给一个变量(A=4)，但不可以将任何东西赋给一个常数(4=A)。

​		基本类型的赋值都是直接的，而不像对象，赋予的只是其内存的引用。举个例子，a=b，如果b是基本类型，那么赋值操作会将b的值复制一份给变量a，此后若a的值发生改变是不会影响到b的。

​		如果是为对象赋值，那么结果就不一样了。对一个对象进行操作时，我们实际上操作的是它的引用。所以我们将右边的对象赋予给左边时，赋予的只是该对象的引用。此时，两者指向的堆中的对象还是同一个：

```java
class Tank{
    int level;
}

public class Assignment{
    public static void main(String[] args){
        Tank t1 = new Tank();
        Tank t2 = new Tank();
        t1.level = 9;
        t2.level = 47;
        System.out.println("1: t1.level:" + t1.level + ", t2.level:" + t2.level);
        t1 = t2;
        System.out.println("1: t1.level:" + t1.level + ", t2.level:" + t2.level);
        t1.level = 27;
        System.out.println("1: t1.level:" + t1.level + ", t2.level:" + t2.level);
    }
}
//1: t1.level: 9, t2.level: 47
//1: t1.level: 47, t2.level: 47
//1: t1.level: 27, t2.level: 27
```

​		这是一个简单的Tank类，在main()方法创建了两个实例对象。两个对象的level属性分别被赋予不同的值。然后，t2的值被赋予给t1。在Java中，由于赋予的只是对象的引用，改变t1也就改变t2.这是因为t1和t2此时指向的是堆中同一个对象。

​		这种现象通常称为别名(aliasing)，这是Java处理对象的一种基本方式。但是你若不想出现这里的别名引起混淆的话，你可以这么做：

```java
t1.level = t2.level;
```

​		这样做保留了两个单独的的对象，而不是丢弃一个并将t1和t2绑定到同一个对象。

### 方法调用中的别名现象

​		当我们把对象传递给方法时，会发生别名现象。

```java
class Letter{
    char c;
}

public class PassObject{
    static void f(Letter y){
        y.c = 'z';
    }
    
    public static void main(String[] args){
        Letter x = new Letter();
        x.c = 'a';
        System.out.println("1: x.c:" + x.c);
        f(x);
        System.out.println("2: x.c:" + x.c);
    }
}

//1: x.c: a
//1: x.c: z
```

## 算术运算符

​		Java的基本运算符与其他大多编程语言是相同的。

```java
// operators/MathOps.java
// The mathematical operators
import java.util.*;

public class MathOps {
    public static void main(String[] args) {
        // Create a seeded random number generator:
        Random rand = new Random(47);
        int i, j, k;
        // Choose value from 1 to 100:
        j = rand.nextInt(100) + 1;
        System.out.println("j : " + j);
        k = rand.nextInt(100) + 1;
        System.out.println("k : " + k);
        i = j + k;
        System.out.println("j + k : " + i);
        i = j - k;
        System.out.println("j - k : " + i);
        i = k / j;
        System.out.println("k / j : " + i);
        i = k * j;
        System.out.println("k * j : " + i);
        i = k % j;
        System.out.println("k % j : " + i);
        j %= k;
        System.out.println("j %= k : " + j);
        // 浮点运算测试
        float u, v, w; // Applies to doubles, too
        v = rand.nextFloat();
        System.out.println("v : " + v);
        w = rand.nextFloat();
        System.out.println("w : " + w);
        u = v + w;
        System.out.println("v + w : " + u);
        u = v - w;
        System.out.println("v - w : " + u);
        u = v * w;
        System.out.println("v * w : " + u);
        u = v / w;
        System.out.println("v / w : " + u);
        // 下面的操作同样适用于 char, 
        // byte, short, int, long, and double:
        u += v;
        System.out.println("u += v : " + u);
        u -= v;
        System.out.println("u -= v : " + u);
        u *= v;
        System.out.println("u *= v : " + u);
        u /= v;
        System.out.println("u /= v : " + u);    
    }
}
//j : 59
//k : 56
//j + k : 115
//j - k : 3
//k / j : 0
//k * j : 3304
//k % j : 56
//j %= k : 3
//v : 0.5309454
//w : 0.0534122
//v + w : 0.5843576
//v - w : 0.47753322
//v * w : 0.028358962
//v / w : 9.940527
//u += v : 10.471473
//u -= v : 9.940527
//u *= v : 5.2778773
//u /= v : 9.940527
```

​		为了生成随机数，程序先创建一个Random对象。不带参数的Random对象会利用当前时间用作随机数生成器的“种子”(seed)，从而为程序的每次执行成不同的输出。为了每个示例末尾的输出尽可能一致，以便可以使用外部工具进行验证。所以我们在创建Random对象时提供种子(随机数生成器的初始值，其始终为特定种子值产生相同的序列)，让程序每次执行都生成相同的随机数，如此以来输出结果就是可验证的。若需要生成随机值，可以删除种子参数。该对象通过调用方法nextInt()和nextFloat()(还可以调用nextLong()或nextDouble())，使用Random对象生成许多不同类型的随机数。nextInt()的参数设置生成的数字的上限，下限为零，为了避免零除的可能性，结果偏移1。

### 一元加减运算符

​		一元加+减-运算符的操作和二元是相同的。编译器可自动识别使用任何方式解析运算：

```java
x = -a;
x = a * -b;
x = a * (-b);
```

​		一元减号可以得到数据的负值。一元加号的作用相反，不过它唯一能影响的就是把较小的数值类型自动转换为int类型。

## 递增和递减

​		其中递增++和递减- -，意为“增加或减少一个单位”。举个例子来说，假设a是一个int类型的值，则表达式++a就等价于a=a+1。递增和递减运算符不仅可以修改变量，还可以生成变量的值。

​		每种类型的运算符，都有两个版本可供选择；通常将其称为“前缀”和“后缀”。“前递增”表示++运算符位于变量或表达式前面；而“后递增”表示++运算符位于变量的后面。类似地，“前递减”意味着- -运算符位于变量的前面；而“后递减”意味着 - -运算符位于变量的后面。对于前递增和前递减(如++a或- - a)，先会执行递增/减运算，再返回值。而对于后递增和后递减(如a++或a - -)，会先返回值，再执行递增/减运算：

```java
// operators/AutoInc.java
// 演示 ++ 和 -- 运算符
public class AutoInc {
    public static void main(String[] args) {
        int i = 1;
        System.out.println("i: " + i);
        System.out.println("++i: " + ++i); // 前递增
        System.out.println("i++: " + i++); // 后递增
        System.out.println("i: " + i);
        System.out.println("--i: " + --i); // 前递减
        System.out.println("i--: " + i--); // 后递减
        System.out.println("i: " + i);
    }
}
//i: 1
//++i: 2
//i++: 2
//i: 3
//--i: 2
//i-- : 2
//i: 1
```

​		对于前缀形式，我们将在执行递增/减操作后获取值；使用后缀形式，我们将在执行递增/减操作之前获取值。它们是唯一具有“副作用”的运算符(除那些涉及赋值的以外)——它们修改了操作数的值。

## 关系运算符

​		关系运算符==和!=同样适用于所有对象之间的比较，但它们比较的内容却经常困扰Java初学者：

```java
// operators/Equivalence.java
public class Equivalence {
    public static void main(String[] args) {
        Integer n1 = 47;
        Integer n2 = 47;
        System.out.println(n1 == n2);
        System.out.println(n1 != n2);
    }
}
//true
//false
```

​		表达式System.out.println(n1 == n2)将会输出比较的结果。因为两个Integer对象相同，所以先输出true，再输出false。但是，尽管对象的内容一样，对象的引用却不一样。==和!=比较的是对象的引用，所以输出实际上应该先是false，再输出true(如你把45改成128，那么打印的结果就是这样，因为Integer内部维护着一个IntegerCache的缓存，，默认缓存范围是[-128,127]，所以[-128,127]之间的值用\==和!=比较也能得到正确的结果，但是不推荐用关系运算符比较)。

​		那么怎么比较两个对象的内容是否相同呢？你必须使用所有对象(不包括基本类型)中都存在的equals()方法：

```java
// operators/EqualsMethod.java
public class EqualsMethod {
    public static void main(String[] args) {
        Integer n1 = 47;
        Integer n2 = 47;
        System.out.println(n1.equals(n2));
    }
}
//true
```

​		上面的结果看起来是我们所期望的。但其实事情并非那么简单。下面我们来创建自己的类：

```java
// operators/EqualsMethod2.java
// 默认的 equals() 方法没有比较内容
class Value {
    int i;
}

public class EqualsMethod2 {
    public static void main(String[] args) {
        Value v1 = new Value();
        Value v2 = new Value();
        v1.i = v2.i = 100;
        System.out.println(v1.equals(v2));
    }
}
//false
```

​		上例的结果再次令人困惑：结果是false。原因是equals()的默认行为是比较对象的引用而非具体内容。因此，除非你在新类中覆写equals()方法，否则我们将获取不到现要的结果。

## 逻辑运算符

​		每个逻辑运算符&&(AND)、||(OR)、!(非)根据参数的逻辑关系生成布尔值true或false。下面的代码示例中使用了关系运算符和逻辑运算符：

```java
// operators/Bool.java
// 关系运算符和逻辑运算符
import java.util.*;
public class Bool {
    public static void main(String[] args) {
        Random rand = new Random(47);
        int i = rand.nextInt(100);
        int j = rand.nextInt(100);
        System.out.println("i = " + i);
        System.out.println("j = " + j);
        System.out.println("i > j is " + (i > j));
        System.out.println("i < j is " + (i < j));
        System.out.println("i >= j is " + (i >= j));
        System.out.println("i <= j is " + (i <= j));
        System.out.println("i == j is " + (i == j));
        System.out.println("i != j is " + (i != j));
        // 将 int 作为布尔处理不是合法的 Java 写法
        //- System.out.println("i && j is " + (i && j));
        //- System.out.println("i || j is " + (i || j));
        //- System.out.println("!i is " + !i);
        System.out.println("(i < 10) && (j < 10) is "
        + ((i < 10) && (j < 10)) );
        System.out.println("(i < 10) || (j < 10) is "
        + ((i < 10) || (j < 10)) );
    }
}
//i = 58
//j = 55
//i > j is true
//i < j is false
//i >= j is true
//i <= j is false
//i == j is false
//i != j is true
//(i < 10) && (j < 10) is false
//(i < 10) || (j < 10) is false

```

​		在Java逻辑运算中，我们不能像C/C++那样使用非布尔值，而仅能使用AND、OR、NOT。上面的例子中，我们将使用非布尔值的表达式注释掉了。请注意，如果在预期为String类型的位置上使用boolean类型的值，则结果会自动转为适当的文本格式(即“true”或“false”字符串)。

​		我们可以将前一个程序中int的定义替换为除boolean之外的任何其他基本数据类型。但请注意，float类型的数值比较严格，只要两个数字的最小位不同则两个数仍然不相等；只要数字最小位是大于0的，那么它就不等于0。

### 短路

​		逻辑运算符支持一种称为“短路”(short-circuiting)的现象。整个表达式会在运算到可以明确结果时就停止并返回结果，这意味着该逻辑表达式的后半部分不会被执行到：

```java
// operators / ShortCircuit.java 
// 逻辑运算符的短路行为
public class ShortCircuit {

    static boolean test1(int val) {
        System.out.println("test1(" + val + ")");
        System.out.println("result: " + (val < 1));
        return val < 1;
    }

    static boolean test2(int val) {
        System.out.println("test2(" + val + ")");
        System.out.println("result: " + (val < 2));
        return val < 2;
    }

    static boolean test3(int val) {
        System.out.println("test3(" + val + ")");
        System.out.println("result: " + (val < 3));
        return val < 3;
    }

    public static void main(String[] args) {
        boolean b = test1(0) && test2(2) && test3(2);
        System.out.println("expression is " + b);
    }
}
//test1(0)
//result: true
//test2(2)
//result: false
//expression is false
```

​		每个测试都对参数执行比较并返回true或false。同时控制台也会在方法执行时打印它们的执行状态：

```java
test1(0) && test2(2) && test3(2)
```

​		你可能预期是程序会执行3个test方法并返回。我们来分析一下：第一个方法返回true，因此表达式会继续走下去。紧接着，第二个方法的返回结果是false。这就代表这整个表达式的结果肯定为false，所以就没有必须再判断剩下的表达式部分了。

​		所以，运用“短路”可以节省部分不必要的运算，从而提高程序潜在的性能。

## 字面值常量

​		通常，当我们程序中插入一个字面值常量(Literal)时，编译器会确切地识别它的类型。当类型不明确时，必须辅以字面值常量关联来帮助编译器识别：

```java
// operators/Literals.java
public class Literals {
    public static void main(String[] args) {
        int i1 = 0x2f; // 16进制 (小写)
        System.out.println(
        "i1: " + Integer.toBinaryString(i1));
        int i2 = 0X2F; // 16进制 (大写)
        System.out.println(
        "i2: " + Integer.toBinaryString(i2));
        int i3 = 0177; // 8进制 (前导0)
        System.out.println(
        "i3: " + Integer.toBinaryString(i3));
        char c = 0xffff; // 最大 char 型16进制值
        System.out.println(
        "c: " + Integer.toBinaryString(c));
        byte b = 0x7f; // 最大 byte 型16进制值  01111111;
        System.out.println(
        "b: " + Integer.toBinaryString(b));
        short s = 0x7fff; // 最大 short 型16进制值
        System.out.println(
        "s: " + Integer.toBinaryString(s));
        long n1 = 200L; // long 型后缀
        long n2 = 200l; // long 型后缀 (容易与数值1混淆)
        long n3 = 200;
    
        // Java 7 二进制字面值常量:
        byte blb = (byte)0b00110101;
        System.out.println(
        "blb: " + Integer.toBinaryString(blb));
        short bls = (short)0B0010111110101111;
        System.out.println(
        "bls: " + Integer.toBinaryString(bls));
        int bli = 0b00101111101011111010111110101111;
        System.out.println(
        "bli: " + Integer.toBinaryString(bli));
        long bll = 0b00101111101011111010111110101111;
        System.out.println(
        "bll: " + Long.toBinaryString(bll));
        float f1 = 1;
        float f2 = 1F; // float 型后缀
        float f3 = 1f; // float 型后缀
        double d1 = 1d; // double 型后缀
        double d2 = 1D; // double 型后缀
        // (long 型的字面值同样适用于十六进制和8进制 )
    }
}
//i1: 101111
//i2: 101111
//i3: 1111111
//c: 1111111111111111
//b: 1111111
//s: 111111111111111
//blb: 110101
//bls: 10111110101111
//bli: 101111101011111010111110101111
//bll: 101111101011111010111110101111
```

​		在本文值的后面添加的字符可以让编译器识别该文本值得类型。对于Long型数值，结尾使用大写L或小写l皆可(不推荐使用l，因为与阿拉伯数字1容易混淆)。大写F或小写f表示float浮点数。大写D或小写d表示double双精度。

​		十六进制，适用于所有整型数据类型，由前导0x或0X表示，后跟0-9或a-f(大写或小写)。如果我们在初始化某个类型的数值时，赋值超出其范围，那么编译器会报错。在上例的代码中，char、byte和short的值已经是最大了。如果超过这些值，编译器将会自动转型为int，并且提示我们需要声明强制类型转换，意味着我们已经越过该类的范围界限。

​		八进制由0~7之间数字和前导0表示。

​		Java7引入了二进制的字面值常量，由前导0b或0B表示，它可以初始化所有的整数类型。

​		使用整数值类型时，显示其二进制形式会很有用。Long型Integer型中这很容易实现，调用其静态的toBinaryString()方法即可。但是请注意，若将较小的类型传递给Integer.toBinaryString()时，类型将自动转型为int。

### 下划线

​		Java7中有一个深思熟虑的补充：我们可以在数字字面量中包含下划线_，以使结果更加清晰。这对于大数值的分组特别有用。代码示例：

```java
// operators/Underscores.java
public class Underscores {
    public static void main(String[] args) {
        double d = 341_435_936.445_667;
        System.out.println(d);
        int bin = 0b0010_1111_1010_1111_1010_1111_1010_1111;
        System.out.println(Integer.toBinaryString(bin));
        System.out.printf("%x%n", bin); // [1]
        long hex = 0x7f_e9_b7_aa;
        System.out.printf("%x%n", hex);
    }
}
//3.41435936445667E8
//101111101011111010111110101111
//2fafafaf
//7fe9b7aa
```

​		下面为合理的使用规则：

* 仅限单_，不能多条相连；
* 数值开头和结尾不允许出现_；
* F、D和L的前后禁止出现_；
* 二进制前导b和十六进制x前后禁止出现_。



​		注意%n的使用。熟悉C风格的程序员可能习惯于看到\\n来表示换行符。问题在于它给你一个“Unix风格”的换行符。此外，如果我们使用的是Windows，则必须指定\\r\\n。这就是Java用%n实现的可以忽略平台间差异而生成适当的换行符，但只有当你使用System.out.printf()或System.out.format()时。对于System.out.println()，我们仍然必须使用\\n；如果你使用%n，println()只会输出%n而不是换行符。

### 指数计数法

```java
// operators/Exponents.java
// "e" 表示 10 的几次幂
public class Exponents {
    public static void main(String[] args) {
        // 大写 E 和小写 e 的效果相同:
        float expFloat = 1.39e-43f;
        expFloat = 1.39E-43f;
        System.out.println(expFloat);
        double expDouble = 47e47d; // 'd' 是可选的
        double expDouble2 = 47e47; // 自动转换为 double
        System.out.println(expDouble);
    }
}
//1.39E-43
//4.7E48
```

​		在科学与工程学领域，e代表自然对数的基数，约等于2.718(Java里用一种更精确的double值Math.E来表示自然对数)。指数表达式“1.39x e-43”一位置“1.39x2.718的-43次方”。然而，自FORTRAN语言发明后，人们自然而然地觉得e代表“10的几次幂”。

​		在Java中看到类似“1.39e-43f”这样的表达式，请转换思维，从程序设计的角度思考它；它正真的含义是“1.39x10的-43次方”。

​		注意若编译器能够正确地识别类型，就不必使用后缀字符：

```java
long n3 = 200;
```

​		它并不存在含糊不清的地方，所以200后面的L大可省去。然而，对于以下语句：

```java
float f4 = 1e-43f; //10 的幂数
```

​		编译器通常会将指数作为double类型来处理，所以倘若没有这个后缀f，编译器就会报错，提示我们应该将double型转换成float型。

## 位运算符

​		位运算符允许我们操作一个整型数字中的单个二进制。位运算会对两个整数对应的位执行布尔代数，从而产生结果。

​		若两个输入位都是1，则按位“与运算符”\&运算后结果是1，否则结果是0。若两个输入位至少有一个是1，则按位“或运算”\|运算后结果是1；只有在两个输入位都是0的情况下，运算结果才是0。若两个输入位的某一个是1，另一个不是1，那么按位“异或运算符”\^运算后结果才是1.按位“非运算符”\~属于一元运算符；它只对一个自变量进行操作(其他所有运算符都是二元运算符)。按位非运算后结果于输入位相反。例如输入0，则输出1；输入1，则输出0。

​		位运算符和逻辑运算符都是用了同样的字符，只不过数量不同。位短，所以位运算符只有一个字符。位运算符可与等号\=联合使用已接收结果及赋值：\&=，\|=，\^=都是合法的(由于\~是一元运算符，所以不可与\=联合使用)。

​		我们将Boolean类型被视为“单位值”(one-bit value)，所以它多少有些独特的地方。我们可以针对boolean变量执行与、或、异或运算，但不能执行非运算。对于布尔值，位运算符具有与逻辑运算符相同的效果，只是它们不会中途“短路”。此外，针对布尔值进行的位运算为我们新增了一个“异或”逻辑运算符，它并未包括在逻辑运算符的列表中。在移位表达式中，禁止使用布尔值。

## 移位运算符

​		移位运算符面向的对象也是二进制的“位”。它只能用于处理整数类型。左移位运算符\<<能将其左边的运算对象向左移动右侧指定的位数(在低位补0)。右移位运算符\>>则相反。右移位运算符有“正”，“负”值：若值为正，则在高位插入0；若值为负，则在高位插入1。Java也添加了一种“不分正负”的右移位运算(\>>>)，它使用了“零扩展”(zero extension)：无论正负，都在高位插入0。

​		若移动char、byte或short，则会在移动之前将其提升为int，结果为int。仅使用右值(rvalue)的5个低阶位。这可防止我们移动超过int范围的位数。若对一个long值进行处理，最后得到的结果也是long。

​		移位可以与等号\<<=或\>>=或\>>>=组合使用。左值被替换为其移位运算后的值。但是，问题来了，当无符号右移与赋值相结合时，若将其与byte或short一起使用的话，则结果错误。取而代之的是，它们被提升为int型并右移，但在重新赋值时被截断。在这种情况下，结果为-1。

```java
// operators/URShift.java
// 测试无符号右移
public class URShift {
    public static void main(String[] args) {
        int i = -1;
        System.out.println(Integer.toBinaryString(i));
        i >>>= 10;
        System.out.println(Integer.toBinaryString(i));
        long l = -1;
        System.out.println(Long.toBinaryString(l));
        l >>>= 10;
        System.out.println(Long.toBinaryString(l));
        short s = -1;
        System.out.println(Integer.toBinaryString(s));
        s >>>= 10;
        System.out.println(Integer.toBinaryString(s));
        byte b = -1;
        System.out.println(Integer.toBinaryString(b));
        b >>>= 10;
        System.out.println(Integer.toBinaryString(b));
        b = -1;
        System.out.println(Integer.toBinaryString(b));
        System.out.println(Integer.toBinaryString(b>>>10));
    }
}
//11111111111111111111111111111111
//1111111111111111111111
//1111111111111111111111111111111111111111111111111111111111111111
//111111111111111111111111111111111111111111111111111111
//11111111111111111111111111111111
//11111111111111111111111111111111
//11111111111111111111111111111111
//11111111111111111111111111111111
//11111111111111111111111111111111
//1111111111111111111111
```

```java
// operators/BitManipulation.java
// 使用位运算符
import java.util.*;
public class BitManipulation {
    public static void main(String[] args) {
        Random rand = new Random(47);
        int i = rand.nextInt();
        int j = rand.nextInt();
        printBinaryInt("-1", -1);
        printBinaryInt("+1", +1);
        int maxpos = 2147483647;
        printBinaryInt("maxpos", maxpos);
        int maxneg = -2147483648;
        printBinaryInt("maxneg", maxneg);
        printBinaryInt("i", i);
        printBinaryInt("~i", ~i);
        printBinaryInt("-i", -i);
        printBinaryInt("j", j);
        printBinaryInt("i & j", i & j);
        printBinaryInt("i | j", i | j);
        printBinaryInt("i ^ j", i ^ j);
        printBinaryInt("i << 5", i << 5);
        printBinaryInt("i >> 5", i >> 5);
        printBinaryInt("(~i) >> 5", (~i) >> 5);
        printBinaryInt("i >>> 5", i >>> 5);
        printBinaryInt("(~i) >>> 5", (~i) >>> 5);
        long l = rand.nextLong();
        long m = rand.nextLong();
        printBinaryLong("-1L", -1L);
        printBinaryLong("+1L", +1L);
        long ll = 9223372036854775807L;
        printBinaryLong("maxpos", ll);
        long lln = -9223372036854775808L;
        printBinaryLong("maxneg", lln);
        printBinaryLong("l", l);
        printBinaryLong("~l", ~l);
        printBinaryLong("-l", -l);
        printBinaryLong("m", m);
        printBinaryLong("l & m", l & m);
        printBinaryLong("l | m", l | m);
        printBinaryLong("l ^ m", l ^ m);
        printBinaryLong("l << 5", l << 5);
        printBinaryLong("l >> 5", l >> 5);
        printBinaryLong("(~l) >> 5", (~l) >> 5);
        printBinaryLong("l >>> 5", l >>> 5);
        printBinaryLong("(~l) >>> 5", (~l) >>> 5);
    }

    static void printBinaryInt(String s, int i) {
        System.out.println(
        s + ", int: " + i + ", binary:\n " +
        Integer.toBinaryString(i));
    }

    static void printBinaryLong(String s, long l) {
        System.out.println(
        s + ", long: " + l + ", binary:\n " +
        Long.toBinaryString(l));
    }
}
//-1, int: -1, binary:
//11111111111111111111111111111111
//+1, int: 1, binary:
//1
//maxpos, int: 2147483647, binary:
//1111111111111111111111111111111
//maxneg, int: -2147483648, binary:
//10000000000000000000000000000000
//i, int: -1172028779, binary:
//10111010001001000100001010010101
//~i, int: 1172028778, binary:
// 1000101110110111011110101101010
//-i, int: 1172028779, binary:
//1000101110110111011110101101011
//j, int: 1717241110, binary:
//1100110010110110000010100010110
//i & j, int: 570425364, binary:
//100010000000000000000000010100
//i | j, int: -25213033, binary:
//11111110011111110100011110010111
//i ^ j, int: -595638397, binary:
//11011100011111110100011110000011
//i << 5, int: 1149784736, binary:
//1000100100010000101001010100000
//i >> 5, int: -36625900, binary:
//11111101110100010010001000010100
//(~i) >> 5, int: 36625899, binary:
//10001011101101110111101011
//i >>> 5, int: 97591828, binary:
//101110100010010001000010100
//(~i) >>> 5, int: 36625899, binary:
//10001011101101110111101011
    ...

```

## 三元运算符

​		三元运算符，也称为条件运算符。下面是它的表达式格式：
​		布尔表达式？值1：值2

​		若表达式计算为true，则返回值1；如果表达式的计算为false，则返回值2。

​		当然，也可以换用普通的**if-else**语句，但三元运算符更加简洁。与if-else不同的是，三元运算符是有返回结果的：

```java
// operators/TernaryIfElse.java
public class TernaryIfElse {
    
static int ternary(int i) {
    return i < 10 ? i * 100 : i * 10;
}

static int standardIfElse(int i) {
    if(i < 10)
        return i * 100;
    else
        return i * 10;
}

    public static void main(String[] args) {
        System.out.println(ternary(9));
        System.out.println(ternary(10));
        System.out.println(standardIfElse(9));
        System.out.println(standardIfElse(10));
    }
}
//900
//100
//900
//100
```

​		可以看出，ternary()中的代码更简短。然而，standardIfElse()中的代码更易于理解且不要求更多的录入。所以在挑选三元运算符时，请务必权衡一下利弊。

## 字符串运算

​		这个运算符在Java里有一项特殊用途：连接字符串。

​		我们注意到运用String + 时有一些有趣的现象。若表达式以一个String类型开头，那么后续所有运算对象都必须是字符串：

```java
// operators/StringOperators.java
public class StringOperators {
    public static void main(String[] args) {
        int x = 0, y = 1, z = 2;
        String s = "x, y, z ";
        System.out.println(s + x + y + z);
        // 将 x 转换为字符串
        System.out.println(x + " " + s);
        s += "(summed) = "; 
        // 级联操作
        System.out.println(s + (x + y + z));
        // Integer.toString()方法的简写:
        System.out.println("" + x);
    }
}
//x, y, z 012
//0 x, y, z
//x, y, z (summed) = 3
//0
```

​		注意：上例中第1输出语句的执行结果是012而并非3，这是因为编译器将其分别转换为其字符串形式然后与字符串变量s连接。在第2条输出语句中，编译器将开头的变量转换为了字符串，由此可以看出，这种转换与数据位置无关，只要当中有一条数据是字符串类型，其他非字符串数据都被转换为字符串形式并连接。最后一条输出语句，我们可以看出\+=运算符可以拼接其右侧的字符串连接结果并重赋值给自身变量s。括号()可以控制表达式的计算顺序，以便在显示int之前对其进行实际求和。

​			请注意最后方法中的最后一个例子：我们经常会看到一个空字符串“ ”跟着一个基本数据类型。这样可以隐式地将其转换为字符串，以代替繁琐的显示调用方法(Integer.toString())。

## 常见陷阱

​		使用运算符时很容易犯的一个错误是，在还没搞清楚表达式的计算方式时就试图忽略括号()。

```java
while(x = y) {
// ...
}

```

​		显然，程序员愿意是测试等价性\==，而非赋值\=。在Java中，这样的表达式并结果并不会转化为一个布尔值。而编译器会试图把这个int类型的数据转换为预期应接收的布尔类型。最后，我们将会在试图运行前收到编译期错误。因此，Java天生避免了这种陷阱发生的可能性。

​		唯有一种情况例外：当变量x和y都是布尔值，例如x=y是一个逻辑表达式。除此之外，之前的那个例子，很大可能是错误的。

## 类型转换

​		“类型转换”(Casting)的作用是“与一个模型匹配”。在适当的时候，Java会将一种数据类型自动转换成另一种。例如，假设我们为float变量赋值一个整数型，计算机会将int自动转换为float。我们可以在程序未自动转换时显式、强制地使此类型发生转换。

​		要执行强制转换，需要将所需要的数据类型放在任何左侧的括号内：

```java
// operators/Casting.java
public class Casting {
    public static void main(String[] args) {
        int i = 200;
        long lng = (long)i;
        lng = i; // 没有必要的类型提升
        long lng2 = (long)200;
        lng2 = 200;
        // 类型收缩
        i = (int)lng2; // Cast required
    }
}

```

​		你可以这样地去转换一个数值类型的变量。但上例这种做法是多余的：因为编译器会在必要时自动提升int数据类型为long型。

​		当然，为了程序逻辑清晰或提醒自己留意，我们也可以显示地类型转换。在其他情况下，类型转换只有在代码编译时才显出其重要特性。在Java里，类型转换则是一种比较安全的操作。但是，若将数据类型进行“向下转换”(Narrowing Conversion)的操作(将容量较大的数据类型转换成容量较小的类型)，可能会发生信息丢失的危险。此时，编译器会强迫我们进行转型，好比在提醒我们：改操作可能危险，若你坚持着这么做，对不起，请明确需要转换的类型。对于“向上转换”(Widening conversion)，则不必进行显示的类型转换，因为较大类型的数据肯定能容纳较小类的数据，不会造成任何信息的丢失。

​		除了布尔类型，Java允许任何基本类型的数据转换为另外一种基本类型的数据。此外，类是不能进行类型转换的。为了将一个类型转换为另一个类型，需要使用特殊的方法。

### 截断和舍入

​		在执行“向下转换”时，必须注意数据的截断和舍入问题。若从浮点数转换为整型值，Java会做什么呢？例如：浮点数29.7被转换为整型值，结果会是29还是30？

```java
// operators/CastingNumbers.java
// 尝试转换 float 和 double 型数据为整型数据
public class CastingNumbers {
    public static void main(String[] args) {
        double above = 0.7, below = 0.4;
        float fabove = 0.7f, fbelow = 0.4f;
        System.out.println("(int)above: " + (int)above);
        System.out.println("(int)below: " + (int)below);
        System.out.println("(int)fabove: " + (int)fabove);
        System.out.println("(int)fbelow: " + (int)fbelow);
    }
}
//(int)above: 0
//(int)below: 0
//(int)fabove: 0
//(int)fbelow: 0
```

​		因此，答案是，从float和double转换为整数值，小数位将被截断。若你想对结果进行四舍五入，可以使用java.lang.Math的round()方法：

```java
// operators/RoundingNumbers.java
// float 和 double 类型数据的四舍五入
public class RoundingNumbers {
    public static void main(String[] args) {
        double above = 0.7, below = 0.4;
        float fabove = 0.7f, fbelow = 0.4f;
        System.out.println(
        "Math.round(above): " + Math.round(above));
        System.out.println(
        "Math.round(below): " + Math.round(below));
        System.out.println(
        "Math.round(fabove): " + Math.round(fabove));
        System.out.println(
        "Math.round(fbelow): " + Math.round(fbelow));
    }
}
//Math.round(above): 1
//Math.round(below): 0
//Math.round(fabove): 1
//Math.round(fbelow): 0
```

​		因为round()方法是java.lang的一部分，所以我们无需通过import就可以使用。

### 类型提升

​		你会发现，如果对小于int的基本数据类型(char、byte和short)执行任何算术或按位操作，这些值会在执行操作之前类型提升为int，并且结果值的类型为int。若想重新使用较小的类型，必须使用强制转换。通常，表达式中最大的数据类型是决定表达式结果的数据类型。float类型和double型相乘，结果是double型的；int和long相加，结果是long型。

## Java没有sizeof

​		Java不需要sizeof()方法来满足这种需求，因为所有类型的大小在不同平台上是相同的。我么不必考虑这个层次的移植问题——Java本身就是一种“与平台无关”的语言。

## 运算符总结

​		在char、byte和short类型中，我么可以看到算术运算符的“类型转换”效果。我们必须要显示强制类型转换才能将结果重新赋值为原始类型。对于int类型的运算则不用转换，因为默认就是int型。虽然我们不再停下来思考这一切是否安全，但是两个大的int型整数相乘时，结果有可能超出int型的范围，这种情况下结果会发生溢出：

```java
// operators/Overflow.java
// 厉害了！数据溢出了！
public class Overflow {
    public static void main(String[] args) {
        int big = Integer.MAX_VALUE;
        System.out.println("big = " + big);
        int bigger = big * 4;
        System.out.println("bigger = " + bigger);
    }
}
//big = 2147483647
//bigger = -4
```

​		编译器没有报错或警告，运行时一切看起来都无异常。诚然，Java是优秀的，但是还不足够优秀。

​		对于char、byte或者short，混合赋值并不需要类型转换。即使为它们执行转换操作，也会获得与直接算术运算相同的结果。另外，省略类型转换可以使代码显得更加简练。总之，除boolean以外，其他任何两种类型间都可以进行类型转换。当我们向下转换类型时，需要注意结果的范围是否溢出，否则我们就很有可能在不知不觉中丢失精度。