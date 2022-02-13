---
title: Spring JdbcTemplate 整合c3p0  设计 dao 层
comments: true
date: 2018-08-16 10:50:49
updated: 2018-08-16 10:50:49
tags:
- Spring
- Jdbc
- Mysql
categories:
- java
---

Spring的JdbcTemplate使用起来比jdbc简便很多,不用自己写很多重复的代码(不用手动关闭连接!).使用jdbcTemplate和c3p0设计一个通用的dao类,方便以后继承.
<!--more-->

### 什么是JDBC？
{% blockquote %}
①JDBC（Java DataBase Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。而多的这个template,就是模板,是Spring框架为我们提供的.所以JDBCTemplate就是Spring对JDBC的封装,通俗点说就是Spring对jdbc的封装的模板
{% endblockquote %}

### 创建数据库
我使用mysql,创建一个user表做例子
``` sql 创建user表代码
CREATE TABLE `user` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`username` VARCHAR(50) NOT NULL,
	`password` VARCHAR(50) NOT NULL,
	`sex` TINYINT(1) NOT NULL,
	`regDate` DATETIME NULL DEFAULT NULL,
	`pic` BLOB NULL,
	`enable` TINYINT(1) NOT NULL,
	PRIMARY KEY (`id`)
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB
AUTO_INCREMENT=5;
```


### 引入相应jar包
我使用gradle添加,依赖如下
``` gradle build.gradle
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.11'
    testCompile group: 'junit', name: 'junit', version: '4.12'
    compile group: 'org.apache.tomcat.embed', name: 'tomcat-embed-core', version: '9.0.12'
    compile group: 'mysql', name: 'mysql-connector-java', version: '8.0.12'
    compile group: 'com.mchange', name: 'c3p0', version: '0.9.5.2'
    compile group: 'org.springframework', name: 'spring-context', version: '5.0.9.RELEASE'
    compile group: 'org.springframework', name: 'spring-jdbc', version: '5.0.9.RELEASE'
    testCompile group: 'org.springframework', name: 'spring-test', version: '5.0.9.RELEASE'
}
```

### 配置applicationContext.xml文件
配置applicationContext.xml文件,配置好c3p0的dataSource,然后再配置jdbcTemplate.到时候要用jdbcTemplate直接找Spring要就可以了,非常方便!
``` xml applicationContext.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd ">

    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/mvcdemo"/>
        <property name="user" value="root"/>
        <property name="password" value="root"/>

        <property name="acquireIncrement" value="3"/>
        <property name="minPoolSize" value="2"/>
        <property name="maxPoolSize" value="5"/>
        <property name="maxStatements" value="5"/>
        <property name="maxStatementsPerConnection" value="5"/>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <context:component-scan base-package="cn.nicenan.mvcdemo.dao.impl"/>
</beans>
```

### 设计User类(bean)
很简单,没啥好说的,遵循javabean规范就可以,字段名要跟数据库中对应表的字段名一致,到时候反射才能生效!(不一样的要在写sql的时候取别名)
``` java User
package cn.nicenan.mvcdemo.model;

import java.sql.Blob;
import java.sql.Timestamp;

public class User {
    private int id;
    private String username;
    private String password;

    public boolean getEnable() {
        return enable;
    }

    public void setEnable(boolean enable) {
        this.enable = enable;
    }

    private boolean sex;
    private boolean enable;

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", sex=" + sex +
                ", regDate=" + regDate +
                ", pic=" + pic +
                '}';
    }

    public boolean getSex() {
        return sex;
    }

    public void setSex(boolean sex) {
        this.sex = sex;
    }

    private Timestamp regDate;
    private Blob pic;

    public User() {
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Timestamp getRegDate() {
        return regDate;
    }

    public void setRegDate(Timestamp regDate) {
        this.regDate = regDate;
    }

    public Blob getPic() {
        return pic;
    }

    public void setPic(Blob pic) {
        this.pic = pic;
    }
}
```
### 设计BaseDao类
想实现一个BaseDao类,给其他Dao类继承,要实现通用,把BaseDao设计成泛型类,等子类继承的时候传入bean的类型,BaseDao类就能通过反射方法,给所有bean设置属性.
首先难点是获取T.class,百度后发现这个方法:
``` java
private Class<T> entityClass = (Class<T>) ((ParameterizedType) getClass().getGenericSuperclass()).getActualTypeArguments()[0];

```
我们设计三个通用的方法:
``` java 
    //进行增删改操作
    public int iud(String sql, Object... args) {
        int rows = getJdbcTemplate().update(sql, args);
        return rows;
    }
    
    //出一个对象
    public T get(String sql, Object... args) {
        T result = getJdbcTemplate().queryForObject(sql, new UserRowMapper(), args);
        return result;
    }

    //取出很多对象
    public List<T> getlist(String sql, Object... args) {
        return getJdbcTemplate().query(sql, args, new UserRowMapper());
    }
```
增删改没啥说的,就是调用update方法,传入sql语句参数用?代替,在args传入参数.
JdbcTemplate查询一个(queryForObject)和查询多个对象(query)参数完全一致,主要就是在UserRowMapper这个类的设计,这个类要RowMapper<T>接口,这个接口只有一个mapRow方法,在里面我们可以用反射设计一个通用的方法!
``` java 
private class UserRowMapper implements RowMapper<T> {

        @Override
        public T mapRow(ResultSet rs, int rowNum) throws SQLException {
            try {
                //通过前面获取的T.class 实例化一个对象
                T entity = entityClass.newInstance();
                //取出数据库元数据
                ResultSetMetaData rsmd = rs.getMetaData();
                int count = rsmd.getColumnCount();
                //取出元数据的字段名,根据字段名反射进T
                for (int i = 0; i < count; i++) {
                    String label = rsmd.getColumnLabel(i + 1);
                    Field field = entityClass.getDeclaredField(label);
                    field.setAccessible(true);
                    field.set(entity, rs.getObject(label));
                }
                return entity;

            } catch (InstantiationException | NoSuchFieldException | IllegalAccessException e) {
                e.printStackTrace();
            }
            return null;
        }
    }
```
这样BaseDao类就设计好了,等等创建UserDao的时候只要调用父类的方法就可以了,非常简单!

