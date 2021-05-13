---
title: Java错题
date: 2021-03-31 21:56:34
tags:
- JAVA
categories: JAVA基础
mathjax: true
---

## 20210331

```txt
下面的类哪些可以处理Unicode字符？
-------------------------------
A InputStreamReader
B BufferedReader
C Writer
D PipedInputStream
--------------------------------
正确答案：ABC
解析：
D选项是处理字节流的
```

 <!-- more --> 

```txt
关于JSP生命周期的叙述，下列哪些为真？
----------------------------------------
A JSP会解释成Servlet源文件，然后编译成Servlet类文件
B 每当用户端运行JSP时，jsp init()方法都会运行一次
C 每当用户端运行JSP时，jsp service()方法都会运行一次
D 每当用户端运行JSP时，jsp destroy()方法都会运行一次
-----------------------------------------
正确答案：AC
解析：
JSP生命周期所走过的几个阶段：
* 编译阶段：servlet容器编译servlet源文件，生成servlet类
* 初始化阶段：加载与JSP对应的servlet类，创建其实例，并调用它的初始化方法
* 执行阶段：调用与JSP对应的servlet实例的服务方法
* 销毁阶段：调用与JSP对应的servlet实例的销毁方法，然后销毁servlet实例
```

```txt
Consider the following code:
Integer s = new Integer(9);
Integer t = new Integer(9);
Long u = new Long(9);
which test would return true?
--------------------------------
A (s==u)
B (s==t)
C (s.equals)
D (s.equals(9))
E (s.equals(new Integer(9)))
--------------------------------
正确答案：CDE
解析：
* int和int之间，用==比较，肯定为true。基本数据类型没有equals方法；
* int和Integer比较，Integer会自动拆箱，==和equasl都肯定为true；
* int和new Integer比较，Integer会自动拆箱，调用intValue方法，所以==和equals都肯定为true
* Integer 和Integer比较的时候，由于直接赋值会进行自动装箱。所以当值在[-128,127]中的时候，由于值缓存在IntegerCache中，那么当赋值在这个区间的时候，不会创建新的Integer对象，而是直接从缓存中获取已经创建好的Integer对象。而当大于这个区间的时候，会直接new Integer；
* 当Integer和Integer进行==比较的时候，在[-128.127]区间的时候，为true。不在这个区间，则为false；
* 当Integer和Integer进行equals比较的时候，由于Integer的equals方法进行了重写，比较的是内容，所以为true；
* Integer和new Integer进行==比较的话，肯定为false，Integer和new Integer进行equals进行比较的话，肯定为true；
* new Interger和new Integer进行==比较的时候，肯定为false，进行equals比较的时候，肯定为true原因是new的时候，会在堆中创建对象，分配的地址不同，==比较的是内存地址，所以肯定不同；
* 装箱过程是通过调用包装器的valueOf方法实现的，拆箱过程是通过调用包装器的xxxValue方法实现的；
* 总结：Byte、Short、Integer、Long这几个类的valueOf方法实现类似的。所以在[-128,127]区间内，==比较的时候，值总是相等的，在这个区间外是不等的，而Float和Double则不相等，Boolean的值总是相等的。
```

```txt
在Java语言中，下列关于字符集编码(Character set encoding) 和国际化(i18n)的问题，哪些是正确的？
-------------------------------------------------------
A 每个中文字符占2个字节，每个英文字符占用1字节
B 假设数据库中的字符是GBK编码的，那么显示数据库数据的网页也必须是GBK编码的
C Java的char类型，通常以UTF-16 Big Endian的方式保存一个字符
D 实现国际化应用常用的手段是利用ResourceBundle类
-------------------------------------------------------------
正确答案：CD
解析：
A Java一律采用Unicode编码方式，每个字符无论中文还是英文都占用2个字节
B 不同的编码之间是可以转换的
C Java虚拟机中通常使用UTF-16的方式保存一个字符
D ResourceBundle能够根据Local的不同，选择性的读取Local对应后缀的properties文件，以达到国际化的目的
```

## 20210402

```txt
关于ASCII码和ANSI码，以下说法不正确的是？
-------------------------------------
A 标准ASCII码只使用7个bit
B 在简体中文的Windows系统中，ANSI就是GB2312
C ASCII码是ANSI码的子集
D ASCII码都是可以可打印字符
-------------------------------------
正确答案：D
解析：
* 标准ASCII只是用7个bit，扩展的ASCII使用8个bit
* ANSI通常使用0x00~0x7f范围的1个字节来表示1个英文字符。超出此范围的使用0x80~0xFFFF来编码，即扩展的ASCII编码。不同的ANSI编码之间互不兼容。在简体中文的windows操作系统中，ANSI编码代码GBK编码；在繁体中文Windows操作系统中，ANSI代表Big5；在日文Windows操作系统中，ANSI编码代表Shift_JIS编码。
* ANSI通常使用0x00~0x7f范围的一个字节来表示1个英文字符，即ASCII码
* ASCII码包含一些特殊空字符
```

