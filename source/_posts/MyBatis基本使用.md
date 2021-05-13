---
title: MyBatis基本使用
date: 2021-04-14 22:37:04
tags:
- MyBatis
- SSM
categories: MyBatis
mathjax: true
---

## Mybaties介绍

​		Mybatis是一款优秀的持久层框架。Mybatis免除了几乎所有的JDBC代码以及设置参数和获取结果集的工作。

## 快速入门

①数据准备
MySQL表的创建，记录的添加。

 <!-- more --> 

②导入依赖

```xml
<dependencies>
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
</dependencies>
```

③编写核心配置

在资源目录下创建：mybatis-config.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="${url}"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="org/mybatis/example/BlogMapper.xml"/>
    </mappers>
</configuration>
```

修改其中的一些参数。

④定义接口及对应的xml映射文件
在资源目录下创建对应的文件xml映射文件

⑤编写测试类
获取SqiSession，通过SqiSession.getMapper获取UserDao调用对应的方法

## 入门代码初步理解

测试类中：

```java
public static void main(String[] args)throws IOException{
    String resource = "mybatis-config.xml";//定义核心配置文件的路径
    InputStream inputStream = Resources.getResourceAsStream(resource);
    //传入对应配置文件的输入流，读取配置文件获得SqlSessionFactory对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    //通过SqlSessionFactory获取SqlSession(理解为数据库连接)
    SqlSession session = sqlSessionFactory.openSession();
    //通过SqlSessionFactory获取DAO接口的实现类对象
    UserDao userDao = session.getMapper(UserDao.class);
    //释放资源
    session.close();
}
```

## 光速开发

**IDEA配置代码模板**

可以在IDEA中设置模板

**Mybatis插件**

下载Free Mybatis plugin。在接口中可以自动生成xml文件。在接口中的方法中自动生成写入xml中。

## 参数获取

### 一个参数

#### 基本参数

​		我们可以使用#{}直接来取值，写任意名字都可以获取到参数，但是一般用方法的参数名来取。

接口中的方法定义如下：

```java
User findById(Integer id);
```

xml中内容如下：

```xml
<SELECT id="findById" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user WHERE id = #{id}
</SELECT>
```

#### POJO

接口中的方法定义如下：

```java
User findByUser(user user);
```

xml中内容如下：

```xml
<SELECT id="findByUser" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user WHERE id = #{id} and username = #{username} and age = #{age} and address = #{address}
</SELECT>
```

#### Map

接口中的方法定义如下：

```java
User findByMap(Map map);
```

xml中内容如下：

```xml
<Select id="findByMap" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user WHERE id = #{id} and username = #{username} and age = #{age} and address = #{address}
</Select>
```

```java
Map map = new HashMap();
map.put("id", 2);
map.put("username", "PDD");
map.put("age", 25);
map.put("address", "上海");
userDao.findByMap(map);
```



### 多个参数

​		Mybatis会把多个参数放入一个Map集合中，默认的key是argx和paramx这种格式。

```java
User findByCondition(Integer id, String username);
```

最终map中的键值对如下：

```xml
{arg1=PDD, arg0=2, param1=2, param2=PDD}
```

虽然可以使用对应的默认Key来获取，但是这样的可读性不好。我们一般在参数前使用@Param注解来设置参数名.

接口中的方法的定义：

```java
User findByCondition(@Param("id") Integer id, @Param("username") String username);
```

这样在Mapper中可以这样获取参数：

```xml
<select id="findByCondition" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user WHERE id = #{id} AND username = #{username}
</select>
```

**总结**

如果只有一个参数的时候不用做什么处理；如果有多个参数情况下一定要加上@Param来设置参数名。

## 核心类

### SqlSessionFactory

SqlSessionFactory是一个SqlSession的工厂类，主要用于获取SqlSession对象。其成员方法方法如下：

```java
SqlSession openSession();//默认不传，则为false
//获取SqlSession对象，传入的参数代表去创建的SqlSession是否自动提交
SqlSession openSession(boolean autoCommit);
```

### SqlSession

SqlSession提供了在数据库执行SQL命令所需要的方法。它还提供了事务相关操作。其成员方法如下：

```java
T getMapper(class<T> type);//获取mapper对象
void commit();//提交事务
void rollback();//回滚事务
void close();//释放资源
```

## Mybatis实现增删改查

### 新增

①接口中增加相关方法

```java
void insertUser(User user);
```

②映射文件UserDao.xml增加相应的标签

```xml
<!-- 新增数据 -->
<insert id="insertUser">
    INSERT INTO user VALUE (null , #{username}, #{age}, #{address})
</insert>
```

注意：要记得提交事务：手动提交；或在openSession构造传入true

### 删除

①接口中增加相关方法

```java
void deleteById(Integer id);
```

②映射文件UserDao.xml增加相应的标签

```xml
<!-- 删除数据-->
<delete id="deleteById">
    DELETE FROM user WHERE id = #{id}
</delete>
```

注意：要记得提交事务：手动提交；或在openSession构造传入true

### 修改

①接口中增加相关方法

```java
void updataUser(User user);
```

②映射文件UserDao.xml增加相应的标签

```xml
<!-- 更新数据-->
<update id="updataUser">
    UPDATE user SET username = #{username}, age = #{age}, address = #{address} WHERE id = 2
</update>
```

### 查询

**根据ID查询**

①接口中增加相关方法

```java
User findById(Integer id);
```

②映射文件UserDao.xml增加相应的标签

```xml
<select id="findById" parameterType="integer" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user WHERE id=#{id}
</select>
```

**查询所有**

①接口中增加相关方法

```java
List<User> findAll();
```

②映射文件UserDao.xml增加相应的标签

```xml
<select id="findAll" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user
</select>
```

## 配置文件详解

### properties

在resources目录下有jdbc.properties文件，内容如下：

```xml
jdbc.url=jdbc:mysql://localhost:3306/mybatis_db
jdbc.driver=com.mysql.jdbc.Driver
jdbc.username=root
jdbc.password=123456
```

在mybatis-config.xml中：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--设置配置文件所在路径-->
    <properties resource="jdbc.properties">
    </properties>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!--获取配置文件中配置对应的值来设置连接相关参数 -->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/xiaohupao/dao/UserDao.xml"/>
    </mappers>
</configuration>
```

### settings

可以使用该标签来进行一些设置

例如：

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

### typeAliases

可以用来设置全类名设置别名，简化书写。一般设置一个包下的类全部具有默认别名，默认别名是类目首字母小写。例如：cn.xiaohupao.pojo.User别名为user。

```xml
<typeAliases>
    <package name="com.xiaohupao.pojo"/>
</typeAliases>
```

使用\<package>标签可以指定包名下面需要的Java Bean。也可以使用\<typeAlias alias=“User” type=“com.xiaohupao.pojo.User”/>；也可以使用@Alias注解

### environments

可以配置成适应多种环境，这种机制有助于SQL映射应用于多种数据库之中。

尽管可以配置多个环境，但每个SqlSessionFactory实例只能选择一种环境。所以如果想连接两个数据库，就需要创建两个SqlSessionFactory实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推。

### mappers

该标签的作用是加载映射，加载方式如下几种(主要使用四种)：

①使用相对于类路径的资源引用，例如：

```xml
<mappers>
    <mapper resource="com/xiaohupao/dao/UserDao.xml"/>
</mappers>
```

②使用完全限定资源定位符(URL)
③使用映射器接口实现类的完全限定类名
④将包内的映射器接口实现全部注册为映射器

```xml
<mappers>
    <package name="com/xiaohupao/dao"/>
</mappers>
```

### 打印日志

添加Log4J的jar包；配置Log4J

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

```xml
# 全局日志配置
log4j.rootLogger=ERROR, stdout
# MyBatis 日志配置
log4j.logger.org.mybatis.example.BlogMapper=TRACE
# 控制台输出
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

### 获取参数时#{}和\${}的区别

如果使用#{}它是预编译的sql可以防止SQL注入攻击；如果使用\${}它是直接把参数拿过来直接进行拼接，这样会有SQL注入的危险。

## Mybatis注解开发

我们也可以使用注解来进行开发，用注解替换掉XML。使用注解来映射简单语句会使代码显得更加简洁。对于稍微复杂的语句，Java注解不仅力不从心，还会让你本就复杂的SQL语句更加混乱不堪。所以在实际开发中一般都是使用XML的形式。

①在核心配置文件中配置mapper接口所在包名

```xml
<mappers>
    <package name="com/xiaohupao/dao"/>
</mappers>
```

②在接口对应方法上使用注解来配置需要执行的sql

```java
public interface UserDao {
    @Select("SELECT * FROM user")
    List<User> findAll();
}
```

## 动态SQL

### if

使用if标签进行条件判断，条件成立才会把if标签中的内容进行拼接。
例如：

```xml
<select id="findByCondition" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user WHERE id = #{id}
    <if test="username != null">
        AND username = #{username}
    </if>
</select>
```

如果参数username为null则执行的sql为：SELECT \* FORM user WHERE id=?
如果参数username不为null则执行的sql为：SELECT \* FORM user WHERE id=? AND       username=？

**注意**：在test属性中表示参数不需要写#{}。

### trim

可以使用该标签动态的添加前缀或后缀，也可以使用该标签动态的消除前缀。

#### prefixOverrides属性

用来设置需要被清除的前缀，多个值可以用|分隔，注意|前后不要有空格。例如and|or
例如：

```xml
<select id="findByCondition" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user
    <trim perfixOverrides="and|or">
        and 
    </trim>
</select>
```

最终执行的sql为：SELECT \* FROM user

#### suffixOverrides属性

用来设置需要被清除的后缀可以用|分隔，注意|后不要有空格。例如and|or
例如：

```xml
<select id="findByCondition" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user WHERE
    <trim suffixOverrides="and|or">
        id = #{id} and 
    </trim>
</select>
```

最终执行的sql为：SELECT * FROM user WHERE id = ？

#### prefix属性

用来设置动态添加的前缀，如果标签中有内容就会添加上设置的前缀。
例如：

```xml
<select id="findByCondition" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user
    <trim prefix="where">
        id = #{id}
    </trim>
</select>
```

最终执行的sql为：SELECT * FROM user WHERE id = ？

#### suffix属性

用来设置动态添加的后缀，如果标签中有内容就会添加上设置的后缀。

```xml
<select id="findByCondition" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user
    <trim suffix="id = #{id}">
        where
    </trim>
</select>
```

最终执行的sql为：SELECT * FROM user WHERE id = ？

#### 动态添加前缀WHERE并且消除前缀AND或者OR

```xml
<select id="findByCondition" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user
    <trim prefix="WHERE" prefixOverrides="and|or">
        <if test="id != null">
            id = #{id}
        </if>
        <if test="username != null">
            AND username = #{username}
        </if>
    </trim>
</select>
```

### where

where标签等价于：

```xml
<trim prefix="WHERE" prefixOverrides="and|or"></trim>
```

动态的拼接where并且去除前缀and或者or。

### set

set标签等价于

```xml
<trim prefix="SET" prefixOverrides=","></trim>
```

可以使用set标签动态拼接set并且去除后缀的逗号。

```xml
<update id="updateUser">
    UPDATE user
    <set>
        <if test="username != null">
            username = #{username},
        </if>
        <if test="age != null">
            age = #{age},
        </if>
        <if test="address != null">
            address = #{address},
        </if>
    </set>
    WHERE id = #{id}
</update>
```

如果调用方法传入User对象的id为2，username不为null，其他属性都为null则最终执行sql为：UPDATE USER SET username = ? WHERE id = ?

### foreach

可以使用foreach标签遍历集合或者数组类型的参数，获取其中的元素拿来动态的拼接SQL语句。

```java
List<User> findByIds(@Param("ids") Integer[] ids);
```

```xml
<select id="findByIds" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user
    <where>
        <foreach collection="ids" open="id in (" close=")" item="ida" separator=",">
            #{ida}
        </foreach>
    </where>
</select>
```

collection：表示要遍历的参数；open：表示遍历开始时拼接的语句；item：表示给当前遍历到的元素的取得名字；separator：表示遍历完一次拼接得分隔符；close：表示最后一次遍历拼接得语句。

**注意：**如果方法参数是数组类型，默认的参数名是array，如果方法参数是list集合默认的参数名是list。建议遇到数组或者集合类型的参数统一使用注解@Param进行命名。

### choose、when、otherwise

```xml
<select id="findByIds" resultType="com.xiaohupao.pojo.User">
    SELECT * FROM user
    <where>
        <choose>
            <when test="id != null">
                id = #{id}
            </when>
            <when test="username != null">
                username = #{username}
            </when>
            <otherwise>
                id = 3
            </otherwise>
        </choose>
    </where>
</select>
```

choose类似于java中的switch；when类似于java中的case；otherwise类似于java中的default。
一个choose标签中最多只会有一个when中的判断成立。从上到下进行判断。如果成立了就把标签体的内容拼接到sql中，并且不会进行其他when的判断和拼接。如果所有的when都不成立则拼接otherwise中的语句。

## SQL片段抽取

在xml映射文件中编写SQL语句的时候可能会遇到重复的SQL片段。这种SQL片段我们可以使用sql标签来进行抽取。然后在需要的时候include标签进行使用。

```xml
<sql id="base_sql">id, username, age, address</sql>
<select id = "findAll" resultType-"com.xiaohupao.pojo.User">
    SELECT <include refid="base_sql"></include>
    FROM user
</select>
```

最终执行的sql为：SELECT id, username, age, address FROM user

## 环境案例

### ResultMap

#### 基本使用

我们可以使用resultMap标签来自定义结果集和实体类属性的映射规则。

```xml
resultMap 用来自定义结果和实体类的映射
	属性：
		id 相当于resultMap的唯一标识
		type 用来指定映射到哪个实体类
	id标签：用来指定主键列的映射规则
		属性：
			property 要映射的实体类属性名
			column sql中的列明
	result标签：用来指定普通列的映射
		属性：
			property 要映射的实体类属性名
			column sql中的列明
```

```xml
<mapper namespace="com.xiaohupao.dao.OrderDao">
    <resultMap id="orderMap" type="com.xiaohupao.pojo.Order">
        <id column="id" property="id"></id>
        <result column="createtime" property="createtime"></result>
        <result column="price" property="price"></result>
        <result column="remark" property="remark"></result>
        <result column="user_id" property="userId"></result>
    </resultMap>
    <select id="findAll" resultMap="orderMap">
        SELECT * FROM orders
    </select>
</mapper>
```

#### 自动映射

我们定义resultMap时默认情况下自动映射是开启状态的。也就是如果结果集的列明和我们属性名相同是会自动映射的，我们只需要写特殊情况的映射关系即可。

自动映射的属性为autoMapping=false 则将设置自动映射关闭。

```xml
<resultMap id="orderMap" type="com.xiaohupao.pojo.Order" autoMapping="false">
</resultMap>
```

#### 继承映射关系

啊、我们可以使用resultMap的extends属性来指定一个resultMap，从而复用重复的映射关系配置。

```xml
<resultMap id="baseOrderMap" type="com.xiaohupao.pojo.Order">
    <id column="id" property="id"></id>
    <result column="createtime" property="createtime"></result>
    <result column="price" property="price"></result>
    <result column="remark" property="remark"></result>
</resultMap>

<resultMap id="baseOrderMap" type="com.xiaohupao.pojo.Order" autoMapping="false" extends="baseOrderMap">
    <result column="user_id" property="userId"></result>
</resultMap>                                                             
```

### 多表查询

#### 多表关联查询

**一对一关系**

两个实体之间是一对一关系。例如：我们需要查询订单，需求还需要下单用户的数据，这里的订单相当于用户是一对一。

接口中的定义：

```java
/**
* 根据订单id查询订单，要求把下单用户的信息也查询出来
* @param id
* @return
*/
Order findById(Integer id);
```

因为期望Order中还能包含下单用户的数据，所以可以在Order中增加一个属性

```java
private User user;
```

```mysql
SELECT o.`id`, o.`createtime`, o.`price`, o.`remark`, u.`id` AS uid, u.`username`, u.`age`, u.`address`
FROM ORDERS o, USER u
WHERE o.`user_id` = u.`id`
AND o.`id` = 2
```

我们可以使用如下两种方式封装结果集。

**使用ResultMap对所有字段进行映射**

```xml
<resultMap id="baseOrderMap" type="com.xiaohupao.pojo.Order">
    <id column="id" property="id"></id>
    <result column="createtime" property="createtime"></result>
    <result column="price" property="price"></result>
    <result column="remark" property="remark"></result>
    <result column="user_id" property="userId"></result>
</resultMap>

<resultMap id="orderUserMap" type="com.xiaohupao.pojo.Order" autoMapping="false" extends="baseOrderMap">
    <result column="uid" property="user.id"></result>
    <result column="username" property="user.username"></result>
    <result column="age" property="user.age"></result>
    <result column="address" property="user.address"></result>
</resultMap>

<select id="findById" resultMap="orderUserMap">
    SELECT o.`id`, o.`createtime`, o.`price`, o.`remark`, o.`user_id`, u.`id` AS uid, u.`username`, u.`age`, u.`address`
    FROM ORDERS o, USER u
    WHERE o.`user_id` = u.`id`
    AND o.`id` = #{id}
</select>
```

**使用ResultMap中的association**

```xml
<resultMap id="orderUserMapAssociation" type="com.xiaohupao.pojo.Order" extends="baseOrderMap">
    <association property="user" javaType="com.xiaohupao.pojo.User">
        <id column="uid" property="id"></id>
        <result column="username" property="username"></result>
        <result column="age" property="age"></result>
        <result column="address" property="address"></result>
    </association>
</resultMap>

<select id="findById" resultMap="orderUserMap">
    SELECT o.`id`, o.`createtime`, o.`price`, o.`remark`, o.`user_id`, u.`id` AS uid, u.`username`, u.`age`, u.`address`
    FROM ORDERS o, USER u
    WHERE o.`user_id` = u.`id`
    AND o.`id` = #{id}
</select>
```

**一对多关系**

两个实体类之间是一对多的关系。例如：我们需要查询用户，要求还是该用户所具有的角色信息，这里的用户相对于角色是一对多的。

接口中的定义

```java
User findById(Integer id);
```

由于一个用户可以对应多个角色，所以在实体类中增加集合用于存储角色

```java
private List<Role> roles;
```

```mysql
SELECT u.`id`, u.`username`, u.`age`, u.`address`, r.`id` AS rid, r.`name`, r.`desc`
FROM user u, user_role ur, role r
WHERE u.id = ur.user_id
AND ur.role_id = r.id
AND u.id = 2;
```

封装结果的方式：

```xml
<resultMap id="userBase" type="com.xiaohupao.pojo.User">
    <id column="id" property="id"></id>
    <result column="username" property="username"></result>
    <result column="age" property="age"></result>
    <result column="address" property="address"></result>
</resultMap>

<resultMap id="userRoleMap" type="com.xiaohupao.pojo.User" autoMapping="false" extends="userBase">
    <collection property="roles" ofType="com.xiaohupao.pojo.Role">
        <id column="rid" property="id"></id>
        <result column="name" property="name"></result>
        <result column="desc" property="desc"></result>
    </collection>
</resultMap>

<select id="findById" resultMap="userRoleMap">
    SELECT u.`id`, u.`username`, u.`age`, u.`address`, r.`id` AS rid, r.`name`, r.`desc`
    FROM user u, user_role ur, role r
    WHERE u.id = ur.user_id
    AND ur.role_id = r.id
    AND u.id = #{id}
</select>
```

#### 分部查询

​		如果有需要多表查询的需求我们也可以选择多次查询的方式来查询出我们想要的数据。Mybatis也提供了对应的配置。
​		例如我们需要查询用户。需求还需要查询出该用户所具有的角色信息。我们可以选择先查询User表查询用户信息。然后在去查询相关的角色信息。

**实现步骤**

①定义查询方法

​		因为我们要分两步查询：1.查询User；2.根据用户id查询role信息。所以我们需要定义两个方法，并且把对应的标签也写好。

查询User

```java
User findByUsername(String name);
```

```xml
<select id="findByUsername" resultType="com.xiaohupao.pojo.User">
    SELECT id, username, age, address FROM USER WHERE username = #{username}
</select>
```

根据user_id查询Role

```java
public interface RoleDao {
    List<Role> findRoleByUserId(Integer userId);
}
```

```xml
<select id="findRoleByUserId" resultType="com.xiaohupao.pojo.Role">
    SELECT r.`id`,r.`name`, r.`desc`
    FROM `user_role` ur, `role` r
    WHERE ur.role_id = r.id
    AND ur.user_id = #{userId}
</select>
```

②定义分部查询

```xml
<resultMap id="userBase" type="com.xiaohupao.pojo.User">
    <id column="id" property="id"/>
    <result column="username" property="username"/>
    <result column="age" property="age"/>
    <result column="address" property="address"/>
</resultMap>

<resultMap id="userRoleMapBySelect" type="com.xiaohupao.pojo.User" extends="userBase">
    <collection property="roles" ofType="com.xiaohupao.pojo.Role" select="com.xiaohupao.dao.RoleDao.findRoleByUserId" column="id">
    </collection>
</resultMap>

<select id="findByUsername" resultMap="userRoleMapBySelect">
    SELECT id, username, age, address FROM USER WHERE username = #{username}
</select>
```

##### 设置按需加载

​		我们可以设置按需加载，这样在我们代码中需要用到关联数据的时候才回去查询关联数据。有两种可以配置分别是全局配置和局部配置。

局部配置：设置fetchType属性为lazy

```xml
<resultMap id="userBase" type="com.xiaohupao.pojo.User">
    <id column="id" property="id"/>
    <result column="username" property="username"/>
    <result column="age" property="age"/>
    <result column="address" property="address"/>
</resultMap>

<resultMap id="userRoleMapBySelect" type="com.xiaohupao.pojo.User" extends="userBase">
    <collection property="roles" ofType="com.xiaohupao.pojo.Role" select="com.xiaohupao.dao.RoleDao.findRoleByUserId" column="id" fetchType="lazy">
    </collection>
</resultMap>

<select id="findByUsername" resultMap="userRoleMapBySelect">
    SELECT id, username, age, address FROM USER WHERE username = #{username}
</select>
```

全局配置：设置lazyLoadingEnabled为true

```xml
<!--mybatis-config.xml -->
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
    <setting name="lazyLoadingEnabled" value="true"/>
</settings>
```

### 分页查询-Pagehelper

​		我们可以使用PageHelper非常方便的帮我们实现分页查询，不需要自己在SQL中拼接相关参数，并且能非常方便的获取的总页数总条数等分页相关数据。

#### 实现步骤

①定义方法查询方法以及生成对应标签

```java
List<User> findAll();
```

```xml
<select id="findAll" resultType="com.xiaohupao.pojo.User">
    SELECT id, username, age, address FROM USER
</select>
```

②导入依赖

```xml
<!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.2.0</version>
</dependency>
```

③配置Mybatis核心配置文件使用分页插件

```xml
<!--mybatis-config.xml -->
<plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <property name="Dialect" value="mysql"/>
    </plugin>
</plugins>
```

④开始分页查询

我们只需要在使用查询方法前设置分页参数即可

```java
UserDao userDao = this.sqlSession.getMapper(UserDao.class);
PageHelper.startPage(1,2);
List<User> all = userDao.findAll();
System.out.println(all.get(1));
```

如果需要获取总页数总条数等分页相关数据，只需要创建一个PageInfo对象，把刚刚查询出的返回值作为构造方法参数传入。然后使用pageInfo对象获取即可。

```java
PageInfo<User> userPageInfo = new PageInfo<User>(all);
System.out.println("总条数: " + userPageInfo.getTotal());
System.out.println("总页数：" + userPageInfo.getPages());
System.out.println("当前页：" + userPageInfo.getPageNum());
System.out.println("每页显示长度：" + userPageInfo.getPageSize());
```

#### 一对多多表查询分页问题

​		我们在进行一对多表查询时，如果使用了PageHelper进行分页，会出现关联数据不全的情况，我们可以使用分布查询的方式解决该问题。

## Mybatis缓存

​		Mybatis的缓存其实就是把之前查到的数据存入内存(map)，下次如果还是查询相同的东西，就可以直接从缓存中取，从而提高效率。
​		Mybatis有一级缓存和二级缓存之分，一级缓存(默认开启)是sqlsession级别的缓存。二级缓存相当于mapper级别的缓存。

### 一级缓存

几种不会使用一级缓存的情况：

* 调用相同方法但是传入的参数不同
* 调用相同的方法参数也相同，但是使用的是另外的SqlSession
* 如果查询完后，对同一个表进行了增，删改的操作，都会清空这个SqlSession上的缓存
* 如果手动调用SqlSession的clearCache方法清除了缓存，后面也使用不了缓存

### 二级缓存

注意：只有SqlSession调用了close或者commit后的数据才会进入二级缓存。

#### 开启二级缓存

①全局开启

在Mybatis核心配置文件中配置

```xml
<!--mybatis-config.xml-->
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

②局部开启

在要开启二级缓存的mapper映射文件设置cache标签

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.xiaohupao.dao.UserDao">
    <cache/>
</mapper>
```

#### 使用建议

二级缓存在实际开发中基本不会使用。