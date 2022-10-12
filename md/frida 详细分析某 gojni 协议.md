> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1698434-1-1.html)

> [md]# frida 详细分析某 gojni 协议 frida 是非常流行的 hook 框架，在安卓逆向时无论 Java 层还是 Native 层都可以 hook，而且在 Native 层还提供寄存器上......

![](https://avatar.52pojie.cn/data/avatar/000/87/33/78_avatar_middle.jpg)iokeyz _ 本帖最后由 iokeyz 于 2022-10-12 14:45 编辑_  

frida 详细分析某 gojni 协议
====================

frida 是非常流行的 hook 框架，在安卓逆向时无论 Java 层还是 Native 层都可以 hook，而且在 Native 层还提供寄存器上下文 (`context`)，根据汇编代码，可以方便的分析函数参数和返回值，进一步分析作用。

最近遇到了一个 golang 编写的 so，没有混淆，比较简单，麻烦一点的是其中有个大函数，今天就用 frida 的 `context` 来详细分析一下。

前戏
--

遇到这种 so 我最先想到的是 IDA 动态调试，这样最节省时间，跟个一遍两遍基本就搞定了，参考【2】

但这个 app 比较奇怪，我知道的姿势使了个遍还是没调成，也没找到反调的地方，可以说是寸步难行，可能关键是我太菜了，，，具体症状为：attach 之后还没到断点就 crash，即使某次运气好到了断点，f7/f8 后不是提示找不到进程就是提示没法断下，但 Java 层还活着，希望有经验的师傅能给小弟点提示，感谢！

为脱敏 app 就不提供了，IDA 分析的 so 数据库在这：[https://www.lanzouy.com/ih1O10dp7rfi](https://www.lanzouy.com/ih1O10dp7rfi)

抓包
--

先上 fiddler，结果所有请求的返回竟然都是 fiddler！？竟然被识别了！可恶！

![](https://attach.52pojie.cn/forum/202210/12/135935dr27vfqqv2ik7r7a.png)

**01.png** _(116.42 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk2OHw2Yjc5ZWQ2YnwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

换成 `BurpSuite/Charles` 也都抓不到包，返回都是证书相关的错误；是有 `ssl-pinning` 吗？尝试用 `objection` 过掉；`sslpinning disable` 发现了 `okhttp3`

![](https://attach.52pojie.cn/forum/202210/12/135937v06qzv8gsmbfy55a.png)

**02.png** _(94.92 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk2OXw1ZWJhNzY5YXwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

再次抓包还是被识别。没关系，还有杀器，尝试用 `r0capture` 抓包，这次终于抓到了，但是却没有证书，有点意外：

![](https://attach.52pojie.cn/forum/202210/12/135939i11s5rb12nogh25n.png)

**03.png** _(106.77 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk3MHwyNGQ5MzgyOHwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

尝试用我多年练就的手速抓到之后立即复制到 `Apipost` 请求，结果成功了，那可能只是对常用的抓包工具做了识别，没有内置证书；后来试了下小黄鸟也可以抓，大意了啊，估计黄鸟还没被识别，看来这手速白练了...

![](https://attach.52pojie.cn/forum/202210/12/135941zssbanmwg1xoa81m.png)

**04.png** _(57.11 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk3MXw3YmU2MGMzOXwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

可以看到其中关键的 `header` 都是 `EL` 开头，请求体也被加密了，所幸返回的都是明文。在上上图调用堆栈中发现了 `Interceptor` 的踪迹，看着名字应该是重写的 `intercept(Chain chain)`，接下来我们看一下。

Java 层分析
--------

把包拖入 `Jadx`，定位到 `BaseHeaderInterceptor` 类，如下图：

![](https://attach.52pojie.cn/forum/202210/12/135944ytpawegwlw6wpgtw.png)

**05.png** _(70.51 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk3Mnw2ZDhlMGIxMHwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

确实是重写了 `intercept(Chain chain)` 方法，该方法一般通过 `Chain` 类获取所有 `Request` 类，它包含了请求的所有重要信息，包括 `header/body/url` `等等；Interceptor` 这是个非常重要的类，建立连接、发送请求、处理响应等，更多信息可以阅读[官方文档](https://square.github.io/okhttp/features/interceptors/)。

眼尖的应该一下子就发现了关键位置，第二行代码通过 `chain.request()` 得到 `Request` 类然后做了处理；这里的 `i` 是抽象函数，根据 `BaseHeaderInterceptor` 的子类 `d.m` 找到了 `i` 的函数体，如下：

![](https://attach.52pojie.cn/forum/202210/12/135946gz2onoazf62b4ftu.png)

**06.png** _(54.67 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk3M3xkOWY2NzgyNXwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

进入 m()：  
![](https://attach.52pojie.cn/forum/202210/12/135948gitjo2afo2fn7cui.png)

**07.png** _(126.83 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk3NHwyODJlNWE2MXwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

  
终于看到了 `EL-*`，都存在 `c4` 里面，是 `ZeusHelper.f8295b.c()` 返回的：  
![](https://attach.52pojie.cn/forum/202210/12/135950r1zrsexe63rf608f.png)

**08.png** _(30.4 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk3NXxhM2JkOWJmOXwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

`Zeus.getSign()` 是一个 native 函数，先用 frida hook 一下传入参数和返回值：

```
Java.perform(function(){
    // 直接 hook ZeusHelper.c() 和 getSign() 一样的，只是返回值更好看一点
    let ZeusHelper = Java.use("com.easylive.sdk.network.zeus.ZeusHelper");
    ZeusHelper["c"].implementation = function (str, str2, str3, i2, z) {
        console.log('- called : ' + 'str: ' + str + ', ' + 'str2: ' + str2 + ', ' + 'str3: ' + str3 + ', ' + 'i2: ' + i2 + ', ' + 'z: ' + z);
        let ret = this.c(str, str2, str3, i2, z);
        console.log('- retval :' + ret);
        return ret;
    };
})

```

输出如下：

```
- called : str: /app/rank/contributor/list, str2: bOhwNXsKDstJq1uqZs25UguJ9YYdIHMj, str3: displaySurpass=0&name=20193333&start=0&count=3&sessionid=bOhwNXsKDstJq1uqZs25UguJ9YYdIHMj&type=YEAR, i2: 1, z: true
- retval : SignResult(contentType=text/plain, elauth=bOhwNXsKDstJq1uqZs25UguJ9YYdIHMj, elect=1, elns=+p8GVEjQllUhBdyima4dxw==, elsign=ubrYqLFE3R9Mmh/5hXuRofy0COi941Nn, elver=E1.0, body=7p1CFhK2d3dgInF284bGbpnRAGIDKtJDN9+4erd6kEO7TTX7wlXTKLewexv8lemT2DwWoybLv7N47tYOjOHlYA==)

```

传入的五个参数分别是 `url path、登陆后返回的 sessionid、url-encode 的请求体、ect、一个布尔值`，返回了所有需要的。

分析 gojni
--------

这个 so 是 golang 编写的，没有混淆，分析起来比较省头发，不过在此之前还是需要了解一下 go 的一些特点：

1.  go 的字符串是一个指针加一个长度的形式，而 slice 是加一个长度和一个容量，结构体如下：
    
    ```
    type stringStruct struct {
        str unsafe.Pointer
        len int
    }
    type slice struct {
        array unsafe.Pointer
        len   int
        cap   int
    }
    
    ```
    
2.  Arm 中常用的栈是 `sp < bp` 的，也就是递减的，`临时变量 < sp`，`可用堆栈 > sp`
    
3.  go 是支持多返回值的：
    
    1.  `fastcall` 应该是 `x0,x1,...` 这样返回值
    2.  `usercall` 那返回值应该在栈上，比如：`sp+8` 是传入参数，返回值应该在 `sp+16,sp+24,...`
4.  使用 IDA pro 可以分析出许多 go runtime 函数，对照源码就能扫清分析的障碍
    

### 禁止套娃

`Java_zeus_Zeus_getSign`：把传入的 `java string` 转成了 `go string`  
![](https://attach.52pojie.cn/forum/202210/12/135953uj652jeex9dzsp67.png)

**10.png** _(79.73 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk3NnxlOGVmMDA4M3wxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

`proxyzeus__GetSign`：crosscall2 函数是 c 调用 go 时使用的，参数分别是：cgo 函数指针，参数组（对应调用参数和返回值的结构体指针），参数组长度，ctxt；详细了解可以[参照源码](https://cs.opensource.google/go/go/+/master:src/runtime/cgocall.go)。  
![](https://attach.52pojie.cn/forum/202210/12/135955tsvswq71ecpw7wqc.png)

**11.png** _(72.44 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk3N3w4MzM5YmQ4NHwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

  
`main.proxyzeus__GetSign`  
![](https://attach.52pojie.cn/forum/202210/12/135957s441d2747is54tqq.png)

**12.png** _(16.42 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk3OHxlMGFkMDEwMnwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

  
`main.proxyzeus__GetSign`：有转成了 `go slice`  
![](https://attach.52pojie.cn/forum/202210/12/135959k119zeheah71nexq.png)

**13.png** _(109.48 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk3OXwyODUwMzMzMXwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 13:59 上传

  
`zeus_mobile.GetSign`：对 elect 做了判断  
![](https://attach.52pojie.cn/forum/202210/12/140007mgjoj2bnofajr0g0.png)

**14.png** _(43.46 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk4Mnw5OWQyOTc2MXwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 14:00 上传

五个参数经过五层传递转换，最后到了 `zeus_mobile_zeus.Sign()`，已经套的我晕头转向了，但整体还是比较简单的，其实就是 `java string` 转成了 `go slice`

### 笨方法 hook

`zeus_mobile_zeus.Sign()` 这个函数挺复杂的，有数十个函数调用，两千多条汇编：

![](https://attach.52pojie.cn/forum/202210/12/140003wmf3hmh1t8ld4v1v.png)

**15.png** _(186.88 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk4MHxiZjY0Y2FlNXwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 14:00 上传

因为是多返回值，所以有很多黄色的未知变量，看汇编就清晰很多了，以 encoding_base64._ptr_Encoding.EncodeToString 为例：

![](https://attach.52pojie.cn/forum/202210/12/140005piju55kmbz4m9kgn.png)

**16.png** _(44.11 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU1OTk4MXxmYjRjOTJkMHwxNjY1NTY5MTgyfDkzMDQwMnwxNjk4NDM0&nothumb=yes)

2022-10-12 14:00 上传

很明显是 `usercall`，传入的是：`bp-1c0h,bp-1c8h,bp-1d0h,bp-1d8h`，返回值为：`bp-1b0h,bp-1b8h`，下面是用 frida hook 该函数，在 OnEnter 和 OnLeave 中分别打印这些值：

```
Java.perform(function(){
    // 3562B0  encoding_base64._ptr_Encoding.EncodeToString
    Interceptor.attach(Module.findBaseAddress("libgojni.so").add(0x3562B0), {
        onEnter: function (args) {
            // 计算 bp 的地址
            this.cbp = this.context.sp.add(0x1E0);
            // 时间戳
            var timestamp = Date.now();
            console.log(timestamp, "enter: encoding_base64._ptr_Encoding.EncodeToString()", 
                this.cbp.add(-0x1D8).readU64().toString(16),
                this.cbp.add(-0x1D0).readU64().toString(16),
                this.cbp.add(-0x1C8).readU64().toString(16),
                this.cbp.add(-0x1C0).readU64().toString(16),
            );
        },
        onLeave: function (rets) {
            var timestamp = Date.now();
            console.log(timestamp, "leave: encoding_base64._ptr_Encoding.EncodeToString()",
                this.cbp.add(-0x1B8).readU64().toString(16),
                this.cbp.add(-0x1B0).readU64().toString(16),
            );
        }
    });
})

```

刷新一下 app：

```
1665543749563 enter: encoding_base64._ptr_Encoding.EncodeToString() 4000118000 40000a0030 10 18
1665543749564 leave: encoding_base64._ptr_Encoding.EncodeToString() 40000a0078 18

```

根据上面提到的 `string` 和 `slice` 结构，可以大胆的猜测参数中有个 `slice`，返回值是`string`：

```
// 加入这两行，分别到 OnEnter 和 OnLeave
let length = this.cbp.add(-0x1C8).readU64();
console.log(this.cbp.add(-0x1D0).readPointer().readByteArray(length));

let length = this.cbp.add(-0x1B0).readU64()
console.log(this.cbp.add(-0x1B8).readPointer().readByteArray(length));

```

打印出来了：

```
1665543749563 enter: encoding_base64._ptr_Encoding.EncodeToString() 4000118000 40000a0030 10 18
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  42 23 2e ee 06 9b 44 18 18 2c 33 d3 21 62 22 a2  B#....D..,3.!b".
1665543749564 leave: encoding_base64._ptr_Encoding.EncodeToString() 40000a0078 18
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  51 69 4d 75 37 67 61 62 52 42 67 59 4c 44 50 54  QiMu7gabRBgYLDPT
00000010  49 57 49 69 6f 67 3d 3d                          IWIiog==

```

利用这种笨方法，挨个 hook 这些函数调用，代码如下：

```
Java.perform(function(){
    // 0x5BAEE0 zeus_mobile_security.AesCbcEncryptWithBase64 -> body
    Interceptor.attach(Module.findBaseAddress("libgojni.so").add(0x5BAEE0), {
        onEnter: function (args) {
            this.cbp = this.context.sp.add(0x1E0);
            var timestamp = Date.now();
            let length1 = this.cbp.add(-0x1D0).readU64();
            let length2 = this.cbp.add(-0x1B8).readU64();
            let length3 = this.cbp.add(-0x1A0).readU64();
            console.log(timestamp, "enter: zeus_mobile_security.AesCbcEncryptWithBase64()", 
                this.cbp.add(-0x1D8).readU64().toString(16),
                this.cbp.add(-0x1D0).readU64().toString(16),
                this.cbp.add(-0x1C8).readU64().toString(16),
                this.cbp.add(-0x1C0).readU64().toString(16),
                this.cbp.add(-0x1B8).readU64().toString(16),
                this.cbp.add(-0x1B0).readU64().toString(16),
                this.cbp.add(-0x1A8).readU64().toString(16),
                this.cbp.add(-0x1A0).readU64().toString(16),
                this.cbp.add(-0x198).readU64().toString(16),
            );
            console.log(this.cbp.add(-0x120).toString(), this.cbp.add(-0x1C0).readPointer().toString());
            console.log(this.cbp.add(-0x1D8).readPointer().readByteArray(length1));
            console.log(this.cbp.add(-0x1C0).readPointer().readByteArray(length2));
            console.log(this.cbp.add(-0x1A8).readPointer().readByteArray(length3));
        },
        onLeave: function (rets) {
            var timestamp = Date.now();
            let length = this.cbp.add(-0x188).readU64()
            console.log(timestamp, "leave: zeus_mobile_security.AesCbcEncryptWithBase64()",
                this.cbp.add(-0x190).readU64().toString(16),
                this.cbp.add(-0x188).readU64().toString(16),
            );
            console.log(this.cbp.add(-0x190).readPointer().readByteArray(length));
        }
    });

    // 5BB440  zeus_mobile_security.SFib
    Interceptor.attach(Module.findBaseAddress("libgojni.so").add(0x5BB440), {
        onEnter: function (args) {
            this.cbp = this.context.sp.add(0x1E0);
            var timestamp = Date.now();
            let length1 = this.cbp.add(-0x1D0).readU64();
            let length2 = this.cbp.add(-0x1B8).readU64();
            console.log(timestamp, "enter: zeus_mobile_security.SFib()", 
                this.cbp.add(-0x1D8).readU64().toString(16),
                this.cbp.add(-0x1D0).readU64().toString(16),
                this.cbp.add(-0x1C8).readU64().toString(16),
                this.cbp.add(-0x1C0).readU64().toString(16),
                this.cbp.add(-0x1B8).readU64().toString(16),
            );
            console.log(this.cbp.add(-0x1D8).readPointer().readByteArray(length1));
            console.log(this.cbp.add(-0x1C0).readPointer().readByteArray(length2));
        },
        onLeave: function (rets) {
            var timestamp = Date.now();
            let length = this.cbp.add(-0x1A8).readU64()
            console.log(timestamp, "leave: zeus_mobile_security.SFib()",
                this.cbp.add(-0x1B0).readU64().toString(16),
                this.cbp.add(-0x1A8).readU64().toString(16),
            );
            console.log(this.cbp.add(-0x1B0).readPointer().readByteArray(length));
        }
    });

    // // 440490  crypto_sha512.Sum512
    Interceptor.attach(Module.findBaseAddress("libgojni.so").add(0x440490), {
        onEnter: function (args) {
            this.cbp = this.context.sp.add(0x1E0);
            var timestamp = Date.now();
            let length1 = this.cbp.add(-0x1D0).readU64();
            console.log(timestamp, "enter: crypto_sha512.Sum512()", 
                this.cbp.add(-0x1D8).readU64().toString(16),
                this.cbp.add(-0x1D0).readU64().toString(16),
                this.cbp.add(-0x1C8).readU64().toString(16),
            );
            console.log(this.cbp.add(-0x1D8).readPointer().readByteArray(length1));
        },
        onLeave: function (rets) {
            var timestamp = Date.now();
            // let length = this.cbp.add(-0x188).readU64()
            console.log(timestamp, "leave: crypto_sha512.Sum512()", "\n",
                this.cbp.add(-0x1C0).readByteArray(64),
            );
            // console.log(this.cbp.add(-0x190).readPointer().readByteArray(length));
        }
    });

    // // 3562B0  encoding_base64._ptr_Encoding.EncodeToString
    Interceptor.attach(Module.findBaseAddress("libgojni.so").add(0x3562B0), {
        onEnter: function (args) {
            this.cbp = this.context.sp.add(0x1E0);
            var timestamp = Date.now();
            let length1 = this.cbp.add(-0x1C8).readU64();
            console.log(timestamp, "enter: encoding_base64._ptr_Encoding.EncodeToString()", 
                this.cbp.add(-0x1D8).readU64().toString(16),
                this.cbp.add(-0x1D0).readU64().toString(16),
                this.cbp.add(-0x1C8).readU64().toString(16),
                this.cbp.add(-0x1C0).readU64().toString(16),
            );
            console.log(this.cbp.add(-0x1D0).readPointer().readByteArray(length1));
        },
        onLeave: function (rets) {
            var timestamp = Date.now();
            let length = this.cbp.add(-0x1B0).readU64()
            console.log(timestamp, "leave: encoding_base64._ptr_Encoding.EncodeToString()",
                this.cbp.add(-0x1B8).readU64().toString(16),
                this.cbp.add(-0x1B0).readU64().toString(16),
            );
            console.log(this.cbp.add(-0x1B8).readPointer().readByteArray(length));
        }
    });

    // 458CE0  crypto_rand.Read
    Interceptor.attach(Module.findBaseAddress("libgojni.so").add(0x458CE0), {
        onEnter: function (args) {
            this.cbp = this.context.sp.add(0x1E0);
            var timestamp = Date.now();
            let length1 = this.cbp.add(-0x1D0).readU64();
            console.log(timestamp, "enter: crypto_rand.Read()", 
                this.cbp.add(-0x1D8).readU64().toString(16),
                this.cbp.add(-0x1D0).readU64().toString(16),
                this.cbp.add(-0x1C8).readU64().toString(16),
            );
            console.log(this.cbp.add(-0x1D8).readPointer().readByteArray(length1));
        },
        onLeave: function (rets) {
            var timestamp = Date.now();
            let length = this.cbp.add(-0x1D0).readU64()
            console.log(timestamp, "leave: crypto_rand.Read()",
                this.cbp.add(-0x1D8).readU64().toString(16),
                this.cbp.add(-0x1D0).readU64().toString(16),
                this.cbp.add(-0x1C8).readU64().toString(16),
                this.cbp.add(-0x1B8).readU64().toString(16),
                this.cbp.add(-0x1B0).readU64().toString(16),
            );
            console.log(this.cbp.add(-0x1D8).readPointer().readByteArray(length));
        }
    });

    // 320250 time.Now 
    Interceptor.attach(Module.findBaseAddress("libgojni.so").add(0x320250), {
        onEnter: function (args) {
            this.cbp = this.context.sp.add(0x1E0);
            var timestamp = Date.now();
            let length1 = this.cbp.add(-0x1D0).readU64();
            console.log(timestamp, "enter: time.Now()");
        },
        onLeave: function (rets) {
            var timestamp = Date.now();
            let length = this.cbp.add(-0x1C8).readPointer().add(4).readU64()
            console.log(timestamp, "leave: time.Now()",
                this.cbp.add(-0x1D8).readU64().toString(16),
                this.cbp.add(-0x1D0).readU64().toString(16),
                this.cbp.add(-0x1C8).readU64().toString(16),
            );
            // console.log(this.cbp.add(-0x1C8).readPointer().readPointer().readByteArray(length));
        }
    });
})

```

写起来还是比较轻松的，照着抄一下汇编就行了，代码都差不多，输出大概如下，对照着汇编代码分析：

```
# 生成 12h 长的随机数
1665547733397 enter: crypto_rand.Read() 4000134990 c c
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  00 00 00 00 00 00 00 00 00 00 00 00              ............
1665547733398 leave: crypto_rand.Read() 4000134990 c c 0 0
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  38 93 5b 95 96 d0 05 a2 ea 76 c3 04              8.[......v..
# 获取时间戳，需要注意时间戳的格式为：https://cs.opensource.google/go/go/+/master:src/time/time.go;drc=1b316e3571190964d960c6a7af3e17e887c70d45;l=129
1665547733398 enter: time.Now()
1665547733398 leave: time.Now() c0c9ad5557c2c67d 3b3ec52cd 770be38c80
# 前 12B 为随机数，后面 4B 是与时间戳异或经过某种规则异或得到，得到的是 elns
1665547733398 enter: encoding_base64._ptr_Encoding.EncodeToString() 4000132000 400009e210 10 18
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  38 93 5b 95 96 d0 05 a2 ea 76 c3 04 5b d5 66 40  8.[......v..[.f@
1665547733398 leave: encoding_base64._ptr_Encoding.EncodeToString() 400009e240 18
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  4f 4a 4e 62 6c 5a 62 51 42 61 4c 71 64 73 4d 45  OJNblZbQBaLqdsME
00000010  57 39 56 6d 51 41 3d 3d                          W9VmQA==
# 加密的内容是 body，通过 IDA 静态分析可知 key 是固定的，iv 后 12B 也是随机数，前 4B 未知
1665547733399 enter: zeus_mobile_security.AesCbcEncryptWithBase64() 40005ac090 2a 30 4000051b30 10 10 40001349a0 10 10
0x4000051b30 0x4000051b30
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  73 65 73 73 69 6f 6e 69 64 3d 44 46 63 75 73 55  sessionid=DFcusU
00000010  68 70 52 4d 58 77 44 30 38 44 51 67 6e 55 4b 58  hpRMXwD08DQgnUKX
00000020  4f 6f 39 32 53 32 6c 58 49 41                    Oo92S2lXIA
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  15 c3 c2 db 93 fb bf a9 0c ff bc 11 8e 9a 53 bd  ..............S.
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  89 30 fe d1 38 93 5b 95 96 d0 05 a2 ea 76 c3 04  .0..8.[......v..
1665547733400 enter: encoding_base64._ptr_Encoding.EncodeToString() 4000132000 40005ac0c0 30 30
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  1a 64 d4 c2 23 2e dd 23 ec 4e fc d9 4c 52 23 30  .d..#..#.N..LR#0
00000010  ad e9 63 de af ba 90 4f 9b c9 01 26 7d e3 ff 17  ..c....O...&}...
00000020  ba c9 0e ef 19 32 dc 61 8a 75 5e ab 0c f0 18 73  .....2.a.u^....s
1665547733400 leave: encoding_base64._ptr_Encoding.EncodeToString() 40000d03c0 40
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  47 6d 54 55 77 69 4d 75 33 53 50 73 54 76 7a 5a  GmTUwiMu3SPsTvzZ
00000010  54 46 49 6a 4d 4b 33 70 59 39 36 76 75 70 42 50  TFIjMK3pY96vupBP
00000020  6d 38 6b 42 4a 6e 33 6a 2f 78 65 36 79 51 37 76  m8kBJn3j/xe6yQ7v
00000030  47 54 4c 63 59 59 70 31 58 71 73 4d 38 42 68 7a  GTLcYYp1XqsM8Bhz
1665547733401 leave: zeus_mobile_security.AesCbcEncryptWithBase64() 40000d03c0 40
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  47 6d 54 55 77 69 4d 75 33 53 50 73 54 76 7a 5a  GmTUwiMu3SPsTvzZ
00000010  54 46 49 6a 4d 4b 33 70 59 39 36 76 75 70 42 50  TFIjMK3pY96vupBP
00000020  6d 38 6b 42 4a 6e 33 6a 2f 78 65 36 79 51 37 76  m8kBJn3j/xe6yQ7v
00000030  47 54 4c 63 59 59 70 31 58 71 73 4d 38 42 68 7a  GTLcYYp1XqsM8Bhz
# SFib() 函数的参数都是固定的，返回值也是固定的
1665547733401 enter: zeus_mobile_security.SFib() 770bde0ce0 30 30 770b6d8cff 10
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  29 dc 8f 42 6d 3f b8 f5 b2 74 71 48 47 8a 8b f0  )..Bm?...tqHG...
00000010  ef 6c d3 5d c9 b7 8b 77 00 13 23 3c 19 99 ac 36  .l.]...w..#<...6
00000020  ad dd 6f b2 3b 61 62 ea 7b b4 d2 1b ad 75 a4 bd  ..o.;ab.{....u..
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  68 3b 31 54 4b 21 4d 26 41 2c 49 36 7a 32 73 37  h;1TK!M&A,I6z2s7
1665547733402 leave: zeus_mobile_security.SFib() 4000510360 20
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  61 65 38 62 36 64 34 62 65 66 61 39 37 65 61 34  ae8b6d4befa97ea4
00000010  66 33 34 62 31 62 34 32 32 34 32 31 35 34 66 31  f34b1b42242154f1
# 计算一个拼接字符串的 sha512
# 这个字符串是：SFib() + elauth + elect + elns + elver + urlpath + enc_body
1665547733403 enter: crypto_sha512.Sum512() 400024e9c0 b6 c0
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  61 65 38 62 36 64 34 62 65 66 61 39 37 65 61 34  ae8b6d4befa97ea4
00000010  66 33 34 62 31 62 34 32 32 34 32 31 35 34 66 31  f34b1b42242154f1
00000020  44 46 63 75 73 55 68 70 52 4d 58 77 44 30 38 44  DFcusUhpRMXwD08D
00000030  51 67 6e 55 4b 58 4f 6f 39 32 53 32 6c 58 49 41  QgnUKXOo92S2lXIA
00000040  31 4f 4a 4e 62 6c 5a 62 51 42 61 4c 71 64 73 4d  1OJNblZbQBaLqdsM
00000050  45 57 39 56 6d 51 41 3d 3d 45 31 2e 30 2f 61 70  EW9VmQA==E1.0/ap
00000060  70 2f 67 65 6e 65 72 61 6c 2f 73 70 6c 61 73 68  p/general/splash
00000070  53 63 72 65 65 6e 47 6d 54 55 77 69 4d 75 33 53  ScreenGmTUwiMu3S
00000080  50 73 54 76 7a 5a 54 46 49 6a 4d 4b 33 70 59 39  PsTvzZTFIjMK3pY9
00000090  36 76 75 70 42 50 6d 38 6b 42 4a 6e 33 6a 2f 78  6vupBPm8kBJn3j/x
000000a0  65 36 79 51 37 76 47 54 4c 63 59 59 70 31 58 71  e6yQ7vGTLcYYp1Xq
000000b0  73 4d 38 42 68 7a                                sM8Bhz
1665547733404 leave: crypto_sha512.Sum512()
            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  3e de 54 9c 52 ab 03 3a c4 2f 48 a0 c5 26 34 d5  >.T.R..:./H..&4.
00000010  ca 64 1d 22 f0 09 f9 aa 63 2c 14 45 fc e0 90 09  .d."....c,.E....
00000020  4a 72 3f 5c 46 a5 18 f0 57 c6 d6 a3 2c b0 5c cf  Jr?\F...W...,.\.
00000030  b2 5c 78 48 b1 7a 97 d7 2a 19 e8 b6 61 dd da aa  .\xH.z..*...a...
# 截取 18B 编为 base64
1665547733404 enter: encoding_base64._ptr_Encoding.EncodeToString() 4000132000 4000051ba0 18 40
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  3e de 54 9c 52 ab 03 3a c4 2f 48 a0 c5 26 34 d5  >.T.R..:./H..&4.
00000010  ca 64 1d 22 f0 09 f9 aa                          .d."....
1665547733404 leave: encoding_base64._ptr_Encoding.EncodeToString() 40005103a0 20
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  50 74 35 55 6e 46 4b 72 41 7a 72 45 4c 30 69 67  Pt5UnFKrAzrEL0ig
00000010  78 53 59 30 31 63 70 6b 48 53 4c 77 43 66 6d 71  xSY01cpkHSLwCfmq

```

到目前为止，整个流程已经大致清晰了，不清楚的是 iv 和 elns 的异或是怎么做的，这个方法有很多。

如上所述，go 中奇怪的 `Time` 格式，直接静态看起来不直观，最直接的可以用 `Stalker` 追踪寄存器的变化：

```
5bfa38                ldr x0, [sp, #8] ;          x0 = 0x770d5bbc80 --> 0xc0c9aef2cb280ea1
5bfa3c                ldr x1, [sp, #0x10] ;          x1 = 0xc0c9aef2cb280ea1 --> 0x73f6ac82b
5bfa40                ldr x2, [sp, #0x18] ;          x2 = 0x73f6ac82b --> 0x770d5bbc80 
5bfa44                str x0, [sp, #0x1c0] ;          str = 0xc0c9aef2cb280ea1
5bfa48                str x1, [sp, #0x1c8] ;          str = 0x73f6ac82b
5bfa4c                str x2, [sp, #0x1d0] ;          str = 0x770d5bbc80 
5bfa50                nop ; 
5bfa54                ldr x0, [sp, #0x1c0] ; 
5bfa58                tbz x0, #0x3f, #0x770d237a70 ; 
5bfa5c                ubfx x0, x0, #0x1e, #0x21 ;          x0 = 0xc0c9aef2cb280ea1 --> 0x10326bbcb
5bfa60                mov x27, #0x7f80 ;          x27 = 0x9fe07780 --> 0x7f80
5bfa64                movk x27, #0xd7b1, lsl #16 ;          x27 = 0x7f80 --> 0xd7b17f80
5bfa68                movk x27, #0xd, lsl #32 ;          x27 = 0xd7b17f80 --> 0xdd7b17f80
5bfa6c                add x1, x0, x27 ;          x1 = 0x73f6ac82b --> 0xedad83b4b
5bfa70                mov x27, #0xf700 ;          x27 = 0xdd7b17f80 --> 0xf700
5bfa74                movk x27, #0x7791, lsl #16 ;          x27 = 0xf700 --> 0x7791f700
5bfa78                movk x27, #0xe, lsl #32 ;          x27 = 0x7791f700 --> 0xe7791f700
5bfa7c                sub x0, x1, x27 ;          x0 = 0x10326bbcb --> 0x6346444b
5bfa80                rev w1, w0 ;          x1 = 0xedad83b4b --> 0x4b444663
5bfa84                str w1, [sp, #0x74] ;          str = 0x4b444663
5bfa88                ldr x1, [sp, #0x1b8] ;          x1 = 0x4b444663 --> 0x400011c770
5bfa8c                ldrb w2, [x1] ;          x2 = 0x770d5bbc80 --> 0x0 
5bfa90                ubfx x3, x0, #0x18, #8 ;          x3 = 0x10326bbcb --> 0x63
5bfa94                eor x2, x3, x2 ;          x2 = 0x0 --> 0x63
5bfa98                strb w2, [sp, #0x70] ; 
5bfa9c                ldrb w2, [x1, #1] ;          x2 = 0x63 --> 0x98
5bfaa0                ubfx x3, x0, #0x10, #0x10 ;          x3 = 0x63 --> 0x6346
5bfaa4                eor x2, x3, x2 ;          x2 = 0x98 --> 0x63de
5bfaa8                strb w2, [sp, #0x71] ; 
5bfaac                ldrb w2, [x1, #2] ;          x2 = 0x63de --> 0x8d
5bfab0                ubfx x3, x0, #8, #0x18 ;          x3 = 0x6346 --> 0x634644
5bfab4                eor x2, x3, x2 ;          x2 = 0x8d --> 0x6346c9
5bfab8                strb w2, [sp, #0x72] ; 
5bfabc                ldrb w2, [x1, #3] ;          x2 = 0x6346c9 --> 0x5d
5bfac0                eor x0, x2, x0 ;          x0 = 0x6346444b --> 0x63464416
5bfac4                strb w0, [sp, #0x73] ; 
5bfac8                adrp x0, #0x770d286000 ;          x0 = 0x63464416 --> 0x770d286000  
5bfacc                add x0, x0, #0x460 ;          x0 = 0x770d286000 --> 0x770d286460 
5bfad0                str x0, [sp, #8] ;          str = 0x770d286460

```

关键是 `5bfa7c`h 这个偏移的指令得到了 `0x6346444b`，这是 Unix 时间戳，后面又和随机数的前 4B 异或。iv 的异或是类似的，是随机数的后 4B 与时间戳作异或。

### 流程

到此为止，整个流程已经很清晰，整理一下：

*   elauth 是登录时返回的 token
    
*   elect 是 1
    
*   elver 在 body 存在时为 E1.0 否则 M1.0
    
*   elns 的前 12h 位是 iv 用到的随机数，后 4h 是与 Unix 时间戳异或而得
    
*   body 的本质是一个 AES/CBC + base64
    
    1.  需要加密的内容是请求体: urlencode 格式的各种参数
    2.  key 是固定的，由三个操作数异或而得，Sign() 函数内有这个循环：
        *   op1 = EF 6C D3 5D C9 B7 8B 77 00 13 23 3C 19 99 AC 36
        *   op2 = AD DD 6F B2 3B 61 62 EA 7B B4 D2 1B AD 75 A4 BD
        *   op3 = "Wr~4a-V4wXM6:v[6"
    3.  iv 是变化的，其中后 12h 为随机数，前 4h 也是异或而成的，异或值为 Unix 时间戳
*   Sign 算法是对一个字符串求 sha512 然后截取 18h 做 base64 ，这个字符串的结构为：
    
    1.  ae8b6d4befa97ea4f34b1b42242154f1
        *   这个字符串是由 zeus_mobile_security.SFib() 根据两个固定的参数构造出的
    2.  elauth + elect + elns + elver
    3.  请求路径，如：/app/video/stopped/vids
    4.  body

代码实现
----

流程有了，代码就好写了

```
import time
import string
import random
import base64

from Crypto.Cipher import AES
from Crypto.Hash import SHA512
from Crypto.Util.Padding import pad

def getSign(req_path: str, elauth: str, req_body: str, elect: int, z: bool) -> dict:
    if elect not in [1, 2, 3]: 
        return {}

    elver = "E1.0" if req_body else "M1.0"
    # 生成随机数和时间戳
    rand = b''.join(random.choice(string.printable).encode('utf8') for _ in range(12))
    now = int(time.time()).to_bytes(4, 'big')
    # 计算 elns
    tail = b"".join((i^j).to_bytes(1, 'big') for i,j in zip(now, rand))
    elns = base64.b64encode(rand + tail).decode('utf8')
    # 计算 iv，并加密 body
    if req_body:
        key = bytes.fromhex("15 c3 c2 db 93 fb bf a9 0c ff bc 11 8e 9a 53 bd")
        iv_head = b"".join((i^j).to_bytes(1, 'big') for i,j in zip(now, rand[8:]))
        cipher = AES.new(key, AES.MODE_CBC, iv=iv_head+rand)
        ct = cipher.encrypt(pad(req_body.encode('utf8'), AES.block_size))
        body = base64.b64encode(ct).decode('utf8')
    else:
        body = ""
    # 做 sha512 得到 sign
    head = "ae8b6d4befa97ea4f34b1b42242154f1"
    concat_string = head + elauth + elect + elns + elver + req_path + body
    sha_head = SHA512.new(concat_string.encode('utf8')).digest()[:0x18]
    elsign = base64.b64encode(sha_head).decode('utf8')

    return {
        "contentType": "text/plain",
        "elauth": elauth,
        "elect": elect,
        "elns": elns,
        "elsign": elsign,
        "elver": elver,
        "body": body
    }

```

小结
--

把过程写下来的时间快赶上逆向的时间了，主要想给逆 go 时做个记录：context 还是非常好用的，可以更好的帮助我们更好理解函数功能。

文中废话有点多，感谢各位大佬看到这！

参考
--

1.  [https://chai2010.cn/advanced-go-programming-book/ch2-cgo/ch2-05-internal.html](https://chai2010.cn/advanced-go-programming-book/ch2-cgo/ch2-05-internal.html)
2.  [https://www.52pojie.cn/thread-1691013-1-1.html](https://www.52pojie.cn/thread-1691013-1-1.html)
3.  [https://cs.opensource.google/go/go/+/master:src/runtime](https://cs.opensource.google/go/go/+/master:src/runtime)
4.  [https://bbs.pediy.com/thread-273501.htm](https://bbs.pediy.com/thread-273501.htm)![](https://avatar.52pojie.cn/images/noavatar_middle.gif)torrent 要怎么才能这么厉害