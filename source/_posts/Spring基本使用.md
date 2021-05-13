---
title: Spring基本使用
date: 2021-04-19 22:53:52
tags:
- Spring
- SSM
categories: Spring
mathjax: true
---

## Spring简介

​		目前是JavaEE开发的灵魂框架，它可以简化JavaEE的开发，可以非常方便整合其他框架，无侵入的进行功能的增强。

​		Spring的核心就是控制反转(IOC)和面向切面(AOP)。

 <!-- more --> 

## IOC控制反转

### 概念

控制反转，之前对象的控制权在类上，现在反转到Spring手上。

### SpringIOC快速入门

①导入依赖

导入SpringIOC相关依赖

```xml
<!--pom.xml-->

<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.3</version>
</dependency>
```

②编写配置文件

在resources目录下创建applicationContext.xml文件，文件名可以任意取

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="cn.xiaohupao.dao.impl.StudentDaoImpl" id="studentDao"/>
</beans>
```

class：配置类的全类名；id：配置一个唯一标识

③创建容器从容器中获取对象并测试

```java
public class Demo {
    public static void main(String[] args) {
        //创建容器
        ClassPathXmlApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");

        //获取对象
        StudentDao studentDao = (StudentDao) app.getBean("studentDao");

        //测试
        System.out.println(studentDao.getStudentById(30));
    }
}
```

### bean标签的常用属性配置

#### id

​		bean的唯一标识，同一个Spring容器中不允许重复

#### class

​		全类名，用于反射创建对象

#### scope

​		scope主要有两种：singleton和prototype。在没有配置时默认为单例(singleton)

* 如果设置为singleton：则一个容器只会有这一个bean对象。默认容器创建的时候就会创建该对象
* 如果设置为prototype：则一个容器中会有多个该bean对象。每次调用getBean方法获取时都会创建一个新对象。

## DI依赖注入概念

依赖注入可以理解为IOC的一种应用场景，反转的是对象间依赖关系维护权。

### set方法注入

​		在要注入属性的bean标签中进行配置，前提是该类有提供属性对应的set()方法

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Student {

    private String name;
    private int id;
    private int age;

    private Dog dog;

}
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Dog {
    private String name;
    private int age;
}
```

```xml
<bean class="cn.xiaohupao.domain.Student" id="student">
    <property name="name" value="wk"/>
    <property name="age" value="22"/>
    <property name="id" value="20"/>
</bean>
```

```xml
<bean class="cn.xiaohupao.domain.Dog" id="dog">
    <property name="name" value="小白"/>
    <property name="age" value="2"/>
</bean>

<bean class="cn.xiaohupao.domain.Student" id="student">
    <property name="name" value="wk"/>
    <property name="age" value="22"/>
    <property name="id" value="20"/>
    <property name="dog" ref="dog"/>
</bean>
```

### 有参构造注入

​		在要注入属性的bean标签中进行配置。前提是要有该类有提供对应的有参构造。

```xml
<bean class="cn.xiaohupao.domain.Student" id="student2">
    <constructor-arg name="name" value="wk1"/>
    <constructor-arg name="age" value="22"/>
    <constructor-arg name="id" value="21"/>
    <constructor-arg name="dog" ref="dog"/>
</bean>
```

### 复杂类型属性注入

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Phone {
    private double price;
    private String name;
    private String password;
    private String path;
}
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private int age;
    private String name;
    private Phone phone;
    private List<String> list;
    private List<Phone> phones;
    private Set<String> set;
    private Map<String, Phone> map;
    private int[] arr;
    private Properties properties;
}
```

```xml
<bean class="cn.xiaohupao.domain.Phone" id="phone">
    <constructor-arg name="name" value="iphone"/>
    <constructor-arg name="price" value="3999"/>
    <constructor-arg name="password" value="123456"/>
    <constructor-arg name="path" value="qwe"/>
</bean>

<bean class="cn.xiaohupao.domain.Phone" id="phone1">
    <constructor-arg name="name" value="iphone"/>
    <constructor-arg name="price" value="3999"/>
    <constructor-arg name="password" value="123456"/>
    <constructor-arg name="path" value="qwe"/>
</bean>

