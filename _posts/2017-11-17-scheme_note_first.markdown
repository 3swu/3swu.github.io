---
author: wulei
comments: true
date: 2017-11-17 15:31:04+00:00
link: http://wulei.kim/2017/11/17/scheme_note_first/
slug: scheme_note_first
title: Scheme笔记（一）
wordpress_id: 234
categories:
- Scheme
tags:
- Scheme
---

Example 1: rember.scm

    
    (define rember
      (lambda (a lat)
        (cond
          ((null? lat) (quote ()))
          (else (cond
                  ((eq? a (car lat)) (cdr lat))
                  (else
                   (cons (car lat)
                         (rember a (cdr lat)))))))))
    
    > (rember 'three '(one two three four))
    (one two four)


rember函数的作用：移除列表lat中的第一个a原子。



 	
  1. `((null? lat) (quote ()))`：当前的列表是否是空列表，如果是，那么这次函数执行的结果也为空列表（空列表出去原子a当然得空列表）。

 	
  2. 执行else后的语句。else也为语句，只不过它总是得到#t。

 	
  3. `((eq? a (car lat)) (cdr lat)`：当前列表的第一个元素是否等于a，如果相等，则这次函数执行的结果为(cdr lat)(该列表除去第一个原子剩下的列表)。

 	
  4. 上一个条件不成立，则执行下一条语句。下一条语句是`else`，`else`总是得出#t，所以接着执行`else`之后的语句。

 	
  5. `(cons (car lat)
                     (rember a (cdr lat)))`：递归调用rember函数，并且调用的参数变成了(cdr lat)，并且保存(car lat)的值与调用(rember a (cdr lat))的值进行cons，得到这次执行的列表。


Example 2: insertR.scm

    
    (define insertR
      (lambda (new old lat)
        (cond
          ((null? lat) (quote ()))
          ((eq? old (car lat)) (cons
                                (car lat) (cons new (cdr lat))))
          (else (cons (car lat)
                      (insertR new old (cdr lat)))))))
    
    > (insertR 'new 'two '(one two three two))
    (one two new three two)


insertR函数的作用：在lat列表中的第一个old后加入new原子。

与上例类似。

Summary：



 	
  * 在函数的开始总是先判断null?

 	
  * 使用cons构建列表

 	
  * 将典型原子和一般性递归的结果进行cons


