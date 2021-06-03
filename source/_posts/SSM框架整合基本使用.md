---
title: SSM框架整合基本使用
date: 2021-05-28 23:07:11
tags:
- Spring
- Mybatis
- SpringMVC
- SSM
categories: SSM
mathjax: true
---

## SSM整合

### 步骤分析

首先来分析一下如何把Spring，SpringMVC，Mybatis整合到一起。

#### 步骤

①Spring整合上Mybatis：通过Service层Dao层都注入Spring容器中；

②引入配置SpringMVC：把Controller层注入SpringMVC容器中

③让web项目启动时自动读取spring配置文件来创建Spring容器：可以使用ContextLoaderListener来实现Spring容器的创建。

 <!-- more --> 

#### 常见疑惑

* 为什么要用两个容器？因为Controller如果不放在MVC容器中会没有效果，无法处理请求。而Service如果不放在Spring容器中，声明事务也无法使用。
* SpringMVC容器中的Controller需要依赖Service，能从Spring容器中获取到所依赖的Service对象么？Spring容器相当于父容器，MVC容器相当于是子容器。子容器除了可以使用自己容器中的对象还可以使用父容器中的对象。
* 是什么让两个容器产生这种父子容器关系的？在ContextLoaderListener中，会在创建好容器后把容器存入servletContext域。这样在DispatcgerServlet启动时，创建完SpringMVC容器后会从servletContext域中获取到Spring容器对象，设置为其父容器，这样子容器就能获取到父容器中的bean了。

### 准备工作

①创建web模块：在ProjectStructure中+web。将web文件夹放入main中，并改名为wabapp；并在pom文件中更改打包方式为war。

在pom中引入所有依赖

```xml
<dependencies>

    <!--Spring-context-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.6</version>
    </dependency>
    <!--AOP相关依赖-->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.6</version>
    </dependency>
    <!-- spring-jdbc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.3.5</version>
    </dependency>
    <!-- mybatis整合到Spring的整合包 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.6</version>
    </dependency>
    <!--mybatis依赖-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.6</version>
    </dependency>
    <!--log4j依赖，打印mybatis日志-->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    <!--分页查询，pagehelper-->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>4.0.0</version>
    </dependency>

    <!--mysql驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
    <!-- druid数据源 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.6</version>
    </dependency>

    <!-- junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <!-- spring整合junit的依赖 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.3.5</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.12</version>
        <scope>provided</scope>
    </dependency>


    <!-- servlet依赖 -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
        <scope>provided</scope>
    </dependency>
    <!--jsp依赖 -->
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.1</version>
        <scope>provided</scope>
    </dependency>
    <!--springmvc的依赖-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.6</version>
    </dependency>

    <!-- jackson，帮助进行json转换-->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.0</version>
    </dependency>

    <!--commons文件上传，如果需要文件上传功能，需要添加本依赖-->
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3.3</version>
    </dependency>

</dependencies>
```

数据库初始语句

```mysql
CREATE DATABASE /*!32312 IF NOT EXISTS*/`mybatis_db` /*!40100 DEFAULT CHARACTER SET utf8 */;
USE `mybatis_db`;
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `address` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
insert  into `user`(`id`,`username`,`age`,`address`) values (1,'UZI',19,'上海'),(2,'PDD',25,'上海');
```

### 相关配置

#### ①整合Spring和Mybatis

在resources目录下创建Spring核心配置文件：applicationContext.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.alibaba.com/schema/stat"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd http://www.alibaba.com/schema/stat http://www.alibaba.com/schema/stat.xsd">

    <!--组件扫描，排除controller-->
    <context:component-scan base-package="cn.xiaohupao">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!--读取properties文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--创建连接池注入容器-->
    <bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="driverClassName" value="${jdbc.driver}"/>
    </bean>
    <!--spring整合mybatis后控制的创建获取SqlSessionFactory的对象-->
    <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sessionFactory">
        <!--配置连接池-->
        <property name="dataSource" ref="dataSource"/>
        <!--配置mybatis配置文件的路径-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
    </bean>

    <!--mapper扫描配置，扫描到的mapper对象会被注入Spring容器中-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer" id="mapperScannerConfigurer">
        <property name="basePackage" value="cn.xiaohupao.dao"/>
    </bean>

    <!--开启aop注解支持-->
    <aop:aspectj-autoproxy/>

    <!--声明式事务相关配置-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <tx:annotation-driven/>
