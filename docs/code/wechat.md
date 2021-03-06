# 2 小程序代码组成

## 2.1 JSON配置

## 2.2 WXML模板

和HTML很像

大小写敏感

#### 标签

- 文本标签<text>
- 图片标签<image>

### 2.2.2 数据绑定

##### 实例

```html
<!--pages/wxml/index.wxml-->
<text>当前时间：{{time}}</text>
```

打开 pages/wxml/index.js 文件，在 data 的大括号中加入：`time: (new Date()).toString()`。

```javascript
// pages/wxml/index.js
Page({
  /**
   * 页面的初始数据
   */
  data: {
    time: (new Date()).toString()
  },
})
```

WXML 通过 {{变量名}} 来绑定 WXML 文件和对应的 JavaScript 文件中的 data 对象属性。

属性值也可以动态的去改变，有所不同的是，属性值必须被包裹在双引号中，如下：

##### 代码清单2-8 属性值的绑定

```html
<!-- 正确的写法 -->
<text data-test="{{test}}"> hello world</text>


<!-- 错误的写法  -->
<text data-test={{test}}> hello world </text >
```

变量名是大小写敏感的

没有被定义的变量的或者是被设置为 undefined 的变量不会被同步到 wxml 中

### 2.2.3 逻辑语法

三元运算：

```html
<!-- 根据 a 的值是否等于 10 在页面输出不同的内容 -->
<text>{{ a === 10? "变量 a 等于10": "变量 a 不等于10"}}</text>
```

算数运算：

```html
<!--
{ a: 1,  b: 2, c: 3 }
-->


<view> {{a + b}} + {{c}} + d </view>


<!-- 输出 3 + 3 + d -->
```

类似于算数运算，还支持字符串的拼接，如代码2-11所示。

代码清单2-11 字符串的拼接

```html
<!--
{ name: 'world' }
-->


<view>{{"hello " + name}}</view>


<!-- 输出 hello world -->
```

{{ }}中还可以直接放置数字、字符串或者是数组

代码清单2-12 常量

```html
<text>{{[1,2,3]}}</text>

<!-- 输出 1,2,3 -->



<text>{{"hello world"}}</text>

<!-- 输出 hello world -->
```

### 2.2.4 条件逻辑

WXML 中，使用 wx:if="{{condition}}" 来判断**是否需要渲染该代码块**：

```html
<view wx:if="{{condition}}"> True </view>
```

使用 wx:elif 和 wx:else 来添加一个 else 块：

```html
<view wx:if="{{length > 5}}"> 1 </view>
<view wx:elif="{{length > 2}}"> 2 </view>
<view wx:else> 3 </view>
```

因为 wx:if 是一个控制属性，需要将它添加到一个标签上。如果要一次性判断多个组件标签，可以使用一个 `<block/>` 标签将多个组件包装起来，并在上边使用 wx:if 控制属性。

```html
<block wx:if="{{true}}">
  <view> view1 </view>
  <view> view2 </view>
</block>
```

### 2.2.5 列表渲染

在组件上使用 wx:for 控制属性绑定一个数组，即可使用数组中各项的数据重复渲染该组件。

默认数组的当前项的下标变量名默认为 index，数组当前项的变量名默认为 item

代码清单2-13 列表渲染示例

```html
<!-- array 是一个数组 -->
<view wx:for="{{array}}">
  {{index}}: {{item.message}}
</view>

<!-- 对应的脚本文件
Page({
  data: {
    array: [{
      message: 'foo',
    }, {
      message: 'bar'
    }]
  }
})
-->
```

**使用 wx:for-item 指定数组当前元素的变量名，使用 wx:for-index 指定数组当前下标的变量名：**

```html
<view wx:for="{{array}}" wx:for-index="idx" wx:for-item="itemName">
  {{idx}}: {{itemName.message}}
</view>
```

类似 `block wx:if` ，也可以将 `wx:for` 用在 `<block/>` 标签上，以渲染一个包含多节点的结构块。例如：

```html
<block wx:for="{{[1, 2, 3]}}">
  <view> {{index}}: </view>
  <view> {{item}} </view>
</block>
```

如果列表中项目的位置会动态改变或者有新的项目添加到列表中，并且希望列表中的项目保持自己的特征和状态（如 `<input/>` 中的输入内容， `<switch/>` 的选中状态），需要使用 `wx:key` 来指定列表中项目的唯一的标识符。

`wx:key` 的值以两种形式提供：

1. 字符串，代表在 for 循环的 array 中 item 的某个 property，该 property 的值需要是列表中唯一的字符串或数字，且不能动态改变。
2. 保留关键字 this 代表在 for 循环中的 item 本身，这种表示需要 item 本身是一个唯一的字符串或者数字，如：

当数据改变触发渲染层重新渲染的时候，会校正带有 key 的组件，框架会确保他们被重新排序，而不是重新创建，以确保使组件保持自身的状态，并且提高列表渲染时的效率。

代码清单2-14 使用 wx:key 示例（WXML）

