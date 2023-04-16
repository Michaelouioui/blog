# React 常考基础知识点

这一章节我们将来学习 React 的一些经常考到的基础知识点。

## 生命周期

在 V16 版本中引入了 Fiber 机制。这个机制一定程度上的影响了部分生命周期的调用，并且也引入了新的 2 个 API 来解决问题，关于 Fiber 的内容将会在下一章节中讲到。

在之前的版本中，如果你拥有一个很复杂的复合组件，然后改动了最上层组件的 `state`，那么调用栈可能会很长



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/6/25/164358b0310f476c~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)



调用栈过长，再加上中间进行了复杂的操作，就可能导致长时间阻塞主线程，带来不好的用户体验。Fiber 就是为了解决该问题而生。

Fiber 本质上是一个虚拟的堆栈帧，新的调度器会按照优先级自由调度这些帧，从而将之前的同步渲染改成了异步渲染，在不影响体验的情况下去分段计算更新。



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/6/25/164358f89595d56f~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)



对于如何区别优先级，React 有自己的一套逻辑。对于动画这种实时性很高的东西，也就是 16 ms 必须渲染一次保证不卡顿的情况下，React 会每 16 ms（以内） 暂停一下更新，返回来继续渲染动画。

对于异步渲染，现在渲染有两个阶段：`reconciliation` 和 `commit` 。前者过程是可以打断的，后者不能暂停，会一直更新界面直到完成。

**Reconciliation** 阶段

- `componentWillMount`
- `componentWillReceiveProps`
- `shouldComponentUpdate`
- `componentWillUpdate`

**Commit** 阶段

- `componentDidMount`
- `componentDidUpdate`
- `componentWillUnmount`

因为 Reconciliation 阶段是可以被打断的，所以 Reconciliation 阶段会执行的生命周期函数就可能会出现调用多次的情况，从而引起 Bug。由此对于 Reconciliation 阶段调用的几个函数，除了 `shouldComponentUpdate` 以外，其他都应该避免去使用，并且 V16 中也引入了新的 API 来解决这个问题。

`getDerivedStateFromProps` 用于替换 `componentWillReceiveProps` ，该函数会在初始化和 `update` 时被调用

```
class ExampleComponent extends React.Component {
  // Initialize state in constructor,
  // Or with a property initializer.
  state = {};

  static getDerivedStateFromProps(nextProps, prevState) {
    if (prevState.someMirroredValue !== nextProps.someValue) {
      return {
        derivedData: computeDerivedState(nextProps),
        someMirroredValue: nextProps.someValue
      };
    }

    // Return null to indicate no change to state.
    return null;
  }
}
```

`getSnapshotBeforeUpdate` 用于替换 `componentWillUpdate` ，该函数会在 `update` 后 DOM 更新前被调用，用于读取最新的 DOM 数据。

## setState

`setState` 在 React 中是经常使用的一个 API，但是它存在一些的问题经常会导致初学者出错，核心原因就是因为这个 API 是异步的。

首先 `setState` 的调用并不会马上引起 `state` 的改变，并且如果你一次调用了多个 `setState` ，那么结果可能并不如你期待的一样。

```
handle() {
  // 初始化 `count` 为 0
  console.log(this.state.count) // -> 0
  this.setState({ count: this.state.count + 1 })
  this.setState({ count: this.state.count + 1 })
  this.setState({ count: this.state.count + 1 })
  console.log(this.state.count) // -> 0
}
```

第一，两次的打印都为 0，因为 `setState` 是个异步 API，只有同步代码运行完毕才会执行。`setState` 异步的原因我认为在于，`setState` 可能会导致 DOM 的重绘，如果调用一次就马上去进行重绘，那么调用多次就会造成不必要的性能损失。设计成异步的话，就可以将多次调用放入一个队列中，在恰当的时候统一进行更新过程。

第二，虽然调用了三次 `setState` ，但是 `count` 的值还是为 1。因为多次调用会合并为一次，只有当更新结束后 `state` 才会改变，三次调用等同于如下代码

```
Object.assign(  
  {},
  { count: this.state.count + 1 },
  { count: this.state.count + 1 },
  { count: this.state.count + 1 },
)w
```

当然你也可以通过以下方式来实现调用三次 `setState` 使得 `count` 为 3

