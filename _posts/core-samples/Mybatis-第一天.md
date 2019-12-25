# Mybatis-第一天

## Mybatis 框架概念 特点

### Mybatis特点

​    1.属于持久层框架  持久化(将内存中的对象数据转移到数据库的过程称为持久化)

​           Mybatis   Hibernate  Spring-Data-Jpa

​    2.半自动化 ORM 框架

​           ORM:对象关系映射思想

​                 面向对象OOP                    关系型数据库

​                      类(User)                           t_user                              

​                    成员变量                              字段

​                      类对象                                 记录

​             半自动化  VS  自动化 

​            Mybatis-半自动化:

​                          1.表需要手动进行设计

​                                     主表   从表

​                          2.应用程序提供sql -基本功(以查询为主  

​                                     单表查询-(条件过滤  排序 分组 子查询 聚合查询-聚合函数)

​                                     多表连接查询

​                                    )

​                           3.依赖数据库平台

​                    优点:上手简单(基于原生的JDBC封装)   优化比较灵活  适合做互联网项目

​             Hibernate-自动化ORM 框架

​                         1.表可以通过框架自动化创建

​                         2.省掉基本的sql (基本的添加  查询 更新  删除)

​                        3.不依赖数据库平台

​                      缺陷:学习成本较高   优化难度较大  适用于传统的软件( OA |  图书馆管理系统 | ERP。。 )  不适合大型的互联网项目(电商  金融项目)

### 什么是 MyBatis？

​	MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

​      1.支持定制化sql  (程序员编写sql)

​      2.支持存储过程调用(数据库端脚本)

​      3.映射处理(结果映射)

屏蔽原生的jdbc 代码( Connection PS  ResultSet  资源关闭 )

XML 配置  |  注解配置



## Mybatis 环境搭建

### 1.构建Maven 普通工程-quick-start 工程

### 2.添加坐标

```xml
<dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>


    <!-- mybatis jar 包依赖 -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.1</version>
    </dependency>
    <!-- 数据库驱动 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.39</version>
    </dependency>

    <!-- log4j 日志打印 -->
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.16</version>
    </dependency>

<build>
    <!--
          Maven 项目
             如果源代码(src/main/java)存在xml  properties  tld 等文件  maven 默认不会自动编译该文件到输出目录
             如果要编译源代码中xml properties tld 等文件  需要显式配置resources 标签
        -->
    <resources>
      <resource>
        <directory>src/main/resources</directory>
      </resource>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.xml</include>
          <include>**/*.properties</include>
          <include>**/*.tld</include>
        </includes>
        <filtering>false</filtering>
      </resource>
    </resources>
  </build>
```

### 3.添加日志文件log4j.properties

src/main/resources目录下

```properties
# Global logging configuration
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

### 4.添加全局配置文件 mybatis.xml

src/main/resources

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
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <!--
        映射文件加载配置
    -->
    <mappers>
        <!--
           resource:包路径  com/shsxt/xxx/xxxMapper.xml
        -->
        <mapper resource="com/shsxt/mappers/UserMapper.xml"/>
    </mappers>
</configuration>
```

### 5.添加User 对象

​    src/main/java

​        com.shsxt.vo

```java
public class User {
    private  Integer id;
    private  String userName;
    private String userPwd;
    private String flag;
    private Date createTime;
    
    ...
    }
```

### 6.添加Sql 映射文件

src/main/java

​      com.shsxt.mappers

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.shsxt.mappers.UserMapper">
    <!--
       mapper:namespace 命名空间
             值全局唯一 推荐使用  包+文件名(不包含后缀): com.shsxt.mappers.UserMapper
    -->


    <!--
       查询标签 select 又称为 Statement
         标签基本属性配置
           id:Statement 唯一标识  当前文件内值唯一
           parameterType:入参类型   基本类型  String Date JavaBean  Map  数组  List
           resultType:结果类型  基本类型 String Date  JavaBean  Map  List
         标签体:sql 串
    -->
    <select id="queryUserById" parameterType="int" resultType="com.shsxt.vo.User">
       select id,user_name as userName,user_pwd as userPwd,flag,create_time as createTime
       from user
       where id=#{userId}
  </select>
</mapper>
```

7.添加测试代码

​      src/test/java

```java
    @Test
    public void test() throws IOException {
        /**
         * 1.加载全局配置文件 构建sqlSessionFactory
         * 2.获取会话SqlSession
         * 3.调用方法执行查询
         * 4.关闭会话
         */
        InputStream is = Resources.getResourceAsStream("mybatis.xml");
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
        SqlSession session = factory.openSession();
        // statament:namespace.statementId
        User user = session.selectOne("com.shsxt.mappers.UserMapper.queryUserById",75);
        System.out.println(user);
        session.close();
    }
```

### 8.启动测试

## Mysql sql 入参配置

​	基本类型(包装类型)、String、Date、JavaBean、Map、List、数组

### 基本类型

```xml
 <sql id="user_columns">
        id,user_name as userName,user_pwd as userPwd,flag,create_time as createTime
   </sql>

<select id="queryUserById" parameterType="int" resultType="com.shsxt.vo.User">
       select <include refid="user_columns"/>
       from user
       where id=#{userId}
  </select>