```html
<switch wx:for="{{objectArray}}" wx:key="unique" > {{item.id}} </switch>
<button bindtap="switch"> Switch </button>
<button bindtap="addToFront"> Add to the front </button>


<switch wx:for="{{numberArray}}" wx:key="*this" > {{item}} </switch>
<button bindtap="addNumberToFront"> Add Number to the front </button>
```

代码清单2-15 使用 wx:key 示例（JavaScript）

```javascript
Page({
  data: {
    objectArray: [
      {id: 5, unique: 'unique_5'},
      {id: 4, unique: 'unique_4'},
      {id: 3, unique: 'unique_3'},
      {id: 2, unique: 'unique_2'},
      {id: 1, unique: 'unique_1'},
      {id: 0, unique: 'unique_0'},
    ],
    numberArray: [1, 2, 3, 4]
  },
  switch: function(e) {
    const length = this.data.objectArray.length
    for (let i = 0; i < length; ++i) {
      const x = Math.floor(Math.random() * length)
      const y = Math.floor(Math.random() * length)
      const temp = this.data.objectArray[x]
      this.data.objectArray[x] = this.data.objectArray[y]
      this.data.objectArray[y] = temp
    }
    this.setData({
      objectArray: this.data.objectArray
    })
  },
  addToFront: function(e) {
    const length = this.data.objectArray.length
    this.data.objectArray = [{id: length, unique: 'unique_' + length}].concat(this.data.objectArray)
    this.setData({
      objectArray: this.data.objectArray
    })
  },
  addNumberToFront: function(e){
    this.data.numberArray = [ this.data.numberArray.length + 1 ].concat(this.data.numberArray)
    this.setData({
      numberArray: this.data.numberArray
    })
  }
})
```

### 2.2.6 模板

WXML提供模板（template），可以在模板中定义代码片段，然后在不同的地方调用。使用 name 属性，作为模板的名字。然后在 `<template/>` 内定义代码片段，如：

代码清单2-16 定义模板

```html
<template name="msgItem">
  <view>
    <text> {{index}}: {{msg}} </text>
    <text> Time: {{time}} </text>
  </view>
</template>
```

使用 is 属性，声明需要的使用的模板，然后将模板所需要的 data 传入，如代码2-17所示。

代码清单2-17 模板使用示例

```html
<!--
item: {
  index: 0,
  msg: 'this is a template',
  time: '2016-06-18'
}
-->


<template name="msgItem">
  <view>
    <text> {{index}}: {{msg}} </text>
    <text> Time: {{time}} </text>
  </view>
</template>


<template is="msgItem" data="{{...item}}"/>

<!-- 输出
0: this is a template Time: 2016-06-18
-->
```

is可以动态决定具体需要渲染哪个模板，如代码2-18所示。

代码清单2-18 动态使用模板

```html
<template name="odd">
  <view> odd </view>
</template>


<template name="even">
  <view> even </view>
</template>


<block wx:for="{{[1, 2, 3, 4, 5]}}">
  <template is="{{item % 2 == 0 ? 'even' : 'odd'}}"/>
</block>



<!-- 输出
odd
even
odd
even
odd
-->
```

### 2.2.7 引用

WXML 提供两种文件引用方式import和include。

import 可以在该文件中使用目标文件定义的 template，如：

在 item.wxml 中定义了一个叫 item的 template ：

```html
<!-- item.wxml -->
<template name="item">
  <text>{{text}}</text>
</template>
```

在 index.wxml 中引用了 item.wxml，就可以使用 item模板：

```html
<import src="item.wxml"/>

<template is="item" data="{{text: 'forbar'}}"/>
```

需要注意的是 import 有作用域的概念，即只会 import 目标文件中定义的 template，而不会 import 目标文件中 import 的 template，简言之就是 import 不具有递归的特性。

例如：C 引用 B，B 引用A，在C中可以使用B定义的 template，在B中可以使用A定义的 template ，但是C不能使用A定义的template ，如代码2-19、代码2-20、代码2-21所示。

代码清单2-19 模板 A

```html
<!-- A.wxml -->
<template name="A">
  <text> A template </text>
</template>
```

代码清单2-20 模板 B

```html
<!-- B.wxml -->
<import src="a.wxml"/>

<template name="B">
  <text> B template </text>
</template>
```

代码清单2-21 模板 C

```html
<!-- C.wxml -->
<import src="b.wxml"/>

<template is="A"/>  <!-- 这里将会触发一个警告，因为 b 中并没有定义模板 A -->

<template is="B"/>
```

include 可以将目标文件中除了 `<template/> <wxs/>` 外的整个代码引入，相当于是拷贝到 include 位置，如代码2-22、代码2-23、代码2-24所示。

代码清单2-22 index.wxml

```html
<!-- index.wxml -->
<include src="header.wxml"/>

<view> body </view>

<include src="footer.wxml"/>
```

代码清单2-23 header.wxml

```html
<!-- header.wxml -->
<view> header </view>
```

代码清单2-24 footer.wxml

```html
<!-- footer.wxml -->
<view> footer </view>
```

### 2.2.8 共同属性

所有wxml 标签都支持的属性称之为共同属性，如表2-1所示。

表2-1 共同属性

