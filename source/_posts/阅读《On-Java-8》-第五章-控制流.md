---
title: 阅读《On Java 8》--第五章 控制流
date: 2021-04-23 13:58:25
tags:
- JAVASE
- 《On Java 8》
categories: JAVASE
mathjax: true
---

在Java中，涉及的关键字包括if-else、while、do-while、for、return、break和选择语句switch。Java并不支持备受诟病的goto。尽管如此，在Java中我们仍然可以进行类似的逻辑跳转，但较之经典的goto用法限制更多。

 <!-- more --> 

## true和false

​		所有的条件语句都利用条件表达式的“真”或“假”来决定执行路径。例如：a\==b。它利用了条件表达式来比较a和b的值是否相等。该表达式返回true和false：

```java
// control/TrueFalse.java
public class TrueFalse {
    public static void main(String[] args) {
        System.out.println(1 == 1);
        System.out.println(1 == 2);
    }
}
//true
//false 
```

​		注意：在Java中使用数值作为布尔值是非法的。如果想在布尔测试中使用一个非布尔值，那么首先需要使用条件表达式来产生boolean类型的结果，例如：if(a != 0)

## if-else

​		if-else语句是控制程序执行流程最基本的形式。其中else是可选的，因为可以有两种形式的if：

```java
if(Boolean-expression) 
    “statement” 
```

```java
if(Boolean-expression) 
    “statement”
else
  “statement”
```

​		布尔表达式(boolean-expression)必须生成boolean类型的结果，执行语句statement既可以是分号;结尾的一条简单语句，也可以是包含在大括号{}内的复合语句——封闭在大括号内的一组简单语句。

```java
// control/IfElse.java
public class IfElse {
  static int result = 0;
  static void test(int testval, int target) {
    if(testval > target)
      result = +1;
    else if(testval < target) // [1]
      result = -1;
    else
      result = 0; // Match
  }

  public static void main(String[] args) {
    test(10, 5);
    System.out.println(result);
    test(5, 10);
    System.out.println(result);
    test(5, 5);
    System.out.println(result);
  }
}
//1
//-1
//0
```

​		注意：else if 并非新关键字，它仅是else后紧跟的一条if语句。

## 迭代语句

​		while、do-while和for用来控制循环语句(有时也称迭代语句)。只有控制循环的布尔表达式计算结果为false，循环语句才会停止。

### while

​		while循环的形式是：

```java
while(Boolean-expression) 
  statement
```

​		执行语句之前在每一次循环前，判断布尔表达式返回值是否为true。下列可产生随机数，直到满足特定条件：

```java
// control/WhileTest.java
// 演示 while 循环
public class WhileTest {
  static boolean condition() {
    boolean result = Math.random() < 0.99;
    System.out.print(result + ", ");
    return result;
  }
  public static void main(String[] args) {
    while(condition())
      System.out.println("Inside 'while'");
    System.out.println("Exited 'while'");
  }
}
//true, Inside 'while'
//true, Inside 'while'
//true, Inside 'while'
//true, Inside 'while'
//true, Inside 'while'
//...________...________...________...________...
//true, Inside 'while'
//true, Inside 'while'
//true, Inside 'while'
//true, Inside 'while'
//false, Exited 'while'
```

​		其中condition()方法使用了Math库的静态方法random()。该方法的作用是产生0和1之间(包括0，但不包括1)的一个double值。

​		result的值是通过比较运算符\<产生的boolean类型的结果。当控制台输出boolean型值得，会自动将其转换为对应的文字形式true或false。此处while条件表达式代表：“仅在condition()返回false时体制循环”。

### do-while

​		do-while的格式如下：

```java
do 
    statement
while(Boolean-expression);
```

​		while和do-while之间的唯一区别是：即使条件表达式返回结果为false，do-while语句也至少会执行一次。在while循环体中，如布尔表达式首次返回的结果就为false，那么循环体内的语句不会被执行。实际应用这，while形式比do-while更为常用。

### for

