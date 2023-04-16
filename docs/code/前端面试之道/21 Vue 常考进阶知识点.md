# Vue 常考进阶知识点

这一章节我们将来学习 Vue 的一些经常考到的进阶知识点。这些知识点相对而言理解起来会很有难度，可能需要多次阅读才能理解。

## 响应式原理

Vue 内部使用了 `Object.defineProperty()` 来实现数据响应式，通过这个函数可以监听到 `set` 和 `get` 的事件。

```
var data = { name: 'yck' }
observe(data)
let name = data.name // -> get value
data.name = 'yyy' // -> change value

function observe(obj) {
  // 判断类型
  if (!obj || typeof obj !== 'object') {
    return
  }
  Object.keys(obj).forEach(key => {
    defineReactive(obj, key, obj[key])
  })
}

function defineReactive(obj, key, val) {
  // 递归子属性
  observe(val)
  Object.defineProperty(obj, key, {
    // 可枚举
    enumerable: true,
    // 可配置
    configurable: true,
    // 自定义函数
    get: function reactiveGetter() {
      console.log('get value')
      return val
    },
    set: function reactiveSetter(newVal) {
      console.log('change value')
      val = newVal
    }
  })
}
```

以上代码简单的实现了如何监听数据的 `set` 和 `get` 的事件，但是仅仅如此是不够的，因为自定义的函数一开始是不会执行的。只有先执行了依赖收集，才能在属性更新的时候派发更新，所以接下来我们需要先触发依赖收集。

```
<div>
    {{name}}
</div>
```

在解析如上模板代码时，遇到 `{{name}}` 就会进行依赖收集。

接下来我们先来实现一个 `Dep` 类，用于解耦属性的依赖收集和派发更新操作。

```
// 通过 Dep 解耦属性的依赖和更新操作
class Dep {
  constructor() {
    this.subs = []
  }
  // 添加依赖
  addSub(sub) {
    this.subs.push(sub)
  }
  // 更新
  notify() {
    this.subs.forEach(sub => {
      sub.update()
    })
  }
}
// 全局属性，通过该属性配置 Watcher
Dep.target = null
```

以上的代码实现很简单，当需要依赖收集的时候调用 `addSub`，当需要派发更新的时候调用 `notify`。

接下来我们先来简单的了解下 Vue 组件挂载时添加响应式的过程。在组件挂载时，会先对所有需要的属性调用 `Object.defineProperty()`，然后实例化 `Watcher`，传入组件更新的回调。在实例化过程中，会对模板中的属性进行求值，触发依赖收集。

因为这一小节主要目的是学习响应式原理的细节，所以接下来的代码会简略的表达触发依赖收集时的操作。

```
class Watcher {
  constructor(obj, key, cb) {
    // 将 Dep.target 指向自己
    // 然后触发属性的 getter 添加监听
    // 最后将 Dep.target 置空
    Dep.target = this
    this.cb = cb
    this.obj = obj
    this.key = key
    this.value = obj[key]
    Dep.target = null
  }
  update() {
    // 获得新值
    this.value = this.obj[this.key]
    // 调用 update 方法更新 Dom
    this.cb(this.value)
  }
}
```

以上就是 `Watcher` 的简单实现，在执行构造函数的时候将 `Dep.target` 指向自身，从而使得收集到了对应的 `Watcher`，在派发更新的时候取出对应的 `Watcher` 然后执行 `update` 函数。

接下来，需要对 `defineReactive` 函数进行改造，在自定义函数中添加依赖收集和派发更新相关的代码。

```
function defineReactive(obj, key, val) {
  // 递归子属性
  observe(val)
  let dp = new Dep()
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      console.log('get value')
      // 将 Watcher 添加到订阅
      if (Dep.target) {
        dp.addSub(Dep.target)
      }
      return val
    },
    set: function reactiveSetter(newVal) {
      console.log('change value')
      val = newVal
      // 执行 watcher 的 update 方法
      dp.notify()
    }
  })
}
```

以上所有代码实现了一个简易的数据响应式，核心思路就是手动触发一次属性的 getter 来实现依赖收集。

现在我们就来测试下代码的效果，只需要把所有的代码复制到浏览器中执行，就会发现页面的内容全部被替换了。

```
var data = { name: 'yck' }
observe(data)
function update(value) {
  document.querySelector('div').innerText = value
}
// 模拟解析到 `{{name}}` 触发的操作
new Watcher(data, 'name', update)
// update Dom innerText
data.name = 'yyy' 
```

