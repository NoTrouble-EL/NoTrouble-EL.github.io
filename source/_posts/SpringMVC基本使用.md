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

打包方式为war

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

	//若参数多个，则用数组的形式
    @GetMapping(value = "/test/testParams", params = "code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "/success.jsp";
    }
}
```

```tex
GET http://localhost:81/test/testParams?code=12313
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

### 相关注解其他属性

#### required

代表是否必须，默认值为true(也就是必须要有对应的参数，如果没有就会报错)。如果对应的参数可传可不传，则可以设置为false。

#### defaultValue

如果对应的参数没有，则可以使用defaultValue属性设置默认值。

```java
public class UserController {
    @RequestMapping(value = "/testRequestPara")
    public String testRequestParam(@RequestParam(value = "id", required = false, defaultValue = "576") Integer id, @RequestParam("name") String name, @RequestParam("likes") String[] likes){
        System.out.println("testRequestParam");
        System.out.println("id: " + id + ", name: " + name + ",likes: " + Arrays.toString(likes));
        return "/success.jsp";
    }
}
```

```tex
GET http://localhost:81//testRequestPara?age=25&name=xiaohupao&likes=Game&likes=Game

testRequestParam
id: 576, name: xiaohupao,likes: [Game, Game]
```

## 类型转换器

虽然获取参数看起来非常轻松，但是在这个过程中是有可能出现一些问题的。例如：请求参数为success=1我们期望把这个参数获取出来赋值给一个boolean类型的变量。这里就涉及到String->Boolean类型的转换了。实际上SPringMVC中内置了很多类型转换器来进行类型转换。也有专门进行String->Boolean类型转换的转换器StringToBooleanConverter。

如果是符合SpringMVC内置转换器的转换规则就可以很轻松的实现转换。但是如果不符合转换器的规则呢？

例如，请求参数为birthday=2004-12-12我们期望把这个请求参数取出来赋值给一个Date类型的变量，就不符合内置的规则了。内置的可以把2004/12/12这种格式进行转换。这种情况下我们就可以选择自定义类型转换。

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.springframework.core.convert.support;

import java.util.HashSet;
import java.util.Set;
import org.springframework.core.convert.converter.Converter;
import org.springframework.lang.Nullable;

final class StringToBooleanConverter implements Converter<String, Boolean> {
    private static final Set<String> trueValues = new HashSet(8);
    private static final Set<String> falseValues = new HashSet(8);

    StringToBooleanConverter() {
    }

    @Nullable
    public Boolean convert(String source) {
        String value = source.trim();
        if (value.isEmpty()) {
            return null;
        } else {
            value = value.toLowerCase();
            if (trueValues.contains(value)) {
                return Boolean.TRUE;
            } else if (falseValues.contains(value)) {
                return Boolean.FALSE;
            } else {
                throw new IllegalArgumentException("Invalid boolean value '" + source + "'");
            }
        }
    }

    static {
        trueValues.add("true");
        trueValues.add("on");
        trueValues.add("yes");
        trueValues.add("1");
        falseValues.add("false");
        falseValues.add("off");
        falseValues.add("no");
        falseValues.add("0");
    }
}
```

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/18 16:56
 */
@Controller
public class ConverterController {

    @RequestMapping(value = "/testConverter")
    public String testConverter(@RequestParam(value = "success") Boolean success){
        System.out.println("testConverter");
        System.out.println(success);
        return "/success.jsp";
    }
}
```

```tex
GET http://localhost:81/testConverter?success=1

testConverter
true
```

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/18 16:56
 */
@Controller
public class ConverterController {

    @RequestMapping(value = "/testConverter")
    public String testConverter(@RequestParam(value = "success") Boolean success){
        System.out.println("testConverter");
        System.out.println(success);
        return "/success.jsp";
    }

    @RequestMapping(value = "/testDateConverter")
    public String testDateConverter(@RequestParam(value = "birthday") Date birthday){
        System.out.println("testDateConverter");
        System.out.println(birthday);
        return "/success.jsp";
    }
}
```

```tex
GET http://localhost:81/testDateConverter?birthday=1998/10/25

testDateConverter
Sun Oct 25 00:00:00 CST 1998
```

### 自定义类型转换器

**①创建类实现Converter接口**

```java
package cn.xiaohupao.converter;

import org.springframework.core.convert.converter.Converter;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/18 17:22
 */
public class StringToDateConverter implements Converter<String, Date> {