<bean class="cn.xiaohupao.domain.User" id="user">
    <property name="age" value="10"/>
    <property name="name" value="wk1"/>
    <property name="phone" ref="phone"/>
    <property name="list">
        <list>
            <value>95</value>
            <value>576</value>
        </list>
    </property>
    <property name="phones">
        <list>
            <ref bean="phone"/>
            <ref bean="phone1"/>
        </list>
    </property>
    <property name="set">
        <set>
            <value>set1</value>
            <value>set2</value>
        </set>
    </property>
    <property name="map">
        <map>
            <entry key="wk1" value-ref="phone"/>
            <entry key="wk2" value-ref="phone1"/>
        </map>
    </property>
    <property name="arr">
        <array>
            <value>1</value>
            <value>2</value>
            <value>3</value>
        </array>
    </property>
    <property name="properties">
        <props>
            <prop key="wk1">v1</prop>
            <prop key="wk1">v2</prop>
        </props>
    </property>
</bean>
```

## Lombok插件

帮助生成set get方法，构造器，hashCode，equals，toString等的方法

## SPEL

可以在配置文件中使用SPEL表达式。

```xml
<constructor-arg name="price" value="#{3888+111}"/>
<property name="phone" value="#{phone}"/>
```

SPEL需要写道value属性中，不能写道ref属性中。

## 配置文件

### 读取properties文件

我们可以让Spring读取properties文件的key/value，然后使用其中的值。

①设置读取properties

在Spring配置文件中加入如下标签：指定要读取的文件的路径。

```xml
<context:property-placeholder location="classpath:jdbc.properties"/>
```

其中的classpath表示类加载路径下。
我们也会用到如下的写法classpath:\*.properties

②使用配置文件中的值

在我们需要的时候可以使用\${key}来表示具体的值。注意要在value属性中的使用才可以。

```xml
<!--jdbc.properties -->
jdbc.url=jdbc:mysql://localhost:3306/mybatis_db?useSSL=false
jdbc.driver=com.mysql.jdbc.Driver
jdbc.username=root
jdbc.password=123456
```

```xml
<context:property-placeholder location="classpath:jdbc.properties"/>
<bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
    <property name="driverClassName" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

### 引入Spring配置文件

​		我们可以在主的配置文件中通过import标签的resource属性，引入其他的xml配置文件

```xml
<!--applicationContext.xml-->
<import resource="classpath:jdbc.xml"/>
```

## bean的配置

### name属性

​		我们可以用name属性来给bean取名，例如：

```xml
<bean class="cn.xiaohupao.domain.Dog" id="dog" name="dog1, dog2">
    <property name="name" value="小白"/>
    <property name="age" value="2"/>
</bean>
```

​		获取的时候可以使用这个名字来获取。

### lazy-init属性

​		可以控制bean的创建时间，如果设置为true就是在第一次获取该对象的时候才去创建。

### init-method属性

​		可以用来设置初始化方法，设置完后容器创建完对象就会自动帮我们调用对应的方法。
​		**注意：配置的初始化方法只能是空参的。**

### destory-method属性

​		可以用来设置销毁前调用的方法，设置完后容器销毁对象前就会自动帮我们调用对应的方法

### factory-bean&factory-method属性

​		当我们需要Spring容器使用工厂类来创建对象放入Spring容器的时候可以使用factory-bean&factory-method属性

**配置实例工厂创建对象**

```xml
<!--创建实例工厂类-->
<bean class="cn.xiaohupao.factory.CarFactory" id="carFactory"></bean>
<!--使用实例工厂创建Car放入容器-->
<!--factory-bean用来指定使用哪个工厂对象-->
<!--factory-method 用来指定使用哪个工厂方法-->
<bean factory-bean="carFactory" factory-method="getCar" id="car"></bean>
```

```java
ClassPathXmlApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
Car c = (Car) app.getBean("car");
```

**配置静态工厂创建对象**

```xml
<!--创建静态工厂类-->
<bean class="cn.xiaohupao.factory.CarStaticFactory" factory-method="getCar" id="car2"></bean>
```

## Spring注解开发

​		为了简化配置，Spring支持使用注解代替xml配置。

### 注解开发准备工作

​		如果要使用注解开发必须要开启组件扫描，这样加了注解的类才会被识别出来。Spring才能去解析其中的注解。

```xml
<!--applicationContext.xml-->
<context:component-scan base-package="cn.xiaohupao"/>
```

​		启动组件扫描，扫描对应扫描的包路径，该包及其子包下所有的类都会被扫描，加载包含指定注解的类。

### IOC相关注解

#### @Component，@Controller，@Service，@Repository

