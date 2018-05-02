# mybatis学习笔记(3)-入门程序二

标签： mybatis

---

**Contents**

  - [映射文件](#映射文件)
  - [程序代码](#程序代码)
    - [Error updating database.  Cause: org.apache.ibatis.executor.ExecutorException: A query was run and no Result Maps were found for the Mapped Statement 'test.insertUser!selectKey'.  It's likely that neither a Result Type nor a Result Map was specified.](#error-updating-database--cause-orgapacheibatisexecutorexecutorexception-a-query-was-run-and-no-result-maps-were-found-for-the-mapped-statement-testinsertuserselectkey--its-likely-that-neither-a-result-type-nor-a-result-map-was-specified)
    - [The error may exist in sqlmap/User.xml](#the-error-may-exist-in-sqlmapuserxml)
    - [The error may involve test.insertUser!selectKey-Inline](#the-error-may-involve-testinsertuserselectkey-inline)
    - [The error occurred while setting parameters](#the-error-occurred-while-setting-parameters)
    - [SQL: SELECT LAST_INSERT_ID()](#sql-select-last_insert_id)
    - [Cause: org.apache.ibatis.executor.ExecutorException: A query was run and no Result Maps were found for the Mapped Statement 'test.insertUser!selectKey'.  It's likely that neither a Result Type nor a Result Map was specified.](#cause-orgapacheibatisexecutorexecutorexception-a-query-was-run-and-no-result-maps-were-found-for-the-mapped-statement-testinsertuserselectkey--its-likely-that-neither-a-result-type-nor-a-result-map-was-specified)
  - [总结](#总结)
  - [mybatis和hibernate本质区别和应用场景](#mybatis和hibernate本质区别和应用场景)



---


添加、删除、更新用户


## 映射文件

 - User.xml,在入门程序一基础上增加(增删改)

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
    <select id="findUserById" parameterType="String" resultType="com.iot.mybatis.po.User">
        SELECT * FROM  user  WHERE id=#{value}
    </select>
    <!-- 根据用户名称模糊查询用户信息，可能返回多条
	resultType：指定就是单条记录所映射的java对象类型
	${}:表示拼接sql串，将接收到参数的内容不加任何修饰拼接在sql中。
	使用${}拼接sql，引起 sql注入
	${value}：接收输入参数的内容，如果传入类型是简单类型，${}中只能使用value
	 -->
    <select id="findUserByName" parameterType="java.lang.String" resultType="com.iot.mybatis.po.User">
        SELECT * FROM user WHERE username LIKE '%${value}%'
    </select>

    <insert id="insertUser" parameterType="com.iot.mybatis.po.User">
        <!--
         将插入数据的主键返回，返回到user对象中

         SELECT LAST_INSERT_ID()：得到刚insert进去记录的主键值，只适用与自增主键

         keyProperty：将查询到主键值设置到parameterType指定的对象的哪个属性
         order：SELECT LAST_INSERT_ID()执行顺序，相对于insert语句来说它的执行顺序
         resultType：指定SELECT LAST_INSERT_ID()的结果类型

        <selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
            SELECT LAST_INSERT_ID()
        </selectKey>
        INSERT INTO user (username,birthday,sex,address)values (#{username},#{birthday},#{sex},#{address})
        -->
        <!--
            使用mysql的uuid（）生成主键
            执行过程：
            首先通过uuid()得到主键，将主键设置到user对象的id属性中
            其次在insert执行时，从user对象中取出id属性值
             -->
        <selectKey keyProperty="id" order="BEFORE" resultType="java.lang.String">
            SELECT uuid()
        </selectKey>
        insert into user(id,username,birthday,sex,address) value(#{id},#{username},#{birthday},#{sex},#{address})

    </insert>

    <!-- 删除 用户
        根据id删除用户，需要输入 id值
         -->
    <delete id="deleteUser" parameterType="java.lang.String">
        delete from user where id=#{id}
    </delete>

    <!-- 根据id更新用户
    分析：
    需要传入用户的id
    需要传入用户的更新信息
    parameterType指定user对象，包括 id和更新信息，注意：id必须存在
    #{id}：从输入 user对象中获取id属性值
     -->
    <update id="updateUser" parameterType="com.iot.mybatis.po.User">
        update user set username=#{username},birthday=#{birthday},sex=#{sex},address=#{address}
        where id=#{id}
    </update>
</mapper>
```


(注：这里的`birthday`字段在mysql表中是DATE类型，在User类中`birthday`属性是java的`java.util.Date`类型，并没有进行转换就插入成功了。

看到有的文章说，在字段中有Date和DateTime类型，在插入数据时只要将实体的属性设置成Timestamp就会对应mysql的DateTime类型，Date会对应mysql的Date类型:
`#{modified_date,jdbcType=TIMESTAMP}、#{date,jdbcType=DATE}`

我上面的`birthday`，配置成`#{birthday,jdbcType=TIMESTAMP}`，结果也插入成功了，具体实现待查)

## 程序代码

- User.java,在入门程序一基础上增加三个测试方法

```java
 package com.iot.mybatis.po;

import java.util.Date;

/**
 * Created by Administrator on 2016/2/21.
 */
public class User {
    //属性名要和数据库表的字段对应
    private String id;
    private String username;// 用户姓名
    private String sex;// 性别
    private Date birthday;// 生日
    private String address;// 地址

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User [id=" + id + ", username=" + username + ", sex=" + sex
                + ", birthday=" + birthday + ", address=" + address + "]";
    }
}
```

Main.java

```java
package com.iot.mybatis.po;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;
import java.util.Date;

public class Main {

    public static void main(String[] args) throws IOException {
        System.out.println("Hello World!");
        User user = new User();
        user.setUsername("王大军");
        user.setBirthday(new Date());
        user.setSex("1");
        user.setAddress("河南郑州");
        insertUser(user);

        user = new User();
        user.setUsername("王二军");
        user.setBirthday(new Date());
        user.setSex("1");
        user.setAddress("河南郑州");
        insertUser(user);

        user = new User();
        user.setUsername("王小军");
        user.setBirthday(new Date());
        user.setSex("2");
        user.setAddress("河南郑州");
        insertUser(user);

        List<User> users =  findUserByName("王");

        System.out.println(users);

        user = users.get(0);

        user.setAddress("湖南长沙");

        updateUser(user);

       User ussechage=  findUserById(user.getId());
        System.out.println(ussechage);
        for (User u:users
             ) {
         deleteUser(u.getId());
        }
    }

    public static void insertUser(User user) throws IOException {
        // mybatis配置文件
        String resource = "SqlMapConfig.xml";
        // 得到配置文件流
        InputStream inputStream = Resources.getResourceAsStream(resource);

        // 创建会话工厂，传入mybatis的配置文件信息
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder()
                .build(inputStream);

        // 通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 插入用户对象

        sqlSession.insert("test.insertUser", user);
        // 提交事务
        sqlSession.commit();

        // 获取用户信息主键
        System.out.println(user.getId());
        // 关闭会话
        sqlSession.close();

    }

    public static void deleteUser(String id) throws IOException {
        // mybatis配置文件
        String resource = "SqlMapConfig.xml";
        // 得到配置文件流
        InputStream inputStream = Resources.getResourceAsStream(resource);

        // 创建会话工厂，传入mybatis的配置文件信息
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder()
                .build(inputStream);

        // 通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();

        // 传入id删除 用户
        sqlSession.delete("test.deleteUser", id);

        // 提交事务
        sqlSession.commit();

        // 关闭会话
        sqlSession.close();

    }

    public static void updateUser(User user) throws IOException {
        // mybatis配置文件
        String resource = "SqlMapConfig.xml";
        // 得到配置文件流
        InputStream inputStream = Resources.getResourceAsStream(resource);

        // 创建会话工厂，传入mybatis的配置文件信息
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder()
                .build(inputStream);

        // 通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 更新用户信息

        sqlSession.update("test.updateUser", user);

        // 提交事务
        sqlSession.commit();

        // 关闭会话
        sqlSession.close();

    }

    public static User findUserById(String id) throws IOException {
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
        User user = sqlSession.selectOne("test.findUserById", id);
        // 释放资源
        sqlSession.close();
        return user;
    }

    public static List<User> findUserByName(String name) throws IOException {
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
        List<User> list = sqlSession.selectList("test.findUserByName", name);

        sqlSession.close();

        return list;
    }
}

```


- 自增主键返回

```xml
<selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
          SELECT LAST_INSERT_ID()
</selectKey>
```

如果没有在上面的配置中配置`resultType`，则会报下面的异常


```
org.apache.ibatis.exceptions.PersistenceException: 
### Error updating database.  Cause: org.apache.ibatis.executor.ExecutorException: A query was run and no Result Maps were found for the Mapped Statement 'test.insertUser!selectKey'.  It's likely that neither a Result Type nor a Result Map was specified.
### The error may exist in sqlmap/User.xml
### The error may involve test.insertUser!selectKey-Inline
### The error occurred while setting parameters
### SQL: SELECT LAST_INSERT_ID()
### Cause: org.apache.ibatis.executor.ExecutorException: A query was run and no Result Maps were found for the Mapped Statement 'test.insertUser!selectKey'.  It's likely that neither a Result Type nor a Result Map was specified.

	...

Caused by: org.apache.ibatis.executor.ExecutorException: A query was run and no Result Maps were found for the Mapped Statement 'test.insertUser!selectKey'.  It's likely that neither a Result Type nor a Result Map was specified.


```


## 总结

- `#{}`和`${}`

`#{}`表示一个占位符号，`#{}`接收输入参数，类型可以是简单类型，pojo、hashmap。

如果接收简单类型，`#{}`中可以写成value或其它名称。

`#{}`接收pojo对象值，通过OGNL读取对象中的属性值，通过属性.属性.属性...的方式获取对象属性值。

`${}`表示一个拼接符号，会引用sql注入，所以**不建议使用`${}`**。

`${}`接收输入参数，类型可以是简单类型，pojo、hashmap。

如果接收简单类型，`${}`中只能写成value。

`${}`接收pojo对象值，通过OGNL读取对象中的属性值，通过属性.属性.属性...的方式获取对象属性值。



## mybatis和hibernate本质区别和应用场景

- hibernate

是一个标准ORM框架（对象关系映射）。入门门槛较高的，不需要程序写sql，sql语句自动生成了。对sql语句进行优化、修改比较困难的。

应用场景：适用与需求变化不多的中小型项目，比如：后台管理系统，erp、orm、oa。。

- mybatis

专注是sql本身，需要程序员自己编写sql语句，sql修改、优化比较方便。mybatis是一个不完全的ORM框架，虽然程序员自己写sql，mybatis也可以实现映射（输入映射、输出映射）。

应用场景：适用与需求变化较多的项目，比如：互联网项目。

企业进行技术选型，以低成本高回报作为技术选型的原则，根据项目组的技术力量进行选择。


----

> 作者[@brianway](http://brianway.github.io/)更多文章：[个人网站](http://brianway.github.io/) | [CSDN](http://blog.csdn.net/h3243212/) | [oschina](http://my.oschina.net/brianway)


