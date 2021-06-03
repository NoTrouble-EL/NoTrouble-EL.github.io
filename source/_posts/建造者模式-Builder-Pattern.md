---
title: 建造者模式(Builder Pattern)
date: 2021-06-01 22:57:15
tags:
- DesignPatterns
categories: DesignPatterns
mathjax: true
---

​		封装一个对象的构造过程，并允许按步骤构造。

​		建造者模式(Builder Pattern)：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。建造者模式是一种对象创建型模式。建造者模式一步一步创建一个复杂的对象，它允许用户只通过指定复杂对象的类型和内容就可以构建它们，用户不需要知道内部的具体构建细节。

​		主要作用：可以在用户不知道对象的建造过程和细节的情况下就可以直接创建复杂的对象。

 <!-- more --> 

​		建造者模式具体的例子有两种实现方式：

* 通过静态内部类方式实现零件无序装配
* 通过Client、Director、Builder和Product形成的建造者模式

## 通过静态内部类方式实现零件无序装配

```java
package cn.xiaohupao.builder;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/1 21:35
 */
public class Computer {

    private final String cpu;
    private final String ram;
    private final int usbCount;
    private final String keyboard;
    private final String display;

    private Computer(Builder builder){
        this.cpu = builder.cpu;
        this.ram = builder.ram;
        this.usbCount = builder.usbCount;
        this.keyboard = builder.keyboard;
        this.display = builder.display;
    }

    @Override
    public String toString() {
        return "Computer{" +
                "cpu='" + cpu + '\'' +
                ", ram='" + ram + '\'' +
                ", usbCount=" + usbCount +
                ", keyboard='" + keyboard + '\'' +
                ", display='" + display + '\'' +
                '}';
    }

    public static class Builder{
        private String cpu;
        private String ram;
        private int usbCount;
        private String keyboard;
        private String display;

        public Builder(String cpu, String ram){
            this.cpu = cpu;
            this.ram = ram;
        }

        public int getUsbCount() {
            return usbCount;
        }

        public Builder setUsbCount(int usbCount) {
            this.usbCount = usbCount;
            return this;
        }

        public String getKeyboard() {
            return keyboard;
        }

        public Builder setKeyboard(String keyboard) {
            this.keyboard = keyboard;
            return this;
        }

        public String getDisplay() {
            return display;
        }

        public Builder setDisplay(String display) {
            this.display = display;
            return this;
        }

        public Computer build(){
            return new Computer(this);
        }
    }
}
```

```java
package cn.xiaohupao.builder;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/1 21:46
 */
public class Client {
    public static void main(String[] args) {
        Computer computer1 = new Computer.Builder("Inter", "三星").build();
        System.out.println(computer1);

        Computer computer2 = new Computer.Builder("Inter", "海盗船").setKeyboard("罗技")
                .setDisplay("三星显示器").setUsbCount(4).build();
        System.out.println(computer2);
    }
}
```

## 通过Client、Director、Builder和Product形成的建造者模式

```java
package cn.xiaohupao.builder;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/1 22:37
 */
public abstract class Builder {

    protected Product product = new Product();

    public abstract void buildPartA();
    public abstract void buildPartB();
    public abstract void buildPartC();

    public Product getResult(){
        return product;
    }
}
```

```java
package cn.xiaohupao.builder;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/1 22:41
 */
public class ConcreteBuilder extends Builder{

    @Override
    public void buildPartA() {
        product.setPartA("partA");
    }

    @Override
    public void buildPartB() {
        product.setPartB("partB");
    }

    @Override
    public void buildPartC() {
        product.setPartC("partC");
    }
}
```

```java
package cn.xiaohupao.builder;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/1 22:38
 */
public class Product {

    private String partA;
    private String partB;
    private String partC;

    public void setPartA(String partA) {
        this.partA = partA;
    }

    public void setPartB(String partB) {
        this.partB = partB;
    }

    public void setPartC(String partC) {
        this.partC = partC;
    }

    @Override
    public String toString() {
        return "Product{" +
                "partA='" + partA + '\'' +
                ", partB='" + partB + '\'' +
                ", partC='" + partC + '\'' +
                '}';
    }
}
```

```java
package cn.xiaohupao.builder;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/1 22:45
 */
public class Director {

    private Builder builder;

    public Director(Builder builder){
        this.builder = builder;
    }

    public void setBuilder(Builder builder) {
        this.builder = builder;
    }

    public Product build(){

        builder.buildPartA();
        builder.buildPartB();
        builder.buildPartC();

        return builder.getResult();
    }
}
```

```java
package cn.xiaohupao.builder;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/1 22:49
 */
public class BuilderClient {
    public static void main(String[] args) {
        ConcreteBuilder concreteBuilder = new ConcreteBuilder();
        Director director = new Director(concreteBuilder);
        Product product = director.build();
        System.out.println(product);
    }
}
```

​		优点：产品的建造和表示分离，实现解耦；将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰；增加新的具体建造者无需修改原有类库的代码、易于拓展，符合“开闭原则”。

​		缺点：产品必须有共同点，限制了使用范围；如内部变化复杂。会有很多的建造类，难以维护。

​		应用场景：需要生成的产品对象有复杂的内部结构，这些产品对象具备共性；隔离复杂对象的创建和使用，并使得相同的创建过程可以创建不同的产品；需要生成的对象内部属性本身相互依赖；适用于一个具有较多的零件的产品的创建过程。