| **属性名**   | **类型**     | **描述**       | **注解**                                 |
| :----------- | :----------- | :------------- | ---------------------------------------- |
| id           | String       | 组件的唯一标识 | 整个页面唯一                             |
| class        | String       | 组件的样式类   | 在对应的 WXSS 中定义的样式类             |
| style        | String       | 组件的内联样式 | 可以动态设置的内联样式                   |
| hidden       | Boolean      | 组件是否显示   | 所有组件默认显示                         |
| data-*       | Any          | 自定义属性     | 组件上触发的事件时，会发送给事件处理函数 |
| bind*/catch* | EventHandler | 组件的事件     |                                          |



## 2.3 WXSS 样式

WXSS（WeiXin Style Sheets）是一套用于小程序的样式语言，用于描述WXML的组件样式，也就是视觉上的效果。

WXSS与Web开发中的CSS类似。为了更适合小程序开发，WXSS对CSS做了一些补充以及修改。

### 2.3.1 文件组成

项目公共样式：根目录中的app.wxss为项目公共样式，它会被注入到小程序的每个页面。页面样式：与app.json注册过的页面同名且位置同级的WXSS文件。比如图2-8 app.json注册了pages/rpx/index页面，那pages/rpx/index.wxss为页面pages/rpx/index.wxml的样式。

其它样式：其它样式可以被项目公共样式和页面样式引用，引用方法查看本章中的2.3.3小节。

在小程序开发中，开发者不需要像Web开发那样去优化样式文件的请求数量，只需要考虑代码的组织即可。样式文件最终会被编译优化，具体的编译原理我们留在后面章节再做介绍。

### 2.3.2 尺寸单位

在WXSS中，引入了rpx（responsive pixel）尺寸单位。引用新尺寸单位的目的是，适配不同宽度的屏幕，开发起来更简单。

同一个元素，在不同宽度的屏幕下，如果使用px为尺寸单位，有可能造成页面留白过多。

小程序编译后，rpx会做一次px换算。换算是以375个物理像素为基准，也就是在一个宽度为375物理像素的屏幕下，1rpx = 1px。

举个例子：iPhone6屏幕宽度为375px，共750个物理像素，那么1rpx = 375 / 750 px = 0.5px。

### 2.3.3 WXSS引用

在CSS中，开发者可以这样引用另一个样式文件：`@import url('./test_0.css')`

这种方法在请求上不会把test_0.css合并到index.css中，也就是请求index.css的时候，会多一个test_0.css的请求。

在小程序中，我们依然可以实现样式的引用，样式引用是这样写：

```css
@import './test_0.wxss'
```

由于WXSS最终会被编译打包到目标文件中，用户只需要下载一次，在使用过程中不会因为样式的引用而产生多余的文件请求。

### 2.3.4 内联样式

WXSS内联样式与Web开发一致:

```html
<!--index.wxml-->

<!--内联样式-->
<view style="color: red; font-size: 48rpx"></view>
```

小程序支持动态更新内联样式：

```html
<!--index.wxml-->

<!--可动态变化的内联样式-->
<!--
{
  eleColor: 'red',
  eleFontsize: '48rpx'
}
-->
<view style="color: {{eleColor}}; font-size: {{eleFontsize}}"></view>
```

### 2.3.5 选择器

目前支持的选择器如表2-2所示。

表2-2 小程序WXSS支持的选择器

| **类型**     | **选择器** | **样例**      | **样例描述**                                   |
| :----------- | :--------- | :------------ | :--------------------------------------------- |
| id选择器     | #id        | #firstname    | 选择拥有 id="firstname" 的组件                 |
| 元素选择器   | element    | view checkbox | 选择所有文档的 view 组件和所有的 checkbox 组件 |
| 伪元素选择器 | ::after    | view::after   | 在 view 组件后边插入内容                       |
| 类选择器     | .class     | .intro        | 选择所有拥有 class="intro" 的组件              |
| 伪元素选择器 | ::before   | view::before  | 在 view 组件前边插入内容                       |

内联>id>class>element

### 2.3.6 官方样式库

https://github.com/Tencent/weui-wxss

## 2.4 JavaScript 脚本

### 2.4.1 ECMAScript

在大部分开发者看来，ECMAScript和JavaScript表达的是同一种含义，但是严格的说，两者的意义是不同的。ECMAScript是一种由Ecma国际通过ECMA-262标准化的脚本程序设计语言， JavaScript 是 ECMAScript 的一种实现。理解 JavaScript 是 ECMAScript 一种实现后，可以帮助开发者理解小程序中的 JavaScript同浏览器中的 JavaScript 以及 NodeJS 中的 JavaScript 是不相同的。

ECMA-262 规定了 ECMAScript 语言的几个重要组成部分：

1. 语法
2. 类型
3. 语句
4. 关键字
5. 操作符
6. 对象

浏览器中JavaScript

- ECMAScript
- DOM
- BOM

浏览器中的JavaScript 是由 ECMAScript 和 BOM（浏览器对象模型）以及 DOM（文档对象模型）组成的，Web前端开发者会很熟悉这两个对象模型，它使得开发者可以去操作浏览器的一些表现，比如修改URL、修改页面呈现、记录数据等等。

NodeJS中JavaScript

- ECMAScript
- NPM
- Native