```

### String 类型

```xml
 <select id="queryUserByUserName" parameterType="string" resultType="com.shsxt.vo.User">
        select  <include refid="user_columns"/>  from user where user_name=#{userName}
   </select>
```

### JavaBean 类型

```xml
 <select id="queryUserByUserNameAndUserPwd" parameterType="UserQuery" resultType="User">
           <!--
               userName,userPwd  为 UserQuery 类成员变量名
           -->
        select  <include refid="user_columns"/>  from user where user_name=#{userName} and user_pwd=#{userPwd}
    </select>
```

### Map 类型

```xml
 <select id="queryUserByUserNameAndUserPwdMap" parameterType="map" resultType="User">
           <!--
               uname  map 中key 的名称
               password map 中key 的名称
           -->
        select  <include refid="user_columns"/>  from user where user_name=#{uname} and user_pwd =#{password}
    </select>
```

### 数组类型

```xml
<!--
collection  值  array|list  当数组放在map 中传入时collection值此时为key的名称!!! 
-->
<update id="updateUserPasswordByIds" >
        update user set user_pwd="111111" where id in
        <foreach collection="array" item="item"  open="(" separator="," close=")">
            #{item}
        </foreach>
    </update>
```

## Mysql sql 结果参数配置

```
结果参数:基本类型(4类8种)  String Date JavaBean  List  Map  List<Map>
resultType:String Date JavaBean  map
resultMap:属于一个标签id 值 为另一个resultMap 标签的id 
```

### 基本类型

```xml
<select id="countUser" resultType="int">
        select count(1) from user
    </select>
```

### 日期类型

```xml
  <select id="queryUserCreateTimeByUserId" parameterType="int" resultType="date">
        select create_time from user where  id=#{userId}
    </select>
```



### JavaBean 

```xml
 <sql id="user_columns">
        id,user_name as userName,user_pwd as userPwd,flag,create_time as createTime
   </sql> 
<select id="queryUserByUserName" parameterType="string" resultType="com.shsxt.vo.User">
        select  <include refid="user_columns"/>  from user where user_name=#{userName}
    </select>
```

### Map

```xml
 <select id="queryUsersByUserNameLikeMap" parameterType="string" resultType="map">
        select <include refid="user_columns"/> from user where user_name like concat('%',#{userName},'%')
    </select>
```

### List

```xml
   <select id="queryUsersByUserNameLike" parameterType="int" resultType="User">
        select <include refid="user_columns"/> from user where user_name like concat('%',#{userName},'%')
    </select>
    
    
    
     @Override
    public List<User> queryUsersByUserNameLike(String userName) {
        SqlSession session = factory.openSession();
        List<User> users = session.selectList("com.shsxt.mappers.UserMapper.queryUsersByUserNameLike",userName);
        session.close();
        return users;
    }
```

### ResultMap 接收结果

```xml
  <resultMap id="user_map" type="User">
        <!--
           column:返回的列名
           property:User 对象成员变量
        -->
        <result column="id" property="id"></result>
        <result column="user_name" property="userName"></result>
        <result column="user_pwd" property="userPwd"></result>
        <result column="flag" property="flag"></result>
        <result column="create_time" property="createTime"></result>
    </resultMap>
    <select id="queryUserById02" parameterType="int" resultMap="user_map">
        select id, user_name, user_pwd, flag, create_time
        from user
        where id=#{userId}
    </select>
```

## Mybatis 常见元素

### 1.properties 

​      注意元素的配置顺序

```xml
（properties?, settings?, typeAliases?, typeHandlers?, objectFactory?, objectWrapperFactory?, reflectorFactory?, plugins?, environments?, databaseIdProvider?, mappers?)
```

```xml
 <properties resource="jdbc.properties"></properties>
```

### 2.typeAliases

```xml
    <typeAliases>
        <!--
            配置01
        -->
      <!--  <typeAlias type="com.shsxt.vo.User" alias="User"></typeAlias>
        <typeAlias type="com.shsxt.query.UserQuery" alias="UserQuery"></typeAlias>-->


        <!--
           配置02  指定包路径  该包下所有Java Bean 均起别名 默认 类名   推荐
        -->
        <package name="com.shsxt.vo"/>
        <package name="com.shsxt.query"/>
    </typeAliases>
```

### 3.mappers

```xml
  <!--
        映射文件加载配置
    -->
    <mappers>
        <!--
           resource:包路径  com/shsxt/xxx/xxxMapper.xml
        -->
        <mapper resource="com/shsxt/mappers/UserMapper.xml"/>
        <!--<mapper class="com.shsxt.dao.AccountDao"></mapper>-->
        <package name="com.shsxt.dao"/>
    </mappers>


```

```java
public interface AccountDao {

    @Select("select id,aname,user_id as userId,money,remark,create_time as createTime,update_time as updateTime" +
            " from  account where id=#{id} ")
    public Account queryAccountById(Integer id);
}

   @Test
    public void test() throws IOException {
        InputStream is = Resources.getResourceAsStream("mybatis.xml");
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
        SqlSession session =factory.openSession();
        /**
         *  获取接口的代理对象  运行期动态为AccountDao 创建代理对象
         */
        AccountDao accountDaoProxy = session.getMapper(AccountDao.class);
        System.out.println(accountDaoProxy.queryAccountById(150));
        session.close();
    }
```