​		for循环可能是最常用的迭代形式。该循环在第一次迭代之前执行初始化。随后，它会执行布尔表达式，并在每次迭代结束时，进行某种形式的步进。for循环的形式是：

```java
for(initialization; Boolean-expression; step)
  statement
```

​		初始化(initialization)表达式、布尔表达式(Boolean-expression)，或者步进(step)运算，都可以为空。每次迭代之前都会判断布尔表达式的结果是否成立。一旦计算结果为false，则跳出for循环体并继续执行后面的代码。每次循环结束时，都会执行一次步进。

​		for循环通常用于“计数”任务：

```java
// control/ListCharacters.java

public class ListCharacters {
  public static void main(String[] args) {
    for(char c = 0; c < 128; c++)
      if(Character.isLowerCase(c))
        System.out.println("value: " + (int)c +
          " character: " + c);
  }
}
//value: 97 character: a
//value: 98 character: b
//value: 99 character: c
//value: 100 character: d
//value: 101 character: e
//value: 102 character: f
//value: 103 character: g
//value: 104 character: h
//value: 105 character: i
//value: 106 character: j
//  ...
```

​		注意：变量c是在for循环执行时才被定义的，并不是在主方法的开头。c的作用域范围仅在for循环体内。

​		在Java和C++中，我们可以在整个块使用变量声明，并且可以在需要时才定义变量。

​		上例中使用了java.lang.Character包装类，该类不仅包含了基本类型char值，还封装了一些有用的方法。例如这里就用到了静态方法isLowerCase()来判断字符是否为小写。

### 逗号操作符

​		在Java中逗号运算符仅有一种用法：在for循环的初始化和步进控制中定义多个变量。我们可以使用逗号分隔多个语句，并顺序计算这些语句。注意：要求定义的变量类型相同：

```java
// control/CommaOperator.java

public class CommaOperator {
  public static void main(String[] args) {
    for(int i = 1, j = i + 10; i < 5; i++, j = i * 2) {
      System.out.println("i = " + i + " j = " + j);
    }
  }
}
//i = 1 j = 11
//i = 2 j = 4
//i = 3 j = 6
//i = 4 j = 8
```

​		上例中int类型声明包含了i和j。实际上，在初始化部分我们可以定义任意数量的同类型变量。注意：在Java中，仅允许for循环在控制表达式中定义变量。我们不能将此方法与其他的循环语句和选择语句中一起使用。同时，我们可以看到：无论在初始化还是在步进部分，语句都是顺序执行的。

## for-in语法

​		Java5引入了更为简洁的“增强版for循环”语法来操纵数组和集合。大部分文档也称其为for-each语法。

​		for-in无需你创建int变量和步进来控制循环计数：

```java
// control/ForInFloat.java

import java.util.*;

public class ForInFloat {
  public static void main(String[] args) {
    Random rand = new Random(47);
    float[] f = new float[10];
    for(int i = 0; i < 10; i++)
      f[i] = rand.nextFloat();
    for(float x : f)
      System.out.println(x);
  }
}
//0.72711575
//0.39982635
//0.5309454
//0.0534122
//0.16020656
//0.57799757
//0.18847865
//0.4170137
//0.51660204
//0.73734957
```

​		上例中我们展示了传统for循环的用法。接下来再来看for-in的用法：

```java
for(float x : f) {
```

​		这条语句定义了一个float类型的变量x，继而将每一个f的元素赋值给它。

​		任何一个返回数组的方法都可以使用for-in循环语法来遍历元素。例如String类有一个方法toCharArray()，返回值类型为char数组，我们很容易地在for-in循环中遍历它：

```java
// control/ForInString.java

public class ForInString {
  public static void main(String[] args) {
    for(char c: "An African Swallow".toCharArray())
      System.out.print(c + " ");
  }
}
//A n   A f r i c a n   S w a l l o w
```

​		for-in循环适用于任何可迭代(iterable)的对象。

​		通常，for循环语句都会在一个整型数值序列中步进：

```java
for(int i = 0; i < 100; i++)
```