    @Override
    public Date convert(String source) {
        //String->Date 1998-10-25
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        Date date = null;
        try {
            date = simpleDateFormat.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```

**②实现Converter方法**

见上

**③配置让SpringMVC使用自己定义转换器**

```xml
<!--解决响应乱码-->
<mvc:annotation-driven conversion-service="myConversionService">
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <constructor-arg value="utf-8"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean class="org.springframework.context.support.ConversionServiceFactoryBean" id="myConversionService">
    <property name="converters">
        <set>
            <bean class="cn.xiaohupao.converter.StringToDateConverter"/>
        </set>
    </property>
</bean>
```

### 日期转换简便解决方案

在参数前加上@DateTimeFormat注解，设置其中属性patten。

```java
/**
 * @Author: xiaohupao
 * @Date: 2021/5/18 16:56
 */
@Controller
public class ConverterController {

    @RequestMapping(value = "/testConverter")
    public String testConverter(@RequestParam(value = "success") Boolean success){
        System.out.println("testConverter");
        System.out.println(success);
        return "/success.jsp";
    }

    @RequestMapping(value = "/testDateConverter")
    public String testDateConverter(@DateTimeFormat(pattern = "yyyy-MM-dd") @RequestParam(value = "birthday") Date birthday){
        System.out.println("testDateConverter");
        System.out.println(birthday);
        return "/success.jsp";
    }
}
```

## 响应体响应数据

无论是RestFul风格还是之前web阶段接触过的异步请求，都需要把数据转成json放入响应体中。

### 数据放入响应体

SringMVC为我们提供了@ResponseBody非常方便的把json放到响应体中。

@ResponseBody可以加在哪些东西上？类上，方法上。

如果要把方法的返回值放到响应体中，则在方法上加上该注解；如果要把类中的每个方法的返回值都放到响应体中，则在类上加上该注解。

### 数据转换成json

SpringMVC可以把我们进行json的转换，不过需要进行响应配置。

#### 配置

**①导入jackson依赖**

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<!--帮助进行json转换-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
```

**②开启mvc的注解驱动**

```xml
<mvc:annotation-driven></mvc:annotation-driven>
```

#### 使用

只要把要转换的数据直接作为方法的返回值返回即可。springMVC会帮助我们把返回值转换成json。

**例**

要求定义个RestFul风格的接口，该接口可以用来根据id查询用户。请求路径要求为/response/use，请求方式要求为GET。而请求参数id要写在请求路径上，例如/response/user/1这里1就是id。要求获取参数id去查询对应id的用户信息(模拟查询即可，可以选择直接new一个User对象)，并且转换成json响应到响应体中。

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/18 19:52
 */
@Controller
@RequestMapping(value = "/response")
public class ResponseController {

    @GetMapping(value = "/user/{id}")
    @ResponseBody
    public User testResponse(@PathVariable(value = "id") Integer id){
        User user = new User(id, "576", 23);
        return user;
    }
}
```

```tex
http://localhost:81/response/user/1

{
    "id": 1,
    "name": "576",
    "age": 23
}
```

要求定义一个RestFul风格的接口，该接口可以查询所有用户。请求路径为/response/user，请求方式要求为GET。去查询所有用户信息(模拟查询即可，可以选择直接创建集合，添加几个User对象)，并且转换成json响应到响应体中。

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/18 19:52
 */
@Controller
@RequestMapping(value = "/response")
public class ResponseController {
    @GetMapping(value = "/user")
    @ResponseBody
    public List<User> testResponse1(){
        List<User> list = new ArrayList<User>();
        list.add(new User(1, "576", 23));
        list.add(new User(2, "95", 25));
        return list;
    }
}
```

```tex
GET http://localhost:81/response/user

[
    {
        "id": 1,
        "name": "576",
        "age": 23
    },
    {
        "id": 2,
        "name": "95",
        "age": 25
    }
]
```

注意：**@RestController 相当于 @Controller + @ResponseBody**

## 页面的跳转

在SpringMVC中我们可以非常轻松的实现页面跳转，只需要把方法的返回值写成要跳转页面的路径即可。例如：

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/20 19:25
 */
@Controller
public class PageJumpController {

    @RequestMapping(value = "/testjump")
    public String testJump(){
        return "/success.jsp";
    }
}
```

默认的跳转其实是转发的方式跳转的。我们也可以选择加上标识，在要跳转的路径上加上forward:。这样SpringMVC也会帮我们进行请求转发。例如：

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/20 19:25
 */
@Controller
public class PageJumpController {

    @RequestMapping(value = "/testjump")
    public String testJump(){
        return "forward:/success.jsp";
    }
}
```

```tex
GET http://localhost:81/testjump

success
```

如果想实现重定向跳转则可以在跳转路径前加上redirect:进行标识。这样SpringMVC就会帮我们进行重定向跳转。

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/20 19:25
 */
@Controller
public class PageJumpController {

    @RequestMapping(value = "/testjump")
    public String testJump(){
        return "redirect:/success.jsp";
    }
}
```

## 视图解析器

如果我们经常需要跳转页面，并且页面所在的路径比较长，我们每次写完整路径会显得有点麻烦，我们可以配置视图解析器，设置跳转页面的前缀和后缀。这样可以简化我们的书写。

### 使用步骤

**①配置视图解析器**

我们需要往SpringMVC容器中注入InternalResourceViewResolver对象。

```xml
<!--配置视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="resourceViewResolver">
    <property name="prefix" value="/WEB-INF/page/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

**②页面跳转**

视图解析器会在逻辑视图的基础上拼接得到物理试图。

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/20 19:25
 */
@Controller
public class PageJumpController {

    @RequestMapping(value = "/testjump")
    public String testJump(){
        return "redirect:/success.jsp";
    }

    @RequestMapping(value = "/testToJsp")
    public String testToJsp(){
        return "test";
    }
}
```

```tex
GET http://localhost:81/testToJsp
```

### 不进行前后缀拼接

如果在配置了视图解析器的情况下，某些方法中并不想拼接前后缀去跳转，这种情况下我们可以在跳转路径前加上forward:或者redirect:进行标识。这样就不会进行前后缀的拼接了。

## 获取原生对象及相关数据

我们之前在web阶段我们经常要使用request对象，response，session对象等。我们也可以通过SpringMVC获取到这些对象。(不过在MVC中我们很少获取这些对象，因为有更简洁的方式，避免了我们使用这些原生对象相对频繁的API。)

我们只需要在方法上添加对应类型的参数即可，但是注意数据类型不要写错了，SpringMVC会把我们需要的对象传给我们的形参。例如：

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/20 20:29
 */
@Controller
public class RequestResponseController {

    @RequestMapping(value = "/getReqAndRes")
    public String getReqAndRes(HttpServletRequest request, HttpServletResponse response, HttpSession session){
        System.out.println();
        return "test";
    }
}
```

## 获取请求头和Cookie

### 获取请求头

在方法中定义一个参数，参数前加上@RequestHeader注解，知道要获取的请求头名即可获取对应请求头的值。例如：想要获取device-type这个请求则可以按照如下方式定义方法：

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/20 20:29
 */
@Controller
public class RequestResponseController {

    @RequestMapping(value = "/getReqAndRes")
    public String getReqAndRes(HttpServletRequest request, HttpServletResponse response, HttpSession session){
        System.out.println();
        return "test";
    }

    @RequestMapping(value = "/getHeader")
    public String getHeader(@RequestHeader(value = "device-Type", required = false) String deviceType){
        if ("ios".equals(deviceType)) {
            System.out.println(deviceType);
        }else{
            System.out.println("not ios");
        }
        return "test";
    }
}
```

```tex
GET http://localhost:81/getHeader
device-Type ios
ios
```

### 获取Cookie

在方法中定义一个参数，参数前加上@CookieValue注解，知道要获取的cookie名即可获取对应cookie的值。例如：想要获取JSESSIONID的cookie值，则可以按照如下定义方法。

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/20 20:29
 */
@Controller
public class RequestResponseController {

    @RequestMapping(value = "/getReqAndRes")
    public String getReqAndRes(HttpServletRequest request, HttpServletResponse response, HttpSession session){
        System.out.println();
        return "test";
    }

    @RequestMapping(value = "/getHeader")
    public String getHeader(@RequestHeader(value = "device-Type", required = false) String deviceType){
        if ("ios".equals(deviceType)) {
            System.out.println(deviceType);
        }else{
            System.out.println("not ios");
        }
        return "test";
    }

    @RequestMapping(value = "/getCookie")
    public String getCookie(@CookieValue(value = "JSESSIONID") String sessionId){
        System.out.println(sessionId);
        return "test";
    }
}
```

```tex
GET http://localhost:81/getCookie
Cookie JSESSIONID=CF5F147F5A430C7550EB3

CF5F147F5A430C7550EB3ECD855B3F17
```

## JSP开发模式

如果我们使用JSP进行开发，那我们就需要在域中存数据，然后跳转到对应的JSP页面中，在JSP页面中获取域中的数据然后进行相关处理。

使用如果是类似于JSP开发的模式就会涉及到往域中存数据和携带数据跳转到跳转页面的操作。

### 往Request域存数据并跳转

#### 使用Model

我们可以使用model来往域中存数据。然后使用之前的方式实现页面跳转。

例如：我们要求访问/testRequestScope这个路径时能往Request域中存name和title数据，然后跳转到/WEB-INF/page/testScope.jsp这个页面。在jsp中获取域中的数据：

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/22 16:27
 */

@Controller
public class JspController {

    @RequestMapping(value = "/testRequestScope")
    public String testRequestScope(Model model){
        //往域中存数据
        model.addAttribute("name", "576");
        model.addAttribute("title", "95");
        return "testScope";
    }
}
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${requestScope.get("name")}
    ${requestScope.get("title")}
</body>
</html>
```

```tex
GET http://localhost:81/testRequestScope

576 95
```

**域对象的理解**

所谓的从域当中获取值，就是把request对象当中的attributes这个Map中的值通过键获取出来。

#### 使用ModelAndView

我们可以使用ModelAndView来往域中存数据和页面跳转。

例如：我们要求访问/testRequestScope2这个路径时能往域中存name和title数据，然后跳转到/WEB-INF/page/testScope.jsp这个页面。在jsp中获取域中的数据。

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/22 16:27
 */

@Controller
public class JspController {

    @RequestMapping(value = "/testRequestScope")
    public String testRequestScope(Model model){
        //往域中存数据
        model.addAttribute("name", "576");
        model.addAttribute("title", "95");
        return "testScope";
    }

    @RequestMapping(value = "/testRequestScope2")
    public ModelAndView testRequestScope2(ModelAndView modelAndView){
        //往域中存数据
        modelAndView.addObject("name", "576");
        modelAndView.addObject("title", "95");

        //页面跳转
        modelAndView.setViewName("testScope");
        return modelAndView;
    }
}
```

```tex
GET http://localhost:81/testRequestScope2

576 95
```

**注意**要把modelAndView对象作为方法的返回值返回。

### 从Request域中获取数据

我们可以使用@RequestAttribute把它加在方法参数上，可以让SpringMVC帮我们从Request域中获取相关数据。

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/22 16:27
 */

@Controller
public class JspController {

    @RequestMapping(value = "/testRequestScope")
    public String testRequestScope(Model model){
        //往域中存数据
        model.addAttribute("name", "576");
        model.addAttribute("title", "95");
        return "testScope";
    }

    @RequestMapping(value = "/testRequestScope2")
    public ModelAndView testRequestScope2(ModelAndView modelAndView, HttpServletRequest request){
        //往域中存数据
        modelAndView.addObject("name", "576");
        modelAndView.addObject("title", "95");

        //页面跳转
        modelAndView.setViewName("testScope");
        return modelAndView;
    }

    @RequestMapping(value = "/testGetAttribute")
    public String testGetAttribute(@RequestAttribute(value = "org.springframework.web.servlet.HandlerMapping.bestMatchingPattern")
                                               String value){
        System.out.println(value);
        return "testScope";
    }
}
```

```tex
GET http://localhost:81/testGetAttribute

/testGetAttribute
```

### 往Session域存数据并跳转

我们可以使用@SessionAttrlbutes注解来进行标识，用里面的属性来标识哪些数据要存入Session域。

例如：我们要求访问/testSessionScope这个路径时能往域中存入name和title数据，然后跳转到/WEB-INF/page/testScope.jsp这个页面。在jsp中获取Session域中的数据。

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.SessionAttributes;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/22 16:27
 */

@Controller
@SessionAttributes({"name","title"})//表示name和title属性也要存储一份到Session域中
public class JspController {

    @RequestMapping(value = "/testRequestScope")
    public String testRequestScope(Model model){
        //往域中存数据
        model.addAttribute("name", "576");
        model.addAttribute("title", "95");
        return "testScope";
    }

    @RequestMapping(value = "/testRequestScope2")
    public ModelAndView testRequestScope2(ModelAndView modelAndView, HttpServletRequest request){
        //往域中存数据
        modelAndView.addObject("name", "576");
        modelAndView.addObject("title", "95");

        //页面跳转
        modelAndView.setViewName("testScope");
        return modelAndView;
    }

    @RequestMapping(value = "/testGetAttribute")
    public String testGetAttribute(@RequestAttribute(value = "org.springframework.web.servlet.HandlerMapping.bestMatchingPattern")
                                               String value){
        System.out.println(value);
        return "testScope";
    }

    @RequestMapping(value = "/testSessionScope")
    public String testSessionScope(Model model){
        //往域中存数据
        model.addAttribute("name", "576");
        model.addAttribute("title", "95");
        return "testScope";
    }
}
```

```tex
GET http://localhost:81/testSessionScope

576 95
576 95
```

### 获取Session域中的数据

我们可以使用@SessionAttribute把它加在方法参数上，可以让SpringMVC帮我们从Session域中获取相关数据。

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.SessionAttribute;
import org.springframework.web.bind.annotation.SessionAttributes;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/22 16:27
 */

@Controller
@SessionAttributes({"name","title"})
public class JspController {

    @RequestMapping(value = "/testRequestScope")
    public String testRequestScope(Model model){
        //往域中存数据
        model.addAttribute("name", "576");
        model.addAttribute("title", "95");
        return "testScope";
    }

    @RequestMapping(value = "/testRequestScope2")
    public ModelAndView testRequestScope2(ModelAndView modelAndView, HttpServletRequest request){
        //往域中存数据
        modelAndView.addObject("name", "576");
        modelAndView.addObject("title", "95");

        //页面跳转
        modelAndView.setViewName("testScope");
        return modelAndView;
    }

    @RequestMapping(value = "/testGetAttribute")
    public String testGetAttribute(@RequestAttribute(value = "org.springframework.web.servlet.HandlerMapping.bestMatchingPattern")
                                               String value){
        System.out.println(value);
        return "testScope";
    }

    @RequestMapping(value = "/testSessionScope")
    public String testSessionScope(Model model){
        //往域中存数据
        model.addAttribute("name", "576");
        model.addAttribute("title", "95");
        return "testScope";
    }

    @RequestMapping(value = "/testGetSessionAttr")
    public String testGetSessionAttr(@SessionAttribute(value = "name") String name, @SessionAttribute(value = "title") String title){
        System.out.println(name);
        System.out.println(title);
        return "testScope";
    }
}
```

```tex
GET http://localhost:81/testGetSessionAttr
```

## 拦截器

### 应用场景

如果我们想在多个Handler方法执行之前或者之后都进行一些处理，甚至某些情况下不需要拦截掉，不让Handler方法执行。那么可以使SpringMVC为我们提供的拦截器。

### 拦截器和过滤器的区别

过滤器是在Servlet执行之后进行处理，而拦截器是对Handler(处理器)执行前后进行处理。

### 创建并配置拦截器

#### ①创建实现HandlerInterceptor接口

#### ②实现方法

```java
package cn.xiaohupao.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/22 19:22
 */
public class MyInterceptor implements HandlerInterceptor {
    
    /**
     * 方法会在Handler执行之前执行。可以在其中进行一些前置的判断或处理
     * @param request 当前请求对象
     * @param response 当前向响应对象
     * @param handler 真正处理请求的handler方法封装的对象，对象中有这个方法的相关信息
     * @return 代表是否放行，如果为true则放行，如果为false则拦截目标方法
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle");
        //返回值代表是否放行，如果为true则放行，如果为false则拦截目标方法
        return false;
    }
	
    /**
     * 方法会在Handler执行之后执行。我们可以对其中域中的数据进行修改，可以修改要跳转的页面。
     * @param request 当前请求的对象
     * @param response 当前响应对象
     * @param handler 真正处理请求的handler方法封装的对象，对象中有这个方法的相关信息
     * @param modelAndView handler方法执行后的modelAndView对象。我们可以修改其中要跳转的路径，或域中的数据。
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle");
    }
	
    /**
     * 方法会在Handler执行之后执行，我们不可以对域中的数据进行修改，也没有办法要修改的转转路径。我们这个方法一般进行一些资源的释放。
     * @param request 当前的请求对象
     * @param response 当前的响应对象
     * @param handler 真正处理请求的handler方法封装的对象，对象中有这个方法的相关信息
     * @param ex 异常对象
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion");
    }
}
```

#### ③配置拦截器

```xml
<!--配置拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <!--配置拦截器要拦截的路径-->
        <!--
                /test/*     这种会拦截如下  /test/add  /test/delete
                            但是拦截不了多级路径下的内容    /test/add/abc    /test/delete/bcd

                /test/**    这种会拦截多级路径下的内容   例如： /test/add  /test/add/abc

			   /*  代表当前一级路径
                /** 代表当前一级路径或多级路径
            -->
        <mvc:mapping path="/**"/>
        <!--配置拦截器派出的路径-->
        <mvc:exclude-mapping path="/"/>
        <!--配置拦截器注对象注入容器-->
        <bean class="cn.xiaohupao.interceptor.MyInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

```tex
GET http://localhost:81/test/testPath

preHandle
testPath
postHandle
afterCompletion
```

### 拦截器方法及参数详解

**拦截器的方法**

preHandle方法会在Handler方法执行之前执行，我们可以在其中进行一些前置的判断或处理。

postHandle方法会在Handler方法执行前之后执行，我们可以在其中对域中的数据进行修改，也可以修改要跳转的页面。

afterCompletion方法会在最后执行，这个时候已经没有办法对域中的数据进行修改，也没有办法修改要跳转的路径。我们这个方法一般进行一些资源的释放。

**参数详解**

```tex
/**
* 方法会在Handler执行之前执行。可以在其中进行一些前置的判断或处理
* @param request 当前请求对象
* @param response 当前向响应对象
* @param handler 真正处理请求的handler方法封装的对象，对象中有这个方法的相关信息
* @return 代表是否放行，如果为true则放行，如果为false则拦截目标方法
* @throws Exception
*/
     
     /**
     * 方法会在Handler执行之后执行。我们可以对其中域中的数据进行修改，可以修改要跳转的页面。
     * @param request 当前请求的对象
     * @param response 当前响应对象
     * @param handler 真正处理请求的handler方法封装的对象，对象中有这个方法的相关信息
     * @param modelAndView handler方法执行后的modelAndView对象。我们可以修改其中要跳转的路径，或域中的数据。
     * @throws Exception
     */
     
     /**
     * 方法会在Handler执行之后执行，我们不可以对域中的数据进行修改，也没有办法要修改的转转路径。我们这个方法一般进行一些资源的释放。
     * @param request 当前的请求对象
     * @param response 当前的响应对象
     * @param handler 真正处理请求的handler方法封装的对象，对象中有这个方法的相关信息
     * @param ex 异常对象
     * @throws Exception
     */
```

### 案例-登陆状态拦截器

#### 需求

我们的接口需要用户登录状态的检验，如果用户没有登录则跳转到登录页面，登陆的情况下则可以正常访问我们的接口。

#### 分析

* 如何判断是否登录？
  * 登录时往Session写入用户相关信息，然后在其他请求中从Session中获取这些信息，如果获取不到说明不是登录状态。
* 很多接口都要写判断代码？难道在每个Handler中写判断逻辑？
  * 用拦截器中进行登录状态的判断
* 登录接口是否应该进行拦截？
  * 不能拦截
* 静态资源是否要进行拦截？
  * 不要拦截

**步骤分析**

①登录页面，请求发送给登录接口

②登录接口中，校验用户名和密码是否正确(模拟校验即可，先不查询数据库)。如果用户密码正确，登陆成功。把用户名写入session中。

③定义其他请求的Handler方法

④定义拦截器来进行登录状态判断。如果能从session中获取用户名则说明是登录的状态，则放行；如果获取不到，则说明未登录，跳转到登录页面。

#### 代码实现

**环境搭建**

* 1.在Project Structure中+选择web；

* 2.将web包拖入main中，并将web命名为webapp；

* 3.在pom中改变打包方式为war；

* 4.打开Project Stucture检查Web中Web的资源目录是否正确；

* 5.整合SpringMVC配置

  * pom中的依赖配置

    ```xml
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
    
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
        </dependency>
    </dependencies>
    ```

  * 在pom中配置Tomcat插件

    ```xml
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
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>6</source>
                    <target>6</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ```

* 6.配置web.xml

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
      <url-pattern>/loginDemo</url-pattern>
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

* 在resources中创建springmvc配置文件

  ```xml
  <!--
              SpringMVC主键扫描controller包即可
          -->
  <context:component-scan base-package="cn.xiaohupao.controller"/>
  <!-- 解决静态资源访问问题，如果不加mvc:annotation-driven会导致无法访问handler-->
  <mvc:default-servlet-handler/>
  <!--解决响应乱码-->
  <!--<mvc:annotation-driven conversion-service="myConversionService">-->
  <mvc:annotation-driven>
      <mvc:message-converters>
          <bean class="org.springframework.http.converter.StringHttpMessageConverter">
              <constructor-arg value="utf-8"/>
          </bean>
      </mvc:message-converters>
  </mvc:annotation-driven>
  ```

##### 登录功能代码实现

①登录页面，请求发送给登录接口

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form method="post" action="/loginDemo/login">
        用户名：<input type="text" name="username">
        密码：<input type="password" name="password">
        <input type="submit">
    </form>
</body>
</html>
```

②登录接口，校验用户名密码是否正确(模拟即可，暂不考虑查表)，登陆成功后，把用户名写入session中。

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

import javax.servlet.http.HttpSession;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/24 21:01
 */
@Controller
public class LoginController {

    @PostMapping(value = "/login")
    public String login(@RequestParam(value = "username") String username, @RequestParam(value = "password") String password, HttpSession session){
        System.out.println(username);
        System.out.println(password);
        //往session域中写入用户名来代表登录成功
        session.setAttribute("username", username);
        return "/WEB-INF/page/success.jsp";
    }
}
```

##### 登陆状态检验代码实现

**①定义拦截器**

**②重写方法，在preHandle方法中实现状态校验**

```java
package cn.xiaohupao.interceptor;

import org.springframework.util.ObjectUtils;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/24 21:51
 */
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //从session获取用户名，判断是否存在，如果获取不到说明未登录，如果获取到了说明，说明之前登录了,放行。
        HttpSession session = request.getSession();
        String username = (String) session.getAttribute("username");
        if (ObjectUtils.isEmpty(username)){
            //未登录，重定向
            String contextPath = request.getServletContext().getContextPath();
            response.sendRedirect(contextPath + "/static/login.html");
        }else{
            //获取到了，则放行
            return true;
        }
        return false;
    }
}
```

**③配置拦截器**

登录相关接口不应该拦截；静态资源不应拦截

在spring-mvc中配置

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <!--要拦截的路径-->
        <mvc:mapping path="/**"/>
        <!--排除拦截-->
        <mvc:exclude-mapping path="/static/**"/>
        <mvc:exclude-mapping path="/WEB-INF/page/**"/>
        <mvc:exclude-mapping path="/login"/>
        <bean class="cn.xiaohupao.interceptor.LoginInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

当在pom修改项目路径时，则需要在端口号后面写入项目路径

```tex
GET http://localhost:81/loginDemo/test 
```

### 多拦截器执行顺序

如果我们配置了多个拦截器，拦截器的顺序是按照配置的先后顺序的。若在preHandler都返回true的情况下：Request->preHandle1->preHandle2->preHandle3->Handle处理器->postHandle3->postHandle2->postHandle1->afterCompletion3->afterCompletion2->afterCompletion1->Response.

若拦截器3的preHandler方法返回值为false：Request->preHandle1->preHandle2->preHandle3->afterCompletion2->afterCompletion1->Response.

* 只有所有拦截器都放行了，postHandle方法才会被执行。
* 只有当前拦截器放行了，当前拦截器的afterCompletion方法才会执行。

### 统一异常处理

​		我们在实际项目中Dao层和Service层的异常都会被抛到Controller层。但是，如果我们在Controller的方法中都加上了异常的try…catch处理也会显的非常的繁琐。

#### HandlerExceptionResolver

①实现接口

```java
package cn.xiaohupao.resolver;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 20:12
 */
@Component
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {

    /**
     * 如果handler方法出现异常，就会调用到该方法，我们可以在本方法中进行统一的异常处理
     * @param httpServletRequest request对象
     * @param httpServletResponse response对象
     * @param o 出现异常的handler方法封装的对象
     * @param e 异常对象
     * @return
     */
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        return null;
    }
}
```

②重写方法

```java
package cn.xiaohupao.resolver;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 20:12
 */
@Component
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {

    /**
     * 如果handler方法出现异常，就会调用到该方法，我们可以在本方法中进行统一的异常处理
     * @param httpServletRequest request对象
     * @param httpServletResponse response对象
     * @param o 出现异常的handler方法封装的对象
     * @param e 异常对象
     * @return modelAndView对象
     */
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        //获取异常信息，把异常信息放入域对象中
        String msg = e.getMessage();
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("msg", msg);
        //跳转到error.jsp
        modelAndView.setViewName("error");
        return modelAndView;

    }
}
```



③注入容器

可以使用注解注入也可以使用xml配置注入。这里使用注解注入的方式。在类上加@Compoment注解，注意要保证类能被组件扫描扫描到。

#### @ControllerAdvice

①创建类，加上@ControllerAdvice注解进行标识

```java
package cn.xiaohupao.resolver;