​		上述四个类都是加到类上的。它们都可以起到类似bean标签的作用。可以把加了该注解类的对象放入Spring容器中。实际上在使用的时候选择任意一个都可以。但是后三个注解是语义注解。如果是Service类要求使用@Service；如果是Dao类要求使用@Repository；如果是Controller类(SpringMVC中)要求使用@Controller；如果是其它类可以使用@Component。

### DI相关注解

​		如果一个bean已经放入Spring容器中了。那么我们可以使用下列注解实现属性注入，让Spring容器帮我们完成属性赋值。

#### @Vaule

​		主要用于String，Integer等可以直接赋值的属性注入。不依赖于setter方法，支持SPEL表达式。

```java
@Data
@Service("userService")
@AllArgsConstructor
@NoArgsConstructor
public class UserServiceImpl implements UserService {
    private UserDao userDao;
    @Value("199")
    private int num;
    @Value("xiaohupao")
    private String str;
    @Value("#{19+2}")
    private Integer age;
    @Override
    public void show() {
        userDao.show();
    }
}
```

#### @AutoWired

​		Spring会给加了该注解的属性自动注入数据类型相同的对象

```java
@Data
@Service("userService")
@AllArgsConstructor
@NoArgsConstructor
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;
    @Value("199")
    private int num;
    @Value("xiaohupao")
    private String str;
    @Value("#{19+2}")
    private Integer age;
    @Override
    public void show() {
        userDao.show();
    }
}
```

**AutoWired中的属性介绍**
@Autowired(required=false)不注入也可以，表示不是必须的。

#### @Qualifier

​		如果相同类型的bean在容器中有多个，单独使用@AutoWired就不能满足要求，这时候可以再加上@Qualifier来指定bean的名字从容器中获取bean的注入。

注意事项：该注解不能单独使用。

### xml配置文件相关注解

#### @Configuration

​		标注在类上，表示当前类是一个配置类，我们可以用注解完全替换掉xml配置文件。

​		注意：如果使用配置类替换了xml配置，spring容器要使用：AnnotationConfigApplicationContext

```java
@Configuration
@ComponentScan(basePackages = "cn.xiaohupao")
public class ApplicationConfig {
}
```

```java
public class Demo2 {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(ApplicationConfig.class);
        UserService userService = (UserService) app.getBean("userService");
        System.out.println(userService);
    }
}
```

#### @ComponentScan

​		可以用来代替context:component-scan标签来配置组件扫描。

#### @Bean

​		可以用来代替bean标签，主要用于第三方类的注入。

```java
@Configuration
@ComponentScan(basePackages = "cn.xiaohupao")

public class ApplicationConfig {

    @Bean("dataSource")
    public  DruidDataSource getDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mybatis_db?useSSL=false");
        return dataSource;
    }
}
```

​		注意事项：如果同一种类型的对象在容器中只有一个，我们可以不设置bean的名称。

#### @PropertySource

​		可以用来替换context:property-placeholder，让Spring读取指定的properties文件。然后使用@Value来获取读取到的值

```java
@Configuration
@ComponentScan(basePackages = "cn.xiaohupao")
@PropertySource("jdbc.properties")
public class ApplicationConfig {

    @Value("${jdbc.driver}")
    private String driverClassName;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;
    @Value("${jdbc.url}")
    private String url;

    @Bean("dataSource")
    public  DruidDataSource getDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setUrl(url);
        return dataSource;
    }
}
```

```xml
jdbc.url=jdbc:mysql://localhost:3306/mybatis_db?useSSL=false
jdbc.driver=com.mysql.jdbc.Driver
jdbc.username=root
jdbc.password=123456
```

​		注意事项：在使用@Value注解获取值时要使用\${key}来获取。

## 如何选择

* SSM：项目中的类和IOC和DI都使用注解，对第三方jar包中的类，配置组件扫描时使用xml进行配置
* SpringBoot：纯注解开发

## AOP

### 概念

​		AOP为Aspect Oriented Programming的缩写，意为：面向切面编程。是一种可以在不修改原来的核心代码的情况下给程序动态统一进行增强的一种技术。

​		SpringAOP：批量对Spring容器中bean的方法做增强，并且这种增强不会与原来方法中的代码耦合。

### 快速入门

#### 需求

需求让service包下所有类的所有方法在调用前都输出：方法被调用了

#### 准备工作

①添加依赖

需要添加SpringIOC相关依赖和AOP相关依赖

