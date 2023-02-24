> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7189431733782052919)

JS 逆向之补环境过瑞数详解
==============

**“瑞数”** 是逆向路上的一座大山，是许多 JS 逆向者绕不开的一堵围墙，也是跳槽简历上的一个亮点，我们必须得在下次跳槽前攻克它！！ 好在现在网上有很多讲解瑞数相关的文章，贴心的一步一步教我们去分析瑞数流程，分析如何去扣瑞数逻辑，企图以此教会我们 _(手动狗头)_。却鲜有文章详细去讲解如何通过纯补环境的方式过瑞数。今天，它来了！

为了让大家彻底搞定瑞数这个老大哥，本文将从以下四个部分进行描述：

1.  **rs 的流程逻辑**
2.  **浅谈扣代码过 rs**
3.  **详解补环境过 rs**
4.  **扣代码与补环境对比**
5.  _弯道超车环节_

篇幅较长，坐稳发车咯！ _注：本文内容均以一个新人友好型 rs4 网站： 网上房地产，为例；_

### 一：rs 的流程逻辑

我们在做逆向的时候，首先得分析出哪些`加密参数`是需要逆向的，然后再是去逆向这些参数。当然瑞数也是一样。 所以我们第一步就是`明确逆向的目标`：

*   现象：上了 rs 的网站会请求两次 page_url，第二次请求 page_url 时才能得到正确的页面内容；
*   分析：分析其请求体，发现第二次请求 page_url 时带上了 cookie_s 和 cookie_t, 而 cookies_s 是来自第一次请求 page_url 时其响应头 set 的;![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09d94e174af54accb729af2fa2e4d842~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)
*   结论：那么我们的目标就确定了 -- 破解`cookie_t的生成逻辑`;

现在，我们知道了需要逆向的加密参数是 cookie_t，那么 cookie_t 是从何而来呢，我们先分析一下网站的请求

#### 瑞数网站请求流程分析：

**`第一次请求：`** 请求 page_url，响应状态码 202，响应头中 set 了 cookie_s； ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4863f9340f640e2be4f1c33c7e90f58~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp) 响应体是 HTML 源码，从上到下可以分为四部分：我先剧透一下`它们的作用`

1.  **一个 meta 标签**，其 content 内容很长且是`动态`的（每次请求会变化），会在 eval 执行第二层 JS 代码时使用到；
2.  **一个外链 js 文件**，一般同一页面中其内容是`固定`的，下面的自执行函数会解密文件内容生成 eval 执行时需要的 JS 源码，也就是第二层 vm 代码；
3.  **一个大自执行函数**（每次请求首页都会`动态`变化），主要是解密外链 JS 内容，给 window 添加一些属性如 $_ts，会在 vm 中使用;
4.  末尾的两个 script 标签中的函数调用，会`更新cookie`，使其变长。这里我们可以不用关注这个。![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e4a9890706b4462820b8743b39e3470~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

**`第二次请求：`** 请求外链 js，一般内容是固定的； ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6147020cdaf441759b2f50deca15d946~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

**`第三次请求：`** 请求 page_url, 返回 200，携带 cookie_s,cookie_T 即可正常请求； ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14885a674ba54cc09f1bf3b3a5787e5b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

那么当我们在浏览器中访问该网站时到底发生了什么？其中有什么值得我们关注的？ 我们先人工模拟一下`浏览器加载page_url源码`会发生什么：

1.  浏览器加载 meta ；
2.  浏览器请求`外链js`并执行 js 内容；
3.  执行 page_url 源码`自执行函数`，它内部会将`外链js解密`成 eval 需要的万行 js 字符串，并给 window.$_ts 添加了很多属性，然后调用 eval 函数`进入VM`执行解密后的 js，**生成 cookie**，eval 执行完毕，继续执行自执行函数；
4.  执行末尾 script 标签中的代码，这些代码会更新 cookie_t（**可以不用管**）
5.  执行 setTimeout、eventlistener 回调函数（**可以不用管**）

`瑞数执行流程图解`如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed9f180982c5410595f5059b28347f03~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

这里我们需要关注`eval调用`的位置（也就是`VM的入口`），**cookie 生成的位置**。

> 注：浏览器 v8 调用 eval 执行代码时会开启一个虚拟机（VM + 数字）去执行 JS 代码。

我们直接`hook eval`或`搜索.call`就可以定位到调用 eval 的位置 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cb50dfc84e54e218c8ed43f1dafa290~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp) _$Ln 就是`解密后`的 js 代码字符串；进入 vm 是一个自执行函数，其中有`生成cookie`的逻辑，定位 cookie 可以直接 hook 或在 vm 代码中搜索 (5) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c97ad6c9ce72406a9836cea1c1951af6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

