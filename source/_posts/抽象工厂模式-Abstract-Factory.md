---
title: 抽象工厂模式(Abstract Factory)
date: 2021-06-01 20:30:54
tags:
- DesignPatterns
categories: DesignPatterns
mathjax: true
---

提供一个接口，用于创建相关的对象家族。具体工厂类可以创建多个“大类”对象，但是若增加产品类型还是需要修改抽象工厂和具体工厂，违反开闭原则。

 <!-- more --> 

```java
package cn.xiaohupao.abstractfactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 22:43
 */
public interface AbstractFactory {
    ProductA createProductA();
    ProductB createProductB();
}
```

```java
package cn.xiaohupao.abstractfactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 22:46
 */
public class createProductBy1 implements AbstractFactory{
    @Override
    public ProductA createProductA() {
        return new ProductA1();
    }

    @Override
    public ProductB createProductB() {
        return new ProductB1();
    }
}
```

```java
package cn.xiaohupao.abstractfactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 22:48
 */
public class createProductBy2 implements AbstractFactory{
    @Override
    public ProductA createProductA() {
        return new ProductA2();
    }

    @Override
    public ProductB createProductB() {
        return new ProductB2();
    }
}
```

```java
package cn.xiaohupao.abstractfactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 22:36
 */
public interface ProductA {

    /**
     * A类产品的不同实现
     */
    void print();
}
```

```java
package cn.xiaohupao.abstractfactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 22:38
 */
public class ProductA1 implements ProductA{
    @Override
    public void print() {
        System.out.println("1号实现产品A");
    }
}
```

```java
package cn.xiaohupao.abstractfactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 22:38
 */
public class ProductA2 implements ProductA{
    @Override
    public void print() {
        System.out.println("2号实现产品A");
    }
}
```

```java
package cn.xiaohupao.abstractfactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 22:37
 */
public interface ProductB {

    /**
     * 产品B的不同实现
     */
    void print();
}
```

```java
package cn.xiaohupao.abstractfactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 22:39
 */
public class ProductB1 implements ProductB{
    @Override
    public void print() {
        System.out.println("1号实现产品B");
    }
}
```

```java
package cn.xiaohupao.abstractfactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 22:40
 */
public class ProductB2 implements ProductB{
    @Override
    public void print() {
        System.out.println("2号实现产品B");
    }
}
```

