---
layout: post
title:  "setTimeout/setInterval在Chrome/FF非激活标签页下节流的问题"
date:   2014-09-18 14:12:12
categories: 问题记录
---

先说问题表现：
定时间隔为100ms的倒计时功能，在chrome下，若切换到其它标签页浏览，再回到倒计时所在页面时，倒计时明显失准，可能切出浏览了1分钟，回头发现计时器只走了5秒。

写了段代码放到console里运行观察此现象，计时间隔100ms，每10次即1秒在console里做一次打印
{% highlight javascript %}
var i = 0;
setInterval(function(){
  i++; 
  if (i%10 === 0){//
    console.log(i)
  }
}, 100);
{% endhighlight %}
正常情况下应该是10，20，30……这样每秒钟打印一次，但在chrome中，当切换到其它标签页浏览一段时间再切回时，能明显发现计时器的【停滞】
经试验，当计时器间隔放宽到1秒时，卡壳现象消失。IE下则完全没有停滞的问题

初步的猜测，是浏览器为了优化性能而这么做的

放谷狗搜索之，在[stackoverflow][http://stackoverflow.com/questions/6032429/chrome-timeouts-interval-suspended-in-background-tabs]上发现了相关的讨论
而这位-MARK RUSHAKOFF的[论述][http://pivotallabs.com/chrome-and-firefox-throttle-settimeout-setinterval-in-inactive-tabs/]则比较全面，在最后也列出了一些解决的方案

