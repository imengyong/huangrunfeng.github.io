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

* 配置仓库，在setting->GitHub Pages，

* github上搜gaohaoyang.github.io,把下下来，解压，把需要的文件覆盖倒自己git目录，并push
  ```sh
  git add .
  git commit -m "update blog style"
  git push
  ```

* 输入网址访问 https://xxx.github.io/, 查看效果

## 目录结构说明


## 参考说明：

* 浩阳神：https://gaohaoyang.github.io/

* https://blog.csdn.net/xudailong_blog/article/details/78762262

