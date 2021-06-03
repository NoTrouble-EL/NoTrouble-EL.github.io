---
title: 原型模式(Prototype Pattern)
date: 2021-06-02 22:45:16
tags:
- DesignPatterns
categories: DesignPatterns
mathjax: true
---

​		使用原型实例指定要创建对象的类型，通过复制这个原型来创建新对象。

 <!-- more --> 

## 浅拷贝的实现

```java
package cn.xiaohupao.prototype;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/2 20:54
 */
public interface Myclone {
    /**
     * 克隆方法
     * @return 克隆对象
     */
    Object clone();
}
```

```java
package cn.xiaohupao.prototype;

import java.util.Date;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/2 20:55
 */
public class Student implements Myclone{

    private String name;
    private Date time;

    public Student(String name, Date time) {
        this.name = name;
        this.time = time;
    }

    public Student(Student student){
        this.name = student.name;
        this.time = student.time;
    }

    public String getName() {
        return name;
    }

    public Date getTime() {
        return time;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", time=" + time +
                '}';
    }


    @Override
    public Object clone() {
        return new Student(this);
    }
}
```

```java
package cn.xiaohupao.prototype;

import java.util.Date;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/2 21:00
 */
public class Client {
    public static void main(String[] args) {
        Student lsn  = new Student("576", new Date());
        System.out.println(lsn);
        Student lsnCopy = new Student(lsn);
        System.out.println(lsnCopy);
    }
}
```

## 深拷贝的实现

```java
package cn.xiaohupao.prototype;

import java.util.Date;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/2 21:16
 */
public class User implements Cloneable{

    private String name;
    private Integer age;
    private Date birthday;

    public User(String name, Integer age, Date birthday) {
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", birthday=" + birthday +
                '}';
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        User v = (User) super.clone();
        v.birthday = (Date) this.birthday.clone();

        return v;
    }
}
```

```java
package cn.xiaohupao.prototype;

import java.util.Date;

/**
 * @Author: xiaohupao
 * @Date: 2021/6/2 21:23
 */
public class UserClient {
    public static void main(String[] args) throws CloneNotSupportedException {
        Date birthday = new Date();
        User lsn = new User("lsn", 23, birthday);
        User lsnCopy = (User) lsn.clone();

        System.out.println(lsn);
        System.out.println(lsnCopy);
        birthday.setTime(22131231);
        lsn.setAge(25);
        System.out.println(lsn);
        System.out.println(lsnCopy);
    }
}
```

