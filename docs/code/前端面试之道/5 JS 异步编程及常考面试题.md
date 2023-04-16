# JS 异步编程及常考面试题

在上一章节中我们了解了常见 ES6 语法的一些知识点。这一章节我们将会学习异步编程这一块的内容，鉴于异步编程是 JS 中至关重要的内容，所以我们将会用三个章节来学习异步编程涉及到的重点和难点，同时这一块内容也是面试常考范围，希望大家认真学习。

## 并发（concurrency）和并行（parallelism）区别

```!
涉及面试题：并发与并行的区别？
```

异步和这小节的知识点其实并不是一个概念，但是这两个名词确实是很多人都常会混淆的知识点。其实混淆的原因可能只是两个名词在中文上的相似，在英文上来说完全是不同的单词。

并发是宏观概念，我分别有任务 A 和任务 B，在一段时间内通过任务间的切换完成了这两个任务，这种情况就可以称之为并发。

并行是微观概念，假设 CPU 中存在两个核心，那么我就可以同时完成任务 A、B。同时完成多个任务的情况就可以称之为并行。

## 回调函数（Callback）

```!
涉及面试题：什么是回调函数？回调函数有什么缺点？如何解决回调地狱问题？
```

回调函数应该是大家经常使用到的，以下代码就是一个回调函数的例子：

```js
ajax(url, () => {
    // 处理逻辑
})
```

但是回调函数有一个致命的弱点，就是容易写出回调地狱（Callback hell）。假设多个请求存在依赖性，你可能就会写出如下代码：

```js
ajax(url, () => {
    // 处理逻辑
    ajax(url1, () => {
        // 处理逻辑
        ajax(url2, () => {
            // 处理逻辑
        })
    })
})
```

以上代码看起来不利于阅读和维护，当然，你可能会想说解决这个问题还不简单，把函数分开来写不就得了

```js
function firstAjax() {
  ajax(url1, () => {
    // 处理逻辑
    secondAjax()
  })
}
function secondAjax() {
  ajax(url2, () => {
    // 处理逻辑
  })
}
ajax(url, () => {
  // 处理逻辑
  firstAjax()
})
```

以上的代码虽然看上去利于阅读了，但是还是没有解决根本问题。

回调地狱的根本问题就是：

1. 嵌套函数存在耦合性，一旦有所改动，就会牵一发而动全身
2. 嵌套函数一多，就很难处理错误

当然，回调函数还存在着别的几个缺点，比如不能使用 `try catch` 捕获错误，不能直接 `return`。在接下来的几小节中，我们将来学习通过别的技术解决这些问题。

## Generator

```!
涉及面试题：你理解的 Generator 是什么？
```

`Generator` 算是 ES6 中难理解的概念之一了，`Generator` 最大的特点就是可以控制函数的执行。在这一小节中我们不会去讲什么是 `Generator`，而是把重点放在 `Generator` 的一些容易困惑的地方。

```js
function *foo(x) {
  let y = 2 * (yield (x + 1))
  let z = yield (y / 3)
  return (x + y + z)
}
let it = foo(5)
console.log(it.next())   // => {value: 6, done: false}
console.log(it.next(12)) // => {value: 8, done: false}
console.log(it.next(13)) // => {value: 42, done: true}
```

你也许会疑惑为什么会产生与你预想不同的值，接下来就让我为你逐行代码分析原因

- 首先 `Generator` 函数调用和普通函数不同，它会返回一个迭代器
- 当执行第一次 `next` 时，传参会被忽略，并且函数暂停在 `yield (x + 1)` 处，所以返回 `5 + 1 = 6`
- 当执行第二次 `next` 时，传入的参数等于上一个 `yield` 的返回值，如果你不传参，`yield` 永远返回 `undefined`。此时 `let y = 2 * 12`，所以第二个 `yield` 等于 `2 * 12 / 3 = 8`
- 当执行第三次 `next` 时，传入的参数会传递给 `z`，所以 `z = 13, x = 5, y = 24`，相加等于 `42`

`Generator` 函数一般见到的不多，其实也于他有点绕有关系，并且一般会配合 co 库去使用。当然，我们可以通过 `Generator` 函数解决回调地狱的问题，可以把之前的回调地狱例子改写为如下代码：

