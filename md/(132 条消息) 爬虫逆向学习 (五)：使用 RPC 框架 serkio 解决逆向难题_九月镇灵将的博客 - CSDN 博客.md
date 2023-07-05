> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43845191/article/details/127194529)

### serkio 应用实战

*   [前言](#_1)
*   [实战开发](#_5)
*   *   [多次调用加密方法破解失败](#_7)
    *   [如何刷新加密方法](#_10)
    *   [同一个浏览器的加密代码如何给不同用户使用](#_19)
*   [注意事项](#_46)
*   [总结](#_61)

前言
==

最近在工作中遇到了一个[反爬虫](https://so.csdn.net/so/search?q=%E5%8F%8D%E7%88%AC%E8%99%AB&spm=1001.2101.3001.7020)产品，处于技术能力和新产品迭代更新快的考虑，最后选择使用 RPC 技术解决问题，因为 serkio 框架帮我们封装好了服务，且自身具备一定的负载均衡能力，所以选择它作为 RPC 实现方案。  
新手入门请参考 K 哥的文章，我也是通过这篇继续学习的。  
[RPC 技术及其框架 Sekiro 在爬虫逆向中的应用，加密数据一把梭](https://blog.csdn.net/kdl_csdn/article/details/123074729)

实战开发
====

由于是工作业务相关的开发，设计隐私问题，这里就不一一展示开发过程了，我大概罗列出开发过程中遇到的问题

多次调用加密方法破解失败
------------

在部署好 Sekiro 后，调用 RPC 服务已经能够拿到加密参数生成结果了，但是在多次调试后发现生成的结果用于请求时会失败，根据浏览器抓包请求流程确定请求接口定期会返回 419，结果会携带一个 data，当出现 419 必须将 data 进行处理，因为这个 data 会作用于下一次加密结果生成，流程图如下：  
![](https://img-blog.csdnimg.cn/97758ca6e31f446b849fd4e70fea7138.png)

如何刷新加密方法
--------

419 的 data 怎么刷新到加密方法里呢？这个需要在 js 处理响应数据中找到相关的处理代码（onResponse），然后只要将这段代码在 RPC 服务中执行即可。

```
if (request.hasOwnProperty('dunm_data')) {
    let _0x37561b = _0x365f28(request['dunm_data'], window);
    _0x37561b["run"]();
    window.localStorage.dunm_data = request['dunm_data'];
}

```

同一个浏览器的加密代码如何给不同用户使用
--------------------

使用同一份加密代码加密到的结果，然后用不同 cookie 发起请求，发现不是浏览器登录用户的请求结果是 419，重试也无效，确定加密过程会引用用户的信息，删除浏览器的登录信息后，原浏览器登录用户用加密结果发起请求也无效，但是在重新设置 cookie 后有效，确定只要动态设置 cookie 即可实现同一个浏览器的加密代码给不同用户使用。  
使用 Object.defineProperty 动态设置 cookie 和 ua

```
var client = new SekiroClient("ws://127.0.0.1:5612/business-demo/register?group=shanghai&clientId=" + guid());
client.registerAction("getLosEncrypt", function (request, resolve, reject) {
    if (request.hasOwnProperty('dunm_data')) {
        let _0x37561b = _0x365f28(request['dunm_data'], window);
        _0x37561b["run"]();
        window.localStorage.dunm_data = request['dunm_data'];
    }
    if (request.hasOwnProperty('cookie')) {
        Object.defineProperty(document, 'cookie', {
            value: request['cookie'],
            writable: true
        });
    }
    if (request.hasOwnProperty('userAgent')) {
        Object.defineProperty(navigator, 'userAgent', {
            value: request['userAgent'],
            writable: true
        });
    }
    let result = window["ssx91m$212"](request['hurl'], request['post_data'], {});
    resolve(result)
})

```

注意事项
====

1.  浏览器弹窗会将浏览器瞄点定位到弹框，导致 RPC 服务连接不上，所以我们需要处理弹窗，让其无法弹出，只需要重写方法即可

```
window.alert = function(str){
	return true;
}
window.compile = function(str){
	return true;
}

```

2.  RPC 其实不会产生太大的浏览器内存，我在三台服务器中部署了 sekiro，通过监控资源情况确定不会产生过大的内存占用
3.  sekiro 如何实现负载均衡呢？其实 sekiro 可以创建很多个 group，每个 group 可以创建多个 client 动态使用，也就是对于一个 group 来说其实是有一定的负载均衡能力，但是如果对多个 group 进行负载均衡，非商用版的话需要自己实现
4.  调用 rpc 服务时由于传参太大导致调用失败，这个时候可以使用 post，能实现一样的效果![](https://img-blog.csdnimg.cn/8b096643eccb4364ab61a1f2a5c35e73.png)
5.  如何动态实现 js 注入？sekiro 调用浏览器需要提前注入代码，这一步前期可以靠人工进行实现，但是后期想稳定或者部署量一大，人工就有点呛了，这里有两个方案：一个是使用油猴插件直接 hook 加密加密方案注入 js 代码；另外一种是如果注入场景是针对整个 js 文件的话，使用油猴工具很难实现，这时候可以使用 Reres 插件，能够对浏览器文件进行本地映射替换。  
    Reres 规则注意：本地路径需要`file:///`开头

总结
==

当我们不考虑去逆向 js 来实现加密参数的话，可以考虑使用 RPC 技术，它不需要加载多余的资源，稳定性和效率明显都更高，也不需要考虑浏览器指纹、各种环境。但是，由于服务时注入到浏览器 js 文件中的，所以需要维护浏览器窗口的稳定性，且如果网站对 ua 等浏览器信息进行强校验的话，其实 RPC 也很难使用。