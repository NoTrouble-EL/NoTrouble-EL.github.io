---
title: 工厂方法(Factory Method)
date: 2021-06-01 19:28:11
tags:
- DesignPatterns
categories: DesignPatterns
mathjax: true
---

​		定义一个创建对象的接口，但由子类决定要实例化哪个类。工厂方法把实例化操作推迟到子类。

​		在简单工厂中，创建对象的是另一个类，而在工厂方法中，是由子类来创建对象。

 <!-- more --> 

```java
package cn.xiaohupao.factorymethod;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 21:40
 */
public interface Product {

    /**
     * 产品描述
     */
    void print();
}
```

```java
package cn.xiaohupao.factorymethod;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 21:40
 */
public class ProductA implements Product{

    @Override
    public void print() {
        System.out.println("ProductA");
    }
}
```

```java
package cn.xiaohupao.factorymethod;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 21:41
 */
public class ProductB implements Product{
    @Override
    public void print() {
        System.out.println("ProductB");
    }
}
```

```java
package cn.xiaohupao.factorymethod;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 21:42
 */
public class ProductC implements Product{
    @Override
    public void print() {
        System.out.println("ProductC");
    }
}
```

```java
package cn.xiaohupao.factorymethod;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 21:43
 */
public interface Factory {
    /**
     * 创建产品的方法
     * @return 各种产品
     */
    Product createProduct();
}
```

```java
package cn.xiaohupao.factorymethod;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 21:46
 */
public class ProductAFactory implements Factory{
    @Override
    public Product createProduct() {
        return new ProductA();
    }
}
```

```java
package cn.xiaohupao.factorymethod;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 21:47
 */
public class ProductBFactory implements Factory{

    @Override
    public Product createProduct() {
        return new ProductB();
    }
}
```

```java
package cn.xiaohupao.factorymethod;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 21:49
 */
public class ProductCFactory implements Factory{
    @Override
    public Product createProduct() {
        return new ProductC();
    }
}
```

```java
package cn.xiaohupao.factorymethod;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/31 21:54
 */
public class Client {
    public static void main(String[] args) {
        Factory factoryA = new ProductAFactory();
        Product productA = factoryA.createProduct();
        productA.print();

        Factory factoryB = new ProductBFactory();
        Product productB = factoryB.createProduct();
        productB.print();
    }
}
```

​		在JDK中具体的应用为：Collection、Iterator、LinkedList、ArrayList、ListItr、Itr。
