> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1749758-1-1.html)

> [*] 浏览器抓包分析 1. m3u8 包分析还是经典的 m3u8 文件格式，对比上个版本没有太大出入，key_url 一样没有用上。

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)ling02123 _ 本帖最后由 ling02123 于 2023-2-23 14:47 编辑_  

> demo 'aHR0cHM6Ly9zdHVkZW50LWFwaS5peWluY2Fpc2hpamlhby5jb20vZXAvcGMvbG9naW4='

*   **浏览器抓包分析**  
    

  
**1.  m3u8 包分析**  
![](https://attach.52pojie.cn/forum/202302/23/144004ymetex8z5m5adzmm.png)

**Fohdu15KRc89-y6_JZBArAJiRo_8.png** _(112.57 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM3M3wwNjFhZDlhM3wxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:40 上传

  
**还是经典的 m3u8 文件格式，对比上个版本没有太大出入，key_url 一样没有用上。既然没有抓到 key 链接，直接转去分析 ts 解密逻辑。**  
****2. ts 包分析****随便断住一个 ts 包，跟进解密函数。****  
****如图****  
![](https://attach.52pojie.cn/forum/202302/23/144022dmbjbvm0oi2mk00y.png)

**FknWS2XOrtWomTwenRjqcTKSs9pM.png** _(116.34 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM3NHw3M2JmNDljMHwxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:40 上传

  
**参数说明 ： **j  :  ts 字节流** **      q  :  30 位数组** **$  :   16 位数组**毫无疑问这里就是解密操作了，接下重点分析 参数如何生成以及后续的解密操作。**  
**  
**

*   ****参数生成 & 解密操作****  
    

  
****准备工作：****  
![](https://attach.52pojie.cn/forum/202302/23/144055d0vn8y8vk4atlyoc.png)

**Fg5Qc7ZDaGNFNGXHBeyR3uGptm3Q.png** _(100.77 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM3NXwzZGYzM2Y0YnwxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:40 上传

  
****如图 ，网页上浪浪已经提前把 console hook 了，我们需要在网页加载的时候保存一份 console 方便后续分析。****  

*   ****初分析：****  
    

  
![](https://attach.52pojie.cn/forum/202302/23/144124yb2ba8hw2hhg2hdw.png)

**FsJ1v6C1DqaaMC6uB9a2tqzRFvDC.png** _(120.08 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM3NnxkZmE2MDA4YnwxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:41 上传

  
![](https://attach.52pojie.cn/forum/202302/23/144133zqwjqb5gd6sgk89s.png)

**Fh5ug_WrPNRy29Pnc7K_09GsuU8E.png** _(62 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM3N3xhNjViZjVlY3wxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:41 上传

  
**仔细观察这个解密函数，你会毫无头绪，往下拉来观察一下这个函数如何初始化，发现是一个类似于 JSVMP 的东西，先传入大个很长的字符串还有一些乱七八糟的东西。多次调试发现里面的明文很少很少，挑出几个有明文的地方插桩分析。**  
![](https://attach.52pojie.cn/forum/202302/23/144153c01jcna2608asajl.png)

**FmgqEIPcl1vORhsyKfiK2dD9orPm.png** _(54.37 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM3OHw5ODY1NTM0ZHwxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:41 上传

  
**例如这个地方的 oe.apply() 其实是函数调用，能看到一些明文，可以在这里下断，可以多下几个日志断点，信息越多越容易分析。

*   **下面结合输出的日志来辅佐分析：**  
    

  
![](https://attach.52pojie.cn/forum/202302/23/144214k3xnv3wnckwnvxst.png)

**FnL2frwuDKykAZsFQCSed0U6oyIh.png** _(155.89 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM3OXw3YzkxMzY2MHwxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:42 上传

**OK，日志嘎嘎输出。****

*   ****输出图 1 分析：****  
    

****第一步：**开局我们拿到了一个 30 位的数组，在第一步的时候他把数组拿到了。**然后进行了一下   fromCharCode()  经过尝试** **'1-yuYeqlPlHMU85XD+UOdM+QHmG8/P'   字符串就是 30 位数组 ，**顺便一提 yuYeqlPlHMU85XD+UOdM+QHmG8/P 是由 '**play_licenses**' 接口返回  
数组如下，下面还会一直分析数组：[49, 45, 121, 117, 89, 101, 113, 108, 80, 108, 72, 77, 85, 56, 53, 88, 68, 43, 85, 79, 100, 77, 43, 81, 72, 109, 71, 56, 47, 80]  
**第二 三步：**关键词 test, 那必须是正则的 test，可以看到他把一开始加上的 '**1-'** 去掉了，变成'yuYeqlPlHMU85XD+UOdM+QHmG8/P'  就是接口直接返回的明文  
![](https://static.52pojie.cn/static/image/hrline/line1.png)  
**![](https://attach.52pojie.cn/forum/202302/23/144231lsvvyqqpiq5qrqpb.png)

**FsP3maa045-qOBFgNqILSupm4dbl.png** _(74.05 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM4MHw2OGM2ZjdmNnwxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:42 上传

**  
**

*   ********输出图 2 分析：********  
    

**很显然 他直接 atob 这个明文，atob 就是浏览器原生的 base64 解密函数。base64 解密完的就是下面第二步的字符串。'&#202;&#230;\x1E&#170;S&#229;\x1C&#197;<&#229;p&#254;P&#231;Lù\x01&#230;\x1B&#207;&#207;'这个样子很奇怪，把它弄成 Uint8Array 试试。[202, 230, 30, 170, 83, 229, 28, 197, 60, 229, 112, 254, 80, 231, 76, 249, 1, 230, 27, 207, 207]  len:21**  
![](https://attach.52pojie.cn/forum/202302/23/144559mdffd1s0r51m4d0p.png)

**Fr1HO6qDnL36vEgCflMLi-8Pm2oc.png** _(27.8 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM4MnwwOGMwMDcyM3wxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:45 上传

**  
**

*   ********输出图 3 分析：********  
    

**![](https://static.52pojie.cn/static/image/hrline/line1.png)  
这里他又把 21 位变成了 18 位的数组 ，嗨哟你干嘛真的是，对比一下 18 位跟 21 位的区别 [230, 30, 170, 83, 229, 28, 197, 60, 229, 112, 254, 80, 231, 76, 249, 1, 230, 27]  len:18 发现他把前面一个和最后俩个分割掉了。  
![](https://static.52pojie.cn/static/image/hrline/line1.png)  
**![](https://attach.52pojie.cn/forum/202302/23/144541zx0hos5htxu1oxut.png)

**FiV4G87wkO1qIhEoCOPVLGBvPOy_.png** _(154.98 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM4MXxmZDI1MWZjOXwxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:45 上传

**  
**

*   ********输出图 4 分析：********  
    

**[**230**, 30, 170, 83, 229, 28, 197, 60, 229, 112, 254, 80, 231, 76, 249, 1, 230, 27]  <- 18 位数组**从 18 位数组钟取出第一位，****然后神秘数字 250 出现了 跟 230 干了一些事情，然后 28 出现了；****28 跟多次出现的 21 干了点事情，49 就出现了；****然后 String.fromCharCode(49) 就变成了 '1';**

* * *

okok, 我按照这个分析一波，大胆推测一下 **250 ^ 230 = 28&#65279;** 28 + 21 = 49230 -> '1'同理 能得到第二个神秘数字就是 **85****  
![](https://attach.52pojie.cn/forum/202302/23/144627c6qqpim5sssph8p6.png)

**FixA_iedeelKv_fPQ1BuRt_VupYV.png** _(121.11 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM4M3w3YjZkOGY0YnwxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:46 上传

  
****

*   ****输出图 5 分析：****  
    

[230, 30, 170, 83, **229**, 28, **197**, 60, 229, 112, 254, 80, 231, 76, 249, 1, 230, 27]  <- 18 位数组图 5 为 处理数组第 7 位的流程。第一步神秘数字变成了 299，你会发现卧槽，原来神秘数字竟在我身边 ，**299 是 197 在数组的前俩位。**okok, 上面分析的  **229** ^ **197** = 32 依旧成立，但 32 + 21 != 55 你干嘛~，后面咋对不上了很明显  32 + 21 **+ 2** = 55 接下来分析一下这个 '2' 怎么来  
**

*   **最后分析：**  
    

**我们可以根据图 5 分析中前面正确的步骤解密出来的值跟最后浏览器生成的值对比得出中间差值**我们按照思路重新分析一下：[230, 30, 170, 83, 229, 28, 197, 60, 229, 112, 254, 80, 231, 76, 249, 1, 230, 27]  <- 18 位数组先把 18 位填充两位神秘数字成 [250,85,230, 30, 170, 83, 229, 28, 197, 60, 229, 112, 254, 80, 231, 76, 249, 1, 230, 27] <- 20 位然后[Python] _纯文本查看_ _复制代码_

```
qq = [250,85,230, 30, 170, 83, 229, 28, 197, 60, 229, 112, 254, 80, 231, 76, 249, 1, 230, 27]
pp = []
for index,i in enumerate(qq):
    if index>=2:
        a = (qq[index - 2] ^ i) +21
        pp.append(a)
print(pp)  #此时的18位数组
```

[49, 96, 97,  98,  100, 100, 53, 53, 53, 97, 48, 53, 46, 49,  51,  98, 52, 47] <- 此时的 18 位数组 [**49,** 97, 98, 100, 101, 102, **55**, 56, 54, 99, 50, 56, 48, 52, 54, 102, 53, 27]   <- 浏览器生成最后的 18 位  
      [97, 98, 100, 101, 102, 55, 56, 54, 99, 50, 56, 48, 52, 54, 102, 53]       <- AES 解密 16 位数组                                   abdef786c28046f5               <- 16 位数组 String.fromCharCode() 成的字符串你会发现它把 18 位数组最后还原成了 16 位。其实是还原后再次分割了。把前面一位和后面一位分割掉了。  
**所以说有用的只有中间的 16 位**对比中间 16 位可以得到 中间的 差值       [1, 2, 1, 2, 2, 3, 1, 2, 2, 3, 2, 3, 3, 4, 1] 对这些数字敏感 可以发现：  
[Python] _纯文本查看_ _复制代码_

```
str(bin(index)).count('1')
```

可以生成这些数字。分析到现在，完整算法意境分析完了，经过多次测试这个算法能解密其他 **play_licenses，这次分析也算成功了。****  
![](https://attach.52pojie.cn/forum/202302/23/144655fzmufz5ommfffgms.png)

**Fkw_s8OmMOe5ok7ordvM5dQ9NA0f.png** _(70.38 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU5MjM4NHwxMTg2MjNjNnwxNjc3MjAzNTQ3fDkzMDQwMnwxNzQ5NzU4&nothumb=yes)

2023-2-23 14:46 上传

  
****还原算法后也和网页一样能正常解密视频。****

*   ****总结****  
    

**    这次浪浪更新的难度适中，很适合分析，过程很详细，也可以动手分析一下，细节也讲解到位了，把各个分析部分组装一下就能用。**![](https://avatar.52pojie.cn/images/noavatar_middle.gif)bigqyng 跟大佬学习一下 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) yangfan233 向大佬学习![](https://static.52pojie.cn/static/image/smiley/default/40.gif) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) szxizhijiang 文章 很精彩，但是太复杂了，正常人取 key 就是一步到位 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) LuckyClover 看到这样的解密就很舒服，分给你 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) zfyln 没看明白，怎么直接解密![](https://static.52pojie.cn/static/image/smiley/default/27.gif) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) hijack911 不错很喜欢