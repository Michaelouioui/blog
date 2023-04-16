# JS 进阶知识点及常考面试题

在这一章节中，我们将会学习到一些原理相关的知识，不会解释涉及到的知识点的作用及用法，如果大家对于这些内容还不怎么熟悉，推荐先去学习相关的知识点内容再来学习原理知识。

## 手写 call、apply 及 bind 函数

```!
涉及面试题：call、apply 及 bind 函数内部实现是怎么样的？
```

首先从以下几点来考虑如何实现这几个函数

- 不传入第一个参数，那么上下文默认为 `window`
- 改变了 `this` 指向，让新的对象可以执行该函数，并能接受参数

那么我们先来实现 `call`

```js
Function.prototype.myCall = function(context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  context = context || window
  context.fn = this
  const args = [...arguments].slice(1)
  const result = context.fn(...args)
  delete context.fn
  return result
}
```

以下是对实现的分析：

- 首先 `context` 为可选参数，如果不传的话默认上下文为 `window`
- 接下来给 `context` 创建一个 `fn` 属性，并将值设置为需要调用的函数
- 因为 `call` 可以传入多个参数作为调用函数的参数，所以需要将参数剥离出来
- 然后调用函数并将对象上的函数删除

以上就是实现 `call` 的思路，`apply` 的实现也类似，区别在于对参数的处理，所以就不一一分析思路了

```js
Function.prototype.myApply = function(context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  context = context || window
  context.fn = this
  let result
  // 处理参数和 call 有区别
  if (arguments[1]) {
    result = context.fn(...arguments[1])
  } else {
    result = context.fn()
  }
  delete context.fn
  return result
}
```

`bind` 的实现对比其他两个函数略微地复杂了一点，因为 `bind` 需要返回一个函数，需要判断一些边界问题，以下是 `bind` 的实现

```js
Function.prototype.myBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  const _this = this
  const args = [...arguments].slice(1)
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F()，所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}
```

以下是对实现的分析：

- 前几步和之前的实现差不多，就不赘述了
- `bind` 返回了一个函数，对于函数来说有两种方式调用，一种是直接调用，一种是通过 `new` 的方式，我们先来说直接调用的方式
- 对于直接调用来说，这里选择了 `apply` 的方式实现，但是对于参数需要注意以下情况：因为 `bind` 可以实现类似这样的代码 `f.bind(obj, 1)(2)`，所以我们需要将两边的参数拼接起来，于是就有了这样的实现 `args.concat(...arguments)`
- 最后来说通过 `new` 的方式，在之前的章节中我们学习过如何判断 `this`，对于 `new` 的情况来说，不会被任何方式改变 `this`，所以对于这种情况我们需要忽略传入的 `this`

## new

```!
涉及面试题：new 的原理是什么？通过 new 的方式创建对象和通过字面量创建有什么区别？
```

在调用 `new` 的过程中会发生以上四件事情：

1. 新生成了一个对象
2. 链接到原型
3. 绑定 this
4. 返回新对象

根据以上几个过程，我们也可以试着来自己实现一个 `new`

```js
function create() {
  let obj = {}
  let Con = [].shift.call(arguments)
  obj.__proto__ = Con.prototype
  let result = Con.apply(obj, arguments)
  return result instanceof Object ? result : obj
}
```

以下是对实现的分析：

- 创建一个空对象
- 获取构造函数
- 设置空对象的原型
- 绑定 `this` 并执行构造函数
- 确保返回值为对象

对于对象来说，其实都是通过 `new` 产生的，无论是 `function Foo()` 还是 `let a = { b : 1 }` 。

对于创建一个对象来说，更推荐使用字面量的方式创建对象（无论性能上还是可读性）。因为你使用 `new Object()` 的方式创建对象需要通过作用域链一层层找到 `Object`，但是你使用字面量的方式就没这个问题。

```js
function Foo() {}
// function 就是个语法糖
// 内部等同于 new Function()
let a = { b: 1 }
// 这个字面量内部也是使用了 new Object()
```

更多关于 `new` 的内容可以阅读我写的文章 [聊聊 new 操作符](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FKieSun%2FDream%2Fissues%2F14)。

## instanceof 的原理

```!
涉及面试题：instanceof 的原理是什么？
```

`instanceof` 可以正确的判断对象的类型，因为内部机制是通过判断对象的原型链中是不是能找到类型的 `prototype`。