</beans>
```

在resources目录下创建jdbc.properties文件，内容如下：

```properties
jdbc.url=jdbc:mysql://localhost:3306/mybatis_db?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
jdbc.driver=com.mysql.jdbc.Driver
jdbc.username=root
jdbc.password=123456
```

在resources目录下创建**mybatis-config.xml** ，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <settings>
    <!--指定使用log4j打印Mybatis日志-->
    <setting name="logImpl" value="LOG4J"/>
  </settings>
  <typeAliases>
    <!--配置别名包-->
    <package name="cn.xiaohupao.domain"/>
  </typeAliases>
  <plugins>
    <!--分页助手的插件，配置在通用mapper之前-->
    <plugin interceptor="com.github.pagehelper.PageHelper">
      <!--指定方言-->
      <property name="dialect" value="mysql"/>
    </plugin>
  </plugins>
</configuration>
```

在resources目录下创建**log4j.properties** ，内容如下：

```properties
### direct log messages to stdout ###
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

### direct messages to file mylog.log ###
log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.File=c:/mylog.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

### set log levels - for more verbose logging change 'info' to 'debug' ###

log4j.rootLogger=debug, stdout
```

#### ②SpringMVC引入

在resources目录下创建spring-mvc.xml，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

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


    <!--配置视图解析器  前后端不分离项目使用-->
    <!--    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="viewResolver">
                &lt;!&ndash;要求拼接的前缀&ndash;&gt;
                <property name="prefix" value="/WEB-INF/page/"></property>
                &lt;!&ndash;要拼接的后缀&ndash;&gt;
                <property name="suffix" value=".jsp"></property>
            </bean>-->

    <!--配置拦截器-->
    <!--    <mvc:interceptors>

                <mvc:interceptor>
                    &lt;!&ndash;
                    &ndash;&gt;
                    <mvc:mapping path="/**"/>
                    &lt;!&ndash;配置排除拦截的路径&ndash;&gt;
                    <mvc:exclude-mapping path="/"/>
                    &lt;!&ndash;配置拦截器对象注入容器&ndash;&gt;
                    <bean class=""></bean>
                </mvc:interceptor>
            </mvc:interceptors>-->

    <!--
              文件上传解析器
              注意：id 必须为 multipartResolver
              如果需要上传文件时可以放开相应配置
          -->
    <!--<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">-->
    <!--&lt;!&ndash; 设置默认字符编码 &ndash;&gt;-->
    <!--<property name="defaultEncoding" value="utf-8"/>-->
    <!--&lt;!&ndash; 一次请求上传的文件的总大小的最大值，单位是字节&ndash;&gt;-->
    <!--<property name="maxUploadSize" value="#{1024*1024*100}"/>-->
    <!--&lt;!&ndash; 每个上传文件大小的最大值，单位是字节&ndash;&gt;-->
    <!--<property name="maxUploadSizePerFile" value="#{1024*1024*50}"/>-->
    <!--</bean>-->
</beans>
```

webapp中修改web.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

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
</web-app>
```

#### ③Spring整合入web项目

让web项目启动的时候就能够创建Spring容器。可以使用Spring提供的监听器ContextLoaderListener，所以我们需要在web.xml中配置这个监听器，并且配置上Spring配置文件的路径。

```xml
<!--配置spring的配置文件路径-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<!--配置监听器，可以再应用被部署的时候创建spring容器-->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

### 编写相关Controller、Service、dao

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:07
 */

@RestController
public class UserController {
    //@RestController注解的作用 = @Controller + @ResponseBody(将方法返回值转化为json放入到响应体中)

    /**
     * 从容器中获取对象，注入
     */
    @Autowired
    private UserService userService;

    @GetMapping(value = "/user/{id}")
    public User findById(@PathVariable(value = "id") Integer id){
        //@PathVariable注解 指定从路径上获取参数。
        //不加注解默认为获取QueryString格式参数
        return userService.findById(id);
    }
}
```

