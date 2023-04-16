# 手写 Promise

在上一章节中我们了解了 `Promise` 的一些易错点，在这一章节中，我们会通过手写一个符合 Promise/A+ 规范的 `Promise` 来深入理解它，并且手写 `Promise` 也是一道大厂常考题，在进入正题之前，推荐各位阅读一下 [Promise/A+ 规范](https://link.juejin.cn/?target=http%3A%2F%2Fwww.ituring.com.cn%2Farticle%2F66566)，这样才能更好地理解这个章节的代码。

## 实现一个简易版 Promise

在完成符合 Promise/A+ 规范的代码之前，我们可以先来实现一个简易版 `Promise`，因为在面试中，如果你能实现出一个简易版的 `Promise` 基本可以过关了。

那么我们先来搭建构建函数的大体框架

```js
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'

function MyPromise(fn) {
  const that = this
  that.state = PENDING
  that.value = null
  that.resolvedCallbacks = []
  that.rejectedCallbacks = []
  // 待完善 resolve 和 reject 函数
  // 待完善执行 fn 函数
}
```

- 首先我们创建了三个常量用于表示状态，对于经常使用的一些值都应该通过常量来管理，便于开发及后期维护
- 在函数体内部首先创建了常量 `that`，因为代码可能会异步执行，用于获取正确的 `this` 对象
- 一开始 `Promise` 的状态应该是 `pending`
- `value` 变量用于保存 `resolve` 或者 `reject` 中传入的值
- `resolvedCallbacks` 和 `rejectedCallbacks` 用于保存 `then` 中的回调，因为当执行完 `Promise` 时状态可能还是等待中，这时候应该把 `then` 中的回调保存起来用于状态改变时使用

接下来我们来完善 `resolve` 和 `reject` 函数，添加在 `MyPromise` 函数体内部

```js
function resolve(value) {
  if (that.state === PENDING) {
    that.state = RESOLVED
    that.value = value
    that.resolvedCallbacks.map(cb => cb(that.value))
  }
}

function reject(value) {
  if (that.state === PENDING) {
    that.state = REJECTED
    that.value = value
    that.rejectedCallbacks.map(cb => cb(that.value))
  }
}
```

这两个函数代码类似，就一起解析了

- 首先两个函数都得判断当前状态是否为等待中，因为规范规定只有等待态才可以改变状态
- 将当前状态更改为对应状态，并且将传入的值赋值给 `value`
- 遍历回调数组并执行

完成以上两个函数以后，我们就该实现如何执行 `Promise` 中传入的函数了

```js
try {
  fn(resolve, reject)
} catch (e) {
  reject(e)
}
```

- 实现很简单，执行传入的参数并且将之前两个函数当做参数传进去
- 要注意的是，可能执行函数过程中会遇到错误，需要捕获错误并且执行 `reject` 函数

最后我们来实现较为复杂的 `then` 函数

```js
MyPromise.prototype.then = function(onFulfilled, onRejected) {
  const that = this
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v
  onRejected =
    typeof onRejected === 'function'
      ? onRejected
      : r => {
          throw r
        }
  if (that.state === PENDING) {
    that.resolvedCallbacks.push(onFulfilled)
    that.rejectedCallbacks.push(onRejected)
  }
  if (that.state === RESOLVED) {
    onFulfilled(that.value)
  }
  if (that.state === REJECTED) {
    onRejected(that.value)
  }
}
```

- 首先判断两个参数是否为函数类型，因为这两个参数是可选参数

- 当参数不是函数类型时，需要创建一个函数赋值给对应的参数，同时也实现了透传，比如如下代码

  ```js
  // 该代码目前在简单版中会报错
  // 只是作为一个透传的例子
  Promise.resolve(4).then().then((value) => console.log(value))
  ```

- 接下来就是一系列判断状态的逻辑，当状态不是等待态时，就去执行相对应的函数。如果状态是等待态的话，就往回调函数中 `push` 函数，比如如下代码就会进入等待态的逻辑

  ```js
  new MyPromise((resolve, reject) => {
    setTimeout(() => {
      resolve(1)
    }, 0)
  }).then(value => {
    console.log(value)
  })
  ```

以上就是简单版 `Promise` 实现，接下来一小节是实现完整版 `Promise` 的解析，相信看完完整版的你，一定会对于 `Promise` 的理解更上一层楼。

## 实现一个符合 Promise/A+ 规范的 Promise

这小节代码需要大家配合规范阅读，因为大部分代码都是根据规范去实现的。

我们先来改造一下 `resolve` 和 `reject` 函数

```js
function resolve(value) {
  if (value instanceof MyPromise) {
    return value.then(resolve, reject)
  }
  setTimeout(() => {
    if (that.state === PENDING) {
      that.state = RESOLVED
      that.value = value
      that.resolvedCallbacks.map(cb => cb(that.value))
    }
  }, 0)
}
function reject(value) {
  setTimeout(() => {
    if (that.state === PENDING) {
      that.state = REJECTED
      that.value = value
      that.rejectedCallbacks.map(cb => cb(that.value))
    }
  }, 0)
}
```

- 对于 `resolve` 函数来说，首先需要判断传入的值是否为 `Promise` 类型
- 为了保证函数执行顺序，需要将两个函数体代码使用 `setTimeout` 包裹起来

接下来继续改造 `then` 函数中的代码，首先我们需要新增一个变量 `promise2`，因为每个 `then` 函数都需要返回一个新的 `Promise` 对象，该变量用于保存新的返回对象，然后我们先来改造判断等待态的逻辑

```js
if (that.state === PENDING) {
  return (promise2 = new MyPromise((resolve, reject) => {
    that.resolvedCallbacks.push(() => {
      try {
        const x = onFulfilled(that.value)
        resolutionProcedure(promise2, x, resolve, reject)
      } catch (r) {
        reject(r)
      }
    })

    that.rejectedCallbacks.push(() => {
      try {
        const x = onRejected(that.value)
        resolutionProcedure(promise2, x, resolve, reject)
      } catch (r) {
        reject(r)
      }
    })
  }))
}
```

- 首先我们返回了一个新的 `Promise` 对象，并在 `Promise` 中传入了一个函数
- 函数的基本逻辑还是和之前一样，往回调数组中 `push` 函数
- 同样，在执行函数的过程中可能会遇到错误，所以使用了 `try...catch` 包裹
- 规范规定，执行 `onFulfilled` 或者 `onRejected` 函数时会返回一个 `x`，并且执行 `Promise` 解决过程，这是为了不同的 `Promise` 都可以兼容使用，比如 JQuery 的 `Promise` 能兼容 ES6 的 `Promise`

接下来我们改造判断执行态的逻辑

```js
if (that.state === RESOLVED) {
  return (promise2 = new MyPromise((resolve, reject) => {
    setTimeout(() => {
      try {
        const x = onFulfilled(that.value)
        resolutionProcedure(promise2, x, resolve, reject)
      } catch (reason) {
        reject(reason)
      }
    })
  }))
}
```

- 其实大家可以发现这段代码和判断等待态的逻辑基本一致，无非是传入的函数的函数体需要异步执行，这也是规范规定的
- 对于判断拒绝态的逻辑这里就不一一赘述了，留给大家自己完成这个作业

最后，当然也是最难的一部分，也就是实现兼容多种 `Promise` 的 `resolutionProcedure` 函数

```js
function resolutionProcedure(promise2, x, resolve, reject) {
  if (promise2 === x) {
    return reject(new TypeError('Error'))
  }
}
```

首先规范规定了 `x` 不能与 `promise2` 相等，这样会发生循环引用的问题，比如如下代码

```js
let p = new MyPromise((resolve, reject) => {
  resolve(1)
})
let p1 = p.then(value => {
  return p1
})
```

然后需要判断 `x` 的类型

```js
if (x instanceof MyPromise) {
    x.then(function(value) {
        resolutionProcedure(promise2, value, resolve, reject)
    }, reject)
}
```

这里的代码是完全按照规范实现的。如果 `x` 为 `Promise` 的话，需要判断以下几个情况：

1. 如果 `x` 处于等待态，`Promise` 需保持为等待态直至 `x` 被执行或拒绝
2. 如果 `x` 处于其他状态，则用相同的值处理 `Promise`

当然以上这些是规范需要我们判断的情况，实际上我们不判断状态也是可行的。

接下来我们继续按照规范来实现剩余的代码

```js
let called = false
if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
  try {
    let then = x.then
    if (typeof then === 'function') {
      then.call(
        x,
        y => {
          if (called) return
          called = true
          resolutionProcedure(promise2, y, resolve, reject)
        },
        e => {
          if (called) return
          called = true
          reject(e)
        }
      )
    } else {
      resolve(x)
    }
  } catch (e) {
    if (called) return
    called = true
    reject(e)
  }
} else {
  resolve(x)
}
```

- 首先创建一个变量 `called` 用于判断是否已经调用过函数
- 然后判断 `x` 是否为对象或者函数，如果都不是的话，将 `x` 传入 `resolve` 中
- 如果 `x` 是对象或者函数的话，先把 `x.then` 赋值给 `then`，然后判断 `then` 的类型，如果不是函数类型的话，就将 `x` 传入 `resolve` 中
- 如果 `then` 是函数类型的话，就将 `x` 作为函数的作用域 `this` 调用之，并且传递两个回调函数作为参数，第一个参数叫做 `resolvePromise` ，第二个参数叫做 `rejectPromise`，两个回调函数都需要判断是否已经执行过函数，然后进行相应的逻辑
- 以上代码在执行的过程中如果抛错了，将错误传入 `reject` 函数中

以上就是符合 Promise/A+ 规范的实现了，如果你对于这部分代码尚有疑问，欢迎在评论中与我互动。

## 小结

这一章节我们分别实现了简单版和符合 Promise/A+ 规范的 `Promise`，前者已经足够应付大部分面试的手写题目，毕竟写出一个符合规范的 `Promise` 在面试中不大现实。后者能让你更加深入地理解 `Promise` 的运行原理，做技术的深挖者。

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（97）

[![破防了兄弟们的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/774534639392055)

[破防了兄弟们](https://juejin.cn/user/774534639392055)

产品经理26天前

function promiseAll(promises){
 // 返回一个promise实例
 return new Promise((resolve, reject) => {
 // 做一个判断参数是否是数组
 if(!Array.isArray(promises)){
 return reject(new TypeError('arguments must be Array'))
 }
 let count = 0,
 newValues = new Array(promise.length) // 接收新的结果参数 建立一个伪数组
 for(let i = 0; i < promises.length; i++){ 
 // 运用promise特性 只会有一个状态
 Promise.resolve(promises[i])
 .then(res = > {
 count++
 newValues[i] = res // 把每次返回成功的数据添加到数组中
 if(count === promises.length){ // 数据接收完成
 return resolve(newValues) 
 }
 }, rej = > reject(rej))
 
 }
 }) 
}

展开

点赞

回复

[![actcool的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3967479190130206)

[actcool![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3967479190130206)

4月前

有一说一，这是我见过最完整的手写promise.then了。 
同时实现了链式调用、异步、return封装成Promise、then捕获Error。 绝！！

1

回复

[![Mr.HoTwo的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/976022054633998)

[Mr.HoTwo](https://juejin.cn/user/976022054633998)

7月前

冲鸭~

点赞

回复

[![flowColor的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/4494459261975464)

[flowColor![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/4494459261975464)

前端er12月前

有没有完整版本的？

点赞

回复

[![钱得乐的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/606586151903320)

[钱得乐![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/606586151903320)

1年前

请问什么情况下called会生效然后直接reture？能举一个例子么

点赞

回复

[![走进科学爱学习的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/mirror-assets/168e085a05365b84f86~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/3245414054901735)

[走进科学爱学习](https://juejin.cn/user/3245414054901735)

1年前

const p1 = new MyPromise(resolve=>{
resolve(new MyPromise(resolve=>{resolve(42)}))
})
p1.then(res=>console.log(res)) // MyPromise
const p2 = new Promise(resolve=>{
 resolve(new Promise(resolve=>{resolve(42)}))
})
p2.then(res=>console.log(res)) // 42

展开

点赞

1

[![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/mirror-assets/168e085a05365b84f86~tplv-t2oaga2asx-no-mark:24:24:24:24.awebp)](https://juejin.cn/user/3245414054901735)

[走进科学爱学习](https://juejin.cn/user/3245414054901735)

1年前

对resolve和reject的参数是thenable的情况也要做处理

点赞

回复

[![不写bug的米公子的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2840793776929454)

[不写bug的米公子![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/2840793776929454)

程序员1年前

\```js
class MyPromise {
	static PENDING = 'pending'
	static RESOLVED = 'resolved'
	static REJECTED = 'rejected'
	constructor(fn) {
	typeof fn === 'function' ? fn(this.resolve.bind(this), this.reject.bind(this)) : null;
	this.state = MyPromise.PENDING;
	this.value = null;
	this.newPromise = null;
	this.resolvedCallback = null;
	this.rejectedCallback = null;
	}
	then(onFulfilled) {
	this.resolvedCallback = onFulfilled;
	this.newPromise = new MyPromise();
	return this.newPromise;
	}
	catch(onRejected) {
	this.rejectedCallback = onRejected;
	}
	resolve(value) {
	if (this.state !== MyPromise.PENDING) return false;
	this.value = value;
	this.state = MyPromise.RESOLVED;
	if (this.newPromise) this.newPromise.resolve(this.resolvedCallback(value));
	}
	reject(err) {
	if (this.state !== MyPromise.PENDING) return false;
	this.value = err;
	this.state = MyPromise.REJECTED;
	this.rejectedCallbacks(err);
	}
}
const promise = new MyPromise((resolve, reject) => {
	setTimeout(resolve, 1000, 'hello');
});
promise.then(res => {
	console.log(res);
	res += ' world';
	return res;
}).then(res => {
	console.log(res);
	res += '!';
	return res;
}).then(res => {
	console.log(res);
}).catch(e => {
	console.error(e);
});
\```

展开

2

回复

[![三条裤子的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/571401777450414)

[三条裤子](https://juejin.cn/user/571401777450414)

2年前

比较详细的promise写法，参考resolve来写reject，不能实现透传。

点赞

回复

[![Marcus_的头像](https://p3-passport.byteacctimg.com/img/mosaic-legacy/3792/5112637127~300x300.image)](https://juejin.cn/user/1468603263621832)

[Marcus_](https://juejin.cn/user/1468603263621832)

2年前

我不太明白为什么要加个called做标记，什么情况会导致有调用多次的呢？

点赞

3

[![img](https://p3-passport.byteacctimg.com/img/mosaic-legacy/3792/5112637127~300x300.image)](https://juejin.cn/user/1468603263621832)

[Marcus_](https://juejin.cn/user/1468603263621832)

2年前

明白了

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/26044009546295)

[candisecandise![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/26044009546295)

1年前

我也不太明白，为啥要加咧~

点赞

回复

查看更多回复

[![yonSemn的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1257497032137671)

[yonSemn![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1257497032137671)

前端开发 @ 韭菜园2年前

好难，实现的代码看不太明白，代码讲解不够详细，懵😵

37

回复

[![kur的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3157453124406423)

[kur](https://juejin.cn/user/3157453124406423)

切图仔2年前

手写Promise太TMD难了

20

回复

[![云流的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3544481218168333)

[云流![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/3544481218168333)

前端2年前

照这个写法，then方法中的self.status不应该永远等于‘pending’么，因为改变状态都是setTimeout()异步的，但是.then()是同步。同样异步的只有x和resolutionProcedure()了，这里进行状态判断是因为x的状态未知吗 ？ 这里的嵌套让我理解起来非常痛苦

1

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/26044009546295)

[candisecandise![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/26044009546295)

1年前

我也觉得，resolve里面所有代码都放到setTimeout里面之后，then里面的回调函数所有状态都会在PENDING，那就永远不会走FULLFILLED 和 REJECTED流程了

点赞

回复

[![nl不分的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1081575171435047)

[nl不分](https://juejin.cn/user/1081575171435047)

bug搬运工 @ bug山泉2年前

/ 
 const PENDING = "pending"
 const RESOLVE = "resolve"
 const REJECT = "reject"
 function MyPromise(fn) {
 const that = this
 that.status = PENDING // MyPromise 内部状态
 that.value = null // 传入 resolve 和 reject 的值
 that.resolveCallbacks = [] // 保存 then 中resolve的回调函数
 that.rejectCallbacks = [] // 保存 then 中reject的回调函数
 // resolve 函数 Promise内部调用 resolve 函数 例:new MyPromise((resolve,reject)=>{resolve(1)})
 function resolve(val) {
 if (that.status === PENDING) {
 that.status = resolve
 that.value = val
 that.resolveCallbacks.forEach(cb => cb(that.value))
 }
 }
 // reject 函数 Promise内部调用的 reject 函数 例:new MyPromise((resolve,reject)=>{reject(1)})
 function reject(val) {
 if (that.status === PENDING) {
 that.status = REJECT
 that.value = val
 that.rejectCallbacks.forEach(cb => cb(that.value))
 }
 }
 // 调用传入 MyPromise 内的方法 例:new MyPromise((resolve,reject)=>{}) fn=(resolve,reject)=>{}
 try {
 fn(resolve, reject)
 } catch (error) {
 reject(error)
 }
 }
 // 在原型上添加then方法
 MyPromise.prototype.then = function (onFulfilled, onRejected) {
 const that = this
 // 判断传入的是否为函数
 onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v
 onRejected = typeof onRejected === 'function' ? onRejected : r => {
 throw r
 }
 //如果 Promise 内部存在异步代码，调用then方法时，此时 promise 内部还是 PENDING 状态，将 then 里面的函数添加进回调数组，当异步处理完成后调用 MyPromise 内部的 resolve 或者 reject 函数
 if (that.status === PENDING) {
 that.resolveCallbacks.push(onFulfilled)
 that.rejectCallbacks.push(onRe

展开

5

2

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1081575171435047)

[nl不分](https://juejin.cn/user/1081575171435047)

2年前

resolve方法内有个错误 that.status = resolve 要改为 that.status = RESOLVE

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2928754708978551)

[Orime小猪![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/2928754708978551)

1年前

牛皮

点赞

回复

[![待我长发及腰就是我的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1697301683255694)

[待我长发及腰就是我](https://juejin.cn/user/1697301683255694)

2年前

有没有大佬告诉我，这里的y是哪里来的，怎么突然多了一个变量还是未定义的 then.call(
 x,
 y => {
 if (called) return
 called = true
 resolutionProcedure(promise2, y, resolve, reject)
 },
 e => {
 if (called) return
 called = true
 reject(e)
 }

展开

点赞

3

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2752832847490557)

[屁儿快跑](https://juejin.cn/user/2752832847490557)

2年前

y是传入的第二个函数参数的参数

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/272334614968126)

[三耳朵![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/272334614968126)

2年前

Y是promise或者thenable对象resolve的解析值

1

回复

查看更多回复

[![允南的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3913917125895997)

[允南![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/3913917125895997)

2年前

if (x instanceof MyPromise) {
 x.then(function(value) {
 resolutionProcedure(promise2, value, resolve, reject)
 }, reject)
}
这段代码，规范中讲的是如果 x 处于执行态，用相同的值执行 promise，个人理解应该是直接resolve(value), 这里为什么还要使用resolutionProcedure

展开

点赞

1

[![img](https://p6-passport.byteacctimg.com/img/user-avatar/05628b398d2e2dfe39ff9a655575da36~300x300.image)](https://juejin.cn/user/3913917125895997)

[允南![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/3913917125895997)

2年前

评论删不了，大致理解了，如果执行的终值value还是Promise的话，那promise2继续接管，直到得到一个非Promise的终值

点赞

回复

[![KJ不打烊的头像](https://p6-passport.byteacctimg.com/img/user-avatar/9443e7e4cba265a6b68a3d967b8bff3d~300x300.image)](https://juejin.cn/user/588993962975581)

[KJ不打烊](https://juejin.cn/user/588993962975581)

摸鱼攻城师 @ gdut3年前

为什么要使用setTimeout来控制执行顺序？求解

2

1

[![img](https://p6-passport.byteacctimg.com/img/user-avatar/05628b398d2e2dfe39ff9a655575da36~300x300.image)](https://juejin.cn/user/3913917125895997)

[允南![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/3913917125895997)

2年前

个人觉得可以参考规范里面的调用时机：onFulfilled 和 onRejected 只有在执行环境堆栈仅包含平台代码时才可被调用。

点赞

回复

[![Ezreal同志的头像](https://p6-passport.byteacctimg.com/img/user-avatar/2dfe80257368e124ae041d058ccf12aa~300x300.image)](https://juejin.cn/user/1468603263355117)

[Ezreal同志![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1468603263355117)

前端CVS工程师 @ 外卖厂3年前

Promise 建议增加all和race 的内容

4

2

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3456520288732280)

[Skeanmy](https://juejin.cn/user/3456520288732280)

3年前

对，建议大家看看all和race，头条面试会问Promise的api，说then catch不够的

点赞

回复

[![img](https://p6-passport.byteacctimg.com/img/user-avatar/2dfe80257368e124ae041d058ccf12aa~300x300.image)](https://juejin.cn/user/1468603263355117)

[Ezreal同志![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1468603263355117)

回复

[Skeanmy](https://juejin.cn/user/3456520288732280)

3年前

喜马拉雅也面到了，问的时候，一脸懵逼，经过提醒才想到这些

“

对，建议大家看看all和race，头条面试会问Promise的api，说then catch不够的

”

点赞

回复

[![前端胖头鱼的头像](https://p3-passport.byteacctimg.com/img/user-avatar/1957416d67153775f273064ea766a8d3~300x300.image)](https://juejin.cn/user/3438928099549352)

[前端胖头鱼![lv-5](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f8d51984638784aff27209d38b6cd3bf.svg)](https://juejin.cn/user/3438928099549352)

FE @ 公众号: 前端胖头鱼3年前

that.resolvedCallbacks.map(cb => cb(that.value)) 
如果是为了做遍历的话 应该用forEach更合适吧

6

1

[![img](https://p6-passport.byteacctimg.com/img/user-avatar/cdbdc72ef9fdeecc71db7eeb89c69c07~300x300.image)](https://juejin.cn/user/2013961031258269)

[little123![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2013961031258269)

2年前

认同

点赞

回复

[![韩林的头像](https://p9-passport.byteacctimg.com/img/user-avatar/957df299a6285dc445d6f1f70713ce40~300x300.image)](https://juejin.cn/user/4019470240324862)

[韩林](https://juejin.cn/user/4019470240324862)

前端工程师 @ 阿巴巴本地生活3年前

666

1

回复

[![lh0330的头像](https://p6-passport.byteacctimg.com/img/mosaic-legacy/3795/3033762272~300x300.image)](https://juejin.cn/user/1574156380943181)

[lh0330](https://juejin.cn/user/1574156380943181)

3年前

您好，我想问一下问题，我们这边有一个功能是表格批量上架，一页是10条数据（有分页和搜索的功能）如果数据选中1000条，一页数据就10条 每一次渲染一页的数据就会渲染10000次 有没有好的优化方案

点赞

回复

查看全部 97 条回复