### Object.defineProperty 的缺陷

以上已经分析完了 Vue 的响应式原理，接下来说一点 `Object.defineProperty` 中的缺陷。

如果通过下标方式修改数组数据或者给对象新增属性并不会触发组件的重新渲染，因为 `Object.defineProperty` 不能拦截到这些操作，更精确的来说，对于数组而言，大部分操作都是拦截不到的，只是 Vue 内部通过重写函数的方式解决了这个问题。

对于第一个问题，Vue 提供了一个 API 解决

```
export function set (target: Array<any> | Object, key: any, val: any): any {
  // 判断是否为数组且下标是否有效
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    // 调用 splice 函数触发派发更新
    // 该函数已被重写
    target.length = Math.max(target.length, key)
    target.splice(key, 1, val)
    return val
  }
  // 判断 key 是否已经存在
  if (key in target && !(key in Object.prototype)) {
    target[key] = val
    return val
  }
  const ob = (target: any).__ob__
  // 如果对象不是响应式对象，就赋值返回
  if (!ob) {
    target[key] = val
    return val
  }
  // 进行双向绑定
  defineReactive(ob.value, key, val)
  // 手动派发更新
  ob.dep.notify()
  return val
}
```

对于数组而言，Vue 内部重写了以下函数实现派发更新

```
// 获得数组原型
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)
// 重写以下函数
const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
methodsToPatch.forEach(function (method) {
  // 缓存原生函数
  const original = arrayProto[method]
  // 重写函数
  def(arrayMethods, method, function mutator (...args) {
  // 先调用原生函数获得结果
    const result = original.apply(this, args)
    const ob = this.__ob__
    let inserted
    // 调用以下几个函数时，监听新数据
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // 手动派发更新
    ob.dep.notify()
    return result
  })
})
```

## 编译过程

想必大家在使用 Vue 开发的过程中，基本都是使用模板的方式。那么你有过「模板是怎么在浏览器中运行的」这种疑虑嘛？

首先直接把模板丢到浏览器中肯定是不能运行的，模板只是为了方便开发者进行开发。Vue 会通过编译器将模板通过几个阶段最终编译为 `render` 函数，然后通过执行 `render` 函数生成 Virtual DOM 最终映射为真实 DOM。

接下来我们就来学习这个编译的过程，了解这个过程中大概发生了什么事情。这个过程其中又分为三个阶段，分别为：

1. 将模板解析为 AST
2. 优化 AST
3. 将 AST 转换为 `render` 函数

在第一个阶段中，最主要的事情还是通过各种各样的正则表达式去匹配模板中的内容，然后将内容提取出来做各种逻辑操作，接下来会生成一个最基本的 AST 对象

```
{
    // 类型
    type: 1,
    // 标签
    tag,
    // 属性列表
    attrsList: attrs,
    // 属性映射
    attrsMap: makeAttrsMap(attrs),
    // 父节点
    parent,
    // 子节点
    children: []
}
```

然后会根据这个最基本的 AST 对象中的属性，进一步扩展 AST。

当然在这一阶段中，还会进行其他的一些判断逻辑。比如说对比前后开闭标签是否一致，判断根组件是否只存在一个，判断是否符合 HTML5 [Content Model](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FGuide%2FHTML%2FContent_categories) 规范等等问题。

接下来就是优化 AST 的阶段。在当前版本下，Vue 进行的优化内容其实还是不多的。只是对节点进行了静态内容提取，也就是将永远不会变动的节点提取了出来，实现复用 Virtual DOM，跳过对比算法的功能。在下一个大版本中，Vue 会在优化 AST 的阶段继续发力，实现更多的优化功能，尽可能的在编译阶段压榨更多的性能，比如说提取静态的属性等等优化行为。

最后一个阶段就是通过 AST 生成 `render` 函数了。其实这一阶段虽然分支有很多，但是最主要的目的就是遍历整个 AST，根据不同的条件生成不同的代码罢了。

## NextTick 原理分析

`nextTick` 可以让我们在下次 DOM 更新循环结束之后执行延迟回调，用于获得更新后的 DOM。

在 Vue 2.4 之前都是使用的 microtasks，但是 microtasks 的优先级过高，在某些情况下可能会出现比事件冒泡更快的情况，但如果都使用 macrotasks 又可能会出现渲染的性能问题。所以在新版本中，会默认使用 microtasks，但在特殊情况下会使用 macrotasks，比如 v-on。

