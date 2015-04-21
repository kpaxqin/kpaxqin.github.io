---
layout: post
title:  "setTimeout/setInterval在Chrome/FF非激活标签页下节流的问题"
date:   2014-09-18 14:12:12
categories: bugs&issues
---

先说问题表现：

定时间隔为100ms的倒计时功能，在chrome下，若切换到其它标签页浏览，再回到倒计时所在页面时，倒计时明显失准，可能切出浏览了1分钟，回头发现计时器只走了5秒。

写了段代码放到console里运行观察此现象，计时间隔100ms，每10次即1秒在console里做一次打印
{% highlight javascript %}
var i = 0;
setInterval(function(){
  i++;
  if (i%10 === 0){//每10次计时即1秒钟打印一次
    console.log(i)
  }
}, 100);
{% endhighlight %}
正常情况下应该是10，20，30……这样每秒钟打印一次，但在chrome中，当切换到其它标签页浏览一段时间再切回时，能明显发现计时器的`停滞`

经试验，当计时器间隔放宽到`1秒`时，`停滞`现象消失。IE下则完全没有停滞的问题

初步的猜测是浏览器为了优化性能而这么做的，放宽后台标签页的时钟精度应该能节省相当可观的cpu资源，特别对于我这种经常10+标签页同开的用户

放谷狗搜索之，在stackoverflow上发现了相关的[讨论][stackoverflow]
而这位`MARK RUSHAKOFF`的[论述][pivotallabs]则比较全面，在最后也列出了一些针对自动化测试的解决方案，不过对业务代码的参考价值似乎不大



[stackoverflow]: http://stackoverflow.com/questions/6032429/chrome-timeouts-interval-suspended-in-background-tabs
[pivotallabs]: http://pivotallabs.com/chrome-and-firefox-throttle-settimeout-setinterval-in-inactive-tabs/