import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.servlet.ModelAndView;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 20:48
 */
@Component
@ControllerAdvice
public class MyControllerAdvice {

    @ExceptionHandler(value = {NullPointerException.class, ArrayIndexOutOfBoundsException.class})
    public ModelAndView handlerException(Exception ex, ModelAndView modelAndView){
        //如果出现了相关异常，就会调用该方法
        String msg = ex.getMessage();
        modelAndView.addObject("msg", msg);
        modelAndView.setViewName("error");
        return modelAndView;
    }
}
```

②定义异常处理方法

```java
package cn.xiaohupao.resolver;

import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.servlet.ModelAndView;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 20:48
 */
@Component
@ControllerAdvice
public class MyControllerAdvice {

    @ExceptionHandler(value = {NullPointerException.class, ArrayIndexOutOfBoundsException.class})
    public ModelAndView handlerException(Exception ex, ModelAndView modelAndView){
        //如果出现了相关异常，就会调用该方法
        String msg = ex.getMessage();
        modelAndView.addObject("msg", msg);
        modelAndView.setViewName("error");
        return modelAndView;
    }
}
```

定义异常处理方法，使用@ExceptionHandler标识可以处理的异常

③注入容器

使用@Compoment注解

#### 总结

在实际的项目中一般会选择@ControllerAdvice来进行异常的统一处理。如果在前后端不分离的项目中，异常处理一般是跳转到错误页面，让用户有个更好的体验。而前后端分离的项目中，异常处理一般把异常信息封装到Json中写入响应体。无论是哪种情况，使用@ControllerAdvice的写法都比较方便的实现。

返回json格式

```java
package cn.xiaohupao.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 21:11
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Result {

    private String msg;
    private Integer code;

}
```

```java
package cn.xiaohupao.resolver;

