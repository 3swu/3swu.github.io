---
author: wulei
comments: true
date: 2017-08-04 15:48:24+00:00
link: http://wulei.kim/2017/08/04/servlet-simpleregister/
slug: servlet-simpleregister
title: Servlet实例：简单的注册功能
wordpress_id: 149
categories:
- Java
- PHP
- Servlet
tags:
- 注册
---

继续上一篇博文。在上一篇中，实现了一个简单的登录功能。光有登录显然不够，而且在上一篇文章中可以看到，在login.html页面中留了一个register链接。在数据库的建立过程中，也加入了registertime字段。显然现在到了必须要实现注册功能的时候了:)

关于注册模块实现的思路：很简单，判断输入的用户名和密码是否合法，如果合法则在数据表中添加一条新的记录，否则不会添加进数据表。关于数据的合法性，一是数据格式的合法，包括用户名的密码不能为空，至少多少位，由什么开头，不能包含什么字符等，本文为了简化功能，只简单的判断用户名的密码是否为空，而且选择了一个更愚蠢的方法：用Servlet进行判断。显然在实际的工程中，数据的合法性都是由JavaScript脚本来判断。还有一种数据的合法性，就是不能添加数据表中已有的用户。


## 创建表单


建立WebContent/register.html

    
    <!DOCTYPE html>
    <html>
    <head>
    	<title>register</title>
    </head>
    <body bgcolor="white" text="black" link="blue" vlink="purple" alink="red">
    <h1 align="center">register</h1>
    <form name="form2" action="register" method="post">
    <p align="center">username:<input type="text" name="username" /></p>
    <p align="center">password:<input type="password" name="password" /></p>
    <p align="center"><input type="submit" value="submit" /><input type="reset" value="reset" /></p>
    <h3 align="center"><a href="login.html">login</a></h3>
    </form>
    </body>
    </html>


保持了和login.html一致的风格。同样使用POST方法。


## 创建Servlet


在工程中新建一个Servlet，同样放在com.wulei.RegisterLogin包中，命名为register。

    
    package com.wulei.RegisterLogin;
    
    import java.io.IOException;
    import javax.servlet.ServletException;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    import java.sql.*;
    import java.text.SimpleDateFormat;
    import java.io.*;
    import java.util.Date;
    
    public class register extends HttpServlet {
    	private static final long serialVersionUID = 1L;
    	private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    	private static final String DB_URL = "jdbc:mysql://localhost/wulei";
    	private static final String USER = "root";
    	private static final String PASS = "******";
        public register() {
            super();
        }
    
    	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    		Connection conn = null;
    		Statement stmt = null;
    		response.setContentType("text/html");
    		
    		String userName = ""+request.getParameter("username");
    		String userPass = ""+request.getParameter("password");
    		
    		PrintWriter out = response.getWriter();
    		if(!(userName.equals("") || userPass.equals(""))){
    			try{
    				Class.forName(JDBC_DRIVER);
    				conn = DriverManager.getConnection(DB_URL, USER, PASS);
    				stmt = conn.createStatement();
    				
    				String sql = "select * from user where username='" + userName + "'";
    				ResultSet st = stmt.executeQuery(sql);
    				if(!st.next()){
    					sql = "insert into user (username,password,registertime) values ('" +
    							userName + "','" + userPass + "','" + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()) +
    							"')";
    					stmt.execute(sql);
    					out.print("<h1>register success.</h1>");
    				}else{
    					out.println("ERROR: Username '"+st.getString("username")+"' is existed.");
    				}
    			}catch(Exception ex){
    				ex.printStackTrace();
    			}
    		}else{
    			out.println("ERROR: Username or Password can not empty.");
    		}
    	}
    
    	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
    		doGet(request, response);
    	}
    
    }


处理过程：
1.先判断用户名和密码是否同时不为空，如果是，则往下进行。

2.查询数据表中是否已经存在该用户名，如果不存在，则继续进行。

3.建立SQL语句并执行。其中，关于registertime字段的处理，需要用的Java日期的格式化。首先import进Java的日期类java.util.Date，还有字符串格式化类java.text.SimpleDateFormat。使用如下语句即可得到格式化后的当前时间：

`new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date())`

首先创建一个指定格式的格式化类实例，调用其format()方法并传入一个Date对象。即可返回格式化后的时间字符串。

需要注意的是，SQL插入语句中并没有对id字段和statues字段进行处理，因为这两个字段都是设置了默认会添加的，id会自增保证索引的唯一。statues默认置为1。

在上一篇登录和这一篇的注册两个Servlet中，异常的处理都是使用Exception对象，这是非常偷懒的做法，正确的方法应该像《JDBC基础》那一片博文中给出的那样！


## 在web.xml中注册Servlet


注册方法和login一样，不再给出。


## 测试


首先测试两种不合法的情况：

![](http://wulei.kim/wp-content/uploads/2017/08/t1.png) ![](http://wulei.kim/wp-content/uploads/2017/08/t2.png)

接下来注册一个用户叫Elon：

![](http://wulei.kim/wp-content/uploads/2017/08/t3.png) ![](http://wulei.kim/wp-content/uploads/2017/08/t4.png)


## 没有对比，就没有伤害


这是我用PHP实现的同样的功能，表单代码几乎一样。

database.php

    
    <?php
    /**
     * Created by PhpStorm.
     * User: wulei
     * Date: 2017/5/20
     * Time: 16:01
     */
    
    class database{
    
        private static $_instance;
        private $_dbConfig=array(
            'dbhost'=>'127.0.0.1:3306',
            'dbuser'=>'root',
            'dbpass'=>'******',
        );
        private static $_connectRes;
    
        private function __construct()
        {
        }
    
        public static function getInstance()
        {
            if(!self::$_instance instanceof self)
                self::$_instance = new self();
            return self::$_instance;
        }
    
        public function connect()
        {
            self::$_connectRes = mysql_connect($this->_dbConfig['dbhost'],$this->_dbConfig['dbuser'],$this->_dbConfig['dbpass']);
            if(!self::$_connectRes)
            {
                die("database connect failed:".mysql_error());
            }
    
            mysql_select_db('WULEI');
    
            return self::$_connectRes;
        }
    }


login.php

    
    <?php
    require_once "database.php";
    
    $conn=database::getInstance()->connect();
    
    $username=$_POST['username'];
    $password=$_POST['password'];
    
    $sql="select * from user where username='$username' and password ='$password'";
    $retval=mysql_query($sql,$conn);
    $row=mysql_fetch_assoc($retval);
    if($row){
    	echo "login successful";
    }
    else{
    	echo $sql;
    }


register.php

    
    <?php
    require_once "database.php";
    $conn=database::getInstance()->connect();
    
    if($_POST['username'] && $_POST['password']){
    $username=$_POST['username'];
    $password=$_POST['password'];
    
    $sql="select * from user where username='$username'";
    $retval=mysql_query($sql,$conn);
    $row=mysql_fetch_assoc($retval);
    if($row){
    	echo "username".$username."existed,please use another username.";
    }else{
    	$sql="insert into user (username,password,registertime)value('$username','$password',NOW())";
    	$retval=mysql_query($sql,$conn);
    	$sql="select * from user where username='$username' and password='$password'";
    	$retval=mysql_query($sql,$conn);
    	$row=mysql_fetch_assoc($retval);
    	if($row){
    		echo "register successful";
    	}
    	else{
    		echo "register failed";
    	}
    
    }
    }
    ?>


其中数据库部分作为一个单独的文件。

可以看见，相同的功能，PHP实现更简单直观，所以为什么有人说PHP是世界上最好的语言:)


