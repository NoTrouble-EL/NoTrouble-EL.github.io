---
title: SpringBoot的基本使用
date: 2021-07-10 22:11:11
tags:
- SpringBoot
categories: SpringBoot
mathjax: true
---

## 为什么要学习SpringBoot

* 在SSM中需要写很多的配置才能进行正常的使用
* 实现一个功能需要引入很多依赖，尤其是要自己去维护依赖的版本，容易出现依赖冲突等问题

SpringBoot就能很好的解决上述问题

 <!-- more --> 

## SpringBoot是什么

Spring是基于Spring开发的全新框架，相当于对Spring做了又一层封装

其设计的目的是用来简化Spring的初始搭建开发过程，该框架使用了特定的方式进行配置，从而使开发人员不需要定义样板化的配置；并且对第三方依赖的添加也进行了封装简化

Spring能做的他都能做，并且简化了配置，并且提供了Spring没有的：

* 内嵌WEB容器，不需要再部署到web容器中
* 提供准备好的特性，如指标、健康检查和外部化配置

最大特点：自动配置、起步依赖

## 快速入门

### 基本环境要求

JDK8；Maven3.5

```xml
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>hhttps://maven.aliyun.com/repository/public</url>
    </mirror>
</mirrors>

<localRepository>D:/apache-maven-3.6.3/m2/repository</localRepository>
```

### HelloWorld

**配置父工程**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
</parent>
```

**添加依赖**

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-parent -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

</dependencies>
```

**创建启动类**

@SpringBootApplication注解标识为启动类

```java
package cn.xiaohupao;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @Author: xiaohupao
 * @Date: 2021/7/10 21:19
 */
@SpringBootApplication
public class HelloApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class, args);
    }
}
```

**定义Controller**

```java
package cn.xiaohupao.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author: xiaohupao
 * @Date: 2021/7/10 21:27
 */
/*@Controller
@ResponseBody*/
@RestController // 相当于@Controller + @ResponseBody
public class HelloController {
    @GetMapping(value = "/hello")
    public String hello(){
        return "HelloSpringBoot";
    }
}
```

运行测试是直接运行启动类的main方法

### 常见问题及解决方案

**访问时404**

将Controller放在启动类所在的包及其子包下

### 打包部署

可以把SpringBoot的项目打成jar包直接运行

**添加maven插件**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**maven打包**

package

**运行jar包**

在jar包所在的目录执行命令

```bash
java -jar jar包名称
```

### 快速构建

## 起步依赖

SpringBoot依靠父类项目的版本锁定和starter机制让我们能够更轻松的实现对依赖的管理
