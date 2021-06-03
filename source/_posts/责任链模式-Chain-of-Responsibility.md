---
title: 责任链模式(Chain of Responsibility)
date: 2021-06-02 22:46:56
tags:
- DesignPatterns
categories: DesignPatterns
mathjax: true
---

​		使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链发送该请求，直到有一个对象处理它为止。

​		责任链模式是一种处理请求的模式，它让多个处理器都有机会处理该请求，直到其中某个处理成功为止。责任链模式把多个处理器串成链，然后让请求在链上传递。		

 <!-- more --> 

```java
package cn.xiaohupao.chainofresponsibility;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/2 22:06
 */
public abstract class Handler {
    protected Handler nextHandler;
    public void setNextHandler(Handler handler){
        this.nextHandler = handler;
    }
    public abstract void process(Integer info);
}
```

```java
package cn.xiaohupao.chainofresponsibility;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/2 22:08
 */
public class Leader extends Handler{
    @Override
    public void process(Integer info) {
        if (info > 0 && info < 11){
            System.out.println("Leader! 处理!");
        }else {
            nextHandler.process(info);
        }
    }
}
```

```java
package cn.xiaohupao.chainofresponsibility;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/2 22:10
 */
public class Boss extends Handler{
    @Override
    public void process(Integer info) {
        System.out.println("Boss! 处理!");
    }
}
```

```java
package cn.xiaohupao.chainofresponsibility;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/2 22:12
 */
public class Client {
    public static void main(String[] args) {
        Handler level1 = new Leader();
        Handler level2 = new Boss();
        level1.setNextHandler(level2);

        level1.process(10);
        level1.process(12);
    }
}
```

