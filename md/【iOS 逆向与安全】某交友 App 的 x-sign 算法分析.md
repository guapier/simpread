> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1754811-1-1.html)

> [md]## 1. 目标某交友软件的 x-sign 算法分析。

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)witchan _ 本帖最后由 witchan 于 2023-3-6 16:14 编辑_  

1. 目标
-----

某交友软件的 x-sign 算法分析。

2. 操作环境
-------

*   mac 系统
    
*   frida-ios-dump：砸壳
    
*   Charles：抓包
    
*   已越狱 iOS 设备：脱壳及 frida 调试
    
*   IDA Pro：静态分析
    

3. 流程
-----

### 寻找切入点

在账号密码登录页，点击登录，通过 Charles 抓包获取到关键词为 x-sign，这也就是我们的切入点：

> ![](https://raw.githubusercontent.com/witchan/pic/master/640-20230306160112776.jpeg)

### 分析过程

使用 frida-ios-dump 的砸壳命令`dump.py com.wemomo.momoappdemo1` 砸壳获取到 ipa 文件，再使用 IDA Pro 编译 ipa 文件，然后搜索搜索字符串 x-sign，结果如下:

> ![](https://raw.githubusercontent.com/witchan/pic/master/640-20230306160652839.jpeg)

查找该字符串的交叉引用，发现只有一处方法有使用：

> ![](https://raw.githubusercontent.com/witchan/pic/master/640-20230306161146020.jpeg)

使用 frida-trace 的`frida-trace -UF -m "-[TBSDKRequest setHTTPRequestHeader]"`命令跟踪该函数，发现点击登录按钮，该方法并没有调用，说明该 x-sign 并非抓包获取到的 x-sign 参数。既然 x-sign 在 header 里。那我们再使用`frida-trace -UF -m "-[NSMutableURLRequest setValue:forHTTPHeaderField:]"`命令尝试跟踪设置 header 的函数，js 代码如下:

> ```
> {
> onEnter(log, args, state) {
> log('--------------- start ---------------')
> log(`-[NSMutableURLRequest setValue:${new ObjC.Object(args[2])} forHTTPHeaderField:${new ObjC.Object(args[3])}]`);
> log('NSMutableURLRequest setValue:forHTTPHeaderField:]\n' +
>  Thread.backtrace(this.context, Backtracer.ACCURATE)
>  .map(DebugSymbol.fromAddress).join('\n') + '\n');
> log('--------------- end ---------------')
> },
> onLeave(log, retval, state) {
> }
> }
> 
> ```

点击登录后，获取到关键信息如下：

> ```
> witchan@witchandeMacBook-Air ~ % frida-trace -UF -m "-[NSMutableURLRequest setValue:forHTTPHeaderField:]"
> Instrumenting...
> -[NSMutableURLRequest setValue:forHTTPHeaderField:]: Loaded handler at "/Users/witchan/__handlers__/NSMutableURLRequest/setValue_forHTTPHeaderField_.js"
> Started tracing 1 function. Press Ctrl+C to stop.
>           /* TID 0x303 */
> 5679 ms  --------------- start ---------------
> 5679 ms  -[NSMutableURLRequest setValue:TpBMMTGM5Lcu+FOochRUPZgo8Pg= forHTTPHeaderField:X-SIGN]
> 5679 ms  NSMutableURLRequest setValue:forHTTPHeaderField:]
> 0x101893dcc MomoChat!-[NSMutableURLRequest(CustomHeader) injectParaToHeader:]
> 0x105cdebcc MomoChat!-[MDRequestSerializer requestWithMethod:URLString:parameters:error:]
> 0x1033ceb80 MomoChat!-[AFHTTPSessionManager dataTaskWithHTTPMethod:URLString:parameters:headers:uploadProgress:downloadProgress:success:failure:]
> 0x1033ce294 MomoChat!-[AFHTTPSessionManager POST:parameters:headers:progress:success:failure:]
> 0x105cdc138 MomoChat!-[MDHTTPSessionManager requestWithApiParam:success:failure:]
> 0x105cbd4b4 MomoChat!+[MDApiBase postRequestWithApiParam:]
> 0x104e99904 MomoChat!-[MDLoginService accountLoginWithAccount:params:completion:]
> 0x104e80fd0 MomoChat!-[MDAccountLoginManager requestAccountLoginWithAccount:params:completion:]
> 0x104eb2958 MomoChat!-[MDRegLoginAccountViewController didClickCompleteButton:]
> 0x216999300 UIKitCore!-[UIApplication sendAction:to:from:forEvent:]
> 0x216442424 UIKitCore!-[UIControl sendAction:to:forEvent:]
> 0x216442744 UIKitCore!-[UIControl _sendActionsForEvents:withEvent:]
> 0x2164417b0 UIKitCore!-[UIControl touchesEnded:withEvent:]
> 0x2165bb2d0 UIKitCore!_UIGestureEnvironmentUpdate
> 0x2165b93a8 UIKitCore!-[UIGestureEnvironment _deliverEvent:toGestureRecognizers:usingBlock:]
> 0x2165b9188 UIKitCore!-[UIGestureEnvironment _updateForEvent:window:]
> 5679 ms  --------------- end ---------------
> 
> ```

根据调用堆栈可知设置 x-sign 的函数在`[NSMutableURLRequest injectParaToHeader:]`，继续 trace 该函数`frida-trace -UF -m "-[NSMutableURLRequest injectParaToHeader:]"`，js 代码如下：

```
{
  onEnter(log, args, state) {
    log(`-[NSMutableURLRequest injectParaToHeader:]`);
        var obj = new ObjC.Object(args[2])
        var ivar = obj.$ivars
        for (var key in ivar) {
        log("k:" + key + " v:" + ivar[key]); 
        }
  },
  onLeave(log, retval, state) {
  }
}

```

得到日志如下：

```
  2016 ms  -[NSMutableURLRequest injectParaToHeader:]
  2016 ms  k:isa v:MDApiParam
  2016 ms  k:_compressed v:true
  2016 ms  k:_loggedIn v:true
  2016 ms  k:_licienceKey v:63c92d0c
  2016 ms  k:_paraDic v:{
\nBc4grei+NtA8CMpMOGw=";G90EcFCPigXsWIIvM++Z4Uzf033xcRe1544/byyAWPRDZJJn0TCpp/UgevDQVJ
    "code_version" = 2;
    "map_id" = 782054678088;
\nbkMIYHkZkBGwC3Ix4q5ksZrxiRD/Xq3v2EGGb7Asv5mByy7ZQ6pG1TYZ";9bcNyVCkUsYGBlUTMc/8ZoFy6ClL
}
  2016 ms  k:_backupDic v:{
    "_idfa_" = "21D2A6AF-58D5-4E0A-ABBA-FC5972DAE0BD";
    "_net_" = wifi;
    "_uid_" = 3d6777a2d9e59e5da0cf8b6b75e0f1fb;
    account = 13333333333;
    bindSource = "bind_source_new_login";
    ct = "1661094132.711969";
    lat = 0;
    lng = 0;
    password = e3ceb5881a0a1fdaad01296d7554868d;
    rqid = e3f4be75;
    uid = 3d6777a2d9e59e5da0cf8b6b75e0f1fb;
}
  2016 ms  k:_tokenString v:null
  2016 ms  k:_currentLV v:1
  2016 ms  k:_decodeFailedCount v:0
  2016 ms  k:_signString v:QV+H6sFy/QHCfrFndOKn/IhfnIg=
  2016 ms  k:_isThirdRequest v:false
  2016 ms  k:_delegate v:null
  2016 ms  k:_successSelector v:0x0
  2016 ms  k:_failSelector v:0x0
  2016 ms  k:_errorSelector v:0x0
  2016 ms  k:_identifier v:null
  2016 ms  k:_headers v:null
  2016 ms  k:_cookie v:null
  2016 ms  k:_userInfo v:null
  2016 ms  k:_timeout v:15
  2016 ms  k:_requestType v:3
  2016 ms  k:_urlString v:https://api.immomo.com/api/v2/login?fr=1005362800969
  2016 ms  k:_urlHost v:api.immomo.com
  2016 ms  k:_uploadData v:null
  2016 ms  k:_otherPara v:null
  2016 ms  k:_state v:0
  2016 ms  k:_successCallbackBlock v:[object Object]
  2016 ms  k:_failCallbackBlock v:[object Object]
  2016 ms  k:_errorCallbackBlock v:[object Object]
  2069 ms  -[NSMutableURLRequest injectParaToHeader:]
  2069 ms  k:isa v:MDApiParam
  2069 ms  k:_compressed v:false
  2069 ms  k:_loggedIn v:true
  2069 ms  k:_licienceKey v:null
  2069 ms  k:_paraDic v:{
    "_idfa_" = "21D2A6AF-58D5-4E0A-ABBA-FC5972DAE0BD";
    "_net_" = wifi;
    "_uid_" = 3d6777a2d9e59e5da0cf8b6b75e0f1fb;
    "click_sign" = "click_account_login_button";
    type = "login_register_click";
}
  2069 ms  k:_backupDic v:null
  2069 ms  k:_tokenString v:null
  2069 ms  k:_currentLV v:0
  2069 ms  k:_decodeFailedCount v:0
  2069 ms  k:_signString v:null
  2069 ms  k:_isThirdRequest v:false
  2069 ms  k:_delegate v:null
  2069 ms  k:_successSelector v:0x0
  2069 ms  k:_failSelector v:0x0
  2069 ms  k:_errorSelector v:0x0
  2069 ms  k:_identifier v:null
  2069 ms  k:_headers v:null
  2069 ms  k:_cookie v:null
  2069 ms  k:_userInfo v:null
  2069 ms  k:_timeout v:15
  2069 ms  k:_requestType v:1
  2069 ms  k:_urlString v:https://api.immomo.com/v1/log/common/abtestupload?fr=1005362800969
  2069 ms  k:_urlHost v:api.immomo.com
  2069 ms  k:_uploadData v:null
  2069 ms  k:_otherPara v:null
  2069 ms  k:_state v:0
  2069 ms  k:_successCallbackBlock v:null
  2069 ms  k:_failCallbackBlock v:null
  2069 ms  k:_errorCallbackBlock v:null

```

根据请求头的 x-sign 值发现，x-sign 参数在 MDApiParam 类的_signString 属性里，那我们跟踪 MDApiParam 类的 setSignString: 方法，`frida-trace -UF -m "-[MDApiParam setSignString:]"`，结果跟踪失败。那就说明该方法在 MDApiParam 的父类，那我们调整 trace 命令为`frida-trace -UF -m "-[* setSignString:]"`，js 代码如下：

> ```
> {
> onEnter(log, args, state) {
>   log(`-[MDApiBaseParam setSignString:${new ObjC.Object(args[2])}]`);
>       log('MDApiBaseParam setSignString:]\n' +
>           Thread.backtrace(this.context, Backtracer.ACCURATE)
>           .map(DebugSymbol.fromAddress).join('\n') + '\n');
>       log('--------------- end ---------------')
> },
> onLeave(log, retval, state) {
> }
> }
> 
> ```

跟踪日志如下：

> ```
> 2838 ms  -[MDApiBaseParam setSignString:3XEvF8ZcUY1v7bknbQYWKsrajvQ=]
> 2838 ms  MDApiBaseParam setSignString:]
> 0x1018a6f3c MomoChat!+[MDPrivateKeyGenerator handleRegisterPara:withGenerator:operatorItem:userAgentData:]
> 0x105cc6ec0 MomoChat!-[MDAPIOperatorLayer statusOfSendingRequestPara:]
> 0x105cde7f0 MomoChat!-[MDRequestSerializer requestWithMethod:URLString:parameters:error:]
> 0x1033ceb80 MomoChat!-[AFHTTPSessionManager dataTaskWithHTTPMethod:URLString:parameters:headers:uploadProgress:downloadProgress:success:failure:]
> 0x1033ce294 MomoChat!-[AFHTTPSessionManager POST:parameters:headers:progress:success:failure:]
> 0x105cdc138 MomoChat!-[MDHTTPSessionManager requestWithApiParam:success:failure:]
> 0x105cbd4b4 MomoChat!+[MDApiBase postRequestWithApiParam:]
> 0x104e99904 MomoChat!-[MDLoginService accountLoginWithAccount:params:completion:]
> 0x104e80fd0 MomoChat!-[MDAccountLoginManager requestAccountLoginWithAccount:params:completion:]
> 0x104eb2958 MomoChat!-[MDRegLoginAccountViewController didClickCompleteButton:]
> 0x216999300 UIKitCore!-[UIApplication sendAction:to:from:forEvent:]
> 0x216442424 UIKitCore!-[UIControl sendAction:to:forEvent:]
> 0x216442744 UIKitCore!-[UIControl _sendActionsForEvents:withEvent:]
> 0x2164417b0 UIKitCore!-[UIControl touchesEnded:withEvent:]
> 0x2165bb2d0 UIKitCore!_UIGestureEnvironmentUpdate
> 0x2165b93a8 UIKitCore!-[UIGestureEnvironment _deliverEvent:toGestureRecognizers:usingBlock:]
> 
> 2838 ms  --------------- end ---------------
> 
> ```

根据调用堆栈，在 IDA Pro 查看 [MDPrivateKeyGenerator handleRegisterPara:withGenerator:operatorItem:userAgentData:] 函数，该函数有使用 ollvm 混淆，不过，通过关键词搜索，发现目标函数：

> ![](https://raw.githubusercontent.com/witchan/pic/master/640-20230306160732422.jpeg)

发现生成 sign 的函数为 MDPrivateKeyGenerator 类的 signStringForEncryptedData:userAgentData:withTDKey: 方法。`frida-trace -UF -m "*[MDPrivateKeyGenerator signStringForEncryptedData:userAgentData:withTDKey:]" -m "*[MDPrivateEncryptor encryptData:withPassword:error:]" -m "*[NSJSONSerialization dataWithJSONObject:options:error:]"`该方法，对应 js 代码如下：

[MDPrivateKeyGenerator signStringForEncryptedData:userAgentData:withTDKey:]

> ```
> {
> onEnter(log, args, state) {
>     var obj = new ObjC.Object(args[2]);
>     var ua = new ObjC.Object(args[3]);
>  log(`+[MDPrivateKeyGenerator signStringForEncryptedData:${obj} userAgentData:${ua.bytes().readUtf8String(ua.length())
> } withTDKey:${new ObjC.Object(args[4])}]`);
>   log('MDPrivateKeyGenerator signStringForEncryptedData from:\n' +
>           Thread.backtrace(this.context, Backtracer.ACCURATE)
>           .map(DebugSymbol.fromAddress).join('\n') + '\n');
> },
> onLeave(log, retval, state) {
>     log(`+[MDPrivateKeyGenerator signStringForEncryptedData:userAgentData:withTDKey:]=${new ObjC.Object(retval)}=`);
> }
> }
> 
> ```

[MDPrivateEncryptor encryptData:withPassword:error:]

> ```
> {
> onEnter(log, args, state) {
>     var obj = new ObjC.Object(args[2]);
>  log(`+[MDPrivateEncryptor encryptData:${obj} withPassword:${args[3]} error:${args[4]}]`);
> },
> onLeave(log, retval, state) {
>     var obj = new ObjC.Object(retval);
>     log(`+[MDPrivateEncryptor encryptData:withPassword:error:]=${obj}=`);
> }
> }
> 
> ```

[NSJSONSerialization dataWithJSONObject:options:error:]

> ```
> {
> onEnter(log, args, state) {
>  log(`+[NSJSONSerialization dataWithJSONObject:${new ObjC.Object(args[2])} options:${args[3]} error:${args[4]}]`);
> },
> onLeave(log, retval, state) {
>     var obj = new ObjC.Object(retval);
>     log(`+[NSJSONSerialization dataWithJSONObject:options:error:]=${obj}=`);
> }
> }
> 
> ```

关键日志如下：

> ```+[NSJSONSerialization dataWithJSONObject:{  
> +[NSJSONSerialization dataWithJSONObject:{  
> "_idfa_"="21D2A6AF-58D5-4E0A-ABBA-FC5972DAE0BD";  
> "_net_" = wifi;  
> "_uid_" = 3d6777a2d9e59e5da0cf8b6b75e0f1fb;  
> account = 13333333333;  
> bindSource = "bind_source_new_login";  
> ct = "1661097991.706133";  
> lat = 0;  
> lng = 0;  
> password = e3ceb5881a0a1fdaad01296d7554868d;  
> rqid = b0a993ba;  
> uid = 3d6777a2d9e59e5da0cf8b6b75e0f1fb;  
> } options:0x0 error:0x0]  
> +[NSJSONSerialization dataWithJSONObject:options:error:]=<7b227569 64223a22 33643637 37376132 64396535 39653564 61306366 38623662 37356530 66316662 222c225f 6e65745f 223a2277 69666922 2c226374 223a2231 36363130 39373939 312e3730 36313333 222c225f 69646661 5f223a22 32314432 41364146 2d353844 352d3445 30412d41 4242412d 46433539 37324441 45304244 222c226c 6174223a 2230222c 22706173 73776f72 64223a22 65336365 62353838 31613061 31666461 61643031 32393664 37353534 38363864 222c2262 696e6453 6f757263 65223a22 62696e64 5f736f75 7263655f 6e65775f 6c6f6769 6e222c22 72716964 223a2262 30613939 33626122 2c226163 636f756e 74223a22 31333333 33333333 33333322 2c226c6e 67223a22 30222c22 5f756964 5f223a22 33643637 37376132 64396535 39653564 61306366 38623662 37356530 66316662 227d>=  
> +[MDPrivateEncryptor encryptData:<7b227569 64223a22 33643637 37376132 64396535 39653564 61306366 38623662 37356530 66316662 222c225f 6e65745f 223a2277 69666922 2c226374 223a2231 36363130 39373939 312e3730 36313333 222c225f 69646661 5f223a22 32314432 41364146 2d353844 352d3445 30412d41 4242412d 46433539 37324441 45304244 222c226c 6174223a 2230222c 22706173 73776f72 64223a22 65336365 62353838 31613061 31666461 61643031 32393664 37353534 38363864 222c2262 696e6453 6f757263 65223a22 62696e64 5f736f75 7263655f 6e65775f 6c6f6769 6e222c22 72716964 223a2262 30613939 33626122 2c226163 636f756e 74223a22 31333333 33333333 33333322 2c226c6e 67223a22 30222c22 5f756964 5f223a22 33643637 37376132 64396535 39653564 61306366 38623662 37356530 66316662 227d> withPassword:0x282806bc0 error:0x0]  
> +[MDPrivateEncryptor encryptData:withPassword:error:]=<020350f1 d8c4006a 2daffbca efd85d33 0b76f031 c23d38f6 7c8a1ef9 ed054baa d07aeba0 fb780f76 6822ac1b a68c9254 3cc58d5e 8707e40b 8f68ff0c fbcdea46 db6cc822 1c1e7e84 332805d5 88edcb1a 182a1db7 4c0e4568 75ee7859 b5f4af82 67ec6662 03e41725 740412e2 655d04a9 fc9321fe 5220ed14 076d2a2d 8d840686 0107810f 886cd438 eec7b0ce 4f244ecc acafb5ad 285bec58 e714962d 9e4e5fba 75ac82cd cf23c69e 862f0833 9e337628 c3b79aa3 2571ff8f 4b3f44c9 52357277 5beb8dce f1e9cc9a 6e763b6e eb2ae747 d00d571b 3d668e09 f37dec21 3aaf6bd6 b8714b98 b1b1513c ef5f080c fc07b17b a0cc5991 ac9e58d7 b1c14edb 084b1c0e 05ac1bb8 15b5efcb a64f1f86 035a1e84 b326a8bc 355093d6 dd8a12d8 96bbe3e0 9a252878 18cfc115 78507b3c 20baedf4 495c969b 4e433364 ceddf0a3 f759aab0 4402cfe9 4aa934>=  
> +[MDPrivateKeyGenerator signStringForEncryptedData:<020350f1 d8c4006a 2daffbca efd85d33 0b76f031 c23d38f6 7c8a1ef9 ed054baa d07aeba0 fb780f76 6822ac1b a68c9254 3cc58d5e 8707e40b 8f68ff0c fbcdea46 db6cc822 1c1e7e84 332805d5 88edcb1a 182a1db7 4c0e4568 75ee7859 b5f4af82 67ec6662 03e41725 740412e2 655d04a9 fc9321fe 5220ed14 076d2a2d 8d840686 0107810f 886cd438 eec7b0ce 4f244ecc acafb5ad 285bec58 e714962d 9e4e5fba 75ac82cd cf23c69e 862f0833 9e337628 c3b79aa3 2571ff8f 4b3f44c9 52357277 5beb8dce f1e9cc9a 6e763b6e eb2ae747 d00d571b 3d668e09 f37dec21 3aaf6bd6 b8714b98 b1b1513c ef5f080c fc07b17b a0cc5991 ac9e58d7 b1c14edb 084b1c0e 05ac1bb8 15b5efcb a64f1f86 035a1e84 b326a8bc 355093d6 dd8a12d8 96bbe3e0 9a252878 18cfc115 78507b3c 20baedf4 495c969b 4e433364 ceddf0a3 f759aab0 4402cfe9 4aa934> userAgentData:MomoChat/9.2.6 ios/4268 (iPhone 6; iOS 12.5.5; zh_CN; iPhone7,2; S1) withTDKey:LZekgG6VHCbwKhzTeSHYx+BbJJOUJsggLZekgG6VHCbwKhzT]  
> MDPrivateKeyGenerator signStringForEncryptedData from:  
> 0x1018a6f1c MomoChat!+[MDPrivateKeyGenerator handleRegisterPara:withGenerator:operatorItem:userAgentData:]  
> 0x105cc6ec0 MomoChat!-[MDAPIOperatorLayer statusOfSendingRequestPara:]  
> 0x105cde7f0 MomoChat!-[MDRequestSerializer requestWithMethod:URLString:parameters:error:]  
> 0x1033ceb80 MomoChat!-[AFHTTPSessionManager dataTaskWithHTTPMethod:URLString:parameters:headers:uploadProgress:downloadProgress:success:failure:]  
> 0x1033ce294 MomoChat!-[AFHTTPSessionManager POST:parameters:headers:progress:success:failure:]  
> 0x105cdc138 MomoChat!-[MDHTTPSessionManager requestWithApiParam:success:failure:]  
> 0x105cbd4b4 MomoChat!+[MDApiBase postRequestWithApiParam:]  
> 0x104e99904 MomoChat!-[MDLoginService accountLoginWithAccount:params:completion:]  
> 0x104e80fd0 MomoChat!-[MDAccountLoginManager requestAccountLoginWithAccount:params:completion:]  
> 0x104eb2958 MomoChat!-[MDRegLoginAccountViewController didClickCompleteButton:]  
> 0x216999300 UIKitCore!-[UIApplication sendAction:to:from:forEvent:]  
> 0x216442424 UIKitCore!-[UIControl sendAction:to:forEvent:]  
> 0x216442744 UIKitCore!-[UIControl _sendActionsForEvents:withEvent:]  
> 0x2164417b0 UIKitCore!-[UIControl touchesEnded:withEvent:]  
> 0x2165bb2d0 UIKitCore!_UIGestureEnvironmentUpdate  
> 0x2165b93a8 UIKitCore!-[UIGestureEnvironment _deliverEvent:toGestureRecognizers:usingBlock:]
> 
> +[MDPrivateKeyGenerator signStringForEncryptedData:userAgentData:withTDKey:]=LbQ1kIg2MGf+5uXtUR4MguFYTIE==

经过整理得出：签名函数 [MDPrivateKeyGenerator signStringForEncryptedData:userAgentData:withTDKey:] 的入参分别为：

*   EncryptedData:
    
    调用 [MDPrivateEncryptor encryptData:withPassword:error:] 生成，入参为：
    
    ```
    {
      "_idfa_" = "21D2A6AF-58D5-4E0A-ABBA-FC5972DAE0BD";
      "_net_" = wifi;
      "_uid_" = 3d6777a2d9e59e5da0cf8b6b75e0f1fb;
      account = 13333333333;
      bindSource = "bind_source_new_login";
      ct = "1661097991.706133";
      lat = 0;
      lng = 0;
      password = e3ceb5881a0a1fdaad01296d7554868d;
      rqid = b0a993ba;
      uid = 3d6777a2d9e59e5da0cf8b6b75e0f1fb;
    }
    
    ```
    
*   userAgentData:
    
    `MomoChat/9.2.6 ios/4268 (iPhone 6; iOS 12.5.5; zh_CN; iPhone7,2; S1)`也就是请求头的 user-agent 字段
    
*   TDKey：LZekgG6VHCbwKhzTeSHYx+BbJJOUJsggLZekgG6VHCbwKhzT
    

函数 [MDPrivateKeyGenerator signStringForEncryptedData:userAgentData:withTDKey:] 被混淆过了，抠出关键代码如下：

```
id __cdecl +[MDPrivateKeyGenerator signStringForEncryptedData:userAgentData:withTDKey:](MDPrivateKeyGenerator_meta *self, SEL a2, NSData* inData, NSData* ua, id key)
{
  NSMutableData *data = [[NSMutableData alloc] initWithData:ua];
    [data appendData:inData];
    return [MDPrivateKeyGenerator signDataForData: data withKey: key];
}

```

[MDPrivateKeyGenerator signDataForData:withKey:] 仍然被混淆，继续抠出关键代码如下：

```
  case 1048019035:
                    objc_retainAutorelease(data1);
                    data_byte = objc_msgSend(data1, bytes, v26);
                    v38 = (__int64 *)v33;
                    v22 = (__int64 *)v33;
                    *((_DWORD *)v33 + 4) = 0;
                    *v22 = 0LL;
                    v22[1] = 0LL;
                    objc_retainAutorelease(key2);
                    key_c = objc_msgSend(key2, UTF8String, v26);
                    outData = (__int64 *)v33;
                    dataLength = (unsigned __int64)objc_msgSend(data1, length1, v26);
                    v40 = (unsigned int)sub_101137F5C(data_byte, key_c, outData, dataLength) == 0;
                    v6 = 1136488291;
                    break;
                }

```

继续进入 sub_101137F5C 函数：

```
__int64 __fastcall sub_101137F5C(char *a1, _QWORD *a2, void *a3, signed int length)
{
  size_t v4; // x22
  size_t v5; // x23
  signed int v6; // w9
  char *v7; // x24
  char *v8; // x24
  signed int v9; // w8
  BOOL v11; // [xsp+4h] [xbp-7Ch]
  char *inData; // [xsp+8h] [xbp-78h]
  _QWORD *key; // [xsp+10h] [xbp-70h]
  void *outData; // [xsp+18h] [xbp-68h]
  signed int v15; // [xsp+24h] [xbp-5Ch]
  bool v16; // [xsp+2Bh] [xbp-55h]
  unsigned int v17; // [xsp+2Ch] [xbp-54h]

  inData = a1;
  v11 = length < 1 || a1 == 0LL || a2 == 0LL || a3 == 0LL;
  v4 = length;
  v5 = length + 8;
  v6 = -982936543;
  key = a2;
  outData = a3;
  do
  {
    while ( 1 )
    {
      while ( 1 )
      {
        while ( 1 )
        {
          while ( 1 )
          {
            v9 = v6;
            if ( v6 <= -213277501 )
              break;
            if ( v6 > 1153753992 )
            {
              if ( v6 > 1967097480 )
              {
                if ( v6 == 1967097481 )
                {
                  v7 = (char *)malloc(v5);
                  memcpy(v7, inData, v4);
                  *(_QWORD *)&v7[v4] = *key;
                  sub_101137BEC((__int64)v7, (__int64)outData, v5);
                  free(v7);
                  goto LABEL_8;
                }
                if ( v6 == 1993019028 )
LABEL_6:
                  v6 = -1283420476;
              }
              else if ( v6 == 1153753993 )
              {
                v6 = -1085362011;
              }
              else if ( v6 == 1602184948 )
              {
LABEL_8:
                v6 = -2054010553;
              }
            }
            else if ( v6 > 932805686 )
            {
              if ( v6 == 932805687 )
              {
                v6 = -776523138;
                v16 = v11;
              }
              else if ( v6 == 1133444574 )
              {
                v15 = 0;
                v6 = -816491233;
              }
            }
            else
            {
              v6 = 1153753993;
              if ( v9 != -213277500 )
              {
                v6 = v9;
                if ( v9 == 589407864 )
                  v6 = 1153753993;
              }
            }
          }
          if ( v6 <= -1072802235 )
            break;
          if ( v6 > -816491234 )
          {
            if ( v6 == -816491233 )
            {
              v17 = v15;
              goto LABEL_6;
            }
            if ( v6 == -776523138 )
            {
              if ( v16 )
                v6 = -213277500;
              else
                v6 = 1602184948;
            }
          }
          else
          {
            v6 = 932805687;
            if ( v9 != -1072802234 )
            {
              v6 = v9;
              if ( v9 == -982936543 )
                v6 = 932805687;
            }
          }
        }
        if ( v6 <= -1283420477 )
          break;
        if ( v6 == -1283420476 )
        {
          v6 = -2014741442;
        }
        else if ( v6 == -1085362011 )
        {
          v15 = -970;
          v6 = -816491233;
        }
      }
      if ( v6 != -2054010553 )
        break;
      v8 = (char *)malloc(v5);
      memcpy(v8, inData, v4);
      *(_QWORD *)&v8[v4] = *key;
      sub_101137BEC((__int64)v8, (__int64)outData, v5);
      free(v8);
      v6 = 1133444574;
    }
  }
  while ( v6 != -2014741442 );
  return v17;
}

```

发现该函数最终调用了 sub_101137BEC 函数，继续点进去：

```
__int64 __fastcall sub_101137BEC(__int64 inData, __int64 outData, signed int length)
{
  signed int v3; // w20
  signed int v4; // w9
  signed int v5; // w8
  __int64 v7; // [xsp+0h] [xbp-70h]
  __int64 v8; // [xsp+8h] [xbp-68h]
  void *v9; // [xsp+10h] [xbp-60h]
  bool v10; // [xsp+19h] [xbp-57h]
  bool v11; // [xsp+1Ah] [xbp-56h]
  bool v12; // [xsp+1Bh] [xbp-55h]
  unsigned int v13; // [xsp+1Ch] [xbp-54h]

  v7 = length;
  v10 = inData == 0;
  v4 = -1578351266;
  v8 = inData;
  v9 = (void *)outData;
  v11 = length < 1;
  while ( 1 )
  {
    while ( 1 )
    {
      while ( 1 )
      {
        while ( 1 )
        {
          v5 = v4;
          if ( v4 > -472472763 )
            break;
          if ( v4 > -1578351267 )
          {
            if ( v4 > -1093978826 )
            {
              v4 = -1307992632;
              if ( v5 != -1093978825 )
              {
                v4 = v5;
                if ( v5 == -954022793 )
                {
                  v13 = v3;
                  v4 = 682513349;
                }
              }
            }
            else if ( v4 == -1578351266 )
            {
              if ( v10 || v11 )
                v4 = -1093978825;
              else
                v4 = 1410854159;
            }
            else if ( v4 == -1307992632 )
            {
              v4 = -472472762;
            }
          }
          else if ( v4 > -1972614761 )
          {
            v4 = 566635925;
            if ( v5 != -1972614760 )
            {
              v4 = v5;
              if ( v5 == -1711298395 )
              {
                if ( v12 )
                  v4 = 841496535;
                else
                  v4 = -1093978825;
              }
            }
          }
          else
          {
            v4 = 682513349;
            if ( v5 != -2077253293 )
            {
              v4 = v5;
              if ( v5 == -1988385219 )
              {
                v3 = 0;
                v4 = -954022793;
              }
            }
          }
        }
        if ( v4 > 589005999 )
          break;
        if ( v4 > -69020687 )
        {
          if ( v4 == -69020686 )
          {
            sub_101C0BC50(v8, v7, v9);
            v4 = 589006000;
          }
          else if ( v4 == 566635925 )
          {
            v4 = -1988385219;
          }
        }
        else if ( v4 == -472472762 )
        {
          v3 = -978;
          v4 = -954022793;
        }
        else if ( v4 == -318483307 )
        {
          v4 = -1307992632;
        }
      }
      if ( v4 > 841496534 )
        break;
      if ( v4 == 589006000 )
      {
        v12 = sub_101C0BC50(v8, v7, v9) != 0LL;
        v4 = -1711298395;
      }
      else if ( v4 == 682513349 )
      {
        v4 = 1752038933;
      }
    }
    v4 = 566635925;
    if ( v5 != 841496535 )
    {
      v4 = 589006000;
      if ( v5 != 1410854159 )
      {
        v4 = v5;
        if ( v5 == 1752038933 )
          break;
      }
    }
  }
  return v13;
}

```

发现该函数最终调用了 sub_101C0BC50 函数，再点进去：

```
_DWORD *__fastcall sub_101C0BC50(char *inData, __int64 length, void *outData)
{
  __int64 length2; // x20
  char *inData2; // x21
  _DWORD *outData2; // x19
  __int64 v7; // [xsp+0h] [xbp-90h]

  length2 = length;
  inData2 = inData;
  if ( outData )
    outData2 = outData;
  else
    outData2 = &unk_10AC6B920;
  if ( !(unsigned int)sub_101C0BC08(&v7) )
    return 0LL;
  sub_101C0AA00(&v7, inData2, length2);
  sub_101C0BB24(outData2, (unsigned int *)&v7);
  sub_101C06228((__int64)&v7, 96LL);
  return outData2;
}

```

使用 frida 脚本打印 sub_101C0BC50 参数：

js 代码：

```
var addr = 0x1C0BC50
var baseAddr = Module.findBaseAddress('MomoChat')!;
Interceptor.attach(baseAddr.add(addr), {
    onEnter: function(args) {
        console.log(addr.toString(16), "=begin tatget Addr====================================================================");
        console.log(args[0].readCString());
        console.log(args[1]);
    }, onLeave: function(retval) { 
        var ret = ObjC.classes.NSData.dataWithBytes_length_(retval, 20);
        console.log("ret:", ret)
        console.log(addr.toString(16), "=end  tatget Addr====================================================================");
    }
});

```

日志如下：

```
1c0bc50 =begin tatget Addr====================================================================
���Ls�a'
        ��SRf�{�^�؛N$�6�����.�-�
0x31
ret: <e60f257b e0d30731 1767b31d 69877e97 2c4a6b68>
1c0bc50 =end  tatget Addr====================================================================
1c0bc50 =begin tatget Addr====================================================================
�%{
0x4
ret: <efc9e314 8643a3a5 6c68f597 d7786dc8 6e612142>
1c0bc50 =end  tatget Addr====================================================================
1c0bc50 =begin tatget Addr====================================================================
{"uid":"3d6777a2d9e59e5da0cf8b6b75e0f1fb","_net_":"wifi","ct":"1661534490.693195","_idfa_":"21D2A6AF-58D5-4E0A-ABBA-FC5972DAE0BD","lat":"0","password":"e3ceb5881a0a1fdaad01296d7554868d","bindSource":"bind_source_new_login","rqid":"de7161c2","account":"13333333333","lng":"0","_uid_":"3d6777a2d9e59e5da0cf8b6b75e0f1fb"}
0x13e
ret: <305af756 afbb064b a1786925 657d15f9 2c45ae27>
1c0bc50 =end  tatget Addr====================================================================
1c0bc50 =begin tatget Addr====================================================================
0Z�V
0x4
ret: <cd712d01 ef9086a0 5df8ad81 45f84e75 3275f38f>
1c0bc50 =end  tatget Addr====================================================================
1c0bc50 =begin tatget Addr====================================================================
MomoChat/9.2.6 ios/4268 (iPhone 6; iOS 12.5.5; zh_CN; iPhone7,2; S1)0Z�V
0x193
ret: <7ab2461f 1b9fc68d 79469c19 2bb653a9 2ef706a6>
1c0bc50 =end  tatget Addr====================================================================
1c0bc50 =begin tatget Addr====================================================================
}��
0x4
ret: <97d96923 4d342ba2 d7767838 838707fd 6dc19ebb>
1c0bc50 =end  tatget Addr====================================================================

```

<7ab2461f 1b9fc68d 79469c19 2bb653a9 2ef706a6> 该数据转换成 base64 后，正是 x-sign。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

结果
--

sub_101C0BC50 函数就是最终生成 x-sign 的地方，sub_101C0AA00 函数对入参进行加工，sub_101C0BB24 最终 生成 x-sign 参数。sub_101C0BB24 函数的代码如下：

```
signed __int64 __fastcall sub_101C0BB24(_DWORD *a1, unsigned int *inData)
{
  unsigned int *inData2; // x20
  _DWORD *outData; // x19
  unsigned int *v4; // x21
  __int64 v5; // x8
  signed __int64 v6; // x9
  unsigned int v7; // w8

  inData2 = inData;
  outData = a1;
  v4 = inData + 7;
  v5 = inData[23];
  *((_BYTE *)inData + v5 + 28) = -128;
  v6 = v5 + 1;
  if ( (unsigned int)v5 >= 0x38 )
  {
    bzero((char *)v4 + v6, 63 - v5);
    sub_101C0AB08(inData2, v4, 1LL);
    v6 = 0LL;
  }
  bzero((char *)v4 + v6, 56 - v6);
  v7 = bswap32(inData2[5]);
  inData2[21] = bswap32(inData2[6]);
  inData2[22] = v7;
  sub_101C0AB08(inData2, v4, 1LL);
  inData2[23] = 0;
  sub_101C06228((__int64)v4, 64LL);
  *outData = bswap32(*inData2);
  outData[1] = bswap32(inData2[1]);
  outData[2] = bswap32(inData2[2]);
  outData[3] = bswap32(inData2[3]);
  outData[4] = bswap32(inData2[4]);
  return 1LL;
}

```

这应该就是生成 x-sign 的最终函数，就先分析到这吧。![](https://avatar.52pojie.cn/images/noavatar_middle.gif)p5525 比较深奥，看不懂