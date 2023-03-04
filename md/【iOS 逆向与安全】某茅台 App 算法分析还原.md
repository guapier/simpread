> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1752900-1-1.html)

> [md]# 1. 目标某茅台软件的 actParam 算法分析还原。

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)witchan _ 本帖最后由 witchan 于 2023-3-2 09:31 编辑_  

1. 目标
=====

某茅台软件的 actParam 算法分析还原。

2. 使用工具
=======

*   mac 系统
    
*   frida-ios-dump：砸壳
    
*   已越狱 iOS 设备：脱壳及 frida 调试
    
*   IDA Pro：静态分析
    
*   Charles：抓包工具
    
*   Shadowrocket：小火箭，配合 Charles 使用
    

3. 流程
=====

处理启动闪退
------

在 IDA Pro 搜索 SVC 得到如下函数列表：

![](https://witchan-1256170176.cos.ap-chengdu.myqcloud.com/890179_6G2UHYEU4AJ369U.jpg)

NOP 掉 sub_函数的最后一行汇编后，即可正常运行 App

![](https://witchan-1256170176.cos.ap-chengdu.myqcloud.com/890179_YUGYQC7AWRAGR4V.jpg)

处理登录闪退
------

启动 App，在登录页使用命令`frida-trace -UF -m "-[UIViewController viewDidAppear:]"`，然后进入到任意页后再返回登录页，获取到当前类为 XXLoginViewController，再使用命令`frida-trace -UF -m "-[*LoginViewController *]"`，跟踪该类。点击登录后，啥也没获取到，呵呵。不过，看 LoginViewController 类有 loginButton，在该方法看到登录按钮绑定的事件为 LoginViewController 类的 login 方法。使用 IDA Pro 打开该方法：

```
void __cdecl -[XXLoginViewController login](XXLoginViewController *self, SEL a2)
{
  if ( qword_100F492E0 )
    exit(0);
}
```

这么大的 exit 函数，查看该条件是在什么地方赋值，查看该变量的交叉引用

![](https://witchan-1256170176.cos.ap-chengdu.myqcloud.com/890179_6PNEEDD8HSGFAB9.jpg)

在 sub_1002F52F44 函数里对该变量进行赋值。接下来就是处理该函数的赋值逻辑（该方法仅是保证检测环境的 qword_100F492E0 为 0，不排除有其他地方仍然有越狱检测等操作）：

1、处理 embedded.mobileprovision 文件。（重签名后会生成该文件）

```
{
  onEnter(log, args, state) {
        this.fileName = new ObjC.Object(args[2]);
    log(`-[NSBundle pathForResource:${new ObjC.Object(args[2])} ofType:${new ObjC.Object(args[3])}]`);
  },
  onLeave(log, retval, state) {
          if (this.fileName.toString().toLowerCase().indexOf("embedded") != -1 ) {
                  retval.replace(0x0)
          }
          log(`-[NSBundle pathForResource:ofType:]=${new ObjC.Object(retval)}=`);
  }
}
```

2、对 sub_1002F52F44 函数里，对该变量进行赋值的地方，一一处理。保证不会执行 qword_100F492E0=1，处理完后即可正常登录：

![](https://witchan-1256170176.cos.ap-chengdu.myqcloud.com/890179_SGBKTADQCXWTMQ5.jpg)

寻找 actParam 算法
--------------

在 IDA Pro 搜索 actParam 字符串，没有发现该字符，说明该字符串被处理过了。按之前的套路，既然这在 body 里，那我们就使用命令`frida-trace -U -f com.xxx.xxx -m "*[NSMutableURLRequest setHTTPBody:]" -m "*[NSBundle pathForResource:ofType:]"`获取到关键日志如下：

```
-[NSMutableURLRequest setHTTPBody:<7b226163 74506172 616d223a 22496469 77776474 52644542 68646548 6b614a62 71314a35 3972386a 35684c6a 33653334 76576d74 67523374 47486170 4e4c7752 73326237 31495435 614f6c32 42506a44 4f386277 7a793270 34664c34 6f483878 746c3048 78337069 716c6b50 4a4a5555 54677950 6a39397a 6c455750 34375c2f 596f397a 414b7762 6e726768 47585351 526a4f51 706c3636 31694541 347a6157 42657a51 3d3d222c 22697465 6d496e66 6f4c6973 74223a5b 7b22636f 756e7422 3a312c22 6974656d 4964223a 22313030 3536227d 5d2c2273 65737369 6f6e4964 223a3336 362c2273 686f7049 64223a22 31353135 31303132 32303031 227d>
NSMutableURLRequest setHTTPBody called from:
0x1007b8090 /var/containers/Bundle/Application/AFABDD2B-D35D-4F60-A4A1-DFC92CCC3251/iXX_1_2_15.app/XXX!-[AFJSONRequestSerializer requestBySerializingRequest:withParameters:error:]
0x1007b3a24 /var/containers/Bundle/Application/AFABDD2B-D35D-4F60-A4A1-DFC92CCC3251/iXX_1_2_15.app/XXX!-[AFHTTPRequestSerializer requestWithMethod:URLString:parameters:error:]
0x1007ab270 /var/containers/Bundle/Application/AFABDD2B-D35D-4F60-A4A1-DFC92CCC3251/iXX_1_2_15.app/XXX!-[AFHTTPSessionManager dataTaskWithHTTPMethod:URLString:parameters:headers:uploadProgress:downloadProgress:success:failure:]
0x1009557dc /var/containers/Bundle/Application/AFABDD2B-D35D-4F60-A4A1-DFC92CCC3251/iXX_1_2_15.app/XXX!-[XXBaseRequest startWithSuccess:failure:]
0x100a1b724 /var/containers/Bundle/Application/AFABDD2B-D35D-4F60-A4A1-DFC92CCC3251/iXX_1_2_15.app/XXX!-[XXReserveViewController reserve]
0x100a1b0fc XXX!0x2770fc (0x1002770fc)
0x100b0b6e4 /var/containers/Bundle/Application/AFABDD2B-D35D-4F60-A4A1-DFC92CCC3251/iXX_1_2_15.app/XXX!-[RACSubscriber sendNext:]
0x100af3654 /var/containers/Bundle/Application/AFABDD2B-D35D-4F60-A4A1-DFC92CCC3251/iXX_1_2_15.app/XXX!-[RACPassthroughSubscriber sendNext:]
0x1aeca8ac4 UIKitCore!-[UIGestureRecognizerTarget _sendActionWithGestureRecognizer:]
0x1aecb0ccc UIKitCore!_UIGestureRecognizerSendTargetActions
0x1aecae670 UIKitCore!_UIGestureRecognizerSendActions
0x1aecadb9c UIKitCore!-[UIGestureRecognizer _updateGestureWithEvent:buttonEvent:]
0x1aeca1c78 UIKitCore!_UIGestureEnvironmentUpdate
0x1aeca13a8 UIKitCore!-[UIGestureEnvironment _deliverEvent:toGestureRecognizers:usingBlock:]
0x1aeca1188 UIKitCore!-[UIGestureEnvironment _updateForEvent:window:]
0x1af0b97d0 UIKitCore!-[UIWindow sendEvent:]
```

body 里正是我们提交里的参数，根据堆栈可知网络请求是在 [XXReserveViewController reserve] 发起的，使用 IDA Pro 查看对应的代码可知网络请求类是 XXReserveRequest。使用命令`frida-trace -U -f com.xxx.xxx -m "*[XXReserveRequest *]" -m "*[NSBundle pathForResource:ofType:]"`跟踪该类，获取到的日志如下：

```
-[XXReserveRequest setShopId:0x280dfd920]
-[XXReserveRequest setSessionId:0x16f]
-[XXReserveRequest setItemInfoList:0x280ec7f20]
-[XXReserveRequest ydLogId]
-[XXReserveRequest itemInfoList]
-[XXReserveRequest sessionId]
-[XXReserveRequest shopId]
-[XXReserveRequest ydToken]
-[XXReserveRequest requestParams]
-[XXReserveRequest ydLogId]
-[XXReserveRequest itemInfoList]
-[XXReserveRequest sessionId]
-[XXReserveRequest shopId]
-[XXReserveRequest ydToken]
-[XXReserveRequest getTYWZlObj]
-[XXReserveRequest requestParams]={
    actParam = "IdiwwdtRdEBhdeHkaJbq1J59r8j5hLj3e34vWXXgR3sN9Cp03vhtPomwYoD2EtZM6dvuKXTI3BIjWffzkuwBXUIsms+sHaMW2D+1CMCwdZe2sLG0FXnHUIJIpXblOBJrlRAHk9Bn3fFSZsjYhaJoNw==";
}=
-[XXReserveRequest requestHeader]
-[XXReserveRequest getTYWZlObj]
+[XXReserveRequest requestPath]
+[XXReserveRequest requestPath]
+[XXReserveRequest requestType]
+[XXReserveRequest responseDataMapping]
-[XXReserveRequest .cxx_destruct]
```

根据日志可发现 actParam 在 requestParams 方法里返回的，继续使用 IDA Pro 查看该代码

![](https://witchan-1256170176.cos.ap-chengdu.myqcloud.com/890179_NGWBH28Q6T5C392.jpg)

sub 函数一直点进去，就会发现 (unsigned int)CCCrypt(v6, 0LL, 1LL, &v43, 32LL, v27, v28, v29, v41, v26, &v42) ，接下来就是对该函数进行调试，由于该方法，需要特写时间段才能使用。所以在此，我们查看此 sub 函数的交叉引用，发现在下单时，也有调用该加密函数。

使用命令`frida-trace -U -f com.xxx.xxx -m "*[NSBundle pathForResource:ofType:]" -i CCCrypt`打印该参数，在创建订单时，获取到日志如下：

js 代码

```
{
        onEnter: function(log, args, state) {
                this.op = args[0]
                this.alg = args[1]
                this.options = args[2]
                this.key = args[3]
                this.keyLength = args[4]
                this.iv = args[5]
                this.dataIn = args[6]
                this.dataInLength = args[7]
                this.dataOut = args[8]
                this.dataOutAvailable = args[9]
                this.dataOutMoved = args[10]

                log('CCCrypt(' +
                        'op: ' + this.op + '[0:加密,1:解密]' + ', ' +
                        'alg: ' + this.alg + '[0:AES128,1:DES,2:3DES]' + ', ' +
                        'options: ' + this.options + '[1:ECB,2:CBC,3:CFB]' + ', ' +
                        'key: ' + this.key + ', ' +
                        'keyLength: ' + this.keyLength + ', ' +
                        'iv: ' + this.iv + ', ' +
                        'dataIn: ' + this.dataIn + ', ' +
                        'inLength: ' + this.inLength + ', ' +
                        'dataOut: ' + this.dataOut + ', ' +
                        'dataOutAvailable: ' + this.dataOutAvailable + ', ' +
                        'dataOutMoved: ' + this.dataOutMoved + ')')

                if (this.op == 0) {
                        log("dataIn:")
                        log(hexdump(ptr(this.dataIn), {
                                length: this.dataInLength.toInt32(),
                                header: true,
                                ansi: true
                        }))
                        log("key: ")
                        log(hexdump(ptr(this.key), {
                                length: this.keyLength.toInt32(),
                                header: true,
                                ansi: true
                        }))
                        log("iv: ")
                        log(hexdump(ptr(this.iv), {
                                length: this.keyLength.toInt32(),
                                header: true,
                                ansi: true
                        }))
                }
        },
        onLeave: function(log, retval, state) {
                if (this.op == 1) {
                        log("dataOut:")
                        log(hexdump(ptr(this.dataOut), {
                                length: Memory.readUInt(this.dataOutMoved),
                                header: true,
                                ansi: true
                        }))
                        log("key: ")
                        log(hexdump(ptr(this.key), {
                                length: this.keyLength.toInt32(),
                                header: true,
                                ansi: true
                        }))
                        log("iv: ")
                        log(hexdump(ptr(this.iv), {
                                length: this.keyLength.toInt32(),
                                header: true,
                                ansi: true
                        }))
                } else {
                        log("dataOut:")
                        log(hexdump(ptr(this.dataOut), {
                                length: Memory.readUInt(this.dataOutMoved),
                                header: true,
                                ansi: true
                        }))
                }
                log("CCCrypt did finish")
        }
}
```

日志

```
96341 ms  CCCrypt(op: 0x0[0:加密,1:解密], alg: 0x0[0:AES128,1:DES,2:3DES], options: 0x1[1:ECB,2:CBC,3:CFB], key: 0x16eecea70, keyLength: 0x20, iv: 0x2821adbe0, dataIn: 0x2815a80c0, inLength: undefined, dataOut: 0x281794580, dataOutAvailable: 0xa4, dataOutMoved: 0x16eecea68)
 96341 ms  dataIn:
 96341 ms              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
2815a80c0  7b 22 64 65 6c 69 76 65 72 4d 65 74 68 6f 64 22  {"deliverMethod"
2815a80d0  3a 2d 31 2c 22 61 64 64 72 65 73 73 49 6e 66 6f  :-1,"addressInfo
2815a80e0  22 3a 7b 22 73 68 69 70 41 64 64 72 65 73 73 49  ":{"shipAddressI
2815a80f0  64 22 3a 22 33 37 39 36 31 36 30 22 7d 2c 22 75  d":"3711160"},"u
2815a8100  73 65 72 49 64 22 3a 22 31 30 36 39 38 36 36 32  serId":"10008662
2815a8110  38 30 22 2c 22 69 74 65 6d 4c 69 73 74 22 3a 5b  80","itemList":[
2815a8120  7b 22 63 6f 75 6e 74 22 3a 31 2c 22 73 70 75 49  {"count":1,"spuI
2815a8130  64 22 3a 22 34 32 35 22 2c 22 73 74 6f 72 65 49  d":"425","storeI
2815a8140  64 22 3a 22 32 35 31 35 31 30 31 38 38 30 31 30  d":"251510182010
2815a8150  22 7d 5d 7d                                      "}]}
 96341 ms  key:
 96341 ms              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
16eecea70  71 62 68 61 6a 69 6e 6c 64 65 70 6d 75 63 73 6f  qbhajinldepmucso
16eecea80  6e 61 61 61 63 63 67 79 70 77 75 76 63 6a 61 61  naaaccgypwuvcjaa
 96341 ms  iv:
 96341 ms              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
2821adbe0  32 30 31 38 35 33 34 37 34 39 39 36 33 35 31 35  2018534749963515
2821adbf0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
 96345 ms  dataOut:
 96345 ms              0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
281794580  b0 fd c3 5f a8 e8 de 45 80 ac 23 e6 2b d1 3f 2c  ..._...E..#.+.?,
281794590  5d 4b 88 e9 88 99 7a 76 ab 7b c2 c5 88 40 fd 4f  ]K....zv.{...@.O
2817945a0  38 37 c5 48 61 06 b1 a9 33 0d 1a 7d a7 59 3d fb  87.Ha...3..}.Y=.
2817945b0  05 88 cc 18 b8 68 cb ce a7 33 1e 32 f1 1f a2 ae  .....h.....2....
2817945c0  0b 50 04 75 ed 19 36 eb 64 bd d7 79 a8 13 6a 4d  .P.u..6.d..y..jM
2817945d0  9c bf 72 1c 42 6c 41 cc 76 86 85 cf bd 58 5b db  ..r.BlA.v....X[.
2817945e0  05 74 ec e6 55 6e 84 10 ae zz 61 b8 64 81 7b 6f  .t..Un....a.d.{o
2817945f0  02 df 42 a6 13 8e a6 41 ee 1a 0a b5 65 cb 45 29  ..B....F....e.E)
281794600  9b 86 ea 63 f6 8e 92 6c 1a 11 8b 70 09 b7 b5 ad  ...c...l...p....
281794610  c6 f7 2c 70 91 12 eb 62 45 a4 0e b5 75 29 c4 be  ..,p...bE...u)..
 96345 ms  CCCrypt did finish
```

日志里的 dataOut 数据，转换为 base64 后，和接口里的参数一致 (注：日志里的敏感信息被我处理过)。

结果
==

见 CCCrypt 日志。在分析过程中，当我们发现参数是 base64 的时候，也可以先拦截 base64EncodedStringWithOptions 或 CCCrypt 函数，运气好的话，也能快速定位到加密算法。![](https://avatar.52pojie.cn/data/avatar/000/80/73/63_avatar_middle.jpg)askmoon510

> [rogernash 发表于 2023-3-2 17:02](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=45806297&ptid=1752900)  
> 楼主厉害了，要是有提高中签的技术就更好了

那是服务端的算法了，和客户端无关  
咋可能提高中签率呢，只能说实现自动预约，自动做任务赚小茅运 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) guijian 楼主，做 ios 逆向分析多久了？![](https://avatar.52pojie.cn/images/noavatar_middle.gif)唐小样儿 能自动抢茅台吗 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) lzx217 看不懂，但是觉得好厉害 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) tzb199771 同问，修改之后能自动枪茅台吗 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) ErenLuo android 可以一样分析吗 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) 17674083271  
看不懂，但是觉得好厉害![](https://avatar.52pojie.cn/images/noavatar_middle.gif)晴朗的秋天 看不懂，但觉得好厉害。请问这个可以自动抢茅台吗？i 茅台里面只有预约呢？![](https://avatar.52pojie.cn/images/noavatar_middle.gif)guhuishou

> [晴朗的秋天 发表于 2023-3-2 12:38](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=45802671&ptid=1752900)  
> 看不懂，但觉得好厉害。请问这个可以自动抢茅台吗？i 茅台里面只有预约呢？

感觉楼主逆向的应该是提交预约单的关键算法，应该可以用于实现自动预约![](https://avatar.52pojie.cn/images/noavatar_middle.gif)晴朗的秋天

> [guhuishou 发表于 2023-3-2 13:02](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=45802871&ptid=1752900)  
> 感觉楼主逆向的应该是提交预约单的关键算法，应该可以用于实现自动预约

那也好的，就是有时候会经常忘记预约申购呢。