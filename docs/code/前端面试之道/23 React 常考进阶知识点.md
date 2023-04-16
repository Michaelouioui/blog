# React 常考进阶知识点

这一章节我们将来学习 React 的一些经常考到的进阶知识点，并且这章节还需要配合第十九章阅读，其中的内容经常会考到。

## HOC 是什么？相比 mixins 有什么优点？

很多人看到高阶组件（HOC）这个概念就被吓到了，认为这东西很难，其实这东西概念真的很简单，我们先来看一个例子。

```
function add(a, b) {
    return a + b
}
```

现在如果我想给这个 `add` 函数添加一个输出结果的功能，那么你可能会考虑我直接使用 `console.log` 不就实现了么。说的没错，但是如果我们想做的更加优雅并且容易复用和扩展，我们可以这样去做：

```
function withLog (fn) {
    function wrapper(a, b) {
        const result = fn(a, b)
        console.log(result)
        return result
    }
    return wrapper
}
const withLogAdd = withLog(add)
withLogAdd(1, 2)
```

其实这个做法在函数式编程里称之为高阶函数，大家都知道 React 的思想中是存在函数式编程的，高阶组件和高阶函数就是同一个东西。我们实现一个函数，传入一个组件，然后在函数内部再实现一个函数去扩展传入的组件，最后返回一个新的组件，这就是高阶组件的概念，作用就是为了更好的复用代码。

其实 HOC 和 Vue 中的 mixins 作用是一致的，并且在早期 React 也是使用 mixins 的方式。但是在使用 class 的方式创建组件以后，mixins 的方式就不能使用了，并且其实 mixins 也是存在一些问题的，比如：

- 隐含了一些依赖，比如我在组件中写了某个 `state` 并且在 `mixin` 中使用了，就这存在了一个依赖关系。万一下次别人要移除它，就得去 `mixin` 中查找依赖
- 多个 `mixin` 中可能存在相同命名的函数，同时代码组件中也不能出现相同命名的函数，否则就是重写了，其实我一直觉得命名真的是一件麻烦事。。
- 雪球效应，虽然我一个组件还是使用着同一个 `mixin`，但是一个 `mixin` 会被多个组件使用，可能会存在需求使得 `mixin` 修改原本的函数或者新增更多的函数，这样可能就会产生一个维护成本

HOC 解决了这些问题，并且它们达成的效果也是一致的，同时也更加的政治正确（毕竟更加函数式了）。

## 事件机制

React 其实自己实现了一套事件机制，首先我们考虑一下以下代码：

```
const Test = ({ list, handleClick }) => ({
    list.map((item, index) => (
        <span onClick={handleClick} key={index}>{index}</span>
    ))
})
```

以上类似代码想必大家经常会写到，但是你是否考虑过点击事件是否绑定在了每一个标签上？事实当然不是，JSX 上写的事件并没有绑定在对应的真实 DOM 上，而是通过事件代理的方式，将所有的事件都统一绑定在了 `document` 上。这样的方式不仅减少了内存消耗，还能在组件挂载销毁时统一订阅和移除事件。

另外冒泡到 `document` 上的事件也不是原生浏览器事件，而是 React 自己实现的合成事件（SyntheticEvent）。因此我们如果不想要事件冒泡的话，调用 `event.stopPropagation` 是无效的，而应该调用 `event.preventDefault`。

那么实现合成事件的目的是什么呢？总的来说在我看来好处有两点，分别是：

- 合成事件首先抹平了浏览器之间的兼容问题，另外这是一个跨浏览器原生事件包装器，赋予了跨浏览器开发的能力
- 对于原生浏览器事件来说，浏览器会给监听器创建一个事件对象。如果你有很多的事件监听，那么就需要分配很多的事件对象，造成高额的内存分配问题。但是对于合成事件来说，有一个事件池专门来管理它们的创建和销毁，当事件需要被使用时，就会从池子中复用对象，事件回调结束后，就会销毁事件对象上的属性，从而便于下次复用事件对象。

## 更新内容

- [React 进阶系列：Hooks 该怎么用](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FKieSun%2FDream%2Fissues%2F15)

## 小结

你可能会惊讶于这一章节的内容并不多的情况，其实你如果将两章 React 以及第十九章的内容全部学习完后，基本上 React 的大部分面试问题都可以解决。

当然你可能会觉得看的还不过瘾，这不需要担心。我已经决定写一个免费专栏「React 进阶」，专门讲解有难度的问题。比如组件的设计模式、新特性、部分源码解析等等内容。当然这些内容都是需要好好打磨的，所以更新的不会很快，有兴趣的可以持续关注，都会更新链接在这一章节中。

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（25）