import cn.xiaohupao.pojo.Result;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.ModelAndView;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/25 20:48
 */
@Component
@ControllerAdvice
public class MyControllerAdvice {

    @ExceptionHandler(value = {NullPointerException.class, ArrayIndexOutOfBoundsException.class, ArithmeticException.class})
    public ModelAndView handlerException(Exception ex, ModelAndView modelAndView){
        //如果出现了相关异常，就会调用该方法
        String msg = ex.getMessage();
        modelAndView.addObject("msg", msg);
        modelAndView.setViewName("error");
        return modelAndView;
    }

    @ExceptionHandler(value = {NullPointerException.class, ArrayIndexOutOfBoundsException.class, ArithmeticException.class})
    @ResponseBody
    public Result handlerException1(Exception ex){
        Result result = new Result();
        result.setMsg(ex.getMessage());
        result.setCode(500);
        return result;
    }
}
```

### 文件上传

#### 文件上传要求

http协议规定了我们在进行文件上传时的请求格式要求。所以在进行文件上传时，除了在表单中增加一个用于上传文件的表单项(input标签，type=file)外必须满足以下的条件才能进行上传。

**①请求方式为POST请求**

如果使用表单进行提交的话，可以把form标签的method属性设置为POST。例如：

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
        <form action="/upload" method="post" enctype="multipart/form-data">
            <input type="file" name="uploadFile">
            <input type="submit">
        </form>
    </body>
</html>
```

