---
layout: post
title:  "Elm入门实践——类型篇"
description: "Understanding type in Elm."
date:   2016-06-12 23:00:00 +0800
categories: elm functional-programming front-end
---

记得Facebook曾经在一次社区活动上说过，随着他们越来越多地使用Javascript，很快就面临了曾经在PHP上遇到的问题：这东西到底是啥？

动态语言就像把双刃剑，你可以爱死它的灵活性，也可能因为一个小的疏忽而损失惨重。Elm选择了静态强类型，这通常也是多数函数式语言的选择，没有了OO语言中`类`的概念，强大的类型系统负责解决一切“这是什么？”的问题


## 类型注解
也可以叫做类型签名，Elm 使用冒号`:`来注明类型，在Hello world的基础上，让我们分别定义一个变量和函数，并且注明类型

```js
import Html exposing (..)
import Html.Attributes exposing (..)

elm : String
elm = "elm"

sayHello : String -> String
sayHello name = 
  "Hello, " ++ name

main =
  div [class "hello"] 
    [ span [] [text (sayHello elm)]
    ]
```

尝试将elm的值"elm"改为数字，看看会发生什么？

```js
Detected errors in 1 module.


-- TYPE MISMATCH ---------------------------------------------------------------

The type annotation for `elm` does not match its definition.

5| elm : String
         ^^^^^^
The type annotation is saying:

    String

But I am inferring that the definition has this type:

    number
```

编译器发现了错误，并且能够定位到具体的行数。

如果不声明类型呢？如果注释掉类型注解

```elm
import Html exposing (..)
import Html.Attributes exposing (..)

--elm : String
elm = 6

--sayHello : String -> String
sayHello name = 
  "Hello, " ++ name

main =
  div [class "hello"] 
    [ span [] [text (sayHello elm)]
    ]

```

重新编译，还是会报错，只是错误信息变了，这次是第14行：

```js
Detected errors in 1 module.


-- TYPE MISMATCH ---------------------------------------------------------------

The argument to function `sayHello` is causing a mismatch.

14|                      sayHello elm)
                                  ^^^
Function `sayHello` is expecting the argument to be:

    String

But it is:

    number
```

即使没有显式的类型注解，Elm的类型推导系统也会发挥作用，此处通过类型推导认为`sayHello`函数的参数应该是字符串，但是传入了数字，因此报错。

对比两次不同的错误提示可以看出，类型注解能让编译器更准确地发现和定位错误。随着学习的深入你会慢慢喜欢上类型系统带来的安全感：如果编译失败，明确的提示能帮助你快速定位问题。而只要编译通过，程序就一定能运行。


## 基本类型和List

### 基本类型

