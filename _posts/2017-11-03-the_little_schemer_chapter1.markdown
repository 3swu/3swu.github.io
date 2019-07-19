---
author: wulei
comments: true
date: 2017-11-03 14:28:11+00:00
link: http://wulei.kim/2017/11/03/the_little_schemer_chapter1/
slug: the_little_schemer_chapter1
title: 《The Little Schemer》笔记之Scheme五法
wordpress_id: 228
categories:
- Scheme
tags:
- Scheme
---

### 为什么学习Scheme？


不同于常用的C++/Java等指令式语言，也不同于熟悉的面向对象的编程范式。Lisp是一个纯粹的函数式编程语言(FP)。虽然在工作生产中使用到函数式编程的机会少之又少，但是通过函数式编程能习得一种不同于以往的思维方式，使我们更能纯粹的理解计算的本质。只有学习过不同的思维方式，才能自由的思考。而Scheme作为Lisp的一门方言，比Common Lisp等方言更加简洁，所以学习Scheme是一个不二选择。

《计算机程序的构造和解释》(SICP)这本书使用Scheme作为讲解抽象系统设计的主要语言，非常希望能够拜读这本经典，所以学习Scheme还是很有必要。

除去上述Scheme的优点，只要一个理由就足够让我们去学习它了，那就是Scheme够酷！


### _The Little Schemer - Chapter 1_





 	
  * 原子：以字符，数字或特殊字符组成的字符(串)称为一个原子


Lisp或Scheme表示：`(quote atom)或'atom`



 	
  * 列表：用括号将原子或列表或者原子和列表包括起来成为列表


Lisp或Scheme表示：`(quote (atom))或'(atom)`



 	
  * 第一法：car法则：基本元件car仅定义为针对非空列表


car：列表中的第一个S-表达式

    
    (car '(((hotdogs))(and)(piclke) relish))
    
    => ((hotdogs))





 	
  * 第二法：cdr法则：基本元件cdr仅定义为针对非空列表。任意非空列表的cdr总是另一个列表


cdr：列表l中扣除(car l)中的部分

    
    (cdr
      (cdr
        '((b)(x y)((c)))))
    
    => (((c)))





 	
  * 第三法：cons法则：基本元件cons需要两个参数。第二个参数必须是一个列表。结果是一个列表


cons：cons元件添加任意的S-表达式到列表开头

    
    (cons
      '((help) this) '(is very ((hard) to learn)))
    
    => (((help) this) is very ((hard) to learn))





 	
  * 第四法：null?法则：基本元件null?仅定义为针对列表


null?：判断列表是否为空列表

    
    (null?
      (quote ()))
    
    => #t
    
    (null?
      (quote (a b c)))
    
    => #f





 	
  * 第五法：eq?法则：基本元件eq?需要两个参数。每个参数都必须是一个非数字的原子


eq?：判断两个非数字原子是否是同一个原子

    
    (define
      l (quote (beans beans we need jelly beans)))
    
    (eq?
      (car l)(car
               (cdr l)))
    
    => #t



