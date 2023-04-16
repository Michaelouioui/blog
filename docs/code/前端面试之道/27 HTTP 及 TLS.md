# HTTP 及 TLS

这一章节我们将来学习 HTTP 及 TLS 协议中的内容。

## HTTP 请求中的内容

HTTP 请求由三部分构成，分别为：

- 请求行
- 首部
- 实体

请求行大概长这样 `GET /images/logo.gif HTTP/1.1`，基本由请求方法、URL、协议版本组成，这其中值得一说的就是请求方法了。

请求方法分为很多种，最常用的也就是 `Get` 和 `Post` 了。虽然请求方法有很多，但是更多的是传达一个语义，而不是说 `Post` 能做的事情 `Get` 就不能做了。如果你愿意，都使用 `Get` 请求或者 `Post` 请求都是可以的。更多请求方法的语义描述可以阅读 [文档](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FMethods)。

> 常考面试题：Post 和 Get 的区别？

首先先引入副作用和幂等的概念。

副作用指对服务器上的资源做改变，搜索是无副作用的，注册是副作用的。

幂等指发送 M 和 N 次请求（两者不相同且都大于 1），服务器上资源的状态一致，比如注册 10 个和 11 个帐号是不幂等的，对文章进行更改 10 次和 11 次是幂等的。因为前者是多了一个账号（资源），后者只是更新同一个资源。

在规范的应用场景上说，Get 多用于无副作用，幂等的场景，例如搜索关键字。Post 多用于副作用，不幂等的场景，例如注册。

在技术上说：

- Get 请求能缓存，Post 不能
- Post 相对 Get 安全一点点，因为Get 请求都包含在 URL 里（当然你想写到 `body` 里也是可以的），且会被浏览器保存历史纪录。Post 不会，但是在抓包的情况下都是一样的。
- URL有长度限制，会影响 Get 请求，但是这个长度限制是浏览器规定的，不是 RFC 规定的
- Post 支持更多的编码类型且不对数据类型限制

### 首部

首部分为请求首部和响应首部，并且部分首部两种通用，接下来我们就来学习一部分的常用首部。

**通用首部**

| 通用字段          | 作用                                            |
| ----------------- | ----------------------------------------------- |
| Cache-Control     | 控制缓存的行为                                  |
| Connection        | 浏览器想要优先使用的连接类型，比如 `keep-alive` |
| Date              | 创建报文时间                                    |
| Pragma            | 报文指令                                        |
| Via               | 代理服务器相关信息                              |
| Transfer-Encoding | 传输编码方式                                    |
| Upgrade           | 要求客户端升级协议                              |
| Warning           | 在内容中可能存在错误                            |

**请求首部**

| 请求首部            | 作用                               |
| ------------------- | ---------------------------------- |
| Accept              | 能正确接收的媒体类型               |
| Accept-Charset      | 能正确接收的字符集                 |
| Accept-Encoding     | 能正确接收的编码格式列表           |
| Accept-Language     | 能正确接收的语言列表               |
| Expect              | 期待服务端的指定行为               |
| From                | 请求方邮箱地址                     |
| Host                | 服务器的域名                       |
| If-Match            | 两端资源标记比较                   |
| If-Modified-Since   | 本地资源未修改返回 304（比较时间） |
| If-None-Match       | 本地资源未修改返回 304（比较标记） |
| User-Agent          | 客户端信息                         |
| Max-Forwards        | 限制可被代理及网关转发的次数       |
| Proxy-Authorization | 向代理服务器发送验证信息           |
| Range               | 请求某个内容的一部分               |
| Referer             | 表示浏览器所访问的前一个页面       |
| TE                  | 传输编码方式                       |

**响应首部**

| 响应首部           | 作用                       |
| ------------------ | -------------------------- |
| Accept-Ranges      | 是否支持某些种类的范围     |
| Age                | 资源在代理缓存中存在的时间 |
| ETag               | 资源标识                   |
| Location           | 客户端重定向到某个 URL     |
| Proxy-Authenticate | 向代理服务器发送验证信息   |
| Server             | 服务器名字                 |
| WWW-Authenticate   | 获取资源需要的验证信息     |

**实体首部**

