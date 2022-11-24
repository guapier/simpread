> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.pediy.com](https://bbs.pediy.com/thread-274941.htm)

> [原创] 关于某好车主 ios 版本分析

前言:  

小弟最近刚学 ios 逆向，刚好最近在分析 hcz 这个 app 的安卓版本它的加密都在底层 so 里, 但是 so 包做了加密, 改了偏移, 还有 frida 反调试不好分析, 这正好符合我遇到困难就换路子的习惯。切换到 ios 看看这个 app 于是就有这篇文章了。app 都做了 SSL 双向认证, 所以抓包有点困难这里不就讲了。主要讲 X-pa-Sign 和 Sparataid 这两个参数。这里我用的是 3.58.1 版本 因为这个砸壳后代码清晰度好些, 新版本的加密方法没改都是通用的 我已经测试过了。

这篇文章包含了 lldb 动态调试, frida-ios-dump 砸壳, 还有 frida-trace 静态分析.

1. 砸壳
-----

1.1. 这里采取 frida-ios-dump 操作

用起来很简单直接下载这个包 然后打开要砸壳的 APP、命令行运行 python dump.py 进程名 然后等待就行.

1.2. 砸完壳后拿到 ipa 文件, 改成 zip 格式, 解压打开找到里面可执行的文件一般跟 app 名称一样. 把他扔到 ida 中，等待 ida 分析完成就好。这里要等待蛮久的。

2.ida+lldb 分析![](https://bbs.pediy.com/upload/attach/202210/884888_75H8A6C9XXTBHMQ.jpg)

搜索要 X-PA-SIGN 要到引用位置

![](https://bbs.pediy.com/upload/attach/202210/884888_VTXR2UN8ZARZGUG.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_W9PKDR7E6ZUNHFU.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_2QG9M24EQRZM9V9.jpg)

定位到这个方法, 对比抓包结果判断这个方法就是用于添加请求头的位置, 分析代码找到 X-PA-SIGN 应该是由下面这个函数返回

![](https://bbs.pediy.com/upload/attach/202210/884888_BTFCBGCHKH88N3X.jpg)

点进去看

![](https://bbs.pediy.com/upload/attach/202210/884888_V8GU5PY2M8PSGPU.jpg)![](https://bbs.pediy.com/upload/attach/202210/884888_5ENTTDSBD2XNVSV.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_FXYYK7Q8YT2HZWC.jpg)对比抓包生成的 sign 和代码分析可差不多知道 X-PA-SIGN 就是 SHA256 加密, 下面我们来分析 sub_1014DACA4 的入参

lldb 调试

在 x-code 目录中找到对于版本的 DeveloperDiskImage.dmg , 路径大 致 / Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/9.2 /DeveloperDiskImage.dmg

双击 DeveloperDiskImage.dmg 在 / Volumes/DeveloperDiskImage/usr/bin/ 下找到 debugserver.

debugserver 复制出来

创建一个 entitlement.xml 文件与 debugserver 保持同一目录

```
 
 com.apple.springboard.debugapplications  get-task-allow  task_for_pid-allow  run-unsigned-code 
```

用 codesign 签名  

codesign -s - --entitlements entitlement.xml -f debugserver

发送到手机中 scp -r debugserver root@localhost -p xxxx :/usr/bin

*   然后运行 debugserver 127.0.0.1:1998 -a  /var/mobile/app 进程

![](https://bbs.pediy.com/upload/attach/202210/884888_STMM785WX92YMHP.jpg)

记得在电脑端转发这个 1998 端口 也可以自己定义 iproxy 装个就好了

![](https://bbs.pediy.com/upload/attach/202210/884888_42H7VYX97DM7EYJ.jpg)

然后再电脑端另起一个窗口启动 lldb

然后再输入 process connect connect://localhost:1998 等待连接

![](https://bbs.pediy.com/upload/attach/202210/884888_R7GNNJY47Z9AJK4.jpg)

连上了 按 c 继续

拿到首地址

![](https://bbs.pediy.com/upload/attach/202210/884888_SAKUQBYA82Q6P36.jpg)  

在刚才那个方法 sub_1014DACA4 下断点看看入参

![](https://bbs.pediy.com/upload/attach/202210/884888_GVK2M7VKQCK3ZDW.jpg)

触发断点, 打印寄存器情况, 这 10 个寄存器代表函数的 10 个入参

![](https://bbs.pediy.com/upload/attach/202210/884888_4VE6WRQHF4T825X.jpg)

分别打印寄存器的值, 这里跟我在安卓上抓到计算 X-PA-SIGN 方法入参对比

![](https://bbs.pediy.com/upload/attach/202210/884888_DFCH2249GHBA99U.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_5FR3R3YU2Q3XZZ4.jpg)

x0 为 http 请求方法、x1 请求路径 、x2 为请求参数、x3 为请求 body base64 编码、x4 时间戳、x5 为平台 、x6 为安卓和 ios 特定的字符串、x7 为固定值 1 那么拿到入参后、继续分析![](https://bbs.pediy.com/upload/attach/202210/884888_5V47VFFKNBEKEJC.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_7RB7S8482BQ6BTP.jpg)

很明显了就是把 GET 请求就是把这些参数按照顺序拼起来

在看看 POST 方法的

![](https://bbs.pediy.com/upload/attach/202210/884888_UEYP7PXQQ6W4GKY.jpg)

也是把参数拼接来, 路径后接一个请求报文的 base64 编码

那么我们先拿到这个函数的结果然后去判断是否是标准的 sh256 加密

![](https://bbs.pediy.com/upload/attach/202210/884888_S3YSYG6DECRYS3H.jpg)![](https://bbs.pediy.com/upload/attach/202210/884888_HEGW9FDXY69HJN7.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_UUZ5NKVW6AJ58PQ.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_S27RCAHQBMJUSZD.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_ACM3E9DEMPT3XVA.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_QCHTKC2JT56HUET.jpg)

好了 到这 X-Pa-Sign 分析完了就是 Sha256 对拼接的参数进行加密

2.1.spartaid
------------

这里开始分析 sparataid 的生成 在 ida 中找到这个生成的位置

![](https://bbs.pediy.com/upload/attach/202210/884888_YVAUW5RV5YD58TH.jpg)

具体这 Lion_meta 有三个方法 通过对前面两个方法进行 hook 大概猜到用于收集设备信息 然后把前两步收集到的信息传入第三个方法进行加密

![](https://bbs.pediy.com/upload/attach/202210/884888_BVRN9WQH4YNE69W.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_QVCKBTTP3FRTGHS.jpg)

hook 该方法拿到的入参和返回值就很明显这是设备信息收集进行加密的方法, 具体分析下这个方法怎么加密的，看代码大概猜了下如下图

![](https://bbs.pediy.com/upload/attach/202210/884888_SEWMFFY5J5KYGA7.jpg)

这就很简单了直接打印下 CCCrypt 的入参和返回值就知道、是那种加密 和 key 是多少

![](https://bbs.pediy.com/upload/attach/202210/884888_VZG6QBBKG4UDHF6.jpg)

![](https://bbs.pediy.com/upload/attach/202210/884888_CXSMHGP845EAHJR.jpg)

很明显了 AES ecb psck7 加密 key 为固定值 OkqDu3nfoVGNIyQA 不过这里要注意一点这个 ios 这个编码出来的结果跟 java 和 python 有一点的不一样 要处理下 这里就不解释了 自己去看吧！

[看雪 2022 KCTF 秋季赛 防守篇规则，征题截止日期 11 月 12 日！（iPhone 14 等你拿！）](https://bbs.pediy.com/thread-274513.htm)

最后于 2022-10-31 16:05 被那年没下雪编辑 ，原因：

[#逆向分析](forum-161-1-118.htm) [#协议分析](forum-161-1-120.htm) [#脱壳反混淆](forum-161-1-122.htm)