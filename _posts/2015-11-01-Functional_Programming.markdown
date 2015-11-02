---
layout: post
title:  "函数式编程——入门笔记与React实践"
description: "函数式编程入门笔记；Curry与Compose；使用函数式编程处理react组件事件"
date:   2015-11-01 19:30:00
categories: FP functional-programming curry compose react
---

##前言
最近在看近来很火的函数式编程教程[《Mostly Adequate Guide》](book) （中文版：[《JS函数式编程指南》](book_cn)），收获很大。对于函数式编程的初学者，这本书不仅深入浅出，更让人感受到函数式编程的优势和美感，强烈推荐给想要学习函数式编程的朋友。

这篇文章是我个人的一个学习笔记，在总结知识的同时，也尝试以React组件的输入事件响应为例，用函数式编程去应对实际项目中的场景。

下文涉及React的代码出于阅读考虑有一定删减，完整代码在我的[Github](https://github.com/kpaxqin/fp-note/commits/master)。

lodash与ramda部分代码由于比较简单，想看运行结果的话可以直接到[lodash](https://lodash.com/docs)或[ramda](http://ramdajs.com/0.18.0/docs/)官网打开console运行。


## 纯函数

纯函数引用原书的描述：
>纯函数是这样一种函数，即相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用。
>

`相同的输入，永远会得到相同的输出`，通常意味着对外部状态解耦。

所谓外部状态，最常见的例子就是this，如果你的函数是：

```js
function(){
	return 'hello, ' + this.name；
}
```
那它不可能是纯函数——你永远不知道this.name会被谁改写，测试用例也不可能覆盖所有情况。如果正巧有一个外部函数，它每隔一个月将this.name改写成`'shit'`，你和测试人员熬了几个通宵没有发现一点问题，你也信心满满——用函数给客人打招呼实在太简单。项目上线后，你买好机票正准备出门度假，却接到老板的电话让你滚回公司改bug，而你对事情的状况没有一点头绪……

提倡函数式编程的人认为，这种**共享状态导致的混乱是绝大多数bug的万恶之源**

其实某种程度上这早已成为共识：不提倡全局变量其实就是这个道理。也许深刻意识到纯函数的优势还需要一点时间，也许你觉得纯函数不错，但对于如何在项目中使用它完全没有头绪，不用着急，现在我们暂时先记住：
>做一个纯粹的函数，一个脱离了低级趣味的函数

## Curry与Compose

curry和compose可以说是函数式编程的**众妙之门**，而且必须相辅相成才见威力。就我个人而言，见过一些讲函数式的教程，讲了curry，我也知道了什么是curry，但是curry怎么用？能带来什么好处呢？还是没讲清楚，然后马不停蹄地往前讲functor讲monad，作为资质不那么高的函数式菜鸟，很快就云里雾中，不明觉厉了。

###Curry
curry的本质是函数的`部分应用`。听起来有点遥远，事实上类似的需求我们经常会遇到：

```js
class Form extends React.Component {
	setField(key){
		return (e)=>{
			this.setState({
		   	  [key]: e.target.value
		   })
		}
	}
	render(){
		const {name, address} = this.state;
		return (
			<form>
				<input 
					value={name}
					onChange={this.setField('name')} 
					/>
				<input 
					value={address}
					onChange={this.setField('address')} 
					/>
			</form>
		)
	}
}
```
[完整代码 step_1](https://github.com/kpaxqin/fp-note/blob/6816bfd024f4389de89f7924870a6ef1633536e8/src/index.jsx)

借助高阶函数式的function return function，对不同的key我们能够复用响应事件并setState的逻辑，上例可以认为就是脱掉了马甲的`部分应用`。

换个写法试试：

```js
const setFieldOnContext = _.curry(function(context, key, e){
	context.setState({
		[key]: e.target.value
	})
});

class Form extends React.Component{
	render(){
		const {name, address} = this.state;
		const setField = setFieldOnContext(this);
		return (
			<form>
				<input 
					value={name}
					onChange={setField('name')} 
					/>
				<input 
					value={address}
					onChange={setField('address')} 
					/>
			</form>
		)
	}
}
```
[完整代码 step_2](https://github.com/kpaxqin/fp-note/blob/353c0f39554dded4c72bbc9a9d5d51ed38aa36f1/src/index.jsx)

部分应用的特性使得我们可以把关注点分散到每一个参数，在render函数中
>const setField = setFieldOnContext(this);

设定了当前上下文，因为你肯定不会设置其它component的state，而具体到每一个onChange则关注不同的目标key。

也许你会想curry是让代码变得好看了一点，但也仅此而已，它只是用新的姿势解决问题，并没有解决新的问题或产生新的价值。

当然不是，**curry真正产生的价值和魅力的地方，是它对组合的友好。**

###Compose

对逻辑进行组合，这样的需求其实很常见，当我想要：
>将一个数组去重，然后筛选，最后排序

很多时候会写成这样：

```js
import _ from 'lodash'

function filterFn(v){
	return typeof v === 'number';
}
function sortFn(v){
	return Math.abs(v);
}

_.sortBy(_.filter(_.uniq([1, 1, 3, 4, 2, 'a', -10]), filterFn), sortFn);
// -> [1, 2, 3, 4, -10]
```
嵌套的代码难以阅读，就像回调地狱一样。自然的逻辑应该是顺序而非嵌套的，因此很多人会更喜欢”链式“写法：

```js
_([1, 1, 3, 4, 2, 'a', -10]).
	uniq().
	filter(filterFn).
	sortBy(sortFn).
	value();
// -> [1, 2, 3, 4, -10]
```
看起来顺眼多了，用瓶子把东西封起来操作的思路很棒（`functor`就是这么干的，下次我们会细说）。然而问题在于，`_(x)`的原型链上可供我们链式调用的函数是有限的，这限制了我们的**逻辑表现力**。

一个典型的场景是代码调试：我们想知道每一步的返回值，以便定位问题。然而无论是单步调试还是log打印，在面对链式代码时都显得有些束手无策（chrome devtool可以选中部分代码并执行，但对编译生成的代码不管用），如果你不想每次debug都把要打印的值扔给临时变量搞得一地鸡毛的话，或许可以这样：

```js
//_ is lodash
_.prototype.log = function log(label){
	var value = this.value();
	console.log(label, value);
	return _(value);
};

_([1, 1, 3, 4, 2, 'a', -10]).
	uniq().
	log('does uniq() works right? ').
	filter(filterFn).
	sortBy(sortFn).
	value();
```
可惜lodash原生并没有提供这样的log函数。这不难理解，原型链有尽而需求场景无穷，扩充原型来满足业务场景是注定被动的。

即使你打算打破教条
>不是你的对象不要动
>               ——隔壁老王法则

决定像上面代码一样扩充第三方对象的原型，这个log函数仍然有太多怪异的地方，解包`var value = this.value()`和封包`return _(value)`的过程让人感到多余——有种脱掉裤子，放了个屁，然后穿回去的即视感。更重要的是这当中还伴随着对this关键字的依赖，或许你现在觉得没什么大不了的，但我希望你在看完这篇文章后能对this有更审慎的想法。

如果你还有其它更好的debug方法和经验，请一定分享出来。不过现在，让我们以Ramda为例，看看在函数式的世界里，问题是如何被解决的：

```js
import R from 'ramda'

var log = R.curry(function (label, value){
	console.log(label, value);
	return value
});

R.compose(
	R.reverse, 
	log('why we need a reverse ?'),
	R.sort(sortFn), 
	R.filter(filterFn), 
	R.uniq
)([1, 1, 3, 4, 2, 'a', -10])
// ->  [1, 4, 3, 2, -10]
```
`R.compose`接收一组函数并返回了一个新的函数，而数据就像经过一条逻辑流水线一样，从最后一个函数，一步步地向前接受处理。

`R.sort(fn, data)`和`R.filter(fn, data)`都是curry函数，你应该已经注意到它们和lodash的同名函数有所不同——参数顺序是相反的。这就是curry与compose协同工作的奥秘：**compose通常只能针对一元函数，而curry则使得多元函数可以一元化**。

函数curry化，并把可变性高复用性低的参数后置，是函数式库的特征之一，也是写自定义函数时需要注意的地方。我们的log函数就遵循了这一点。

组合相比链式最大的优势，是函数可以自由而专注：不再受原型链的约束，也不再看this的脸色。对比之前的log函数，现在的版本没有了多余的解包与封包，也不再依赖this——现在它是一个纯函数。

很多人会用`_`而不是`R`作为ramda的变量名，我们接下来也会这样

>关于组合更多的内容，还是强烈建议移步《Mostly Adequate Guide》的 [第 5 章: 代码组合](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/ch5.html)

##应用实践

在大致了解了函数组合后，让我们继续前面事件响应的例子，先回顾一下，之前我们用curry改写了`setField`函数，得到`setFieldOnContext`：

```js
const setFieldOnContext = _.curry(function(context, key, e){
    context.setState({
        [key]: e.target.value
    })
});

class Form extends React.Component{
    render(){
        const {name, address} = this.state;
        const setField = setFieldOnContext(this);
        return (
            <form>
                <input 
                    value={name}
                    onChange={setField('name')} 
                    />
                <input 
                    value={address}
                    onChange={setField('address')} 
                    />
            </form>
        )
    }
}
```

`setFieldOnContext`函数已经能帮我们节省一些重复代码，就像它的前辈`setField`一样，然而它的职责还分离得不够干净：对`e.target.value`的依赖使得它只能处理原生事件对象。假设我们有一些第三方组件（比如接下来会遇到的X组件），它们的`onChange`抛出了并不标准的事件对象，甚至可能直接把value扔了出来。看起来`setFieldOnContext`有些不从心，难道我们只能回到`复制--粘贴--修改`的怀抱吗？是时候借用组合的力量了：

```js
import _ from 'ramda'

const getValueFromEvent = function(e){
	return e.target.value;
};
const getValueFromX = function(x){
	return x.value
}
const setFieldOnContext = _.curry(function(context, key, value){
  context.setState({
    [key]: value
  })
});

class Form extends React.Component{
	render(){
		const {name, x} = this.state;
		const setField = setFieldOnContext(this);
		return (
			<form>
				<input
	            value={name}
	            onChange={_.compose(setField('name'), getValueFromEvent)}
	            />
          		<X
	            value={address}
	            onChange={_.compose(setField('address'), getValueFromX)}
	            />
			</form>
		)
	}
}

```
[完整代码 step_3](https://github.com/kpaxqin/fp-note/blob/2204cf4bf82d6b1f8a0473919cba0bb76ed79d55/src/index.jsx)

借助compose，我们的函数职责更加分离，setField只关心设值，对值的转换则由其它函数负责，虽然目前实现的版本用起来还有一些啰嗦，但我们得到了三个关注点（职责）高度分离的、可复用的函数。


在接着讨论前，让我们先统一一下用词，下面我会把`getValueFromEvent `和`getValueFromX ` 这样的值转换函数称作valueAdapter，正如它们的角色（适配器模式中的适配器）一样。

刚刚的代码之所以啰嗦，问题出在参数顺序和复用度不一致。

`_.compose(..., valueAdapter)`其本质是对**一类**事件进行适配，而我们把它放在参数最后，这导致适配的工作落在了每**一次**事件声明上。随着项目的发展，情况会是这样：

```js
<form>
	<input
		value={name}
		onChange={_.compose(setField('foo'), getValueFromEvent)}
		/>
	<input
		value={name}
		onChange={_.compose(setField('bar'), getValueFromEvent)}
		/>
	<input
		value={name}
		onChange={_.compose(setField('baz'), getValueFromEvent)}
		/>
	<input
		value={name}
		onChange={_.compose(setField('baa'), getValueFromEvent)}
		/>
	<input
		value={name}
		onChange={_.compose(setField('zzz'), getValueFromEvent)}
		/>
</form>

```
满眼的`getValueFromEvent`，完全背离了我们抽象出`valueAdapter`的初衷！

这重申了curry的要点：**通常我们会按照复用程度从高到低地排列参数**，比如在同一个组件中，context的复用度最高，而key则次之，event没有复用度——每个事件源都是单独的。至于`valueAdapter`们，它们的复用范围是一类组件。因此，在上例中我们更希望得到一个参数顺序类似于`fn(valueAdapter, context, key, event)`的函数。

下面是封装一层函数做参数顺序转换然后curry化的简单实现：

```js
import _ from 'ramda'

const getValueFromEvent = function(e){
	return e.target.value;
};
const getValueFromX = function(x){
	return x.value
}
const setFieldOnContext = _.curry(function(context, key, value){
  context.setState({
    [key]: value
  })
});

const getFieldSetter= _.curry(function(valueAdapter, context, name){
	//返回真正的event handler
	return _.compose(setFieldOnContext(context, name), valueAdapter);
});

const setFieldForEvent = getFieldSetter(getValueFromEvent);

const setFieldForX = getFieldSetter(getValueFromX);

React.createClass({
	render(){
		const {name, x} = this.state;
		return (
			<form>
				<input 
					value={name}
					onChange={setFieldForEvent(this, 'name')} 
					/>
				<X 
					value={x}
					onChange={setFieldForX(this, 'x')} 
					/>
			</form>
		)
	}
})

```
[完整代码 step_4](https://github.com/kpaxqin/fp-note/blob/b4bfa8f83ae714ca225751f0ae08e0b11386096a/src/index.jsx)

数一数，我们一下子有了六个函数！或许你会为此感到不安：是不是弄错了什么？

不必担心，仔细看看，这六个函数都有各自的复用价值，随着项目的发展和膨胀，响应事件值的需求随处可见，而重复的代码和逻辑会慢慢蚕食可维护性。把高度解耦的函数们（比如valueAdapter们）组合起来，会让我们更轻松的应对挑战。

还有一点，上面六个函数中有五个都是纯函数！除了`setFieldOnContext`，每个函数的输入输出都是唯一映射的（虽然有的输出是函数），没有依赖外部状态，也没有任何的副作用。

追求纯函数有时候会比较困难，但它是值得的，如果你的函数依赖了this，或者其它外部状态，那最好重新审视你的代码——至少把不安全的依赖剥离到最小范围。`setFieldOnContext`就是个例子，借助`context`变量而不是`this`，我们可以不用在意compose出来的event handler的this是谁，如何传递。自由组合函数的前提，就是它们不管在哪儿都始终如一。尽管React的setState返回`undefined`导致`setFieldOnContext`不能成为纯函数，我们也尽力让它更加接近这一目标

当然这一版本的实现仍不完美：`setFieldOnContext`把值直接设到了state的属性上，有时候这并不是我们想要的结果。在此我先不给出实现，留给看官思考和动手。

>补充：我的实现见 [代码step_5](https://github.com/kpaxqin/fp-note/blob/28caec834cbb772bfe77d56dbfdfa8a5be6fc40a/src/index.jsx) , 感谢[Young](https://github.com/littlehaker)的建议与启发

下一篇，我会借助Promise这个老面孔来介绍Functor和Monad——这两个你甚至没有见过，却无处不在的概念。




[book]:  http://drboolean.gitbooks.io/mostly-adequate-guide/
[book_cn]:  https://llh911001.gitbooks.io/mostly-adequate-guide-chinese
