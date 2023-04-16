# Webpack 性能优化

在这一的章节中，我不会浪费篇幅给大家讲如何写配置文件。**如果你想学习这方面的内容，那么完全可以去官网学习**。在这部分的内容中，我们会聚焦于以下两个知识点，并且每一个知识点都属于高频考点：

- 有哪些方式可以减少 Webpack 的打包时间
- 有哪些方式可以让 Webpack 打出来的包更小

## 减少 Webpack 打包时间

### 优化 Loader

对于 Loader 来说，影响打包效率首当其冲必属 Babel 了。因为 Babel 会将代码转为字符串生成 AST，然后对 AST 继续进行转变最后再生成新的代码，项目越大，**转换代码越多，效率就越低**。当然了，我们是有办法优化的。

首先我们可以**优化 Loader 的文件搜索范围**

```
module.exports = {
  module: {
    rules: [
      {
        // js 文件才使用 babel
        test: /\.js$/,
        loader: 'babel-loader',
        // 只在 src 文件夹下查找
        include: [resolve('src')],
        // 不会去查找的路径
        exclude: /node_modules/
      }
    ]
  }
}
```

对于 Babel 来说，我们肯定是希望只作用在 JS 代码上的，然后 `node_modules` 中使用的代码都是编译过的，所以我们也完全没有必要再去处理一遍。

当然这样做还不够，我们还可以将 Babel 编译过的文件**缓存**起来，下次只需要编译更改过的代码文件即可，这样可以大幅度加快打包时间

```
loader: 'babel-loader?cacheDirectory=true'
```

### HappyPack

受限于 Node 是单线程运行的，所以 Webpack 在打包的过程中也是单线程的，特别是在执行 Loader 的时候，长时间编译的任务很多，这样就会导致等待的情况。

**HappyPack 可以将 Loader 的同步执行转换为并行的**，这样就能充分利用系统资源来加快打包效率了

```
module: {
  loaders: [
    {
      test: /\.js$/,
      include: [resolve('src')],
      exclude: /node_modules/,
      // id 后面的内容对应下面
      loader: 'happypack/loader?id=happybabel'
    }
  ]
},
plugins: [
  new HappyPack({
    id: 'happybabel',
    loaders: ['babel-loader?cacheDirectory'],
    // 开启 4 个线程
    threads: 4
  })
]
```

### DllPlugin

**DllPlugin 可以将特定的类库提前打包然后引入**。这种方式可以极大的减少打包类库的次数，只有当类库更新版本才有需要重新打包，并且也实现了将公共代码抽离成单独文件的优化方案。

接下来我们就来学习如何使用 DllPlugin

```
// 单独配置在一个文件中
// webpack.dll.conf.js
const path = require('path')
const webpack = require('webpack')
module.exports = {
  entry: {
    // 想统一打包的类库
    vendor: ['react']
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].dll.js',
    library: '[name]-[hash]'
  },
  plugins: [
    new webpack.DllPlugin({
      // name 必须和 output.library 一致
      name: '[name]-[hash]',
      // 该属性需要与 DllReferencePlugin 中一致
      context: __dirname,
      path: path.join(__dirname, 'dist', '[name]-manifest.json')
    })
  ]
}
```

然后我们需要执行这个配置文件生成依赖文件，接下来我们需要使用 `DllReferencePlugin` 将依赖文件引入项目中

```
// webpack.conf.js
module.exports = {
  // ...省略其他配置
  plugins: [
    new webpack.DllReferencePlugin({
      context: __dirname,
      // manifest 就是之前打包出来的 json 文件
      manifest: require('./dist/vendor-manifest.json'),
    })
  ]
}
```

### 代码压缩

在 Webpack3 中，我们一般使用 `UglifyJS` 来压缩代码，但是这个是单线程运行的，为了加快效率，我们可以使用 `webpack-parallel-uglify-plugin` 来并行运行 `UglifyJS`，从而提高效率。

在 Webpack4 中，我们就不需要以上这些操作了，只需要将 `mode` 设置为 `production` 就可以默认开启以上功能。代码压缩也是我们必做的性能优化方案，当然我们不止可以压缩 JS 代码，还可以压缩 HTML、CSS 代码，并且在压缩 JS 代码的过程中，我们还可以通过配置实现比如删除 `console.log` 这类代码的功能。

### 一些小的优化点

我们还可以通过一些小的优化点来加快打包速度