```xml
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.6</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.3</version>
</dependency>
```

②相关bean要注入容器中

开启组件扫描；加@Service注解

```java
@Service
public class PhoneService {

    public void deleteAll(){
        System.out.println("PhoneService中deleteAll的核心代码");
    }
}
```

```java
@Service
public class UserService {

    public void deleteAll(){
        System.out.println("UserService中deleteAll的核心代码");
    }
}
```

#### 实现AOP

①开启AOP注解支持

使用aop:sapectj-autoproxy标签

```xml
<aop:aspectj-autoproxy/>
```

或者在配置类上使用@EnableAspectJAutoProxy注解

②创建切面类

创建一个类，在类上加上@Component和@Aspect

```java
@Component
@Aspect
public class MyAspect {

    @Pointcut("execution(* cn.xiaohupao.service.impl.*.*(..))")
    public void pt(){
    }

    @Before("pt()")
    public void methodbefore(){
        System.out.println("方法被调用了！");
    }
}
```

使用@Pointcut注解来指定要被增强的方法；@Before注解来给我们的增强代码所在的方法进行标识，并且指定了增强代码是在被增强方法执行之前执行的。

③测试

```java
public class Demo3 {
    public static void main(String[] args) {
        //创建容器
        AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(ApplicationConfig.class);

        //获取对象
        PhoneService phoneService = app.getBean(PhoneService.class);
        phoneService.deleteAll();
    }
}
```

### AOP核心概念

* joinpoint(连接点)：所谓的连接点就是指那些可以被增强到的点。在spring中，这些点指的是方法，因为spring只支持方法类型的连接点。
* Pointcut(切入点)：所谓切入点是指被增强的连接点(方法)
* Advice(通知/增强)：所谓通知是具体增强的代码
* Target(目标对象)：被增强的对象就是目标对象
* Aspect(切面)：是切入点和通知(介入)的结合
* Proxy(代理)：一个类被AOP增强后，就产生一个结果代理类

### 切点确定

#### 切点表达式

可以使用切点表达式来表示要对哪些方法进行增强。

写法：execution([修饰符] 返回值类型 包名.类名.方法名(参数))

* 访问修饰符可以省略，大部分情况下可以省略
* 返回值类型、包名、类名、方法名可以使用星号\*代表任意
* 包名与类名之间一个点.代表当前包下的类，两个点表示当前包及其子包下的类
* 参数列表可以使用两个点表示任意个数，任意参数类型的参数列表

```java
execution(* cn.xiaohupao.service.*.*(..)) //表示cn.xiaohupao.service包下任意类，方法名任意，参数列表任意，返回值类型任意
execution(* cn.xiaohupao.service..*.*(..))//表示cn.xiaohupao.service包下及其子包下任意类，方法名任意，参数列表任意，返回值类型任意
execution(* cn.xiaohupao.service.*.*())//表示cn.xiaohupao.service包下任意类，方法名任意，参数列表为空参，返回值类型任意
execution(* cn.xiaohupao.service.*.delete*(..))//表示cn.xiaohupao.service包下任意类，方法名要求以delete开头，参数列表为空参，返回值类型任意
```

#### 切点函数@annotation

我们也可以在要增强的方法上加上注解。然后使用@annotation来表示对加了什么注解的方法进行增强。

①自定义一个注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface InvokeLog {
}
```

②给需要增强的方法增加注解

```java
@Service
public class PhoneService {

    @InvokeLog
    public void deleteAll(){
        System.out.println("PhoneService中deleteAll的核心代码");
    }
}
```

③自切面类这使用@annotation来确定增强的方法

```java
@Component
@Aspect
public class MyAspect {

    @Pointcut("@annotation(cn.xiaohupao.aspect.InvokeLog)")
    public void pt(){
    }

    @Before("pt()")
    public void methodbefore(){
        System.out.println("方法被调用了！");
    }
}
```

### 通知分类

* @Before：前置通知，在目标方法执行前执行
* @AfterReturning：返回后通知，在目标方法执行后执行，如果出现异常不会执行
* @After：后置通知，在目标方法返回结果之后执行，无论是否出现异常都会执行
* @AfterThrowing：异常通知，在目标方法抛出异常后执行
* @Around：环绕通知，围绕着方法执行

理解不同通知执行时机。(利用伪代码来理解单个通知的执行时机，不能用来理解多个通知情况下的执行顺序。如果需要配置多个通知我们会选择使用Around通知，更加清晰并且好用)

```java
public Object test(){
    before();//@Before前置通知
    try{
        Object ret = 目标方法();//目标方法调用
        afterReturing();//@AfterReturning返回后通知
    }catch(Throwable throwable){
        throwable.printStackTrace();
        afterThrowing();//@AafterThrowing异常通知
    }finally{
        after();//@After后置通知
    }
    return ret;
}
```

环绕通知非常特殊，它可以对目标方法进行全方位的增强。

```java
@Component
@Aspect
public class MyAspect {

