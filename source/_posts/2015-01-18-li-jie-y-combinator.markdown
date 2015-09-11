---
layout: post
title: "理解Y combinator"
date: 2015-01-18 12:36:11 +0800
comments: true
categories:
---
很多次看到Y combinator, 但是一直没有好好理解它。[The Little Schemer](http://www.amazon.com/The-Little-Schemer-4th-Edition/dp/0262560992)上也有一章讲Y combinator讲得很详细的。不过当时看了没有理解明白。真正理解它的方式还是自己来推导一遍最好。

## 为什么要有Y combinator
<!-- more -->
Y combinator提供了一种不用特殊手段就可以实现自引用(self-reference)的方式。像在Scheme中，定义全局函数也可以实现自引用, 比如下面的阶乘函数的实现：
{% codeblock lang:lisp %}
(define fact
  (lambda (n)
    (if (< n 2) 1
      (* n (fact (- n 1))))))
{% endcodeblock %}
在某种程度上([参考](http://www.dreamsongs.com/NewFiles/WhyOfY.pdf))，使用define定义是不那么好的，因为它使函数的定义依赖于全局变量空间(the global variable space)。而全局变量空间是容易被破坏或者修改的。
##如何推导出Y combinator
The Little Schemer的第9章对于推导Y combinator讲得很详细。不过既然自己试着推导了一遍，记录一下也挺好。上面说到Y的目的就是不要有define，那么一个没有define的阶乘函数就是这样一个:
{% codeblock lang:lisp %}
(lambda (n)
  (if (< n 2) 1
    (* n (? (- n 1)))))
{% endcodeblock %}
这个函数里的?是我们暂时不知道要放什么的地方。不过就暂时这样的话，也能计算阶乘1和2，比如下面的计算就是可以计算1的阶乘：
{% codeblock lang:lisp %}
((lambda (n)
  (if (< n 2) 1
    (* n (? (- n 1))))) 1)
{% endcodeblock %}

那么?里面到底是要放什么呢？要放的是一个真正的阶乘函数。如果有define定义，那真正的阶乘函数就是上面的fact。但是我们要去掉对define的依赖，就必须另找其它方法。
###一个神奇的函数
先把?里面的东西放一下，来看一个神奇的函数:
{% codeblock lang:lisp %}
((lambda (m)
   (m m))
 (lambda (l)
   (l l)))
{% endcodeblock %}
 把这个函数展开:
{% codeblock lang:lisp %}
((lambda (l)
   (l l))
 (lambda (l)
   (l l)))
{% endcodeblock %}
得到是自己！这称为不动点(这里我说得应该不严格)。不动点理论很有意思，在很多领域里都有应用。比如求2的平方根，你可以通过找函数f(x) = 0.5 * ( x + 2 / x)的不动点来实现。查看wiki的不动点理论，有更多惊喜等着你:[http://en.wikipedia.org/wiki/Fixed-point_theorem](http://en.wikipedia.org/wiki/Fixed-point_theorem)

##转化一下，看起来快解决了

看了前面讲的那个不动点函数，我们可以找到?了。在找?之前，我们可以把?提出来，可能帮助理解。下面的转化还是那个只能求1和2的阶乘函数：
{% codeblock lang:lisp %}
((lambda (m)
   (m ?))
 (lambda (l)
   (lambda (n)
     (if (< n 2) 1
       (* n (l (- n 1)))))))
{% endcodeblock %}
你也可以在上面加入一点功能，让它能求3的阶乘：
{% codeblock lang:lisp %}
((lambda (m)
   (m (m ?)))
 (lambda (l)
   (lambda (n)
     (if (< n 2) 1
       (* n (l (- n 1)))))))
{% endcodeblock %}
也可以再加入4的阶乘的求解：
{% codeblock lang:lisp %}
((lambda (m)
   (m (m (m ?))))
 (lambda (l)
   (lambda (n)
     (if (< n 2) 1
       (* n (l (- n 1)))))))
{% endcodeblock %}
然后是5的阶乘：
{% codeblock lang:lisp %}
((lambda (m)
   (m (m (m (m ?)))))
 (lambda (l)
   (lambda (n)
     (if (< n 2) 1
       (* n (l (- n 1)))))))
{% endcodeblock %}
你会发现，这样挺累的。。。另外，你也注意到了在上面的求3，4，5的阶乘函数里的?是什么其实不重要。那就把它换成m吧！如下：
{% codeblock lang:lisp %}
((lambda (m)
   (m m))
 (lambda (l)
   (lambda (n)
     (if (< n 2) 1
       (* n (l (- n 1)))))))
{% endcodeblock %}
但是这还是没达到我们的目的。上面的还是只能求1和2的阶乘。
##那个神奇的函数

你还记得那个神奇的函数不？
{% codeblock lang:lisp %}
((lambda (m)
   (m m))
 (lambda (l)
   (l l)))
{% endcodeblock %}
想一想，把这个想法应用到前面最后我们转化的函数上：
{% codeblock lang:lisp %}
((lambda (m)
   (m m))
 (lambda (l)
   (lambda (n)
     (if (< n 2) 1
       (* n ((l l) (- n 1)))))))
{% endcodeblock %}
你会发现这函数已经可以求任意数的阶乘了！这一步的跳跃有点大。你如果要真正理解它，需要自己好好想想。

##收工
上面最后我们已经实现了不用define进行自引用了。但是要看到Y combinator，需要整理一下上面的结果。首先(l l)是返回一个接收一个参数的函数，我们可以把它等价为([参考](http://www.amazon.com/The-Little-Schemer-4th-Edition/dp/0262560992)):
{% codeblock lang:lisp %}
(lambda (x)
  ((l l) x))
{% endcodeblock %}
那可以得到：
{% codeblock lang:lisp %}
((lambda (m)
   (m m))
 (lambda (l)
   ((lambda (f)
      (if (< n 2) 1
        (* n (f (- n 1)))))
    (lambda (x)
      ((l l) x)))))
{% endcodeblock %}
进一步抽像出:
{% codeblock lang:lisp %}
(lambda (f)
  (if (< n 2) 1
    (* n (f (- n 1)))))
{% endcodeblock %}
就可以得到:
{% codeblock lang:lisp %}
((lambda (fac)
   ((lambda (m)
      (m m))
    (lambda (l)
      (fac (lambda (x)
             ((l l) x))))))
 (lambda (f)
   (lambda (n)
     (if (< n 2) 1
       (* n (f (- n 1)))))))
{% endcodeblock %}
而上面的部分:
{% codeblock lang:lisp %}
((lambda (m)
   (m m))
 (lambda (l)
   (fac (lambda (x)
          ((l l) x)))))
{% endcodeblock %}
就是Y combinator，更规范一些的定义是:
{% codeblock lang:lisp %}
(define Y
  (lambda (fun)
    ((lambda (m) (m m))
     (lambda (m)
       (fun (lambda (x) ((m m) x)))))))
{% endcodeblock %}
这个函数Y接收一个参数fun,返回这个函数自身。举一个例子:
{% codeblock lang:lisp %}
((Y (lambda (len)
      (lambda (lst)
        (if (null? lst) 0
          (+ 1 (len (cdr lst))))))) '(a b c e))
{% endcodeblock %}
以上函数返回4.