我们也可以试着实现一下 `instanceof`

```js
function myInstanceof(left, right) {
  let prototype = right.prototype
  left = left.__proto__
  while (true) {
    if (left === null || left === undefined)
      return false
    if (prototype === left)
      return true
    left = left.__proto__
  }
}
```

以下是对实现的分析：

- 首先获取类型的原型
- 然后获得对象的原型
- 然后一直循环判断对象的原型是否等于类型的原型，直到对象原型为 `null`，因为原型链最终为 `null`

## 为什么 0.1 + 0.2 != 0.3

```!
涉及面试题：为什么 0.1 + 0.2 != 0.3？如何解决这个问题？
```

先说原因，因为 JS 采用 IEEE 754 双精度版本（64位），并且只要采用 IEEE 754 的语言都有该问题。

我们都知道计算机是通过二进制来存储东西的，那么 `0.1` 在二进制中会表示为

```js
// (0011) 表示循环
0.1 = 2^-4 * 1.10011(0011)
```

我们可以发现，`0.1` 在二进制中是无限循环的一些数字，其实不只是 `0.1`，其实很多十进制小数用二进制表示都是无限循环的。这样其实没什么问题，但是 JS 采用的浮点数标准却会裁剪掉我们的数字。

IEEE 754 双精度版本（64位）将 64 位分为了三段

- 第一位用来表示符号
- 接下去的 11 位用来表示指数
- 其他的位数用来表示有效位，也就是用二进制表示 `0.1` 中的 `10011(0011)`

那么这些循环的数字被裁剪了，就会出现精度丢失的问题，也就造成了 `0.1` 不再是 `0.1` 了，而是变成了 `0.100000000000000002`

```js
0.100000000000000002 === 0.1 // true
```

那么同样的，`0.2` 在二进制也是无限循环的，被裁剪后也失去了精度变成了 `0.200000000000000002`

```js
0.200000000000000002 === 0.2 // true
```

所以这两者相加不等于 `0.3` 而是 `0.300000000000000004`

```js
0.1 + 0.2 === 0.30000000000000004 // true
```

那么可能你又会有一个疑问，既然 `0.1` 不是 `0.1`，那为什么 `console.log(0.1)` 却是正确的呢？

因为在输入内容的时候，二进制被转换为了十进制，十进制又被转换为了字符串，在这个转换的过程中发生了取近似值的过程，所以打印出来的其实是一个近似值，你也可以通过以下代码来验证

```js
console.log(0.100000000000000002) // 0.1
```

那么说完了为什么，最后来说说怎么解决这个问题吧。其实解决的办法有很多，这里我们选用原生提供的方式来最简单的解决问题

```js
parseFloat((0.1 + 0.2).toFixed(10)) === 0.3 // true
```

## 垃圾回收机制

```!
涉及面试题：V8 下的垃圾回收机制是怎么样的？
```

V8 实现了准确式 GC，GC 算法采用了分代式垃圾回收机制。因此，V8 将内存（堆）分为新生代和老生代两部分。

## 新生代算法

新生代中的对象一般存活时间较短，使用 Scavenge GC 算法。

在新生代空间中，内存空间分为两部分，分别为 From 空间和 To 空间。在这两个空间中，必定有一个空间是使用的，另一个空间是空闲的。新分配的对象会被放入 From 空间中，当 From 空间被占满时，新生代 GC 就会启动了。算法会检查 From 空间中存活的对象并复制到 To 空间中，如果有失活的对象就会销毁。当复制完成后将 From 空间和 To 空间互换，这样 GC 就结束了。

## 老生代算法

老生代中的对象一般存活时间较长且数量也多，使用了两个算法，分别是标记清除算法和标记压缩算法。

在讲算法前，先来说下什么情况下对象会出现在老生代空间中：

- 新生代中的对象是否已经经历过一次 Scavenge 算法，如果经历过的话，会将对象从新生代空间移到老生代空间中。
- To 空间的对象占比大小超过 25 %。在这种情况下，为了不影响到内存分配，会将对象从新生代空间移到老生代空间中。

老生代中的空间很复杂，有如下几个空间