- `resolve.extensions`：用来表明文件后缀列表，默认查找顺序是 `['.js', '.json']`，如果你的导入文件没有添加后缀就会按照这个顺序查找文件。我们应该尽可能减少后缀列表长度，然后将出现频率高的后缀排在前面
- `resolve.alias`：可以通过别名的方式来映射一个路径，能让 Webpack 更快找到路径
- `module.noParse`：如果你确定一个文件下没有其他依赖，就可以使用该属性让 Webpack 不扫描该文件，这种方式对于大型的类库很有帮助

## 减少 Webpack 打包后的文件体积

> 注意：该内容也属于性能优化领域。

### 按需加载

想必大家在开发 SPA 项目的时候，项目中都会存在十几甚至更多的路由页面。如果我们将这些页面全部打包进一个 JS 文件的话，虽然将多个请求合并了，但是同样也加载了很多并不需要的代码，耗费了更长的时间。那么为了首页能更快地呈现给用户，我们肯定是希望首页能加载的文件体积越小越好，**这时候我们就可以使用按需加载，将每个路由页面单独打包为一个文件**。当然不仅仅路由可以按需加载，对于 `loadash` 这种大型类库同样可以使用这个功能。

按需加载的代码实现这里就不详细展开了，因为鉴于用的框架不同，实现起来都是不一样的。当然了，虽然他们的用法可能不同，但是底层的机制都是一样的。都是当使用的时候再去下载对应文件，返回一个 `Promise`，当 `Promise` 成功以后去执行回调。

### Scope Hoisting

**Scope Hoisting 会分析出模块之间的依赖关系，尽可能的把打包出来的模块合并到一个函数中去。**

比如我们希望打包两个文件

```
// test.js
export const a = 1
// index.js
import { a } from './test.js'
```

对于这种情况，我们打包出来的代码会类似这样

```
[
  /* 0 */
  function (module, exports, require) {
    //...
  },
  /* 1 */
  function (module, exports, require) {
    //...
  }
]
```

但是如果我们使用 Scope Hoisting 的话，代码就会尽可能的合并到一个函数中去，也就变成了这样的类似代码

```
[
  /* 0 */
  function (module, exports, require) {
    //...
  }
]
```

这样的打包方式生成的代码明显比之前的少多了。如果在 Webpack4 中你希望开启这个功能，只需要启用 `optimization.concatenateModules` 就可以了。

```
module.exports = {
  optimization: {
    concatenateModules: true
  }
}
```

### Tree Shaking

**Tree Shaking 可以实现删除项目中未被引用的代码**，比如

```
// test.js
export const a = 1
export const b = 2
// index.js
import { a } from './test.js'
```

对于以上情况，`test` 文件中的变量 `b` 如果没有在项目中使用到的话，就不会被打包到文件中。

如果你使用 Webpack 4 的话，开启生产环境就会自动启动这个优化功能。

## 小结

在这一章节中，我们学习了如何使用 Webpack 去进行性能优化以及如何减少打包时间。

Webpack 的版本更新很快，各个版本之间实现优化的方式可能都会有区别，所以我没有使用过多的代码去展示如何实现一个功能。**这一章节的重点是学习到我们可以通过什么方式去优化，具体的代码实现可以查找具体版本对应的代码即可。**

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（18）