**②请求头Content-Type必须为multipart/form-data**

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
        <form action="/upload" method="post" enctype="multipart/form-data">
            <input type="file" name="uploadFile">
            <input type="submit">
        </form>
    </body>
</html>
```

#### SpringMVC接收上传过来的文件

SpringMVC使用commons-fileupload的包对文件上进行了封装，我们只需要引入相关依赖和进行相应配置就可以很轻松的实现文件上传功能。

**①导入依赖**

在pom中导入依赖

```xml
<!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
```

**②配置**

在spring-mvc中配置

```xml
<!-- 文件上传解析器
            id 必须为multipartResolver-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 默认字符编码-->
    <property name="defaultEncoding" value="uft-8"/>
    <!-- 一次请求上传的文件总大小的最大值，单位是字节-->
    <property name="maxUploadSize" value="#{1024*1024*50}"/>
    <!-- 每个上传文件大小的最大值，单位是字节-->
    <property name="maxUploadSizePerFile" value="#{1024*1024*10}"/>
</bean>
```

**③接收上传的文件数据并处理**

上传表单

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/upload" method="post" enctype="multipart/form-data">
        <input type="file" name="uploadFile">
        <input type="submit">
    </form>
</body>
</html>
```

```java
package cn.xiaohupao.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/26 21:07
 */
@Controller
public class UploadController {

    @PostMapping(value = "/upload")
    public String upload(@RequestParam(value = "uploadFile") MultipartFile uploadFile1) throws IOException {
        uploadFile1.transferTo(new File("test.sql"));
        return "forward:/success.jsp";
    }
}
```

