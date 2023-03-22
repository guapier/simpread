> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/m0_37802824/article/details/126330801)

一、前言
====

###### 一般大多数网站、APP 最常用的是 http、https 协议，而某两款最火的短视频 dy（某音）、ks（某手）最新版使用的是 [quic](https://so.csdn.net/so/search?q=quic&spm=1001.2101.3001.7020) 协议（见附录 1），导致 fiddler 和 charles 无法直接抓到包（某音 app 13.5 版本以下可以直接抓到包）。

###### 网上有说用 fiddler + xposed + justTrustMe 能绕过某音的 sslpinning，呵呵，别傻了好不好，dy（某音）、ks（某手）用的是非系统的 sslpinning 技术，使用那个方案只能绕过 Java 层的 sslpinning，而 dy（某音）、ks（某手）的 sslpinning 是在 so 层，根本行不通的好吧！不要再误人子弟了！（此方案亲测无用）

###### 而我给大家带来的几种方案都是亲测可行的，主要思路是禁用或者绕过 quic 相关的 so 加载，迫使它走 https 协议。某音的相关 so 文件是 libsscronet.so ，接下来看方案准备前的环节。

二、方案
====

###### 方案前准备

_**将抓包工具证书转换并添加到 Android 系统证书**_。今天我们用的抓包工具是 charles、模拟器是夜神模拟器，首先需要做的是将 Charles 证书转换并添加到 Android 系统证书, 方便 HTTPS 抓包。（**Android 7.0 以上版本必须要把证书添加到系统信任区，否则无法抓取 https 请求**）。  
具体步骤教程看：  
[https://blog.csdn.net/m0_59683157/article/details/122812586](https://blog.csdn.net/m0_59683157/article/details/122812586)  
其中 openssl 需要单独下载安装，下载安装教程看： [https://blog.csdn.net/houjixin/article/details/25806151](https://blog.csdn.net/houjixin/article/details/25806151)

###### 方案（一）：直接删除 libsscronet.so。

1、在夜神模拟器上，直接安装某音高版本 apk：直接把 apk 拉进模拟器，即可安装  
2、安装完以后，使用 adb 命令进入到模拟器终端 shell 环境（没安装 adb 的，可以直接用模拟器的 adb 命令，进入到模拟器 bin 目录，即可使用 adb 命令，也可先加到 windows 环境变量，方便后续使用）：**adb shell**  
3、进入到 so 文件目录:  
**cd /data/data/com.ss.android.ugc.aweme/lib**  
4、最后删除 libsscronet.so：**rm libsscronet.so**  
![](https://img-blog.csdnimg.cn/a6d4f9fe09ea48b581e1af89e8a00898.png#pic_center)  
5、最后即可抓包成功

###### 方案（二）：安装 xposed 插件，绕过 quic 协议

1、下载安装 xposed。在模拟器中 xposed 下载安装教程：[https://baijiahao.baidu.com/s?id=1720356526386732365&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1720356526386732365&wfr=spider&for=pc)  
2、xposed 插件安装  
（1）下载安装 AndroidStudio，到官网下载安装即可  
（2）用 AndroidStudio 搭建 xposed 环境，demo 编写教程：[https://blog.csdn.net/qq_39551311/article/details/119826692](https://blog.csdn.net/qq_39551311/article/details/119826692)  
（3）编写 dy（某音）的 xposed 插件（在 hook 代码主体部分替换为如下代码）：

```
XposedHelpers.findAndHookMethod("org.chromium.CronetClient", lpparam.classLoader, "tryCreateCronetEngine", Context.class, boolean.class, boolean.class, boolean.class, boolean.class, String.class, Executor.class, boolean.class, new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                Util.xposedLog("CronetClient disable tryCreateCronetEngine");
                param.setResult(null);
            }
        });
```

（4）如果不想编写这个插件，qq 联系我发给你，**qq:1723332145**

###### 方案（三）：直接用 IDA 反编译 libsscronet.so 文件，将 ssl 验证的地方修改

![](https://img-blog.csdnimg.cn/fa7a36ffd30e4879a9b2573f5d7e41cb.jpeg#pic_center)

教程可参考这两篇：[https://bbs.pediy.com/thread-269028.htm](https://bbs.pediy.com/thread-269028.htm)、  
[https://blog.csdn.net/chl191623691/article/details/123668069](https://blog.csdn.net/chl191623691/article/details/123668069)

###### 方案（四）：直接用 Frida 来 hook libsscronet.so 文件，在验证 sslpinning 地方动态将值改为 0 即可

三、结语
====

上面第一、二种方案是最容易处理 quic 协议的方案，尤其推荐第二种，是比较通用的解决 quic 的方式，第三、四种方案涉及到反编译 so 文件，然后 hook 或者修改 so 文件绕过 sslpinning 的方案，比较麻烦，由于篇幅原因，只介绍大概流程，不详细介绍，如果有需要或者不明白的小伙伴可以单独私聊我，**qq:1723332145**  
**注：这里的逆向技术仅用于学习、研究，不可用于商用或者违法用途，另外本文章属于原创，原创不易，请各位转发的时候也请注明转发地址，否则一旦发现，后果自负，谢谢各位观看。**

四、附录
====

###### （一）Quic 协议

QUIC（Quick UDP Internet Connection）是谷歌制定的一种互联网[传输层协议](https://so.csdn.net/so/search?q=%E4%BC%A0%E8%BE%93%E5%B1%82%E5%8D%8F%E8%AE%AE&spm=1001.2101.3001.7020)，它基于 UDP 传输层协议，同时兼具 TCP、TLS、HTTP/2 等协议的可靠性与安全性，可以有效减少连接与传输延迟，更好地应对当前传输层与应用层的挑战。）

###### （二）某音 xposed 插件

获取插件资源，联系我 qq：1723332145