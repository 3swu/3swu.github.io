---
author: wulei
comments: true
date: 2017-08-25 16:03:17+00:00
link: http://wulei.kim/2017/08/26/java-reflect/
slug: java-reflect
title: Java 反射机制
wordpress_id: 180
categories:
- Java
tags:
- 反射
---

<blockquote>_资料来源：[慕课网-反射](http://www.imooc.com/video/3725)_</blockquote>


反射是Java的高级特性之一，也是许多框架比如Spring的基础。反射，是对类的分析剖解，使程序可以在运行时自己去分析类。


## Class类


在面向对象的概念中，万物皆对象。在Java中，虽然Java是一个强面向对象的语言，已经把面向对象做到了极致，但还是有两个东西不是对象。一是静态的成员，而是普通的数据类型，比如int类型。好在Java中有包装类这一概念弥补了这一缺陷，int类型的包装类是Integer类。

那么类是否是对象呢？答案是肯定的：**类是对象，是java.lang.Class类的对象**。所以简单地说，Java中所有的类包括我们自己创造的类，都是Class类的实例对象。

Class类对象的表示？

创建一个演示类`class Foo{}`。在Class类的源码中可以看到，Class类的构造函数是私有的，也就是说，我们不能够通过调用构造函数的方式来创建一个Class类的实例对象。

这个实例对象有三种表示方式：



 	
  1. 

    
    Class c1 = Foo.class;


从这种表示方法可以看出：任何一个类都有一个隐含的静态数据成员class。

 	
  2. 

    
    Foo foo1 = new Foo();
    //foo1是Foo类的一个实例对象
    Class c2 = foo1.getClass();


第二种表达方式是：已经知道该类的对象，通过调用该对象的getClass()方法。

 	
  3. 

    
    Class c3 = Class.forName();


第三种表达方式，调用Class类的静态成员函数forName()来得到实例对象，forName()函数中需要传入类的全名，包括包名和类名。需要注意的是，这个函数需要放在try/catch语句块中，可能会抛出ClassNotFoundException异常。


在这里的c1, c2, c3是什么？他们都是Foo类的类类型（class type）。类类型指的是：任何类都是Class类的实例对象，那么这个实例对象就是这个类的类类型。

这里的c1 == c2 == c3，因为一个类的类类型是唯一的。而且我们完全可以通过类类型来实例化这个类，得到一个对象。

    
    Foo foo2 = (Foo)c1.getInstance();


通过c1对象的getInstance()方法来创建一个Foo对象，但是必须要强制造型。而且这个语句必须放在try/catch语句块中，可能抛出InstantiationException和IllegalAccessException异常。值得注意的是，只有当类有无参数的构造方法使，才能使用这个方法，因为这个方法会调用类的无参构造函数。


## 动态加载类


在上一部分中，Class类实例对象的第三种表示方法，也是动态加载类的方法。类加载根据加载类的时间段不同分为静态加载类和动态加载类。

静态加载类指的是在程序编译阶段就加载类，而动态加载类指的是在运行时才加载类。比如有几个功能类，我们在调用他的函数中必须为加载每个功能类new一个对象，而且只要有一个类出错或者没有实现，程序就不能完成编译，这是因为使用new关键字创建对象是静态加载类。通过动态加载类，就可以解决这问题。

在程序中通过`Class.forNamee("类的全名");`在运行时加载类，再通过类类型得到实例对象，那么问题来了，不同的类的类类型不一样，怎么在通过getInstance()方法创建对象时为每一个对象造型。解决方法就是，实现一个所有功能类的接口，让每一个类去实现这个接口。这样我们就可以造型为这个接口类。


## 获取方法信息


万物皆对象，同样的，对象的方法也是对象，是`java.lang.reflect.Method`类的对象。接下来，就用Class类和Method类的方法来获取一个方法的信息并打印出来：

    
    package reflectDemo;
    
    import java.lang.reflect.Method;
    
    public class ClassUtil {
    
    	public static void getMethodMessage(Object obj) {
    		Class c = obj.getClass();
    		System.out.println("类的名称：" + c.getName());
    		Method[] ms = c.getMethods();
    		for(Method m : ms) {
    			Class returnType = m.getReturnType();
    			System.out.print(returnType.getName() + " ");
    			System.out.print(m.getName() + "(");
    			Class[] paramTypes = m.getParameterTypes();
    			for(Class paramType : paramTypes) {
    				System.out.print(paramType.getName() + ",");
    			}
    			System.out.print(");\n");
    		}
    	
    	}
    }


这个类中有一个静态成员函数getMethodMessage(Object obj)用来打印一个对象的所有成员函数的信息。



 	
  1. 使用getClass()方法获得这个对象所属的类的类类型。

 	
  2. 使用Class类的getName()方法可以获得这个类的全名，如果要得到这个类简单的名字，不包括包名，则可以使用Class类的getSimpleName()方法。

 	
  3. 使用Class类的getMethods()方法可以得到这个类的所有public成员函数，包括从父类继承得到的。使用getDeclaredMethods()可以获得所有自己声明的成员函数，不管是任何访问权限，但是不包括父类的成员函数。

 	
  4. 使用Method类的getReturnType()方法得到这个方法的返回值类型，返回的是这个返回类型的类类型。

 	
  5. 使用Method类的getParameterTypes()方法获得这个方法的参数列表的类型的类类型。



    
    package reflectDemo;
    public class Test {
    	public static void main(String[] args) {
    		String s = "hello";
    		ClassUtil.getMethodMessage(s);
    	}
    }


使用这个类测试上面的类，传入一个String对象，可以得到String类的所有成员函数的信息，运行结果如下：

    
    类的名称：java.lang.String
    boolean equals(java.lang.Object,);
    java.lang.String toString();
    int hashCode();
    int compareTo(java.lang.String,);
    int compareTo(java.lang.Object,);
    int indexOf(java.lang.String,int,);
    int indexOf(java.lang.String,);
    int indexOf(int,int,);
    int indexOf(int,);
    java.lang.String valueOf(int,);
    java.lang.String valueOf(long,);
    java.lang.String valueOf(float,);
    java.lang.String valueOf(boolean,);
    java.lang.String valueOf([C,);
    java.lang.String valueOf([C,int,int,);
    java.lang.String valueOf(java.lang.Object,);
    java.lang.String valueOf(char,);
    java.lang.String valueOf(double,);
    char charAt(int,);
    int codePointAt(int,);
    int codePointBefore(int,);
    int codePointCount(int,int,);
    int compareToIgnoreCase(java.lang.String,);
    java.lang.String concat(java.lang.String,);
    boolean contains(java.lang.CharSequence,);
    boolean contentEquals(java.lang.CharSequence,);
    boolean contentEquals(java.lang.StringBuffer,);
    java.lang.String copyValueOf([C,);
    java.lang.String copyValueOf([C,int,int,);
    boolean endsWith(java.lang.String,);
    boolean equalsIgnoreCase(java.lang.String,);
    java.lang.String format(java.util.Locale,java.lang.String,[Ljava.lang.Object;,);
    java.lang.String format(java.lang.String,[Ljava.lang.Object;,);
    void getBytes(int,int,[B,int,);
    [B getBytes(java.nio.charset.Charset,);
    [B getBytes(java.lang.String,);
    [B getBytes();
    void getChars(int,int,[C,int,);
    java.lang.String intern();
    boolean isEmpty();
    java.lang.String join(java.lang.CharSequence,[Ljava.lang.CharSequence;,);
    java.lang.String join(java.lang.CharSequence,java.lang.Iterable,);
    int lastIndexOf(int,);
    int lastIndexOf(java.lang.String,);
    int lastIndexOf(java.lang.String,int,);
    int lastIndexOf(int,int,);
    int length();
    boolean matches(java.lang.String,);
    int offsetByCodePoints(int,int,);
    boolean regionMatches(int,java.lang.String,int,int,);
    boolean regionMatches(boolean,int,java.lang.String,int,int,);
    java.lang.String replace(char,char,);
    java.lang.String replace(java.lang.CharSequence,java.lang.CharSequence,);
    java.lang.String replaceAll(java.lang.String,java.lang.String,);
    java.lang.String replaceFirst(java.lang.String,java.lang.String,);
    [Ljava.lang.String; split(java.lang.String,);
    [Ljava.lang.String; split(java.lang.String,int,);
    boolean startsWith(java.lang.String,int,);
    boolean startsWith(java.lang.String,);
    java.lang.CharSequence subSequence(int,int,);
    java.lang.String substring(int,);
    java.lang.String substring(int,int,);
    [C toCharArray();
    java.lang.String toLowerCase(java.util.Locale,);
    java.lang.String toLowerCase();
    java.lang.String toUpperCase();
    java.lang.String toUpperCase(java.util.Locale,);
    java.lang.String trim();
    void wait();
    void wait(long,int,);
    void wait(long,);
    java.lang.Class getClass();
    void notify();
    void notifyAll();
    java.util.stream.IntStream chars();
    java.util.stream.IntStream codePoints();




## 获取成员变量和构造函数的信息


同样的，类的成员变量和构造函数也是对象，分别是java.lang.reflect.Field和java.lang.reflect.Constructor类的对象。

通过这两个类的操作，同样的可以获取到类的成员变量和构造函数的信息，接着在上面这个类中实现这两个方法 ，先看获取成员变量的函数：

    
           public static void getFieldMessage(Object obj) {
    		Class c = obj.getClass();
    		System.out.println("类的名称：" + c.getName());
    		Field[] f = c.getDeclaredFields();
    		for(Field field : f) {
    			String fieldType = field.getType().getName();
    			String fieldName = field.getName();
    			System.out.println(fieldType + " " + fieldName + ";");
    		}
    	}


首先import进java.lang.reflect.Field类,因为类中的成员变量大多数都是私有的，而getFields()方法只能获取public变量，所以这里建议使用getDeclaredFields()方法，这次测试一下Class类的成员变量，输出结果如下：

    
    类的名称：java.lang.Class
    int ANNOTATION;
    int ENUM;
    int SYNTHETIC;
    java.lang.reflect.Constructor cachedConstructor;
    java.lang.Class newInstanceCallerCache;
    java.lang.String name;
    java.security.ProtectionDomain allPermDomain;
    boolean useCaches;
    java.lang.ref.SoftReference reflectionData;
    int classRedefinedCount;
    sun.reflect.generics.repository.ClassRepository genericInfo;
    long serialVersionUID;
    [Ljava.io.ObjectStreamField; serialPersistentFields;
    sun.reflect.ReflectionFactory reflectionFactory;
    boolean initted;
    [Ljava.lang.Object; enumConstants;
    java.util.Map enumConstantDirectory;
    java.lang.Class$AnnotationData annotationData;
    sun.reflect.annotation.AnnotationType annotationType;
    java.lang.ClassValue$ClassValueMap classValueMap;


获取构造函数的信息：

    
    	public static void getConstructorMessage(Object obj) {
    		Class c = obj.getClass();
    		System.out.println("类的名称：" + c.getName());
    		Constructor[] cs = c.getDeclaredConstructors();
    		for(Constructor c1 : cs) {
    			System.out.print(c1.getName() + " ");
    			Class[] paramTypes = c1.getParameterTypes();
    			System.out.print("(");
    			for(Class paramType : paramTypes) {
    				System.out.print(paramType.getName() + ",");
    			}
    			System.out.println(")");
    		}
    	}


同样的，使用getConstructors()方法只能获得public的构造函数，所以使用getDeclaredConstructors()方法，因为类中的所有构造方法都必须在类中声明，所以这样能获得所有的构造方法。其余的和上文基本类似。测试String类的构造方法，输出如下：

    
    类的名称：java.lang.String
    java.lang.String ([B,int,int,)
    java.lang.String ([B,java.nio.charset.Charset,)
    java.lang.String ([B,java.lang.String,)
    java.lang.String ([B,int,int,java.nio.charset.Charset,)
    java.lang.String ([B,int,int,java.lang.String,)
    java.lang.String ([C,boolean,)
    java.lang.String (java.lang.StringBuilder,)
    java.lang.String (java.lang.StringBuffer,)
    java.lang.String ([B,)
    java.lang.String ([I,int,int,)
    java.lang.String ()
    java.lang.String ([C,)
    java.lang.String (java.lang.String,)
    java.lang.String ([C,int,int,)
    java.lang.String ([B,int,)
    java.lang.String ([B,int,int,int,)




## 方法反射的基本操作


在上文中获取方法信息的内容中已经涉及到了一点方法反射的基本操作。方法反射的基本思想，是首先获取某一个方法。如何获取某个特定的方法呢？方法的名称和方法的参数列表能够唯一的确定某个方法。第二就是方法反射的操作，也就是执行这个反射的方法，使用method.invoke(对象,参数列表)方法即可操作。我认为这里与普通的使用对象来调用方法不同，可以理解为“使用特定的方法来调用某个对象”。

    
    package reflectDemo;
    
    import java.lang.reflect.Method;
    
    public class methodReflect {
    
    	public static void main(String[] args) {
    		A a1 = new A();
    		Class c = a1.getClass();
    		try {
    			Method m = c.getMethod("print", new Class[] {int.class,int.class});
    			Object o = m.invoke(a1, new Object[] {10,20});
    		}catch(Exception ex) {
    			ex.printStackTrace();
    		}
    	}
    }
    
    class A{
    	public void print(int a,int b) {
    		System.out.println(a + b);
    	}
    }


要获取到A类的print方法，首先要得到A类的类类型。然后使用类类型的getMethod()方法获得某个特定的方法。这个方法的第一个参数传入要获得的方法的名字，第二个参数传入这个方法的参数列表，类型为一个可变参数类型，所以除了上面的方法，可以直接传入可变个数的参数。

然后使用Method对象的invoke()方法，第一个传入指定的对象，第二个也为可变参数列表，方法与上文相同。这个invoke()方法会返回这个反射方法执行后应该返回的参数，所以这样的调用方法与a1.print()方法的执行效果完全一致。当方法无返回值时，Object对象o为null。

总结下来，Java的反射操作都是建立在**Class，Field，Constructor，Method**这四个类的基础上。