#### MultipartFile常见用法

* 获取上传文件的原名 uploadFile.getOriginalFilename()
* 获取文件类型的MIME类型 uploadFile.getContentType()
* 获取上传文件的大小 uploadFile.getSize()
* 获取对应上传文件的输入流 uploadFile.getInputStream()

### 文件下载

#### 文件下载要求

如果我们想要提供文件下载的功能。Http协议要求我们必须满足如下规则：

**①设置相应头Content-Type**

要求把提供下载文件的MIME类型作为响应头Content-Type的值

```java
package cn.xiaohupao.utils;

import javax.servlet.ServletContext;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.FileInputStream;
import java.net.URLEncoder;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/27 17:31
 */
public class DownLoadUtils {
    /**
     * 该方法可以快速实现设置两个下载需要的响应头和把文件数据写入响应体
     * @param filePath 该文件的相对路径
     * @param context  ServletContext对象
     * @param response
     * @throws Exception
     */
    public static void downloadFile(String filePath, ServletContext context, HttpServletResponse response) throws Exception {
        String realPath = context.getRealPath(filePath);
        File file = new File(realPath);
        String filename = file.getName();
        FileInputStream fis = new FileInputStream(realPath);
        String mimeType = context.getMimeType(filename);//获取文件的mime类型
        response.setHeader("content-type",mimeType);
        String fname= URLEncoder.encode(filename,"UTF-8");
        response.setHeader("Content-disposition","attachment; filename="+fname+";"+"filename*=utf-8''"+fname);
        ServletOutputStream sos = response.getOutputStream();
        byte[] buff = new byte[1024 * 8];
        int len = 0;
        while((len = fis.read(buff)) != -1){
            sos.write(buff,0,len);
        }
        sos.close();
        fis.close();
    }
}
```