NodeJS中的JavaScript 是由 ECMAScript 和 NPM以及Native模块组成，NodeJS的开发者会非常熟悉 NPM 的包管理系统，通过各种拓展包来快速的实现一些功能，同时通过使用一些原生的模块例如 FS、HTTP、OS等等来拥有一些语言本身所不具有的能力。

小程序中 JavaScript 构成

- ECMAScript
- 小程序框架
- 小程序API

小程序中的 JavaScript 是由ECMAScript 以及小程序框架和小程序 API 来实现的。同浏览器中的JavaScript 相比没有 BOM 以及 DOM 对象，所以类似 JQuery、Zepto这种浏览器类库是无法在小程序中运行起来的，同样的缺少 Native 模块和NPM包管理的机制，小程序中无法加载原生库，也无法直接使用大部分的 NPM 包。

### 2.4.2 小程序的执行环境

明白了小程序中的 JavaScript 同浏览器以及NodeJS有所不同后，开发者还需要注意到另外一个问题，不同的平台的小程序的脚本执行环境也是有所区别的。

小程序目前可以运行在三大平台：

1. iOS平台，包括iOS9、iOS10、iOS11
2. Android平台
3. 小程序IDE

这种区别主要是体现三大平台实现的 ECMAScript 的标准有所不同。截止到当前一共有七个版本的ECMAScript 标准，目前开发者大部分使用的是 ECMAScript 5 和 ECMAScript 6 的标准，但是在小程序中， iOS9和iOS10 所使用的运行环境并没有完全的兼容到 ECMAScript 6 标准，一些 ECMAScript 6 中规定的语法和关键字是没有的或者同标准是有所不同的，例如：

1. 箭头函数
2. let const
3. 模板字符串
4. …

所以一些开发者会发现有些代码在旧的手机操作系统上出现一些语法错误。为了帮助开发者解决这类问题，小程序IDE提供语法转码工具帮助开发者，将 ECMAScript 6代码转为 ECMAScript 5代码，从而在所有的环境都能得到很好的执行。开发者需要在项目设置中，勾选 ES6 转 ES5 开启此功能。

### 2.4.3 模块化

浏览器中，所有 JavaScript 是在运行在同一个作用域下的，定义的参数或者方法可以被后续加载的脚本访问或者改写。同浏览器不同，小程序中可以将任何一个JavaScript 文件作为一个模块，通过module.exports 或者 exports 对外暴露接口。

请看是一个简单模块示例，B.js 引用模块A，并使用A暴露的multiplyBy2方法完成一个变量乘以 2 的操作。

代码清单2-26 模块示例

```javascript
// moduleA.js
module.exports = function( value ){
  return value * 2;
}
```

代码清单2-27 引用模块A

```javascript
// B.js

// 在B.js中引用模块A
var multiplyBy2 = require('./moduleA')
var result = multiplyBy2(4)
```

代码清单2-28 在需要使用这些模块的文件中，使用 require(path) 将公共代码引入

```javascript
var common = require('common.js')
Page({
  helloMINA: function() {
    common.sayHello('MINA')
  },
  goodbyeMINA: function() {
    common.sayGoodbye('MINA')
  }
})
```

### 2.4.4 脚本的执行顺序

浏览器中，脚本严格按照加载的顺序执行，如代码2-29所示。

代码清单2-29 浏览器中的脚本

```html
<html>
<head>
  <!-- a.js
  console.log('a.js')
   -->
  <script src ="a.js"></script>
  <script>
    console.log('inline script')
  </script>

  <!-- b.js
  console.log('b.js')
   -->
  <script src ="b.js"></script>
</head>
</html>
```

以上代码的输出是：

```
a.js

inline script

b.js
```

而在小程序中的脚本执行顺序有所不同。小程序的执行的入口文件是 app.js 。并且会根据其中 require 的模块顺序决定文件的运行顺序，代码2-30是一个 app.js 示例。

代码清单2-30 app.js

```javascript
/* a.js
console.log('a.js')
*/
var a = require('./a.js')
console.log('app.js')

/* b.js
console.log('b.js')
*/
var b = require('./b.js')
```

以上代码的输出顺序是：

```
a.js

app.js

b.js
```

当 app.js 执行结束后，小程序会按照开发者在 app.json 中定义的 pages 的顺序，逐一执行。如代码2-31所示。

代码清单2-31 app.json 文件

```json
{
  "pages": [
    "pages/index/index",
    "pages/log/log",
    "pages/result/result"
  ],
  "window": {}
}
```

代码清单2-32 app.js文件

```javascript
// app.js
console.log('app.js')
```

代码清单2-33 pages/index/index.js 文件

```javascript
// pages/index/index
console.log('pages/index/index')
```

代码清单2-34 page/log/log.js 文件

```javascript
// pages/log/log
console.log('pages/log/log')
```

代码清单2-35 page/result/result.js 文件

```javascript
// pages/result/result
console.log('pages/result/result')
```

以上文件执行后输出的结果如下：

```
app.js

pages/index/index

pages/log/log

pages/result/result
```

### 2.4.5 作用域

同浏览器中运行的脚本文件有所不同，小程序的脚本的作用域同 NodeJS 更为相似。

