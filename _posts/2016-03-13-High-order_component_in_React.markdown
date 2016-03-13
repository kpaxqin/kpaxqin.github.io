---
layout: post
title:  "React进阶——使用高阶组件（Higher-Order Components）优化代码"
description: "An easy, light-weight & composable way to improve your code in React."
date:   2016-03-13 11:30:00
categories: react high-order component HOC compose
---

## 什么是高阶组件

>Higher-Order Components (HOCs) are JavaScript functions which add functionality to existing component classes. 

通过函数向现有组件类添加功能，就是高阶组件。

让我们先来看一个可能是史上最无聊的高阶组件：

```js
function noId() {
  return function(Comp) {
    return class NoID extends Component {
      render() {
        const {id, ...others} = this.props;
        return (
          <Comp {...others}/>
        )
      }
    }
  }
}

const WithoutID = noId()(Comp);

```

这个例子向我们展示了高阶组件的工作方式：通过函数和闭包，改变已有组件的行为——这里是忽略`id`属性——而完全不需要修改任何代码。

之所以称之为`高阶`，是因为在React中，这种嵌套关系会反映到组件树上，层层嵌套就好像高阶函数的function in function一样，如图：

![HOC-img](https://github.com/kpaxqin/kpaxqin.github.io/raw/master/img/Screen%20Shot%202016-03-10%20at%209.13.49%20AM.png)

从图上也可以看出，组件树虽然嵌套了多层，但是实际渲染的DOM结构并没有改变。
如果你对这点有疑问，不妨自己写写例子试下，加深对React的理解。现在可以先记下结论：我们可以放心的使用多层高阶组件，甚至重复地调用，而不必担心影响输出的DOM结构。

借助函数的逻辑表现力，高阶组件的用途几乎是无穷无尽的：

## 适配器
有的时候你需要替换一些已有组件，而新组件接收的参数和原组件并不完全一致。

你可以修改所有使用旧组件的代码来保证传入正确的参数——考虑改行吧如果你真这么想

也可以把新组件做一层封装：

```js
class ListAdapter extends Component {
	mapProps(props) {
		return {/* new props */}
	}
	render() {
		return <NewList {...mapProps(this.props)} />
	}
}
```

如果有十个组件需要适配呢？如果你不想照着上面写十遍，或许高阶组件可以给你答案

```js
function mapProps(mapFn) {
	return function(Comp) {
		return class extends Component {
			render() {
				return <Comp {...mapFn(this.props)}/>
			}
		}
	} 
}

const ListAdapter = mapProps(mapPropsForNewList)(NewList);

```

借助高阶组件，关注点被分离得更加干净：只需要关注真正重要的部分——属性的mapping。

这个例子有些价值，却仍然不够打动人，如果你也这么想，请往下看：

## 处理副作用

纯组件易写易测，越多越好，这是常识。然而在实际项目中，往往有许多的状态和副作用需要处理，最常见的情况就是异步了。

假设我们需要异步加载一个用户列表，通常的代码可能是这样的：

```js
class UserList extends Component {
  constructor(props) {
    super();
    this.state = {
      list: []
    }
  }
  componentDidMount() {
    loadUsers()
      .then(data=> 
        this.setState({list: data.userList})
      )
  }
  render() {
    return (
      <List list={this.state.list} />
    )
  }
  /* other bussiness logics */
}
```

实际情况中，以上代码往往还会和其它一些业务函数混杂在一起——我们创建了一个**业务**与**副作用**混杂的、**有状态**的组件。

如果再来一个书单列表呢？再写一个BookList然后把loadUsers改成loadBooks ？
不仅代码重复，大量有状态和副作用的组件，也使得应用更加难以测试。

也许你会说可以考虑使用Flux。它确实可以让你的代码更清晰，但是在有些场景下使用Flux就像大炮打蚊子。比如一个异步的下拉列表，如果要考虑复用的话，传统的Flux/Reflux几乎无法优雅的处理，Redux稍好一些，但仍然很难做优雅。关于flux/redux的缺点不深入，有兴趣的可以参考[Cycle.js作者的文章](http://staltz.com/why-react-redux-is-an-inferior-paradigm.html?utm_source=javascriptweekly&utm_medium=email)

回到问题的本源：其实我们只想要一个能复用的异步下拉列表而已啊！

高阶函数试试？

```js
function connectPromise({promiseLoader, mapResultToProps}) {
  return Comp=> {
    return class AsyncComponent extends Component {
      constructor(props) {
        super();
        this.state = {
          result: undefined
        }
      }
      componentDidMount() {
        promiseLoader()
          .then(result=> this.setState({result}))
      }
      render() {
        return (
          <Comp {...mapResultToProps(props)} {...this.props}/>
        )
      }
    }
  }
}


const UserList = connectPromise({
	promiseLoader: loadUsers,
	mapResultToProps: result=> ({list: result.userList})
})(List); //List can be a pure component

const BookList = connectPromise({
	promiseLoader: loadBooks,
	mapResultToProps: result=> ({list: result.bookList})
})(List);

```

不仅大大减少了重复代码，还把散落各处的异步逻辑装进了可以单独管理和测试的笼子，而在业务场景中，只需要`纯组件 + 配置` 就能实现相同的功能——而无论是`纯组件`还是`配置`，都是对单元测试友好的，至少比异步组件友好多了。

## 使用curry & compose

高阶组件的另一个亮点，就是对函数式编程的友好。你可能已经注意到，目前我写的所有高阶函数，都是形如：

```js
config => {
	return Component=> {
		return HighOrderCompoent
	}
}
```

表示为`config=> Component=> Component`。

写成嵌套的函数是为了手动curry化，而参数的顺序（为什么不是`Component=> config=> Component`），则是为了组合方便。关于curry与compose的使用，可以移步我的[另一篇blog](https://segmentfault.com/a/1190000003939990?_ea=544245)

举个栗子，前面讲了适配器和异步，我们可以很快就组合出两者的结合体：使用NewList的异步用户列表

```js
UserList = compose(
  connectPromise({
	promiseLoader: loadUsers,
	mapResultToProps: result=> ({list: result.userList})
  }),
  mapProps(mapPropsForNewList)
)(NewList);

```


## 总结
在团队内部分享里，我的总结是三个词 Easy, Light-weight & Composable. 

其实高阶组件并不是什么新东西，本质上就是`Decorator`模式在React的一种实现，但在相当一段时间内，这个优秀的模式都被人忽略。在我看来，大部分使用`mixin`和`class extends`的地方，使用高阶组件都是更好的方案——毕竟组合优于继承，而mixin——个人觉得没资格参与讨论。

使用高阶组件还有两个好处：

1. 适用范围广，它不需要es6或者其它需要编译的特性，有函数的地方，就有HOC。
2. Debug友好，它能够被React组件树显示，所以可以很清楚地知道有多少层，每层做了什么。相比之下无论是mixin还是继承，都显得非常隐晦。

值得庆幸的是，社区也明显注意到了高阶组件的价值，无论是大家非常熟悉的[react-redux](https://github.com/reactjs/react-redux) 的`connect`函数，还是[redux-form](https://github.com/erikras/redux-form)，高阶组件的应用开始随处可见。

下次当你想写`mixin`或`class extends`的时候，不妨也考虑下高阶组件。