| 实体首部         | 作用                           |
| ---------------- | ------------------------------ |
| Allow            | 资源的正确请求方式             |
| Content-Encoding | 内容的编码格式                 |
| Content-Language | 内容使用的语言                 |
| Content-Length   | request body 长度              |
| Content-Location | 返回数据的备用地址             |
| Content-MD5      | Base64加密格式的内容 MD5检验值 |
| Content-Range    | 内容的位置范围                 |
| Content-Type     | 内容的媒体类型                 |
| Expires          | 内容的过期时间                 |
| Last_modified    | 内容的最后修改时间             |

### 常见状态码

状态码表示了响应的一个状态，可以让我们清晰的了解到这一次请求是成功还是失败，如果失败的话，是什么原因导致的，当然状态码也是用于传达语义的。如果胡乱使用状态码，那么它存在的意义就没有了。

状态码通常也是一道常考题。

**2XX 成功**

- 200 OK，表示从客户端发来的请求在服务器端被正确处理
- 204 No content，表示请求成功，但响应报文不含实体的主体部分
- 205 Reset Content，表示请求成功，但响应报文不含实体的主体部分，但是与 204 响应不同在于要求请求方重置内容
- 206 Partial Content，进行范围请求

**3XX 重定向**

- 301 moved permanently，永久性重定向，表示资源已被分配了新的 URL
- 302 found，临时性重定向，表示资源临时被分配了新的 URL
- 303 see other，表示资源存在着另一个 URL，应使用 GET 方法获取资源
- 304 not modified，表示服务器允许访问资源，但因发生请求未满足条件的情况
- 307 temporary redirect，临时重定向，和302含义类似，但是期望客户端保持请求方法不变向新的地址发出请求

**4XX 客户端错误**

- 400 bad request，请求报文存在语法错误
- 401 unauthorized，表示发送的请求需要有通过 HTTP 认证的认证信息
- 403 forbidden，表示对请求资源的访问被服务器拒绝
- 404 not found，表示在服务器上没有找到请求的资源

**5XX 服务器错误**

- 500 internal sever error，表示服务器端在执行请求时发生了错误
- 501 Not Implemented，表示服务器不支持当前请求所需要的某个功能
- 503 service unavailable，表明服务器暂时处于超负载或正在停机维护，无法处理请求

## TLS

HTTPS 还是通过了 HTTP 来传输信息，但是信息通过 TLS 协议进行了加密。

TLS 协议位于传输层之上，应用层之下。首次进行 TLS 协议传输需要两个 RTT ，接下来可以通过 Session Resumption 减少到一个 RTT。

在 TLS 中使用了两种加密技术，分别为：对称加密和非对称加密。

**对称加密**：

对称加密就是两边拥有相同的秘钥，两边都知道如何将密文加密解密。

这种加密方式固然很好，但是问题就在于如何让双方知道秘钥。因为传输数据都是走的网络，如果将秘钥通过网络的方式传递的话，一旦秘钥被截获就没有加密的意义的。

**非对称加密**：

有公钥私钥之分，公钥所有人都可以知道，可以将数据用公钥加密，但是将数据解密必须使用私钥解密，私钥只有分发公钥的一方才知道。

这种加密方式就可以完美解决对称加密存在的问题。假设现在两端需要使用对称加密，那么在这之前，可以先使用非对称加密交换秘钥。

简单流程如下：首先服务端将公钥公布出去，那么客户端也就知道公钥了。接下来客户端创建一个秘钥，然后通过公钥加密并发送给服务端，服务端接收到密文以后通过私钥解密出正确的秘钥，这时候两端就都知道秘钥是什么了。

**TLS 握手过程如下图：**



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/12/1635260126b3a10c~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)



客户端发送一个随机值以及需要的协议和加密方式。

服务端收到客户端的随机值，自己也产生一个随机值，并根据客户端需求的协议和加密方式来使用对应的方式，并且发送自己的证书（如果需要验证客户端证书需要说明）

客户端收到服务端的证书并验证是否有效，验证通过会再生成一个随机值，通过服务端证书的公钥去加密这个随机值并发送给服务端，如果服务端需要验证客户端证书的话会附带证书

服务端收到加密过的随机值并使用私钥解密获得第三个随机值，这时候两端都拥有了三个随机值，可以通过这三个随机值按照之前约定的加密方式生成密钥，接下来的通信就可以通过该密钥来加密解密

通过以上步骤可知，在 TLS 握手阶段，两端使用非对称加密的方式来通信，但是因为非对称加密损耗的性能比对称加密大，所以在正式传输数据时，两端使用对称加密的方式通信。

