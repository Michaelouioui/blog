# JS 基础知识点及常考面试题（二）

在这一章节中我们继续来了解 JS 的一些常考和容易混乱的基础知识点。

## == vs ===

```!
涉及面试题：== 和 === 有什么区别？
```

对于 `==` 来说，如果对比双方的类型**不一样**的话，就会进行**类型转换**，这也就用到了我们上一章节讲的内容。

假如我们需要对比 `x` 和 `y` 是否相同，就会进行如下判断流程：

1. 首先会判断两者类型是否**相同**。相同的话就是比大小了

2. 类型不相同的话，那么就会进行类型转换

3. 会先判断是否在对比 `null` 和 `undefined`，是的话就会返回 `true`

4. 判断两者类型是否为

    

   ```
   string
   ```

    

   和

    

   ```
   number
   ```

   ，是的话就会将字符串转换为

    

   ```
   number
   ```

   ```js
   1 == '1'
         ↓
   1 ==  1
   ```

5. 判断其中一方是否为

    

   ```
   boolean
   ```

   ，是的话就会把

    

   ```
   boolean
   ```

    

   转为

    

   ```
   number
   ```

    

   再进行判断

   ```js
   '1' == true
           ↓
   '1' ==  1
           ↓
    1  ==  1
   ```

6. 判断其中一方是否为

    

   ```
   object
   ```

    

   且另一方为

    

   ```
   string
   ```

   、

   ```
   number
   ```

    

   或者

    

   ```
   symbol
   ```

   ，是的话就会把

    

   ```
   object
   ```

    

   转为原始类型再进行判断

   ```js
   '1' == { name: 'yck' }
           ↓
   '1' == '[object Object]'
   ```

```!
思考题：看完了上面的步骤，对于 [] == ![] 你是否能正确写出答案呢？
```

如果你觉得记忆步骤太麻烦的话，我还提供了流程图供大家使用：

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/12/19/167c4a2627fe55f1~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