在文件中声明的变量和函数只在该文件中有效，不同的文件中可以声明相同名字的变量和函数，不会互相影响，如代码2-36、代码2-37所示。

代码清单2-36 在脚本 a.js 中定义局部变量

```javascript
// a.js
// 定义局部变量
var localValue = 'a'
```

代码清单2-37 在脚本 b.js 中无法访问 a.js 定义的变量

```javascript
// b.js
// 定义局部变量
console.log(localValue) // 触发一个错误 b.js中无法访问 a.js 中定义的变量
```

当需要使用全局变量的时，通过使用**全局函数 getApp()** 获取全局的实例，并设置相关属性值，来达到设置全局变量的目的，如代码2-38、代码2-39所示。

代码清单2-38 在脚本 a.js 中设置全局变量

```javascript
// a.js
// 获取全局变量
var global = getApp()
global.globalValue = 'globalValue'
```

代码清单2-39 在脚本 b.js 中访问 a.js 定义的全局变量

```javascript
// b.js
// 访问全局变量
var global = getApp()
console.log(global.globalValue) // 输出 globalValue
```

需要注意的是，上述示例只有在 a.js 比 b.js 先执行才有效，当需要保证全局的数据可以在任何文件中安全的被使用到，那么可以在 App() 中进行设置，如代码2-40、代码2-41、代码2-42所示。

代码清单2-40 定义全局变量

```javascript
// app.js
App({
  globalData: 1
})
```

代码清单2-41 获取以及修改 global 变量的方法

```javascript
// a.js
// 局部变量
var localValue = 'a'

// 获取 global 变量
var app = getApp()

// 修改 global 变量
app.globalData++  // 执行后 globalData 数值为 2
```

代码清单2-42 获取 global 变量

```javascript
// b.js
// 定义另外的局部变量，并不会影响 a.js 中文件变量
var localValue = 'b'

// 如果先执行了 a.js 这里的输出应该是 2
console.log(getApp().globalData)
```

# 3 理解小程序宿主环境

## 3.1 渲染层和逻辑层

小程序的运行环境分成渲染层和逻辑层，第2章提到过 WXML 模板和 WXSS 样式工作在渲染层，JS 脚本工作在逻辑层。小程序的渲染层和逻辑层分离是经过很多考虑得出来的模型，在第6章我们会详细阐述这个模型背后的原理以及产生的问题。在本章我们会先介绍这个模型的基本工作方式。

### 3.1.1 渲染“Hello World”页面

我们看看小程序是如何把脚本里边的数据渲染在界面上的。
WXML模板使用 view 标签，其子节点用 {{ }} 的语法绑定一个 msg 的变量，如代码清单3-1所示。
代码清单3-1 渲染“Hello World”WXML代码

```html
<view>{{ msg }}</view>
```

在 JS 脚本使用 this.setData 方法把 msg 字段设置成 “Hello World” ，如代码清单3-2所示。
代码清单3-2 渲染“Hello World”JS脚本

```javascript
Page({
  onLoad: function () {
    this.setData({ msg: 'Hello World' })
  }
})
```

从这个例子我们可以看到3个点：
1.渲染层和数据相关。
2.逻辑层负责产生、处理数据。
3.逻辑层通过 Page 实例的 setData 方法传递数据到渲染层。
关于第1点，涉及了“数据驱动”的概念，我们会在3.1.3节详细讨论，我们现在先看看第3点涉及的“通信模型”。

### 3.1.2 通信模型

小程序的渲染层和逻辑层分别由2个线程管理：渲染层的界面使用了WebView 进行渲染；逻辑层采用JsCore线程运行JS脚本。一个小程序存在多个界面，所以渲染层存在多个WebView线程，这两个线程的通信会经由微信客户端（下文中也会采用Native来代指微信客户端）做中转，逻辑层发送网络请求也经由Native转发，小程序的通信模型如图3-1所示。







## 3.2 程序与页面

从逻辑组成来说，一个小程序是由多个“页面”组成的“程序”。这里要区别一下“小程序”和“程序”的概念，往往我们需要在“程序”启动或者退出的时候存储数据或者在“页面”显示或者隐藏的时候做一些逻辑处理，了解程序和页面的概念以及它们的生命周期是非常重要的。

### 3.2.1 程序

“小程序”指的是产品层面的程序，而“程序”指的是代码层面的程序实例，为了避免误解，下文采用App来代替代码层面的“程序”概念。

#### 1. 程序构造器App()

宿主环境提供了 App() 构造器用来注册一个程序App，需要留意的是App() 构造器必须写在项目根目录的app.js里，App实例是单例对象，在其他JS脚本中可以使用宿主环境提供的 getApp() 来获取程序实例。
代码清单3-3 getApp() 获取App实例

```javascript
// other.js
var appInstance = getApp()
```

App() 的调用方式如代码清单3-4所示，App构造器接受一个Object参数，参数说明如表3-1所示，其中onLaunch / onShow / onHide 三个回调是App实例的生命周期函数，我们会在后文展开；onError我们暂时不在本章展开，我们会在第8章里详细讨论；App的其他参数我们也放在后文进行展开。
代码清单3-4 App构造器