​		正因如此，除非先创建一个int数组，否则我们无法使用for-in循环操作。在onjava包中封装了range()方法可以自动生成恰当的数组。

```java
// control/ForInInt.java

import static onjava.Range.*;

public class ForInInt {
  public static void main(String[] args) {
    for(int i : range(10)) // 0..9
      System.out.print(i + " ");
    System.out.println();
    for(int i : range(5, 10)) // 5..9
      System.out.print(i + " ");
    System.out.println();
    for(int i : range(5, 20, 3)) // 5..20 step 3
      System.out.print(i + " ");
    System.out.println();
    for(int i : range(20, 5, -3)) // Count down
      System.out.print(i + " ");
    System.out.println();
  }
}
//0 1 2 3 4 5 6 7 8 9
//5 6 7 8 9
//5 8 11 14 17
//20 17 14 11 8
```

​		range()方法已被重载(重载：同名方法，参数列表或类型不同)。上例中range()方法有多种重载形式：第一种产生从0至范围上限(不包含)的值；第二种产生参数一至参数二(不包含)范围内的整数值；第三种形式有一个步进值，因此它每次的增量为该值；第四种range()无参方法是该生成器最简单的版本。

​		for-in语法可以节省我们编写代码的时间。更重要的是，它提高了代码可读性以及更好地描述代码示意图而不是详细说明这操作细节。

## return

​		在Java中有几个关键字代表无条件分支，这意味无需任何测试即可发生。这些关键字包括return，break，continue和跳转到带标签语句的方法，类似于其它语言中的goto。

​		**return**关键字有两方面的作用：1.指定一个方法的返回值(在方法返回类型非void的情况下)；2.退出当前方法，并返回作用1中值。我们可以利用return的这些特点来改写上例IfElse.java文件中的test()方法：

```java
// control/TestWithReturn.java

public class TestWithReturn {
  static int test(int testval, int target) {
    if(testval > target)
      return +1;
    if(testval < target)
      return -1;
    return 0; // Match
  }

  public static void main(String[] args) {
    System.out.println(test(10, 5));
    System.out.println(test(5, 10));
    System.out.println(test(5, 5));
  }
}
//1
//-1
//0
```

​		这里不需要else，因为该方法执行到return就结束了。

​		如果在方法签名中定义了返回值类型为void，那么在代码执行结束时会有一个隐式的return。也就是说我们不用在总是在方法中显示地包含return语句。注意：如果你的方法声明的返回值类型为非void类型，那么则必须确保每个代码路径都返回一个值。

## break和continue

​		在任何迭代语句的主体内，都可以使用break和continue来控制循环的流程。其中break表示跳出当前循环体。而continue表示停止本次循环，开始下一次循环：

```java
// control/BreakAndContinue.java
// Break 和 continue 关键字

import static onjava.Range.*;

public class BreakAndContinue {
  public static void main(String[] args) {
    for(int i = 0; i < 100; i++) { // [1]
      if(i == 74) break; // 跳出循环
      if(i % 9 != 0) continue; // 下一次循环
      System.out.print(i + " ");
    }
    System.out.println();
    // 使用 for-in 循环:
    for(int i : range(100)) { // [2]
      if(i == 74) break; // 跳出循环
      if(i % 9 != 0) continue; // 下一次循环
      System.out.print(i + " ");
    }
    System.out.println();
    int i = 0;
    //  "无限循环":
    while(true) { // [3]
      i++;
      int j = i * 27;
      if(j == 1269) break; // 跳出循环
      if(i % 10 != 0) continue; // 循环顶部
      System.out.print(i + " ");
    }
  }
}
//0 9 18 27 36 45 54 63 72
//0 9 18 27 36 45 54 63 72
//10 20 30 40
```

