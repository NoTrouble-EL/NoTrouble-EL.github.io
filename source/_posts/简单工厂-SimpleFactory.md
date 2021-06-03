---
title: 简单工厂(SimpleFactory)
date: 2021-05-25 22:33:51
tags:
- DesignPatterns
categories: DesignPatterns
mathjax: true
---

​		在创建对象时不向客户暴露内部细节的暴露，并提供一个创建对象的通用接口。

​		简单工厂就是把实例化的操作单独放在一个类中，这个类就成为简单工厂类，让简单工厂类来决定应该用哪个具体子类来实例化。

​		简单工厂的好处是将客户类和具体的子类的实现解耦，客户类不再需要知道有哪些子类以及应当实例化哪个子类。客户类往往有多个，若不使用简单工厂，那么所有的客户类都要知道所有子类的细节。缺点是：再增加子类后，在简单工厂的方法中的逻辑都需要修改。

 <!-- more --> 

```java
package cn.xiaohupao.simpleFactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 21:40
 */
public interface Product {
    /**
     * 描述产品的方法
     */
    void print();
}
```

```java
package cn.xiaohupao.simpleFactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 21:42
 */
public class ProductA implements Product{
    @Override
    public void print() {
        System.out.println("ProductA");
    }
}
```

```java
package cn.xiaohupao.simpleFactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 21:43
 */
public class ProductB implements Product{
    @Override
    public void print() {
        System.out.println("ProductB");
    }
}
```

```java
package cn.xiaohupao.simpleFactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 21:44
 */
public class ProductSimpleFactory {
    public static Product createProduct(String type){
        if ("A".equals(type)){
            return new ProductA();
        }else {
            return new ProductB();
        }
    }
}
```

```java
package cn.xiaohupao.simpleFactory;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 21:47
 */
public class Client {
    public static void main(String[] args) {

        Product productA = ProductSimpleFactory.createProduct("A");
        productA.print();
        Product productB = ProductSimpleFactory.createProduct("B");
        productB.print();
    }
}
```

​		在JDK中的应用：DateFormat类中，可以根据传入的参数不同，可以判断timeStype、dateStyle等等返回不同的DateFormat的子类。
