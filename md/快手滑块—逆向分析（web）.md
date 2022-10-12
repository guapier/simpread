> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1697353-1-1.html)

> [md] 目标 URL：>aHR0cHM6Ly9wYXNzcG9ydC5rdWFpc2hvdS5jb20vcGMvYWNjb3VudC9sb2dpbi8/c2lkPWt1YWlzaG91LndlYi5jcC......

![](https://avatar.52pojie.cn/data/avatar/001/61/01/37_avatar_middle.jpg)timeslover _ 本帖最后由 timeslover 于 2022-10-9 22:34 编辑_  

目标 URL：

> aHR0cHM6Ly9wYXNzcG9ydC5rdWFpc2hvdS5jb20vcGMvYWNjb3VudC9sb2dpbi8/c2lkPWt1YWlzaG91LndlYi5jcC5hcGkmY2FsbGJhY2s9aHR0cHMlM0ElMkYlMkZjcC5rdWFpc2hvdS5jb20lMkZyZXN0JTJGaW5mcmElMkZzdHMlM0Zmb2xsb3dVcmwlM0RodHRwcyUyNTNBJTI1MkYlMjUyRmNwLmt1YWlzaG91LmNvbSUyNTJGYXJ0aWNsZSUyNTJGcHVibGlzaCUyNTJGdmlkZW8lMjUzRm9yaWdpbiUyNTNEd3d3Lmt1YWlzaG91LmNvbSUyNnNldFJvb3REb21haW4lM0R0cnVl

![](https://img-blog.csdnimg.cn/795f5c46e2db4d3a8e9d4847c312c7ac.png)

此处 iframe 新开一个窗口，可以减少其他的干扰

![](https://img-blog.csdnimg.cn/b72b0273c175484a9b1370addbe753b9.png)

失效的话，将上面的重新获取一下，重新复制 iframe 的 URL，进去就可以了

![](https://img-blog.csdnimg.cn/acaaf91359f34edca7ee81c3227359a5.png)

验证不通过

![](https://img-blog.csdnimg.cn/7c7a9b86dc3d4a42a11a68d3c9f97dac.png)  
验证通过

![](https://img-blog.csdnimg.cn/006344794e974d5bb0d18ee5b821cabb.png)

检验 URL 和参数

> [https://captcha.zt.kuaishou.com/rest/zt/captcha/sliding/kSecretApiVerify](https://captcha.zt.kuaishou.com/rest/zt/captcha/sliding/kSecretApiVerify)

![](https://img-blog.csdnimg.cn/498af49bbd624165994a8c0d15fdd876.png)

全局搜一下参数 verifyParam

![](https://img-blog.csdnimg.cn/91f563e5ed604fa8ac7c0b6392114d0d.png)

c 参数是一个做了加密的长字符串

![](https://img-blog.csdnimg.cn/fe60485f04fe4c1bb21875f337359901.png)

此处需要注意，当我断在 6374 的时候，c 还是 undefined

![](https://img-blog.csdnimg.cn/f315ff1b1f3d49699f30bffc2c091065.png)  
F8 运行到 6394 的时候，他的值就计算出来了  
![](https://img-blog.csdnimg.cn/8027ca37ef1c4a5b84260af23190370e.png)

回到 case 为 0 处，有个 **r** 对象，其中有个 trajectory 参数，大概猜测是路径加密后的值，可以先从这里入手

![](https://img-blog.csdnimg.cn/a5c3e026f3c94ed89009cc837d9faec4.png)

全局搜索一下参数 “trajectory”

![](https://img-blog.csdnimg.cn/c507c56d10924f20bd06c3063d13a215.png)

此处搜到了 3 个，分别下断点，拖动之后，最后是断在行号 **8863** 的这个

![](https://img-blog.csdnimg.cn/fdedf1fb0c4e4d2ca63f2a43484c6933.png)

结果是经过一个参数 **c.slice(1)**

![](https://img-blog.csdnimg.cn/a0d9ed14b73244aa9f586101c0fe2cd9.png)

此处是他的拖动路径数据

![](https://img-blog.csdnimg.cn/93b046e1cfab4efb8996c83b08015f08.png)

此处 c 的计算在这个位置

![](https://img-blog.csdnimg.cn/8b57ecda0d214eaca3e0027b919e38e8.png)

此处为指纹 hash

![](https://img-blog.csdnimg.cn/f14db364a44e4a8582cca5ca34822058.png)

到这一步，就可以把路径算法扣下来，路径还原结果如下

```
const t = {
  trajectory: [
    [0, 22, 1664078425847],
    [7, 22, 1664078425857],
    [25, 20, 1664078425864],
    [50, 18, 1664078425875],
    [79, 17, 1664078425881],
    [108, 15, 1664078425891],
    [141, 13, 1664078425898],
    [170, 11, 1664078425904],
    [199, 9, 1664078425911],
    [228, 8, 1664078425918],
    [257, 7, 1664078425926],
    [289, 5, 1664078425934],
    [315, 4, 1664078425942],
    [333, 3, 1664078425950],
    [355, 3, 1664078425958],
    [369, 3, 1664078425966],
    [384, 3, 1664078425976],
    [398, 3, 1664078425985],
    [413, 3, 1664078425990],
    [423, 3, 1664078425997],
    [431, 3, 1664078426007],
    [438, 3, 1664078426014],
    [449, 3, 1664078426023],
    [452, 3, 1664078426030],
    [456, 3, 1664078426036],
    [460, 3, 1664078426044],
    [467, 3, 1664078426052],
    [471, 3, 1664078426211]
  ]
}
let n = {}

const f = {
  uhUaq: function (n, t) {
    return n - t
  },
  FuwxQ: function (n, t) {
    return n + t
  }
}

let r = t.trajectory[0] ? t.trajectory[0][2] : 0

let c = t.trajectory.slice(-100).reduce(function (n, t) {
  return f.FuwxQ(
    n,
    ','
      .concat(t[0], '|')
      .concat(t[1], '|')
      .concat(f.uhUaq(t[2], r))
  )
}, '')

console.log(c.slice(1))

```

本地结果和浏览器验证结果匹配，说明算法没有扣错

![](https://img-blog.csdnimg.cn/aab1f42a95074356b1c3d4ccee9477f1.png)

然后我们再回到之前的这个位置，目前已经把路径的参数给解决了，还剩下 **captchaExtraParam**，**captchaSn**，**gpuInfo**

![](https://img-blog.csdnimg.cn/c46943886d684b138b69065f350b8760.png)

在这个位置下断点，当 **case** 为 **0** 执行完之后，**t** 参数的 **sent** 会把最后的计算结果算出来并赋值

![](https://img-blog.csdnimg.cn/b021779497c04b97a977f89df34b2cb6.png)  
![](https://img-blog.csdnimg.cn/316b26ffbeb84cc99a295ae84d084e0b.png)

这里 hook 去跟一下

![](https://img-blog.csdnimg.cn/40e8388c195b4a3ebb0f784a56f502f9.png)  
跟到这里的时候，Shift+F11

在此处下断点，重新刷新，进到 f 函数里面

![](https://img-blog.csdnimg.cn/c6dead7383094eb08a43e795479ebae1.png)

进到这里之后，F8 运行，观察 **n** 形参的变化

![](https://img-blog.csdnimg.cn/1b449818f4fd4472875b109dbf08b4fb.png)

n 变量的变化是这样的  
1、先是 UA 和指纹信息

![](https://img-blog.csdnimg.cn/64e4842331ae4603b274d3162acd8ec0.png)

2、然后是会传一个超长的数组

![](https://img-blog.csdnimg.cn/3b638eaf27cc45209aa71debcd8273cf.png)

3、最后就是运算出结果

![](https://img-blog.csdnimg.cn/45140b72e06040d58d6554234dc8631c.png)

当 n 出现数组的时候，F11 进入，会进到这个位置，这里就是最后结果生成的地方

![](https://img-blog.csdnimg.cn/be53faa11abf4ce98d00d5525b884fd1.png)

以下是每个值的参数和输出的结果  
![](https://img-blog.csdnimg.cn/52d61ab1ca6c49e6b37976299af9e86f.png)

![](https://img-blog.csdnimg.cn/8ba0d09459ec413b873563385aa90610.png)

![](https://img-blog.csdnimg.cn/69cda9b9ad3247d28316a6a79654d74f.png)

![](https://img-blog.csdnimg.cn/55223a7656ce4526878937ed7b0d951b.png)

到了断点位置，  
先执行完 **u = e[i("0x32")]**，可以看到 e 的输出是一个大数组  
然后执行 e[i("0x33")](i("0x34"), n[i("0x35")](u)[i("0x36")](t[i("0x2b")]));  
最后 e 里面的参数就计算出来了

![](https://img-blog.csdnimg.cn/d801e7c8ccc14d7986c8ff2afdfb5d7d.png)

所以我们只需要在这个地方注入我们需要的代码就可以，  
然后将 **c.a[i("0x31")]，d，e[i("0x33")](i("0x34"), n[i("0x35")](u)[i("0x36")](t[i("0x2b")]))** 导出到全局，然后 rpc 调用，或者你也可以扣代码，我这里是直接自己写一个 WSS 去调算法，想扣代码的大佬自己去扣，这里就不多废话了。

最后本地验证一下，成功获得凭证数据

![](https://img-blog.csdnimg.cn/604ddded90614060910e2d324182defa.png)

**补补补补补补补：**

**captchaSn** 参数是来自 **[https://captcha.zt.kuaishou.com/rest/zt/captcha/sliding/config](https://captcha.zt.kuaishou.com/rest/zt/captcha/sliding/config)** 这个接口的返回数据

![](https://img-blog.csdnimg.cn/d98133a39bf3400485ddaf014981c06a.png)

**captchaSession** 参数又是来自上一个接口 **[https://id.kuaishou.com/pass/kuaishou/sms/requestMobileCode](https://id.kuaishou.com/pass/kuaishou/sms/requestMobileCode)**

![](https://img-blog.csdnimg.cn/77f844a1589d46b8860b9addcaeebb10.png) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) 太复杂了, 图片转二值化 黑色为 1 白色为 0     图块 011111111111111111111110  匹配 缺口 10000000000000000000001 即可 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) ycgzs 学到了学到了 以为我们 APP 的抢购加这个就万无一失了呢 看来还要进一步防范一下了 ![](https://avatar.52pojie.cn/data/avatar/000/93/53/52_avatar_middle.jpg) zhzhch335 大佬的代码看着看着就睡着了![](https://static.52pojie.cn/static/image/smiley/default/4.gif) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) xixicoco 看不懂，希望出个成品 da's![](https://avatar.52pojie.cn/images/noavatar_middle.gif)shukaisa 谢谢分享 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) sjz051 感谢分享![](https://avatar.52pojie.cn/images/noavatar_middle.gif)wang754782072 感谢大佬分享 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) d199212 谢谢分享 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) xzhtx 谢谢分享![](https://avatar.52pojie.cn/data/avatar/001/61/01/37_avatar_middle.jpg)abu21

> [ycgzs 发表于 2022-10-10 01:06](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=44279788&ptid=1697353)  
> 太复杂了, 图片转二值化 黑色为 1 白色为 0     图块 011111111111111111111110  匹配 缺口 1000000000000000000 ...

这个问题不在本文章讨论范围，你要识别的话，可以参考之前我发的树美文章，还有顶象文章，里面有写利用 opencv 的识别算法，如果你觉得识别率不高，可以考虑用 yolo 等工具，自己训练数据，做数据模型