​		在第一个输出中，这个for循环中，i的值永远不会达到100，因为一旦i等于74，break语句就会中断循环。通常，只有在不知道中断条件何时满足时，才需要break。因为i不能被9整除，continue语句就会使循环从头开始。如果能够整除，则将值显示出来。在第二个输出中，使用for-in语法，结果相同。在第三个输出中，无限循环while循环，循环内的break语句可终止循环。注意，continue语句可将控制权移回循环的顶部，而不会执行continue之后的任何操作。因此，只有当i的值被10整除时才会输出。在输出中，显示值0，因为0%9产生0。还有一种无限循环的形式：for(;;)。它在编译器看来，与while(true)无异，使用哪种完全取决于你的编程品味。

## 臭名昭著的goto

​		goto关键字很早就在程序设计语言中出现。事实上，goto起源于汇编(assembly language)语言中的程序控制：“若条件A成立，则跳到这里；否则就跳到那里”。

​		尽管**goto**仍是Java的一个保留字，但其并未被正式启用。可以说，Java中并不支持goto。然而，在break和continue这两个关键字的身上，我们仍能看出一些goto的影子。它们并不属于一次跳转，而是中断循环语句的一种方法。之所以把它纳入goto问题中一起讨论，是由于它们使用了相同的机制：标签。

​		“标签”是后面跟一个冒号的标识符：

```java
label1:
```

​		对Java来说，唯一用到标签的地方是在循环语句之前。进一步说，它实际需要紧靠在循环语句的前方——在标签和循环之间置入任何语句都是不明智的。而在循环之前设置标签的唯一理由是：我们希望在其中嵌套另一个循环或者一个开关。这是由于break和continue关键字通常只中断当前循环，若搭配标签一起使用，它们就会中断并跳转到标签所在的地方开始执行：

```java
label1:
outer-iteration { 
  inner-iteration {
  // ...
  break; // [1] 
  // ...
  continue; // [2] 
  // ...
  continue label1; // [3] 
  // ...
  break label1; // [4] 
  } 
}
```

​		在[1]break中断内部循环，并在外部循环结束。[2]continue移回内部循环起始处。但在条件3中，continue label1却同时中断内部循环及外部循环，并移回至label1处。[3]随后，它实际是继续循环，但却从外部循环开始。[4]break label1也会中断所有循环，并回到label1处，但并不重新进入循环。也就是说，它实际是完全中止了两个循环。

​		下面为for循环的一个例子：

```java
// control/LabeledFor.java
// 搭配“标签 break”的 for 循环中使用 break 和 continue

public class LabeledFor {
  public static void main(String[] args) {
    int i = 0;
    outer: // 此处不允许存在执行语句
    for(; true ;) { // 无限循环
      inner: // 此处不允许存在执行语句
      for(; i < 10; i++) {
        System.out.println("i = " + i);
        if(i == 2) {
          System.out.println("continue");
          continue;
        }
        if(i == 3) {
          System.out.println("break");
          i++; // 否则 i 永远无法获得自增 
               // 获得自增 
          break;
        }
        if(i == 7) {
          System.out.println("continue outer");
          i++;  // 否则 i 永远无法获得自增 
                // 获得自增 
          continue outer;
        }
        if(i == 8) {
          System.out.println("break outer");
          break outer;
        }
        for(int k = 0; k < 5; k++) {
          if(k == 3) {
            System.out.println("continue inner");
            continue inner;
          }
        }
      }
    }
    // 在此处无法 break 或 continue 标签
  }
}
//i = 0
//continue inner
//i = 1
//continue inner
//i = 2
//continue
//i = 3
//break
//i = 4
//continue inner
//i = 5
//continue inner
//i = 6
//continue inner
//i = 7
//continue outer
//i = 8
//break outer
```

​		注意break会中断for循环，而且在抵达for循环的末尾之前，递增表达式不会执行。由于break跳过了递增表达式，所以递增会在i\==3的情况下直接执行。在\==7的情况下，continue outer语句也会到达循环顶部，而且也会跳过递增，所以它也是直接递增的。

​		如果没有break outer语句，就没有办法在一个内部循环里找到出外部循环的路径。这是由于break本身只能中断最内层的循环(对于continue同样如此)。当然，若想在中断循环的同时推出方法，简单地用一个return即可。