```
handle() {
  this.setState((prevState) => ({ count: prevState.count + 1 }))
  this.setState((prevState) => ({ count: prevState.count + 1 }))
  this.setState((prevState) => ({ count: prevState.count + 1 }))
}
```

如果你想在每次调用 `setState` 后获得正确的 `state` ，可以通过如下代码实现

```
handle() {
    this.setState((prevState) => ({ count: prevState.count + 1 }), () => {
        console.log(this.state)
    })
}
```

## 性能优化

这小节内容集中在组件的性能优化上，这一方面的性能优化也基本集中在 `shouldComponentUpdate` 这个钩子函数上做文章。

> PS：下文中的 state 指代了 state 及 props

在 `shouldComponentUpdate` 函数中我们可以通过返回布尔值来决定当前组件是否需要更新。这层代码逻辑可以是简单地浅比较一下当前 `state` 和之前的 `state` 是否相同，也可以是判断某个值更新了才触发组件更新。一般来说不推荐完整地对比当前 `state` 和之前的 `state` 是否相同，因为组件更新触发可能会很频繁，这样的完整对比性能开销会有点大，可能会造成得不偿失的情况。

当然如果真的想完整对比当前 `state` 和之前的 `state` 是否相同，并且不影响性能也是行得通的，可以通过 immutable 或者 immer 这些库来生成不可变对象。这类库对于操作大规模的数据来说会提升不错的性能，并且一旦改变数据就会生成一个新的对象，对比前后 `state` 是否一致也就方便多了，同时也很推荐阅读下 immer 的源码实现。

另外如果只是单纯的浅比较一下，可以直接使用 `PureComponent`，底层就是实现了浅比较 `state`。

```
class Test extends React.PureComponent {
  render() {
    return (
      <div>
        PureComponent
      </div>
    )
  }
}
```

这时候你可能会考虑到函数组件就不能使用这种方式了，如果你使用 16.6.0 之后的版本的话，可以使用 `React.memo` 来实现相同的功能。

```
const Test = React.memo(() => (
    <div>
        PureComponent
    </div>
))
```

通过这种方式我们就可以既实现了 `shouldComponentUpdate` 的浅比较，又能够使用函数组件。

## 通信

其实 React 中的组件通信基本和 Vue 中的一致。同样也分为以下三种情况：

- 父子组件通信
- 兄弟组件通信
- 跨多层级组件通信
- 任意组件

### 父子通信

父组件通过 `props` 传递数据给子组件，子组件通过调用父组件传来的函数传递数据给父组件，这两种方式是最常用的父子通信实现办法。

这种父子通信方式也就是典型的单向数据流，父组件通过 `props` 传递数据，子组件不能直接修改 `props`， 而是必须通过调用父组件函数的方式告知父组件修改数据。

### 兄弟组件通信

对于这种情况可以通过共同的父组件来管理状态和事件函数。比如说其中一个兄弟组件调用父组件传递过来的事件函数修改父组件中的状态，然后父组件将状态传递给另一个兄弟组件。

### 跨多层次组件通信

如果你使用 16.3 以上版本的话，对于这种情况可以使用 Context API。

```
// 创建 Context，可以在开始就传入值
const StateContext = React.createContext()
class Parent extends React.Component {
  render () {
    return (
      // value 就是传入 Context 中的值
      <StateContext.Provider value='yck'>
        <Child />
      </StateContext.Provider>
    )
  }
}
class Child extends React.Component {
  render () {
    return (
      <ThemeContext.Consumer>
        // 取出值
        {context => (
          name is { context }
        )}
      </ThemeContext.Consumer>
    );
  }
}
```

### 任意组件

这种方式可以通过 Redux 或者 Event Bus 解决，另外如果你不怕麻烦的话，可以使用这种方式解决上述所有的通信情况

## 小结

总的来说这一章节的内容更多的偏向于 React 的基础，另外 React 的面试题还会经常考到 Virtual DOM 中的内容，所以这块内容大家也需要好好准备。

下一章节我们将来了解一些 React 的进阶知识内容。

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（84）