**②设置响应头Content-disposition**

要求把文件名经过URL编码后的值写入响应头Content-disposition。但是要求符合以下格式，因为这样可以解决不同浏览器中文文件名乱码问题。

```java
package cn.xiaohupao.utils;

import javax.servlet.ServletContext;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.FileInputStream;
import java.net.URLEncoder;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/27 17:31
 */
public class DownLoadUtils {
    /**
     * 该方法可以快速实现设置两个下载需要的响应头和把文件数据写入响应体
     * @param filePath 该文件的相对路径
     * @param context  ServletContext对象
     * @param response
     * @throws Exception
     */
    public static void downloadFile(String filePath, ServletContext context, HttpServletResponse response) throws Exception {
        String realPath = context.getRealPath(filePath);
        File file = new File(realPath);
        String filename = file.getName();
        FileInputStream fis = new FileInputStream(realPath);
        String mimeType = context.getMimeType(filename);//获取文件的mime类型
        response.setHeader("content-type",mimeType);
        String fname= URLEncoder.encode(filename,"UTF-8");
        response.setHeader("Content-disposition","attachment; filename="+fname+";"+"filename*=utf-8''"+fname);
        ServletOutputStream sos = response.getOutputStream();
        byte[] buff = new byte[1024 * 8];
        int len = 0;
        while((len = fis.read(buff)) != -1){
            sos.write(buff,0,len);
        }
        sos.close();
        fis.close();
    }
}
```

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.utils.DownLoadUtils;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/27 17:32
 */