```js
function *fetch() {
    yield ajax(url, () => {})
    yield ajax(url1, () => {})
    yield ajax(url2, () => {})
}
let it = fetch()
let result1 = it.next()
let result2 = it.next()
let result3 = it.next()
```

## Promise

```!
涉及面试题：Promise 的特点是什么，分别有什么优缺点？什么是 Promise 链？Promise 构造函数执行和 then 函数执行有什么区别？
```

`Promise` 翻译过来就是承诺的意思，这个承诺会在未来有一个确切的答复，并且该承诺有三种状态，分别是：

1. 等待中（pending）
2. 完成了 （resolved）
3. 拒绝了（rejected）

这个承诺一旦从等待状态变成为其他状态就永远不能更改状态了，也就是说一旦状态变为 resolved 后，就不能再次改变

```
new Promise((resolve, reject) => {
  resolve('success')
  // 无效
  reject('reject')
})
```

当我们在构造 `Promise` 的时候，构造函数内部的代码是立即执行的

```js
new Promise((resolve, reject) => {
  console.log('new Promise')
  resolve('success')
})
console.log('finifsh')
// new Promise -> finifsh
```

`Promise` 实现了链式调用，也就是说每次调用 `then` 之后返回的都是一个 `Promise`，并且是一个全新的 `Promise`，原因也是因为状态不可变。如果你在 `then` 中 使用了 `return`，那么 `return` 的值会被 `Promise.resolve()` 包装

```js
Promise.resolve(1)
  .then(res => {
    console.log(res) // => 1
    return 2 // 包装成 Promise.resolve(2)
  })
  .then(res => {
    console.log(res) // => 2
  })
```

当然了，`Promise` 也很好地解决了回调地狱的问题，可以把之前的回调地狱例子改写为如下代码：

```js
ajax(url)
  .then(res => {
      console.log(res)
      return ajax(url1)
  }).then(res => {
      console.log(res)
      return ajax(url2)
  }).then(res => console.log(res))
```

前面都是在讲述 `Promise` 的一些优点和特点，其实它也是存在一些缺点的，比如无法取消 `Promise`，错误需要通过回调函数捕获。

## async 及 await

```!
涉及面试题：async 及 await 的特点，它们的优点和缺点分别是什么？await 原理是什么？
```

一个函数如果加上 `async` ，那么该函数就会返回一个 `Promise`

```js
async function test() {
  return "1"
}
console.log(test()) // -> Promise {<resolved>: "1"}
```

`async` 就是将函数返回值使用 `Promise.resolve()` 包裹了下，和 `then` 中处理返回值一样，并且 `await` 只能配套 `async` 使用

```js
async function test() {
  let value = await sleep()
}
```

`async` 和 `await` 可以说是异步终极解决方案了，相比直接使用 `Promise` 来说，优势在于处理 `then` 的调用链，能够更清晰准确的写出代码，毕竟写一大堆 `then` 也很恶心，并且也能优雅地解决回调地狱问题。当然也存在一些缺点，因为 `await` 将异步代码改造成了同步代码，如果多个异步代码没有依赖性却使用了 `await` 会导致性能上的降低。

```js
async function test() {
  // 以下代码没有依赖性的话，完全可以使用 Promise.all 的方式
  // 如果有依赖性的话，其实就是解决回调地狱的例子了
  await fetch(url)
  await fetch(url1)
  await fetch(url2)
}
```

下面来看一个使用 `await` 的例子：

```js
let a = 0
let b = async () => {
  a = a + await 10
  console.log('2', a) // -> '2' 10
}
b()
a++
console.log('1', a) // -> '1' 1
```

对于以上代码你可能会有疑惑，让我来解释下原因

- 首先函数 `b` 先执行，在执行到 `await 10` 之前变量 `a` 还是 0，因为 `await` 内部实现了 `generator` ，`generator` 会保留堆栈中东西，所以这时候 `a = 0` 被保存了下来
- 因为 `await` 是异步操作，后来的表达式不返回 `Promise` 的话，就会包装成 `Promise.reslove(返回值)`，然后会去执行函数外的同步代码
- 同步代码执行完毕后开始执行异步代码，将保存下来的值拿出来使用，这时候 `a = 0 + 10`