## 20210404

```txt
事务隔离级别是由谁实现的？
-------------------------
A Java应用程熙
B Hibernate
C 数据库系统
D JDBC驱动程序
--------------------------
正确答案：C
解析：
* 事务隔离级别是由数据库系统实现，是数据库系统本身的一个功能
```

```txt
当我们需要所有线程都执行到某一处，才进行后面的代码执行我们可以使用？
------------------------------------------------------------
A CountDownLatch
B CyclicBarrier
C Semaphore
D Future
-------------------------------------------------------------
正确答案：A
解析：
* CountDownLatch允许一个线程或多个线程等待特定情况，同步完成线程中其他任务
* CyclicBarrier和CountDownLatch一样可以协同多个线程，让指定数量的线程等待其他所有线程满足某些条件之后才继续执行
* Semaphore通常叫它信号量，可以用来控制同时访问特定资源的线程数量，通过协调各个线程，以保证合理的使用资源
* Future表示一个可能还没有完成的异步任务的结果，针对这个结果可以添加Callback以便在任务执行成功或失败后作出相应的操作
```

```txt
下面的对象创建方法中哪些会调用构造方法？
-----------------------------------
A new语句创建对象
B 调用Java.io.ObjectInputStream的readObject方法
C java反射机制使用java.lang.Class或java.lang.reflect.Constructor的newInstance()方法
D 调用对象的clone()方法
-------------------------------------
正确答案：AC
解析：
* 构造函数的作用是完成对象的初始化
* B和D中，对象的初始化并不是通过构造函数完成的，而是读取别的内存中的对象的各个域来完成的
* new和反射都可以创建对象，但需要注意的是它们创建对象时执行的构造函数的类型不一样
* new方式创建对象，其初始化过程中可以执行带参的构造器，也可以执行无参构造器
* 反射，通过静态的newInstance()方法，其执行的构造器一定是无参构造器。
```

```txt
下面哪些赋值语句是正确的？
---------------------------
A long test=012
B float f=-412
C int other=(int)true
D double d=0x12345678
E byte b=128
---------------------------
正确答案：ABD
解析：
* 012默认为int类型，赋给long型没错
* -412默认为int类型，赋给float没错
* C中布尔类型无法强制转换为int或其他数值类型
* 0x12345678默认应该是int型，赋值给double没错
* E中128>127默认为int类型，然而byte默认范围为-128~127(-2^7~2^7-1)
```

## 20210405

```txt
JSP表达式的写法：
---------------
A <% expression %>
B <=% expression %>
C <%= expression %>
D <expression/>
----------------
正确答案：C
解析：
* jsp表达式：<%=变量或表达式%>
* jsp脚本：<% java代码 %>
* jsp声明：<%!变量或方法 %>
* jsp注释：<%!--jsp注释--%>
```

```txt
下面关于变量及其范围的陈述不正确的是？
----------------------------------
A 实例变量是类的成员变量
B 实例变量用关键字static声明
C 在方法中定义的局部变量在该方法被执行时创建
D 局部变量在使用前必须被初始化
-----------------------------------
正确答案：BC
解析：
* 类的成员变量包括实例变量和类变量(静态变量)，成员方法包括实例方法和类方法(静态方法)
* 类变量(静态变量)用关键字static声明
* 方法中的局部变量在方法被调用加载时开始入栈时创建，方法入栈创建栈帧包括局部变量表操作数栈，局部变量表存放局部变量，并非在执行该方法时被创建
* 局部变量被使用前必须初始化
```

## 20210406

```txt
以下代码段执行后的输出结果为：
public class Test{
public static void main(String[] args){
System.out.println(test());
}
private static int test(){
int temp = 1;
try{
System.out.println(temp);
return ++temp;
}catch(Exception e){
System.out.println(temp);
return ++temp;
}finally{
++temp;
System.out.println(temp);
}
}
}
--------------------------
A 1,2,2
B 1,2,3
C 1,3,3
D 1,3,2
-------------------------
正确答案：D
解析：
* 输出try里面的初始temp：1；temp=2；保存return里面temp的值：2；执行finally的语句temp：3，输出temp：3；返回try中的return语句，返回存在里面的temp的值：2；输出temp:2。
```

```txt
Java8中，下面哪个类用到了解决哈希冲突的开放定址法
---------------------------------------------
A LinkedHashSet
B HashMap
C ThreadLocal
D TreeMap
---------------------------------------------
正确答案：C
解析：
* HashMap链地址法
* ThreadLocal开放式地址
```