```javascript
App({
  onLaunch: function(options) {},
  onShow: function(options) {},
  onHide: function() {},
  onError: function(msg) {},
  globalData: 'I am global data'
})
```

表3-1 App构造器的参数

| 参数属性 | 类型     | 描述                                                         |
| :------- | :------- | :----------------------------------------------------------- |
| onLaunch | Function | 当小程序初始化完成时，会触发 onLaunch（全局只触发一次）      |
| onShow   | Function | 当小程序启动，或从后台进入前台显示，会触发 onShow            |
| onHide   | Function | 当小程序从前台进入后台，会触发 onHide                        |
| onError  | Function | 当小程序发生脚本错误，或者 API 调用失败时，会触发 onError 并带上错误信息 |
| 其他字段 | 任意     | 可以添加任意的函数或数据到 Object 参数中，在App实例回调用 this 可以访问 |

#### 2. 程序的生命周期和打开场景

初次进入小程序的时候，微信客户端初始化好宿主环境，同时从网络下载或者从本地缓存中拿到小程序的代码包，把它注入到宿主环境，初始化完毕后，微信客户端就会给App实例派发onLaunch事件，App构造器参数所定义的onLaunch方法会被调用。
进入小程序之后，用户可以点击右上角的关闭，或者按手机设备的Home键离开小程序，此时小程序并没有被直接销毁，我们把这种情况称为“小程序进入后台状态”，App构造器参数所定义的onHide方法会被调用。
当再次回到微信或者再次打开小程序时，微信客户端会把“后台”的小程序唤醒，我们把这种情况称为“小程序进入前台状态”，App构造器参数所定义的onShow方法会被调用。
我们可以看到，App的生命周期是由微信客户端根据用户操作主动触发的。为了避免程序上的混乱，我们不应该从其他代码里主动调用App实例的生命周期函数。
在微信客户端中打开小程序有很多途径：从群聊会话里打开，从小程序列表中打开，通过微信扫一扫二维码打开，从另外一个小程序打开当前小程序等，针对不同途径的打开方式，小程序有时需要做不同的业务处理，所以微信客户端会把打开方式带给onLaunch和onShow的调用参数options，示例代码以及详细参数如代码清单3-5和表3-2所示。需要留意小程序的宿主环境在迭代更新过程会增加不少打开场景，因此要获取最新的场景值说明请查看官方文档：https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/app.html。

代码清单3-5 onLaunch和onShow带参数

```javascript
App({
  onLaunch: function(options) { console.log(options) },
  onShow: function(options) { console.log(options) }
})
```

表3-2 onLaunch,onShow参数

| 字段                   | 类型   | 描述                                                    |
| :--------------------- | :----- | :------------------------------------------------------ |
| path                   | String | 打开小程序的页面路径                                    |
| query                  | Object | 打开小程序的页面参数query                               |
| scene                  | Number | 打开小程序的场景值，详细场景值请参考小程序官方文档      |
| shareTicket            | String | shareTicket，详见小程序官方文档                         |
| referrerInfo           | Object | 当场景为由从另一个小程序或公众号或App打开时，返回此字段 |
| referrerInfo.appId     | String | 来源小程序或公众号或App的 appId，详见下方说明           |
| referrerInfo.extraData | Object | 来源小程序传过来的数据，scene=1037或1038时支持          |

表3-3 以下场景支持返回 referrerInfo.appId

| 场景值 | 场景               | appId信息含义                         |
| :----- | :----------------- | :------------------------------------ |
| 1020   | 公众号 profile     | 页相关小程序列表 返回来源公众号 appId |
| 1035   | 公众号自定义菜单   | 返回来源公众号 appId                  |
| 1036   | App 分享消息卡片   | 返回来源应用 appId                    |
| 1037   | 小程序打开小程序   | 返回来源小程序 appId                  |
| 1038   | 从另一个小程序返回 | 返回来源小程序 appId                  |
| 1043   | 公众号模板消息     | 返回来源公众号 appId                  |

#### 3. 小程序全局数据

我们在 3.1.2节说到小程序的JS脚本是运行在JsCore的线程里，小程序的每个页面各自有一个WebView线程进行渲染，所以小程序切换页面时，小程序逻辑层的JS脚本运行上下文依旧在同一个JsCore线程中。
在上文中说道App实例是单例的，因此不同页面直接可以通过App实例下的属性来共享数据。App构造器可以传递其他参数作为全局属性以达到全局共享数据的目的。
代码清单3-6 小程序全局共享数据

```javascript
// app.js
App({
  globalData: 'I am global data' // 全局共享数据
})
// 其他页面脚本other.js
var appInstance = getApp()
console.log(appInstance.globalData) // 输出: I am global data
```

与此同时，我们要特别留意一点，所有页面的脚本逻辑都跑在同一个JsCore线程，页面使用setTimeout或者setInterval的定时器，然后跳转到其他页面时，这些定时器并没有被清除，需要开发者自己在页面离开的时候进行清理。

### 3.2.2 页面

一个小程序可以有很多页面，每个页面承载不同的功能，页面之间可以互相跳转。为了叙述简洁，我们之后讨论所涉及的“页面”概念特指“小程序页面”。

