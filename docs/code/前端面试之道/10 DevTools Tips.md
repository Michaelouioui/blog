# DevTools Tips

这一章节的内容可能和面试没有太大关系，但是如果你能很好地使用 DevTools 的话，它能够很好地帮助你提高生产力和解决问题的能力。在这一章节中，我不会去介绍大家经常使用的功能，重点在于让大家学习到一些使用 DevTools 的技巧。

## Elements

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674aad0f69568b2~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

这个功能肯定是大家经常用到的，我们可以通过它来可视化所有的 DOM 标签，可以查看任何 DOM 的属性，接下来我们就来学习一下关于这方面的 Tips。

### Element 状态

你可能会在开发中遇到这么一个场景：给一个 `a` 标签设置了多种状态下的样式，但是如果手动去改变状态的话就有点麻烦，这时候这个 Tips 就能帮你解决这个问题。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674ab358e794a71~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

可以从上图中看到，无论你想看到元素的何种状态下的样式，都只需要勾选相对应的状态就可以了，这是不是比手动更改方便多了？

### 快速定位 Element

通常页面都是可以滚动的，那么如果想查看的元素不在当前窗口的话，你还需要滚动页面才能找到元素，这时候这个 Tips 就能帮你解决这个问题。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674ac22d8044f4b~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

当点击这个选项的时候，页面就会自动滚动到元素所在的位置，这样比边滚动边查看是否找到元素的方式方便多了。

### DOM 断点

给 JS 打断点想必各位都听过，但是 DOM 断点知道的人应该就少了。如果你想查看一个 DOM 元素是如何通过 JS 更改的，你就可以使用这个功能。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674ad1104faf69c~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

当我们给 `ul` 添加该断点以后，一旦 `ul` 子元素发生了改动，比如说增加了子元素的个数，那么就会自动跳转到对应的 JS 代码

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674ad27ee181161~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

其实不光可以给 DOM 打断点，我们还可以给 Ajax 或者 Event Listener 打断点。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674af1f8fc819c7~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

### 查看事件

我们还可以通过 DevTools 来查看页面中添加了多少的事件。假如当你发现页面滚动起来有性能上的问题时，就可以查看一下有多少 `scroll` 事件被添加了

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674ad5291419bb3~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

### 找到之前查看过的 DOM 元素

不知道你是否遇到过这样的问题，找不到之前查看过的 DOM 元素在哪里了，需要一个个去找这就有点麻烦了，这时候你就可以使用这个功能。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674ad91b7771b01~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

我们可以通过 `$0` 来找到上一次查看过的 DOM 元素，`$1` 就是上上次的元素，之后以此类推。这时候你可能会说，打印出来元素有啥用，在具体什么位置还要去找啊，不用急，马上我就可以解决这个问题

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674adbf740598f6~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

当你点击这个选项时，页面立马会跳转至元素所在位置，并且 DevTools 也会变到 Elements 标签。

## Debugging

给 JS 打断点想必大家都会，但是打断点也是有一个不为人知的 Tips 的。

```js
for (let index = 0; index < 10; index++) {
  // 各种逻辑
  console.log(index)
}
```

对于这段代码来说，如果我只想看到 `index` 为 `5` 时相应的断点信息，但是一旦打了断点，就会每次循环都会停下来，很浪费时间，那么通过这个小技巧我们就可以圆满解决这个问题

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674aebbbb36cc35~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

首先我们先右键断点，然后选择 `Edit breakpoint...` 选项

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674aec3d3f3e70d~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

在弹框内输入 `index === 5`，这样断点就会变为橙色，并且只有当符合表达式的情况时断点才会被执行

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/25/1674aed4d18967e9~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

## 小结

虽然这一章的内容并不多，但是涉及到的几个场景都是日常经常会碰到的，希望这一章节的内容会对大家有帮助。如果大家对于这个章节的内容存在疑问，欢迎在评论区与我互动。

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（39）