PS：以上说明的都是 TLS 1.2 协议的握手情况，在 1.3 协议中，首次建立连接只需要一个 RTT，后面恢复连接不需要 RTT 了。

## 小结

总结一下内容：

- HTTP 经常考到的内容包括：请求方法、首部的作用以及状态码的含义
- TLS 中经常考到的内容包括：两种加密方式以及握手的流程

留言

![img](https://p9-passport.byteacctimg.com/img/user-avatar/5e41ee1c076d80557fa78839ed4c71c7~300x300.image)

发表评论

全部评论（23）

[![shaw2048的头像](https://p9-passport.byteacctimg.com/img/mosaic-legacy/3796/2975850990~300x300.image)](https://juejin.cn/user/2840793775349406)

[shaw2048](https://juejin.cn/user/2840793775349406)

7月前

tls握手过程写的有点模糊，希望能完善下，谢谢

点赞

回复

[![陌上兮月的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/430664290928360)

[陌上兮月![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/430664290928360)

FE技术爱好者🧪 @ FE1年前

我记得post请求不是不能做缓存，而且大多数情况做了没用。要做缓存只要服务端在响应头返回缓存相关字段就可以（比如expires，cache-control），但是大多数浏览器不认post类型的缓存。

点赞

回复

[![你若像风的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2277843823766967)

[你若像风![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/2277843823766967)

前端开发工程师1年前

写得很好。

点赞

回复

[![绿衣黄裳的头像](https://p9-passport.byteacctimg.com/img/user-avatar/ba76a1a62905fa80c5f7b28593efb4cd~300x300.image)](https://juejin.cn/user/4089838988702766)

[绿衣黄裳](https://juejin.cn/user/4089838988702766)

1年前

汇总一下content-type

1

回复

[![画饼师的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/184373686046110)

[画饼师![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/184373686046110)

2年前

搜索关键字不是无副作用的吧，搜多了不就变成热搜词了么

点赞

回复

[![行走的皮卡丘的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1169536101660782)

[行走的皮卡丘](https://juejin.cn/user/1169536101660782)

扫地僧 @ PWRD3年前

《首先服务端将公钥公布出去，那么客户端也就知道公钥了。接下来客户端创建一个秘钥，然后通过公钥加密并发送给服务端，服务端接收到密文以后通过私钥解密出正确的秘钥，这时候两端就都知道秘钥是什么了。》
这段文字，服务端收到密文，通过自己私钥解密出正确的秘钥？还有就是客户端创建的秘钥，服务端怎么知道客户端秘钥呢，还是说我的理解是错的，客户端和服务端都有自己秘钥，客户端用私钥加密发送，然后服务端用私钥解密？

1

3

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/1662117312467806)

[小谷围弟中弟](https://juejin.cn/user/1662117312467806)

3年前

客户端用服务端的非对称加密公钥去加密对称加密秘钥，发给服务端，服务端接收后用自己的非对称加密秘钥解开加密信息，得到对称加密的秘钥

点赞

回复

[![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/2752832847490557)

[屁儿快跑](https://juejin.cn/user/2752832847490557)

2年前

试着回答下: 服务端收到密文，通过自己私钥解密出正确的秘钥?
自己的密钥指的是 买https证书的时候认证机构给的一对非对称加密的私钥,
服务端收到的密文是客户端使用非对称加密的公钥加密之后的数据.

点赞

回复

查看更多回复

[![LYLYXY的头像](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8AQMAAAAAMksxAAAAA1BMVEUAAACnej3aAAAAAXRSTlMAQObYZgAAAA5JREFUKM9jGAWjAAcAAAIcAAE27nY6AAAAAElFTkSuQmCC)](https://juejin.cn/user/4248168659423629)

[LYLYXY](https://juejin.cn/user/4248168659423629)

我是切图仔3年前

http首部的作用是啥

点赞

1

[![img](https://p9-passport.byteacctimg.com/img/user-avatar/8a914ad38206c67a4bbfd67fb7cfe430~300x300.image)](https://juejin.cn/user/2752832846694206)

[Connie_豆](https://juejin.cn/user/2752832846694206)

2年前

\1. 缓存相关的参数：cache-control, if-modified-since.....这些参数都是用来和客户端商量是否以及怎么设置缓存，查询缓存是否可用等
\2. 请求参数的格式及数据，表单，大文本（文件上传）
我知道的大概这些吧

点赞

回复

[![austld的头像](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/mirror-assets/168e09116ba581f17e5~tplv-t2oaga2asx-no-mark:100:100:100:100.awebp)](https://juejin.cn/user/3104676567320279)

[austld](https://juejin.cn/user/3104676567320279)

3年前

写的真是一般啊

25

回复

[![前端贫困教师的头像](https://p9-passport.byteacctimg.com/img/user-avatar/0d93e320ee9a121042d9e0a487807c22~300x300.image)](https://juejin.cn/user/3825956194627470)

[前端贫困教师![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/3825956194627470)

程序员 @ 快手3年前

tls握手随机值不太对吧？客户端形成a发送给服务端，服务端收到a形成b发送给客户端，客户端收到b形成c发送给服务端，服务端收到c形成d。这么一算是客户端3个，服务端4个随机值啊，为什么作者说两端均有3个随机值

点赞

2

[![img](https://p9-passport.byteacctimg.com/img/user-avatar/15196a31bbba1ed5f5d20011def1d192~300x300.image)](https://juejin.cn/user/184373683957134)

[helianthus![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/e108c685147dfe1fb03d4a37257fb417.svg)](https://juejin.cn/user/184373683957134)

回复

[abukii0_0](https://juejin.cn/user/2823201590891006)

3年前

这篇写的很好

“

a(Sender生成)和b(Receiver生成)都是明文随机数, c(Sender生成)经公钥加密后生成Premaster secret(即预主秘钥), 发送给Receiver. 双方各自以a, b, c生成session key(对话密钥).
可参考下[![img](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)www.ruanyifeng.com](https://link.juejin.cn/?target=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2014%2F09%2Fillustration-ssl.html)

”

点赞

回复

[![img](https://p3-passport.byteacctimg.com/img/user-avatar/74f55dcf207eb8f4e4415f1700102b4f~300x300.image)](https://juejin.cn/user/588993962446184)

[熊川宇](https://juejin.cn/user/588993962446184)

3小时前

服务端收到c以后，是解密出来获取，并没有生成你所谓的d

点赞

回复

[![六白告的头像](https://p26-passport.byteacctimg.com/img/user-avatar/0120e220b60b433304b805bdba70ba70~300x300.image)](https://juejin.cn/user/1239904846615575)

[六白告![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/1239904846615575)

前端扑街3年前

希望加入跨站攻击的面试内容

2

回复

[![碧螺春的头像](https://p6-passport.byteacctimg.com/img/user-avatar/8c65fa61d14dbb07ec7f205a569e0f89~300x300.image)](https://juejin.cn/user/835284567072573)

[碧螺春![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/636691cd590f92898cfcda37357472b8.svg)](https://juejin.cn/user/835284567072573)

前端开发工程师 @ 京东3年前

写的一般

17

回复

[![周序猿的头像](https://p26-passport.byteacctimg.com/img/user-avatar/e914f767e6dd2a87dfdd45722fb75fec~300x300.image)](https://juejin.cn/user/4494459261689335)

[周序猿](https://juejin.cn/user/4494459261689335)

辣鸡程序员 @ 18线小公司3年前

502 Bad Gateway：作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。
504 Gateway Timeout：服务器响应超时。

4

回复

[![Swiftly10583的头像](https://p26-passport.byteacctimg.com/img/user-avatar/c4b7183642539e92624918bb95521e22~300x300.image)](https://juejin.cn/user/1451011077052471)

[Swiftly10583![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/1451011077052471)

会客户端的前端 @ 东方财富3年前

花了十天左右，把写的知识点都看完了。等待更新的，以及针对当时不知道的点复习以下。

点赞

回复

[![Swiftly10583的头像](https://p26-passport.byteacctimg.com/img/user-avatar/c4b7183642539e92624918bb95521e22~300x300.image)](https://juejin.cn/user/1451011077052471)

[Swiftly10583![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/f597b88d22ce5370bd94495780459040.svg)](https://juejin.cn/user/1451011077052471)

会客户端的前端 @ 东方财富3年前

这里秒速不清楚发，在TLS下，最终用于对称加密数据的用的是哪个随机数？或者是组合？

点赞

1

[![img](https://p26-passport.byteacctimg.com/img/user-avatar/a386aa8db73c9678458ec34161472ca5~300x300.image)](https://juejin.cn/user/712139233840407)

[yck![lv-7](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/7a2d00014351211140dd9bc0ba3afd8c.svg)](https://juejin.cn/user/712139233840407)

（作者）3年前

组合