[![方春的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/17181813258797)

[方春](https://juejin.cn/user/17181813258797)

1月前

现在不是挂在document上了

点赞

回复

[![前半的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/501033035378424)

[前半![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/501033035378424)

7月前

说好的下一章讲fiber 呢

点赞

回复

[![前半的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/501033035378424)

[前半![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/501033035378424)

7月前

首先得举例 mixins是啥

点赞

回复

[![mytac的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/747323636582184)

[mytac![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/747323636582184)

独立开发 @ 独立开发2年前

希望更下关于路由的，被面的频次比较高

2

回复

[![DarKniGht的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3808364010160024)

[DarKniGht![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3808364010160024)

前端工程师2年前

感觉HOC应该不算是函数式的政治正确吧，hook应该才是函数式的政治正确，而hook的诞生几乎就可以很大程度取代了HOC

1

回复

[![前端古力士的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2770425030652462)

[前端古力士![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/2770425030652462)

前端工程师 @ 字节跳动2年前

老师,我想请问下react的生态会考哪些?

4

回复

[![王唯佳的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1486195452285287)

[王唯佳![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/1486195452285287)

developer @ 蔚来3年前

另外冒泡到 document 上的事件也不是原生浏览器事件，而是 React 自己实现的合成事件（SyntheticEvent）。因此我们如果不想要事件冒泡的话，调用 event.stopPropagation 是无效的，而应该调用 event.preventDefault。
event.preventDefault 是阻止 控件的默认 行为，无法阻止冒泡的
event.stopPropagation 才是阻止冒泡的 方法
这个地方写错了吧

1

1

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1486195452285287)

[王唯佳![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/1486195452285287)

3年前

event.stopPropagation 阻止合成事件 冒泡的方法

点赞

回复

[![欧阳在掘金32800的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/mirror-assets/168e08a690801f5926d~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/588993962443160)

[欧阳在掘金32800](https://juejin.cn/user/588993962443160)

3年前

连政治正确都来了，真是......

点赞

回复

[![石子路的头像](https://p3-passport.byteacctimg.com/img/user-avatar/dc07fd55cf48af53f993df30d7de5b6b~300x300.image)](https://juejin.cn/user/2541726612594952)

[石子路](https://juejin.cn/user/2541726612594952)

视觉体验还原师 @ 字节跳动3年前

这一章节，含金量不高

9

回复

[![难受aaabbbccc的头像](https://p9-passport.byteacctimg.com/img/user-avatar/51e1edf19ceb4c488791d110819f7c40~300x300.image)](https://juejin.cn/user/3931509311675070)

[难受aaabbbccc![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3931509311675070)

前端工程师3年前

阻止冒泡不是 `e.nativeEvent.stopImmediatePropagation();` 这个吗？

1

回复

[![难受aaabbbccc的头像](https://p9-passport.byteacctimg.com/img/user-avatar/51e1edf19ceb4c488791d110819f7c40~300x300.image)](https://juejin.cn/user/3931509311675070)

[难受aaabbbccc![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3931509311675070)

前端工程师3年前

不是调用阻止冒泡不是 `e.nativeEvent.stopImmediatePropagation();` 这个吗？

1

回复

[![我只有三块钱的头像](https://p6-passport.byteacctimg.com/img/user-avatar/6da8cac3e5899f9afb207afdb2026c5b~300x300.image)](https://juejin.cn/user/1275089219486893)

[我只有三块钱](https://juejin.cn/user/1275089219486893)

前端开发 @ 深圳~金融相关3年前

捕获函数式推崇者一枚

点赞

回复

[![日明的头像](https://p26-passport.byteacctimg.com/img/user-avatar/044152de837911a4757c58ea986bcca1~300x300.image)](https://juejin.cn/user/1151943916400487)

[日明![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/1151943916400487)

前端工程师 @ 闪畅3年前

react这两篇感觉没啥用啊..

2

回复

[![ZedCoding的头像](https://p9-passport.byteacctimg.com/img/user-avatar/5a7bc1423ba1db61723cc44924335708~300x300.image)](https://juejin.cn/user/3790771822532110)

[ZedCoding](https://juejin.cn/user/3790771822532110)

前端 @ 杭州高级切图工作室3年前

没啥用

7

回复

[![牛晋的头像](https://p3-passport.byteacctimg.com/img/user-avatar/253b3effd45dc088e8ddc349988e74ab~300x300.image)](https://juejin.cn/user/1926000099988056)

[牛晋](https://juejin.cn/user/1926000099988056)

想做手艺人3年前

老实说，受限于篇幅长度，这篇「 React 进阶」确实有点浅。不如附加一个 React 进阶路线图

点赞

回复

[![张熙沐枫的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/21/1690e7fd47b865d0~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/3861140565926808)

[张熙沐枫](https://juejin.cn/user/3861140565926808)

高级前端3年前

0.0

点赞

回复

[![小林其的头像](https://p6-passport.byteacctimg.com/img/user-avatar/ebb80602e89824383d8d7938e1d7f873~300x300.image)](https://juejin.cn/user/2576910984940573)

[小林其](https://juejin.cn/user/2576910984940573)

软件工程师 @ 没得公司3年前

不知道怎么说，进阶先讲了hoc，却不深入讲一下柯里化和高阶组件的生命周期；我很不熟的render props和render callback也不讲一下吗？

1

7

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/a386aa8db73c9678458ec34161472ca5~300x300.image)](https://juejin.cn/user/712139233840407)

[yck![lv-7](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/7a2d00014351211140dd9bc0ba3afd8c.svg)](https://juejin.cn/user/712139233840407)

（作者）3年前

柯里化就更加是 FP 的东西了，说了会有更多人更加模糊，另外高阶组件的生命周期还和别的生命周期不一样了？
其他你说的 render props 这些，近期会更新新的文章的

点赞

回复

[![img](https://p9-passport.byteacctimg.com/img/mosaic-legacy/3796/2975850990~300x300.image)](https://juejin.cn/user/2049145403089566)

[正正果实](https://juejin.cn/user/2049145403089566)

3年前

期待React进阶

点赞

回复

查看更多回复