```txt
下列哪些操作会使线程释放锁资源？
----------------------------
A sleep()
B wait()
C join()
D yield()
----------------------------
正确答案：BC
解析：
* sleep() 在指定时间内让当前正在执行的线程暂停执行，但不会释放“锁标志”。不推荐使用。sleep()使当前线程进入阻塞状态，在指定时间内不会执行。
* wait() 在其他线程调用对象的notify或nitifyAll方法前，导致当前线程等待。线程会释放掉它所占有的"锁标志"，从而使别的线程有机会抢占该锁。当前线程必须拥有当前对象锁。如果当前线程不是此锁的拥有者，会抛出IllegalMonitorStateException异常。唤醒当前对象锁的等待线程使用notify或notifyAll方法，也必须拥有相同的对象锁，否则也会抛出IllegalMonitorStateException异常。waite()和notify()必须在synchronized函数或synchronized block中进行调用。如果non-synchronized block中进行调用，虽然能通过编译，但在运行时会发生IllegalMonitorStateException的异常。
* yield() 暂停当前正在执行的线程对象。yield()只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。yield()只能使同优先级或更高优先级的线程有执行的机会。
* join() 等待该线程终止。等待调用join方法的线程结束，再继续执行。如t.join()主要用于等待t线程运行结束，如无此句，main则会执行完毕，导致结果不可预测。
```

## 20210407

```txt
String s = new String("xyz");创建了几个StringObject？
----------------------------------------------------
A 两个或一个都有可能
B 两个
C 一个
D 三个
----------------------------------------------------
正确答案：A
解析：
* String对象的两种创建方式：
* 第一种方式：String str1 = "aaa";是在常量池中获取对象("aaa"属于字符串字面量，u因此编译时期会在常量池中创建一个字符串对象)
* 第二种方式：String str2 = new String("aaa")；一共会创建两个字符串对象一个在堆中，一个在常量池中(前提是常量池中还没有"aaa"字符串对象)
```

```txt
问这个程序的输出结果是什么？
-------------------------
package Wangyi;
class Base
{
    public void method()
    {
        System.out.println("Base");
    } 
}
class Son extends Base
{
    public void method()
    {
        System.out.println("Son");
    }
    
    public void methodB()
    {
        System.out.println("SonB");
    }
}
public class Test01
{
    public static void main(String[] args)
    {
        Base base = new Son();
        base.method();
        base.methodB();
    }
}
-------------------------------
A Base SonB
B Son SonB
C Base Son SonB
D 编译不通过
-------------------------------
正确答案：D
解析：
* 多态问题，编译看左边，运行看右边。编译的时候没有该方法，运行的时候结果看new的对象是谁，就调用的谁。
```

```txt
下面选项中，哪些是interface中合法的定义？
-------------------------------------
A public void mian(String[] args);
B private int getSum();
C boolean setFlag(Boolean[] test);
D public float get(int x);
-------------------------------------
正确答案：ACD
解析：
* interface中的默认方法为public abstract的，变量默认为public static final
* JDK8以后，允许我们在接口中定义static方法和default方法
```

## 20210408

```txt
执行以下代码的输出结果是？
----------------------
public class Demo{
　public static void main(String args[]){
　　　int num = 10;
　　　System.out.println(test(num));
}
public static int test(int b){
　　　try
　　　{
　　　　b += 10;
　　　　return b;
　　　}
　　　catch(RuntimeException e)
　　　{
　　　}
　　　catch(Exception e2)
　　　{
　　　}
　　　finally
　　　{
　　　　b += 10;
　　　　return b;
　　　}
　　}
}
-------------------
A 10
B 20
C 30
D 40
-------------------
正确答案：C
解析：
* 如果finally块中有return语句的话，它将覆盖掉函数中其他return语句。
```

```txt
URL u = new URL("http://www.123.com");如果www.123.com不存在，则返回
------------------------------------------------------------------
A http://www.123.com
B ""
C null
D 抛出异常
------------------------------------------------------------------
正确答案：A
解析：
* new URL()时必须捕获检查异常，这个异常是由于字符串的格式和URL不符合导致的，与网址是否存在无关。URL的toStream方法返回字符串，无论网址是否存在。
```

```tet
关于Java集合下列说法不正确的有哪些？
--------------------------------
A hashSet它是线程安全的，不允许存储相同的对象
B ConcurrentHashMap它是线程安全的，其中存储的键的对象可以重复，值对象不能重复
C Collection接口是List接口和Set接口的父接口，通常情况下不被直接使用
D ArrayList线程安全的，允许存放重复对象
-----------------------------------
正确答案：ABD
解析：
* HashSet不是线程安全的，属于Set接口下的实现类，Set下的实现类特征就是无序，不允许存储相同的对象
* ConcurrentHashMap它是线程安全的的，其中存储的值对象可以重复键对象不能重复
* Collection接口是List接口和Set接口的父接口，通常情况下不被直接使用
* ArrayList线程不安全的，底层是数组实现的，允许存放重复对象
```

```txt
以下哪些JVM的垃圾回收方式采用的是复制算法回收？
------------------------------------------
A 新生代串行收集器
B 老年代串行收集器
C 并行收集器
D 新生代并行回收收集器
E 老年代并行回收收集器
F cms收集器
------------------------------------------
正确答案：AD
解析：
```







