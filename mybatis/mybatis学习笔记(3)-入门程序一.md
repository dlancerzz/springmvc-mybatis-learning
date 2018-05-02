# mybatis学习笔记(3)-入门程序一

标签： mybatis

---

**Contents**

  - [工程结构](#工程结构)
- [Global logging configuration](#global-logging-configuration)
- [Console output...](#console-output)
  - [映射文件](#映射文件)
  - [程序代码](#程序代码)
  - [总结](#总结)



---

mybatis入门程序

## 工程结构


在IDEA中新建了一个普通的java项目，新建文件夹lib,加入jar包,工程结构如图。

![mybatis_入门程序一-工程结构图](http://7xph6d.com1.z0.glb.clouddn.com/mybatis_%E5%85%A5%E9%97%A8%E7%A8%8B%E5%BA%8F%E4%B8%80-%E5%B7%A5%E7%A8%8B%E7%BB%93%E6%9E%84%E5%9B%BE.png)



- asm

ASM 是一个 Java 字节码操纵框架。它可以直接以二进制形式动态地生成 stub 类或其他代理类，或者在装载时动态地修改类。ASM 提供类似于 BCEL 和 SERP 之类的工具包的功能，但是被设计得更小巧、更快速，这使它适用于实时代码插装。

- cglib

CGLIB(Code Generation Library)是一个开源项目！
是一个强大的，高性能，高质量的Code生成类库，它可以在运行期扩展Java类与实现Java接口。Hibernate支持它来实现PO(Persistent Object 持久化对象)字节码的动态生成。

- commons-logging

Jakarta  Commons-logging（JCL）是apache最早提供的日志的门面接口。提供简单的日志实现以及日志解耦功能。
JCL能够选择使用Log4j（或其他如slf4j等）还是JDK Logging，但是他不依赖Log4j，JDK Logging的API。如果项目的classpath中包含了log4j的类库，就会使用log4j，否则就使用JDK Logging。使用commons-logging能够灵活的选择使用那些日志方式，而且不需要修改源代码。（类似于JDBC的API接口）

- javassist

Javassist是一个动态类库，可以用来检查、”动态”修改以及创建 Java类。其功能与jdk自带的反射功能类似，但比反射功能更强大。

- slf4j

SLF4J，即简单日志门面（Simple Logging Facade for Java），不是具体的日志解决方案，它只服务于各种各样的日志系统。按照官方的说法，SLF4J是一个用于日志系统的简单Facade，允许最终用户在部署其应用时使用其所希望的日志System

- log4j.properties

```
# Global logging configuration
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n

```



- SqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 和spring整合后 environments配置将废除-->
    <environments default="development">
        <environment id="development">
            <!-- 使用jdbc事务管理，事务控制由mybatis-->
            <transactionManager type="JDBC" />
            <!-- 数据库连接池,由mybatis管理-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://120.25.162.238:3306/mybatis001?characterEncoding=utf-8" />
                <property name="username" value="root" />
                <property name="password" value="123" />
            </dataSource>
        </environment>
    </environments>

</configuration>
```

## 映射文件

- sqlmap/Accounts.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace 命名空间，作用就是对sql进行分类化管理,理解为sql隔离
 注意：使用mapper代理方法开发，namespace有特殊重要的作用
 -->
<mapper namespace="test">
    <!-- 在映射文件中配置很多sql语句 -->
    <!--需求:通过id查询用户表的记录 -->
    <!-- 通过select执行数据库查询
     id:标识映射文件中的sql，称为statement的id
     将sql语句封装到mappedStatement对象中，所以将id称为statement的id
     parameterType:指定输入参数的类型
     #{}标示一个占位符,
     #{value}其中value表示接收输入参数的名称，如果输入参数是简单类型，那么#{}中的值可以任意。

     resultType：指定sql输出结果的映射的java对象类型，select指定resultType表示将单条记录映射成java对象
     -->
    <select id="findUserById" parameterType="int" resultType="com.iot.mybatis.po.Accounts">
        SELECT * FROM  Accounts  WHERE ID=#{value}
    </select>

    <!-- 根据用户名称模糊查询用户信息，可能返回多条
	resultType：指定就是单条记录所映射的java对象类型
	${}:表示拼接sql串，将接收到参数的内容不加任何修饰拼接在sql中。
	使用${}拼接sql，引起 sql注入
	${value}：接收输入参数的内容，如果传入类型是简单类型，${}中只能使用value
	 -->
    <select id="findUserByName" parameterType="java.lang.String" resultType="com.iot.mybatis.po.Accounts">
        SELECT * FROM Accounts WHERE Nickname LIKE '%${value}%'
    </select>
</mapper>
```




在sqlMapConfig.xml中加载Accounts.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 和spring整合后 environments配置将废除-->
    <environments default="development">
        <environment id="development">
            <!-- 使用jdbc事务管理，事务控制由mybatis-->
            <transactionManager type="JDBC" />
            <!-- 数据库连接池,由mybatis管理-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://rm-bp1dt9mpug49w8c45jo.mysql.rds.aliyuncs.com/test_db?Port=3306&amp;useSSL=false" />
                <property name="username" value="app" />
                <property name="password" value="App@2176109" />
            </dataSource>
        </environment>
    </environments>
    <!-- 加载映射文件-->
    <mappers>
        <mapper resource="sqlmap/Accounts.xml"/>
    </mappers>
</configuration>
```

## 程序代码

- po类`Accounts.java`

```java
package com.iot.mybatis.po;
import java.util.Date;

public class Accounts {
    //属性名要和数据库表的字段对应
    private int ID;
    private String Nickname;// 用户姓名
    private Date CreateTime;// 生日
    private String Email;// 地址

    public int getID() {
        return ID;
    }

    public void setID(int ID) {
        this.ID = ID;
    }

    public String getNickname() {
        return Nickname;
    }

    public void setNickname(String Nickname) {
        this.Nickname = Nickname;
    }


    public Date getCreateTime() {
        return CreateTime;
    }

    public void setCreateTime(Date CreateTime) {
        this.CreateTime = CreateTime;
    }

    public String getEmail() {
        return Email;
    }

    public void setEmail(String Email) {
        this.Email = Email;
    }

    @Override
    public String toString() {
        return "Accounts [ID=" + ID + ", Nickname=" + Nickname + ", CreateTime="
                + CreateTime + ", Email=" + Email + "]";
    }
}

```


- 测试代码

```java
package com.iot.mybatis.po;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class Main {

    public static void main(String[] args) throws IOException {
        System.out.println("Hello World!");
        findUserByIdTest();

        findUserByNameTest();
    }

    public static void findUserByIdTest() throws IOException {
        // mybatis配置文件
        String resource = "SqlMapConfig.xml";
        // 得到配置文件流
        InputStream inputStream =  Resources.getResourceAsStream(resource);
        //创建会话工厂，传入mybatis配置文件的信息
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        // 通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 通过SqlSession操作数据库
        // 第一个参数：映射文件中statement的id，等于=namespace+"."+statement的id
        // 第二个参数：指定和映射文件中所匹配的parameterType类型的参数
        // sqlSession.selectOne结果 是与映射文件中所匹配的resultType类型的对象
        // selectOne查询出一条记录
        Accounts user = sqlSession.selectOne("test.findUserById", 1);
        System.out.println(user);
        // 释放资源
        sqlSession.close();
    }

    public static void findUserByNameTest() throws IOException {
        // mybatis配置文件
        String resource = "SqlMapConfig.xml";
        // 得到配置文件流
        InputStream inputStream = Resources.getResourceAsStream(resource);
        // 创建会话工厂，传入mybatis的配置文件信息
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder()
                .build(inputStream);
        // 通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // list中的user和映射文件中resultType所指定的类型一致
        List<Accounts> list = sqlSession.selectList("test.findUserByName", "管理员");
        System.out.println(list);
        sqlSession.close();
    }
}

```


输出：

```
C:\Java\jdk1.8.0_171\bin\java "-javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2018.1\lib\idea_rt.jar=56153:C:\Program Files\JetBrains\IntelliJ IDEA 2018.1\bin" -Dfile.encoding=UTF-8 -classpath C:\Java\jdk1.8.0_171\jre\lib\charsets.jar;C:\Java\jdk1.8.0_171\jre\lib\deploy.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\access-bridge-64.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\cldrdata.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\dnsns.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\jaccess.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\jfxrt.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\localedata.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\nashorn.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\sunec.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\sunjce_provider.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\sunmscapi.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\sunpkcs11.jar;C:\Java\jdk1.8.0_171\jre\lib\ext\zipfs.jar;C:\Java\jdk1.8.0_171\jre\lib\javaws.jar;C:\Java\jdk1.8.0_171\jre\lib\jce.jar;C:\Java\jdk1.8.0_171\jre\lib\jfr.jar;C:\Java\jdk1.8.0_171\jre\lib\jfxswt.jar;C:\Java\jdk1.8.0_171\jre\lib\jsse.jar;C:\Java\jdk1.8.0_171\jre\lib\management-agent.jar;C:\Java\jdk1.8.0_171\jre\lib\plugin.jar;C:\Java\jdk1.8.0_171\jre\lib\resources.jar;C:\Java\jdk1.8.0_171\jre\lib\rt.jar;C:\java_code\mybatis01\out\production\mybatis01;C:\java_code\mybatis01\lib\asm-3.3.1.jar;C:\java_code\mybatis01\lib\cglib-2.2.2.jar;C:\java_code\mybatis01\lib\commons-logging-1.1.1.jar;C:\java_code\mybatis01\lib\javassist-3.17.1-GA.jar;C:\java_code\mybatis01\lib\log4j-api-2.6.3-CUSTOM.jar;C:\java_code\mybatis01\lib\log4j-core-2.6.3-CUSTOM.jar;C:\java_code\mybatis01\lib\log4j-1.2.17.jar;C:\java_code\mybatis01\lib\mysql-connector-java-5.1.34_1.jar;C:\java_code\mybatis01\lib\org.apache.felix.ipojo.annotations-1.12.1.jar;C:\java_code\mybatis01\lib\abstract-jdbc-driver-0.5.jar;C:\java_code\mybatis01\lib\mybatis-3.2.7.jar;C:\java_code\mybatis01\lib\slf4j-api-1.7.5.jar;C:\java_code\mybatis01\lib\slf4j-log4j12-1.7.5.jar com.iot.mybatis.po.Main
Hello World!
DEBUG [main] - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] - Opening JDBC Connection
DEBUG [main] - Created connection 876563773.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@343f4d3d]
DEBUG [main] - ==>  Preparing: SELECT * FROM Accounts WHERE ID=? 
DEBUG [main] - ==> Parameters: 1(Integer)
DEBUG [main] - <==      Total: 1
Accounts [ID=1, Nickname=管理员, CreateTime=Wed May 31 10:02:23 CST 2017, Email=]
DEBUG [main] - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@343f4d3d]
DEBUG [main] - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@343f4d3d]
DEBUG [main] - Returned connection 876563773 to pool.
DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] - Opening JDBC Connection
DEBUG [main] - Created connection 1376400422.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@520a3426]
DEBUG [main] - ==>  Preparing: SELECT * FROM Accounts WHERE Nickname LIKE '%管理员%' 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 15
[Accounts [ID=1, Nickname=管理员, CreateTime=Wed May 31 10:02:23 CST 2017, Email=], Accounts [ID=15, Nickname=管理员, CreateTime=Thu Aug 10 08:41:25 CST 2017, Email=], Accounts [ID=16, Nickname=管理员, CreateTime=Fri Aug 11 10:59:00 CST 2017, Email=], Accounts [ID=17, Nickname=管理员, CreateTime=Fri Aug 11 10:59:45 CST 2017, Email=], Accounts [ID=24, Nickname=鲁滨管理员, CreateTime=Thu Aug 24 12:46:32 CST 2017, Email=], Accounts [ID=25, Nickname=熠点管理员, CreateTime=Fri Aug 25 17:40:13 CST 2017, Email=], Accounts [ID=26, Nickname=芙蓉管理员, CreateTime=Mon Aug 28 16:21:06 CST 2017, Email=], Accounts [ID=30, Nickname=管理员, CreateTime=Fri Sep 01 15:52:13 CST 2017, Email=], Accounts [ID=32, Nickname=世纪缘管理员, CreateTime=Tue Sep 05 18:09:01 CST 2017, Email=], Accounts [ID=37, Nickname=管理员, CreateTime=Mon Sep 11 10:41:51 CST 2017, Email=], Accounts [ID=38, Nickname=蓝盒子管理员, CreateTime=Wed Sep 13 09:26:04 CST 2017, Email=], Accounts [ID=50, Nickname=管理员, CreateTime=Tue Sep 19 11:39:56 CST 2017, Email=], Accounts [ID=154, Nickname=城阳周大生管理员, CreateTime=Wed Oct 25 11:33:31 CST 2017, Email=], Accounts [ID=174, Nickname=澳德乐管理员, CreateTime=Mon Oct 30 09:41:13 CST 2017, Email=], Accounts [ID=308, Nickname=万福管理员, CreateTime=Fri Jan 12 22:16:15 CST 2018, Email=]]
DEBUG [main] - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@520a3426]
DEBUG [main] - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@520a3426]
DEBUG [main] - Returned connection 1376400422 to pool.

