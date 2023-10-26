> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1849061-1-1.html)

> [md]# search_wechat_key## 本程序用来搜索微信数据库的密钥 key 信息 ** 不要用于非法用途！** ** 不要用于非法用途！** ** 不要用于非法用途！** ## .......

![](https://avatar.52pojie.cn/data/avatar/000/43/82/64_avatar_middle.jpg)sunbeat _ 本帖最后由 sunbeat 于 2023-10-26 16:46 编辑_  

search_wechat_key
=================

本程序用来搜索微信数据库的密钥 key 信息
----------------------

**不要用于非法用途！**  
**不要用于非法用途！**  
**不要用于非法用途！**  

原理
--

一般情况下，key 要在运行的微信进程内存中拿到，内存偏移在每个版本都不一样，大部分工具是对每个版本维护一套偏移，但是当出现新版本的时候都要重新找偏移。  
其实，除了这个方法外，还有一个更通用的方法就是内存暴力搜索找到能用于解密的密钥位置，当然如果对进程全部内存扫一遍肯定不行，所以项目里用下面这种方式缩小密钥内存范围加快扫描速度：

1.  微信登录设备类型基本只有 iphone、android，在内存中先搜到设备类型所在内存，key 就在它的前面，向前搜就行；  
2.  key 的内存地址和登录设备类型在大部分版本是 16 字节对齐的，但也有非 16 字节对齐，这里用的是每次向前 1 字节方式，避免遇到非对齐情况；  
    每次读 32 字节，然后判断读取的是不是微信 sqlite 的数据库密钥。   密钥找到后，可以为后续读取微信聊天记录做准备。   

已测试版本列表
-------

其它未测试版本不代表不能用，这个列表只是我本地有过的环境。

*   3.9.7.29 （64 位版）
*   3.9.7.28 （32 位版）

用法类似：
-----

```
C:\Users\Win10\Desktop\pc_wechat\dist>search_wecaht_key.exe  
2023-10-26 13:52:01,281 - search_wecaht_key.py- INFO - wechat version：3.9.7.28  
2023-10-26 13:52:01,583 - search_wecaht_key.py- INFO - db_file:C:\Users\Win10\Documents\WeChat Files\wxid_cpyn7pe119rxxx\Msg\Misc.db  
2023-10-26 13:52:01,699 - search_wecaht_key.py- INFO - phone_addr:713817B8  
2023-10-26 13:52:01,888 - search_wecaht_key.py- INFO - found key pointer addr:71381744, key_addr:7417E70  
2023-10-26 13:52:01,888 - search_wecaht_key.py- INFO - key:eb204727b80a44dea374ee92df27fb06ef7218098ae7473bafc695b9b5a9xxxx  
2023-10-26 13:52:01,888 - search_wecaht_key.py- INFO - eb204727b80a44dea374ee92df27fb06ef7218098ae7473bafc695b9b5a9xxxx  

```

如何手动寻找偏移
--------

使用 CheatEngine 在内存中搜索找到字符串 android（我的手机是安卓，如果是 iphone 的，用 iphone 搜），必须是在 `WeChatWin.dll` 内存范围内

下载地址: [https://pan.baidu.com/s/1R3U3sleKvUUxg50ZdbWEkw?pwd=dbeq](https://pan.baidu.com/s/1R3U3sleKvUUxg50ZdbWEkw?pwd=dbeq) 提取码: dbeq

源码：[https://github.com/sunhanaix/search_wechat_key](https://github.com/sunhanaix/search_wechat_key)

[CE 浏览 adroid 附近内存信息. jpg](forum.php?mod=attachment&aid=MjY1MTg0NHxmYzI1MmI3Y3wxNjk4MzExMTA3fDkzMDQwMnwxODQ5MDYx&nothumb=yes) _(127.87 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjY1MTg0NHxmYzI1MmI3Y3wxNjk4MzExMTA3fDkzMDQwMnwxODQ5MDYx&nothumb=yes)

2023-10-26 16:38 上传 [![](https://static.52pojie.cn/static/image/common/rleft.gif)](javascript:;) [![](https://static.52pojie.cn/static/image/common/rright.gif)](javascript:;)

![](https://attach.52pojie.cn/forum/202310/26/163820f6u8uz8mohohkvoo.jpg) [CE 找 android 关键字. png](forum.php?mod=attachment&aid=MjY1MTg0NXxlMjZkMmZkNXwxNjk4MzExMTA3fDkzMDQwMnwxODQ5MDYx&nothumb=yes) _(79.31 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjY1MTg0NXxlMjZkMmZkNXwxNjk4MzExMTA3fDkzMDQwMnwxODQ5MDYx&nothumb=yes)

2023-10-26 16:38 上传 [![](https://static.52pojie.cn/static/image/common/rleft.gif)](javascript:;) [![](https://static.52pojie.cn/static/image/common/rright.gif)](javascript:;)

![](https://attach.52pojie.cn/forum/202310/26/163822y1z566r7rn5nq1nr.png) [CE 找到 key.jpg](forum.php?mod=attachment&aid=MjY1MTg0NnwzN2NjMDgwN3wxNjk4MzExMTA3fDkzMDQwMnwxODQ5MDYx&nothumb=yes) _(211.66 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjY1MTg0NnwzN2NjMDgwN3wxNjk4MzExMTA3fDkzMDQwMnwxODQ5MDYx&nothumb=yes)

2023-10-26 16:38 上传 [![](https://static.52pojie.cn/static/image/common/rleft.gif)](javascript:;) [![](https://static.52pojie.cn/static/image/common/rright.gif)](javascript:;)

![](https://attach.52pojie.cn/forum/202310/26/163825c701xb90lzdp94r7.jpg) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif)Elaineliu 这个可以支持一下