```java
package cn.xiaohupao.service;

import cn.xiaohupao.domain.User;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:24
 */
public interface UserService {
    /**
     * 根据id查询用户
     * @param id 用户id
     * @return 用户对象
     */
    User findById(Integer id);
}
```

```java
package cn.xiaohupao.service.impl;

import cn.xiaohupao.dao.UserDao;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:49
 */
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;
    @Override
    public User findById(Integer id) {
        return userDao.findById(id);
    }
}
```

```java
package cn.xiaohupao.dao;

import cn.xiaohupao.domain.User;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:54
 */
public interface UserDao {
    /**
     * 根据Id查询用户
     * @param id 用户id
     * @return 用户对象
     */
    User findById(Integer id);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="cn.xiaohupao.dao.UserDao">
    <select id="findById" resultType="cn.xiaohupao.domain.User">
        SELECT * FROM USER WHERE id = #{id}
    </select>
</mapper>
```



## 案例

### 响应格式统一

我们要保证一个项目中所有接口返回的数据格式的统一。这样无论是前端还是移动端开发获取到我们的数据后都能更方便的进行统一处理。

我们将定义以下结果封装类：

```java
package cn.xiaohupao.common;

import com.fasterxml.jackson.annotation.JsonInclude;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 11:06
 */
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ResponseResult<T> {

    /**
     * 状态码
     */
    private Integer code;

    /**
     * 提示信息，如果有错误，前端可以获取该字段进行提示
     */
    private String msg;

    /**
     * 查询到的结果数据
     */
    private T data;

    public ResponseResult(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public ResponseResult(Integer code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```

@JsonInclude(JsonInclude.Include.NON_NULL)这个注解可以将值为null的属性转换为json格式的时候不出现。

之前的Controller接口可以修改为返回封装的类型：

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.common.ResponseResult;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:07
 */

@RestController
public class UserController {
    //@RestController注解的作用 = @Controller + @ResponseBody(将方法返回值转化为json放入到响应体中)

    /**
     * 从容器中获取对象，注入
     */
    @Autowired
    private UserService userService;

    @GetMapping(value = "/user/{id}")
    public ResponseResult<User> findById(@PathVariable(value = "id") Integer id){
        //@PathVariable注解 指定从路径上获取参数。
        //不加注解默认为获取QueryString格式参数
        User user = userService.findById(id);
        if (user == null){
            return new ResponseResult<>(500, "没有该用户");
        }
        return new ResponseResult<>(200, "查询成功", user);
    }
}
```

### 查询所有用户

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.common.ResponseResult;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:07
 */

@RestController
public class UserController {
    //@RestController注解的作用 = @Controller + @ResponseBody(将方法返回值转化为json放入到响应体中)

    /**
     * 从容器中获取对象，注入
     */
    @Autowired
    private UserService userService;

    @GetMapping(value = "/user")
    public ResponseResult<List<User>> findAll(){
        List<User> list = userService.findAll();
        if (list == null){
            return new ResponseResult<>(500, "没有该用户");
        }
        return new ResponseResult<>(200, "查询成功",list);
    }
}
```

```java
package cn.xiaohupao.service;

import cn.xiaohupao.domain.User;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:24
 */
public interface UserService {
    /**
     * 查询所有用户
     * @return 所有的用户
     */
    List<User> findAll();
}
```

```java
package cn.xiaohupao.service.impl;

import cn.xiaohupao.dao.UserDao;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:49
 */
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    @Override
    public List<User> findAll() {
        return userDao.findAll();
    }
}
```

```java
package cn.xiaohupao.dao;

import cn.xiaohupao.domain.User;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:54
 */
public interface UserDao {
    /**
     * 查询所有用户
     * @return 所有用户信息
     */
    List<User> findAll();
}
```

