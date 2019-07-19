---
author: wulei
comments: true
date: 2017-09-05 08:45:21+00:00
link: http://wulei.kim/2017/09/05/lambda_basic/
slug: lambda_basic
title: Lambda基础
wordpress_id: 190
categories:
- Java
tags:
- Lambda
---

Lambda是Java 8引入的一个新特性。使用Lambda，可以帮助我们写出简洁易懂的代码，有效提高代码可维护性。

什么是Lambda表达式？简单来说，使用Lambda表达式，通常在需要一个函数，但有希望写的更简洁或者不想费神去想一个方法名字的时候使用，也就是匿名函数。所以Lambda表达式就是：能嵌入到其他表达式中的匿名函数（闭包）。有几个重要的意义：



 	
  1. 可以在表达式中直接定义一个函数，而不需要将表达式和函数分开。

 	
  2. 引入了闭包。

 	
  3. 允许函数作为一个对象来进行传递。


下面是示例：

首先定义一个MyInterface接口，然后写一个MyClass类实现这个接口。

    
    interface MyInterface{
           void func();
    }
    
    class MyClass implements MyInterface{
           @Override
           public void func(){
                  System.out.println("MyClass's func()");
           }
    }


然后写一个静态方法调用接口

    
    public static void doWithMyInterface(MyInterface obj){
           obj.func();
    }


通常，上面的代码是这样被调用的

    
    MyClass obj = new MyClass();
    doWithMyInterface(obj);


即定义一个类来实现接口，创建这个类的对象，再传给调用的方法。想这样使用经典的面向对象编码，严格的分为第一步，第二步...，很规范，但是很麻烦，想要提高编程效率，可以使用Lambda表达式。

    
    MyInterface lambdaObj = ()->{
    	System.out.println("Lambda object's func()");
    	};
    doWithMyInterface(lambdaObj);


定义一个Lambda对象，将其引用保存到变量lambdaObj变量中，再把它传给调用的方法，节省了定义一个类的工作。

进一步简化：

    
    doWithMyInterface(()->{System.out.println("Lambda object's func()");});





 	
  * 一个Lambda表达式，可以看成是一小段代码，这段代码可以当作整体看待，可以传递，并且可在需要时随时执行它。

 	
  * Lambda表达式有一个关键点是“延迟执行（deferred execution）”。仅定义一个Lambda表达式对象，并不意味着它会马上执行，只有显式调用它的时候，才会执行。




### 函数式接口


要接收一个Lambda表达式对象，必须是接口类型，并且这种接口，必须是“函数式接口（functional interface）”。所谓函数式接口，就是只定义有一个抽象方法的接口。在Java 8中，使用@FunctionalInterface注解标识一个函数式接口。最主要的标准是只定义有一个抽象方法。在统计某接口中抽象方法的个数时，以下类型的方法是不计算的：



 	
  1. 默认方法

 	
  2. 静态方法

 	
  3. Object类定义的公有方法




### Lambda基本语法


格式如：(parameters) -> expression或(parameters) ->{ statements; } ，前半部分称为方法签名，后半部分称为方法实现。如果表达式包含多条语句，可以使用大括号包围。

Lambda表达式中的参数名字无关紧要，也无需指明参数类型，因为在接口定义时，编译器就知道了方法参数的类型，这是Lambda的“类型自动推断”特性。

注意：

    
    public class Test {
    	public static void main(String[] args){
    		String msg = "Hello";
    		Printer printer = msg -> System.out.println(msg);
    	}
    }
    
    @FunctionalInterface
    interface Printer{
    	void print(String msg);
    }
    
    


这一段代码无法通过编译，原因是：与函数不一样，Lambda表达式并不是定义了一个新的变量作用域，因此，上述代码在同一个变量作用域中重复定义了两个相同的变量名，所以无法通过编译。



 	
  * Lambda表达式可以访问自己定义的局部变量，也可以访问它所在的方法所定义的局部变量，也能访问类的实例变量。

 	
  * Lambda表达式中的代码可以访问外部变量，但要求这些变量是“effectively final”的，所谓“effectively final”，主要针对局部变量而言，它要不“使用final定义”，或虽然没有使用final定义，但只初始化了一次。




### Lambda示例：开启一个新的线程并运行方法


在《多线程基础》中介绍了在Java中多线程编程的最基础的东西。当时通过内部类实现了Runnbale接口。也可以使用匿名内部类：

    
    		new Thread(new Runnable(){
    			public void run(){
    				System.out.println("Thread 1");
    			}
    		}).start();
    		
    		System.out.println("Thread 0");


在Java 8之后，就可以用这样的做法：

    
    		new Thread(()->{
    			System.out.println("Thread 1");
    		}).start();
    		System.out.println("Thread 0");


因为在Java 8中，Runnable接口也是一个函数式接口。


### Lambda示例：给对象排序


在小项目micblog中，已经出现过对象的排序，当时是根据微博对象的发布时间排序。JDK的Collections类定义了一个sort方法，sort方法可以接受一个集合对象和一个Comparator<>对象作为实参，Comparator<>是一个泛型的函数式接口。这样可以对一个对象集合进行排序。假如需要对几个字符串集合进行排序，使用匿名内部类可以这样做：

    
    Collections.sort(collection, new Comparator<String>() {  
        @Override  
        public int compare(String s1, String s2) {  
            return (s1.compareTo(s2));  
        }  
    }); 


使用Lambda表达式：

    
    Collections.sort(collection, (String s1, String s2) -> (s1.compareTo(s2)));




### 方法引用


在上文给对象排序中，如果我们将compare方法封装成类中的一个独立的方法compareString，那么还存在这样的做法：

    
    Collections.sort(collection, (s1, s2) -> this.compareString(s1, s2));


还等价于：

    
    Collections.sort(collection, this::compareString);


这就是方法引用，实际是Lambda表达式的一种简写，方法引用同样支持静态方法引用。


### 默认方法



    
    interface Printer{
           void print();
           default void defaultMethod(){
                  System.out.println("Default Method of Printer");
           }
    }


如上述代码，在定义接口时定义一个默认方法，意味着：只要实现了这个接口的类，都自动拥有这个方法，并且这个方法已经写好，定义新类时无需再实现一遍。

同样的，在接口中可以定义默认静态方法。
