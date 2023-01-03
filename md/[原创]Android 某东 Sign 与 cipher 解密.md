> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.pediy.com](https://bbs.pediy.com/thread-275592.htm)

> [原创]Android 某东 Sign 与 cipher 解密

Android 某东 Sign 与 cipher 解密
---------------------------

> 最近疫情原因，哎，又沦落到了抢抢抢的境地，洋洋洒洒的网页端一气呵成写了个小助手哈，想要买莲花清瘟，一看只能手机 APP 购买。我去！真无语！那么再拿手机端开个刀吧，作如下记录。

 

主要功能涉及如下：

 

1. 解密 sign

 

2. 解密 cipher

### 1.APP 版本：

 

For Android V11.3.2 build98450

### 2. 工具

 

预先善其事，必先利其器！请先准备如下分析工具

1.  [Jadx](https://github.com/skylot/jadx.git)
2.  [IDA](https://hex-rays.com/IDA-pro/)
3.  [Frida](https://frida.re/)
4.  [unidbg](https://github.com/zhkl0228/unidbg)
5.  [Charles](https://www.charlesproxy.com/)
6.  [Pycharm](https://www.jetbrains.com/)(可选)
7.  [Vscode](https://code.visualstudio.com/)（可选）

### 3. 分析

> 手机环境配置抓包请大家自己解决。下面演示购物车数据的抓取功能。从手机到 PC 端代码实现。

#### Charles 抓包

打开手机 app，然后再点击购物车功能

 

`https://api.m.*.com/client.action`

 

我们在 Charles 会看到很多这个地址的请求，我们可以找一下购物车的包，地址中包含 **functionid=cart** 的就是购物车的请求  
![](https://bbs.pediy.com/upload/attach/202212/668329_XFN6RB7QFC9RK4R.png)

 

我们详细的看一下这个请求所携带的数据如下：

 

![](https://bbs.pediy.com/upload/attach/202212/668329_AYMCU9GZJHG6EMX.png)

 

经过多次抓包查看，发现这三个值是变化的。因此我们开始分析 APP。

#### Sign-Jadx 静态分析

把 app 导入到 jadx，我们要追踪 st，sign，sv 这三个参数的来源，那么我们线搜索一下，大概找下 sign 的位置

 

![](https://bbs.pediy.com/upload/attach/202212/668329_FS3M5NW4VA667ED.png)

 

看到我标注的那个方法，**addQueryParameter** 翻译一下不就是添加 sign 参数嘛。点进去分析看看。

 

经过 frida 注入调试，分析出确实是这个方法 但是他是不进这个 if 的，不是这上半部分的代码，而是下半部分的代码

 

![](https://bbs.pediy.com/upload/attach/202212/668329_9JPD9QP6ZTCZT8R.png)

 

最终定位到是下面的这行代码生成的 sign，而且还有其他的 2 个参数 st 与 sv

 

![](https://bbs.pediy.com/upload/attach/202212/668329_FGD89YNEVTXGRN8.png)

 

我们继续点击这个函数进去看一下发现是个接口。有接口必然有实现！

 

![](https://bbs.pediy.com/upload/attach/202212/668329_7UQZR7ZF9HGS4WB.png)

 

那么如何去找它的实现类呢？我们继续搜索搜索这个类的完整路径 `com.jingdong.jdsdk.d.c.s`

 

![](https://bbs.pediy.com/upload/attach/202212/668329_JSFTXSUAX53QYT6.png)

 

我们继续点击进去第一个，发现果真找到了接口的实现类如下：

 

![](https://bbs.pediy.com/upload/attach/202212/668329_MZ5M246D2DRS9HW.png)

 

最终我们看到 sign 是调用的 so.... 白瞎了之前的一顿操作。至此分析结束。那么我们用 frida 来验证下是不是这样。

#### Sign-Frida 注入验证

注意新版京东有检测 frida 的，需要改个进程名跟端口。

 

用 frida 注入如下函数 `BitmapkitUtils.getSignFromJni(context, str, str2, str3, str4, str5);`

```
function HookHandle(clazz) {
    clazz.getSignFromJni.implementation = function (a, b, c, d, e, f) {
        console.log("=======================================")
        var r = this.getSignFromJni(a, b, c, d, e, f);
        console.log("param1: ", a)
        console.log("param2: ", b)
        console.log("param3: ", c)
        console.log("param4: ", d)
        console.log("param5: ", e)
        console.log("param6: ", f)
        console.log("result: ", r)
        return r;
    }
    console.log("HookHandle ok")
}
 
Java.perform(function () {
    Java.choose("dalvik.system.PathClassLoader", {
        onMatch: function (instance) {
            try {
                var clazz = Java.use('com.jingdong.common.utils.BitmapkitUtils');
                HookHandle(clazz)
                return "stop"
            } catch (e) {
                console.log("next")
                console.log(e)
            }
        },
        onComplete: function () {
            console.log("success")
        }
    })
})
```

请使用上面这种注入方法，不然会发生找不到类的错误。

 

![](https://bbs.pediy.com/upload/attach/202212/668329_H9DQSHCEYS5E9RF.png)

 

如上，我们可以看出，st sign sv 都是从 so 来的。

 

至此，sign 的来源已理清。

 

但是我们还有一个 body 的加密字段

 

![](https://bbs.pediy.com/upload/attach/202212/668329_HFRQ5PF5BCMA26S.png)

 

我们继续重复上面的分析。

#### cipher-Jadx 静态分析

继续搜索”cipher“字段

 

![](https://bbs.pediy.com/upload/attach/202212/668329_GKEXH6GBYFJ955T.png)

 

可以看到就一个类，那么就很简单的可以得出加密算法了

 

![](https://bbs.pediy.com/upload/attach/202212/668329_YHRQEZZGC5EVNB8.png)

 

通过 frida 注入这个 b 函数会发现 b 参数 ==MODIFIED_BASE64 就是进入的这个 if，而且加密算法就在 d.b() 这个方法。直接用 java 还原就行了。

### 4. 调用

sign 签名用 [unidbg](https://github.com/zhkl0228/unidbg) 模拟调用 so 就可以了。cipher 这个直接复制 d 这个类就包含了加解密。然后模拟登录再请求购物车信息就 ok 啦。

 

结果展示获取购物车数据：

 

![](https://bbs.pediy.com/upload/attach/202212/668329_86QJ2E4WZPVV7AQ.png)

[[2022 冬季班]《安卓高级研修班 (网课)》月薪三万班招生中～](https://www.kanxue.com/book-section_list-84.htm)

最后于 2022-12-21 15:39 被胡家二少编辑 ，原因： update

[#基础理论](forum-161-1-117.htm) [#逆向分析](forum-161-1-118.htm) [#协议分析](forum-161-1-120.htm) [#脱壳反混淆](forum-161-1-122.htm) [#漏洞相关](forum-161-1-123.htm) [#HOOK 注入](forum-161-1-125.htm) [#源码框架](forum-161-1-127.htm) [#工具脚本](forum-161-1-128.htm)