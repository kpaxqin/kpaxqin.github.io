## Flux: Actions and the Dispatcher


原文: [Flux: Actions and the Dispatcher][0]

Flux是Facebook用于构建javascript应用的架构。它基于单向数据流。利用Flux我们将所有东西都拆分成小的组件（widgets）然后组合成庞大的应用，而Flux成功应对了所有遇到的困难。我们认为这是一种非常好的组织代码的架构，因此非常高兴能够将他贡献给开源社区。Jing Chen在F8会议上[展示了Flux][flux_presented](youtube视频，墙内慎点)，从那以后我们了解到很多人对它非常感兴趣。同时，我们公布了[Flux概览](http://facebook.github.io/flux/docs/overview.html)与带有[入门指南](http://facebook.github.io/flux/docs/todo-list.html)的[TodoMVC示例](https://github.com/facebook/flux/tree/master/examples/flux-todomvc/)。

Flux与其说是一整套的框架，不如说是一种模式，在React之上你只需要写很少的代码就可以开始使用它。直到最近我们都还没有公布Flux的一个重要部分：dispatcher。但随着[Flux代码库](https://github.com/facebook/flux)和[Flux主页](http://facebook.github.io/flux/)的建立，现在我们已经将[dispatcher](http://facebook.github.io/flux/docs/dispatcher.html)开源并与生产环境版本保持同步

## Dispatcher在Flux数据流中的作用
dispatcher是单例的，在Flux应用中它被用作数据流的中央集线器。本质上它登记了一堆回调，并且按*store*注册回调的顺序执行。当有新的数据传入时，它向所有stores注册的回调传播此数据。调用回调的过程由dispatch方法触发，该方法接受一个数据负载对象(data payload object)作为唯一参数。

##Actions 与 ActionCreators
当有新数据进入系统时，无论是通过用户交互还是web api请求，该数据会被封装成一个action——一个包含了数据字段和特别声明的action类型的对象。通常我们会新建一类工具方法的集合叫做ActionCreators，用于创建action对象并传递给dispatcher。

不同的actions用类型区分。当所有stores接收到action时，通常他们通过识别类型来决定是否响应和如何响应。在Flux应用中，stores和views都是“自治”的，他们的行为不由外部决定。actions通过store自身定义并注册的回调进入store，而不是通过类似setter之类的方法调用。

stores的“自治”消除了许多MVC框架中常见的纠结现象。传统MVC框架model间的级联更新往往会导致不稳定的状态并且使得应用难以准确测试。而Flux应用则高度解耦，并且坚守了[迪米特法则](http://en.wikipedia.org/wiki/Law_of_Demeter)：系统中的不同对象之间相互了解得越少越好。这使得软件更加可维护，可扩展，和可测试，并且更利于新团队成员理解。

![flux-diagram][1]

##为什么需要Dispatcher

随着应用规模的增长，Store之间的相互依赖变得无法避免。Store A可能会需要 Store B先执行update，然后才能正确地update自己。Dispatcher可以让我们先行调用并完成Store B所注册的回调，再继续执行Store A的业务。要显式地声明这种依赖，Store需要向dispatcher表明“我要等Store B处理完了才能继续处理当前的action”。而dispatcher的waitFor方法则提供了这种功能。

dispatch方法对回调的遍历是简单、`同步`式的。当在执行callback过程中遇到waifFor方法时，对当前callback的调用就会中断，wairFor方法会根据声明的依赖重新确定遍历顺序`（注：原文字面意思如此，但从dispatcher的源代码看是检查依赖项是否已执行，若否，则立即去执行依赖项）`，当所有依赖都被执行后，原先中止的callback才会继续执行

更进一步地，waifFor方法可以在同一个store回调的不同action处理分支中使用。在某种情况下Store A可能依赖Store B，而在另一种情况下又依赖Store C。在action处理语句块中使用waitFor使我们能够更细粒度地控制依赖关系。

当然也会出现问题，比如循环依赖。A依赖B然后B又依赖A这样的情况，会导致无限循环。目前Flux实现的dispatcher在遇到这种情况时会抛出一个包含错误信息的Error来警告开发者，然后开发者可以选择新建一个Store来避免这种情况。

`注：从源代码看其实不用新建store，只要通过register新注册一个回调id即可，环检测是基于回调id的，Flux的官方demo中每个store有一个dispatcherToken来保存该store的回调id，事实上一个store持有多个回调id也是可行的`



  [0]: http://facebook.github.io/react/blog/2014/07/30/flux-actions-and-the-dispatcher.html

  [1]: https://raw.githubusercontent.com/facebook/flux/master/docs/img/flux-diagram-white-background.png
  
  [flux_presented]: http://youtu.be/nYkdrAPrdcw?t=10m20s
  
  [FRP]: https://www.google.co.jp/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0CCcQFjAA&url=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FFunctional_reactive_programming&ei=2nnPVISpOIGsmAXZoIHgCg&usg=AFQjCNFG3vX9EHPo9dnGH3hVO43fZQ4MEQ
  
  [DFP]: http://en.wikipedia.org/wiki/Dataflow_programming
  
  [FBP]: https://www.google.co.jp/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0CCQQFjAA&url=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FFlow-based_programming&ei=zHrPVMTzF4SNmwXJ1YGgBg&usg=AFQjCNH6O3pU4DGQIsymw59xW5t-mDacRw