[![zy_ch的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2963939079226200)

[zy_ch![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2963939079226200)

FE @ 东哥兄弟3月前

牛皮

点赞

回复

[![Mr.HoTwo的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/976022054633998)

[Mr.HoTwo](https://juejin.cn/user/976022054633998)

5月前

加油

点赞

回复

[![小乔流水吾家的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2295436010068423)

[小乔流水吾家![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2295436010068423)

全栈工程师1年前

学到了最后一个

1

回复

[![Mose的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2673621306968663)

[Mose![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2673621306968663)

前端小菜鸡 @ 待业1年前

学到了最后一个

点赞

回复

[![雪无止尽的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2084329776493032)

[雪无止尽](https://juejin.cn/user/2084329776493032)

前端工程师 @ 龙湖1年前

for循环的断点调试 第一次 get到了

3

回复

[![颜酱的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/905653309941495)

[颜酱![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/905653309941495)

前端酱 @ frontzhm@163.com2年前

涨姿势！！！

1

回复

[![一知特立独行的猫的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/4283353032080365)

[一知特立独行的猫](https://juejin.cn/user/4283353032080365)

前端 @ 北京某大厂2年前

很实用的小技巧![❤](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/2764.svg~tplv-t2oaga2asx-image.image)️

点赞

回复

[![FrangFan的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/272334612076110)

[FrangFan](https://juejin.cn/user/272334612076110)

大前端2年前

请问文章中讲的 DOM 断点 是要什么条件下才可以用？我这边用chrome（版本73）中断不了。

点赞

3

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/3896324939058957)

[FantasticPerson](https://juejin.cn/user/3896324939058957)

2年前

73应该不会断不了 会不会是你的dom根本没有发生变化

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/272334612076110)

[FrangFan](https://juejin.cn/user/272334612076110)

2年前

大概知道原因了，我的理解是它这初始化时是不会触发回调的，只有打完断点，有更新了才会，刷新页面不行，如果有click那些的倒是可以

点赞

回复

查看更多回复

[![__晴天的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/694547078196568)

[__晴天![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/694547078196568)

web @ Encompass3年前

学到了学到了

点赞

回复

[![L-杰祥的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/mirror-assets/168e0952b89ce4c233c~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/114004940315838)

[L-杰祥](https://juejin.cn/user/114004940315838)

3年前

哇哇哇，太实用性了。超赞！

1

回复

[![收废品的头像](https://p26-passport.byteacctimg.com/img/user-avatar/764b4e2f0c4262750520cfd890f3b103~300x300.image)](https://juejin.cn/user/1081575171694743)

[收废品](https://juejin.cn/user/1081575171694743)

捡破烂 @ 华东废品站总部3年前

里面过多过少都用过，最后一个调试技巧就比较有灵性了

2

回复

[![万有_青年的头像](https://p6-passport.byteacctimg.com/img/user-avatar/c7ece29d533ce5a738dfb6d95e1926b9~300x300.image)](https://juejin.cn/user/2963939078450637)

[万有_青年![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/2963939078450637)

前端 @ Tencent3年前

超棒的

点赞

回复

[![尤小小的头像](https://p3-passport.byteacctimg.com/img/user-avatar/8335ec6db89655ac81b1702e5d4adea6~300x300.image)](https://juejin.cn/user/1609340751970440)

[尤小小![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/1609340751970440)

FE3年前

不错

点赞

回复

[![鱼与雨xx的头像](https://p9-passport.byteacctimg.com/img/user-avatar/f8f5e88b4efacbd723a2e2f308d5e8ad~300x300.image)](https://juejin.cn/user/571401778760238)

[鱼与雨xx](https://juejin.cn/user/571401778760238)

前端开发3年前

最后一个超级有用

8

回复

[![hedia的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/7/25/16c2815297b5a16b~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/2594503171530872)

[hedia](https://juejin.cn/user/2594503171530872)

web前端搬砖工 @ 今日头条3年前

很赞

点赞

回复

[![西兰花伟大炮的头像](https://p26-passport.byteacctimg.com/img/user-avatar/30ce581f7cbc922c8168350f950d92fc~300x300.image)](https://juejin.cn/user/3438928101653255)

[西兰花伟大炮](https://juejin.cn/user/3438928101653255)

前端工程师 @ Xxxx3年前

谁知道调试断点时遇到一个很多的循环，怎么跳过循环？

点赞

1

[![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/mirror-assets/168e09c02889ebc2ba4~tplv-t2oaga2asx-no-mark:24:24:24:24.awebp)](https://juejin.cn/user/1961184475223645)

[JasonXiaoSpace](https://juejin.cn/user/1961184475223645)

3年前

什么意思？那就去掉for里的断点，在for外面后面的第一代码打断点

点赞

回复

[![前端小板凳的头像](https://p6-passport.byteacctimg.com/img/mosaic-legacy/3793/3131589739~300x300.image)](https://juejin.cn/user/3667626521010269)

[前端小板凳![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/3667626521010269)

前端工程师3年前

多谢。学到了

点赞

回复

[![wendyfen的头像](https://p26-passport.byteacctimg.com/img/user-avatar/6facc51ae4698612a9b95bff6b51e02d~300x300.image)](https://juejin.cn/user/1538972009381719)

[wendyfen](https://juejin.cn/user/1538972009381719)

web前端开发工程师3年前

最后一个确实很有用

点赞

回复

[![rocky191的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2017/12/7/16030548a8f02132~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/993614240163630)

[rocky191![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/993614240163630)

前端小菜鸟 @ 创业公司3年前

调试技巧

点赞

回复

[![cocoyuanxile的头像](https://p9-passport.byteacctimg.com/img/user-avatar/afb161b6852219fd6fef185fa403b7a3~300x300.image)](https://juejin.cn/user/4353721774377518)

[cocoyuanxile](https://juejin.cn/user/4353721774377518)

前端小透明3年前

谢谢大佬 学习到了。

点赞

回复

查看全部 39 条回复