---
layout: post
title:  "Hadoop学习之项目练习.3 — WEB动静资源集群隔离"
categories: hadoop
tags:  nginx tomcat
author: HuangRunfeng
---

* content
{:toc}

## 目的

Hadoop学习之项目练习— WEB动静资源集群隔离





## 安装与配置nginx
* 依赖zlib/openssl/pcre等组件

  ```
  yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
  ```

* 下载pcre（https://ftp.pcre.org/pub/pcre/），解压进入目录pcre-8.42 ，安装

  ```
  $ ./configure —prefix=/usr/local
  $ make
  $ sudo make install
  $ pcre-config --version
  ```

* 下载nginx（http://nginx.org/en/download.html），解压进入目录nginx-1.14.0，安装

  ```
  $ ./configure —prefix=/usr/local
  $ make
  $ sudo make install
  $ ./usr/local/sbin/nginx -v
  $ ./usr/local/sbin/nginx -s reload            # 重新载入配置文件
  $ ./usr/local/sbin/nginx -s reopen            # 重启 Nginx
  $ ./usr/local/sbin/nginx -s stop              # 停止 Nginx
  ```

* 宿主机上配置nginx 
  
  ```
  $ vi /usr/local/config/nginx.conf
  upstream servers {
            server 192.168.1.100:80 weight=1;
            server 192.168.1.101:80 weight=4;
            server 192.168.1.102:80 weight=4;
            server 192.168.1.103:80 weight=4;
      }
  upstream tomcats {
            server 192.168.1.100:8080 weight=1;
            server 192.168.1.101:8080 weight=4;
            server 192.168.1.102:8080 weight=4;
            server 192.168.1.103:8080 weight=4;
      }
      server {
            listen       8080;
            server_name  localhost;
            #charset koi8-r;
            #access_log  logs/host.access.log  main;
            location ~ .*\.(html|css|js|png|jpg)$ {
                root   html;
               index  index.html index.htm;
                 add_header Kss-Upstream $upstream_addr; # 为了在浏览器查看server的IP
              proxy_pass http://servers;
                }
                location / {
                proxy_pass http://tomcats;
              } 
     }
     ```

## 安装配置tomcat
  解压即可，本例子中用的默认端口8080

## 配置集群
  按上述在hadoop集群上安装nginx和tomcat
  在hadoop100/hadoop101/hadoop102/hadoop103上安装了nginx和tomcat，tomcat采用默认配置，nginx配置如下：
  
  ```
          server {
            listen       80;
            server_name  192.168.1.100;

            #charset koi8-r;

            #access_log  logs/host.access.log  main;
            location / {
                    root   html;
                    index  index.html index.htm;
            }
          }
  ```

## 前端展示
  写几个动态和静态页面，以便测试(本例使用include静态包含)
  本例中iphone.html 如下：
  
  ```
  <script type="text/javascript">
  function getIP(){
    var host = location.host;
    //alert(host);
    document.getElementById("html_ip").innerHTML = host;
  }
  </script>
  <body onload="getIP()">
    <div class="container body-content mycontent">
    <div class="row" style="text-align: center;">
      <div class="col-sm-12">
        <img alt="xiaomi" src="../images/iphone8.jpg">
      </div>
      <p>
        当前html所在ip地址：<span id="html_ip"></span>
      </p>
    </div>
    </div>
  </body>
 ```

## 打包成分发到集群
  打包成`HadoopTrain.jar`，解压静态资源到`/usr/local/html/HadoopTrain`,把`HadoopTrain.jar`放在`tomcat/webapp/`下

## 启动集群
   
  ```
  xcall nginx
  xcall /home/apache-tomcat-7.0.82/bin/startup.sh
  ```

## 访问验证
    浏览器中输入：http://localhost:8080/HadoopTrain/pages/iphone.jsp 查看现象，如图可见动静态分离已经实现

    ![](./imgs/iphone.jsp.png "效果展示")

    ![](./imgs/iphone.htm.png "效果展示")




## 问题与方案

* 启动nginx时`bind() to 0.0.0.0:8080 failed (48: Address already in use)`

  本例中监听端口是8080，tomcat占用了8080端口，应关闭tomcat，再启动nginx

* 运行程序获取数据源时，报一系列的空指针异常 

  原因：jdbc.properties 没有被打war包进去，
  解决方法：在pom.xml中加resource入下配置

  ```
    <resources>
      <resource>  
                <directory>src</directory>  
                <includes>  
                    <include>*.properties</include>  
                    <include>*/*.xml</include>  
                </includes>  
                <filtering>false</filtering>  
            </resource>
    </resources>
  ``` 

* 静态资源没实现代理，资源总是来自总是localhost

  location配置错误，修改如下：
  
  `location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|js|css)$`

* 下面错误是由于把宿主机上的nginx.config,误放到了nginx集群上导致，当访问了特别大时也可能出现下面的问题，在此记录以备后面参考

  1、网页访问，500 Internal Server Error  —Too many open files

     参考：http://www.drupal001.com/2013/07/nginx-open-files-error/


  2、查看nginx日志发现 worker_connections not enough

    参考：http://www.jb51.net/article/113988.htm

  3、查看nginx日志发现 no live upstreams while connecting to upstream, client:

    该问题是配错，集群上的服务器的nginx.config 又配置了监听端口，分发请求到集群其他机器导致



## 参考说明

* 学习过程中，参考诸多帖子教程等，在此感谢，若有不妥，请联系会及时处理~