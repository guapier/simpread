> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1753513-1-1.html)

> [md]### 1. 目标分析某车之家 sign 签名算法的实现 ### 2. 操作环境 - frida- mac 系统 - Charles 抓包 - 越狱 iPhone ### 3. 流程 #### 寻找 ... 【iOS 逆......

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)witchan

### 1. 目标

分析某车之家 sign 签名算法的实现

### 2. 操作环境

*   frida
    
*   mac 系统
    
*   Charles 抓包
    
*   越狱 iPhone
    

### 3. 流程

#### 寻找切入点

通过 Charles 抓包获取到关键词为_sign，这也就是我们的切入点：

![](https://witchan-1256170176.cos.ap-chengdu.myqcloud.com/b5c714bac2c5d84a08bf13d683636307.jpeg)

#### 静态分析

在静态分析前，我们先观察 sign 值的特征，比如 32 位就有可能是 md5，数字加字母加 +/ 然后以 = 号结尾的，就有可能是 base64。

通过肉眼观察，发现 sign 签名的长度是 32 位大写，第一直觉就是 MD5，接下来直接进入动态调试去 hook md5 函数，看看该加密是否为 md5

#### 动态分析

使用 frida 工具的`frida-trace -UF -i CC_MD5`命令跟踪 CC_MD5 函数，代码如下：

```
{
  onEnter(log, args, state) {
      log(`CC_MD5(${args[0].readUtf8String()})`);   
  },
  onLeave(log, retval, state) {
    log(`CC_MD5()${hexdump(retval, {length:16})}`);
  }
}
```

执行`frida-trace -UF -i CC_MD5`后，点击登录按钮，日志输出如下：

```
witchan@witchandeAir ~ % frida-trace -UF -i CC_MD5
Instrumenting...
CC_MD5: Loaded handler at "/Users/witchan/__handlers__/libcommonCrypto.dylib/CC_MD5.js"
Started tracing 1 function. Press Ctrl+C to stop.
           /* TID 0x303 */
  6214 ms  CC_MD5(2222)
  6215 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f051548  93 4b 53 58 00 b1 cb a8 f9 6a 5d 72 f7 2f 16 11  .KSX.....j]r./..
  6216 ms  CC_MD5(@7U$aPOE@$Version1_appidapp.iphone_timestamp1658455619autohomeuaiPhone    12.5.5  autohome    11.25.0 iPhoneisCheckModeratorsRemote1isapp1logincode%31%31%31%31%31%31%31%31%31%31%31reffersessionida2b93cb5da721aa55ca1a87b2e919b3d3cd214e6showmob1userpwd934B535800B1CBA8F96A5D72F72F1611validcode3333@7U$aPOE@$)
  6216 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f051478  50 03 73 09 75 58 bb 53 72 80 20 54 b1 39 2b d4  P.s.uX.Sr. T.9+.
  6219 ms  CC_MD5(https://118.116.0.118/api/UserApi/StandardLoginAHLoginAccountLoginService)
  6219 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f051118  17 9a 87 99 d6 48 43 ab 69 a0 b8 62 1d d0 a8 0d  .....HC.i..b....
  6221 ms  CC_MD5(MGCopyAnswerDeviceName)
  6221 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f0504c8  ae 4a a5 c0 f7 11 1f 08 b1 63 88 1a a4 f8 da 9f  .J.......c......
  6221 ms  CC_MD5(MGCopyAnswerProductVersion)
  6221 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f0504c8  a8 d3 5d 76 55 0a f8 1f d8 96 8a 0d a3 29 b0 80  ..]vU........)..
  6222 ms  CC_MD5(MGCopyAnswerDeviceName)
  6222 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f0504c8  ae 4a a5 c0 f7 11 1f 08 b1 63 88 1a a4 f8 da 9f  .J.......c......
  6222 ms  CC_MD5(MGCopyAnswerProductVersion)
  6222 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f0504c8  a8 d3 5d 76 55 0a f8 1f d8 96 8a 0d a3 29 b0 80  ..]vU........)..
  6223 ms  CC_MD5(@7U$aPOE@$apisign1|a2b93cb5da721aa55ca1a87b2e919b3d3cd214e6|autohomebrush|1658455619@7U$aPOE@$)
  6223 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f050498  70 c0 71 0b 37 23 3a 29 d8 44 02 d7 f6 20 a5 0c  p.q.7#:).D... ..
  6227 ms  CC_MD5(MGCopyAnswerProductVersion)
  6227 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f0527b8  a8 d3 5d 76 55 0a f8 1f d8 96 8a 0d a3 29 b0 80  ..]vU........)..
  6228 ms  CC_MD5(MGCopyAnswerProductVersion)
  6228 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f052748  a8 d3 5d 76 55 0a f8 1f d8 96 8a 0d a3 29 b0 80  ..]vU........)..
           /* TID 0x552bf */
```

搜索 _sign 值 500373097558BB5372802054B1392BD4 后发现结果：

```
6216 ms  CC_MD5(@7U$aPOE@$Version1_appidapp.iphone_timestamp1658455619autohomeuaiPhone    12.5.5  autohome    11.25.0 iPhoneisCheckModeratorsRemote1isapp1logincode%31%31%31%31%31%31%31%31%31%31%31reffersessionida2b93cb5da721aa55ca1a87b2e919b3d3cd214e6showmob1userpwd934B535800B1CBA8F96A5D72F72F1611validcode3333@7U$aPOE@$)
  6216 ms  CC_MD5()            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16f051478  50 03 73 09 75 58 bb 53 72 80 20 54 b1 39 2b d4  P.s.uX.Sr. T.9+.
```

#### 结果

sign 就是一个简单的 32 位的大写 MD5

入参：@7U$aPOE@$Version1_appidapp.iphone_timestamp1658455619autohomeuaiPhone   12.5.5  autohome    11.25.0 iPhoneisCheckModeratorsRemote1isapp1logincode%31%31%31%31%31%31%31%31%31%31%31reffersessionida2b93cb5da721aa55ca1a87b2e919b3d3cd214e6showmob1userpwd934B535800B1CBA8F96A5D72F72F1611validcode3333@7U$aPOE@$

逐个分析：

@7U$aPOE@$ 前缀

Version1 版本

_appidapp.iphone 应用标识

_timestamp1658455619 时间戳

autohomeuaiPhone    12.5.5  autohome    11.25.0 iPhone 手机 UA

isCheckModeratorsRemote1 不晓得

isapp1 是否为手机

logincode%31%31%31%31%31%31%31%31%31%31%31 手机号，原始手机号为 11111111111。当入参为 00123456789 时，该字段为 00%3123456789，结论：string 里的 1 替换为 %31

reffer 来源

sessionida2b93cb5da721aa55ca1a87b2e919b3d3cd214e6 队列 id

showmob1 不晓得

userpwd934B535800B1CBA8F96A5D72F72F1611 密码 MD5

validcode3333 验证码

@7U$aPOE@$ 后缀

![](https://avatar.52pojie.cn/data/avatar/001/10/94/58_avatar_middle.jpg)正己 大佬高产啊 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) witchan

> [正己 发表于 2023-3-3 12:32](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=45815892&ptid=1753513)  
> 大佬高产啊

之前的旧文，搬过来的 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) XMDS 感谢分享 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) Bruce_HD 支持支持，来看看来瞧一瞧。![](https://avatar.52pojie.cn/images/noavatar_middle.gif)海是倒过来的天 感谢分享，不过我只能分析一些没加密的 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) KPKP 感谢分享![](https://static.52pojie.cn/static/image/smiley/default/42.gif) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) sishen444 感谢分享，虽然看不懂 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) lx19931130 感谢分享。。我也看不懂。![](https://avatar.52pojie.cn/images/noavatar_middle.gif)likezqc 感谢分享