hook cookie 代码：

```
// hook  指定cookie赋值
(function () {
    'use strict';
    Object.defineProperty(document, 'cookie', {
        set: function (cookie) {
            if(cookie.indexOf("FSSBBIl1UgzbN7N80T")!= -1){
                debugger;
            }
            return cookie;
        }
    });
})();
复制代码
```

好了，以上就是瑞数的整体流程逻辑，那么我们该如何通过`扣代码`去生成 cookie_t 过瑞数呢？

### 二、浅谈扣代码过瑞数

由于每次请求瑞数 page_url，其源码是`动态`变化的，`VM代码`也是动态变化的，所以我们要保存一份静态代码方便调试； 直接如图这样，固定 page_url 响应即可， ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5019268a186a4604b4cf927a01df2df9~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

这样请求时，page_url 中的 `外链JS`是固定的，`自执行函数`是固定的，`VM中的代码`也是固定的，那么这份固定代码每次生成的 cookie_t 是不是也固定呢？ 答案：**不是**，因为 cookie_t 的生成用到了**随机数**和**时间戳**，我们将这两个变量也 hook 固定住，最终生成的 cookie_t 就是固定的啦。

此时我们就可以开始扣这份静态代码了，扣到 node 生成的`cookie_t`与浏览器执行静态代码生成的`cookie_t` 一致就说明扣成功了。

**扣代码需要扣两部分，page_url 源码部分和 VM 中生成 cookie_t 部分。** 按照我们之前的分析，VM 中代码会用到`window.$_ts`，因此我们得先保证从 page_url 源码中扣下来的代码执行到 eval 时，能正常生成`window.$_ts`。

我们将 page_url 的`自执行函数`和`外链JS内容`先扣下来，meta content 也先扣下来，然后根据执行报错简单补一下环境，使得扣下来代码执行到 eval 时，其`window.$_ts`和解密出的`VM JS代码`与本地静态代码生成的一致。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8305699727354658916c9ec5bf3a4ce3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp) 这就说明我们 page_url 源码部分扣好了，接下来就可以去扣 VM 中生成 cookie_t 的部分。

上面我们分析到`cookie的定位`，可以知道 _$bO 这个函数中生成了二次 cookie_t，那这就是我们`扣代码的起点`，将该函数扣到文件末尾，直接执行，然后缺啥补啥，遇到使用`BOM、DOM api`的代码时，我们可以使用`等价逻辑替换`，比如此处原逻辑是获取 meta 标签中的 content 内容，然后再删除 meta 标签，我们可以直接等价替换成 return meta_content。如图： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49357a00d9d3444c880c64c209fba855~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

如果扣到最后，node 生成的 cookie_t 与本地静态代码生成的**不一致**，则说明我们扣的代码有些**环境检测没过**，我们可以根据 node 与浏览器执行的**差异**去定位，比如 node 执行到某个地方的值与浏览器执行静态代码得到的不一样，说明前面逻辑有差异，继续向前定位，如此反复即可找出所有遗漏的环境检测，完成 VM 部分扣取。

此时我们就完成了这份**静态 rs 代码**的扣取并取得成功，但是 rs 网站的代码是`动态`的啊，每次请求时 `window.$_ts`和`VM js`都会变化，难道我们每份都要去扣吗？不，其实我们现在离真正成功只差一个映射， **VM 的万行 js 代码** 虽然每次都会变化，但是变化的只是变量名，其他的都不变。 映射就是将`动态window.$_ts`中的属性名与 我们扣的 VM 中 JS 用到的`window.$_ts`中的固定属性名一一对应。

