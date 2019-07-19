---
author: wulei
comments: true
date: 2017-08-25 16:27:06+00:00
link: http://wulei.kim/2017/08/26/the_essence_of_the_collection_generics/
slug: the_essence_of_the_collection_generics
title: 浅谈Java集合泛型的本质
wordpress_id: 183
categories:
- Java
tags:
- 反射
- 容器
- 泛型
---

众所周知，Java的集合类型具有泛型的特点，使我们可以在使用集合类时更轻松，更安全。集合类的声明有下面两种方法：

    
    ArrayList list = new ArrayList();
    
    ArrayList<String> list1 = new ArrayList<String>();


上面这种方法没有使用泛型，使得list可以存放任意类型的对象。下面这种方法使用泛型，使得list1只能存放String类型的对象。现在执行下列操作：

    
    System.out.println(list.getClass() == list1.getClass());


将这两个集合对象的类类型进行比较，可以发现，结果为true。也就是说，list和list1是同一种类型的对象。

那么给出结论：**编译之后集合的泛型是去泛型化的，Java中集合的泛型，是防止错误输入的，只在编译阶段有效，绕过编译就无效了。**

怎么验证这个结论？

既然结论中给出集合的泛型绕过编译就无效了，那么可以使用方法的反射来绕过编译，因为反射操作是在编译之后进行的。

    
    		try {
    			Method c = list1.getClass().getMethod("add",Object.class);
    			m.invoke(list1,1);
    		}catch(Exception ex) {
    			ex.printStackTrace();
    		}


反射list1对象的add()方法，可以发现，这样的操作可以进行，int型数据被成功添加进ArrayList<String>对象中，结论得证。
