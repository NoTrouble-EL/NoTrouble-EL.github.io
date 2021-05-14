---
title: SpringMVC基本使用
date: 2021-05-13 21:01:02
tags:
- SpringMVC
- SSM
categories: SpringMVC
mathjax: true
---

## SpringMVC概述

​		Spring为展现层提供的基于MVC设计理念的优秀的Web框架，是目前最主流的MVC框架之一。一种轻量级的、基于MVC的Web层应用框架。它能让我们对请求数据的出来，响应数据的处理，页面的跳转等等常见的Web操作变得更加简单方便。

 <!-- more --> 

## 入门案例

将项目变成Web项目，将生成的目录名字改为webapp，检查Web目录的位置是否正确

**①导入相关依赖**

```xml
<packaging>war</packaging>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <!--端口号 -->
                <port>81</port>
                <!--项目路径-->
                <path>/</path>
                <!--解决get请求中文乱码-->
                <uriEncoding>utf-8</uriEncoding>
            </configuration>
        </plugin>
    </plugins>
</build>

<dependencies>
    <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
    <!--servlet依赖 -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
        <scope>provided</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.jsp-api -->
    <!--jsp依赖-->
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.1</version>
        <scope>provided</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
    <!--springmvc的依赖-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.6</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
    <!--帮助进行json转换-->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.0</version>
    </dependency>

</dependencies>
```

**②配置web.xml**

```xml
<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--
            为DispatcherServlet提供初始化参数的
            设置springmvc配置文件的路径
                name是固定的，必须是contextConfigLocation
                value指的是SpringMVC配置文件的位置
         -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <!--
            指定项目启动就初始化DispatcherServlet
         -->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <!--
             /           表示当前servlet映射除jsp之外的所有请求（包含静态资源）
             *.do        表示.do结尾的请求路径才能被SpringMVC处理(老项目会出现)
             /*          表示当前servlet映射所有请求（包含静态资源,jsp），不应该使用其配置DispatcherServlet
         -->
    <url-pattern>/</url-pattern>
</servlet-mapping>


<!--乱码处理过滤器，由SpringMVC提供-->
<!-- 处理post请求乱码 -->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <!-- name固定不变，value值根据需要设置 -->
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <!-- 所有请求都设置utf-8的编码 -->
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

**③配置springMVC**

在resources目录下创建mvc的配置文件spring-mvc.xml

```xml
<!--
            SpringMVC只扫描controller包即可
        -->
<context:component-scan base-package="cn.xiaohupao.controller"/>
<!-- 解决静态资源访问问题，如果不加mvc:annotation-driven会导致无法访问handler-->
<mvc:default-servlet-handler/>
<!--解决响应乱码-->
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <constructor-arg value="utf-8"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

**④创建测试用的jsp页面**

在webapp下创建success.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    成功
</body>
</html>
```

**⑤编写Controller**

定义一个类，在类上加上@Controller注解，声明其是一个Controller。主要要创建在之前注解扫描所配置的包下。

然后定义一个方法，在方法上加上@RequestMapping来指定哪些请求会被该方法所处理。

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {
    @RequestMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }
}
```

## 设置请求映射规则@RequestMapping

该注解可以加到方法上或者是类上。我们可以用其来设定所能匹配请求的要求。只有符合了设置的要求，请求才能被加了该注解的方法或类处理。

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    String name() default "";

    @AliasFor("path")
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};

    String[] params() default {};

    String[] headers() default {};

    String[] consumes() default {};

    String[] produces() default {};
}
```

### 指定请求路径

path或者value属性都可以用来指定请求路径。

例如：我们期望让请求的资源路径为/test/testPath的请求能够被testPath方法处理则可以写如下代码

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {
    @RequestMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @RequestMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }
}
```

```java
@Controller
@RequestMapping("/test")
public class TestController{
   @RequestMapping("/testPath")
    public String testpath(){
        return "/success.jsp"
    }
}
```

### 指定请求方式

method属性可以用来指定可处理的请求方式。

例如：我们期望让请求的资源路径为/test/testMethod的POST请求能够被testMethod方法处理，则可以写如下代码:

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {
    @RequestMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @RequestMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @RequestMapping(value = "/test/testMethod",method = RequestMethod.POST)
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }
}
```

注意：我们也可以运用如下注解来进行替换

* @PostMapping 等价于 @RequestMapping(method = RequestMethod.POST)
* @GetMapping 等价于 @RequestMapping(method = RequestMethod.GET)
* @PutMapping 等价于 @RequestMapping(method = RequestMethod.PUT)
* @DeleteMapping 等价于 @RequestMapping(method = RequestMethod.DELETE)

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }
}
```

### 指定请求参数

