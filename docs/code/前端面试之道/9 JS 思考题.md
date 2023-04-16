# JS 思考题

之前我们通过了七个章节来学习关于 JS 这部分的内容，那么接下来，会以几道思考题的方式来确保大家理解这部分的内容。

这种方式不仅能加深你对知识点的理解，同时也能帮助你串联起多个碎片知识点。一旦你拥有将多个碎片知识点串联起来的能力，在面试中就不会经常出现一问一答的情况。如果面试官的每个问题你都能引申出一些相关联的知识点，那么面试官一定会提高对你的评价。

```!
思考题一：JS 分为哪两大类型？都有什么各自的特点？你该如何判断正确的类型？
```

首先这几道题目想必很多人都能够很好的答出来，接下来就给大家一点思路讲出与众不同的东西。

**思路引导：**

1. 对于原始类型来说，你可以指出 `null` 和 `number` 存在的一些问题。对于对象类型来说，你可以从垃圾回收的角度去切入，也可以说一下对象类型存在深浅拷贝的问题。
2. 对于判断类型来说，你可以去对比一下 `typeof` 和 `instanceof` 之间的区别，也可以指出 `instanceof` 判断类型也不是完全准确的。

以上就是这道题目的回答思路，当然不是说让大家完全按照这个思路去答题，而是存在一个意识，当回答面试题的时候，尽量去引申出这个知识点的某些坑或者与这个知识点相关联的东西。

```!
思考题二：你理解的原型是什么？
```

**思路引导：**

起码说出原型小节中的总结内容，然后还可以指出一些小点，比如并不是所有函数都有 `prototype` 属性，然后引申出原型链的概念，提出如何使用原型实现继承，继而可以引申出 ES6 中的 `class` 实现继承。

```!
思考题三：bind、call 和 apply 各自有什么区别？
```

**思路引导：**

首先肯定是说出三者的不同，如果自己实现过其中的函数，可以尝试说出自己的思路。然后可以聊一聊 `this` 的内容，有几种规则判断 `this` 到底是什么，`this` 规则会涉及到 `new`，那么最后可以说下自己对于 `new` 的理解。

```!
思考题四：ES6 中有使用过什么？
```

**思路引导：**

这边可说的实在太多，你可以列举 1 - 2 个点。比如说说 `class`，那么 `class` 又可以拉回到原型的问题；可以说说 `promise`，那么线就被拉到了异步的内容；可以说说 `proxy`，那么如果你使用过 Vue 这个框架，就可以谈谈响应式原理的内容；同样也可以说说 `let` 这些声明变量的语法，那么就可以谈及与 `var` 的不同，说到提升这块的内容。

```!
思考题五：JS 是如何运行的？
```

**思路引导：**

这其实是很大的一块内容。你可以先说 JS 是单线程运行的，这里就可以说说你理解的线程和进程的区别。然后讲到执行栈，接下来的内容就是涉及 Eventloop 了，微任务和宏任务的区别，哪些是微任务，哪些又是宏任务，还可以谈及浏览器和 Node 中的 Eventloop 的不同，最后还可以聊一聊 JS 中的垃圾回收。

## 小结

虽然思考题不多，但是其实每一道思考题背后都可以引申出很多内容，大家接下去在学习的过程中也应该始终有一个意识，你学习的这块内容到底和你现在脑海里的哪一个知识点有关联。同时也欢迎大家总结这些思考题，并且把总结的内容链接放在评论中，我会挑选出不错的文章单独放入一章节给大家参考。

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（32）