Process finished with exit code 0

```

## 坑

```
Hello World!
Exception in thread "main" java.io.IOException: Could not find resource SqlMapConfig.xml
	at org.apache.ibatis.io.Resources.getResourceAsStream(Resources.java:110)
	at org.apache.ibatis.io.Resources.getResourceAsStream(Resources.java:97)
	at com.iot.mybatis.po.Main.findUserByIdTest(Main.java:25)
	at com.iot.mybatis.po.Main.main(Main.java:16)

Process finished with exit code 1
```
如果你的输出是这样，你需要将`config`目录作为资源的根节点（Mark Directory as Resources Root）。


## 总结

- `parameterType`

在映射文件中通过parameterType指定输入参数的类型


- `resultType`

在映射文件中通过resultType指定输出结果的类型


- `#{}`和`${}`

`#{}`表示一个占位符号;

`${}`表示一个拼接符号，会引起sql注入，所以不建议使用


- `selectOne`和`selectList`

`selectOne`表示查询一条记录进行映射，使用`selectList`也可以使用，只不过只有一个对象

`selectList`表示查询出一个列表(参数记录)进行映射，不能够使用`selectOne`查，不然会报下面的错:

```
org.apache.ibatis.exceptions.TooManyResultsException: Expected one result (or null) to be returned by selectOne(), but found: 3
```




----

> 作者[@brianway](http://brianway.github.io/)更多文章：[个人网站](http://brianway.github.io/) | [CSDN](http://blog.csdn.net/h3243212/) | [oschina](http://my.oschina.net/brianway)




