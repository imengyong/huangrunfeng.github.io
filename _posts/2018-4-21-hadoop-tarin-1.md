---
layout: post
title:  "Hadoop学习之项目练习.1 — Spring/hibernate整合"
categories: hadoop
tags:  spring hadoop hibernate
author: HuangRunfeng
---

* content
{:toc}

## 目的

hadoop学习之Spring/hibernate整合，记录spring与hibernate整合过程

## 基本接口实现
* 模型实现，对应数据库的表结构
```sh
package cn.runfeng.model;
public class User {
  private int id;
  private String name;
  private int age;
  
  public int getId() {
    return id;
  }
  public void setId(int id) {
    this.id = id;
  }
  public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  }
  public int getAge() {
    return age;
  }
  public void setAge(int age) {
    this.age = age;
  }
  
}
```
* DAO 数据访问层实现，用于与数据库的交互，对数据库进行增删改查等操作
```sh
package cn.runfeng.dao;
import java.util.List;
/**
 * BaseDao通用接口，定义了通用的数据库操作
 * @author runfeng
 */
public interface BaseDao<T> {
  //写操作
  public void saveEntity(T t);
  public void updateEntity(T t);
  public void deleteEntity(T t);
  public void excuteByHql(String hql, Object ...objects); //执行HQL语句
  
  //读操作
  public T getEntity(Integer id);
  public List<T> findByHql(String hql,Object ...objects);//执行HQL语句
}
```

```sh
package cn.runfeng.dao.impl;
import java.lang.reflect.ParameterizedType;
import java.util.List;
import javax.annotation.Resource;
import org.hibernate.SessionFactory;
import org.hibernate.query.Query;
import cn.runfeng.dao.BaseDao;

/**
 * 实现BaseDao通用接口，用于继承
 * @author runfeng

 */
public abstract class BaseDaoImpl<T> implements BaseDao<T> {
  @Resource
  private SessionFactory sf;
  
  private Class clazz;
  public BaseDaoImpl() { //利用构造函数获取具体实例的对象类型
    ParameterizedType type = (ParameterizedType) this.getClass().getGenericSuperclass();//获得是子类的class
    clazz = (Class) type.getActualTypeArguments()[0];
  }
  
  @Override
  public void saveEntity(T t) {
    sf.getCurrentSession().save(t);
  }

  @Override
  public void updateEntity(T t) {
    sf.getCurrentSession().update(t);
  }

  @Override
  public void deleteEntity(T t) {
    sf.getCurrentSession().delete(t);
  }

  @Override
  public void excuteByHql(String hql, Object... objects) {
    Query q = sf.getCurrentSession().createQuery(hql);
    for (int i = 0; i < objects.length; i++) {
      q.setParameter(i, objects[i]);
    }
    q.executeUpdate();
    
  }

  @Override
  public T getEntity(Integer id) {
    return (T) sf.getCurrentSession().get(clazz, id);
  }

  @Override
  public List<T> findByHql(String hql, Object... objects) {
    Query q = sf.getCurrentSession().createQuery(hql);
    for (int i = 0; i < objects.length; i++) {
      q.setParameter(i, objects[i]);
    }
    return q.list();
  }

}
```

```sh
package cn.runfeng.dao.impl;
import org.springframework.stereotype.Repository;
import cn.runfeng.model.User;
/**
 * userDao实现
 * @author runfeng
 */
public class UserDaoImpl extends BaseDaoImpl<User> {

}
```
* Service业务逻辑层代码实现
```sh
package cn.runfeng.service;
import java.util.List;
/**
 * 通用业务逻辑操作接口
 * @author runfeng
 * @param <T>
 */
public interface BaseService<T> {
  //写操作
  public void saveEntity(T t);
  public void updateEntity(T t);
  public void deleteEntity(T t);
  public void excuteByHql(String hql, Object ...objects); //执行HQL语句

  //读操作
  public T getEntity(Integer id);
  public List<T> findByHql(String hql,Object ...objects);//执行HQL语句
  public List<T> findAll();//查找所有实体
}
```

