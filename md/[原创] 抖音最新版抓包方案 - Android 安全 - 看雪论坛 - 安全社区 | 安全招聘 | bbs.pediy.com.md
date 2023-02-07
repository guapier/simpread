> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.kanxue.com](https://bbs.kanxue.com/thread-276015.htm)

> [原创] 抖音最新版抓包方案

* * *

网上有一些 patch so 的方法，我尝试了一下抓包会有一些问题 ![](https://bbs.kanxue.com/upload/attach/202302/884888_UJQJBNKMYDH2TPV.png)  
可以通过 hook java 层如下图所示的地方，dy 默认走的是 quick 协议，但是为了兼容更多版本的手机，有一个降级操作，毕竟担心 cronet 低版本适配不好，所以可以通过 hook 这个方法来使其强制降级到 Http 协议。  
![](https://bbs.kanxue.com/upload/attach/202302/884888_M5GSS6WXSXCY2TR.png)

frida 脚本：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p></td><td><p><code>setImmediate(function() {</code></p><p><code>Java.perform(function() {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>var targetClass</code><code>=</code><code>'org.chromium.CronetClient'</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>var methodName</code><code>=</code><code>'tryCreateCronetEngine'</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>var gclass </code><code>=</code> <code>Java.use(targetClass);</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>gclass[methodName].overload(</code><code>'android.content.Context'</code><code>,</code><code>'boolean'</code><code>,</code><code>'boolean'</code><code>,</code><code>'boolean'</code><code>,</code><code>'boolean'</code><code>,</code><code>'java.lang.String'</code><code>,</code><code>'java.util.concurrent.Executor'</code><code>,</code><code>'boolean'</code><code>).implementation </code><code>=</code> <code>function(arg0,arg1,arg2,arg3,arg4,arg5,arg6,arg7) {</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>}</code></p><p><code>})</code></p><p><code>})</code></p></td></tr></tbody></table>

frida hook 一下就可以抓包了，解决了 so patch 一些包抓不了的问题，应该可以通杀全版本 dy。  
![](https://bbs.kanxue.com/upload/attach/202302/884888_598PUHZFHZY6ZQA.png)

[[招生] 科锐逆向工程师培训 46 期预科班将于 2023 年 02 月 09 日 正式开班](https://bbs.kanxue.com/thread-51839.htm)

返回