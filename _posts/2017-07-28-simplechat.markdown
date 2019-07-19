---
author: wulei
comments: true
date: 2017-07-28 13:41:33+00:00
link: http://wulei.kim/2017/07/28/simplechat/
slug: simplechat
title: 简单的局域网聊天室程序
wordpress_id: 108
categories:
- Java
- 网络编程
tags:
- Socket
- 多线程
---

有了上一篇《多线程基础》，可以将《网络进程间通信》中的聊天软件进一步完善，实现一个聊天室程序。

客户端程序：

    
    <code class="">import java.io.*;
    import java.net.*;
    import java.util.Scanner;
    
    public class ChatClient{
    	private Socket chatSocket;
    	
    	public class ThreadJob implements Runnable{
    		public void run(){
    			while(true){
    				try{
    					BufferedReader reader = new BufferedReader(
    							new InputStreamReader(chatSocket.getInputStream()));
    					while(true)
    						System.out.println("Recive: "+reader.readLine());
    				}catch(IOException ex){
    					ex.printStackTrace();
    				}
    			}
    		}
    	}
    	
    	public void go(){
    		try{
    			chatSocket = new Socket("192.168.1.2",6666);
    			System.out.println("Server connected.");
    			PrintWriter writer = new PrintWriter(chatSocket.getOutputStream());
    			Thread t = new Thread(new ThreadJob());
    			t.start();		
    			while(true){
    				writer.println(getMsg());
    				writer.flush();
    			}
    		}catch(IOException ex){
    			ex.printStackTrace();
    		}
    	}
    	
    	public String getMsg(){
    		return new Scanner(System.in).nextLine();
    	}
    	
    	public static void main(String[] args) {
    		new ChatClient().go();
    	}
    }</code>


服务端程序：

    
    <code class="">import java.io.*;
    import java.net.*;
    import java.util.HashMap;
    
    public class ChatServer {
    	private ServerSocket serverSock;
    	HashMap<String,Socket> socketMap = new HashMap<String,Socket>();
    	HashMap<String,BufferedReader> readerMap = new HashMap<String,BufferedReader>();
    	HashMap<String,PrintWriter> writerMap = new HashMap<String,PrintWriter>();
    	
    	public class ThreadJob implements Runnable{
    		public void run(){
    			try{
    				BufferedReader reader = readerMap.get(Thread.currentThread().getName());
    			
    				while(true){
    					String msg = reader.readLine();
    					System.out.println(Thread.currentThread().getName()+": "+msg);
    					sendToEveryone(msg,Thread.currentThread().getName());
    				}
    			}catch(IOException ex){
    				ex.printStackTrace();
    			}
    		}
    		
    		public void sendToEveryone(String msg,String sendThreadName){
    			for(String threadName : writerMap.keySet()){
    				if(!threadName.equals(sendThreadName)){
    					PrintWriter writer = writerMap.get(threadName);
    					writer.println(msg);
    					writer.flush();
    				}
    			}
    		}
    	}
    	
    	public void go(){
    		try{
    			serverSock = new ServerSocket(6666);
    			int ThreadAmount = 0;
    			while(true){
    				String ThreadName = "Thread" + ThreadAmount;
    				Socket sock = serverSock.accept();
    				System.out.println(ThreadName+" connected");
    				socketMap.put(ThreadName, sock);
    				writerMap.put(ThreadName, new PrintWriter(sock.getOutputStream()));
    				
    				BufferedReader reader = new BufferedReader(
    						new InputStreamReader(
    								sock.getInputStream()));
    				readerMap.put(ThreadName, reader);
    				
    				Thread t = new Thread(new ThreadJob());
    				t.setName(ThreadName);
    				t.start();
    				ThreadAmount ++;
    			}
    		}catch(Exception ex){
    			ex.printStackTrace();
    		}
    	}
    
    	public static void main(String[] args) {
    		new ChatServer().go();
    	}
    }</code>


很遗憾的是，这两个程序忘记写注释了。需要费点功夫读。

在服务端程序中，使用三个HashMap容器存储Socket对象等，以供每个线程使用且线程使用时不必新建串流，减少系统开销。肯定有更好的方法来实现线程的共享，但由于我的水平有限，选择了容器的方法来实现。

这个程序理论上支持无数个客户端连接服务端来加入聊天室，只要服务器的内存足够。下面是执行的效果：

![](http://wulei.kim/wp-content/uploads/2017/07/client.png) ![](http://wulei.kim/wp-content/uploads/2017/07/server.png)

启动了三个客户端程序，上图是三个客户端程序的结果，下图是服务器的结果。