```sh
package cn.runfeng.service;
import cn.runfeng.model.User;
/**
 * userService接口，用于定义UserService的特殊操作
 * @author runfeng
 */
public interface UserService extends BaseService<User> {
}
```

```sh
package cn.runfeng.service.impl;
import java.lang.reflect.ParameterizedType;
import java.util.List;
import javax.annotation.Resource;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import cn.runfeng.dao.BaseDao;
import cn.runfeng.service.BaseService;

/**
 * 通用业务逻辑接口BaseService的实现，用于各业务Service继承
 * @author runfeng
 */@Transactional(isolation=Isolation.DEFAULT,propagation=Propagation.REQUIRED)
public abstract class BaseServiceImpl<T> implements BaseService<T> {

  private BaseDao<T> dao;
  private Class clazz;
  public BaseServiceImpl() {
    ParameterizedType para = (ParameterizedType) this.getClass().getGenericSuperclass();
    clazz = (Class) para.getActualTypeArguments()[0];
  }
  
  
  @Override
  public void saveEntity(T t) {
    dao.saveEntity(t);
  }

  @Override
  public void updateEntity(T t) {
    dao.updateEntity(t);
  }

  @Override
  public void deleteEntity(T t) {
    dao.deleteEntity(t);
  }

  @Override
  public void excuteByHql(String hql, Object... objects) {
    dao.excuteByHql(hql, objects);
  }

  
  @Override
  public T getEntity(Integer id) {
    return dao.getEntity(id);
  }
  
  @Override
  public List<T> findByHql(String hql,Object ...objects){
    return dao.findByHql(hql, objects);
  }

  @Override
  public List<T> findAll() {
    return findByHql("From " + clazz.getName());
  }

}

```

```sh
package cn.runfeng.service.impl;
import javax.annotation.Resource;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;
import cn.runfeng.dao.BaseDao;
import cn.runfeng.model.User;
import cn.runfeng.service.UserService;
/**
 * UserService实现
 * @author runfeng
 *
 */
@Repository("userService")
@Transactional(isolation=Isolation.DEFAULT,propagation=Propagation.REQUIRED)
public class UserServiceImpl extends BaseServiceImpl<User> implements UserService {
  
}

## 模型映射：
在model统计下，添加模型映射文件：User.hbm.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
  <class name="cn.runfeng.model.User" table="user">
    <id name="id" column="id" >
      <generator class="identity"></generator>
    </id>
    <property name="name" column="name"></property>
    <property name="age" column="age"></property>
  </class>
</hibernate-mapping>

## 测试数据源
* 在src目录下添加spring配置文件：`bean.xml`
```sh
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xmlns:mvc="http://www.springframework.org/schema/mvc"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
    http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">
  
  <!-- 分散配置 -->
  <context:property-placeholder location="jdbc.properties"/>
  <!-- 数据源 -->
  <bean id="datasource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.drivercalss}"></property>
    <property name="jdbcUrl" value="${jdbc.url}"></property>
    <property name="user" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>
    
    <property name="maxPoolSize" value="${c3p0.pool.size.max}"></property>
    <property name="minPoolSize" value="${c3p0.pool.size.min}"></property>
    <property name="initialPoolSize" value="${c3p0.pool.size.init}"></property>
    <property name="acquireIncrement" value="${c3p0.pool.size.increment}"></property>
  </bean>
</beans>
```
`jdbc.properties`
```sh
jdbc.drivercalss=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://192.168.1.151:3306/hadoop
jdbc.username=root
jdbc.password=root