上述解释中提到了 `await` 内部实现了 `generator`，其实 `await` 就是 `generator` 加上 `Promise` 的语法糖，且内部实现了自动执行 `generator`。如果你熟悉 co 的话，其实自己就可以实现这样的语法糖。

## 常用定时器函数

```!
涉及面试题：setTimeout、setInterval、requestAnimationFrame 各有什么特点？
```

异步编程当然少不了定时器了，常见的定时器函数有 `setTimeout`、`setInterval`、`requestAnimationFrame`。我们先来讲讲最常用的`setTimeout`，很多人认为 `setTimeout` 是延时多久，那就应该是多久后执行。

其实这个观点是错误的，因为 JS 是单线程执行的，如果前面的代码影响了性能，就会导致 `setTimeout` 不会按期执行。当然了，我们可以通过代码去修正 `setTimeout`，从而使定时器相对准确

```js
let period = 60 * 1000 * 60 * 2
let startTime = new Date().getTime()
let count = 0
let end = new Date().getTime() + period
let interval = 1000
let currentInterval = interval

function loop() {
  count++
  // 代码执行所消耗的时间
  let offset = new Date().getTime() - (startTime + count * interval);
  let diff = end - new Date().getTime()
  let h = Math.floor(diff / (60 * 1000 * 60))
  let hdiff = diff % (60 * 1000 * 60)
  let m = Math.floor(hdiff / (60 * 1000))
  let mdiff = hdiff % (60 * 1000)
  let s = mdiff / (1000)
  let sCeil = Math.ceil(s)
  let sFloor = Math.floor(s)
  // 得到下一次循环所消耗的时间
  currentInterval = interval - offset 
  console.log('时：'+h, '分：'+m, '毫秒：'+s, '秒向上取整：'+sCeil, '代码执行时间：'+offset, '下次循环间隔'+currentInterval) 

  setTimeout(loop, currentInterval)
}

setTimeout(loop, currentInterval)
```

接下来我们来看 `setInterval`，其实这个函数作用和 `setTimeout` 基本一致，只是该函数是每隔一段时间执行一次回调函数。

通常来说不建议使用 `setInterval`。第一，它和 `setTimeout` 一样，不能保证在预期的时间执行任务。第二，它存在执行累积的问题，请看以下伪代码

```js
function demo() {
  setInterval(function(){
    console.log(2)
  },1000)
  sleep(2000)
}
demo()
```

以上代码在浏览器环境中，如果定时器执行过程中出现了耗时操作，多个回调函数会在耗时操作结束以后同时执行，这样可能就会带来性能上的问题。

如果你有循环定时器的需求，其实完全可以通过 `requestAnimationFrame` 来实现

```js
function setInterval(callback, interval) {
  let timer
  const now = Date.now
  let startTime = now()
  let endTime = startTime
  const loop = () => {
    timer = window.requestAnimationFrame(loop)
    endTime = now()
    if (endTime - startTime >= interval) {
      startTime = endTime = now()
      callback(timer)
    }
  }
  timer = window.requestAnimationFrame(loop)
  return timer
}

let a = 0
setInterval(timer => {
  console.log(1)
  a++
  if (a === 3) cancelAnimationFrame(timer)
}, 1000)
```

首先 `requestAnimationFrame` 自带函数节流功能，基本可以保证在 16.6 毫秒内只执行一次（不掉帧的情况下），并且该函数的延时效果是精确的，没有其他定时器时间不准的问题，当然你也可以通过该函数来实现 `setTimeout`。

## 小结

异步编程是 JS 中较难掌握的内容，同时也是很重要的知识点。以上提到的每个知识点其实都可以作为一道面试题，希望大家可以好好掌握以上内容如果大家对于这个章节的内容存在疑问，欢迎在评论区与我互动。

```!
异步编程相关内容并非一章节就能讲完，需要继续浏览后续章节。
```

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（102）

