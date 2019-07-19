---
author: wulei
comments: true
date: 2017-08-16 13:52:26+00:00
link: http://wulei.kim/2017/08/16/project_day1/
slug: project_day1
title: micblog项目开发Day 1
wordpress_id: 165
categories:
- JSP
- Servlet
tags:
- micblog
---

从今天开始，我将会从零开发一个简陋的仿微博的项目。预计将能实现简单的发微博，关注等功能。**项目已经上传到GitHub，并且将同步更新。GitHub地址：[https://github.com/wuleiaty/micblog](https://github.com/wuleiaty/micblog)**

今天完成了数据库的设计，还有登录注册模块的开发。下面是各个包的简介：


## com.util


这个包里面目前只有一个类，是用于最底层的数据库的连接和SQL语句的执行，其中SQL语句执行将查找和增删改分为了两个方法，因为查找需要返回一个ResultSet对象。

    
    package com.util;
    
    import java.sql.*;
    
    public class DbConnect {
    	private Connection conn = null;
    	private Statement stmt = null;
    	private ResultSet rs = null;
    	
    	//连接数据库
    	public void getConnection() {
    		final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    		final String DB_URL = "jdbc:mysql://192.168.1.2/micblog";
    		final String USER = "root";
    		final String PASS = "wulei123";
    		
    		try {
    			Class.forName(JDBC_DRIVER);
    			conn = DriverManager.getConnection(DB_URL, USER, PASS);
    		}catch(Exception ex) {
    			ex.printStackTrace();
    		}
    	}
    	
    	//执行查询语句，传入SQL语句
    	public ResultSet executeQuery(String sql) {
    		getConnection();//获取数据库连接
    		
    		try {
    			stmt = conn.createStatement();
    			rs = stmt.executeQuery(sql);
    			return rs;
    		}catch(SQLException ex) {
    			ex.printStackTrace();
    			return null;
    		}
    	}
    	
    	//专门用于执行增删改操作
    	public void executeOther(String sql) {
    		getConnection();//获取数据库连接
    		try {
    			stmt = conn.createStatement();
    			stmt.execute(sql);
    		}catch(SQLException ex) {
    			ex.printStackTrace();
    		}
    	}
    	
    	public void closeConnection() {
    		if(conn != null) {
    			try {
    				conn.close();
    			}catch(SQLException ex) {
    				ex.printStackTrace();
    			}
    		}
    		
    		if(stmt != null) {
    			try {
    				stmt.close();
    			}catch(SQLException ex) {
    				ex.printStackTrace();
    			}
    		}
    		
    		if(rs != null) {
    			try {
    				rs.close();
    			}catch(SQLException ex) {
    				ex.printStackTrace();
    			}
    		}
    	}
    }




## bean


用于存放JavaBean的包，目前有三个bean类，user、item和comment，分别对应用户 ，微博和评论。虽然我觉得将评论合并在微博中比较合理，但是为了和数据表的设计相对应，选择了将他们分开。


## DAO


将用来存放DAO层类文件，目前完成了登录和注册两个模块。下面是登录loginDAO.java

    
    package com.DAO;
    
    import java.sql.ResultSet;
    import java.sql.SQLException;
    
    import com.bean.*;
    import com.util.DbConnect;
    
    public class loginDAO {
    
    	//登录验证
    	public int checkUser(User user) {
    		String sql = "select * from user where user_username='"
    				+ user.getUser_username() +"' and user_password='"
    				+ user.getUser_password() +"'";
    		//System.out.println("SQL: " + sql);//输出SQL语句，调试使用
    		DbConnect dbconn = new DbConnect();
    		ResultSet rs = dbconn.executeQuery(sql);
    		
    //		try {
    //			rs.next();
    //			//登录成功，返回用户ID
    //			return rs.getInt("user_id");
    //		}catch(SQLException ex) {
    //			ex.printStackTrace();
    //			//System.out.println("login error");
    //			return -1;
    //		}finally {
    //			dbconn.closeConnection();
    //		}
    //		
    		try {
    			if(rs.next())
    				return rs.getInt("user_id");
    			return -1;
    		}catch(SQLException ex) {
    			ex.printStackTrace();
    			return -1;
    		}finally {
    			dbconn.closeConnection();
    		}
    	}
    	
    	public String getNickname(User user) {
    		String sql = "select * from user where user_id='" + user.getUser_id() + "'";
    		//System.out.println("SQL: " + sql);
    		DbConnect dbconn = new DbConnect();
    		ResultSet rs = dbconn.executeQuery(sql);
    		
    		try {
    			rs.next();
    			return rs.getString("user_nickname");
    		}catch(SQLException ex) {
    			ex.printStackTrace();
    			return null;
    		}finally {
    			dbconn.closeConnection();
    		}
    	}
    }


signUpDAO.java

    
    package com.DAO;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    import java.text.SimpleDateFormat;
    import java.util.Date;
    
    import com.bean.User;
    import com.util.*;
    import com.DAO.loginDAO;
    
    public class signUpDAO {
    	
    	//检查昵称是否已经存在
    	public boolean checkNickname(User user) {
    		String sql = "select * from user where user_nickname='" + user.getUser_nickname() + "'";
    
    		DbConnect dbconn = new DbConnect();
    		ResultSet rs = dbconn.executeQuery(sql);
    //		try {
    //			System.out.println(rs.getString("user_nickname"));
    //			rs.next();
    //			//System.out.println("nickname existed");
    //		}catch(SQLException ex) {
    //			ex.printStackTrace();
    //			return true;
    //		}finally {
    //			dbconn.closeConnection();
    //		}
    //		return false;//昵称存在，返回false
    		try {
    			if(!rs.next())
    				return true;
    			return false;
    		}catch(SQLException ex) {
    			ex.printStackTrace();
    			return false;
    		}finally {
    			dbconn.closeConnection();
    		}
    	}
    	
    	public boolean insertUser(User user) {
    		String sql = "insert into user (user_username,user_nickname,user_password,user_regtime)"
    				+ "values ('" + user.getUser_username() + "','"
    				+ user.getUser_nickname() + "','"
    				+ user.getUser_password() + "','"
    				+ new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date())
    				+ "')";
    		
    		DbConnect dbconn = new DbConnect();
    		dbconn.executeOther(sql);
    		dbconn.closeConnection();
    		
    		//用loginDAO类中的方法来检查用户是否添加成功
    		loginDAO logindao = new loginDAO();
    		int checkUserFlag = logindao.checkUser(user);
    		if(checkUserFlag != -1) {
    			user.setUser_id(checkUserFlag);
    			return true;
    		}
    		else {
    			return false;
    		}
    	}
    }


在添加用户记录的insertUser方法中，使用了loginDAO类中的checkUser方法来检查记录是否添加成功，还有一个解决方案可以不这么麻烦，就是在util包中的数据库操作类中使executeOther方法返回一个Int型数值代表数据库中影响的记录数，然后插入操作完成之后判断影响的记录数是否为0，就可以判断插入是否成功。


## servlet


servlet包现在也存放了登录和注册两个Servlet。懒得贴代码了


## JSP


目前有几个测试用的JSP页面，都很简陋，遗憾的是。登陆成功的JSP页面没有使用JavaBean，因为今天遇到点问题，等以后把问题解决了再修复。