c3p0.pool.size.max=10
c3p0.pool.size.min=2
c3p0.pool.size.init=3
c3p0.pool.size.increment=2
```

* 新建test源码包，编写单元测试代码
```sh
package cn.runfeng.test;
import java.sql.Connection;
import java.sql.SQLException;
import javax.sql.DataSource;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class TestDataSource {
  
  @Test
  public void getConn() throws SQLException {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
    DataSource ds = (DataSource) context.getBean("datasource");
    Connection conn = ds.getConnection();
    System.out.println(conn);
  }
} 
```
* 遇到的问题。由于mysql安装不在本机，在另一个虚拟机上，因此要开启mysql的远程访问。报错无法连接远程数据库
`Access denied for user 'root'@'192.168.1.4' (using password: YES)`
解决方法：(未彻底解决)，参考：https://www.cnblogs.com/zx3212/p/6204563.html
```sh 
  vi /etc/my.cnf
  [mysqld] 
  skip-grant-tables 
  重启mysql，登陆
  systemctl restart mariadb
```

## 测试Service代码

* 配置spring配置文件，用注解方式配置spring bean
bean.xml中开启组件扫描
```sh 
<!-- 组间扫描 -->
<context:component-scan base-package="cn.runfeng.dao.impl,cn.runfeng.service.impl"></context:component-scan>
```

类实现加上注解
```sh 
@Repository("userService")
public class UserServiceImpl extends BaseServiceImpl<User> implements UserService {…}

@Repository("userDao")
public class UserDaoImpl extends BaseDaoImpl<User> {…}
```

* 编写单元测试代码
```
package cn.runfeng.test;
import java.util.List;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import cn.runfeng.model.User;
import cn.runfeng.service.UserService;
public class TestUserService {
  @Test
  public void testSave() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
    UserService us =  (UserService) context.getBean("userService");
    User user = new User();
    user.setName("runfeng1");
    user.setAge(19);
    us.saveEntity(user);
  }
}
```
* 执行报错dao空指针异常，dao未被赋值，使用`@Resource`注解赋值。在BaseServiceImpl新增`setDao`方法，在UserServiceImpl中重写该方法
BaseServiceImpl.java
```
@Resource
public void setDao(BaseDao<T> dao) {
  this.dao = dao;
}
```

UserServiceImpl.java
```
@Override
@Resource(name="userDao")
public void setDao(BaseDao<User> dao) {
  super.setDao(dao);
}
```

* 运行报sessionFatory空指针异常，整合spring和hibernate，配置sessionFatory
在配置文件bean.xml中增加hibernate配置
```
<!-- 本地会话工厂，spring整合hibernate -->
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
  <!-- 配置数据源 -->
  <property name="dataSource" ref="datasource" />
  <!-- 配置hibernate属性 -->
  <property name="hibernateProperties">
    <props>
    <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
    <prop key="hibernate.show_sql">true</prop>
    </props>
  </property>
  <!-- 映射文件位置 -->
  <property name="mappingDirectoryLocations">
    <list>
      <value>classpath:cn/runfeng/model</value>
    </list>
  </property>
</bean>
```

* 运行单元测试再次报错：org.hibernate.HibernateException: Could not obtain transaction-synchronized Session for current thread。在`bean.xml`配置文件中配置事务管理
```
<!-- 注解驱动事务管理 -->
<tx:annotation-driven/>
<!-- 事务管理器 -->
<bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
  <property name="sessionFactory" ref="sessionFactory"></property>
</bean>
```

在业务实现类中，增加事务注解
```
@Transactional(isolation=Isolation.DEFAULT,propagation=Propagation.REQUIRED)
public class UserServiceImpl extends BaseServiceImpl<User> implements UserService {…}

@Transactional(isolation=Isolation.DEFAULT,propagation=Propagation.REQUIRED)
public abstract class BaseServiceImpl<T> implements BaseService<T> {…}
```

* 再次运行单元测试，再次报错，原因为创建表结构，执行sql在mysql中创建user表
```
CREATE TABLE IF NOT EXISTS `user`(
   `id` INT UNSIGNED AUTO_INCREMENT,
   `name` VARCHAR(100) NOT NULL,
   `age` INT UNSIGNED NOT NULL,
   PRIMARY KEY ( `id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

至此，spring和hibernate整合结束，单元测试通过


## 参考说明：

* 学习过程中，参考诸多帖子教程等，在此感谢，若有不妥，请联系会及时处理~