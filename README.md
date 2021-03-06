# Geass

Lightweight, Blazing fast, based on [react](https://reactjs.org/), [redux](https://redux.js.org/), [angular-styled](https://angular.io/)，[functional-featured](https://en.wikipedia.org/wiki/Functional_programming) front-end framework.

Geass 是为了获取急速的开发速度而生的前端框架，它基于社区现有的优秀解决方案，并对其进行了一层简单的封装，力求在React的极简，Redux的可信赖，以及Angular的优秀之间取得一种合理的平衡。

<div align="center">
    <img width="500" heigth="800" src="./intuition.jpg">
  <br>
  <br>
</div>

# Features

### 基本想法

Geass 追求组件化，每个组件都是一个独立的单元，通过组合各种组件来完成应用程序的编写。

Geass 的每个组件包含三个文件，追求视图（使用React），样式（使用[Css Modules](https://github.com/css-modules/css-modules)），和逻辑（使用ES6 的Class，还有React，Redux，React-Router等）分离，并将提供 **CLI** 工具帮助一行命令快速创建一个标准组件。

### 事件

Geass 提供 `dispatch` API方法将三类 Action（路由，Http请求，本地事件）统一化处理，强调明确你目前在做的事。

- Http 请求，示例如下：

  ```javascript
  dispatch({ 
      type: 'http',
    	action: 'GET_REMOTE_USER', 
      payload?: { id: 1 },
  });
  ```

  dispatch 有三个参数，两个必要参数和一个可选参数：

- 第一个参数:：

  ```javascript
  { type: typeName: String }
  ```

  typeName = 'http' | 'local' | 'route' ，表示对应的事件类型，分别代表Http请求，本地事件处理，以及路由动作的处理。

- 第二个参数：

  ```javascript
  { action: actionName: String }
  ```

   actionName 代表着对应的动作名。

- 第三个参数：

  ```javascript
  { payload?: payloadData: Object }
  ```

  payloadData代表本次动作需要传递的数据，可以为：null | Object 。


接下来我将会一步一步的讲解这其中的奇思妙想。但是，在吃糖之前，让我们先卧薪尝胆一波。

在原始的React里面，你可能会写出如下的代码：

  举例如下：

  ```javascript
// 下面对比使用Geass前后的React组件书写变化

// 没有使用Geass之前
import React, { Component } from 'react';

class App extends Component {
  render() {
    return (
      <div>
      	<p>{this.props.nowNumber}</p>
      	<button onClick={this.props.handlePlus}>plus</button>
        <button onClick={this.props.handleMinus}>minus</button>
      </div>
    );
  }
}
  ```

  上面的 `handlePlus`  其实是个黑盒子，对吧，虽然最后 `nowNumber` 实际上是发生了变化，但是我们无法确定这个函数里面到底发生了什么：

- 它内部还可能是发起了路由跳转 `route state`


- 它内部可能只是单纯的修改了本地状态 `local state` ，这个状态在退出应用之后就会销毁。
- 它内部可能对远程数据库发起了请求，修改了远程状态 `remote state` ，这个状态将被永久保存。

上面提到的问题是非常显而易见的，而且也是目前绝大多数应用的一个通病，这样的代码显然难以维护，因为你在写完之后，过一段时间再来读以上代码，一个简单的 `handlePlus` 并不能给你提供很多有用的信息对吧？为了弄懂这个函数在背后做了什么事情，你不得不深入这个函数的内部逻辑，去一行一行的阅读其代码，这已经够恐怖了吧？但是更恐怖的是，万一这一行一行的代码又只有很少的注释，天啦！简直是地狱。

其实，变幻一下思路之后也没那么恐怖啦，花点功夫，我们能将函数名改造得更易懂：

- 如果内部发起了路由跳转，我们可以这样来写，`handleChangeRoutePlusNumber`
- 如果内部只是单纯的改变了本地的 `nowNumber` 不引起其他动作，那么我们可以这么写：`handleLocalPlusNumber`
- 如果内部发起了远程请求，那么我们就可以这样来写：`handleRemotePlusNumber`

好了，我们将函数名加长了许多，这样能明确表达我们这个函数将要完成的动作了。

有很多理由来证明这样做会带来很多的好处，进行代码维护时，通过团队之间的约定，就不用过多的去深入这个函数的细节就能读懂这段代码；在进行调错时如果这个函数内部出现错误，那么你能通过函数在引用堆栈里面的语义化名字来快速确定出错位置和出错原因... 

看到这里，可能你觉得这样子已经比较好了，大功告成。好了，你现在可以离开这里了，再见！

你没有走。

还不走？

好吧，我知道你还想继续看下去，因为如果只是简单的进行上面的改进，那么我也没有必要大张旗鼓的搞这么一个框架出来了。

让我们再仔细想一想，然后，你会猛然惊醒...

因为我们上面其实只是针对了一种情况进行处理，那么如果你的这个函数内部不仅要修改路由状态，还要修改本地其他状态，例如：

- 首先修改 `nowNumber`
- 然后修改路由状态

 甚至会对远程的状态进行改变呢？思路转得快的同学可能会马上想到如下的答案：`handleChangeRouteAndLocalPlusNumber` ，怎么样，很长了吧？

可是，这样还不够恐怖，那么我再来补一刀，这个函数里面还要进行远程数据库的读取：我们可能要这么写：`handleChangeRouteAndLocalAndRemotePlusNumber`。

这下你可见识到了恐怖了，如果在整个团队合作时，出现了像上面一样冗长的函数名，那么光是理解上来就要花成吨的时间，而且会让整个代码逻辑看上去冗长，这可不是一个好做法，写代码本应该是一件享受的事，不是嘛？

好的，接下来，就让你体会一下什么叫做超爽的开发体验。让我们来写一个经过Geass改进后的React组件。

先看例子：

```javascript
import React, { Component } from 'react';

class App extends Component {
  render() {
    return (
      <div>
      	<p>{this.props.nowNumber}</p>
      	<button onClick={() => {
      	  dispatch({
            type: 'local',
            action: 'PLUS_NUMBER',
          });
  		  }}>plus</button>
    	  <button onClick={() => {
          dispatch({
            type: 'local',
            action: 'MINUS_NUMBER',
          });
	      }}>minus</button>
      </div>
    );
  }
}
```

怎么样？是不是非常清晰且有层次感，明确的 dispatch 告诉我们，此时从 view 层发出了一个动作，动作的一个参数代表这个动作的类型，动作的第二个参数代表相应的动作名，这简直是一次声明代码的革命性呀！

有些同学看了之后可能会想，这有什么？代码还不是这么长，即使是简单的单一动作都要写的这么复杂？我真的应该付出嘛？

其实我觉得这种付出是非常有必要的，请看接下来的扩展：

如果 `handlePlusNumber` 那个函数内部还做了对路由状态的改变，我们可以这么写：

```javascript
dispatch(['local', 'handlePlusNumber'], ['route', 'handleChangeRoute'])
```

数组中的相应项对应相应的处理函数，怎么样？是不是感觉炫一点了，顾虑应该也消却了很多吧！

哈哈哈，我们在一次调用里面，做了两件事，处理了两种情况，并标注了每件情况对应的类型和相应的处理函数，每个处理函数都只做一件类型的事，这样，我们完成本次代码的编写之后，发布上线，等到其他人来看这段代码时，我们基于约定，就能很容易的确认这段代码做了什么事，而不用深入了解其处理函数背后的细节，大大有利于团队之间的沟通和交流，对代码的可维护性给予了一击重击。

我们还可以基于此做非常多的扩展，例如我们可以再加上一个处理动作，我们还要对远程服务器的 nowNumber 进行修改，我们可以这样写：

```javascript
dispatch(
  ['local', 'handlePlusNumber'], 
  ['route', 'handleChangeRoute'],
  ['remote', 'handleChangeServer'],
)
```

现在你发现了吧，我们不仅让动作更加的明确了，而且让一次动作可以完成多种操作，这可能就是一种革新吧。

### 组件

既然 ES6 代表着未来，那么我们就应该使用最现代化的技术，永远对创新保持着强烈的渴望，永远对强大的技术保持着激动之心。Geass 强调组件化，使用ES6 的类来封装 View 层所需要的状态和方法。通过装饰器来连接两者。

同时Geass 彻底将视图和状态分离，每一个视图层组件都是一个无状态组件，大大简化了排错范围，将状态集中式管理，方便调试，并且采用了非常申明式的方式来书写各种状态和方法，简化代码逻辑，提高了代码的可维护性。

基于以上的的状态，我们标准的Geass 组件可能会这样写：

```javascript
// app.component.js
import { Model } from 'geass';

@Model({
  selector: 'app',
  template: 'app.view.js',
  style: 'app.style.css',
  connect: (state) => {
    /**
     * use connect to pass the data to the component
     * Also, Component 
     *
     */
    return { nowNumber: state.count.nowNumber };
  },
})
export default class App {
  // reducer initial state
  state = {
    nowNumber: 0,
  };

  /**
   * The following three types of actions 
   * describe the fact that something happened.
   *
   * These actions will directly dispatch action to store, and let reducer 
   * update state
   */

  // handle local action
  localAction = {
    handlePlusNumber() {
      dispatch({ type: 'plus' });
    },
    // handle async plus logic, 
    *handlePlusNumber() {
      // this.props 是全局的资源分发中心，任何插件都在其上绑定。
      const { call, put, delay } = this.props;
      // 推辞1秒处理，模仿异步流程
      yield delay(1000);
   	  // 像store发出 minus 动作，reducer更新state
      yield put({ type: 'minus' });
  	},
  };

  // handle route action if exist
  routeAction = {
	handleChangeRoute() {}
  };

  // handle remote action is exist
  remoteAction = {
    // use redux-saga handle async action
	*handleChangeServer() {}
  };

  /**
   * Reducers describe the state changes in response
   * reducer is the only one which can update state
   *
   */
  reducers = {
    plus(state) {
      const newNumber = state.nowNumber + 1;
      return { ...state, nowNumber: newNumber };
    },
    minus(state) {
      const newNumber = state.nowNumber - 1;
      return { ...state, nowNumber: newNumber };
 	},
  };
}
```

接下来我将一步一步讲解这份代码清单。

现代的Web因为各种复杂应用的需求，一值在不断的进化，实际上越来越多的状态由前端来处理，前端与后端的职责在不断的分离，各自都更加专注于自己的领域。实际上，有心的同学可能会发现，Geass就像一个状态机一般，**Action** 作为引起状态变化的条件，**Reducer** 将目前的状态更新到下一个状态，**Connect** 负责将整个状态分发给多个子状态。

我将基于**Action**，**Reducer**，**Connect** 做进一步扩展。

### Action

三大Action：localAction, routeAction, remoteAction，作为状态转换的触发者，它同时存在同步动作和异步动作，