[![hwjfqr的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2137106335671198)

[hwjfqr![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2137106335671198)

前端开发3月前

买上当了

点赞

回复

[![hhhao的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2999123451053181)

[hhhao](https://juejin.cn/user/2999123451053181)

大前端7月前

对不起，我一个客户端看完都觉得这两章太水了![[流汗]](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/jj_emoji_26.1117a72.png)

2

回复

[![happyRen的头像](https://p3-passport.byteacctimg.com/img/mosaic-legacy/3797/2889309425~300x300.image)](https://juejin.cn/user/932036831894664)

[happyRen![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/932036831894664)

前端开发10月前

这钱花的不值，知识点就说的这么少的可怜。作者能再补补吗

2

回复

[![明媚的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3931509312724168)

[明媚](https://juejin.cn/user/3931509312724168)

1年前

能退钱吗！这个小册真的有点坑，不深入不全面，就连setState 也直接说一句异步的就过去了？难道在setTimeout 和原生事件里不是同步的吗？多花钱我不生气，钱花的不值我才生气！如果这文章我觉得这种错了可以理解，毕竟是自己的理解嘛！但是你拿出来卖钱的东西竟然一知半解的，这有点说不过去了吧

16

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2260251635103272)

[木子七![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/2260251635103272)

10月前

确实水，前面就觉得水，但处在能接受的边界，这一章我了个去，再看下一章更气人，跟你来句。“很多同学可能觉得这一章内容少，因为看了前一章基础就能应付 react 大部分考题”，what fk???

点赞

回复

[![枫栖山岚的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1310273593424296)

[枫栖山岚![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1310273593424296)

前端1年前

没有讲hooks 没有讲redux，这俩不都是高频考点么

5

回复

[![洛竹的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/325111174662855)

[洛竹![lv-5](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f8d51984638784aff27209d38b6cd3bf.svg)](https://juejin.cn/user/325111174662855)

木系前端 @ 罗小黑战记1年前

那个展示调用栈的图是怎么搞的呀，小白很好奇！

点赞

回复

[![洛竹的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/325111174662855)

[洛竹![lv-5](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f8d51984638784aff27209d38b6cd3bf.svg)](https://juejin.cn/user/325111174662855)

木系前端 @ 罗小黑战记1年前

那个展示调用栈的图是怎么搞的呀，小白很好奇！

点赞

回复

[![小kian的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3491704658999661)

[小kian![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3491704658999661)

前端工程师 @ 涂鸦智能1年前

react这边实在是太少，并且很多地方容易误解读者。
如果直接用这个去面试，我只能说很难。
关于setState直接是异步，肯定过不了。
setState需要分在哪里调用。并且setState本身内部代码实现也不是异步代码
只能说是“表现的是异步” ，只是因为在“表现为异步”的时候内部调用顺序是在更新之前，所以看起来像是异步
1 setState只在合成函数和钩子函数中表现的是异步。
2 在原声事件，比如addEventListener添加的事件和setTimeOut中是同步。

展开

10

回复

[![小kian的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3491704658999661)

[小kian![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3491704658999661)

前端工程师 @ 涂鸦智能1年前

react这边实在是太少，并且很多地方容易误解读者。
如果直接用这个去面试，我只能说很难。
关于setState直接是异步，肯定过不了。
setState需要分在哪里调用。并且setState本身内部代码实现也不是异步代码
只能说是“表现的是异步” ，只是因为在“表现为异步”的时候内部调用顺序是在更新之前，所以看起来像是异步
1 setState只在合成函数和钩子函数中表现的是异步。
2 在原声事件，比如addEventListener添加的事件和setTimeOut中是同步。

展开

6

回复

[![Soliton的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1292681404751480)

[Soliton![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1292681404751480)

前端 @ 前肝工程师1年前

setState严格来说应该算同步的吧，说是异步，只是说是将要执行的结果通过isBatchingUpdates来判断是否返回

2

回复

[![Soliton的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1292681404751480)

[Soliton![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1292681404751480)

前端 @ 前肝工程师1年前

setState严格来说应该算同步的吧，说是异步，只是说是将要执行的结果通过isBatchingUpdates来判断是否返回

点赞

回复

[![不会喷火的燚龘的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1961184475512046)

[不会喷火的燚龘![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1961184475512046)

前端工程师 @ 字节跳动2年前

setState是异步的么 那setTimeout里怎么说

1

5

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2928754707404141)

[前端修仙记](https://juejin.cn/user/2928754707404141)

1年前

同步啊，那还能怎么说

点赞

回复

[![img](https://p3-passport.byteacctimg.com/img/user-avatar/ec95015d5a5d12dbebab07ee2f909438~300x300.image)](https://juejin.cn/user/782508009733528)

[坨子捏到帮紧](https://juejin.cn/user/782508009733528)

1年前

又有同步 又有异步

点赞

回复

查看更多回复

[![不会喷火的燚龘的头像](https://p9-passport.byteacctimg.com/img/user-avatar/a9bc68ed42dbffbddf7b03872ac1f631~300x300.image)](https://juejin.cn/user/1961184475512046)

[不会喷火的燚龘![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1961184475512046)

前端工程师 @ 字节跳动2年前

setState是异步的么 那setTimeout里怎么说

点赞

4

[![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/3/29/1712550ae2cd6ccb~tplv-t2oaga2asx-no-mark:24:24:24:24.awebp)](https://juejin.cn/user/2928754707404141)

[前端修仙记](https://juejin.cn/user/2928754707404141)

1年前

同步啊，那还能怎么说

点赞

回复

[![img](https://p3-passport.byteacctimg.com/img/user-avatar/ec95015d5a5d12dbebab07ee2f909438~300x300.image)](https://juejin.cn/user/782508009733528)

[坨子捏到帮紧](https://juejin.cn/user/782508009733528)

1年前

又有同步 又有异步

点赞

回复

查看更多回复

[![洛河桥盼的头像](https://p3-passport.byteacctimg.com/img/user-avatar/c47cdc5c7f04db44b6274111992ac7dc~300x300.image)](https://juejin.cn/user/2788017218528216)

[洛河桥盼](https://juejin.cn/user/2788017218528216)

前端 CV 小能手2年前

好像反馈里面都是在说 React 内容好少，我其实也是这么觉得的，希望作者大大看到之后能理解一下 QAQ

4

回复

[![洛河桥盼的头像](https://p3-passport.byteacctimg.com/img/user-avatar/c47cdc5c7f04db44b6274111992ac7dc~300x300.image)](https://juejin.cn/user/2788017218528216)

[洛河桥盼](https://juejin.cn/user/2788017218528216)

前端 CV 小能手2年前

好像反馈里面都是在说 React 内容好少，我其实也是这么觉得的，希望作者大大看到之后能理解一下 QAQ

点赞

回复

[![夏雨雪的头像](https://p3-passport.byteacctimg.com/img/user-avatar/e2fc2816d6d86380b1a65dc3383439bf~300x300.image)](https://juejin.cn/user/3790771823586088)

[夏雨雪](https://juejin.cn/user/3790771823586088)

前端3年前

“以下三种情况”，应该是“四种”。另外Object.assign，后面多一个w

点赞

回复

[![夏雨雪的头像](https://p3-passport.byteacctimg.com/img/user-avatar/e2fc2816d6d86380b1a65dc3383439bf~300x300.image)](https://juejin.cn/user/3790771823586088)

[夏雨雪](https://juejin.cn/user/3790771823586088)

前端3年前

“以下三种情况”，应该是“四种”。另外Object.assign，后面多一个w

点赞

回复

[![O滴K的头像](https://p26-passport.byteacctimg.com/img/mosaic-legacy/3793/3114521287~300x300.image)](https://juejin.cn/user/3808364010679415)

[O滴K](https://juejin.cn/user/3808364010679415)

3年前

React就这么点？

1

回复

[![O滴K的头像](https://p26-passport.byteacctimg.com/img/mosaic-legacy/3793/3114521287~300x300.image)](https://juejin.cn/user/3808364010679415)

[O滴K](https://juejin.cn/user/3808364010679415)

3年前

React就这么点？

点赞

回复

[![return false33442的头像](https://p6-passport.byteacctimg.com/img/user-avatar/dbddd7c392ba7d95b2fb41750e82a074~300x300.image)](https://juejin.cn/user/307518985474349)

[return false33442](https://juejin.cn/user/307518985474349)

3年前

react 这两篇讲的太简单了，很多东西都没讲到

点赞

回复

查看全部 84 条回复