所以在扣的过程中我们需要注意 VM 中哪些地方使用了`外部变量`，（即一个函数中，哪个变量来自其他作用域，就算外部变量），我们需要关注这些外部变量是**从哪来的**，如果是 VM 自执行函数中定义的，那就不用管，如果是来自`window.$_ts`，则需要记录下来，这就是需要映射传入的。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/752b401b01f24419977781475199acab~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp) 这里的计算逻辑使用到了 `window.$_ts._$IK`，所以我们要做映射时要传入这个值；`window.$_ts={_$IK:对应的动态属性名}`

解决完映射，也就成功扣代码过 rs 了。 好了，扣代码到此为止，接下来就是我们本文的重点。

### 三、详解补环境过 rs

不知道补环境原理的同志可以参考我上篇文章：`JS逆向之浏览器补环境详解`; 其实`纯补环境`过瑞数原理很简单，我们来观察**瑞数执行流程图解**，基于浏览器环境执行这些**动态 JS** 可以生成可用的 cookie_t。那么只要我们补的浏览器环境足够完美，使得在这些**动态 JS** 看来，我们`补的环境===浏览器环境`，那么我们补的环境执行这些**动态 JS**，同样也能生成可用的 cookie_t，然后我们再通过 **document.cookie** 将 cookie_t 提取出来不就好了。

用伪代码表示就是：

```
// 补的环境头
window = this;
... 省略大量环境头
// 模拟meta标签及其content
document.createElement('meta');
Meta$content = "{qYnKTJPAw84QfF5jm0I2_1IqhgTvRw8Y0yCBPxIVn6od8AeJE6CBz8ZSU6U...省略";
// 固定的外链js
$_ts=window['$_ts'];if(!$_ts)$_ts={};$_ts.scj=[];$_ts['dfe1675']='þþ+...省略';
// page_url动态自执行函数
(function(){var _$CK=0,_$WI=[[9,3,6,0,4,1; ...ret = _$su.call(_$fr, _$WR); 很长...省略}}}}}}}})();;


// 获取cookie
function get_cookie(){
    return document.cookie;
}
// 获取MmEwMD参数
function get_mme(){{
    XMLHttpRequest.prototype.open("GET","http://脱敏/",true);
    return XMLHttpRequest.prototype.uri;
}}
复制代码
```

这就是我们最后要补成的文件样子，由于 **Meta$content** 和 page_url **动态自执行函数**是动态变化的，因此我们每次请求 page_url 都需要用**正则提取**出这两个字符串，然后拼接到该文件中，然后 python pyexecjs 调用 get_cookie 即可得到可用 cookie_t;

在上面的扣代码过瑞数中也提到了，由于有**随机数**和**时间戳**参与生成 cookie_t 运算，导致同一份`静态JS代码`生成的 cookie_t 是变化的，我们可以通过`hook`使得时间戳和随机数固定，这样同一份静态 JS 生成的 cookie_t 就是固定的。

```
// 固定定随机数和时间戳
Date.prototype.getTime = function(){return 1672931693};
Math.random = function(){return 0.5};
复制代码
```

我们也可以通过最终生成的 cookie_t 来判断，自己`补的环境`是不是 ===`浏览器环境`。 原理很简单，接下来就是如何实践，我们需要补出一份**完美的环境头**使得这份`静态JS`执行得到的 cookie_t 与浏览器执行得到的一致。 因为补环境是一个系统性工作，是有**通用套路**的，我们可以使用上篇文章提到的`补环境框架`进行系统性补环境，我们要做的就是根据`log输出`以及出现的问题不断完善这个框架。

如图，启动框架，上面是我们补的环境头，下面是我们扣的代码， ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e66607518c0458db8bc599ff68a8db1~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

继续调试，当我们`Proxy拦截器`拦截到`BOM、DOM api`的使用时，就会 debugger 住，我们可以根据 **调用栈** 去查看是哪行代码使用的浏览器环境，然后再看到框架中模拟的结果是不是跟浏览器一致，如果不一致，则需要补这个环境。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ef956e0dfa04186b9fad2f5e227b015~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