对于实现 macrotasks ，会先判断是否能使用 `setImmediate` ，不能的话降级为 `MessageChannel` ，以上都不行的话就使用 `setTimeout`

```
if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  macroTimerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else if (
  typeof MessageChannel !== 'undefined' &&
  (isNative(MessageChannel) ||
    // PhantomJS
    MessageChannel.toString() === '[object MessageChannelConstructor]')
) {
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = flushCallbacks
  macroTimerFunc = () => {
    port.postMessage(1)
  }
} else {
  macroTimerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```

以上代码很简单，就是判断能不能使用相应的 API。

## 小结

以上就是 Vue 的几个高频核心问题了，如果你还想了解更多的源码相关的细节，强烈推荐黄老师的 [Vue 技术揭秘](https://link.juejin.cn/?target=https%3A%2F%2Fustbhuangyi.github.io%2Fvue-analysis%2F)。

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（48）

[![醉逍遥neo的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3685218706795368)

[醉逍遥neo![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3685218706795368)

^-^1年前

nextTick原理却讲了一堆异步队列，有啥关系呢，最关键的点应该是怎么知道dom更新完了

点赞

回复

[![CoderYd的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3913917126684398)

[CoderYd](https://juejin.cn/user/3913917126684398)

前端1年前

买完有点后悔了

10

回复

[![有梦想的前端切图仔的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3597257778138808)

[有梦想的前端切图仔![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3597257778138808)

1年前

太水了

6

回复

[![Wishzhang17033的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2313028193485566)

[Wishzhang17033![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2313028193485566)

前端开发工程师1年前

怎么没有diff算法的讲解

2

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/694547080290701)

[喜欢蓝色大海🌊](https://juejin.cn/user/694547080290701)

3月前

上章讲了

点赞

回复

[![toored的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/4353721775966094)

[toored](https://juejin.cn/user/4353721775966094)

1年前

Vue 的 set api，下面这段如何理解呢，__ob__指的是什么
const ob = (target: any).__ob__
 // 如果对象不是响应式对象，就赋值返回
 if (!ob) {
 target[key] = val
 return val
 }
 // 进行双向绑定
 defineReactive(ob.value, key, val)
 // 手动派发更新
 ob.dep.notify()
 return val

展开

点赞

回复

[![难得糊涂wxt的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1838039172680872)

[难得糊涂wxt](https://juejin.cn/user/1838039172680872)

web前端1年前

原理部分和源码有很大出入，还是推荐大家直接看源码，ast nexttick都不对，

2

1

[![img](https://p6-passport.byteacctimg.com/img/user-avatar/7b38bce1559fad25472c1794c8ca6002~300x300.image)](https://juejin.cn/user/2541726613638973)

[光光同学![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/2541726613638973)

10月前

我看了，应该是一个Promise吧

点赞

回复

[![明天下雨不想说话的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1574156381728877)

[明天下雨不想说话](https://juejin.cn/user/1574156381728877)

1年前

nextTick 的原理分析 j讲的太敷衍了把

2

回复

[![Harvey_wang的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/395479914660583)

[Harvey_wang](https://juejin.cn/user/395479914660583)

前端工程师2年前

这个 nextTick 的原理分析基本就只讲了一点。。。没有基础的话根本看不懂

3

2

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/272334611295389)

[fatbin![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/272334611295389)

2年前

基础不就在第七章的Event Loop里讲过了吗。。

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2330620350432110)

[JowayYoung![lv-6](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/74bd93adef7feff4fee26d08c0845b4f.svg)](https://juejin.cn/user/2330620350432110)

2年前

Event Loop 有说过了

点赞

回复

[![大亮子的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1363050147618039)

[大亮子![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1363050147618039)

web前端开发工程师 @ 中农网2年前

v-model原理 这个不热门吗

点赞

1

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/a386aa8db73c9678458ec34161472ca5~300x300.image)](https://juejin.cn/user/712139233840407)

[yck![lv-7](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/7a2d00014351211140dd9bc0ba3afd8c.svg)](https://juejin.cn/user/712139233840407)

（作者）2年前

上一章节就讲过了

点赞

回复

[![周序猿的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/4494459261689335)

[周序猿](https://juejin.cn/user/4494459261689335)

辣鸡程序员 @ 18线小公司3年前

请问下什么时候 microtasks 比事件冒泡更快

点赞

2

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1134351731138184)

[Shoryuken![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1134351731138184)

2年前

这俩货有啥关系， 请问？

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/4125023359474574)

[lxgmwauv](https://juejin.cn/user/4125023359474574)

2年前

执行完script以后，去执行microtasks，这个时候事件有可能还没有触发，参考下这个里面说的UI Rendering是宏任务，但是和微任务基本上是平行处理的说法。
[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)juejin.im](https://juejin.im/post/5b73d7a6518825610072b42b)

点赞

回复

[![砖用冰西瓜的头像](https://p26-passport.byteacctimg.com/img/user-avatar/dee36188b78703b831bb042a0b17b593~300x300.image)](https://juejin.cn/user/1714893869549031)

[砖用冰西瓜![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/1714893869549031)

3年前

有了AST，为什么还要VDOM？他们有什么区别？

点赞

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3139860938108398)

[stone_10493![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/3139860938108398)

2年前

AST 只是抽象语法树，并不能描述 DOM 结构

点赞

回复

[![小彩机的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3227821867543838)

[小彩机](https://juejin.cn/user/3227821867543838)

3年前

constructor(obj, key, cb) { // 将 Dep.target 指向自己 // 然后触发属性的 getter 添加监听 // 最后将 Dep.target 置空 Dep.target = this this.cb = cb this.obj = obj this.key = key this.value = obj[key] Dep.target = null } 这里完全看不懂 有看懂的么

点赞

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3227821867543838)

[小彩机](https://juejin.cn/user/3227821867543838)

3年前

我看懂了

点赞

回复

[![DiegoGuo的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/leancloud-assets/677zSXzxA1MjdUzyVZrETpAAaKiSocjM8Lh9rlEB~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/764915821318446)

[DiegoGuo](https://juejin.cn/user/764915821318446)

前端开发工程师 @ 高德3年前

优秀

点赞

回复

[![光光之歌的头像](https://p26-passport.byteacctimg.com/img/user-avatar/f39a7947d3a655dd6e1baa586904fac7~300x300.image)](https://juejin.cn/user/1943592286831047)

[光光之歌![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/1943592286831047)

无3年前

Dep.target 指向自身，从而使得收集到了对应的 Watcher，是怎么触发这个依赖收集的，你这一步没有在源码体现呀

点赞

5

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3650034335234590)

[sayid95](https://juejin.cn/user/3650034335234590)

3年前

new watcher的时候啊

点赞

回复

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/f39a7947d3a655dd6e1baa586904fac7~300x300.image)](https://juejin.cn/user/1943592286831047)

[光光之歌![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/1943592286831047)

回复

[sayid95](https://juejin.cn/user/3650034335234590)

3年前

new watcher只是把订阅者push 进消息订阅器，解析模板收集依赖压根没有分析，同学建议你再看一遍

“

new watcher的时候啊

”

点赞

回复

查看更多回复

[![马克龙的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/26/171b533d25770887~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/448256474611885)

[马克龙![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/448256474611885)

前端开发3年前

defineReactive(ob.value, key, val) vue 内部改变那里 不是defineReactive(ob, key, val)这样吗

点赞

回复

[![努努的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3034307822105016)

[努努](https://juejin.cn/user/3034307822105016)

3年前

就是从依赖收集那里开始看不懂了，太笼统了

1

回复

[![努努的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3034307822105016)

[努努](https://juejin.cn/user/3034307822105016)

3年前

看不太懂

1

回复

[![努努的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3034307822105016)

[努努](https://juejin.cn/user/3034307822105016)

3年前

看不懂

3

回复

[![飞雁的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/18/1685ea940dd7fe60~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/1151943916662814)

[飞雁](https://juejin.cn/user/1151943916662814)

前端螺丝钉3年前

‘吹毛求疵’一下。“只有先执行了依赖收集，从能在属性更新的时候派发更新，所以接下来我们需要先触发依赖收集。”这里的‘从’应该是‘才’吧

点赞

1

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/a386aa8db73c9678458ec34161472ca5~300x300.image)](https://juejin.cn/user/712139233840407)

[yck![lv-7](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/7a2d00014351211140dd9bc0ba3afd8c.svg)](https://juejin.cn/user/712139233840407)

（作者）3年前

恩

点赞

回复

[![伱是风ル的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/3/26/169b8c02cb5acd14~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/3702810891791134)

[伱是风ル](https://juejin.cn/user/3702810891791134)

前端 @ 嘿嘿嘿3年前

cb 是更新 vdom里的值吗

点赞

回复

查看全部 48 条回复