[![尤雨硒的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3316570113968846)

[尤雨硒![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3316570113968846)

抖腿工程师 @ 自己跳动8天前

rnm退钱，概念都写的有问题

2

回复

[![爱敲代码的头像](https://p6-passport.byteacctimg.com/img/user-avatar/5fa8c6155fee5a3a0ee29b4e1d7302cc~300x300.image)](https://juejin.cn/user/4353721778045357)

[爱敲代码](https://juejin.cn/user/4353721778045357)

1年前

无法取消 Promise 能给个具体案例吗？

1

回复

[![hopekayo14108的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/870468937583006)

[hopekayo14108](https://juejin.cn/user/870468937583006)

1年前

Generator![👍](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f44d.svg~tplv-t2oaga2asx-image.image)

点赞

回复

[![PercyA的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1574156383828151)

[PercyA](https://juejin.cn/user/1574156383828151)

CV学徒 @ CV1年前

Generator![👍](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f44d.svg~tplv-t2oaga2asx-image.image)

点赞

回复

[![你若像风的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2277843823766967)

[你若像风![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2277843823766967)

前端开发工程师1年前

requestAnimationFrame这个api日常工作中用得多吗？秒杀活动之类的的定时器一般用什么？

点赞

2

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/993614243047991)

[xiaoxiaoo![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/993614243047991)

1年前

提高动画流畅性会用它，它的执行频率在动画上看不出来，秒杀活动的倒计时定时器，一般用的是settimeinterval

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1697301682722510)

[Rabbitzzc![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/1697301682722510)

1年前

动画中会经常用到的，不然动画很不稳定

点赞

回复

[![走进科学爱学习的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/mirror-assets/168e085a05365b84f86~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/3245414054901735)

[走进科学爱学习](https://juejin.cn/user/3245414054901735)

1年前

callback的信任问题比回调地域更严重吧

4

回复

[![JJBoom的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/729731449506983)

[JJBoom![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/729731449506983)

1年前

测试发现，setInterval 并没有累计的问题，是浏览器版本的问题吗？

1

回复

[![千寻君34879的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1855631357903096)

[千寻君34879](https://juejin.cn/user/1855631357903096)

2年前

作为查漏补缺的小册子很好

6

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2277843823766967)

[你若像风![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2277843823766967)

1年前

我也是这么觉得的，查缺补漏，作者有些地方讲得稍微浅的需要自己查资料，查资料也是一个加深印象的过程呀。况且有些知识点作者总结得还是很好的。我觉得这个价格买很划算了。

点赞

回复

[![画饼师的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/184373686046110)

[画饼师![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/184373686046110)

2年前

“因为 await 将异步代码改造成了同步代码，如果多个异步代码没有依赖性却使用了 await 会导致性能上的降低。“ 可是await本来就可以await Promise.all()啊 这怎么能算的上缺点的，太牵强了。

1

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3685218704698014)

[koromon](https://juejin.cn/user/3685218704698014)

2年前

Promise.all里面都是异步执行，多个 await 要等前面的执行完才下一个，还是不一样的把

1

回复

[![小迷的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/4494459266406813)

[小迷![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/4494459266406813)

前端2年前

requestAnimationFrame这部分有点儿问题，代码我运行了一遍是错的啊

点赞

回复

[![爱码士不想说话的头像](https://p6-passport.byteacctimg.com/img/user-avatar/76d3434535832af18cf833c4a00b03cc~300x300.image)](https://juejin.cn/user/413072103052477)

[爱码士不想说话](https://juejin.cn/user/413072103052477)

2年前

requestAnimationFrame 这个不就更不准了吗…，每次都有 16ms 以内的误差；用setTimeout链式确实可以解决累加问题，但你修正了一下，累加反而变得更严重了啊，你的修正一旦超过间隔时间咋办，间隔一秒，有一次卡了三秒，用次数控制会立刻执行三次，setInterval反而只会立刻执行两次……

12

回复

[![sophie旭的头像](https://p3-passport.byteacctimg.com/img/user-avatar/3e4e087e89cca513abc5a13bd10a301d~300x300.image)](https://juejin.cn/user/2559318799692952)

[sophie旭](https://juejin.cn/user/2559318799692952)

2年前

定时器文章：[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)palmer.arkstack.cn](https://link.juejin.cn/?target=https%3A%2F%2Fpalmer.arkstack.cn%2F2017%2F12%2F%E4%BB%8EsetTimeout-setInterval%E7%9C%8BJS%E7%BA%BF%E7%A8%8B%2F)

4

回复

[![萱萱的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3825956194892519)

[萱萱![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3825956194892519)

软件开发工程师2年前

操作系统没好好学？同一时间间隔发生的是并发，同时发生的是并行，都并行了还怎么单核？作者说的没错啊

5

3

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2752832847490557)

[屁儿快跑](https://juejin.cn/user/2752832847490557)

2年前

1.你的'同一时间间隔发生的是并发' ,你的意思是同时开始,同时结束么? 在java中多线程编程中,线程并发可不是同时开始,同时结束,实际上是一个线程还未结束,就开始另外一个线程.所以说如果你说的是 同一时间间隔是不恰当的,同一时刻有多个才合适.
2.关于你的'都并行了还怎么单核？' ,照你的意思,在单核计算机时代没有并行这个概念了? 所以说那看你怎么理解并行? 我理解的并行就是在用户看来是一起执行的就是,可以在cpu时间分片或者多核都可以实现.如果你一定要强调同时发生,那么我要承认只有多核才能实现真正意义上的并行. 所以这点,我可以部分承认认知错误.
3.关于作者的原文我可以再说一点
的确,关于并行和并发的概念,人与亦云,的确没有一个统一的标准.
下面的两篇文章解释的就很大不同
[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)md.jsproxy.tk](https://link.juejin.cn/?target=https%3A%2F%2Fmd.jsproxy.tk%2Ffrom-the-scratch%2Fdont-be-confused-between-concurrency-and-parallelism-eac8e703943a)
[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)learn-gevent-socketio-dot-readthedocs-dot-io.ext.jsproxy.tk](https://link.juejin.cn/?target=https%3A%2F%2Flearn-gevent-socketio-dot-readthedocs-dot-io.ext.jsproxy.tk%2Fen%2Flatest%2Fgeneral_concepts.html)
还有知乎上的一个问题,
所以的确作者的解释在我看来是错误的,至少是不清晰的.
此刻的我倒是更愿意采信下面这个文章
[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)md.jsproxy.tk](https://link.juejin.cn/?target=https%3A%2F%2Fmd.jsproxy.tk%2Ffrom-the-scratch%2Fdont-be-confused-between-concurrency-and-parallelism-eac8e703943a)
咱们是理性探讨,你这句 '操作系统没好好学？' 把自己放的有些高呀,高处不胜寒呀.

展开

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/430664257120318)

[dendoink![lv-4](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/2c3fafd535a0625b44a5a57f9f536d77.svg)](https://juejin.cn/user/430664257120318)

2年前

哪有那么多可以杠的，我觉得萱萱和作者说得没毛病啊。
并发就是我从起床到上学之前刷牙，洗脸，吃了早餐，这是并发。
并行就是我一边吃早餐，一边刷手机。
就是个概念性的东西，同时开始同时结束的描述也只是为了强调这个时间段的性质，就像是刷牙洗脸吃早餐都是发生在从起床到上学之前这段时间。
质疑之前也先看看自己理解是否到位吧。

点赞

回复

查看更多回复

[![tianyant的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1926000100522360)

[tianyant![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1926000100522360)

前端工程师 @ tencent2年前

16.6 毫秒 = 1000 / 60

点赞

回复

[![在胖子的路上越走越远的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3104676568642366)

[在胖子的路上越走越远](https://juejin.cn/user/3104676568642366)

3年前

关于你说await把异步代码改造成了同步代码造成的性能降低，通过和朋友交流有不同的意见，他实际上应该还是异步，只不过看上去是同步，js是单线程, 改成同步代码会阻塞 js 引擎执行，async await 是以写同步代码的方式去写异步代码，以前 JQuery 的Ajax 就可以设置请求为同步请求, 就是请求的时候什么都干不了, 除了页面上的css动画在动以外, 所有js逻辑都等着请求结束才执行，这个才是真正的同步

2

6

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/a386aa8db73c9678458ec34161472ca5~300x300.image)](https://juejin.cn/user/712139233840407)

[yck![lv-7](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/7a2d00014351211140dd9bc0ba3afd8c.svg)](https://juejin.cn/user/712139233840407)

（作者）3年前

现在代码都执行完了，只需要请求3个无依赖接口。是通过 Promise.all 的方式好还是三个 await 的方式好？

点赞

回复

[![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/mirror-assets/1690a134c3c116943f3~tplv-t2oaga2asx-no-mark:24:24:24:24.awebp)](https://juejin.cn/user/3104676568642366)

[在胖子的路上越走越远](https://juejin.cn/user/3104676568642366)

回复

[yck](https://juejin.cn/user/712139233840407)

3年前

无依赖就用 Promise.all

“

现在代码都执行完了，只需要请求3个无依赖接口。是通过 Promise.all 的方式好还是三个 await 的方式好？

”

点赞

回复

查看更多回复

[![砖用冰西瓜的头像](https://p26-passport.byteacctimg.com/img/user-avatar/dee36188b78703b831bb042a0b17b593~300x300.image)](https://juejin.cn/user/1714893869549031)

[砖用冰西瓜![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/1714893869549031)

3年前

对于“回调地狱的根本问题”我有保留意见，通过看YDKJS，如果我没有理解错的话，回调主要有两个问题，一个是可读性的问题，另一个是信任问题。这是一个关于回调地狱的总结：[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)juejin.im](https://juejin.im/post/5b45bea65188251b1c3ce1ec)

4

回复

[![前端胖头鱼的头像](https://p3-passport.byteacctimg.com/img/user-avatar/1957416d67153775f273064ea766a8d3~300x300.image)](https://juejin.cn/user/3438928099549352)

[前端胖头鱼![lv-5](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f8d51984638784aff27209d38b6cd3bf.svg)](https://juejin.cn/user/3438928099549352)

FE @ 公众号: 前端胖头鱼3年前

确实是浅尝辄止

19

回复

[![Marcus_的头像](https://p3-passport.byteacctimg.com/img/mosaic-legacy/3792/5112637127~300x300.image)](https://juejin.cn/user/1468603263621832)

[Marcus_](https://juejin.cn/user/1468603263621832)

3年前

"以上代码在浏览器环境中，如果定时器执行过程中出现了耗时操作，多个回调函数会在耗时操作结束以后同时执行，这样可能就会带来性能上的问题。"
interval没有这个问题吧？队列里边已经有待执行的interval，不会重复添加进队列的啊？还是我对这段话有什么误解？

1

2

[![img](https://p9-passport.byteacctimg.com/img/user-avatar/8a914ad38206c67a4bbfd67fb7cfe430~300x300.image)](https://juejin.cn/user/2752832846694206)

[Connie_豆](https://juejin.cn/user/2752832846694206)

2年前

会添加进队列里哇，假设setInterval(fn, 500), fn执行时间是1000ms,那么当执行完一次fn的时候，此时过去了1000毫秒，此时异步队列里已经有两个fn了， 所以看起来他们就是接着执行，并没有起到间隔执行的效果。

点赞

回复

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/a1fe1751ca315cce8842238925c85dee~300x300.image)](https://juejin.cn/user/940837683070839)

[shadow不想说话](https://juejin.cn/user/940837683070839)

2年前

我觉得你这个问题提的很好，我也有这个疑惑

点赞

回复

[![L-杰祥的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/mirror-assets/168e0952b89ce4c233c~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/114004940315838)

[L-杰祥](https://juejin.cn/user/114004940315838)

3年前

38块还想买汽车，这里只有自行车

28

回复

[![Weismann的头像](https://p6-passport.byteacctimg.com/img/user-avatar/f09ddcddfcd9f551128206fe85d45da0~300x300.image)](https://juejin.cn/user/4142615543695293)

[Weismann![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/4142615543695293)

小年糕3年前

并发和并行的概念不对，建议作者看一下《深入理解计算机系统》

3

回复

查看全部 102 条回复