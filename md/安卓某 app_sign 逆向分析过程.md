> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1715751-1-1.html)

> [md]## 开篇 > [安卓协议逆向之 frida hook 百例](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1711668)> 在看到大佬的......

![](https://avatar.52pojie.cn/data/avatar/000/26/85/18_avatar_middle.jpg)a976606645

开篇
--

> [安卓协议逆向之 frida hook 百例](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1711668)  
> 在看到大佬的这篇文章后，想动手跟着分析一波，在下载目标 APP 时没有注意版本号的问题，下载了新版本的，记录下分析的过程。

准备工具
----

```
1. root设备
2. BlackDex
3. frida
4. IDA
5. jadx
```

目标
--

目标是搞到 sign 的算法 看 sign 的长相跟长度 先盲猜一波 MD5  
![](https://pic.imgdb.cn/item/6379083f16f2c2beb11c4e7c.jpg)

开工
--

直接用 BlackDex 打开目标 APP 获取到 dex  
拖入 jadx 开始分析  
搜索接口名字 查找调用

![](https://pic.imgdb.cn/item/6379062516f2c2beb1189b2b.jpg)

发现有两处调用的地方，双击进去探探情况

![](https://pic.imgdb.cn/item/637906f316f2c2beb11a2bbd.jpg)

继续搜索一下定义的名称  查找调用

![](https://pic.imgdb.cn/item/6379088916f2c2beb11cc9e6.jpg)

继续跟进 找到了关键点

![](https://pic.imgdb.cn/item/63790aea16f2c2beb1203897.jpg)

![](https://pic.imgdb.cn/item/63790b7c16f2c2beb120f438.jpg)

![](https://pic.imgdb.cn/item/63790c3e16f2c2beb12266bb.jpg)

![](https://pic.imgdb.cn/item/63790cde16f2c2beb123b178.jpg)

分析 so
-----

解压 apk  找到 lib 下的 `libblackBox.so`  在 IDA 中打开  
在 IDA 方法窗口中搜索`getInterfaceSign`  无果  
推测方法为动态注册  到`JNI_OnLoad`入口找找看

![](https://pic.imgdb.cn/item/63790eab16f2c2beb1275449.jpg)

按下`F5`查看伪代码  
看到`VX+XXX`这种格式的伪代码 鼠标点击定位到`VX`的位置 按`Y`键修复指针

![](https://pic.imgdb.cn/item/63790f1016f2c2beb127c457.jpg)

![](https://pic.imgdb.cn/item/637915bd16f2c2beb12f43f5.jpg)

进入查看  
找到了动态注册的方法 分析上面 JAVA 层调用的`getInterfaceSign`方法 只有一个参数 最终确定 so 层的函数为`sub_49268`

![](https://pic.imgdb.cn/item/6379167c16f2c2beb1300ded.jpg)

继续跟进

![](https://pic.imgdb.cn/item/6379172c16f2c2beb130fb1e.jpg)

![](https://pic.imgdb.cn/item/6379187b16f2c2beb132b1b0.jpg)

```
鼠标点击定位到`a1`的位置 按`Y`键修复指针
一番寻找之后 发现所有的return里面 都有`a1`的参与
那现在目的就明确了 我们需要找到对`a1`有所改动的地方
但是没有发现直接对`a1`赋值的的语句
这说明`a1`是在某一个调用函数里面改变的值
接下来的重点就是调查里面的sub函数们
先着重调查了`a1`为参数的函数  没有找到有用的信息
```

![](https://pic.imgdb.cn/item/63791cd616f2c2beb139ef9c.jpg)

在排查过程中  发现了这个函数 明文上没有`a1`参数的参与  
但是把鼠标移动到`v34`上 弹出的提示信息中 有`a1`的存在 跟进一下

![](https://pic.imgdb.cn/item/63791f6816f2c2beb13c6896.png)

![](https://pic.imgdb.cn/item/6379225a16f2c2beb13f1ef6.png)

![](https://pic.imgdb.cn/item/6379230116f2c2beb13fbae8.jpg)

终于 在`sub_F39A8`发现了关键点 一个小写转大写的函数过程

![](https://pic.imgdb.cn/item/637925de16f2c2beb1422780.jpg)

这段代码中的`sub_4BBE4`就很可疑了，继续跟进

```
__int64 __fastcall sub_F39A8(__int64 a1, __int64 a2, const char *a3)
{
  __int64 v4; // x19

  if ( !a1 && !a3 )
    return 0xFFFFFFFFLL;
  v4 = 0LL;
  if ( sub_4BBE4() )
    return 0xFFFFFFFFLL;
  while ( (int)strlen(a3) > (int)v4 )
  {
    a3[v4] = toupper((unsigned __int8)a3[v4]);
    ++v4;
  }
  return 0LL;
}
```

在这里 发现了一个 MD5 关键函数  
分析到这里 我们该验证一下了 已知该函数的偏移地址是：`0x4BBE4`

![](https://pic.imgdb.cn/item/6379286716f2c2beb14498df.jpg)

frida hook so
-------------

> 我这里使用的是安卓真机 安装了去 root 特征的系统  
> 我不确定目标 APP 有无 root 检测  
> 如测试时启动 APP 有闪退的情况 还需要自行过一下 root 检测

接下来编写 frida 的 hook 脚本  
将 IDA 分析的函数偏移地址填写进脚本

```
Java.perform(function () {
    function get_func_addr(module, offset) {
        var base_addr = Module.findBaseAddress(module);
        console.log("base_addr: " + base_addr);
        console.log(hexdump(ptr(base_addr), {
            length: 16, header: true, ansi: true
        }))
        var func_addr = base_addr.add(offset);
        if (Process.arch == 'arm') return func_addr.add(1);  //如果是32位地址+1
        else return func_addr;
    }

    var func_addr = get_func_addr('libblackBox.so', 0x4BBE4);//参数：so名称  偏移地址
    console.log('func_addr: ' + func_addr);
    console.log(hexdump(ptr(func_addr), {
        length: 16, header: true, ansi: true
    }))

    Interceptor.attach(ptr(func_addr), {
        onEnter: function (args) {//产生调用时hook输入的参数
            console.log("onEnter");
            console.log(args[0].readCString())
        }, onLeave: function (retval) {
            console.log(retval)
        }
    });
});
```

编写好脚本后 使用`adb shell`命令 启动手机上的`frida-server`  注入脚本

```
frida -U -f com.cxxx.xxxxx --no-pause -l hook3.js
```

注入后出现问题了  app 直接闪退

![](https://pic.imgdb.cn/item/6379311116f2c2beb14e3ed0.jpg)

说明 app 识别到了 frida 的特征 ~自杀了属于是~ 我们需要过 frida 检测  
在这里使用了大佬编译的[去特征版本的 strongR-frida](https://github.com/hzzheyang/strongR-frida-android/releases)  
`adb push`到手机后 启动 frida 服务 重新注入 没有发生闪退情况  
并且 可以看到`sub_4BBE4()`的参数被成功打印出来了

![](https://pic.imgdb.cn/item/6379367516f2c2beb152f488.jpg)

在 app 里面的账号登录页面  请求一下 login 接口试试情况

  
果然能看到明文信息  
![](https://attach.52pojie.cn/forum/202211/20/041044vkdu5nmwdz6ww5d7.png)

**微信图片_20221120041005.png** _(239.4 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU3MDk5NnxmZjdiOTgzMXwxNjY5ODc1MDE3fDkzMDQwMnwxNzE1NzUx&nothumb=yes)

2022-11-20 04:10 上传

  
将明文复制出来 MD5 加密一下 看看结果  
![](https://attach.52pojie.cn/forum/202211/20/041443x7jaffyy7sj7d55d.png)

**微信图片_20221120041352.png** _(62.87 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU3MDk5OHwyZjhmNDc2MXwxNjY5ODc1MDE3fDkzMDQwMnwxNzE1NzUx&nothumb=yes)

2022-11-20 04:14 上传

  
验证成功！！  

后记
--

这是我第一次尝试分析 so 整个过程收获了许多  
希望对你也有所帮助

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)JackSon001 so 分析这块讲的有点快奥，根据返回长度能大概猜出大概的 hash 算法，然后通过 hash 查找插件找一下可疑函数，然后再 hook，有可能是 hash 函数变种等等 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) liusan1990 看来还是得要有面向对象的基础编程知识才能看到，学习中。 希望能够尽快达到楼主的水平。![](https://avatar.52pojie.cn/images/noavatar_middle.gif)constwm 到哪都是一推大佬 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) xmnh 路过学习 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) lbg2222000 谢谢分享 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) maroo 学习了，逆向最烦就是 so 了 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) lyghost 能不能分析分析那种子进程附加无法 frida 的![](https://static.52pojie.cn/static/image/smiley/default/lol.gif)![](https://avatar.52pojie.cn/data/avatar/001/10/94/58_avatar_middle.jpg)正己 给个精华，期待后续佳作，早日消除违规![](https://static.52pojie.cn/static/image/smiley/laohu/laohu33.gif) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) 大老牛逼 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) y2006y2006 大佬牛逼啊 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) william888 感谢楼主的分享