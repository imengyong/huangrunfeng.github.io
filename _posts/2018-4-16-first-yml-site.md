---
layout: post
title:  "第一篇博客 -- 搭建GitHub博客"
categories: GitHub
tags:  yml GitHub HTML
author: HuangRunfeng
---

* content
{:toc}

## 目的

记录完整搭建GitHub自定义博客，以及本次工程代码目录说明




## 操作记录

* 注册github，并登录，创建一个`{username}.github.io`的仓库
  ```sh
  cd github/
  echo "# xxx.github.io" >> README.md
  git init
  git add README.md
  git commit -m "first commit"
  git remote add origin git@github.com:xxx/xxx.github.io.git
  git push -u origin master
  ```

* 配置仓库，在setting->GitHub Pages，把master设为Source


* github上搜gaohaoyang.github.io,把下下来，解压，把需要的文件覆盖倒自己git目录，并push
  ```sh
  git add .
  git commit -m "update blog style"
  git push
  ```

* 输入网址访问 https://xxx.github.io/, 查看效果

## 主要目录说明
* 主要目录结构如下：

  `---xxx(仓库名)

    |-- _includes

    |-- _layout

    |-- _posts

    |-- _sass

    |-- css

    |-- js

    |-- node_modules

    |-- pages 

    |-- index.html `

* _includes中是些html文件，构成整体页面布局，包括header、commet、tag、footer等页面导航栏侧边栏等内容; _layout下的default.html等用静态包含include"拼接"起来整体页面布局

* pages 里面是些二级页面，如about、demo等

* css/js/_sass 是页面前端样式文件

* _posts 是博客所在目录，里面存放以时间日期命名的md文件，上面html会解析这些md文件，处理成页面显示的样式和内容


## 参考说明：

* 浩阳神：[https://gaohaoyang.github.io/](https://gaohaoyang.github.io/)

* [https://blog.csdn.net/xudailong_blog/article/details/78762262](https://blog.csdn.net/xudailong_blog/article/details/78762262)