#### 1. 文件构成和路径

一个页面是分三部分组成：界面、配置和逻辑。界面由WXML文件和WXSS文件来负责描述，配置由JSON文件进行描述，页面逻辑则是由JS脚本文件负责。一个页面的文件需要放置在同一个目录下，其中WXML文件和JS文件是必须存在的，JSON和WXSS文件是可选的。
页面路径需要在小程序代码根目录app.json中的pages字段声明，否则这个页面不会被注册到宿主环境中。例如两个页面的文件的相对路径分别为 pages/index/page. *和 pages/other/other.* (*表示wxml/wxss/json/js四个文件)，在app.json的pages字段的代码路径需要去除.*后缀，如代码清单3-7所示，默认pages字段的第一个页面路径为小程序的首页。
代码清单3-7 app.json声明页面路径

```javascript
{
  "pages":[
    "pages/index/page", // 第一项默认为首页
    "pages/other/other"
  ]
}
```

为了叙述方便，下文使用page.wxml / page.wxss / page.json / page.js 来分别代表特定页面的4个文件。

#### 2. 页面构造器Page()

宿主环境提供了 Page() 构造器用来注册一个小程序页面，Page()在页面脚本page.js中调用，Page() 的调用方式如代码清单3-8所示。Page构造器接受一个Object参数，参数说明如表3-4所示，其中data属性是当前页面WXML模板中可以用来做数据绑定的初始数据，我们会在后文展开讨论；onLoad / onReady / onShow / onHide /onUnload 5个回调是Page实例的生命周期函数，我们在后文展开；onPullDownRefresh / onReachBottom / onShareAppMessage / onPageScroll 4个回调是页面的用户行为，我们也会在后文展开。
代码清单3-8 Page构造器

```javascript
Page({
  data: { text: "This is page data." },
  onLoad: function(options) { },
  onReady: function() { },
  onShow: function() { },
  onHide: function() { },
  onUnload: function() { },
  onPullDownRefresh: function() { },
  onReachBottom: function() { },
  onShareAppMessage: function () { },
  onPageScroll: function() { }
})
```

表3-4 Page构造器的参数

| 参数属性          | 类型     | 描述                                                         |
| :---------------- | :------- | :----------------------------------------------------------- |
| data              | Object   | 页面的初始数据                                               |
| onLoad            | Function | 生命周期函数--监听页面加载，触发时机早于onShow和onReady      |
| onReady           | Function | 生命周期函数--监听页面初次渲染完成                           |
| onShow            | Function | 生命周期函数--监听页面显示，触发事件早于onReady              |
| onHide            | Function | 生命周期函数--监听页面隐藏                                   |
| onUnload          | Function | 生命周期函数--监听页面卸载                                   |
| onPullDownRefresh | Function | 页面相关事件处理函数--监听用户下拉动作                       |
| onReachBottom     | Function | 页面上拉触底事件的处理函数                                   |
| onShareAppMessage | Function | 用户点击右上角转发                                           |
| onPageScroll      | Function | 页面滚动触发事件的处理函数                                   |
| 其他              | Any      | 可以添加任意的函数或数据，在Page实例的其他函数中用 this 可以访问 |

#### 3. 页面的生命周期和打开参数

页面初次加载的时候，微信客户端就会给Page实例派发onLoad事件，Page构造器参数所定义的onLoad方法会被调用，onLoad在页面没被销毁之前只会触发1次，在onLoad的回调中，可以获取当前页面所调用的打开参数option，关于打开参数我们放在这一节的最后再展开阐述。
页面显示之后，Page构造器参数所定义的onShow方法会被调用，一般从别的页面返回到当前页面时，当前页的onShow方法都会被调用。
在页面初次渲染完成时，Page构造器参数所定义的onReady方法会被调用，onReady在页面没被销毁前只会触发1次，onReady触发时，表示页面已经准备妥当，在逻辑层就可以和视图层进行交互了。
以上三个事件触发的时机是onLoad早于 onShow，onShow早于onReady。
页面不可见时，Page构造器参数所定义的onHide方法会被调用，这种情况会在使用wx.navigateTo切换到其他页面、底部tab切换时触发。
当前页面使用wx.redirectTo或wx.navigateBack返回到其他页时，当前页面会被微信客户端销毁回收，此时Page构造器参数所定义的onUnload方法会被调用。
我们可以看到，Page的生命周期是由微信客户端根据用户操作主动触发的。为了避免程序上的混乱，我们不应该在其他代码中主动调用Page实例的生命周期函数。
最后我们说一下页面的打开参数query，让我们来设想这样一个场景，我们实现一个购物商城的小程序，我们需要完成一个商品列表页和商品详情页，点击商品列表页的商品就可以跳转到该商品的详情页，当然我们不可能为每个商品单独去实现它的详情页。我们只需要实现一个商品详情页的pages/detail/detail.*(代表WXML/WXSS/JS/JSON文件)*即可，在列表页打开商品详情页时把商品的id传递过来，详情页通过刚刚说的onLoad回调的参数option就可以拿到商品id，从而绘制出对应的商品，代码如代码清单3-9所示。

代码清单3-9 页面的打开参数Page构造器