我们可以使用params属性来对请求参数进行一些限制。可以要求必须具有某些参数，或是某些参数必是某个值，或者是某些参数必须不是某个值。

**要求必须有某参数**

例如：我们期望让请求参数的资源路径为/test/testParams的GET请求，并且请求参数具有code参数的请求能够被testParams方法处理。则可以写如下代码：

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }
	//若参数多个，则用数组的形式
    @GetMapping(value = "/test/testParams", params = "code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }
}
```

**要求必须没有某参数**

使用!

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }
    
    @GetMapping(value = "/test/testParams", params = "!code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }
}
```

**要求参数必须是某个值或者必须不是某个值**

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }
    
    @GetMapping(value = "/test/testParams", params = "code=xiaohupao")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }
}
```

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }
    
    @GetMapping(value = "/test/testParams", params = "code!=xiaohupao")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }
}
```

### 请求头的限制

我们可以使用headers属性来对请求头进行一些限制

例如：我们期望让请求的资源路径为/test/testHeaders的GET请求，并且请求头中具有deviceType的请求能够被testHeaders方法处理。则可以写如下代码：

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testParams", params = "code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testHeaders", headers = "deviceType")
    public String testHeaders(){
        System.out.println("testHeaders处理了请求");
        return "/success.jsp";
    }
}
```

如果是要求不能有deviceType这个请求头可以把它改成如下形式：

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testParams", params = "code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }
    
    @GetMapping(value = "/test/testHeaders", headers = "!deviceType")
    public String testHeaders(){
        System.out.println("testHeaders处理了请求");
        return "/success.jsp";
    }
}
```

如果要求有deviceType这个请求头，并且其值必须是某个值可以改成如下形式：

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testParams", params = "code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testHeaders", headers = "deviceType=ios")
    public String testHeaders(){
        System.out.println("testHeaders处理了请求");
        return "/success.jsp";
    }
}
```

如果要求有deviceType这个请求头，并且其值必须不是某个值可以改成如下形式：

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testParams", params = "code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testHeaders", headers = "deviceType!=ios")
    public String testHeaders(){
        System.out.println("testHeaders处理了请求");
        return "/success.jsp";
    }
}
```

### 指定请求头Content-Type

我们可以使用consumes属性来对Content-Type这个请求头进行一些限制。

例如：我们期望让请求的资源路径为/test/testConsumes的POST请求，并且请求头中的Content-Type头必须为multipart/from-data的请求能够被testConsumes方法处理。则可以写如下代码：

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testParams", params = "code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testHeaders", headers = "deviceType")
    public String testHeaders(){
        System.out.println("testHeaders处理了请求");
        return "/success.jsp";
    }

    @PostMapping(value = "/test/testConsumes", consumes = "multipart/from-data")
    public String testConsumes(){
        System.out.println("testConsumes");
        return "/success.jsp";
    }
}
```

例如：如果我们要求请求头Content-Type的值必须不能为某个multipart/from-data则可以改成如下形式：

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/13 16:04
 */
@Controller
public class TestController {

    @GetMapping("/hello.html")
    public String hello(){
        System.out.println("hello");
        return "/success.jsp";
    }

    @GetMapping("/test/testPath")
    public String testPath(){
        System.out.println("testPath");
        return "/success.jsp";
    }

    @PostMapping("/test/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testParams", params = "code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }

    @GetMapping(value = "/test/testHeaders", headers = "deviceType")
    public String testHeaders(){
        System.out.println("testHeaders处理了请求");
        return "/success.jsp";
    }

    @PostMapping(value = "/test/testConsumes", consumes = "!multipart/from-data")
    public String testConsumes(){
        System.out.println("testConsumes");
        return "/success.jsp";
    }
}
```

## RestFul风格

RestFul是一种网络应用程序的设计风格和开发方式。现在很多互联网的网络接口定义都会符合风格。

主要规则如下：

* 每一个URL代表一种资源
* 客户端使用GET、POST、PUT、DELETE4个表示操作方式的动词对服务端资源进行操作：GET用来获取资源、POST用来新建资源、PUT用来跟新资源、DELETE用来删除资源；
* 简单参数例如id等写到url路径上 例如：/user/1 HTTP GET：获取id=1的user信息 /user/1 HTTP DELETE ：删除id=1的user信息
* 复杂的参数转换成json或者xml写到请求中。

## 获取请求参数

### 获取路径参数

RestFul风格的接口一些参数是在请求路径上的。类似：/user/1 这里的1就是id。如果我们想要获取这种格式的数据可以使用@PathVariable来实现。

例如：要求定义一个RestFul风格的接口，该接口可以用来根据id查询用户。请求路径要求为/user，请求方式要求为GET。而请求参数id要写在请求路径上，例如/user/1 这里的1就是id。我们可以定义如下方法，通过如下方式来获取参数路径：

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/14 15:39
 */