    @Pointcut("@annotation(cn.xiaohupao.aspect.InvokeLog)")
    public void pt(){
    }

    @Around("pt()")
    public void methodAround(ProceedingJoinPoint pjp){
        System.out.println("目标方法前!");
        try {
            pjp.proceed();//目标方法执行
            System.out.println("目标方法后!");
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            System.out.println("出现异常通知!");
        }finally {
            System.out.println("finally中的通知！");
        }
    }
}
```

### 获取被增强方法相关信息

​		我们对实际对方法进行增强时往往还需要获取到被增强代码的相关信息，比如方法名，参数，返回值，异常对象等。

​		我们可以在除了环绕通知外的所有通知方法中增加一个joinPoint类型的参数。这个参数封装了被增强方法的相关信息。**注意：**我们可以通过这个参数获取到除了异常对象和返回值之外的所有信息。无法获取返回值和异常对象。

```java
@Before("pt1()")
public void before(JoinPoint joinPoint){
    Object[] args = joinPoint.getArgs();//方法调用时传入的参数
    Object target = joinPoint.getTarget();//被代理对象
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    Method method = signature.getMethod();
    System.out.println("方法被调用了!");
}
```

​		如果需要获取被增强方法中的异常对象或者返回值则需要在方法参数上增加一个对应类型的参数，并且使用注解的属性进行配置。这样Spring会把你想获取的数据赋值给对应的方法参数。

```java
@AfterThrowing(value = "pt1()",throwing = "e")
public void afterThrowing(JoinPoint joinPoint, Throwable e){
    System.out.println("AfterThrowable");
}
```

​		上面的获取方式特别麻烦难以理解。就可以使用下面这种万能的方法。
​		直接在环绕通知方法中增加一个ProceedingJoinPoint类型的参数。这个参数封装了被增强方法的相关信息。该参数的proceed()方法被调用相当于被增强方法被执行，调用后的返回值就相当于被增强方法的返回值。

### AOP应用案例

#### 需求

现有AI核心功能代码如下：

```java
public class AIController{
    //AI自动回答
    public String getAnswer(String question){
        String str = question.replace("吗","");
        str = str.replace("?", "!");
        return str;
    }
    
    public String fortuneTelling(String name){
        String[] strs = {"风来吴山", "玉泉鱼跃", "夕照雷锋", "云飞玉皇", "梦泉虎跑"};
        int index = name.hashCode() % 3;
        return strs[index];
    }
}
```

```java
@Controller
public class AIController {
    //AI自动回答
    public String getAnswer(String question){
        String str = question.replace("吗","");
        str = str.replace("?", "!");
        return str;
    }

    public String fortuneTelling(String name){
        String[] strs = {"风来吴山", "玉泉鱼跃", "夕照雷锋", "云飞玉皇", "梦泉虎跑"};
        int index = name.hashCode() % 5;
        return strs[index];
    }
}
```

```java
public class Demo4 {
    public static void main(String[] args) {
        //创建容器
        AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(ApplicationConfig.class);
        //获取对象
        AIController aiController = app.getBean(AIController.class);
        //调用方法
        String answer = aiController.getAnswer("你好吗?");
        System.out.println(answer);
        String wk = aiController.fortuneTelling("wk");
        System.out.println(wk);
    }
}
```

​		现在为了保证数据的安全性，要求调用的方法时fortuneTelling传入姓名是经过加密的。我们需要对传入参数进行解密后才能时候。并且要对该方法的返回值进行加密后返回。(后期也可能让其他方法进行相应的加密处理)

​		字符串加密解密直接使用下面的工具类即可：

```java
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.security.SecureRandom;

public class CryptUtil {
    private static final String AES = "AES";

    private static int keysizeAES = 128;

    private static String charset = "utf-8";