​		下面的例子为带标签的break以及continue语句在while循环中的用法：

```java
// control/LabeledWhile.java
// 带标签的 break 和 conitue 在 while 循环中的使用

public class LabeledWhile {
  public static void main(String[] args) {
    int i = 0;
    outer:
    while(true) {
      System.out.println("Outer while loop");
      while(true) {
        i++;
        System.out.println("i = " + i);
        if(i == 1) {
          System.out.println("continue");
          continue;
        }
        if(i == 3) {
          System.out.println("continue outer");
          continue outer;
        }
        if(i == 5) {
          System.out.println("break");
          break;
        }
        if(i == 7) {
          System.out.println("break outer");
          break outer;
        }
      }
    }
  }
}
//Outer while loop
//i = 1
//continue
//i = 2
//i = 3
//continue outer
//Outer while loop
//i = 4
//i = 5
//break
//Outer while loop
//i = 6
//i = 7
//break outer
```

​		同样的规则亦适用于while：

* 简单一个cotinue会退回最内层循环的开始(顶部)，并继续执行。
* 带有标签的cotinue会到达标签的位置，并重新进入紧接在那个标签后面的循环。
* break会中断当前循环，并移离当前标签的末尾。
* 带标签的break会中断当前循环，并移离由那个标签指示的循环的末尾。

大家要记住的重点是：在Java里需要使用标签的唯一理由就是因为有循环嵌套存在，而且想从多层嵌套中break或continue。

​		break和continue标签在编码中的使用频率相对较低。

## switch

​		switch有时也被规划为一种选择语句。根据整数表达式的值，switch语句可以从一系列代码中选择出一段去执行，它的格式如下：

```java
switch(integral-selector) {
    case integral-value1 : statement; break;
    case integral-value2 : statement;    break;
    case integral-value3 : statement;    break;
    case integral-value4 : statement;    break;
    case integral-value5 : statement;    break;
    // ...
    default: statement;
}

```

​		其中intergral-selector(整数选择因子)是一个能够产生整数值的表达式，switch能够将这个表达式的结果与每个integral-value(整数值)相比较。若发现相符的，就执行对应的语句(简单或复合语句，其中并不需要括号)。若没有发现相符的，就执行default语句。

​		在上面的定义中，大家会注意到每个case均以一个break结尾。这样可使执行流程转至switch主体的末尾。这是构建switch语句的一种传统方式，但break是可选的。若省略break，会继续执行后面的case语句的代码，直到遇到一个break为止。通常我们不想出现这种情况，但对于有经验的程序员来说，也许能够善加利用。注意最后的default语句没有break，因为执行流程已到了break的跳转目的地。当然，如果考虑编程风格方法的原因，完全可以在default语句的末尾放置一个break，尽管它并没有任何实际的作用。

​		switch语句是一种实现对路选择的干净利落的一种方式(比如从一些列执行路径中挑选一个)。但它要求使用一个选择因子，并且必须是int或char那样的整数值。例如，假若将一个字串或浮点数作为选择因子使用，那么它们在switch语句里是不会工作的。对于非整数类型(Java7以上版本中的String型除外)，则必须使用一些列if语句。

​		下面这个例子可随机生成字母，并判断它们是元音还是辅音字母：

```java
// control/VowelsAndConsonants.java

// switch 执行语句的演示
import java.util.*;

public class VowelsAndConsonants {
  public static void main(String[] args) {
    Random rand = new Random(47);
    for(int i = 0; i < 100; i++) {
      int c = rand.nextInt(26) + 'a';
      System.out.print((char)c + ", " + c + ": ");
      switch(c) {
        case 'a':
        case 'e':
        case 'i':
        case 'o':
        case 'u': System.out.println("vowel");
                  break;
        case 'y':
        case 'w': System.out.println("Sometimes vowel");
                  break;
        default:  System.out.println("consonant");
      }
    }
  }
}
//y, 121: Sometimes vowel
//n, 110: consonant
//z, 122: consonant
//b, 98: consonant
//r, 114: consonant
//n, 110: consonant
//y, 121: Sometimes vowel
//g, 103: consonant
//c, 99: consonant
//f, 102: consonant
//o, 111: vowel
//w, 119: Sometimes vowel
//z, 122: consonant
//  ...
```