当然了，这个流程图并没有将所有的情况都列举出来，我这里只将常用到的情况列举了，如果你想了解更多的内容可以参考 [标准文档](https://link.juejin.cn/?target=https%3A%2F%2Fwww.ecma-international.org%2Fecma-262%2F5.1%2F%23sec-11.9.1)。

对于 `===` 来说就简单多了，就是判断两者类型和值是否相同。

更多的对比可以阅读这篇 [文章](https://link.juejin.cn/?target=https%3A%2F%2Ffelix-kling.de%2Fjs-loose-comparison%2F)

## 闭包

```!
涉及面试题：什么是闭包？
```

闭包的定义其实很简单：函数 A 内部有一个函数 B，函数 B 可以访问到函数 A 中的变量，那么函数 B 就是闭包。

```js
function A() {
  let a = 1
  window.B = function () {
      console.log(a)
  }
}
A()
B() // 1
```

很多人对于闭包的解释可能是函数嵌套了函数，然后返回一个函数。其实这个解释是不完整的，就比如我上面这个例子就可以反驳这个观点。

在 JS 中，闭包存在的意义就是让我们可以间接访问函数内部的变量。

```!
经典面试题，循环中使用闭包解决 `var` 定义函数的问题
for (var i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i * 1000)
}
```

首先因为 `setTimeout` 是个异步函数，所以会先把循环全部执行完毕，这时候 `i` 就是 6 了，所以会输出一堆 6。

解决办法有三种，第一种是使用闭包的方式

```js
for (var i = 1; i <= 5; i++) {
  ;(function(j) {
    setTimeout(function timer() {
      console.log(j)
    }, j * 1000)
  })(i)
}
```

在上述代码中，我们首先使用了立即执行函数将 `i` 传入函数内部，这个时候值就被固定在了参数 `j` 上面不会改变，当下次执行 `timer` 这个闭包的时候，就可以使用外部函数的变量 `j`，从而达到目的。

第二种就是使用 `setTimeout `的第三个参数，这个参数会被当成 `timer` 函数的参数传入。

```js
for (var i = 1; i <= 5; i++) {
  setTimeout(
    function timer(j) {
      console.log(j)
    },
    i * 1000,
    i
  )
}
```

第三种就是使用 `let` 定义 `i` 了来解决问题了，这个也是最为推荐的方式

```js
for (let i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i * 1000)
}
```

## 深浅拷贝

```!
涉及面试题：什么是浅拷贝？如何实现浅拷贝？什么是深拷贝？如何实现深拷贝？
```

在上一章节中，我们了解了对象类型在赋值的过程中其实是复制了地址，从而会导致改变了一方其他也都被改变的情况。通常在开发中我们不希望出现这样的问题，我们可以使用浅拷贝来解决这个情况。

```js
let a = {
  age: 1
}
let b = a
a.age = 2
console.log(b.age) // 2
```

### 浅拷贝

首先可以通过 `Object.assign` 来解决这个问题，很多人认为这个函数是用来深拷贝的。其实并不是，`Object.assign` 只会拷贝所有的属性值到新的对象中，如果属性值是对象的话，拷贝的是地址，所以并不是深拷贝。

```js
let a = {
  age: 1
}
let b = Object.assign({}, a)
a.age = 2
console.log(b.age) // 1
```

另外我们还可以通过展开运算符 `...` 来实现浅拷贝

```js
let a = {
  age: 1
}
let b = { ...a }
a.age = 2
console.log(b.age) // 1
```

通常浅拷贝就能解决大部分问题了，但是当我们遇到如下情况就可能需要使用到深拷贝了

```js
let a = {
  age: 1,
  jobs: {
    first: 'FE'
  }
}
let b = { ...a }
a.jobs.first = 'native'
console.log(b.jobs.first) // native
```

浅拷贝只解决了第一层的问题，如果接下去的值中还有对象的话，那么就又回到最开始的话题了，两者享有相同的地址。要解决这个问题，我们就得使用深拷贝了。

### 深拷贝

这个问题通常可以通过 `JSON.parse(JSON.stringify(object))` 来解决。

```js
let a = {
  age: 1,
  jobs: {
    first: 'FE'
  }
}
let b = JSON.parse(JSON.stringify(a))
a.jobs.first = 'native'
console.log(b.jobs.first) // FE
```

但是该方法也是有局限性的：

- 会忽略 `undefined`
- 会忽略 `symbol`
- 不能序列化函数
- 不能解决循环引用的对象

```js
let obj = {
  a: 1,
  b: {
    c: 2,
    d: 3,
  },
}
obj.c = obj.b
obj.e = obj.a
obj.b.c = obj.c
obj.b.d = obj.b
obj.b.e = obj.b.c
let newObj = JSON.parse(JSON.stringify(obj))
console.log(newObj)
```

如果你有这么一个循环引用对象，你会发现并不能通过该方法实现深拷贝

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/3/28/1626b1ec2d3f9e41~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

在遇到函数、 `undefined` 或者 `symbol` 的时候，该对象也不能正常的序列化

```js
let a = {
  age: undefined,
  sex: Symbol('male'),
  jobs: function() {},
  name: 'yck'
}
let b = JSON.parse(JSON.stringify(a))
console.log(b) // {name: "yck"}
```

你会发现在上述情况中，该方法会忽略掉函数和 `undefined` 。

但是在通常情况下，复杂数据都是可以序列化的，所以这个函数可以解决大部分问题。

如果你所需拷贝的对象含有内置类型并且不包含函数，可以使用 `MessageChannel`

```js
function structuralClone(obj) {
  return new Promise(resolve => {
    const { port1, port2 } = new MessageChannel()
    port2.onmessage = ev => resolve(ev.data)
    port1.postMessage(obj)
  })
}

var obj = {
  a: 1,
  b: {
    c: 2
  }
}

obj.b.d = obj.b

// 注意该方法是异步的
// 可以处理 undefined 和循环引用对象
const test = async () => {
  const clone = await structuralClone(obj)
  console.log(clone)
}
test()
```

当然你可能想自己来实现一个深拷贝，但是其实实现一个深拷贝是很困难的，需要我们考虑好多种边界情况，比如原型链如何处理、DOM 如何处理等等，所以这里我们实现的深拷贝只是简易版，并且我其实更推荐使用 [lodash 的深拷贝函数](https://link.juejin.cn/?target=https%3A%2F%2Flodash.com%2Fdocs%23cloneDeep)。

```js
function deepClone(obj) {
  function isObject(o) {
    return (typeof o === 'object' || typeof o === 'function') && o !== null
  }

  if (!isObject(obj)) {
    throw new Error('非对象')
  }

  let isArray = Array.isArray(obj)
  let newObj = isArray ? [...obj] : { ...obj }
  Reflect.ownKeys(newObj).forEach(key => {
    newObj[key] = isObject(obj[key]) ? deepClone(obj[key]) : obj[key]
  })

  return newObj
}

let obj = {
  a: [1, 2, 3],
  b: {
    c: 2,
    d: 3
  }
}
let newObj = deepClone(obj)
newObj.b.c = 1
console.log(obj.b.c) // 2
```

## 原型

```!
涉及面试题：如何理解原型？如何理解原型链？
```

当我们创建一个对象时 `let obj = { age: 25 }`，我们可以发现能使用很多种函数，但是我们明明没有定义过它们，对于这种情况你是否有过疑惑？

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/16/1671d15f45fcedea~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp****&h=73&f=png&s=8860)

当我们在浏览器中打印 `obj` 时你会发现，在 `obj` 上居然还有一个 `__proto__` 属性，那么看来之前的疑问就和这个属性有关系了。

其实每个 JS 对象都有 `__proto__` 属性，这个属性指向了原型。这个属性在现在来说已经不推荐直接去使用它了，这只是浏览器在早期为了让我们访问到内部属性 `[[prototype]]` 来实现的一个东西。

讲到这里好像还是没有弄明白什么是原型，接下来让我们再看看 `__proto__` 里面有什么吧。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/16/1671d2c5a6bcccc4~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

看到这里你应该明白了，原型也是一个对象，并且这个对象中包含了很多函数，所以我们可以得出一个结论：对于 `obj` 来说，可以通过 `__proto__` 找到一个原型对象，在该对象中定义了很多函数让我们来使用。

在上面的图中我们还可以发现一个 `constructor` 属性，也就是构造函数

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/16/1671d329ec98ec0b~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

打开 `constructor` 属性我们又可以发现其中还有一个 `prototype` 属性，并且这个属性对应的值和先前我们在 `__proto__` 中看到的一模一样。所以我们又可以得出一个结论：原型的 `constructor` 属性指向构造函数，构造函数又通过 `prototype` 属性指回原型，但是并不是所有函数都具有这个属性，`Function.prototype.bind()` 就没有这个属性。

其实原型就是那么简单，接下来我们再来看一张图，相信这张图能让你彻底明白原型和原型链

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/16/1671d387e4189ec8~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

看完这张图，我再来解释下什么是原型链吧。其实原型链就是多个对象通过 `__proto__` 的方式连接了起来。为什么 `obj` 可以访问到 `valueOf` 函数，就是因为 `obj` 通过原型链找到了 `valueOf` 函数。

对于这一小节的知识点，总结起来就是以下几点：

- `Object` 是所有对象的爸爸，所有对象都可以通过 `__proto__` 找到它
- `Function` 是所有函数的爸爸，所有函数都可以通过 `__proto__` 找到它
- 函数的 `prototype` 是一个对象
- 对象的 `__proto__` 属性指向原型， `__proto__` 将对象和原型连接起来组成了原型链

如果你还想深入学习原型这部分的内容，可以阅读我之前写的[文章](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FKieSun%2FDream%2Fissues%2F2)

## 小结

以上就是全部的常考和容易混乱的基础知识点了，下一章节我们将会学习 ES6 部分的知识。如果大家对于这个章节的内容存在疑问，欢迎在评论区与我互动。

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（242）

[![流色的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1495771201421192)

[流色](https://juejin.cn/user/1495771201421192)

1月前

在《你不知道的JS》书里，对闭包的描述是：当函数可以记住并访问所在的词法作用域时，就产生了闭包。闭包指的是作用域的引用。

1

回复

[![Runnn的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2295436011122733)

[Runnn![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2295436011122733)

问题不大🤭 @ W1月前

万物尽头是null![👻](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f47b.svg~tplv-t2oaga2asx-image.image)

1

回复

[![zy_ch的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2963939079226200)

[zy_ch![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2963939079226200)

FE @ 东哥兄弟4月前

恺哥yyds

点赞

回复

[![dingsheng的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3386151544827607)

[dingsheng![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/3386151544827607)

Stay Hungry, Stay Foolish @ 自由职业者6月前

闭包的定义其实很简单：函数 A 内部有一个函数 B，函数 B 可以访问到函数 A 中的变量，那么函数 B 就是闭包。你这句话也只能说 这种情况下B符合闭包
真正的定义应该是： 将一个函数转移到声明它的词法作用域之外，当这个函数调用时，仍然持有声明它的词法作用域的引用。
至于很多人说的 函数A内声明一个函数B，然后返回函数B就叫闭包，就更不正确了，难道我下面这幅图里的代码不符合闭包吗？至于为啥都这么说，估计是因为ES6之前没有块级作用域的概念，只有通过函数来创建一个作用域.

展开

![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)

2

2

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3034307822105016)

[努努](https://juejin.cn/user/3034307822105016)

1月前

这个地方其实不是不对的，，可以看devtools 应该是函数A是一个闭包

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3175045313343031)

[小小小小前端](https://juejin.cn/user/3175045313343031)

回复

[努努](https://juejin.cn/user/3034307822105016)

1月前

兄弟 你这评论错了吧![[发呆]](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/jj_emoji_4.28b310a.png)

“

这个地方其实不是不对的，，可以看devtools 应该是函数A是一个闭包

”

点赞

回复

[![Yauoi的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2313028194009725)

[Yauoi](https://juejin.cn/user/2313028194009725)

前端 @ 保密7月前

万物尽头是null

1

回复

[![bigdriedfish的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2154698523281719)

[bigdriedfish](https://juejin.cn/user/2154698523281719)

9月前

所以Object 和Function 到底谁是爸爸呢

点赞

3

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/501033033535566)

[Aceee](https://juejin.cn/user/501033033535566)

9月前

Object啊，Object是所有对象的爸爸啊，函数也是对象。

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3597257778929869)

[gcfkhy](https://juejin.cn/user/3597257778929869)

回复

[Aceee](https://juejin.cn/user/501033033535566)

4月前

JS对象是终极 就连数组也是 object

“

Object啊，Object是所有对象的爸爸啊，函数也是对象。

”

点赞

回复

查看更多回复

[![NaldoWang的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2049145403359112)

[NaldoWang](https://juejin.cn/user/2049145403359112)

前端开发10月前

原型链这个图不错，之前看过《你不知道的Javascript》中讲的比较详细

点赞

回复

[![于先森的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3579665589736360)

[于先森![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3579665589736360)

前端工程师 @ 菜的睡不着觉11月前

赋值那里感觉可以改成这个 之后不判断函数 否则赋值后函数为空
newObj[key] = isObject(obj[key]) ? deepClone(obj[key]) : typeof obj[key] === 'function'? new Function('return '+ obj[key].toString())() : obj[key];

![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)

点赞

回复

[![秋山曾的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1556564193582797)

[秋山曾![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1556564193582797)

前端工程师1年前

这个深拷贝还不如不判断函数呢，拷贝后函数直接变空对象了![😂](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f602.svg~tplv-t2oaga2asx-image.image)

7

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/940837683070839)

[shadow不想说话](https://juejin.cn/user/940837683070839)

3月前

我也想说这个判断对象时处理函数有问题。。写的也太简单了点

点赞

回复

[![Monicaaa321的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1802854802921070)

[Monicaaa321](https://juejin.cn/user/1802854802921070)

1年前

原型原型链这篇讲得很清晰：[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)juejin.im](https://juejin.im/post/6844903749345886216)

2

回复

[![gqg的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3166189815372734)

[gqg![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3166189815372734)

前端开发工程师 @ 南方传媒集团有限公司1年前

为什么{name:1} == '1'会报错Unexpected token '=='

点赞

2

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/694547078720696)

[橘子在冬季![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/694547078720696)

1年前

{}里面的代码应该是会被执行的

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/26044007449390)

[冰河世纪](https://juejin.cn/user/26044007449390)

1年前

语法不对，用()包括起来运行试试

点赞

回复

[![Lany的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1028798616451278)

[Lany![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1028798616451278)

前端开发者1年前

我有个疑问，就是浅拷贝不是复制引用吗，而深拷贝是复制原地址。就是浅拷贝是像window里复制一份快捷方式一样，而深拷贝是直接复制快捷方式指向的资源

点赞

1

[![img](https://p3-passport.byteacctimg.com/img/mosaic-legacy/3797/2889309425~300x300.image)](https://juejin.cn/user/3175045312548984)

[音乐鱼](https://juejin.cn/user/3175045312548984)

1年前

浅拷贝是拷贝一层，深拷贝是向下递归拷贝。

点赞

回复

[![shuaiyafei的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/5/16b265eb4d109150~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/78820567418685)

[shuaiyafei](https://juejin.cn/user/78820567418685)

低级前端开发工程师1年前

day1

点赞

回复

[![洛竹的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/325111174662855)

[洛竹![lv-5](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f8d51984638784aff27209d38b6cd3bf.svg)](https://juejin.cn/user/325111174662855)

木系前端 @ 罗小黑战记1年前

少了作用域和作用域链的面试题

3

回复

[![Christine_Tang的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3280598428550222)

[Christine_Tang](https://juejin.cn/user/3280598428550222)

前端开发工程师2年前

Function 是所有函数的爸爸，所有函数都可以通过 __proto__ 找到它。这句话说的不对吧？不应该是Function.prototype 是所有函数的爸爸，所有函数都可以通过 __proto__ 找到它吗？

33

7

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1714893871395261)

[千叶风行![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/1714893871395261)

1年前

有道理

点赞

回复

[![img](https://p6-passport.byteacctimg.com/img/user-avatar/8550f1ec10377bbf7ddba90590b6f52e~300x300.image)](https://juejin.cn/user/694547077933069)

[阿豪C](https://juejin.cn/user/694547077933069)

1年前

Function.prototype.constructor === Function

点赞

回复

查看更多回复

[![广阔天地的头像](https://p26-passport.byteacctimg.com/img/user-avatar/7179d3e259dea720f0157a68519362ea~300x300.image)](https://juejin.cn/user/131597126862135)

[广阔天地![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/131597126862135)

陷阵之志，有死无生 @ 德玛西亚2年前

！可将变量转换成boolean类型，null、undefined、NaN以及空字符串('')取反都为true，其余都为false

1

回复

[![余腾靖的头像](https://p6-passport.byteacctimg.com/img/user-avatar/b4ad31b0fdc0dac5a51fda622e3e021d~300x300.image)](https://juejin.cn/user/2664871915684493)

[余腾靖![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/2664871915684493)

写 AE 和 PS 脚本的 @ 稿定设计2年前

原型可以看看我这篇文章[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)juejin.im](https://juejin.im/entry/5e2ff7f9e51d453c9e1560d2)

11

4

[![img](https://p6-passport.byteacctimg.com/img/user-avatar/b07bcf7b0754eff6abc72b6149dbc83a~300x300.image)](https://juejin.cn/user/4125023359209917)

[曾侃](https://juejin.cn/user/4125023359209917)

2年前

很棒很详细

点赞

回复

[![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/3/31/1712f7197f7dae47~tplv-t2oaga2asx-no-mark:24:24:24:24.awebp)](https://juejin.cn/user/4054654616085821)

[阿东全能](https://juejin.cn/user/4054654616085821)

2年前

已浏览,赞赞

点赞

回复

查看更多回复

[![寻_路的头像](https://p26-passport.byteacctimg.com/img/user-avatar/7d9fc8e84b8f04ecff81bc332c9735eb~300x300.image)](https://juejin.cn/user/3403743730103991)

[寻_路![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/3403743730103991)

前端开发工程师2年前

基于克隆的原型模式，如果要解释原型与原型链的话，我准备这么去说。

点赞

1

[![img](https://p9-passport.byteacctimg.com/img/user-avatar/11440560195bf87efb34217c55cb6c8a~300x300.image)](https://juejin.cn/user/1028798616451278)

[Lany![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1028798616451278)

1年前

不是基于委托吗

点赞

回复

[![sbotlp的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/3/31/1627cb4747067fe1~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/131597125296622)

[sbotlp](https://juejin.cn/user/131597125296622)

前端2年前

[] == ![],这篇很详细
[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)blog.csdn.net](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fqiqi_77_%2Farticle%2Fdetails%2F79456605)

5

2

[![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/12/14/167aaca18cbbeae5~tplv-t2oaga2asx-no-mark:24:24:24:24.awebp)](https://juejin.cn/user/2066737589927048)

[maker_feng](https://juejin.cn/user/2066737589927048)

2年前

好

点赞

回复

[![img](https://p9-passport.byteacctimg.com/img/user-avatar/c5ec95a4ae56ce4c0674d475ea759dae~300x300.image)](https://juejin.cn/user/1750078241901262)

[赵十八![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/1750078241901262)

1年前

感觉这个思考题有点突兀，思考题的关键在![]，并不是==。

点赞

回复

[![sbotlp的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/3/31/1627cb4747067fe1~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/131597125296622)

[sbotlp](https://juejin.cn/user/131597125296622)

前端2年前

[]!==[]
[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)blog.csdn.net](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fqiqi_77_%2Farticle%2Fdetails%2F79456605)

点赞

回复

查看全部 242 条回复