[![一咻的头像](https://p3-passport.byteacctimg.com/img/user-avatar/f6ae15df125a3f55d2fe2a9838e0c43c~300x300.image)](https://juejin.cn/user/624178334545704)

[一咻![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/624178334545704)

web前端6月前

[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)juejin.cn](https://juejin.cn/post/6981841495338778660) 最佳实践

1

回复

[![yuuki爱学习的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2928754706893933)

[yuuki爱学习](https://juejin.cn/user/2928754706893933)

10月前

是lodash

点赞

回复

[![tony1990atgz的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/676954892665181)

[tony1990atgz](https://juejin.cn/user/676954892665181)

1232年前

可以给一个比较完整的 demo 吗

2

回复

[![买个萝卜的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1961184474695790)

[买个萝卜](https://juejin.cn/user/1961184474695790)

2年前

ParallelUglifyPlugin 现在都用这个打包了，支持更好一点

点赞

回复

[![小丸子8292的头像](https://p26-passport.byteacctimg.com/img/user-avatar/114d76e029099936e4c5c2fc8457a6d2~300x300.image)](https://juejin.cn/user/4248168660469479)

[小丸子8292](https://juejin.cn/user/4248168660469479)

前端开发2年前

减少文件体积应该再加一个开启gzip压缩

3

回复

[![承重墙右的头像](https://p9-passport.byteacctimg.com/img/user-avatar/95a4c0cf24ca1fce40e57e11b094955d~300x300.image)](https://juejin.cn/user/3808364008337550)

[承重墙右](https://juejin.cn/user/3808364008337550)

FE @ SF3年前

一..一目了然

1

1

[![img](https://p9-passport.byteacctimg.com/img/mosaic-legacy/3797/2889309425~300x300.image)](https://juejin.cn/user/1187128286401934)

[你在想什么](https://juejin.cn/user/1187128286401934)

2年前

一..一

点赞

回复

[![zexiplus的头像](https://p9-passport.byteacctimg.com/img/user-avatar/33af61b301919da939e6122af78d1521~300x300.image)](https://juejin.cn/user/3227821869640990)

[zexiplus](https://juejin.cn/user/3227821869640990)

web前端 @ Coding3年前

减少文件体积应该再加一个开启gzip压缩

14

回复

[![华尔兹绚丽的头像](https://p9-passport.byteacctimg.com/img/user-avatar/af7be715968568e294bb8bd61d6cdf8f~300x300.image)](https://juejin.cn/user/588993962184862)

[华尔兹绚丽](https://juejin.cn/user/588993962184862)

前端开发 @ 闪送3年前

webpack这个给不了demo,只能项目里边思考边实践，这样才能看出优化后的效果

2

回复

[![Slim_GutsOverFear的头像](https://p26-passport.byteacctimg.com/img/user-avatar/9f1fa6d2342162b991cee82bb50d21bb~300x300.image)](https://juejin.cn/user/78820565845639)

[Slim_GutsOverFear](https://juejin.cn/user/78820565845639)

FE @ 腾讯3年前

最新的webpack配置里面推荐使用optimization.splitChunks，在entry里面明确的提示不要再添加vendors了

4

回复

[![FreePotato的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/4/16328a47a9447189~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/1926000099743022)

[FreePotato![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1926000099743022)

前端工程师 @ 天上人间3年前

优化 Loader 的部分，有了include 是不是可以就不要exclude了

1

1

[![img](https://p3-passport.byteacctimg.com/img/user-avatar/5b40d96004a8df7eb0621231fcc122c3~300x300.image)](https://juejin.cn/user/2119514150673432)

[我是谁我在哪aaa](https://juejin.cn/user/2119514150673432)

10月前

树形结构include了根再exclude子树.

点赞

回复

[![苏笑小的头像](https://p9-passport.byteacctimg.com/img/user-avatar/cacbc98e744761d8dfe59d0f54233980~300x300.image)](https://juejin.cn/user/2999123451317582)

[苏笑小![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2999123451317582)

Bug开发工程师3年前

可以给一个比较完整的 demo 吗

点赞

回复

[![ZedCoding的头像](https://p9-passport.byteacctimg.com/img/user-avatar/5a7bc1423ba1db61723cc44924335708~300x300.image)](https://juejin.cn/user/3790771822532110)

[ZedCoding](https://juejin.cn/user/3790771822532110)

前端 @ 杭州高级切图工作室3年前

可以给一个比较完整的 demo 吗 太零碎了

1

回复

[![小丸子8292的头像](https://p26-passport.byteacctimg.com/img/user-avatar/114d76e029099936e4c5c2fc8457a6d2~300x300.image)](https://juejin.cn/user/4248168660469479)

[小丸子8292](https://juejin.cn/user/4248168660469479)

前端开发3年前

可以给一个比较完整的 demo 吗

2

回复

[![Tia_Brother的头像](https://p9-passport.byteacctimg.com/img/user-avatar/cafba92584ab6448c6cb410e1b021bab~300x300.image)](https://juejin.cn/user/4142615541058718)

[Tia_Brother](https://juejin.cn/user/4142615541058718)

前端开发工程师 @ 徐州广联3年前

可以给一个比较完整的 demo 吗

4

回复

[![不曾亏欠的头像](https://p6-passport.byteacctimg.com/img/user-avatar/c6e19d0306733abb5a249e8a372e27cb~300x300.image)](https://juejin.cn/user/729731451069245)

[不曾亏欠![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/729731451069245)

胶水工程师3年前

受益匪浅