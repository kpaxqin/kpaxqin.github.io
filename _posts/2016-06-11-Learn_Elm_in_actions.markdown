---
layout: post
title:  "Elm入门实践——基础篇"
description: "Learn you a elm for great good."
date:   2016-06-11 14:50:00 -0800
categories: elm functional-programming front-end
---

## 简介
[Elm](http://elm-lang.org/) 是一门专注于Web前端的纯函数式语言。你可能没听说过它，但一定听说过Redux，而Redux的核心reducer就是受到了Elm的启发。

随着整个React社区往函数式方向发展，Elm作为前端函数式编程的先驱和风向标，毫无疑问是值得去学习和借鉴的。

如果你打算开始函数式编程，与其阅读零碎的文章试图弄明白那些晦涩的Monad/Functor们，动手写点熟悉的东西也许是更好的方式。接下我会以常见的Counter/CounterList为例，一步步地带你了解如何使用Elm构建应用。

由于内容较多，计划分四篇，大致内容分布如下：

1. 基础篇：Elm介绍、基础。使用在线编辑器实现Counter
2. 类型篇：Elm的类型系统
3. 进阶篇：本地工程的搭建，在本地实现Counter List
4. 完结篇：处理副作用，Elm与Redux对比


### 下载和准备
>本文的内容都基于官网提供的[在线编辑器](http://elm-lang.org/try)，可以稍后再配置本地环境

你可以在[官网下载安装包](http://elm-lang.org/install)，作为前端开发者，从[NPM下载](https://www.npmjs.com/package/elm)也是很好的选择，个人推荐后者

在安装成功后，打开命令行输入elm，会看到版本和帮助信息。


### 有用的学习资料
官网提供了[文档](http://elm-lang.org/docs) 和大量的[examples](http://elm-lang.org/examples) ，然而个人一直不太喜欢Elm的一点就是官方文档，无论是组织的合理性还是完整性都有所欠缺，即使是像[Syntax](http://elm-lang.org/docs/syntax)这样务求全面的地方，也有很多遗漏的知识点，在无形中增加了初学者的学习成本。

本文接下来会尽量讲解涉及到的知识点，如果遇到困难，除了官网外，以下两个链接也是不错的补充：

* [Learn X in Y minutes](https://learnxinyminutes.com/docs/elm/)：可以看成是对官网Syntax 的补充，不仅覆盖了一些官网忽略的点，很多解释也更加详细
* [Elm for JS](https://github.com/elm-guides/elm-for-js)：针对Javascript开发者的常见疑点解答，学习过程中有理解不了的地方不妨看看。

## Hello world
按照套路，现在是Hello world时间，官网有[在线版](http://elm-lang.org/examples/hello-html)，代码如下：

```js
import Html exposing (text)

main =
  text "Hello, World!"
```

非常简单，却隐含了几个重要的知识点：

### 函数调用
`text "Hello, World"`是Elm中的函数调用，类似于JS中的`text("Hello world")`，它将一个字符串转换成Html文本。

在很多语言中，函数调用都是括号，参数用逗号分隔，比如`fn(arg1, arg2)`，Elm的函数调用符为空格，参数也使用空格分隔，这点初看起来别扭，实际上并不难适应。

>调用符和分隔参数都是空格，如何区分呢？
>
>答案是不需要区分，Elm所有函数都是自动柯里化的，对于柯里函数`fn(arg1, arg2)`和`fn(arg1)(arg2)`等价，使用空格作为调用符，即`(fn arg1) arg2`，注意这里的括号仅用来表示代码执行顺序，省略后即为`fn arg1 arg2`



### 模块引用
第一行代码的`import Html exposing (text)`是模块引用，和ES6中的`import {text} from 'Html'`非常相似，但有一点需要注意，它同时导入了`Html`**和**`text`，而非只有`text`，让我们验证一下，修改在线Hello world中的代码：

```js
import Html exposing (text)

main =
  Html.div [] [text "Hello, World!"]

```

我们可以在代码中使用`Html.div`，证明`Html`同样被导入了当前作用域。`Html.div`也是个[函数](http://package.elm-lang.org/packages/elm-lang/html/1.0.0/Html#div)，接收两个数组，前者为属性数组，后者则是子元素。这种创建元素的方式其实非常常见：[React.createElment](https://facebook.github.io/react/docs/top-level-api.html#react.createelement)和[hyperscript](https://github.com/dominictarr/hyperscript#h-tag-attrs-text-elements)都是这个套路。

>没用过`React.createElement`？JSX[帮你做了](https://facebook.github.io/react/docs/jsx-in-depth.html#the-transform)而已

由于Html包含了几乎所有浏览器标签的渲染函数，一个个写进exposing不免繁琐（想象下有多少原生标签）。让我们再做一点微小的工作，使用`exposing(..)`来让代码更加简洁。同时，我们尝试给div添加class属性

```js
import Html exposing (..)
import Html.Attributes exposing (..)

main =
  div [class "hello"] 
    [ span [] [text "Hello, World!"]
    ]
```
> 由于不够严谨，并不推荐在生产代码中使用`exposing(..)`

和渲染标签一样，在Elm中属性的创建也是由函数完成的，上例我们使用了`Html.Attributes`模块的`class`函数

## Counter
有了Hello world的经验，让我们再往前一步，创建一个在线版的Counter，这里是React做的效果展示：[https://jsfiddle.net/Kpaxqin/pu53jd89/2/](https://jsfiddle.net/Kpaxqin/pu53jd89/2/)

### 静态View和数据
上面我们使用了`Html.div`来渲染div，同理，我们可以使用`Html.button`来渲染按钮。稍微修改下刚才的代码即可：

```js
import Html exposing (..)

main =
  div []
  [button [] [text "-"]
  ,text (toString 1)
  ,button [] [text "+"]
  ]
```

现在div有三个子元素——两个button和一个数字，一个静态的Counter就这么构建出来了，非常简单。

抽象是程序员的基本素养，把数字`1`写死在视图里显然是很业余的表现。将渲染视图这个行为封装成函数更加合理：

```js
import Html exposing (..)

view model =
  div []
    [ button [] [text "-"]
    ,text (toString model)
    ,button [] [text "+"]
  ]
  
initModel = 3
  
main = view initModel
```

在这里我们创建了一个函数，第一行是 `函数名 + 参数`，和调用一样都使用空格分隔，等号后面的就是函数体，除非一个函数特别简单，多数时候我们倾向于将函数体换行写。

### Update
有了静态界面，接下来应该让它“动”起来，响应用户操作了。

首先，让我们定义两种操作：

```js
type Msg = Increment | Decrement
```

接下来，定义这两种操作如何改变数据：

```js
update msg model = 
  case msg of 
    Increment -> 
      model + 1
    Decrement ->
      model - 1
```
update函数中的msg是我们刚刚定义的Msg类型的消息，model则是当前数据的值，如果你了解Redux的话一定会想：这不就是Reducer的`(action, state)=> nextState`吗？确实如此，Reducer的概念正是受到了Elm的启发，在最终章我们会继续探讨这个话题

还有一点你可能已经注意到了，无论是前面的view还是这里的update函数，它们都没有return关键字！这是函数式语言非常重要的特点：一切都是expression，都需要有返回值。这强制你去表达`要什么`，而不是`做什么`。

简单的例子就是case语句和if语句：

```js
/* case statement */

//elm
case arg of
  value1 -> 
    result1
  value2 ->
    result2
    
//javascript
switch (expression) {
  case value1: 
    /*do sth*/ 
    return result1; 
    break
  case value2: 
    /*do sth*/ 
    return result2; 
    break 

/* if statement */

//elm
//else is required
if 3 > 2 then "cat" else "dog"

//javascript
if (3 > 2) {
  return 'cat'
} else { //else statement is optional
  return 'dog'
}

```

对`要什么`的分解在函数式思维中非常重要，通常会和递归联系起来，本文并不打算深入，建议有兴趣了解的朋友可以学习Elm官网[Examples](http://elm-lang.org/examples)中 functional stuff - recursion 小节下的例子


### 动态View
之前我们创建了一个静态的View，它没有任何事件相关的代码，因此也不可能响应用户行为。接下来让我们补全这一部分

```js
import Html exposing (..)
import Html.Events exposing (onClick)

view model =
  div []
    [ button [onClick Decrement] [text "-"]
    ,text (toString model)
    ,button [onClick Increment] [text "+"]
  ]

```

在第2行我们引入了Html.Events模块中`onClick`函数，`onClick Decrement`可以理解为当click事件发生时，它会输出一个`Decrement`消息。

可是向谁输出？输出的消息如何传递给update函数呢？让我们回顾一下所有的代码：

```js
import Html exposing (..)
import Html.Events exposing (onClick)

type Msg = Increment | Decrement

update msg model = 
  case msg of 
    Increment -> 
      model + 1
    Decrement ->
      model - 1

view model =
  div []
    [ button [onClick Decrement] [text "-"]
    ,text (toString model)
    ,button [onClick Increment] [text "+"]
  ]
  
initModel = 3
  
main = view initModel

```

目前为止，界面仍然是静态的。我们有了**数据**，**具备行为的视图**，**按行为改变数据的逻辑**，却没有将它们`粘合`成一个应用。

Elm为我们提供了这样的方法，在Html.App模块中

```js
import Html.App as App

main = App.beginnerProgram {model = initModel, view = view, update = update}
```

注意这里的方法名叫`beginnerProgram`，它的参数分别代表了：`Model`, `View`, `Update`，这是，Elm架构的最简形态（不考虑异步等副作用），也是任何符合Elm架构的组件都必不可少的三个部分，完整代码如下：

```js
import Html exposing (..)
import Html.Events exposing (onClick)
import Html.App as App

type Msg = Increment | Decrement

update msg model = 
  case msg of 
    Increment -> 
      model + 1
    Decrement ->
      model - 1

view model =
  div []
    [ button [onClick Decrement] [text "-"]
    , text (toString model)
    , button [onClick Increment] [text "+"]
  ]
  
initModel = 3

main = App.beginnerProgram {model = initModel, view = view, update = update}
```

## 小结
通过这个简单的Counter相信你已经对Elm有了初步的了解，如果回顾上面的代码你会发现其实函数式语言并不是那么晦涩或高深。

下一章中我们将会了解Elm的类型，并用类型优化Counter的代码