如此往复的补环境，直到最后可以生成`cookie_t`，判断是否与浏览器本地生成的一致，如果不一致则使用二分法去定位，看是哪个浏览器环境没补好，直到 最终得到正确的`cookie_t`，这里贴一下最后执行效果图： 这是最后打印的一部分`环境检测点`： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b286c68f0264dfcbe296faedaff5db3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

这是取出最终得到的`cookie_t`： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e5862824a9f4c80a730015f3d7df67c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

同理，`MmEwMD`参数的补环境也是一样的逻辑，当环境头补到完美时，在 python 中执行最终结果文件，即可得到如下结果： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a69780f009974708828cd50d864ac82b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 四、补环境与扣代码总结：

对于`js逆向`来说，这是两种常规且实用的手段，也各有优劣势；

不管使用哪种方式，我们都是先从网站中将加密 JS 代码扣出，然后再选择是继续扣代码，将使用到的`浏览器环境api`进行逻辑替换；还是使用补环境，让加密 JS 代码仿佛在浏览器环境中运行。

*   扣代码与补环境都依赖对 JS 的熟练度，扣代码更侧重 js 语法和代码逻辑，补环境更侧重原型链及 BOM、DOM 对象的模拟。
*   扣代码熟练度依赖逆向经验，补环境几乎只依赖 JS 熟练度。
*   扣代码需要调试跟踪大量逻辑，对于 rs，如果不解混淆的话，屁股得坐出痔疮；
*   扣代码需要替换环境检测逻辑，所以也需要知道哪里使用了浏览器环境； 补环境框架可以只用于监测浏览器环境的使用，可以当做扣代码的一个辅助工具。
*   由于瑞数是动态的，扣代码只能扣一份静态的，所以需要找到 vm 中使用到的所有动态属性进行映射。而补环境是通用的，补的越多，可通杀的网站就越多。
*   扣代码比补环境执行效率高，毕竟补环境的代码数比扣代码多很多，可以通过剔除不需要的环境来缩小差距；
*   扣代码人工耗时远高于补环境。

总而言之，`扣代码`侧重 js 语法和代码逻辑，其熟练度依赖于逆向经验，对不同网站要扣的不一样，难以通用，人工效率低，但是程序执行效率高。 `补环境`侧重原型链及浏览器环境模拟，熟练度几乎只依赖对 JS 的原理掌握程度，对于不同网站补的越多可通杀的网站越多，人工效率巨高，但是程序执行效率不高。

### 五、弯道超车环节

过瑞数几乎是每个新手逆向者的小目标，也是面试官常见的提问点。通过本篇文章，我们已经了解了 `瑞数的流程及破解思路`，接下来我们可以尝试自己从头实现一个完善的补环境框架，去`纯环境黑盒过瑞数`，但是这会花费很长一段时间来进行开发，而且其中有很多重复性工作比较无聊（复制粘贴对比等）。

**走快车道：** 本文该版本 **补环境框架** 是基于上篇文章框架进行系统性完善的，目前可以过瑞数这种级别网站，可以说是相当完善了，补了相当多的环境，如果你想省下大量时间、`极大提高效率`、直接`弯道超车`的话，可以 微信联系我：**dengshengfeng666** 付费源码借鉴； 春节优惠价 199（节后恢复统一固定价 233），付完直接发框架项目源码（有最新图文 readme 可直接小白上手），后续有疑问可以直接问我。 或者直接在 CSDN 私信我。

**该版本与上版本改进点：**

1.  优化 Proxy13 种拦截器，做到了 Proxy 的极限，并使其能递归代理，支持 a.b.c.d.e... 级别的深度检测；
2.  完善最终结果文件. js 的调用机制，使其无需改动，能直接用于 V8 和 node 调用；
3.  新增部分 BOM、DOM 对象，如：XMLHttpRequest，XMLHttpRequestEventTarget 等等；
4.  补全 rs 用到的所有浏览器环境方法，使其可以黑盒过 rs；
5.  优化调试方式，可以在想断的时候断点，不想断的时候跳过检测；
6.  增加 py 直接调用结果文件案例，可以使用 python 以 v8 和 node 方式调用；
7.  优化 readme，图文介绍环境配置及使用方式。

弯道超车，从我做起 该版本框架目录： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/731ac25f288a4a0981134670cc652e52~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)