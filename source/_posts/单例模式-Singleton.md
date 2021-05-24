---
title: 单例模式(Singleton)
date: 2021-05-23 22:44:35
tags:
- DesignPatterns
categories: DesignPatterns
mathjax: true
---

在运行时确保一个类只有一个实例，并提供该实例的全局访问点。

​		在多种写法中需要考虑到以下几点：1.是否线程安全；2.是否懒加载；3.是否能通过反射，序列化破坏

 <!-- more --> 

## 饿汉式

​		饿汉式实现的单例模式不会产生线程不安全的问题。但是直接实例化的方式在类加载就初始化了，可能占用不必要的内存。

```java
package cn.xiaohupao.singleton;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/23 17:12
 *
 * 单例模式的实现方式：饿汉式
 * 特点：在类声明的时候即完成了初始化
 * 优点：实现简单，执行效率高，线程安全
 * 缺点：类加载时就初始化，可能占用不必要的内存
 */
public class Hungry {

    /**
     * 若要被静态方法所使用，该成员变量也必须为static
     */
    private static final Hungry HUNGRY = new Hungry();

    /**
     * 构造方法私有化
     */
    private Hungry(){}

    /**
     * 公有静态方法来获取对象，公共保证外部能访问；静态保证通过类名来访问。
     * @return 唯一对象
     */
    public static Hungry getInstance(){
        return HUNGRY;
    }
}
```

## 懒汉式——线程不安全

​		在该实现中，私有静态变量被延迟初始化——在使用时才初始化。如果没有用到该类，则不会实例化lazyMan，从而节约资源。但是该实现的单例模式在多线程环境下将会被破坏(线程不安全的)。如果多个线程同时进入到判断条件if (lazyMan == null)时，并且此时lazyMan为null，那么将会有多个线程执行实例化操作即：lazyMan = new LazyMan()；这将导致实例化多次lazyMan。

```java
package cn.xiaohupao.singleton;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/23 17:35
 *
 * 单例模式的实现方式：懒汉式——线程不安全
 * 特点：在使用的时候才初始化，即懒加载
 * 优点：避免了不必要的初始化
 * 缺点：线程不安全
 */

public class LazyMan {

    /**
     * 若要被静态方法所使用，该成员变量也必须为static
     */
    private static LazyMan lazyMan;

    /**
     * 构造方法私有化
     */
    private LazyMan(){}

    /**
     * 公有静态方法来获取对象，公共保证外部能访问；静态保证通过类名来访问。
     * @return 唯一的对象
     */
    public static LazyMan getInstance(){
        if (lazyMan == null){
            lazyMan = new LazyMan();
        }

        return lazyMan;
    }
}
```

## 懒汉式——线程安全之同步方法

​		在getInstance方法上加锁，那么在一个时间点只能有一个线程进入到该方法，从而避免了多次实例化lazyMan2。但当一个线程进入该方法之后，其他线程都必须等待，即使当lazyMan2已经被实例化后，也需要等待。这将让线程阻塞时间过长，有一定的性能损耗。

```java
package cn.xiaohupao.singleton;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/23 18:16
 *
 * 单例模式的实现方式：懒汉式——线程安全
 * 特点：在使用的时候才初始化，即懒加载
 * 优点：避免了不必要的初始化，线程安全
 * 缺点：使用synchronized关键字对操作加锁，有一定的性能损耗，每一次获取实例都需要加锁
 */
public class LazyMan2 {

    private static LazyMan2 lazyMan2;

    private LazyMan2(){}

    public synchronized static LazyMan2 getInstance(){
        if (lazyMan2 == null){
            lazyMan2 = new LazyMan2();
        }

        return lazyMan2;
    }
}
```

## 懒汉式——线程安全之双重检验锁(DCL)

