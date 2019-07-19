---
author: wulei
comments: true
date: 2019-01-03 22:28:00+00:00
link: http://wuleiaty.github.io/2019/01/03/higher-order-procedures/
slug: higher-order-procedures
title: Scheme中的高阶函数
tags:
- Scheme
- 高阶函数
---

### 过程作为参数

考虑数学中的序列求和问题，对于求和记号后的$f(x)$，如果对于每一个不同的函数，编写不同的求和过程，无疑是相当麻烦且不具有效率的方式。如计算给定范围内的整数的立方和：
``` scheme
(define (sum-cubes a b)
    (if (> a b)
        0
        (+ (cube a) (sum-cubes (+ a 1) b))))
```
这个求值过程中，可以抽象出每一项和函数表达式为$f(x)=x^3$，每一项增长的表达式为$a_{x+1} = a_x + 1$，那么可以按照这样的模式，将这两项翻译为形式参数：
``` scheme
(define (sum term a next b)
    (if (> a b)
        0
        (+ (term a) (sum term (next a) b))))
```
增加了term和next作为过程形式参数，那么只要给出具体的过程参数，就可以完成这个过程：
``` scheme
(define (inc n) (+ n 1))
```
比如可以使用这个过程计算1到10的立方和：
``` scheme
(sum cube 1 inc 10)
```

### 使用lambda构造过程

对于上面的过程，不仅定义计算过程，还单独定义了next参数过程，显得啰嗦没有意义，有时候只想表达过程，而不像表达名字的时候，就可以使用lambda来构造过程next：
``` scheme
(lambda (x) (+ x 1))
```
在解释起对lambda表达式求值时，求值的结果是一个过程，比如在Scheme环境中执行上面的表达式，返回结果应该如下：
``` scheme
> #<procedure>
```
那么对于上面的求立方和的过程，可以使用lambda改写成如下的形式：
``` scheme
(sum cube 1 (lambda (x) (+ x 1)) 10)
```
一般而言，使用lambda与使用define创建的过程完全一样，唯一的不同是使用lambda创建的过程没有一个名字与其关联，即一个匿名函数。也可以使用define将一个名字关联到一个lambda表达式：
``` scheme
(define inc (lambda (x) (+ x 1)))
```
事实上这与上面的inc过程的定义等价。

### 使用let创建局部变量
lambda的一个应用是创建局部变量，比如对于一个函数$f(x, y) = (x+1)^3 + (y+1)^3$，如果我们希望表述为这样的形式：   
$$
\begin{cases}
a=x+1\\
b=y+1\\
f(x,y) = a^3+b^3
\end{cases}$$   
除了x、y这两个约束变量，还希望创建a、b这两个中间变量，就可以利用局部过程来约束局部变量：
``` scheme
(define (f x y)
    ((lambda (a b) 
        (+ (cube a)
            (cube b)))
        (+ x 1)
        (+ y 1)))
```
这种方式使程序变得复杂且难读，使用let即可以创建局部变量：
``` scheme
(define (f x y)
    (let ((a (+ x 1))
          (b (+ y 1)))
        (+ (cube a)
           (cube b))))
```
let过程的第一个参数是一个名字-表达式对偶的表，在这个表中约束的局部变量作用于后面的body中。let表达式可被解释为上一种语法形式，所以let表达式只是作为lambda的语法糖。

#### 实例：使用区间折半寻找方程的零点

``` scheme
(define search
  (lambda (f neg-point pos-point)
                 (let ((midpoint (average neg-point pos-point)))
                   (if (close-enough? neg-point pos-point)
                       midpoint
                       (let ((test-value (f midpoint)))
                         (cond ((positive? test-value)
                                (search f neg-point midpoint))
                               ((negative? test-value)
                                (search f midpoint pos-point))
                               (else
                                midpoint)))))))

(define close-enough?
  (lambda (x y)
    (< (abs (- x y)) 0.001)))

(define average
  (lambda (x y)
    (/ (+ x y) 2)))

;user interface
(define half-interval-method
  (lambda (f a b)
    (let ((a-value (f a))
          (b-value (f b)))
      (cond ((and (negative? a-value) (positive? b-value))
             (search f a b))
            ((and (negative? b-value) (positive? a-value))
             (search f b a))
            (else
             (error "Values are not of opposite sign" a b))))))
```
search过程给出了区间折半求函数零点的方式，为了防止使用过程时使用了两个不正确的参数，即两个点的函数值同号（区间内没有解），创建了一个接口过程。比如求$π$的近似值，$π$正好是$sin(x) = 0$在2和4之间的根：
``` scheme
(half-interval-method sin 2.0 4.0)
```
### 过程作为返回值
在计算中生成新过程是应用中常见的场景，也是lambda最重要的应用，比如要求出$x$与$f(x)$的平均值，这是常见的平均阻尼过程，可以这样表述：
``` scheme
(define (average-damp f)
    (lambda (x) (average x (f x))))
```
这个过程产生一个新的过程，这个新的过程将参数f应用于自己，比如平方函数产生的过程应用与10：
``` scheme
((average-damp square) 10)
```