@Controller
public class DownloadController {

    @RequestMapping(value = "/download")
    public void downLoad(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //文件下载
        DownLoadUtils.downloadFile("/WEB-INF/file/576.txt",request.getServletContext(), response);
    }
}
```

```java
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    hello
    <a href="/download">下载文件</a>
</body>
</html>
```

### SpringMVC的执行流程

因为我们有两种开发模式：

一种是类似于JSP的开发流程：把数据放入域对象，然后进行页面跳转。

另一种是前后端分离的开发模式：把数据转化为json放入响应体。

#### 类JSP开发模式执行流程

用户：1.发起请求url ——> DispatcherServlet前端控制器：2.查找能够处理的Handler ——>  HandlerMapping处理器映射器：3.返回执行链HandlerExecuttionChain(HandlerInterceptor1，HandlerInterceptor2，…) ——> DispatcherServlet前端控制器：4.请求适配器执行 ——> HandlerAdapter：5.执行Handler方法，把数据处理转换成合适的类型然后作为方法参数传入 ——> Handler处理器(Controller中能出来请求的方法)：6.对方法返回值进行转换成ModelAndView，如果使用了@ResponseBody的话返回的ModelAndView为null。会把Handler方法的返回值放入响应体中 ——> Handler处理器(Controller中能出来请求的方法)：7.返回ModelAndView ——> DispatcherServlet前端控制器：8.如果ModelAndView不为空则请求视图解析器 ——> ViewResolver视图解析器：9.返回view ——> DispatcherServlet前端控制器10.response响应 ——> 视图。

​	1.用户发起请求被DispatchServlet所处理

​	2.DispatchServlet通过HandlerMapping根据具体的请求查找能处理这个请求的Handler。**（HandlerMapping主要是处理请求和Handler方法的映射关系的）**

​	3.HandlerMapping返回一个能够处理请求的执行链给DispatchServlet，这个链中除了包含Handler方法也包含拦截器。

​	4.DispatchServlet拿着执行链去找HandlerAdater执行链中的方法。

​	5.HandlerAdater会去执行对应的Handler方法，把数据处理转换成合适的类型然后作为方法参数传入 

​	6.Handler方法执行完后的返回值会被HandlerAdapter转换成ModelAndView类型。**（HandlerAdater主要进行Handler方法参数和返回值的处理。）**

​	7.返回ModelAndView给DispatchServlet.

​	8.如果对于的ModelAndView对象不为null，则DispatchServlet把ModelAndView交给 ViewResolver 也就是视图解析器解析。

​	9.ViewResolver 也就是视图解析器把ModelAndView中的viewName转换成对应的View对象返回给DispatchServlet。**（ViewResolver 主要负责把String类型的viewName转换成对应的View对象）**

​	10.DispatchServlet使用View对象进行页面的展示。

#### 前后端分离执行流程

用户：1.发起请求url ——> DispatcherServlet前端控制器：2.查找能够处理的Handler ——>  HandlerMapping处理器映射器：3.返回执行链HandlerExecuttionChain(HandlerInterceptor1，HandlerInterceptor2，…) ——> DispatcherServlet前端控制器：4.请求适配器执行 ——> HandlerAdapter：5.执行Handler方法，把数据处理转换成合适的类型然后作为方法参数传入 ——> Handler处理器(Controller中能出来请求的方法)：6.@ResponseBody的话返回的ModelAndView为null。会把Handler方法返回值放入响应体当中  ——> Handler处理器(Controller中能出来请求的方法)：7.返回ModelAndView  ——> DispatcherServlet前端控制器：由于ModelAndView为null，不会执行视图解析器。

1.用户发起请求被DispatchServlet所处理

​	2.DispatchServlet通过HandlerMapping根据具体的请求查找能处理这个请求的Handler。**（HandlerMapping主要是处理请求和Handler方法的映射关系的）**

​	3.HandlerMapping返回一个能够处理请求的执行链给DispatchServlet，这个链中除了包含Handler方法也包含拦截器。

​	4.DispatchServlet拿着执行链去找HandlerAdater执行链中的方法。

​	5.HandlerAdater会去执行对应的Handler方法，把数据处理转换成合适的类型然后作为方法参数传入 

​	6.Handler方法执行完后的返回值会被HandlerAdapter转换成ModelAndView类型。由于使用了@ResponseBody注解，返回的ModelAndView会为null ，并且HandlerAdapter会把方法返回值放到响应体中。**（HandlerAdater主要进行Handler方法参数和返回值的处理。）**

​	7.返回ModelAndView给DispatchServlet。

​	8.因为返回的ModelAndView为null,所以不用去解析视图解析和其后面的操作。

