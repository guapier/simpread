> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1729121-1-1.html)

> [md]## Android 某东 Sign 与 cipher 解密 > 最近疫情原因，哎，又沦落到了抢抢抢的境地，洋洋洒洒的网页端一气呵成写了个小助手哈，想要买莲花清瘟，一看只能手机 APP 购买 ... Andr......

 ![](https://avatar.52pojie.cn/data/avatar/000/37/19/27_avatar_middle.jpg) 胡家二少 _ 本帖最后由 胡家二少 于 2022-12-21 17:13 编辑_  

Android 某东 Sign 与 cipher 解密
---------------------------

> 最近疫情原因，哎，又沦落到了抢抢抢的境地，洋洋洒洒的网页端一气呵成写了个小助手哈，想要买莲花清瘟，一看只能手机 APP 购买。我去！真无语！那么再拿手机端开个刀吧，作如下记录。
> 
> 本人所发布的文章仅限用于学习和研究目的；不得将上述内容用于商业或者非法用途，否则，一切后果请用户自负。如有侵权请联系我删除处理。

主要功能涉及如下：

1. 解密 sign

2. 解密 cipher

### 1.APP 版本：

* * *

**For Android V11.3.2 build98450**

### 2. 工具

* * *

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

![](https://attach.52pojie.cn/forum/202212/21/154814xmzsxg1bi01zrb1v.png)

**image-20221221141843811.png** _(38.8 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY1NHxlNGU2MGUxMXwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

我们详细的看一下这个请求所携带的数据如下：

![](https://attach.52pojie.cn/forum/202212/21/154816boimjzgjig5gxqjg.png)

**image-20221221142058963.png** _(42.39 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY1NXwwN2I5Y2MzNXwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

经过多次抓包查看，发现这三个值是变化的。因此我们开始分析 APP。

#### Sign-Jadx 静态分析

把 app 导入到 jadx，我们要追踪 st，sign，sv 这三个参数的来源，那么我们线搜索一下，大概找下 sign 的位置

![](https://attach.52pojie.cn/forum/202212/21/154818d88ctz848l1p61k1.png)

**image-20221221145054285.png** _(106.51 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY1NnwzYjE0ZWRkYnwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

看到我标注的那个方法，**addQueryParameter** 翻译一下不就是添加 sign 参数嘛。点进去分析看看。

经过 frida 注入调试，分析出确实是这个方法 但是他是不进这个 if 的，不是这上半部分的代码，而是下半部分的代码

![](https://attach.52pojie.cn/forum/202212/21/154821vfobb8rofhht4fbl.png)

**image-20221221150033413.png** _(71.06 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY1N3w4YzBjOGZhMXwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

最终定位到是下面的这行代码生成的 sign，而且还有其他的 2 个参数 st 与 sv

![](https://attach.52pojie.cn/forum/202212/21/154823g144eltkz3e2v4kk.png)

**image-20221221150227804.png** _(56.99 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY1OHw2YTQ4ZjM1MHwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

我们继续点击这个函数进去看一下发现是个接口。有接口必然有实现！

![](https://attach.52pojie.cn/forum/202212/21/154825jzlhlph4e3qove3j.png)

**image-20221221150429297.png** _(18.57 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY1OXxkODAyNGNmYnwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

那么如何去找它的实现类呢？我们继续搜索搜索这个类的完整路径  `com.jingdong.jdsdk.d.c.s`

![](https://attach.52pojie.cn/forum/202212/21/154827whn6b59627kln2wg.png)

**image-20221221150541963.png** _(59.68 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY2MHw0N2E5NmFlM3wxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

我们继续点击进去第一个，发现果真找到了接口的实现类如下：

![](https://attach.52pojie.cn/forum/202212/21/154829rnpxim4yyn51to0p.png)

**image-20221221150642585.png** _(36.95 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY2MXwzMzhmZTk1MnwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

最终我们看到 sign 是调用的 so.... 白瞎了之前的一顿操作。至此分析结束。那么我们用 frida 来验证下是不是这样。

#### Sign-Frida 注入验证

注意新版京东有检测 frida 的，需要改个进程名跟端口。

用 frida 注入如下函数  `BitmapkitUtils.getSignFromJni(context, str, str2, str3, str4, str5);`

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

![](https://attach.52pojie.cn/forum/202212/21/154831u6pyn1t66khvz6nw.png)

**image-20221221151759726.png** _(31.41 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY2MnxhMjZjZjQ1MXwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

如上，我们可以看出，st sign sv 都是从 so 来的。

至此，sign 的来源已理清。

但是我们还有一个 body 的加密字段

![](https://attach.52pojie.cn/forum/202212/21/154833cxxc57uhzxaz3nxi.png)

**image-20221221152009043.png** _(25.98 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY2M3w2MTQ5Nzg5ZXwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

我们继续重复上面的分析。

#### cipher-Jadx 静态分析

继续搜索”cipher“字段

![](https://attach.52pojie.cn/forum/202212/21/154835n7nxuknetbthsfez.png)

**image-20221221152200707.png** _(45.66 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY2NHw2MmRlNDE2YXwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

可以看到就一个类，那么就很简单的可以得出加密算法了

![](https://attach.52pojie.cn/forum/202212/21/154837a0ppug25b3ux8x88.png)

**image-20221221152300274.png** _(72.27 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY2NXwwOGI5Yjk2ZnwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传

通过 frida 注入这个 b 函数会发现 b 参数 ==MODIFIED_BASE64 就是进入的这个 if，而且加密算法就在 d.b() 这个方法。直接用 java 还原就行了。

### 4. 调用

sign 签名用 [unidbg](https://github.com/zhkl0228/unidbg) 模拟调用 so 就可以了。cipher 这个直接复制 d 这个类就包含了加解密。然后模拟登录再请求购物车信息就 ok 啦。

结果展示获取购物车数据：

![](https://attach.52pojie.cn/forum/202212/21/154839z1cop9ocaozjdcao.png)

**image-20221221152726950.png** _(66.76 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4MDY2NnxkZjc0ZmNiMHwxNjcxNjk4NDUwfDkzMDQwMnwxNzI5MTIx&nothumb=yes)

2022-12-21 15:48 上传