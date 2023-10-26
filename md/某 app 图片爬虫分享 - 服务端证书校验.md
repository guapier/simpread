> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1847904-1-1.html)

> [md]# 前置我们今天要研究的是 获取 app 的数据解密操作：## 1. 解决抓包 首先 先通过代 {过}{滤} 理 chlars 抓包会发现抓不到包，打开 socksDroid 一样抓不到包。

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)miren66

前置我们今天要研究的是 获取 app 的数据解密操作：

1. 解决抓包
-------

首先 先通过代 {过}{滤} 理 chlars 抓包会发现抓不到包，打开 socksDroid 一样抓不到包。

提示：400 No required SSL certificate was sent

那么我们就该解决这个抓不到问题，万能的百度过后，搜寻知识（部分借助于佩奇大佬的课件）

### 服务端证书校验

*   客户端校验
    
    ```
    - 在客户端中预设证书信息
    - 客户端向服务端发送请求，将服务端返回的证书信息（公钥）和客户端预设证书信息进行校验
    
    ```
    
*   服务端校验
    
    ```
    - 在客户端预设证书（p12/bks）
    - 客户端向服务端发送请求时，携带证书信息，在服务端会校验客户端携带过来证书的合法性
    
    ```
    

* * *

服务端证书的校验逻辑：

*   在 apk 打包时，将证书 bks 或 p12 格式的证书保存在 assets 或 raw 等目录。
    
