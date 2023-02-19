> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1747740-1-1.html)

> [md]## 在 app 限制代理和 vpn 抓包下的另类抓包解决方案在逆向的过程中，抓包是首要要解决的问题。

![](https://avatar.52pojie.cn/data/avatar/000/93/53/52_avatar_middle.jpg)xixicoco _ 本帖最后由 xixicoco 于 2023-2-18 18:45 编辑_  

在 app 限制代理和 vpn 抓包下的另类抓包解决方案
----------------------------

  
在逆向的过程中，抓包是首要要解决的问题。但是现在的 app 反制措施很多，其中限制代理和 vpn 是抓包是很普遍的。通常这样的解决方案是找到 app 相关的类，进行 hook 或者 pach，但是要找到这样的类或者方法是不容易的，并不适合新手操作。本文介绍一种较为容易的另类解决方案，可以较为全面的解决以上问题：  
硬件环境：

1：处于同一路由器下的同一 ip 段的手机和 pc；

2：手机需要 root 权限，笔者系统为 android8.1，以下或者以上的系统应该也可以（未经测试）；

3：adb usb 调试模式连接好手机；

**首先，下载附件的 clash.rar, 解压后可以看到如下的目录：**

![](https://attach.52pojie.cn/forum/202302/18/181245s0cjgxfzo6f5lzeg.png)

**image-20230218170401912.png** _(11.18 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ2N3w5NGZmNGJjOHwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:12 上传

1：用文本编辑工具打开 config.yaml 文件，笔者的 pc ip 地址为 172.17.145.1，红框内填写你的 pc ip 地址即可，别的地方无需更改，保存。

![](https://attach.52pojie.cn/forum/202302/18/181408on97ye86mbxmmn7o.png)

**image-20230218171900993.png** _(54.6 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ2OXw0ZTJmYjY5M3wxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:14 上传

2：运行 push.bat, 这一步会推送相关文件到手机的 data/local/tmp 目录，运行完毕后，任意键终止即可：

![](https://attach.52pojie.cn/forum/202302/18/181319kpcvxx9fzc8uep98.png)

**image-20230218170924184.png** _(21.72 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ2OHxjYmE5YmRjZHwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:13 上传

3：运行 run.bat, 这一步会加载 clash 程序和相关的 iptable 规则，正常运行的图示如下：

![](https://attach.52pojie.cn/forum/202302/18/181433h02zv88286obbe85.png)

**image-20230218171206864.png** _(38.67 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ3MHxlNzUxMjcyMnwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:14 上传

4：run.bat 后，这个运行窗口会不断的输出信息，在抓包的过程中都要始终保持此窗口。如结束抓包，那么右上角 X 结束程序，同时需运行 clear.bat 清除 iptale 规则，以免给程序运行造成网路的麻烦。

5：在浏览器测试：[http://172.17.145.2:9090，其中 172.17.145.2 为手机的 ip 地址](http://172.17.145.2:9090，其中172.17.145.2为手机的ip地址)

如反馈如下，则正常：

![](https://attach.52pojie.cn/forum/202302/18/181608srgpb01d5r80rqd1.png)

**image-20230218173020940.png** _(14.02 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ3MXwxZmVjOWZjYXwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:16 上传

**其次，打开一款 web 版 clash 控制台，这里用的是 [http://clash.razord.top/（但是这个网址不好使，要翻墙，求推荐好使的 web 控制台](http://clash.razord.top/（但是这个网址不好使，要翻墙，求推荐好使的web控制台)）：**

1：打开网址 [http://clash.razord.top / 后，会弹出](http://clash.razord.top/后，会弹出)：  
![](https://attach.52pojie.cn/forum/202302/18/181654ea5hop611mgphzpm.png)

**image-20230218172711285.png** _(41.93 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ3Mnw2Yzc1YzMzZnwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:16 上传

填写你的手机 ip 地址，端口 9090 ，点确定

2：到设置选项，设置 socks5 端口为 8889，http 端口为 8888，其他不变：

![](https://attach.52pojie.cn/forum/202302/18/181946obbonpbb71n54373.png)

**image-20230218173852192.png** _(50.01 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ3M3xlNGY3MTNkMHwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:19 上传

3：回到策略组这里，选择 proxy_socks5 或者 proxy_http：

![](https://attach.52pojie.cn/forum/202302/18/182013t367kr7f5q0q77ka.png)

**image-20230218174043637.png** _(29.12 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ3NHwwM2QyZjQ4ZnwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:20 上传

[b] 说明：[i] 如果选择的是 http 那么抓包工具 charles，fiddler 和 burp 都可以用。如果选择的 socks5，那么只有 charles 可以用（原因是只有 charles 直接支持 socks 代理）。这两者的区别在于 socks5 有更好的兼容性，有些 app 的包在 socke5 下才可以顺利抓取。当然如果你有好的 http2socks 的工具也可以尝试把 fd 和 burp 的端口转换位 socks，这两款软件个人认为看起来更直观，只是我目前并没有找到这样的工具，求推荐！

  
4：在 clarles 的 proxy settings，里设置如下：

![](https://attach.52pojie.cn/forum/202302/18/182046xn3nq96z9zam6byt.png)

**image-20230218175011373.png** _(30.61 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ3NnxjNTI4ZjBkYnwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:20 上传

在 clarles 的 ssl proxying settings，里设置如下, 其中 inclue 的 host 和 port 都填入 *，只有填写正确才能解密 ssl：

![](https://attach.52pojie.cn/forum/202302/18/182110je0vezvy0yhjzigf.png)

**image-20230218175124322.png** _(27.81 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ3N3w0ZTVmOGZmMHwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:21 上传

5：安装 charles，fd 和 burp 相应的证书。这步从略，网上有很多教程。值得注意的是在 android7 以上，还需要把证书移动到系统目录，推荐用 magisk 的 Move_Certificates 模块，将抓包软件的证书置于系统内让系统信任；

6：顺利抓包的 3 款软件：  
![](https://attach.52pojie.cn/forum/202302/18/182250iabyj9j5dmnz4z6j.png)

**image-20230218175405791.png** _(110.21 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ4MHxlNWY3OTMzZXwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:22 上传

![](https://attach.52pojie.cn/forum/202302/18/182136yjjuagjld63uczdl.png)

**image-20230218175439046.png** _(168.17 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ3OHw1OWU0OGUzZHwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:21 上传

  
![](https://attach.52pojie.cn/forum/202302/18/182208p1kkzpw298pvapf2.png)

**image-20230218175513483.png** _(95.45 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MTQ3OXwzODdjNjgxZHwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw&nothumb=yes)

2023-2-18 18:22 上传

  
fd 和 burp 的代理设置从略，只要设置为 8888 端口即可，很简单。  
  
  
7：只要 pc 和手机的 ip 和端口没有变化，那么第二大步的 web 端下次均无需设置，直接运行 run.bat 和抓包软件即可  
  

  
[b] 最后总结：这种方案是利用 clash 的 core 版本配合 iptable 规则将数据转发到抓包软件，以避免 app 对代理和 vpn 的检测，比较具有普遍性。  
[b] 值得注意的是，即使这样操作后可能某些软件依然有抓包的问题。那么基本上就涉及到 ssl pining 或者证书绑定等。  
[b] 但这个便利的框架已经搭建起来，对应解决相关为题即可 最后该项目教程主要来源于：

[[Clash 版] 安卓上基于透明代理抓包 - SeeFlowerX](https://blog.seeflower.dev/archives/210/#comment-57)，感谢脸哥

码字不易，请大家多给热心评分，多谢！！！

  
  
  
  

![](https://static.52pojie.cn/static/image/filetype/rar.gif)

[clash.part2.rar](forum.php?mod=attachment&aid=MjU5MTQ4OXxjMmZkMDgxNXwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw)

2023-2-18 18:32 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB  

2.18 MB, 下载次数: 91, 下载积分: 吾爱币 -1 CB

![](https://static.52pojie.cn/static/image/filetype/rar.gif)

[clash.part1.rar](forum.php?mod=attachment&aid=MjU5MTQ4OHw0ODE0OGVkZXwxNjc2ODA1NDc5fDkzMDQwMnwxNzQ3NzQw)

2023-2-18 18:32 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB  

2.5 MB, 下载次数: 94, 下载积分: 吾爱币 -1 CB![](https://avatar.52pojie.cn/data/avatar/000/93/53/52_avatar_middle.jpg)xixicoco

> [外酥内嫩 发表于 2023-2-19 00:24](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=45678830&ptid=1747740)  
> 他们到底是怎么发现抓包的

挂代理或者 vpn 都可以被 app 的代码检测到，就看愿不愿意这么搞了  
有些防控严密的，是层层防护一点空都不留  
![](https://avatar.52pojie.cn/images/noavatar_middle.gif)baopushouzhuo 感谢分享 学习了 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif)bestwars 学习收藏了 感谢分享 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif)DreamWave 感谢大佬的分享，学习 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) zjh889 这东西，太好了！![](https://avatar.52pojie.cn/data/avatar/001/18/56/24_avatar_middle.jpg)judgecx fiddler 和 winreshark 能抓到大部分的包 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) LZW1768857595 非常好的分析，感谢分享![](https://avatar.52pojie.cn/images/noavatar_middle.gif)外酥内嫩 他们到底是怎么发现抓包的 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) yinyiniao 6 楼正解！