    public static String parseByte2HexStr(final byte buf[]) {
        final StringBuffer sb = new StringBuffer();
        for (int i = 0; i < buf.length; i++) {
            String hex = Integer.toHexString(buf[i] & 0xFF);
            if (hex.length() == 1) {
                hex = '0' + hex;
            }
            sb.append(hex.toUpperCase());
        }
        return sb.toString();
    }

    public static byte[] parseHexStr2Byte(final String hexStr) {
        if (hexStr.length() < 1)
            return null;
        final byte[] result = new byte[hexStr.length() / 2];
        for (int i = 0;i< hexStr.length()/2; i++) {
            int high = Integer.parseInt(hexStr.substring(i * 2, i * 2 + 1), 16);
            int low = Integer.parseInt(hexStr.substring(i * 2 + 1, i * 2 + 2), 16);
            result[i] = (byte) (high * 16 + low);
        }
        return result;
    }

    private static String keyGeneratorES(final String res, final String algorithm, final String key, final Integer keysize, final Boolean bEncode) {
        try {
            final KeyGenerator g = KeyGenerator.getInstance(algorithm);
            if (keysize == 0) {
                byte[] keyBytes = charset == null ? key.getBytes() : key.getBytes(charset);
                g.init(new SecureRandom(keyBytes));
            } else if (key == null) {
                g.init(keysize);
            } else {
                byte[] keyBytes = charset == null ? key.getBytes() : key.getBytes(charset);
                SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
                random.setSeed(keyBytes);
                g.init(keysize, random);
            }
            final SecretKey sk = g.generateKey();
            final SecretKeySpec sks = new SecretKeySpec(sk.getEncoded(), algorithm);
            final Cipher cipher = Cipher.getInstance(algorithm);
            if (bEncode) {
                cipher.init(Cipher.ENCRYPT_MODE, sks);
                final byte[] resBytes = charset == null? res.getBytes() : res.getBytes(charset);
                return parseByte2HexStr(cipher.doFinal(resBytes));
            } else {
                cipher.init(Cipher.DECRYPT_MODE, sks);
                return new String(cipher.doFinal(parseHexStr2Byte(res)));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static String AESencode(final String res) {
        return keyGeneratorES(res, AES, "aA11*-%", keysizeAES, true);
    }

    public static String AESdecode(final String res) {
        return keyGeneratorES(res, AES, "aA11*-%", keysizeAES, false);
    }

    public static void main(String[] args) {
        System.out.println(
                "加密后:" + AESencode("将要加密的明文")
        );
        System.out.println(
                "解密后:" + AESdecode("730CAE52D85B372FB161B39D0A908B8CC6EF6DA2F7D4E595D35402134C3E18AB")
        );
    }
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Crypt {
}
```

```java
@Component
@Aspect
public class CryptAspect {

    //确定切点
    @Pointcut("@annotation(cn.xiaohupao.aspect.Crypt)")
    public void pt(){}

    //定义通知
    @Around(value = "pt()")
    public String crypt(ProceedingJoinPoint pjp){
        //获取目标方法调用的参数
        Object[] args = pjp.getArgs();
        //对参数进行解密，解密后传入目标方法，执行
        String arg1 = (String) args[0];
        String s = CryptUtil.AESdecode(arg1);//解密
        args[0] = s;
        Object proceed = null;
        String ret = null;
        try {
            proceed = pjp.proceed(args);
            //目标方法执行后需要获取返回值
            ret = (String) proceed;
            //对返回值进行加密后，进行返回
            ret = CryptUtil.AESencode(ret);

        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }

        return ret;
    }
}
```

```java
public class Demo4 {
    public static void main(String[] args) {
        //创建容器
        AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(ApplicationConfig.class);
        //获取对象
        AIController aiController = app.getBean(AIController.class);

        String name = CryptUtil.AESencode("wk");
        String wk = aiController.fortuneTelling(name);
        System.out.println(wk);
        //System.out.println(CryptUtil.AESdecode(wk));
    }
}
```

### 使用xml方式配置AOP

①定义切面类

②目标类和切面类注入到容器中

③配置AOP

```xml
<aop:config>
    <!--定义切点-->
    <aop:pointcut id="pt1" expression="execution(* cn.xiaohupao.service..*.*(..))">	</aop:pointcut>
    <aop:pointcut id="pt2" expression="@annotation(cn.xiaohupao.aspect.InvokeLog)"
    <!--配置切面 -->
    <aop:aspect ref="myAspect">
        <aop:befor method="before" pointcut-ref="pt1">
        </aop:befor>
        <aop:after method="after" pointcut-ref="pt1">
        </aop:after>
        <aop:after-returning method="afterReturning" pointcut-ref="pt1" returning='ret'></aop:after-returning>
        <aop:after-throwing method="afterThrowing" pointcut-fef="pt1" throwing="e"></aop:after-throwing>
    </aop:aspect>
</aop:config>
```

### 多切面顺序问题

​		在默认情况下Spring有它自己的排序规则。(按类名排序)默认排序规则往往不符合我们的要求，我们需要进行特殊控制。如果是注解方式配置的AOP可以在切面类上加@Order注解来控制顺序。@Order中的属性越小，其优先级别越高。如果是XML方式配置的AOP，通过配置顺序来控制。

### AOP原理-动态代理

#### JDK动态代理

​		实际上Spring的AOP其实底层就是使用动态代理来完成的，并且使用了两种动态代理分别是JDK动态代理和Cglib动态代理。

```java
public class Demo {

    public static void main(String[] args) {
        AIController aiController = new AIControllerImpl();
        /*String answer = aiController.getAnswer("你好吗?");
        System.out.println(answer);*/

        //使用动态代理增强这个方法

        //使用JDK动态代理
        ClassLoader classLoader = Demo.class.getClassLoader();
        //参数1 类加载器；参数2 被代理类所实现接口的字节码对象数组；参数3 InvocationHandler实现类
        AIController o = (AIController) Proxy.newProxyInstance(classLoader, AIControllerImpl.class.getInterfaces(), new InvocationHandler() {

            //使用代理对象的方法时，会调用invoke

            /**
             *
             * @param proxy 代理对象
             * @param method 当前被调用的方法封装的
             * @param args 调用方法时传入的参数
             * @return
             * @throws Throwable
             */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

                //判断当前调用的是否是getAnswer方法
                if("getAnswer".equals(method.getName())) {
                    System.out.println("增强！");
                }
                //调用被代理对象的对应方法
                Object ret = method.invoke(aiController, args);
                return ret;

            }
        });

        String answer1 = o.getAnswer("你好吗?");
        System.out.println(answer1);
    }
}
```

#### Cglib动态代理

​		使用的是org.springframework.cglib.proxy.Enhancer类进行实现的

```java
public class CglibDemo{
    public static void main(String[] args){
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(AIControllerImpl.class);
        enhancer.setCallback(new MethodInterceptor(){
           //使用代理对象执行方法是都会调用intercept方法
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable{
                if("getAnswer".equals(method.getName())){
                    System.out.println("增强了!")
                }
                Object ret = methodProxy.invokeSuper(o, objects);
                return ret;
            }
        });
        AIContrillerImpl proxy = (AIContrillerImpl)enhancer.create();
        System.out.println(proxy.fortuneTelling("你好吗?"));
    }
}
```

**总结**

​		JDK动态代理要求被代理的类必须有接口，生成的代理对象是被代理对象的兄弟关系，或者说是代理接口的实现类。Cglib的动态代理不要求被代理的类实现接口，生成的代理对象相当于被代理对象的子类。

​		Spring的AOP默认情况下优先使用的是JDK的动态代理，如果使用不了JDK的动态代理才会使用Cglib的动态代理。

### 切换默认动态代理方式

​		如果我们是采用注解方式配置AOP的话：设置aop:aspectj-autoproxy标签的proxy-target-class属性为true，代理方式就会修改成Cglib。

```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

​		如果我们采用xml方式配置AOP的话：设置aop:config标签的proxy-taget-class属性为true，代理方式就会修改为Cglib

```xml
<aop:config proxy-target-class="true"></aop:config>
```

### Spring整合Junit

①导入依赖

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>RELEASE</version>
    <scope>test</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.5</version>
    <scope>test</scope>
</dependency>
```

②编写测试类

在测试类上加上@RunWith(SpringJUnit4ClassRunner.class)注解，指定让测试运行于Spring环境
@ContextConfiguration注解，指定Spring容器创建需要的配置文件或配置类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = ApplicationConfig.class)
public class SpringTest {

    @Autowired
    private UserService userService;
    @Test
    public void testJunit(){
        userService.show();
    }
}
```

③注入对象进行测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = ApplicationConfig.class)
public class SpringTest {

    @Autowired
    private UserService userService;
    @Test
    public void testJunit(){
        userService.show();
    }
}
```

```java
public interface UserService {
    void show();
}
```

```java
@Data
@Service("userService")
@AllArgsConstructor
@NoArgsConstructor
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;
    @Value("199")
    private int num;
    @Value("xiaohupao")
    private String str;
    @Value("#{19+2}")
    private Integer age;
    @Override
    public void show() {
        userDao.show();
    }
}
```

```java
public interface UserDao {
    void show();
}
```

```java
@Data
@NoArgsConstructor
@Repository("userDao")
public class UserDaoImpl implements UserDao {
    @Override
    public void show() {
        System.out.println("查询数据库，展示查询到的数据");
    }
}
```

### Spring整合Mybatis

①导入依赖

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.6</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.5</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.6</version>
</dependency>

<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.6</version>
</dependency>
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <properties resource="jdbc.properties"/>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <package name="cn.xiaohupao.dao"/>
    </mappers>
</configuration>
```

```java
@Data
@NoArgsConstructor
@Repository("userDao")
public class UserDaoImpl implements UserDao {
    @Override
    public void show() {
        System.out.println("查询数据库，展示查询到的数据");
    }

    @Override
    public List<User1> findAll() {
        return null;
    }
}
```

```java
public interface UserDao {
    void show();
    List<User1> findAll();
}
```

```properties
jdbc.url=jdbc:mysql://localhost:3306/mybatis_db?useSSL=false
jdbc.driver=com.mysql.jdbc.Driver
jdbc.username=root
jdbc.password=123456
```

②往容器中注入整合相关

```xml
<!--读取properties文件-->
<context:property-placeholder location="classpath:jdbc.properties"/>

<!--创建连接池注入容器-->
<bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="driverClassName" value="${jdbc.driver}"/>
</bean>

<bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <!--配置Mybatis配置文件的路径-->
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
</bean>

<!--Mapper扫描器-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer" id="mapperScannerConfigurer">
    <property name="basePackage" value="cn.xiaohupao.dao"/>
</bean>
```

### Spring声明式事务

#### 事务回顾

**事务的概念**

保证一组数据库的操作，要么同时成功，要么同时失败。

**四大特性**

* 隔离性：多个事务之间要相互隔离，不能互相干扰
* 原子性：事务是一个不可分隔的整体，类似于一个不可分割的原子
* 一致性：事务前后这组数据的状态是一致的，要么都成功，要么都失败
* 持久性：事务一旦被提交，这组操作就真的发生了变化。即使数据库故障也不应该对其有影响

#### 声明式事务的概念

​		只要简单的加个注解就可以实现事务控制，不需要事务控制的时候只需要去掉相应的注解即可。

#### 注解实现

①配置事务管理器和事务注解驱动

在spring的配置文件中添加如下配置：

```xml
<!--事务管理器注入spring容器，需要配置一个连接池-->
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!--开启事务注解驱动，配置使用的事务管理器-->
<tx:annotation-driven transaction-manager="txManager"/>
```

②添加注解

在需要进行事务控制的方法或者类上添加@Transactional注解就可以实现事务控制。

注意：如果加在类上，这个类的所有方法都会受事务控制，如果加在方法上，就是那一个方法受事务控制。

#### xml方式实现

#### 属性配置

事务传播行为propagation

| 属性值                     | 行为                                                     |
| -------------------------- | -------------------------------------------------------- |
| REQUIRED(必须要有)         | 外层方法有事务，内层方法就加入，外层没有，内层就新建。   |
| REQUIRED_NEW(必须有新事物) | 外层方法有事务，内层方法新建。外层没有，内层也新建。     |
| SUPPORTS(支持有)           | 外层方法有事务，内层方法就加入。外层没有，内层也就没有。 |
| NOT_SUPPORTED(支持没有)    | 外层方法有事务，内层方法没有。外层没有，内层也没有。     |
| MANDATORY(强制要求有)      | 外层方法有事务，内层方法加入。外层没有，内层就报错。     |
| NEVER(绝不允许有)          | 外层方法有，内层方法就报错。外层没有，内层也就没有。     |

**隔离级别isolation**

Isolation.DEFAULT 使用数据库默认隔离级别

Isolation.READ_UNCOMMITTED

Isolation.READ_COMMITTED

Isolation.REPEATABLE_READ

Isolation.SERIALIZABLE

**readOnly**

如果事务中的操作都是读操作，没有涉及到对数据库的写操作可以设置readOnly为true。这样可以提高效率。