### UserDao和UserDaoImpl
先在UserDao里设计好接口,UserDaoImpl就继承BaseDao类实现UserDao接口里面的方法就可以了.继承BaseDao的时候泛型要填入User类,这样父类才知道要返回什么类型的数据
代码如下:
``` java UserDao
package cn.nicenan.mvcdemo.dao;

import cn.nicenan.mvcdemo.model.User;

import java.util.List;

public interface UserDao {
    int add(User user);

    int update(User user);

    User get(int id);

    User get(String username);

    List<User> query(int sex);
    
}
```

``` java UserDaoImpl
package cn.nicenan.mvcdemo.dao.impl;

import cn.nicenan.mvcdemo.dao.BaseDao;
import cn.nicenan.mvcdemo.dao.UserDao;
import cn.nicenan.mvcdemo.model.User;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public class UserDaoImpl extends BaseDao<User> implements UserDao {

    @Override
    public int add(User user) {
        return super.iud("INSERT INTO user (username,password,sex,regDate,enable)VALUES(?,?,?,?,?)",
                user.getUsername(), user.getPassword(), user.getSex(), user.getRegDate(), 1);
    }

    @Override
    public int update(User user) {

        return super.iud("UPDATE user SET username = ? , password = ? , sex = ?, regDate = ?, pic = ?, enable = ? where id = ?",
                user.getUsername(), user.getPassword(), user.getSex(), user.getRegDate(), user.getEnable(), user.getId());

    }

    @Override
    public User get(int id) {
        return super.get("SELECT * FROM user WHERE id=?", id);
    }

    @Override
    public User get(String username) {
        return super.get("SELECT * FROM user WHERE username=?", username);
    }

    @Override
    public List<User> query(int sex) {
        return super.getlist("SELECT * FROM user WHERE sex=?", sex);
    }

}
```
### 编写测试类
``` java test
import cn.nicenan.mvcdemo.dao.UserDao;
import cn.nicenan.mvcdemo.model.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;
import java.sql.Timestamp;
import java.util.Date;
import java.util.List;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class daoTest {
    @Resource(name = "userDaoImpl")
    UserDao userDao;

    @Test
    public void ceshi() {
        User user = new User();
        user.setUsername("asd");
        user.setPassword("qweas");
        user.setSex(false);
        user.setRegDate(new Timestamp(new Date().getTime()));
        int rows = userDao.add(user);
        System.out.println(rows);
        System.out.println("-------------------------------------------");
        List<User> query = userDao.query(1);
        System.out.println(query);

    }

}
```
输出
```
1
-------------------------------------------
[User{id=1, username='w', password='q', sex=true, regDate=2018-09-13 20:55:53.0, pic=null},
User{id=2, username='q', password='s', sex=true, regDate=2018-09-14 13:18:08.0, pic=null}]
```
成功!

这样设计其他dao层的时候,直接继承BaseDao,就会变得非常轻松了!