```xml
<select id="findAll" resultType="cn.xiaohupao.domain.User">
    SELECT * FROM USER
</select>
```

### 分页查询用户

分页查询的结果除了要包含查询到的用户数据外还要有当前页数，每页条数，总记录数这些分页数据。

分页数据封装类：

```java
package cn.xiaohupao.common;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 12:32
 */
public class PageResult<T> {

    /**
     * 当前页数
     */
    private Integer currentPage;

    /**
     * 每页条数
     */
    private Integer pageSize;

    /**
     * 总记录数
     */
    private Integer total;

    /**
     * 查询到的数据
     */
    private List<T> data;

    public PageResult(Integer currentPage, Integer pageSize, Integer total, List<T> data) {
        this.currentPage = currentPage;
        this.pageSize = pageSize;
        this.total = total;
        this.data = data;
    }

    public Integer getCurrentPage() {
        return currentPage;
    }

    public void setCurrentPage(Integer currentPage) {
        this.currentPage = currentPage;
    }

    public Integer getPageSize() {
        return pageSize;
    }

    public void setPageSize(Integer pageSize) {
        this.pageSize = pageSize;
    }

    public Integer getTotal() {
        return total;
    }

    public void setTotal(Integer total) {
        this.total = total;
    }

    public List<T> getData() {
        return data;
    }

    public void setData(List<T> data) {
        this.data = data;
    }
}
```

分页查询实现：

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.common.ResponseResult;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:07
 */

@RestController
public class UserController {
    //@RestController注解的作用 = @Controller + @ResponseBody(将方法返回值转化为json放入到响应体中)

    /**
     * 从容器中获取对象，注入
     */
    @Autowired
    private UserService userService;

    /**
     * 分页查询所有用户
     * @param pageSize 每页条数
     * @param pageNum 指定页码
     * @return 指定页码中用户的信息
     */
    @GetMapping(value = "/user/{pageSize}/{pageNum}")
    public ResponseResult<PageResult<User>> findByPage(@PathVariable(value = "pageSize") Integer pageSize, @PathVariable(value = "pageNum") Integer pageNum){
        PageResult<User> pageResult = userService.findByPage(pageSize, pageNum);
        return new ResponseResult<>(200, "查询成功", pageResult);
    }
}
```

```java
package cn.xiaohupao.service;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.domain.User;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:24
 */
public interface UserService {

    /**
     * 分页查询用户
     * @param pageSize 每页条数
     * @param pageNum 指定页码
     * @return 当前页码中的用户信息
     */
    PageResult<User> findByPage(Integer pageSize, Integer pageNum);
}
```

```java
package cn.xiaohupao.service.impl;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.dao.UserDao;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:49
 */
@Service
public class UserServiceImpl implements UserService {

    /**
     * dao层
     */
    @Autowired
    private UserDao userDao;

    /**
     * 分页查询所有用户
     * @param pageSize 每页条数
     * @param pageNum 指定页码
     * @return 指定页码中的用户信息
     */
    @Override
    public PageResult<User> findByPage(Integer pageSize, Integer pageNum) {
        //分页查询助手类
        PageHelper.startPage(pageNum, pageSize);

        //查询所有用户
        List<User> list = userDao.findAll();

        //分页查询
        PageInfo<User> pageInfo = new PageInfo<>(list);
        return new PageResult<>(pageInfo.getPageNum(), pageInfo.getPageSize(), (int) pageInfo.getTotal(), list);
    }
}
```

### 插入查询

**Controller层**

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.common.ResponseResult;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:07
 */

@RestController
public class UserController {
    //@RestController注解的作用 = @Controller + @ResponseBody(将方法返回值转化为json放入到响应体中)

    /**
     * 从容器中获取对象，注入
     */
    @Autowired
    private UserService userService;

    /**
     * 插入用户信息
     * @param user 用户信息
     * @return 操作信息
     */
    @PostMapping(value = "/user")
    public ResponseResult insertUser(@RequestBody User user){
        userService.insertUser(user);
        return new ResponseResult(200, "插入数据成功");
    }
}
```

**Service层**

