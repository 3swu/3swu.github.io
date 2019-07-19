---
author: wulei
comments: true
date: 2017-08-30 15:22:42+00:00
link: http://wulei.kim/2017/08/30/deploy_javaweb_project_to_server/
slug: deploy_javaweb_project_to_server
title: 在服务器上部署Java Web项目
wordpress_id: 187
categories:
- Java
tags:
- micblog
---

<blockquote>这是暑假最后一篇博客，开学之后我将根据时间安排尽量更新，估计频率不会太低。因为明天就要上火车，今天想了想，没事干不如将那个微博小玩具部署到服务器上。</blockquote>


本文不会记录太详细的部署过程，因为这只是简单记录一下过程，不是写教程。在服务器上部署项目，简单来说，就是搭建程序的运行环境，和平常的搭建个开发环境过程没有太大区别。



 	
  1. 在本地用ssh客户端连接到服务器，这是操作服务器的基础。

 	
  2. 在服务器上打开ftp服务。如果服务器没有安装ftp服务可以使用VSFTP。安装之后添加用户和默认目录。然后在本地使用ftp客户端测试连接是否正确。

 	
  3. 安装JDK。这是Java项目运行的基础，由于Oracle官网下载jdk的链接有问题，所以直接安装的openjdk，还不用手动解压安装。安装好了之后配置环境变量。

 	
  4. 安装MySQL，由于我的服务器之前搭建博客的原因已经装好了MySQL。正常安装之后建立用户，添加数据库。

 	
  5. 安装TomCat。一样，解压后添加环境变量，启动后在本地浏览器查看:http://服务器地址:8080,显示Tomcat欢迎页则安装成功。

 	
  6. 导入MySQL数据库，在本地数据库服务器使用MySQL命令可以导出整个数据库，用FTP上传到服务器后导入。即可将整个数据库导入。

 	
  7. 在本地Eclipse中将项目导出为.war格式的Web项目文件。FTP到服务器上后移动到TomCat的/webapps文件夹下，TomCat重启就会自动解压。

 	
  8. 在本地浏览器测试可以发现，项目的登陆和注册页正常显示，但是不能正常的登录和注册，抛出了对象空引用异常，所有涉及到的数据库操作都缺少Connection对象，说明没有正常连接到服务器。突然想起来，TomCat没有添加JDBC连接MySQL的驱动，然后用FTP把JRE驱动包上传，移动到TomCat的/lib文件夹下，重启服务，一切正常。


**项目地址：[http://wulei.kim:8080/micblog/](http://wulei.kim:8080/micblog/)**