​		由于Random.nextInt(26)会产生0到25之间的一个值，所以在其上加上一个偏移量a，即可产生小写字母。在case语句中，使用单引号引起的字符也会产生用于比较的整数值。

​		请注意case语句能够堆叠在一起，为一段代码形成多重匹配，即只要符合多种条件的一种，就执行那段特别的代码。这时也应该注意将break语句置于特定的case末尾，否则控制流程会继续往下执行，处理后面的case：

```java
int c = rand.nextInt(26) + 'a';
```

​		此处Random.nextInt()将产生0\~25之间的一个随机int值，它将被加到a上。这表示a将自动被转型为int以执行加法。为了把c当作字符da'yin，必须将其转型为char；否则，将会输出整数。

## switch字符串

​		Java7增加了在字符串上switch的用法：

```java
// control/StringSwitch.java

public class StringSwitch {
  public static void main(String[] args) {
    String color = "red";
    // 老的方式: 使用 if-then 判断
    if("red".equals(color)) {
      System.out.println("RED");
    } else if("green".equals(color)) {
      System.out.println("GREEN");
    } else if("blue".equals(color)) {
      System.out.println("BLUE");
    } else if("yellow".equals(color)) {
      System.out.println("YELLOW");
    } else {
      System.out.println("Unknown");
    }
    // 新的方法: 字符串搭配 switch
    switch(color) {
      case "red":
        System.out.println("RED");
        break;
      case "green":
        System.out.println("GREEN");
        break;
      case "blue":
        System.out.println("BLUE");
        break;
      case "yellow":
        System.out.println("YELLOW");
        break;
      default:
        System.out.println("Unknown");
        break;
    }
  }
}
//RED
//RED
```

​		一旦理解了switch，你会明白这其实就是一个逻辑扩展的**语法糖**。新的编码方式能使得结果更清晰，更易于理解和维护。

​		作为switch字符串的第二个例子，我们重新访问Math.random()。它是否产生从0到1的值，包括还是不包括1呢？在数学术语中，它属于(0,1)、[0,1)、(0,1]、[0,1]中的哪种呢？

​		下面是一个可能提供答案的测试程序。所有命令行参数都作为String对象传递，因此我们可以switch参数来决定要做什么。那么问题来了：如果用户不提供参数，索引到args的数组就会导致程序失败。解决这个问题，我们需要预先检查数组的长度，若长度为0，则使用空字符串“”代替；否则，选择args数组中的第一个元素：

```java
// control/RandomBounds.java

// Math.random() 会产生 0.0 和 1.0 吗？
// {java RandomBounds lower}
import onjava.*;

public class RandomBounds {
  public static void main(String[] args) {
    new TimedAbort(3);
    switch(args.length == 0 ? "" : args[0]) {
      case "lower":
        while(Math.random() != 0.0)
          ; // 保持重试
        System.out.println("Produced 0.0!");
        break;
      case "upper":
        while(Math.random() != 1.0)
          ; // 保持重试
        System.out.println("Produced 1.0!");
        break;
      default:
        System.out.println("Usage:");
        System.out.println("\tRandomBounds lower");
        System.out.println("\tRandomBounds upper");
        System.exit(1);
    }
  }
}
java RandomBounds lower 
// 或者
java RandomBounds upper
```

​		使用onjava包下的TimedAbort类可使程序在三秒后中止。从结果来看，似乎Math.random()产生的随机值里不包含0.0或1.0。这就是该测试容易混淆的地方：若要考虑0至1之间有所不同double数值的可能性，那么这个测试的耗费的时间可能超出了一个人的寿命。这里直接给出正确的结果：Math.random()的结果范围包含0.0，不包含1.0。在数学术语中，可用[0,1)来表示。由此可知，我们必须小心分析实验并了解它们的局限性。