@Controller
public class UserController {

    @GetMapping(value = "/user/{id}")
    public String findUserById(@PathVariable("id") Integer id){
        System.out.println("findUserById");
        System.out.println(id);
        return "/success.jsp";
    }
}
```

```tex
GET http://localhost:81/user/1

findUserById
1
```

例如：如果这个接口，根据id和username查询用户。请求路径要求为/user，请求方式为GET。请求参数为id和name要写在请求路径上，例如：/user/1/xiaohupao 这里1就是id，xiaohupao是name。我们可以定义如下的方法，通过如下方式来获取路径参数：

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/14 15:39
 */
@Controller
public class UserController {

    @GetMapping(value = "/user/{id}")
    public String findUserById(@PathVariable("id") Integer id){
        System.out.println("findUserById");
        System.out.println(id);
        return "/success.jsp";
    }

    @GetMapping(value = "/user/{id}/{name}")
    public String findUserByIdAndName(@PathVariable("id") Integer id, @PathVariable("name") String name){
        System.out.println("findUserByIdAndName");
        System.out.println("id: " + id + ", name: " + name);
        return "/success.jsp";
    }
}
```

```tex
GET http://localhost:81/user/1/xiaohupao

findUserByIdAndName
id: 1, name: xiaohupao
```

### 获取请求体中的Json参数

RestFul风格的接口一些比较复杂的参数会转换成json通过请求体传递过来。这时候我们可以用@RequestBody注解获取请求体中的数据。

**配置**

SpringMVC可以帮我们把json数据换成我们需要的类型，但是需要一些基本配置。SpringMVC默认会使用jackson来进行json解析。所以我们需要导入jackson的依赖。

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<!--帮助进行json转换-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
```

然后还要配置注解驱动

```xml
<!--解决响应乱码-->
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <constructor-arg value="utf-8"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

**使用**

**获取参数封装成实体对象**

例如：我们要求定义一个RestFul风格的接口，该接口可以用来新建用户，请求路径要求为/user，请求方式要求为POST。用户数据会转换成json通过请求体传递：

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/14 15:39
 */
@Controller
public class UserController {

    @GetMapping(value = "/user/{id}")
    public String findUserById(@PathVariable("id") Integer id){
        System.out.println("findUserById");
        System.out.println("id:" + id);
        return "/success.jsp";
    }

    @GetMapping(value = "/user/{id}/{name}")
    public String findUserByIdAndName(@PathVariable("id") Integer id, @PathVariable("name") String name){
        System.out.println("findUserByIdAndName");
        System.out.println("id: " + id + ", name: " + name);
        return "/success.jsp";
    }

    @PostMapping(value = "/user")
    public String insertUser(@RequestBody User user){
        System.out.println("insertUser");
        System.out.println(user);
        return "/success.jsp";
    }
}
```

```java
package cn.xiaohupao.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/14 16:55
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String name;
    private Integer age;
}
```

```tex
Body raw JSON
{
    "id":1025,
    "name":"576",
    "age":23
}

insertUser
User(id=1025, name=576, age=23)
```

**获取参数封装成Map集合**

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/14 15:39
 */
@Controller
public class UserController {

    @GetMapping(value = "/user/{id}")
    public String findUserById(@PathVariable("id") Integer id){
        System.out.println("findUserById");
        System.out.println("id:" + id);
        return "/success.jsp";
    }

    @GetMapping(value = "/user/{id}/{name}")
    public String findUserByIdAndName(@PathVariable("id") Integer id, @PathVariable("name") String name){
        System.out.println("findUserByIdAndName");
        System.out.println("id: " + id + ", name: " + name);
        return "/success.jsp";
    }

    @PostMapping(value = "/user")
    public String insertUser(@RequestBody Map map){
        System.out.println("insertUser");
        System.out.println(map);
        return "/success.jsp";
    }
}
```

```tex
Body raw JSON
{
    "id":1025,
    "name":"576",
    "age":23
}

insertUser
User(id=1025, name=576, age=23)
```

**获取的JSON数据转换成List**

例如：如果请求体传递过来的数据是一个User集合转换成的JSON，则可以按照如下写法：

```tex
[{"id":1025, "name":"576, "age":23},{"id":221, "name":"95", "age":25},{"id":222, "name":"xiaohupao", "age":25}]
```

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/14 15:39
 */
@Controller
public class UserController {

    @GetMapping(value = "/user/{id}")
    public String findUserById(@PathVariable("id") Integer id){
        System.out.println("findUserById");
        System.out.println("id:" + id);
        return "/success.jsp";
    }