```java
package cn.xiaohupao.service;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.domain.User;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:24
 */
public interface UserService {
    /**
     * 插入用户信息
     * @param user 用户信息
     */
    void insertUser(User user);
}
```

**Dao层**

```java
package cn.xiaohupao.dao;

import cn.xiaohupao.domain.User;

import java.util.List;
    /**
     * 插入用户信息
     * @param user 用户信息
     */
    void insertUser(User user);
}
```

**Mapper映射文件**

```xml
<insert id="insertUser">
    INSERT INTO USER VALUES(null, #{username}, #{age}, #{address})
</insert>
```

**测试**

```json
{
    "username":"576",
    "age":23,
    "address":"China"
}
```

```tex
POST http://localhost:80/user
```

```json
{
    "code": 200,
    "msg": "插入数据成功"
}
```

### 删除用户

**Controller层**

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.common.ResponseResult;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:07
 */

@RestController
public class UserController {
    //@RestController注解的作用 = @Controller + @ResponseBody(将方法返回值转化为json放入到响应体中)

    /**
     * 从容器中获取对象，注入
     */
    @Autowired
    private UserService userService;
	
    /**
     * 根据id删除用户
     * @param id 用户的id
     * @return 操作信息
     */
    @GetMapping(value = "/deleteUser/{id}")
    public ResponseResult deleteUserById(@PathVariable(value = "id") Integer id){
        User user = userService.findById(id);
        if (user == null){
            return new ResponseResult(500, "没有该用户");
        }
        userService.deleteUserById(id);
        return new ResponseResult(200, "删除数据成功");
    }
}
```

**Service层**

```java
package cn.xiaohupao.service;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.domain.User;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:24
 */
public interface UserService {

    /**
     * 根据id删除用户
     * @param id 用户的id
     */
    void deleteUserById(Integer id);
}
```

```java
package cn.xiaohupao.service.impl;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.dao.UserDao;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:49
 */
@Service
public class UserServiceImpl implements UserService {

    /**
     * dao层
     */
    @Autowired
    private UserDao userDao;

    /**
     * 根据id删除用户
     * @param id 用户的id
     */
    @Override
    public void deleteUserById(Integer id) {
        userDao.deleteUserById(id);
    }
}
```

**Dao层**

```java
package cn.xiaohupao.dao;

import cn.xiaohupao.domain.User;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:54
 */
public interface UserDao {

    /**
     * 根据id删除用户
     * @param id 用户id
     */
    void deleteUserById(Integer id);
}
```

**Mapper映射文件**

```xml
<delete id="deleteUserById">
    DELETE FROM USER WHERE id = #{id}
</delete>
```

**测试**

```tex
GET http://localhost:80/deleteUser/4
```

```json
{
    "code": 200,
    "msg": "删除数据成功"
}
```

### 更新用户

**Controller层**

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.common.ResponseResult;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:07
 */

@RestController
public class UserController {
    //@RestController注解的作用 = @Controller + @ResponseBody(将方法返回值转化为json放入到响应体中)

    /**
     * 从容器中获取对象，注入
     */
    @Autowired
    private UserService userService;

    /**
     * 跟新用户
     * @param user 用户信息
     * @return 操作信息
     */
    @PutMapping(value = "/user")
    public ResponseResult updateUse(@RequestBody User user){
        userService.updateUse(user);
        return new ResponseResult(200, "数据更新成功");
    }
}
```

**Service层**

```java
package cn.xiaohupao.service;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.domain.User;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:24
 */
public interface UserService {

    /**
     * 更新用户信息
     * @param user 用户信息
     */
    void updateUse(User user);
}
```

```java
package cn.xiaohupao.service.impl;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.dao.UserDao;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:49
 */
@Service
public class UserServiceImpl implements UserService {

    /**
     * dao层
     */
    @Autowired
    private UserDao userDao;

    /**
     * 跟新用户信息
     * @param user 用户信息
     */
    @Override
    public void updateUse(User user) {
        userDao.updateUser(user);
    }
}
```

**Dao层**