基本类型和多数语言是类似的，无非就是`String`, `Char`, `Bool` `Int`, `Float`，可以参考官网的[literals](http://elm-lang.org/docs/syntax#literals)。需要注意在Elm中，`String`必须用双引号，单引号是用来表示`Char`的，字符串单引号党需要适应一下。

### List
严格来说List并不是类型，它的类型是`List a`，其中的`a`被称作`类型变量`，这是因为List作为容器，它可以装String，Int，或者什么都不装，因此类型必须是动态的：

```js
> [ "Alice", "Bob" ]
[ "Alice", "Bob" ] : List String

> [ 1.0, 8.6, 42.1 ]
[ 1.0, 8.6, 42.1 ] : List Float

> []
[] : List a
```

关于`类型变量`后面会继续讨论，在这里我们只需要记住一点：**List不是类型，类似List String这样的才是**。

由于参数只有一个，Elm的List只能容纳单一类型的元素，和Javascript来者不拒的List不同，下面这样的是会被编译器发现并报错的：

```js
list = [1, "a"]

```


## 类型别名

类型别名用于组合或复用已知的类型，比如

```elm
type alias Name = String
type alias Age = Int
  
type alias User = {name: Name, age: Age}

user : User
user =
  { name = "Zhang zhe", age = 89 }
  
setUserName : String -> User -> User
setUserName name user = {user | name = name}

```

它不仅可以让基本类型具备业务语义，还可以为复杂的数据结构组合出合适的、语义化的类型。没有别名的话，setUserName的类型签名就得写成下面这样……一坨：

```js
setUserName : String -> {name: String, age: Int} -> {name: String, age: Int}

```


## Union Types
Union type 是Elm类型系统中最重要的部分之一，它用来表示一组可能的值，每个值叫做一个`Tag`，如下：

```js
type Bool = True | False

type User = Anonymos | Authed
```

其中`True`和`False`， `Anonymos`和`Authed `都是`Tag`名（**注意Tag不是Type**）。看起来很像枚举？不只这样，Union type强大的地方在于：`Tag`可以携带一组已知类型。上面的代码我们虽然能区分两类用户，但并不能获取认证用户的名称，这时候就可以用已知类型结合Tag表达：

```js
type User = Anonymos | Authed String

```

当我们创建Union type的时候，实际上为每个`Tag`都创建了相应的`值构造器`：

```js
> type User = Anonymous | Authed String

> Anonymous
Anonymous : User

> Authed
Authed : String -> User

```

不带其它信息的Anonymous可以直接作为`值`使用（想想`True`和`False`），而带有已知类型的Authed实际上是一个函数，它接受String，返回User类型：

```js
users : List User
users = [
  Anonymous, 
  Authed "Kpax"]
```

>在Haskell中没有Tag的叫法，相似的东西就叫`值构造器`(value constructor)，直接的表明了它的用途：构建属于该类型的值

Tag还可以被解构：

```js
getAuthedUserName : User -> String
getAuthedUserName user =
  case user of 
    Anonymous ->
      ""
    Authed name ->
      name
```
这个函数返回Authed用户的名称，如果是Anonymous用户则返回空字符串。

完整的可在在线编辑器中执行的代码如下：

```js
import Html exposing (..)
import List

type User = Anonymous | Authed String

users : List User
users = [
  Anonymous, 
  Authed "Kpax",
  Authed "qin"]

getAuthedUserName : User -> String
getAuthedUserName user =
  case user of 
    Anonymous ->
      ""
    Authed name ->
      name      

main =
  div [] (List.map (text << getAuthedUserName) users)
```

> `text << getAuthedUserName` 使用了Elm中的[<<](http://package.elm-lang.org/packages/elm-lang/core/4.0.1/Basics#<<)操作符实现两个函数的compose，类似于lodash中的[_.flowRight](https://lodash.com/docs#flowRight)函数


## Type variables
上面已经提到了`List a`类型，其中`a`即类型变量，表示一个当前还不确定的类型，类似于面向对象编程中泛型的概念

map函数的类型签名也使用了类型变量：

```js
map : (a -> b) -> a -> b
```

这使得我们调用map函数`map userToString user`时，只要保证user是User类型即可，map函数并不关心具体的类型。

那么如何定义一个`List a`类型呢？代码如下

```elm
type List a = Empty | Node a (List a)
```

前面说到`Tag`可以携带已知类型，那么是否可以携带正在定义的这个类型呢？答案是肯定的！这就是类型的`递归`，`List a`就是这样一个带有类型参数的递归类型，平时我们写的数组，可以理解为如下代码的语法糖

```elm
-- []
Empty

-- [1]
Node 1 Empty

-- [1, 2, 3]
Node 1 (Node 2 (Node 3 Empty))
```

同样的思路，我们完全可以自己实现二叉树等数据结构，有兴趣的朋友不妨试试，官方文档有[相关章节](http://guide.elm-lang.org/types/union_types.html#binary-trees)可供参考
 

## Counter with type
上一章[基础篇]()我们讲了Counter的实现，代码如下：

```elm
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

让我们用刚刚学习的知识给以上代码添加类型和类型注解

首先，我们有`initModel`这个数据，它的类型是`Int`，不具备任何业务语义，让我们定义一个类型别名`Model`来表示Counter的数据

```elm
type alias Model = Int
```
自然`initModel`的类型应该为`Model`

```elm
initModel : Model
initModel = 3
```

`update`函数的类型签名比较简单，它接受消息`Msg`和当前数据`Model`，返回新的数据`Model`: 

```elm
update : Msg -> Model -> Model
```

`view`函数接受`Model`类型的数据，返回什么呢？如果查阅`div`函数的[文档](http://package.elm-lang.org/packages/elm-lang/html/1.0.0/Html#div)，你会发现返回的是一个带有类型变量的类型`Html msg`。其实很好理解，因为渲染界面的函数不仅要输出Html，当事件发生时还要输出`消息`，输出消息的类型，就是应该赋给变量`msg`的类型，在`Counter`中消息的类型是`Msg`，因此:

```elm
view : Model -> Html Msg
```

完整代码：

```elm
import Html exposing (..)
import Html.Events exposing (onClick)
import Html.App as App

type alias Model = Int

type Msg = Increment | Decrement

update : Msg -> Model -> Model
update msg model = 
  case msg of 
    Increment -> 
      model + 1
    Decrement ->
      model - 1

view : Model -> Html Msg
view model =
  div []
    [ button [onClick Decrement] [text "-"]
    , text (toString model)
    , button [onClick Increment] [text "+"]
  ]

initModel : Model
initModel = 3

main = App.beginnerProgram {model = initModel, view = view, update = update}
```

## 总结
类型的学习可能有些枯燥，但是非常重要。如果你了解redux，你会发现Union type简直天生就是做action的料，比起redux在javascript中使用的字符串既简洁又达意，甚至还可以嵌套组合，谈笑风生！高到不知道哪里去了！

下一章我们将把在线编辑器放到一边，把Counter迁移到本地运行，然后实现一个CounterList，在CounterList中，你会看到Elm是如何复用组件，以及为什么Elm被称为理想的**分形**架构。

>各种架构对比，可以参考Cycle.js作者Andre Staltz的[文章](http://staltz.com/unidirectional-user-interface-architectures.html)
