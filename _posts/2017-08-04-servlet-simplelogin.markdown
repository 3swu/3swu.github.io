---
author: wulei
comments: true
date: 2017-08-04 08:04:06+00:00
link: http://wulei.kim/2017/08/04/servlet-simplelogin/
slug: servlet-simplelogin
title: Servlet实例：简单的登录功能
wordpress_id: 138
categories:
- Servlet
tags:
- Tomcat
- 登录
---

本文用Servlet实现简单的登录功能。登录功能的思路：创建一个login.html实现一个简单的登录表单，用户输入用户名和密码后表单向login Servlet发送POST的HTTP请求。由Servlet连接数据库并查找记录，如果存在用户名和密码匹配的记录，则登录成功。


## 在数据库中创建user数据表


![](http://wulei.kim/wp-content/uploads/2017/08/NC1AJ8KN814S659QY75.png)

首先在数据库中创建数据表user，由上图可见，包含主索引字段id，字段username，字段password，字段registertime和字段statues，statues的默认值为1。其中registertime和statues暂时不会用到。


## 新建Java Web工程


安装好Tomcat并且在Eclipse中配置好Tomcat后，新建一个Dynamic Web Project。可见工程的结构如下图：

![](http://wulei.kim/wp-content/uploads/2017/08/UJKGNWPH36TE0WHKBYAR.png)

编写的Servlet类文件放在src的包中，并且需要在WebContent/WEB-INF/web.xml中注册Servlet，否则Servlet容器（Tomcat）将无法启动Servlet，注册Servlet的方法将在下文说明。


## 创建表单


建立WebContent/login.html。

    
    <!DOCTYPE html>
    <html>
    <head>
    <title>User Login</title>
    </head>
    <body bgcolor="white" text="black" link="blue" vlink="purple" alink="red">
    <h1 align="center">user login</h1>
    <form name="form2" action="login" method="post">
    <p align="center">username:<input type="text" name="username" /></p>
    <p align="center">password:<input type="password" name="password" /></p>
    <p align="center"><input type="submit" value="submit" /><input type="reset" value="reset" /></p>
    <h3 align="center"><a href="">register</a></h3>
    </form>
    </body>
    </html>


action指定Servlet类，method指定使用POST方法。


## 创建Servlet


在工程中New一个Servlet叫做login，并创建一个包。

    
    package com.wulei.RegisterLogin;
    
    import java.io.IOException;
    import java.io.PrintWriter;
    
    import javax.servlet.ServletException;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    import java.sql.*;
    
    public class login extends HttpServlet {
    	private static final long serialVersionUID = 1L;
    	
    	private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    	private static final String DB_URL = "jdbc:mysql://localhost/wulei";
    	private static final String USER = "root";
    	private static final String PASS = "******";
    	
        public login() {
            super();
        }
    
    	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    		Connection conn = null;
    		Statement stmt = null;
    		
    		String userName = request.getParameter("username");
    		String userPass = request.getParameter("password");
    		
    		response.setContentType("text/html");
    		PrintWriter out = response.getWriter();
    		try{
    			Class.forName("com.mysql.jdbc.Driver");
    			conn = DriverManager.getConnection(DB_URL, USER, PASS);
    			stmt = conn.createStatement();
    			String sql = "select * from user where username='" + userName + "' and password='"+ userPass+"'";
    	//		System.out.println(sql);
    			ResultSet rs = stmt.executeQuery(sql);
    			if(rs.next()){
    	//			System.out.println("Not Empty");
    				out.println("<h1>Welcome user ["+rs.getString("username")+"]!");
    			}else{
    				out.println("Username or Password error!");
    			}
    			
    			rs.close();
    			stmt.close();
    			conn.close();
    		}catch(Exception ex){
    			ex.printStackTrace();
    		}
    	}
    
    	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    		doGet(request, response);
    	}
    
    }


在创建Servlet时，Eclipse会自动帮我们import必要的类，并且可以选择让Eclipse帮我们重写doGet()方法和doPost()方法。

需要注意的是，Tomcat运行JDBC程序不能用引用外部JRE的方法将MySQL驱动引入。需要将下载的JRE驱动拷贝至Tomcat安装目录的\lib目录下，否则会抛出java.lang.ClassNotFoundException异常。

Servlet的处理过程：

1.使用HttpServletRequest.getParameter()方法取得表单的参数。

    
    String userName = request.getParameter("username");
    String userPass = request.getParameter("password");


所以，这个Servlet的运行也可以通过访问这样的URL的方式：

URL:http://localhost:8080/RegisterLogin/login?username=*****&password=******

2.通过HttpServletResponse.getWriter()方法实例化一个PrintWriter流对象。作为输出到客户端的通道。

3.建立Servlet与数据库的连接，并且执行SQL查询语句。

4.最后判断ResultSet是否为空，通过Result.next()判断，如果为空，则数据库中不存在对应用户名和密码的记录，登陆失败，如果不为空，则登陆成功。


## 在web.xml中注册Servlet


必须在web.xml中注册Servlet才能使Web容器正确的找到Servlet。如图：

![](http://wulei.kim/wp-content/uploads/2017/08/ATHXANZGSLJUA14QNM7.png)

我们需要做的是在web-app标签下新建两个子标签servlet和servlet-mapping。其中servlet标签下包含servlet-name和servlet-class。servlet-mapping标签下包含servlet-name和url-pattern。值得注意的是，一个Servlet可以让多个url指向它。


## 测试


首先在数据库中新建一条记录用于测试 ，下面分别测试登录成功和失败：

![](http://wulei.kim/wp-content/uploads/2017/08/7CMNTZ5KO7EU_84G@NHN.png) ![](http://wulei.kim/wp-content/uploads/2017/08/3TG5AG2E8Q1@CNFJ.png)![](http://wulei.kim/wp-content/uploads/2017/08/WIQLZ7UDODZ9REFA2C41C.png)