    @GetMapping(value = "/user/{id}/{name}")
    public String findUserByIdAndName(@PathVariable("id") Integer id, @PathVariable("name") String name){
        System.out.println("findUserByIdAndName");
        System.out.println("id: " + id + ", name: " + name);
        return "/success.jsp";
    }

    @PostMapping(value = "/user")
    public String insertUser(@RequestBody User user){
        System.out.println("insertUser");
        System.out.println(user);
        return "/success.jsp";
    }

    @PostMapping(value = "/users")
    public String insertUsers(@RequestBody List<User> users){
        System.out.println("insertUsers");
        System.out.println(users);
        return "/success.jsp";
    }
}
```

```tex
insertUsers
[User(id=1025, name=576, age=23), User(id=221, name=95, age=25), User(id=222, name=xiaohupao, age=25)]
```

**获取JSON数据的注意事项**

如果需要@RequestBody来获取请求体中json并且进行转换，要求请求头Content-Type的值要为application/json

### 获取QueryString格式参数

如果接受参数是使用QueryString的格式的话，我们也可以使用SpringMVC快速获取参数。我们可以使用@RequestParam来获取QueryString格式的参数。

**使用**

**参数单独的获取**

在方法中定义方法参数，方法参数名要和请求参数名一致，这种情况下我们可以省略@RequestParam注解。如果方法参数名和请求参数名不一致，我们可以加上@RequestParam注解

例如：要求定义个接口，该接口的请求路径为/testRequestParam，请求方式无要求。参数为id和name和likes。使用QueryString格式传递。

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import java.util.Arrays;
import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/14 15:39
 */
@Controller
public class UserController {

    @GetMapping(value = "/user/{id}")
    public String findUserById(@PathVariable("id") Integer id){
        System.out.println("findUserById");
        System.out.println("id:" + id);
        return "/success.jsp";
    }

    @GetMapping(value = "/user/{id}/{name}")
    public String findUserByIdAndName(@PathVariable("id") Integer id, @PathVariable("name") String name){
        System.out.println("findUserByIdAndName");
        System.out.println("id: " + id + ", name: " + name);
        return "/success.jsp";
    }

    @PostMapping(value = "/user")
    public String insertUser(@RequestBody User user){
        System.out.println("insertUser");
        System.out.println(user);
        return "/success.jsp";
    }

    @PostMapping(value = "/users")
    public String insertUsers(@RequestBody List<User> users){
        System.out.println("insertUsers");
        System.out.println(users);
        return "/success.jsp";
    }

    @RequestMapping(value = "/testRequestPara")
    public String testRequestParam(@RequestParam("id") Integer id, @RequestParam("name") String name, @RequestParam("likes") String[] likes){
        System.out.println("testRequestParam");
        System.out.println("id: " + id + ", name: " + name + ",likes: " + Arrays.toString(likes));
        return "/success.jsp";
    }
}
```

```tex
GET http://localhost:81//testRequestPara?id=221&name=xiaohupao&likes=Game&likes=Game

testRequestParam
id: 221, name: xiaohupao,likes: [Game, Game]
```

**多个参数封装成对象**

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import java.util.Arrays;
import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/14 15:39
 */
@Controller
public class UserController {

    @GetMapping(value = "/user/{id}")
    public String findUserById(@PathVariable("id") Integer id){
        System.out.println("findUserById");
        System.out.println("id:" + id);
        return "/success.jsp";
    }

    @GetMapping(value = "/user/{id}/{name}")
    public String findUserByIdAndName(@PathVariable("id") Integer id, @PathVariable("name") String name){
        System.out.println("findUserByIdAndName");
        System.out.println("id: " + id + ", name: " + name);
        return "/success.jsp";
    }

    @PostMapping(value = "/user")
    public String insertUser(@RequestBody User user){
        System.out.println("insertUser");
        System.out.println(user);
        return "/success.jsp";
    }

    @PostMapping(value = "/users")
    public String insertUsers(@RequestBody List<User> users){
        System.out.println("insertUsers");
        System.out.println(users);
        return "/success.jsp";
    }

    @RequestMapping(value = "/testRequestPara")
    public String testRequestParam(@RequestParam("id") Integer id, @RequestParam("name") String name, @RequestParam("likes") String[] likes){
        System.out.println("testRequestParam");
        System.out.println("id: " + id + ", name: " + name + ",likes: " + Arrays.toString(likes));
        return "/success.jsp";
    }

    @RequestMapping(value = "/testRequestParas")
    public String testRequestParams(User user){
        System.out.println("testRequestParams");
        System.out.println(user);
        return "/success.jsp";
    }
}
```

```tex
GET http://localhost:81//testRequestParas?id=221&age=25&name=xiaohupao

testRequestParams
User(id=221, name=xiaohupao, age=25)
```