[![走进科学爱学习的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/mirror-assets/168e085a05365b84f86~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/3245414054901735)

[走进科学爱学习](https://juejin.cn/user/3245414054901735)

1年前

第一题的题目到底什么意思

点赞

回复

[![画饼师的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/184373686046110)

[画饼师![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/184373686046110)

2年前

这种思路回答问题很可能就成为传说中的答不到点子上，很容易扯了一大堆，面试官并没有找到自己想听到的部分。

19

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2084329779112807)

[江上酒![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2084329779112807)

1年前

那你就一问一答吧

1

回复

[![华少_月下兴风作浪的头像](https://p9-passport.byteacctimg.com/img/user-avatar/538ee4b3388efd817197c22b388b2fac~300x300.image)](https://juejin.cn/user/800100194723118)

[华少_月下兴风作浪![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/800100194723118)

前端3年前

加一

点赞

回复

[![zenghongtu的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/800100193676968)

[zenghongtu![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/800100193676968)

搬砖 @ 喜马拉雅3年前

第一题感觉应该是基本类型和引用类型

15

3

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3298190613292462)

[littlebirdflying![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/3298190613292462)

3年前

值类型

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3421335914838349)

[邹小邹![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/3421335914838349)

2年前

谁说有两种类型的，看的我一愣

点赞

回复

查看更多回复

[![zy_ch的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2963939079226200)

[zy_ch![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2963939079226200)

FE @ 东哥兄弟3年前

666

点赞

1

[![img](https://p3-passport.byteacctimg.com/img/user-avatar/74f55dcf207eb8f4e4415f1700102b4f~300x300.image)](https://juejin.cn/user/588993962446184)

[熊川宇](https://juejin.cn/user/588993962446184)

2年前

![😄](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f604.svg~tplv-t2oaga2asx-image.image)牛逼，举一反三。 确实要能有这种思路去答能加分。

点赞

回复

[![jeffacode同学的头像](https://p3-passport.byteacctimg.com/img/user-avatar/213928ed37ec02109101eaa621a85c36~300x300.image)](https://juejin.cn/user/835284566811192)

[jeffacode同学](https://juejin.cn/user/835284566811192)

前端开发 @ 不好意思无业游民3年前

思考题一里作者想说的应该是“你可以指出null和undefined存在的一些问题”吧？

1

3

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/6b1ce6e09bfc78942ed3f4f3452c1225~300x300.image)](https://juejin.cn/user/4072246798975991)

[二道思无邪](https://juejin.cn/user/4072246798975991)

3年前

应该说的是 typeof null ，0.1+0.2 !== 0.3

点赞

回复

[![img](https://p3-passport.byteacctimg.com/img/user-avatar/0b341256b30a5c1cc1126a89ba0f058e~300x300.image)](https://juejin.cn/user/676954893988680)

[MingYuan![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/676954893988680)

回复

[二道思无邪](https://juejin.cn/user/4072246798975991)

3年前

哦 是的 还是你说的比较清楚

“

应该说的是 typeof null ，0.1+0.2 !== 0.3

”

点赞

回复

查看更多回复

[![SwiftAcer的头像](https://p26-passport.byteacctimg.com/img/user-avatar/542562129e0dcb52ee9e9b55e43eac10~300x300.image)](https://juejin.cn/user/1451011079680663)

[SwiftAcer](https://juejin.cn/user/1451011079680663)

前端开发3年前

每次都是一问一答，原来我的问题在这里

15

回复

[![Murphy君39852的头像](https://p26-passport.byteacctimg.com/img/user-avatar/734817124695935b75d4cb5dde10ff4b~300x300.image)](https://juejin.cn/user/624178334275911)

[Murphy君39852](https://juejin.cn/user/624178334275911)

3年前

比如并不是所有函数都有 prototype 属性。这个怎么理解？是所有的构造函数才有prototype属性？

2

9

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/a386aa8db73c9678458ec34161472ca5~300x300.image)](https://juejin.cn/user/712139233840407)

[yck![lv-7](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/7a2d00014351211140dd9bc0ba3afd8c.svg)](https://juejin.cn/user/712139233840407)

（作者）3年前

回去看之前的章节

点赞

回复

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/cf6f2547065a9295a1e577b8b1040c9f~300x300.image)](https://juejin.cn/user/4283353030463207)

[FatDong1![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/4283353030463207)

回复

[yck](https://juejin.cn/user/712139233840407)

3年前

Function.prototype.bind()这句有什么应用场景吗

“

回去看之前的章节

”

点赞

回复

查看更多回复

[![上沅兮的头像](https://p3-passport.byteacctimg.com/img/user-avatar/ba0c04c145d6f8a81c6dbfbbeb257155~300x300.image)](https://juejin.cn/user/2840793775616168)

[上沅兮![lv-4](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/2c3fafd535a0625b44a5a57f9f536d77.svg)](https://juejin.cn/user/2840793775616168)

wechat： @ polaris-dxz3年前

我还以为小册能讲的东西能有多细，原来也是引导。。。

3

2

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/af7e12cec49ad7230f5e2ff2cf106d77~300x300.image)](https://juejin.cn/user/2013961032829799)

[🙊就是我56330](https://juejin.cn/user/2013961032829799)

3年前

考试大纲 哈哈哈

点赞

回复

[![img](https://p6-passport.byteacctimg.com/img/mosaic-legacy/3793/3131589739~300x300.image)](https://juejin.cn/user/3421335915861549)

[卷家老大](https://juejin.cn/user/3421335915861549)

3年前

不然为啥叫小册