```c++
enum AllocationSpace {
  // TODO(v8:7464): Actually map this space's memory as read-only.
  RO_SPACE,    // 不变的对象空间
  NEW_SPACE,   // 新生代用于 GC 复制算法的空间
  OLD_SPACE,   // 老生代常驻对象空间
  CODE_SPACE,  // 老生代代码对象空间
  MAP_SPACE,   // 老生代 map 对象
  LO_SPACE,    // 老生代大空间对象
  NEW_LO_SPACE,  // 新生代大空间对象

  FIRST_SPACE = RO_SPACE,
  LAST_SPACE = NEW_LO_SPACE,
  FIRST_GROWABLE_PAGED_SPACE = OLD_SPACE,
  LAST_GROWABLE_PAGED_SPACE = MAP_SPACE
};
```

在老生代中，以下情况会先启动标记清除算法：

- 某一个空间没有分块的时候
- 空间中被对象超过一定限制
- 空间不能保证新生代中的对象移动到老生代中

在这个阶段中，会遍历堆中所有的对象，然后标记活的对象，在标记完成后，销毁所有没有被标记的对象。在标记大型对内存时，可能需要几百毫秒才能完成一次标记。这就会导致一些性能上的问题。为了解决这个问题，2011 年，V8 从 stop-the-world 标记切换到增量标志。在增量标记期间，GC 将标记工作分解为更小的模块，可以让 JS 应用逻辑在模块间隙执行一会，从而不至于让应用出现停顿情况。但在 2018 年，GC 技术又有了一个重大突破，这项技术名为并发标记。该技术可以让 GC 扫描和标记对象时，同时允许 JS 运行，你可以点击 [该博客](https://link.juejin.cn/?target=https%3A%2F%2Fv8project.blogspot.com%2F2018%2F06%2Fconcurrent-marking.html) 详细阅读。

清除对象后会造成堆内存出现碎片的情况，当碎片超过一定限制后会启动压缩算法。在压缩过程中，将活的对象像一端移动，直到所有对象都移动完成然后清理掉不需要的内存。

## 小结

以上就是 JS 进阶知识点的内容了，这部分的知识相比于之前的内容更加深入也更加的理论，也是在面试中能够于别的候选者拉开差距的一块内容。如果大家对于这个章节的内容存在疑问，欢迎在评论区与我互动。

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（111）

[![zy_ch的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2963939079226200)

[zy_ch![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2963939079226200)

FE @ 东哥兄弟3月前

老生代应该还有个引用计数法？

点赞

回复

[![Mr.HoTwo的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/976022054633998)

[Mr.HoTwo](https://juejin.cn/user/976022054633998)

5月前

加油

点赞

回复

[![闹闹不爱闹的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/114004939250392)

[闹闹不爱闹![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/114004939250392)

1年前

function myInstance(l, r) {
 const res = r.prototype;
 while(l) {
 if (l.__proto__ === res) return true;
 l = l.__proto__
 }
 return false;
}

展开

1

回复

[![你若像风的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2277843823766967)

[你若像风![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2277843823766967)

前端开发工程师1年前

写得挺好的，参考了一些资料和作者写的，明白了call 和 apply的实现。bind收尾new那一部分还有些不理解。相信以后能理解的。支持作者。

1

回复

[![我母鸡啊！的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/8451822729021)

[我母鸡啊！![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/8451822729021)

滑水大师🏊‍♀️ @ tuya1年前

function _new(fn,...arg){
 let obj = {}
 let con = [].slice.call(arguments)
 obj.__proto__ = con.prototype //链接原型
 const ret = fn.call(obj, ...arg); //改变this的指向
 return ret instanceof Object ? ret : obj;
}

展开

点赞

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/993614243047991)

[xiaoxiaoo![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/993614243047991)

1年前

你为什么不直接写fn.propotype

点赞

回复

[![chiuwingyan的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2242659450890776)

[chiuwingyan![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/2242659450890776)

前端工程师2年前

new原理那部分，链接到原型，obj = Object.create(Con.prototype)，这种方式会比较好吧？

4

回复

[![画饼师的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/184373686046110)

[画饼师![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/184373686046110)

2年前

myInstanceof和原生还是有挺大差距把，比如原生可以检验null 与undefined， 而该函数会在访问__proto__的时候就报错了

点赞

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/184373686046110)

[画饼师![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/184373686046110)

2年前

基本类型的判断都是错误结果

点赞

回复

[![ZHANGYU的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/4089838986339927)

[ZHANGYU![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/4089838986339927)

成都小前端2年前

假如传入的context本来就有个fn属性，岂不是就被覆盖删掉了

点赞

5

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/254742428393742)

[陈同学Wyatt![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/254742428393742)

2年前

可能性是存在，但极低极低。
你可以加一个context自有方法判断，如果存在的话用fn+随机数的方式来命名临时函数。

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/430664289093176)

[grantguo](https://juejin.cn/user/430664289093176)

1年前

可以用 let fn = Symbol(); context[fn] = this; es6新属性，还可以。

1

回复

查看更多回复

[![YU-SABRINA楚雨的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/4089838986860056)

[YU-SABRINA楚雨](https://juejin.cn/user/4089838986860056)

2年前

这本书差评

77

回复

[![已注销的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1310273592623127)

[已注销](https://juejin.cn/user/1310273592623127)

2年前

[] instanceof Object === true ？

1

6

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1310273592623127)

[已注销](https://juejin.cn/user/1310273592623127)

2年前

[].__proto__ !== Object.prototype 有人能解释以下吗

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/993614239898093)

[沙鸥_![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/993614239898093)

2年前

本来就应该是true啊...

点赞

回复

查看更多回复

[![kur的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3157453124406423)

[kur](https://juejin.cn/user/3157453124406423)

切图仔2年前

建议这一节先讲下call、apply的用法再讲手动实现。。。

4

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/430664257120318)

[dendoink![lv-4](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/2c3fafd535a0625b44a5a57f9f536d77.svg)](https://juejin.cn/user/430664257120318)

2年前

“在这一章节中，我们将会学习到一些原理相关的知识，不会解释涉及到的知识点的作用及用法，如果大家对于这些内容还不怎么熟悉，推荐先去学习相关的知识点内容再来学习原理知识。”
看下开头写的这段话...

点赞

回复

[![我是谁我在哪aaa的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2119514150673432)

[我是谁我在哪aaa](https://juejin.cn/user/2119514150673432)

2年前

// 因为返回了一个函数，我们可以 new F()，所以需要判断
 if (this instanceof F) {
 return new _this(...args, ...arguments)
 }
bind的实现, 这里是什么意思 能举个栗子吗

点赞

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2119514150673432)

[我是谁我在哪aaa](https://juejin.cn/user/2119514150673432)

2年前

new bind后的函数 相当于 new原来的构造函数...但是bind第2-n个参数优先传入

点赞

回复

[![little123的头像](https://p6-passport.byteacctimg.com/img/user-avatar/cdbdc72ef9fdeecc71db7eeb89c69c07~300x300.image)](https://juejin.cn/user/2013961031258269)

[little123![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2013961031258269)

2年前

myApply方法可以依赖myCall方法，不需要写重复代码

点赞

回复

[![Weismann的头像](https://p6-passport.byteacctimg.com/img/user-avatar/f09ddcddfcd9f551128206fe85d45da0~300x300.image)](https://juejin.cn/user/4142615543695293)

[Weismann![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/4142615543695293)

小年糕3年前

myCall与原生call的行为还是有点出入。
尝试以下代码
var demo='1';
function Demo() {return this}
Demo.myCall(demo);
Demo.call(demo);
事实上会报错，因为context是个基础类型的，所以赋予不了context的fn属性，原生call的内部则对传入的demo做了处理demo=new Object(demo)，这样就可以了。
Function.prototype.myCall=function(context) {
 if(typeof this!=='function') {
 throw new TypeError(this + ' not a function');
 }
 if(typeof context !== 'object') context=new Object(context);
 context=context || window;
 context.fn=this;
 const args=[...arguments].slice(1);
 const result=context.fn(...args);
 delete context.fn;
 return result;
}

展开

1

5

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2365804754000360)

[Carry On Hxy](https://juejin.cn/user/2365804754000360)

3年前

感觉确实没有考虑context为基本类型的情况，还有我想问问怎么让它报not a function的错误？

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2365804754000360)

[Carry On Hxy](https://juejin.cn/user/2365804754000360)

3年前

懂了，懂了

点赞

回复

查看更多回复

[![首负的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3966693681662749)

[首负](https://juejin.cn/user/3966693681662749)

挖掘机技术总监 @ 蓝翔新东方3年前

为何 myCall 里要将函数绑定到 fn 上再删除 fn 属性？直接用一个变量声明不是更简洁吗？ context 有 fn 属性怎么办？

1

3

[![img](https://p9-passport.byteacctimg.com/img/user-avatar/8a914ad38206c67a4bbfd67fb7cfe430~300x300.image)](https://juejin.cn/user/2752832846694206)

[Connie_豆](https://juejin.cn/user/2752832846694206)

3年前

因为通过 context.fn()这种方式执行函数，this就是context

点赞

回复

[![img](https://p3-passport.byteacctimg.com/img/user-avatar/82f43f81f40e6b3a2dd3476d6e8b7113~300x300.image)](https://juejin.cn/user/1978776662325128)

[YoungShaw](https://juejin.cn/user/1978776662325128)

3年前

Symbol('fn')

点赞

回复

查看更多回复

[![光光同学的头像](https://p6-passport.byteacctimg.com/img/user-avatar/7b38bce1559fad25472c1794c8ca6002~300x300.image)](https://juejin.cn/user/2541726613638973)

[光光同学![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/2541726613638973)

进击的前端攻城狮 @ 云学堂3年前

Function.prototype.bind2 = function (context) {
 if (typeof this !== "function") {
 throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
 }
 var self = this;
 var arg = [...arguments].slice(1);
 function F() {
 
 //考虑需要绑定的函数可能是构造函数，this不会被绑定，但是参数仍然会传递。
 // this 指向为F
 if(this instanceof F) {
 return new self(...arg.concat(...arguments));
 }
 return self.apply(context, arg.concat(...arguments));
 }
 //继承一下绑定函数属性的值
 function Foo () {}
 Foo.prototype = this.prototype;
 //使用一个空函数进行中转。
 F.prototype = new Foo();
 return F;
}

展开

2

回复

[![zerosrat的头像](https://p9-passport.byteacctimg.com/img/mosaic-legacy/3792/5112637127~300x300.image)](https://juejin.cn/user/1538972006230942)

[zerosrat](https://juejin.cn/user/1538972006230942)

3年前

\```
// es6 的 Number.EPSILON 来解决 0.1 + 0.2 != 0.3 的问题
var result = Math.abs(0.2 - 0.3 + 0.1);
console.log(result);
// expected output: 2.7755575615628914e-17
console.log(result < Number.EPSILON);
// expected output: true
\```

展开

3

回复

[![裸考不挂的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/2/1680d864b58a553b~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/3087084380235127)

[裸考不挂![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3087084380235127)

铲屎官 @ 家里蹲3年前

@yck 这一节应该放到第四章ES6知识点之前，因为第四章涉及到了大量call apply new instanceof

2

回复

[![ConardLi的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/23/169198b85bfbec1d~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/3949101466785709)

[ConardLi![lv-6](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/74bd93adef7feff4fee26d08c0845b4f.svg)](https://juejin.cn/user/3949101466785709)

fe @ 字节跳动（大量HC 欢迎来撩）3年前

delete context.fn . 这句代码是认真的吗？？？

点赞

4

[![img](https://p3-passport.byteacctimg.com/img/user-avatar/24e34b949c10b91b35acf1e252bd84ed~300x300.image)](https://juejin.cn/user/958429870696856)

[蒙面大虾![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/958429870696856)

3年前

这是一个临时属性，为了能过去到当前被绑定的实体，但是又不想影响到原来的实体，所以就清除掉

点赞

回复

[![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/23/169198b85bfbec1d~tplv-t2oaga2asx-no-mark:24:24:24:24.awebp)](https://juejin.cn/user/3949101466785709)

[ConardLi![lv-6](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/74bd93adef7feff4fee26d08c0845b4f.svg)](https://juejin.cn/user/3949101466785709)

回复

[蒙面大虾](https://juejin.cn/user/958429870696856)

3年前

如果实现这样的功能，应该用symbol，现在的做法会把之前对象的fn属性干掉...

“

这是一个临时属性，为了能过去到当前被绑定的实体，但是又不想影响到原来的实体，所以就清除掉

”

1

回复

查看更多回复

[![华少_月下兴风作浪的头像](https://p9-passport.byteacctimg.com/img/user-avatar/538ee4b3388efd817197c22b388b2fac~300x300.image)](https://juejin.cn/user/800100194723118)

[华少_月下兴风作浪![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/800100194723118)

前端3年前

我试了一下，好像不能判断出来，不知道是不是我写错了，你试过了吗![👇](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f447.svg~tplv-t2oaga2asx-image.image)

点赞

回复

查看全部 111 条回复