```javascript
// pages/list/list.js
// 列表页使用navigateTo跳转到详情页
wx.navigateTo({ url: 'pages/detail/detail?id=1&other=abc' })

// pages/detail/detail.js
Page({
  onLoad: function(option) {
        console.log(option.id)
        console.log(option.other)
  }
})
```

小程序把页面的打开路径定义成页面URL，其组成格式和网页的URL类似，在页面路径后使用英文 ? 分隔path和query部分，query部分的多个参数使用 & 进行分隔，参数的名字和值使用 key=value 的形式声明。在页面Page构造器里onLoad的option可以拿到当前页面的打开参数，其类型是一个Object，其键值对与页面URL上query键值对一一对应。和网页URL一样，页面URL上的value如果涉及特殊字符（例如：&字符、?字符、中文字符等，详情参考URI的RFC3986说明 ），需要采用UrlEncode后再拼接到页面URL上。

#### 4. 页面的数据

3.1.4节讨论了小程序界面渲染的基本原理，我们知道小程序的页面结构由WXML进行描述，WXML可以通过数据绑定的语法绑定从逻辑层传递过来的数据字段，这里所说的数据其实就是来自于页面Page构造器的data字段，data参数是页面第一次渲染时从逻辑层传递到渲染层的数据。

代码清单3-10 Page构造器的data参数

```html
<!-- page.wxml -->
<view>{{text}}</view>
<view>{{array[0].msg}}</view>

// page.js
Page({
  data: {
    text: 'init data',
    array: [{msg: '1'}, {msg: '2'}]
  }
})
```

宿主环境所提供的Page实例的原型中有setData函数，我们可以在Page实例下的方法调用this.setData把数据传递给渲染层，从而达到更新界面的目的。由于小程序的渲染层和逻辑层分别在两个线程中运行，所以setData传递数据实际是一个异步的过程，所以setData的第二个参数是一个callback回调，在这次setData对界面渲染完毕后触发。
setData其一般调用格式是 setData(data, callback)，其中data是由多个key: value构成的Object对象。
代码清单3-11 使用setData更新渲染层数据

```javascript
// page.js
Page({
  onLoad: function(){
    this.setData({
      text: 'change data'
    }, function(){
      // 在这次setData对界面渲染完毕后触发
    })
  }
})
```

实际在开发的时候，页面的data数据会涉及相当多的字段，你并不需要每次都将整个data字段重新设置一遍，你只需要把改变的值进行设置即可，宿主环境会自动把新改动的字段合并到渲染层对应的字段中，如下代码所示。data中的key还可以非常灵活，以数据路径的形式给出，例如 this.setData({"d[0]": 100}); this.setData({"d[1].text": 'Goodbye'}); 我们只要保持一个原则就可以提高小程序的渲染性能：每次只设置需要改变的最小单位数据。
代码清单3-12 使用setData更新渲染层数据

```javascript
// page.js
Page({
  data: {
    a: 1, b: 2, c: 3,
    d: [1, {text: 'Hello'}, 3, 4]
  }
  onLoad: function(){
       // a需要变化时，只需要setData设置a字段即可
    this.setData({a : 2})
  }
})
```

此外需要注意以下3点：

1. 直接修改 Page实例的this.data 而不调用 this.setData 是无法改变页面的状态的，还会造成数据不一致。
2. 由于setData是需要两个线程的一些通信消耗，为了提高性能，每次设置的数据不应超过1024kB。
3. 不要把data中的任意一项的value设为undefined，否则可能会有引起一些不可预料的bug。

#### 5. 页面的用户行为

小程序宿主环境提供了四个和页面相关的用户行为回调：

1. 下拉刷新 onPullDownRefresh
   监听用户下拉刷新事件，需要在app.json的window选项中或页面配置page.json中设置enablePullDownRefresh为true。当处理完数据刷新后，wx.stopPullDownRefresh可以停止当前页面的下拉刷新。

2. 上拉触底 onReachBottom
   监听用户上拉触底事件。可以在app.json的window选项中或页面配置page.json中设置触发距离onReachBottomDistance。在触发距离内滑动期间，本事件只会被触发一次。

3. 页面滚动 onPageScroll
   监听用户滑动页面事件，参数为 Object，包含 scrollTop 字段，表示页面在垂直方向已滚动的距离（单位px）。

4. 用户转发 onShareAppMessage
   只有定义了此事件处理函数，右上角菜单才会显示“转发”按钮，在用户点击转发按钮的时候会调用，此事件需要return一个Object，包含title和path两个字段，用于自定义转发内容，如代码清单3-13所示。

   代码清单3-13 使用onShareAppMessage自定义转发字段

   ```javascript
   // page.js
   Page({
   onShareAppMessage: function () {
    return {
      title: '自定义转发标题',
      path: '/page/user?id=123'
    }
   }
   })
   ```

#### 6. 页面跳转和路由

一个小程序拥有多个页面，我们可以通过wx.navigateTo推入一个新的页面，如图3-6所示，在首页使用2次wx.navigateTo后，页面层级会有三层，我们把这样的一个页面层级称为页面栈。









# 零零散散

取随机数

```javascript
Math.floor(Math.random() * range)
```