​		在该实现中，lazyMan3只需要被实例化一次，在这之后的判断不会涉及到多线程排队取对象的问题。加锁的部分只对实例化的代码，只有当lazyMan3没有被实例化时，才需要进行加锁。双重检验锁先判断lazyMan3是否已经被实例化，如果没有被实例化，那么才对实例化语句进行加锁。

​		注意，对lazyMan3采用volatile关键字修饰也是十分有必要的。我们注意在执行实例化操作时，使用构造方法不是原子性操作。构造方法的可以分解为三步执行：1.为lazyMan3分配内存空间；2.初始化lazyMan3；3.将lazyMan3指向分配的内存地址。但是由于JVM具有指令重排的特性，执行顺序有可能为1->3->2，此时当第二个线程调用getInstance()后发现lazyMan3不为空，因此返回lazyMan3，但此时lazyMan3还未被初始化。

​		使用volatile关键字可以禁止JVM的指令重排，保证在多线程环境中也能正常运行。

```java
package cn.xiaohupao.singleton;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/23 19:29
 *
 * 单例模式的实现方式：懒汉式——双重锁检查DCL
 * 特点：使用volatile关键字保证底层指令执行顺序
 * 优点：在入口处判断null，可以省去每次加锁的耗费，提升性能
 */
public class LazyMan3 {

    private volatile static LazyMan3 lazyMan3;

    private LazyMan3(){}

    public static LazyMan3 getInstance(){
        //第一次判断没有同步块，不会涉及到多线程排队取得对象的问题
        if (lazyMan3 == null){
            synchronized (LazyMan3.class){
                //第二次判断是为了让并发的线程没有机会创建第二个对象的可能
                if (lazyMan3 == null){
                    //不是原子性操作
                    lazyMan3 = new LazyMan3();
                    /**
                     * 1.分配内存空间
                     * 2.执行构造方法，初始化对象
                     * 3.把这个对象指向这个空间
                     *
                     * 这将会导致由指令重排造成的问题，当还没有完成构造时，导致其他线程直接获得未构造的对象
                     * 解决方法：在成员变量上加上关键字volatile防止指令重排
                     */
                }
            }
        }

        return lazyMan3;
    }
}
```

## 静态内部类实现

​		当Holder类被加载时，静态内部类SingletonHolder没有被加载进内存。只有当调用getInstance()方法从而触发SingletonHolder.HOLDER时SingletonHolder才会被加载，此时HOLDER初始化实例，并且JVM保证HOLDER只被实例化一次。这种方法不仅具有延迟初始化的好处，而且JVM提供了对线程安全的支持。

```java
package cn.xiaohupao.singleton;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/23 20:02
 *
 * 单例模式的实现方式：静态内部类
 * 特点：在类内部有一个静态内部类
 * 优点：懒加载，减少无效内存占用。不需要锁也能保证线程安全
 * 缺点：无法传参
 */
public class Holder {

    private static class SingletonHolder{
        private static final Holder HOLDER = new Holder();
    }

    private Holder(){}

    public static Holder getInstance(){
        return SingletonHolder.HOLDER;
    }
}
```

​		上述的所有方法都可以通过反射机制来破坏单例模式。若想防止反射攻击，与序列化破坏单例模式。反射机制可以通过setAccessible()方法可以将私有构造器的访问级别设置为public，然后调用构造函数从而实现实例化对象，若要防止这种攻击，可以在构造函数中添加防止多次实例化的代码。

## 使用枚举类实现单例

​		使用枚举类实现单例可以防止通过反射机制和序列化来破坏单例。但是枚举类无法实现懒加载，它在程序启动之初，那就已经把这个内部的实例完全构建好来提供给使用者。

```java
package cn.xiaohupao.singleton;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/23 20:51
 *
 * 单例模式的实现方式：枚举
 * 特点：使用枚举对象达到单例的效果
 * 优点：实现简洁，无偿地提供了序列化机制，线程安全，抵御反射攻击
 * 缺点：无法继承超类
 */
public enum EnumSingle {
    /**
     *
     */
    INSTANCE;
}
```