*   安卓代码，发送请求时 【读取证书文件内容】+ 【证书密码】
    
    ![](https://picdl.sunbangyan.cn/2023/10/24/e71400c9bced211a3395993cf0ad31d5.png)
    

**逆向时**，需要实现：

*   获取 bks 或 p12 证书 文件
*   获取证书相关密码
*   将证书导入到 charles，可以实现抓包（bks 格式需要转换 p12 格式）
*   用 requests 发送请求时，携带证书去发送请求

首先获取 证书和证书密码：

```
// hook密码密码
Java.perform(function () {
    var KeyStore = Java.use("java.security.KeyStore");

    KeyStore.load.overload('java.io.InputStream', '[C').implementation = function (v1, v2) {
        var pwd = Java.use("java.lang.String").$new(v2);
        console.log('\n------------')
        console.log("类型：" + this.getType());
        console.log("密码：" + pwd);
        console.log(JSON.stringify(v1));
        //console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));
        var res = this.load(v1, v2);
        return res;
    };
});
// frida -U -f com.paopaotalk.im -l 1.hook_password.js
// 111111

```

具体逻辑是 hook java.security.KeyStore 这个得到证书密码

### Hook 证书文件

在开发时，是将证书文件加载到 `InputStream` 对象中，后续发送请求是会携带。。。

而我们想要获取证书可有两种方式：

*   定位代码，找到加载证书的文件路径，然后去 apk 中寻找。
    
*   直接 Hook 证书加载位置，将证书的内容从`InputStream`写入到自定义文件，实现自动导出【更加通用，甚至都不需要任何逆向】。
    
    注意：手机要对当前 APP 开启本地硬盘操作权限。
    
    ```
    Java.perform(function () {
      var KeyStore = Java.use("java.security.KeyStore");
      var String = Java.use("java.lang.String");
    
      KeyStore.load.overload('java.io.InputStream', '[C').implementation = function (inputStream, v2) {
          var pwd = String.$new(v2);
          console.log('\n------------')
          console.log("密码：" + pwd, this.getType());
    
          if (this.getType() === "pkcs12") {
              console.log("证书开始导出")
              var myArray = new Array(1024);
              for (var i = 0; i < myArray.length; i++) {
                  myArray[i] = 0x0;
              }
              var buffer = Java.array('byte', myArray);
    
              var file = Java.use("java.io.File").$new("/sdcard/Download/meizhitu-" + new Date().getTime() + ".p12");
              var out = Java.use("java.io.FileOutputStream").$new(file);
              var r;
              while ((r = inputStream.read(buffer)) > 0) {
                  out.write(buffer, 0, r);
              }
              console.log("save success!")
              out.close()
          }
    
          var res = this.load(inputStream, v2);
          return res;
      };
    
    });
    
    // frida -U -f com.mmzztt.app -l 2.hook_cert.js
    
    ```
    

运行后得到 结果如图： 密钥  uX39!dd$#rr_XIyb%

![](https://picst.sunbangyan.cn/2023/10/24/afd5b222189fa739ce609a450c928b5b.png)

var file = Java.use("java.io.File").$new("/sdcard/Download/meizhitu-" + new Date().getTime() + ".p12");

这里面可以看到 在咱们的安卓手机 这个路径可以看到 证书。

![](https://picss.sunbangyan.cn/2023/10/24/8525a6a7f5b0e226374e4ca0d9e3e13b.png)

现在把他放到 chlars 中

import p12

![](https://picst.sunbangyan.cn/2023/10/24/71c81c3edd6a7a493041fc15f3593e44.png)

输入密码： uX39!dd$#rr_XIyb%

![](https://picss.sunbangyan.cn/2023/10/24/ffd97c45265caae62f6d096022ef03a3.png)

![](https://picdm.sunbangyan.cn/2023/10/24/1af3f0417d41e38f58b04102a157c159.png)

现在就可以愉快的抓包了。

![](https://picss.sunbangyan.cn/2023/10/24/47fdb028f63fea30bb588ddf6ee8c1a7.png)

2. 解密 算法助手定位：
-------------

这个不用我多说了吧 )

### 一定要先把目标 app 后台清掉。

不敢放原图（怕违规！）：

复制一段返回值 取算法助手日志去搜索。

![](https://picst.sunbangyan.cn/2023/10/24/ce4f4d95120c38b9dd175f226d4b4564.png)

哦 密钥 和 iv 还有加密方式 全给出了包括堆栈！

![](https://picdm.sunbangyan.cn/2023/10/24/e1d6aec16f7e8b73cd133c071dcc821b.png)

![](https://picdm.sunbangyan.cn/2023/10/24/80e8dca8c2eb5e928ed9d4da731b1085.png)

### 多刷新一下 app  我们会发现 iv 是固定的 0809060801020609 但是 key 不是固定的

那么下一阶段就该反编译 看看 具体逻辑

反编译之前 先经过查壳工具一查 360 加壳

![](https://picss.sunbangyan.cn/2023/10/24/7ad88184a9a8300e99a6a2d6c3b301d0.png)

3. 砸壳
-----

我首先使用 frida dump 脱壳发现脱的不是很干净。

然后我采用  珍惜大佬的 fundex  完整脱下壳

脱壳具体流程就不在这演示了百度一下 都有。

![](https://picst.sunbangyan.cn/2023/10/24/114897551c486e481ed9ef506e7a4e51.png)  
![](https://picst.sunbangyan.cn/2023/10/24/60a0ff4ef15d9fd4ab81560c334ff49b.png)

4.jadx 反编译 ＋frida hook
----------------------

### 根据算法助手的堆栈寻找并且 hook

```
解密Iv（Base64）：MDgwOTA2MDgwMTAyMDYwOQ==
解密Iv（Hex）：30383039303630383031303230363039

解密内容（文本）：=bց���|���Ґ��6�m�}

    at de.robv.android.xposed.XposedBridge$LegacyApiSupport.handleAfter(Unknown Source:33)
        at J.callback(Unknown Source:292)
        at LSPHooker_.doFinal(Unknown Source:11)
        at com.apicloud.signature.AESUtils.decrypt(AESUtils.java:91)
        at com.apicloud.signature.UzSignature.jsmethod_aesDecodeCBCSync_sync(UzSignature.java:566)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.uzmap.pkg.uzcore.uzmodule.b$a.a(Unknown Source:2)
        at com.uzmap.pkg.uzcore.uzmodule.b.a(Unknown Source:29)
        at com.uzmap.pkg.uzcore.uzmodule.c.a(Unknown Source:21)
        at com.uzmap.pkg.a.g.a(Unknown Source:2)
        at com.uzmap.pkg.uzcore.uzmodule.a.f.a(Unknown Source:10)
        at com.uzmap.pkg.uzcore.uzmodule.a.a.E(Unknown Source:54)
        at com.uzmap.pkg.uzcore.uzmodule.a.a.a(Unknown Source:0)
        at com.uzmap.pkg.a.e$a.a(Unknown Source:18)
        at com.apicloud.b.a.a.a(Unknown Source:0)
        at com.apicloud.a.c.a$3.invoke(Unknown Source:46)
        at com.eclipsesource.v8.V8.callObjectJavaMethod(Unknown Source:18)
        at com.eclipsesource.v8.V8._executeScript(Native Method)
        at com.eclipsesource.v8.V8.executeScript(Unknown Source:0)
        at com.eclipsesource.v8.V8.executeScript(Unknown Source:15)
        at com.apicloud.a.c.v.a(Unknown Source:2)
        at com.apicloud.a.c.v.b(Unknown Source:2)
        at com.apicloud.a.c.t.a(Unknown Source:4)
        at com.apicloud.a.c.g$2.run(Unknown Source:8)
        at android.os.Handler.handleCallback(Handler.java:938)
        at android.os.Handler.dispatchMessage(Handler.java:99)
        at android.os.Looper.loop(Looper.java:223)
        at android.app.ActivityThread.main(ActivityThread.java:7656)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:592)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:947


```

![](https://picdl.sunbangyan.cn/2023/10/24/0b3dfc3c44165854d7cc66daf864be6a.png)

frida hook 验证位置：

frida hook 代码：

```
let AESUtils = Java.use("com.apicloud.signature.AESUtils");
AESUtils["decrypt"].overload('java.lang.String', '[B', '[B').implementation = function (str, bArr, bArr2) {
    console.log(`AESUtils.decrypt is called: str=${str}, bArr=${bArr}, bArr2=${bArr2}`);
    let result = this["decrypt"](str, bArr, bArr2);
    console.log(`AESUtils.decrypt result=${result}`);
    return result;
};

//注意这里 可以 使用 经过修改 可以显示字节数组
AESUtils.decrypt.overload('java.lang.String', '[B', '[B').implementation = function (str, bArr, bArr2) {
    var byteArrayToStr = function (byteArray) {
       var result = "";
        for (var i = 0; i < byteArray.length; i++) {
             result += String.fromCharCode(byteArray[i]);
         }
           return result;
      };

    console.log("AESUtils.decrypt is called: str=" + str + ", bArr=" + byteArrayToStr(bArr) + ", bArr2=" + byteArrayToStr(bArr2));
}


```

发现就是这里 ！！

接着分析  bArr, bArr2  是 key 和 iv 再向上寻找 堆栈：

at com.apicloud.signature.UzSignature.jsmethod_aesDecodeCBCSync_sync(UzSignature.java:566)

![](https://picss.sunbangyan.cn/2023/10/24/13abccbda884be6dc48cd2c0ec01624a.png)

`uZModuleContext.optString` 是一个可能是特定库、框架或应用程序中的一个函数调用。这个函数通常用于从某个上下文或对象中获取一个字符串值，如果该值不存在或为 null，则返回一个默认值。

接着 hook optString

```
 `hook let UZModuleContext = Java.use("com.uzmap.pkg.uzcore.uzmodule.UZModuleContext");`
`UZModuleContext["optString"].overload('java.lang.String').implementation = function (str) {`
    `console.log(UZModuleContext.optString is called: str=${str});`
    `let result = this["optString"](str);`
    `console.log(UZModuleContext.optString result=${result});`
    `return result;`
`};

```

![](https://picss.sunbangyan.cn/2023/10/26/8253222735c80a7052a06bff672d9237.png)

![](https://picss.sunbangyan.cn/2023/10/26/8e838907ce1da701296e021901f6984d.png)

他向里面添加了 73921  --> 这里是 参数的返回值。

![](https://picdl.sunbangyan.cn/2023/10/24/55e37b641b8148aa0788ffa3b41756c6.png)

73921   得到  b65ca9e9f5667139  猜测一下是不是 md5

先测试一波：

对比发现就是  md5 的 中间 16 位

![](https://picdm.sunbangyan.cn/2023/10/24/18007fa6163d7f249a50001cd7eea843.png)

接着咱们 hook 一下 md5 就 得出

加密方法其实就是 str  md5 取 16 位得出。

结尾撒花！
-----

就是 aes cbc iv 固定 是 0809060801020609  key 是 返回值 的参数 经过 md5 后取得 中间 16 位得出。