```java
package cn.xiaohupao.dao;

import cn.xiaohupao.domain.User;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:54
 */
public interface UserDao {

    /**
     * 跟新用户信息
     * @param user 用户信息
     */
    void updateUser(User user);
}
```

**Mapper映射文件**

```xml
<update id="updateUser">
    UPDATE USER SET username = #{username}, age = #{age}, address = #{address} WHERE id = #{id}
</update>
```

**测试**

```json
{
    "id":5,
    "username":"95",
    "age":25,
    "address":"China"
}
```

```tex
PUT http://localhost:80/user
```

```json
{
    "code": 200,
    "msg": "数据更新成功"
}
```

## 异常统一处理

我们可以使用@ControllerAdvice实现对异常的统一处理。让异常出现时也能返回响应一个JSON。

```java
package cn.xiaohupao.exception;

import cn.xiaohupao.common.ResponseResult;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 16:43
 */
@org.springframework.web.bind.annotation.ControllerAdvice
@ResponseBody
public class ControllerAdvice {

    @ExceptionHandler(Exception.class)
    public ResponseResult handleException(Exception ex){
        //@ExceptionHandler 注解指定这个方法能处理哪些异常，括号中写入处理异常的字节码对象
        return new ResponseResult(500, ex.getMessage());
    }
}
```

## 拦截器测试

实现HandlerInterceptor接口，重写接口中的方法。在Spring-mvc中配置拦截器

```java
package cn.xiaohupao.interceptor;


import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 16:52
 */
public class InterceptorImpl implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion");
    }
}
```

```xml
<!--配置拦截器-->
<mvc:interceptors>

    <mvc:interceptor>
        <!--
                    -->
        <mvc:mapping path="/**"/>
        <!--配置排除拦截的路径-->
        <!--<mvc:exclude-mapping path="/"/>-->
        <!--配置拦截器对象注入容器-->
        <bean class="cn.xiaohupao.interceptor.InterceptorImpl"/>
    </mvc:interceptor>
</mvc:interceptors>
```

## 声明式事务测试

当作好相应配置时，只要在Service方法上加上@Transactional注解即可。

```java
package cn.xiaohupao.controller;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.common.ResponseResult;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:07
 */

@RestController
public class UserController {
    //@RestController注解的作用 = @Controller + @ResponseBody(将方法返回值转化为json放入到响应体中)

    /**
     * 从容器中获取对象，注入
     */
    @Autowired
    private UserService userService;

    @RequestMapping(value = "/user/test")
    public ResponseResult test(){
        userService.test();
        return new ResponseResult(200, "操作成功");
    }
}
```

```java
package cn.xiaohupao.service;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.domain.User;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:24
 */
public interface UserService {

    void test();
}
```

```java
package cn.xiaohupao.service.impl;

import cn.xiaohupao.common.PageResult;
import cn.xiaohupao.dao.UserDao;
import cn.xiaohupao.domain.User;
import cn.xiaohupao.service.UserService;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 9:49
 */
@Service
public class UserServiceImpl implements UserService {

    /**
     * dao层
     */
    @Autowired
    private UserDao userDao;

    @Override
    @Transactional
    public void test() {
        userDao.insertUser(new User(null, "test1", 11, "cc"));
        System.out.println(1/0);
        userDao.insertUser(new User(null, "test2", 12, "cc"));
    }
}
```

## AOP测试

注意，自己去使用AOP进行增强时，应该是对Service进行增强。不能对Controller进行增强，因为切面类会被放入父容器，而我们的Controller是在子容器中的。父容器不能访问子容器。
并且我们如果需要对Controller进行增强，使用拦截器也会更加的好用。

```java
package cn.xiaohupao.aspect;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

/**
 * @Author: xiaohupao
 * @Date: 2021/5/29 18:17
 */
@Aspect
@Component
public class AspectTest {

    /**
     * 定义切点
     */
    @Pointcut("execution(* cn.xiaohupao.service.*.*(..))")
    public void pt(){
    }

    /**
     * 方法增强
     */
    @Before(value = "pt()")
    public void before(){
        System.out.println("before");
    }
}
```

