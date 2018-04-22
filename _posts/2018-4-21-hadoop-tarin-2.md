---
layout: post
title:  "Hadoop学习之项目练习.2 — SpringMVC整合"
categories: hadoop
tags:  spring hadoop hibernate
author: HuangRunfeng
---

* content
{:toc}

## 目的

hadoop学习之SpringMVC整合，记录springMVC整合过程




## 整改工程，使用Spring 全注解方式
创建RootConfig和MyBeans两个类，替换bean.xml配置方式

RootConfig配置注解扫描路径，作用同bean.xml中的`context:component-scan`字段
  <!-- 组间扫描 -->
  <context:component-scan base-package="cn.runfeng.dao.impl,cn.runfeng.service.impl"></context:component-scan>

’‘’
  package cn.runfeng.config;
  import org.springframework.context.annotation.ComponentScan;
  import org.springframework.stereotype.Component;
  @Component
  @ComponentScan(basePackages={"cn.runfeng.config","cn.runfeng.dao.impl","cn.runfeng.service.impl"})
  public class RootConfig {
  }
```

MyBeans获取数据源、sessionFactory和transactionManager，取代bean.xml中的响应属性配置

```
  package cn.runfeng.config;
  import java.beans.PropertyVetoException;
  import java.io.IOException;
  import java.util.Properties;
  import javax.sql.DataSource;
  import org.hibernate.SessionFactory;
  import org.springframework.context.annotation.Bean;
  import org.springframework.core.io.ClassPathResource;
  import org.springframework.orm.hibernate5.HibernateTransactionManager;
  import org.springframework.orm.hibernate5.LocalSessionFactoryBean;
  import org.springframework.stereotype.Component;
  import com.mchange.v2.c3p0.ComboPooledDataSource;

  @Component
  public class MyBeans {  
    @Bean("prop")
    public Properties getProp(){
      try {
        Properties prop = new Properties();
        ClassPathResource resource = new ClassPathResource("jdbc.properties");
        prop.load(resource.getInputStream());
        return prop;
      } catch (IOException e) {
        e.printStackTrace();
      }
      return null;
    }
  
    @Bean("dataSource")
    public DataSource getDataSource(Properties prop){
      ComboPooledDataSource ds = null;
      try {
        ds = new ComboPooledDataSource();
        ds.setDriverClass(prop.getProperty("jdbc.drivercalss"));
        ds.setJdbcUrl(prop.getProperty("jdbc.url"));
        ds.setUser(prop.getProperty("jdbc.username"));
        
        ds.setMaxPoolSize(Integer.parseInt(prop.getProperty("c3p0.pool.size.max")));
        ds.setMinPoolSize(Integer.parseInt(prop.getProperty("c3p0.pool.size.min")));
        ds.setInitialPoolSize(Integer.parseInt(prop.getProperty("c3p0.pool.size.init")));
        ds.setAcquireIncrement(Integer.parseInt(prop.getProperty("c3p0.pool.size.increment")));
        
      } catch (PropertyVetoException e) {
        e.printStackTrace();
      }
      return ds;
    }
  
    @Bean("sessionFactory")
    public LocalSessionFactoryBean getSessionFactory(DataSource ds,Properties prop){
      LocalSessionFactoryBean sf = new LocalSessionFactoryBean();
      sf.setDataSource(ds);
      sf.setHibernateProperties(prop);
      sf.setMappingDirectoryLocations(new ClassPathResource("cn/runfeng/model"));
      return sf;
    }
    
    @Bean("transactionManager")
    public HibernateTransactionManager getTransactionManager(SessionFactory sessionFactory){
      HibernateTransactionManager hbm = new HibernateTransactionManager();
      hbm.setSessionFactory(sessionFactory);
      return hbm;
    }
  }
```

## 创建WebConfig类设置web相关配置，实现视图解析器，处理前端请求，地址映射

```
  package cn.runfeng.web;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.ComponentScan;
  import org.springframework.stereotype.Component;
  import org.springframework.web.servlet.ViewResolver;
  import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
  import org.springframework.web.servlet.config.annotation.EnableWebMvc;
  import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
  import org.springframework.web.servlet.view.InternalResourceViewResolver;

  @Component
  @EnableWebMvc
  @ComponentScan(basePackages="cn.runfeng.web")
  public class WebConfig extends WebMvcConfigurerAdapter{
    /*
     * 视图解析器
     */
    @Bean
    public ViewResolver viewResolver(){
      InternalResourceViewResolver resolver = new InternalResourceViewResolver("/pages/", ".jsp");
      resolver.setExposeContextBeansAsAttributes(true);
      return resolver;
    }
    
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
      configurer.enable();
    }
  }
```

创建配置分发器类，继承`AbstractAnnotationConfigDispatcherServletInitializer`，初始化springMVC配置

```
  package cn.runfeng.web;
  import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

  import cn.runfeng.config.RootConfig;

  /**
   * 配置分发器初始化类
   * @author runfeng
   *
   */
  public class ConfDispatcherServerInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
      return new Class[]{RootConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
      return new Class[]{WebConfig.class};
    }

    @Override
    protected String[] getServletMappings() {
      return new String[]{"/"};
    }

  }
```

## 创建Controller，接受前端请求并处理，返回处理结果

```
  package cn.runfeng.web.controller;

  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RequestMethod;

  @Controller
  public class HomeController {
    
    @RequestMapping(value="/home",method=RequestMethod.GET)
    public String home(){
      System.out.println("i am comming");
      return "home";
    }
  }
```

## 创建jsp前端页面，接收controller的响应

```
  <%@ page language="java" contentType="text/html; charset=UTF-8"
      pageEncoding="UTF-8"%>
  <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
  <html>
  <head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Home</title>
  </head>
  <body>
    hello, spring MVC + spring + hibernate!
  </body>
  </html>
```
## 配置tomcat服务器，选中工程右键->Run On Server

## 地址栏输入：http://localhost:8080/cn.runfeng.HadoopTrain/home 测试成功

* 注意：工程名要正确，不然无法访问页面，可在工程->properties->web project setting中查看


## 参考说明：

* 学习过程中，参考诸多帖子教程等，在此感谢，若有不妥，请联系会及时处理~