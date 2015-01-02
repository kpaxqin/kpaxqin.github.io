---
layout: post
title:  "setTimeout/setInterval在Chrome/FF非激活标签页下节流的问题"
date:   2014-09-18 14:12:12
categories: bugs&issues
---

#小记Javascript中的AOP
对于AOP（Aspect Oriented Programming）熟悉后端语言特别是JAVA的程序员肯定不陌生，作为Spring等框架的热门特性，其在日志处理、性能监测等方面的应用非常广泛。
[WIKI](http://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E4%BE%A7%E9%9D%A2%E7%9A%84%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1#.E5.9F.BA.E6.9C.AC.E6.A6.82.E5.BF.B5)中对AOP的概念解释如下
>从主关注点中分离出横切关注点是面向侧面的程序设计的核心概念。分离关注点使得解决特定领域问题的代码从业务逻辑中独立出来，业务逻辑的代码中不再含有针对特定领域问题代码的调用，业务逻辑同特定领域问题的关系通过侧面来封装、维护，这样原本分散在在整个应用程序中的变动就可以很好的管理起来。


那么在Javascript中的AOP又是什么样的，使用它有哪些需要注意的地方呢？

前端时间正好看到腾讯[AlloyTeam](http://www.alloyteam.com/)的一篇博客: [用AOP改善javascript代码]（aop_improved_js）

在受到启发的同时，正好最近手头有一些需求可以拿来做实践，也积累了一些个人的感受，在此一并整理

相对于Java繁杂的实现，javascript的实现要`简单`得多，在[用AOP改善javascript代码]（aop_improved_js）一文中实现如下：
{% highlight javascript %}
Function.prototype.before = function(func){
    var __self = this;
    return function(){
        if (func.apply(this, arguments) === false){
            return false;
        }
        return __self.apply(this, arguments);
    }
}

Function.prototype.after = function(func){
    var __self = this;
    return function(){
        var ret = __self.apply(this, arguments);

        if (ret === false){
            return false;
        }

        func.apply(this, arguments);

        return ret;
    }
}
{% endhighlight %}
不到30行代码，AOP就出来了，用法也非常简单，只要 fn1 = fn1.before(fn2);就可以动态地、干净地在fn1之前织入fn2的逻辑，fn1的代码不需要做任何的改动。甚至，它支持 fn1 = fn1.before(fn2).before(fn3)，这样的链式调用，想想真是有点小激动呢。

当然，这段代码仅仅是个示例，存在一些细节上的问题，比如入侵了Function的原型，没有对织入的切面函数做异常捕获等，在此不表。

相比之下，有些思路上的问题更让我在意：
1.before中织入的切面函数func可以阻断主函数(及其后的函数链)执行
2.阻断主函数执行的方式是return false;
3.after中织入的切面函数不具备阻断函数链的“权利”，与before明显不同

##是否允许切面函数阻断函数链？
要探讨这个问题，我们需要先探究“函数链式AOP”的应用场景，




[aop_improved_js]: http://www.alloyteam.com/2013/08/yong-aop-gai-shan-javascript-dai-ma/