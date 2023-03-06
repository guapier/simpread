> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/For_John/article/details/129304270)

### 文章目录

*   [微信小程序逆向分析](#_1)
*   *   [行为监控](#_13)
    *   [分析文件特征](#_29)
    *   [wxapkg 解密思路](#wxapkg_37)
    *   [重新监控](#_46)
    *   [解密分析](#_54)
    *   *   [尾部部分解密](#_97)
        *   *   [解密例程分析](#_99)
            *   [找到 xorKey](#xorKey_119)
        *   [首部 1024 字节解密](#1024_157)
        *   *   [解密例程分析](#_159)
            *   [AES_Key 生成例程分析](#AES_Key_216)
            *   [分组模式以及 iv](#iv_332)

微信小程序逆向分析
=========

WeChatAppEx.exe 版本：2.0.6609.4

以融智云考学生端为例。

网上已经有关于微信小程序解密的非常优秀的文章，本着学习的目的便不参考相关内容。

笔者水平实在有限，如发现纰漏，还请读者不吝赐教。  
如涉及侵权，请联系作者处理。

行为监控
----

工具：[火绒剑](https://so.csdn.net/so/search?q=%E7%81%AB%E7%BB%92%E5%89%91&spm=1001.2101.3001.7020)

首先看看打开一个小程序微信做了点什么，对微信进行火绒行为监控。因为小程序最初在 PC 端运行，必然会相关文件在客户机上释放，所以我们主要关注微信的[文件读写](https://so.csdn.net/so/search?q=%E6%96%87%E4%BB%B6%E8%AF%BB%E5%86%99&spm=1001.2101.3001.7020)行为。

![](https://img-blog.csdnimg.cn/img_convert/6a68d5386c722db3c63b98fa2eafd201.png)

注意到这里有类似文件释放的行为，在监控上访其实同样有读取此文件夹的行为，根据经验这其实就是一个简单的读取相关目录，发现没有相关程序逻辑文件后，主动请求服务器下载相关文件。

那我们的关注点来到`\__APP__.wxapkg`, 根据前人的经验，这就是小程序的主要逻辑所在的地方。

其中可以监控到很多关于调用堆栈的信息，不过这些堆栈附近大概是文件释放相关逻辑，这并不是我们关注的重点。

![](https://img-blog.csdnimg.cn/img_convert/e18d8c63ed56a628d57e849d532231a9.png)

分析文件特征
------

我们用 010Editor 打开文件，任意一个二进制编辑器都可以。

![](https://img-blog.csdnimg.cn/img_convert/822e3d4eecc8263a67d8fedf2bae021c.png)

可以看到明显程序逻辑被加密那么我们关注点就来到了这个文件的解密操作。

wxapkg 解密思路
-----------

那么有两种思路

1.  很经典的思路，既然文件被加密，那么微信客户端装载此程序的时候必然要进行解密，那么必然要进行打开文件的操作，我们对 CreateFile 下断点，应该是可以找到打开文件的操作，记录打开的句柄，同时微信也需要对文件进行读取的操作。我们在 ReadFile 下条件断点，当传入的的句柄是我们获得的小程序文件的句柄时断下，调用堆栈附近应该就有相关解密的操作，这种操作可行性很大，但是相对比较麻烦，微信打开的文件很多，在 CreateFile 下断点可能比较麻烦。
2.  如果微信使用的是比较常规的加密 算法，那么可以通过 IDA 的插件`Findcrypt`看看有没有比较明显的特征。

** 需要注意到的是，在 WeChat 中并没用打开这个 wxapkg 的相关操作。那么斗胆猜测可能是微信重新启动一个加载器对 wxapkg 进行装载。** 那么我们进行火绒剑全局监控，看看有没对 wxapkg 进行读取的操作。

重新监控
----

![](https://img-blog.csdnimg.cn/img_convert/ddc3aa85b0a53542714386d251825f6e.png)

可以观察到名为`WeChatAppEx.exe`对文件做了两次读取的操作，根据经验我们关注第二次读取，双击 File_read 操作，查看调用栈

![](https://img-blog.csdnimg.cn/img_convert/d673955f241cec2ff2ae93b85b0bc8d4.png)

解密分析
----

跟随堆栈，我们在 IDA 和 X64dbg 中分别定位此位置。RVA:0x678353d

![](https://img-blog.csdnimg.cn/img_convert/1fb0a0832749d4d848a47b7b58bb66cc.png)

可以看到此位置对文件进行了读取，我们在 dbg 中看看有没有文件的相关数据。附加到 WeChatAppEx.exe，在对应位置下断点，并运行一个小程序。

![](https://img-blog.csdnimg.cn/img_convert/66834e2680a01eac25d533a13c12b93e.png)

轻而易举的断下，并且我们观察到文件大小非常接近打开的加密文件大小，并且在堆栈中出现了相关文件信息。

我们步过查看`readBuf`内的数据。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

可以看到`WeChatAppEx.exe`读取了加密后的文件。那么我们不难理解`WeChatAppEx.exe`类似一个加载器，在运行时对文件进行解密装载。利用程序要对这一片内存进行读写的特征，我们对这篇内存区域下硬件访问断点（Xdbg 的内存断点是针对内存页的，可能会断在奇怪的地方，可能是笔者不太会使用这种特性）

![](https://img-blog.csdnimg.cn/img_convert/0845222c0c1ead237698e9bc6f4bdd3c.png)

可以看到在 RVA：2784F7A 处断下，比较奇怪的是此处对文件的前 8 字节赋值为 0，不太能理解，索性我们对没有修改的内存再次下硬件访问断点（不要忘记卸载之前的断点）。

![](https://img-blog.csdnimg.cn/img_convert/b44ee7ab72eaec024526b217b6769d37.png)

再次断下时对之后的 8 个字节赋值为 - 1，同样比较难解，我们再次对剩余区域下硬件断点。

![](https://img-blog.csdnimg.cn/img_convert/99cf387728f0d4e48bc5a90df858addc.png)

再次在 RVA：676C91E 处断下，可以看到这里有对字符串操作的相关指令，根据`movsb`指令的功能

```
指令:                                          
     MOVSB, MOVSW, MOVSD
                        
     描述:
     移动字符串数据，复制由ESI寄存器寻址的内存地址处的数据至EDI寻址的内存地址处。
```

拓展的，我们分别观察 RSI 与 RDI 指向的内存区域

![](https://img-blog.csdnimg.cn/img_convert/585034ab26fee88a9d5e60902c902e0f.png)

### 尾部部分解密

#### 解密例程分析

可以看到 RSI 指向的内存中有非完整的加密文件的十六进制形式（前 8 字节，即`V1MMWXß`被忽略），有意思的是，复制操作的指针并没有指向文件头部，而是指向了距离除去`V1MMWXß`之后的 1024 字节之后，这里有分块加密的特征，但是为什么程序没有在解密前 1024 字节处停下呢，可能是笔者疏忽或者程序先解密尾部部分在解密首部，既然已经到这里，我们不妨顺藤摸瓜。

![](https://img-blog.csdnimg.cn/img_convert/228456cc92698a4fb7af40491a393833.png)

可以看到正在向 RDI 中赋值数据，步过至字符串操作完成，我们再对这片内存区域下硬件访问断点。

之后再次在 RVA：676C91E 出断下，类似的，字符串操作完之后，我们再次对 RDI 下硬件访问断点。

再次运行之后再 RVA：302772F 处断下

![](https://img-blog.csdnimg.cn/img_convert/75c88c134cd0f1d600cb20c0392d4e6a.png)

观察到我们下断点的内存区域已经出现了`../image`字样，这无疑是非常让人兴奋的，可能这就是文件解密的位置。事实确实如此，我们取消硬件断点，在上一步断下出的循环中循环几次简单分析就可以确定，这确实是一个解密点，而且是非常简单的异或解密。事实上，这一部分解密过程与微信图片解密相同。

![](https://img-blog.csdnimg.cn/img_convert/69f9b29182a4fd49f060ef1fae9131a8.png)

现在，我们找到了密文的 1024 字节之后部分的解密例程。我们默认忽略前 8 字节的处理，读者可自行分析，装载器并没有对前 8 字节做过多处理，在解密过程中只是简单的忽略。

#### 找到 xorKey

那么我们 rbp 所对应的秘钥`0x34`从何而来呢，追踪异或 Key 使我们接下来要做的事情。

有请尊敬的 IDA 先生，我们在 IDA 转到 RVA：302772A，即异或解密位置追踪 Key 从何而来。

![](https://img-blog.csdnimg.cn/img_convert/e0e539e9b9b0c9ed34eff440db1a8728.png)

根据分析，a3 即是 xorKey，至于`a3*0x1010101010…` 目的是用`int8`类型的值 a3 填满 rbp 至 8 个字节，进行 8 个字节分组异或。我们对 a3 进行简单的重命名—>xorKey_

![](https://img-blog.csdnimg.cn/img_convert/234d91cc35304a7f1bda2520dca6bc04.png)

可以看到 xorKey 作为参数被传入（为提高辨识度，笔者对此函数进行简单的重命名）

![](https://img-blog.csdnimg.cn/img_convert/6ddc2a70b2dca5e1efc28edb0dce2678.png)

查看对此函数的引用，有两个运行时调用，还有一个直接调用，处于防止`跟飞`情况，我们对此函数头下断来找到调用点。

![](https://img-blog.csdnimg.cn/img_convert/c873d9a4cf1ab9b634a89860a1af3386.png)

文件再次断下，并在堆栈中回溯，我们来到调用点 RVA：302759B

在 IDA 中我们了解到 xorKey 在异或解密函数`ContextDecode`中作为第三个参数传递，且类型为`int8`，根据`fastcall`调用约定，我们关注寄存器 R8:000000000014EE34, 最后一个字节，即 0x34。他是怎么来的呢，我们向上分析

![](https://img-blog.csdnimg.cn/img_convert/b662c67a8142c4709982c49094d8f159.png)

VA：00007FF632C97589 处对 r8 最后一个字节进行了赋值，我们看`[rax+rcx-0x2]`是什么。

![](https://img-blog.csdnimg.cn/img_convert/9419204ec7865c0eafd18b11455e2daf.png)

可以看到 rcx 指向一个字符串，实际上这是微信小程序的 ID，即 AppID，进行简单的分析，rax 是 AppID 的长度，而减去 0x2 后，`[rax+rcx-0x2]`指向的是 AppID 字符串的倒数第二个字符，将字符所对应的 ascii 码赋值给 r8，这样，我们 xorKey 就拿到了。我们在 everything 找搜索对应 AppId：wxf2a0156c0235fc4c

![](https://img-blog.csdnimg.cn/img_convert/9af3e63ecb78f3b65b814ccbcfdb7940.png)

可以证实上面的说法。至此，尾部部分解密告一段落。

### 首部 1024 字节解密

#### 解密例程分析

我们再次来到 RVA：0x678353d 上方的 ReadFile 处下断，因为根据之前分析，此处有全部密文出现, 同样我们忽视前 8 字节，对剩余内存内容下硬件访问断点，尝试寻找前 1024 字节解密位置。

![](https://img-blog.csdnimg.cn/img_convert/fd3176092b0436d4dcbf00d290584e36.png)

重新在此处断下（RVA：676C91E），同之前分析，补过字符串操作之灵后，我们跟随目的地 (RDI) 内存区域，对其下硬件访问断点。

![](https://img-blog.csdnimg.cn/img_convert/1094b7cccdf456d3b396e9bc64e3d8ae.png)

再次在此处断下（RVA:676C91E) 断下，重复上述步骤，在目的地地址下断。

运行之后再 RVA：40DFE 处断下

![](https://img-blog.csdnimg.cn/img_convert/60374257ed04247960ab5449e9ec590e.png)

这里就有比较令人兴奋的字段：`the iv:16bytes`，部分加密需要一个向量，我们不妨猜测，这里就是加密函数，我们在 IDA 中来到对应位置。

![](https://img-blog.csdnimg.cn/img_convert/1aeab8d3b280bc1b2e83f9582db1b331.png)

之前笔者已经对一些变量进行分析并且重命名，所以看起来似乎一目了然，这些变量的命名，我们之后逐步分析但不是现在的关注的重点。

引起我们注意的是类似的汇编指令`aesdeclast xmm2, xmm1`, 注意到字样 “aes” 就可以怀疑密文首部采用的是 aes 加密，事实上确实采用的是这用加密，从学习的角度，我们假设并不知情相关特征。

既然到了这一步，不妨运行看看相关内存区域有没有明文信息。

![](https://img-blog.csdnimg.cn/img_convert/d8f2fc6b0ea50d3bdba5ad307555bd28.png)

运行若干步之后，我们在 RSI 所指内存区域中发现明文特征，这与之前分析尾部解密中得出的明文十分类似，至此可以确定，这一部分逻辑即是对前 1024 字节进行解密的逻辑。

那么我们下一步要解决的问题是：” 这是什么加密 “，以便我们能找出秘钥，自行写出解密脚本。

我们百度`aesdeclast xmm2, xmm1`，看看能不能收获一些有用的信息。

下面是来自于互联网的一些资料：

AESDECLAST — Perform Last Round of an AES Decryption Flow

<table><thead><tr><th>Opcode/Instruction</th><th>Op/En</th><th>64/32-bit Mode</th><th>CPUID Feature Flag</th><th>Description</th></tr></thead><tbody><tr><td>66 0F 38 DF /r AESDECLAST xmm1, xmm2/m128</td><td>RM</td><td>V/V</td><td>AES</td><td>Perform the last round of an AES decryption flow, using the Equivalent Inverse Cipher, operating on a 128-bit data (state) from xmm1 with a 128-bit round key from xmm2/m128.</td></tr><tr><td>VEX.128.66.0F38.WIG DF /r VAESDECLAST xmm1, xmm2, xmm3/m128</td><td>RVM</td><td>V/V</td><td>Both AES and AVX flags</td><td>Perform the last round of an AES decryption flow, using the Equivalent Inverse Cipher, operating on a 128-bit data (state) from xmm2 with a 128-bit round key from xmm3/m128; store the result in xmm1.</td></tr></tbody></table>

Description [¶](https://www.laruence.com/x86/AESDECLAST.html#description)

**This instruction performs the last round of the AES decryption flow using the Equivalent Inverse Cipher, with the round key from the second source operand, operating on a 128-bit data (state) from the first source operand,** and store the result in the destination operand.

128-bit Legacy SSE version: The first source operand and the destination operand are the same and must be an XMM register. The second source operand can be an XMM register or a 128-bit memory location. Bits (MAXVL-1:128) of the corresponding YMM destination register remain unchanged.

VEX.128 encoded version: The first source operand and the destination operand are XMM registers. The second source operand can be an XMM register or a 128-bit memory location. Bits (MAXVL-1:128) of the destination YMM register are zeroed.

请注意描述中的加粗部分，其大概意思是`aesdeclast xmm2, xmm1`执行的是反向解密的最后一轮解密过程，xmm1 是 round Key（拓展秘钥，aes 将用户设置的秘钥进行拓展以便于运算），而 xmm2 即是最后一轮解密的数据。

现在我们可以确定这一部分加密使用的是 AES 加密，我们正在分析的是其对应的解密部分。根据 AES 加密的对应的解密过程，最后一轮解密使用的 round Key 正是用户设定的秘钥，关于 AES 使用类似指令的介绍以及加解密的细节问题，笔者收集到一篇优质文章：

> [Intel AES-NI 使用入门 - 被遺忘的海灘 | Nagi’s Blog (x-nagi.com)](https://x-nagi.com/post/aesni.html)

#### AES_Key 生成例程分析

我们再次来到 RVA：40DFE 处

结合引用文章的介绍，我们大概可以得出：![](https://img-blog.csdnimg.cn/img_convert/76658333dcce5549c36d679a0e6f155b.png)

那么我们接下来的关注点放到了拓展秘钥缓冲区，我们跟随秘钥缓冲区的生成会进入到秘钥拓展例程，在那里，我们大概率可以拿到 Key，我们在 IDA 中追踪秘钥缓冲区

![](https://img-blog.csdnimg.cn/img_convert/04817aa0f46cfec8c06566785821bf74.png)

v33 对应的是 Rcx，而拓展秘钥缓冲区即 keyArry 来自于函数外。我们对此函数头下断进行栈回溯。

函数头：

![](https://img-blog.csdnimg.cn/img_convert/da0b2ae50b1668e07c44cc6ee6100044.png)

再次断下后（前几次断下并不能得到我们要的调用栈，因为相关参数中找不到密文缓冲区等特征），根据`fastcall`约定，秘钥应该是 r9 所指缓冲区，实际上，秘钥缓冲区最后十六个字节作为原始 Key 的一部分（16 字节），即未拓展的 Key，秘钥拓展例程通过原始 Key 进行秘钥拓展，不过不注意这个细节也没有关系，这在之后的分析中将会体现。

![](https://img-blog.csdnimg.cn/img_convert/54e17f47dbe95894cd2db0de18836a19.png)

我们回溯到 RVA：2811EB5

![](https://img-blog.csdnimg.cn/img_convert/03f08dcb0a3a2d3313d1aeded93b1c42.png)

在此函数中，keyArry 已经生成，那么我们继续栈回溯，来到 VA：00007FF6444C1137

![](https://img-blog.csdnimg.cn/img_convert/0fc2619e671a76d1c2c88c3b4611b22b.png)

[rcx+0x10] 即是 keyArry，同样分析此函数，发现 keyArry 同样作为参数传入此函数，那么我们继续进行调用栈回溯。

回退到第二次调用栈，我们来到 VA：00007FF6444C1398, 如下图

![](https://img-blog.csdnimg.cn/img_convert/752fdfec5c19e1f95f5d2e7310fcfb76.png)

同样的，[rcx+0x10] 即是 keyArry，同样是作为参数传递进来的，再次进行堆栈回溯，来到 RVA：00000000027F15AB，如下图，这里再次进行了简单的转发，再次进行栈回溯。

![](https://img-blog.csdnimg.cn/img_convert/6d024161e025ce701c7b53cd2da53a92.png)

来到 RVA：285B20C，如下图![](https://img-blog.csdnimg.cn/img_convert/71462340ecd6f1a54528b86b6390301c.png)

我们所说的 keyArry 是 aes 实例化的一个对象，里面存储有拓展秘钥。再此函数进行简单分析后发现，key 同样来自函数外通过参数传递进来。

![](https://img-blog.csdnimg.cn/img_convert/04f17395123064159b38c1bc9a14d3e3.png)

继续堆栈回溯 -_-||，来到 RVA：000000000285AE26

![](https://img-blog.csdnimg.cn/img_convert/885ab5c408a74b6a9cabdc4b30561f0f.png)

继续回溯，来到 RVA：000000000285AED8，同样是一个简单的转发，继续回溯，来到 RVA：0000000003026BF2

如下图

![](https://img-blog.csdnimg.cn/img_convert/1336c55b97f3c0b4ec73e26d89d00f79.png)

如图，[[v18]+0x8] 指向秘钥，终于要计算秘钥了 -_-||，本函数上方有对 v18 的相关操作，如下图

![](https://img-blog.csdnimg.cn/img_convert/00142c3b5ac272d45040b9d95690e608.png)

其实看到‘salt’这几个字符，对秘钥拓展熟悉的朋友应该能马上反应过来这里应该就是拓展秘钥的地方了。我们进到函数里

如图

![](https://img-blog.csdnimg.cn/img_convert/d021f7e7cbff57a852573d7be3827dd9.png)

这就非常明确了，我们百度 Pbkdf2

> PBKDF 的全称是 Password-Based Key Derivation Function，简单的说，PBKDF 就是一个密码衍生的工具。既然有 PBKDF2 那么就肯定有 PBKDF1，那么他们两个的区别是什么呢？PBKDF2 是 PKCS 系列的标准之一，具体来说他是 PKCS#5 的 2.0 版本，同样被作为 RFC 2898 发布。它是 PBKDF1 的替代品，为什么会替代 PBKDF1 呢？那是因为 PBKDF1 只能生成 160bits 长度的 key，在计算机性能快速发展的今天，已经不能够满足我们的加密需要了。所以被 PBKDF2 替换了。在 2017 年发布的 RFC 8018（PKCS #5 v2.1）中，是建议是用 PBKDF2 作为密码 hashing 的标准。PBKDF2 和 PBKDF1 主要是用来防止密码暴力破解的，所以在设计中加入了对算力的自动调整，从而抵御暴力破解的可能性。
> 
> PBKDF2 的工作流程
> 
> PBKDF2 实际上就是将伪散列函数 PRF（pseudorandom function）应用到输入的密码、salt 中，生成一个散列值，然后将这个散列值作为一个加密 key，应用到后续的加密过程中，以此类推，将这个过程重复很多次，从而增加了密码破解的难度，这个过程也被称为是密码加强。

稍微阅读以上引用内容 ，对比此函数参数不难得出：

1.  盐值：saltiest
2.  秘钥：小程序 ID
3.  秘钥拓展算法：PBKDF2
4.  迭代次数：1000
5.  秘钥长度：256 位

既然秘钥长度是 256 位，那么可以推测出加密算法是 AES_256。

注意到的是，它的伪散列算法是可以替换的，那么我们下一步要找出的是它使用的是哪种散列算法

这里笔者简单使用 js 选几种常见的散列算法试一试

```
const crypto = require('crypto');

let appid = "wxf2a0156c0235fc4c";
    crypto.pbkdf2(appid,"saltiest",1000,32,'sha1',(err, derivedKey) => 
    { 
      if (err) throw err; 
      console.log("The password is ",derivedKey.toString('hex'));
    });
```

![](https://img-blog.csdnimg.cn/img_convert/6145779c9a03c7f1ae35a71d1779c087.png)

对比拓展秘钥函数运行之后的返回值 [[rax]+0x8] 中的值：

![](https://img-blog.csdnimg.cn/img_convert/bdeff868c6f60b72c9e9092d54e791e6.png)

可以验证散列算法是‘sha1’，对应我们在分析过程中的拓展秘钥的最后字节部分。

```
00001D5A002CD188     3D9D6AB5E94DDDE8 èÝMéµj.= 
00001D5A002CD190     71122C7B6FFE09D6 Ö.þo{,.q 
00001D5A002CD198     C3981F7A8828924E N.(.z..Ã 
00001D5A002CD1A0     BD14C8D6E69FEA98 .ê.æÖÈ.½ 
00001D5A002CD1A8     3336977C3609F094 .ð.6|.63 
00001D5A002CD1B0     1969D6DC01305252 RR0.ÜÖi.
```

至此，我们找到了秘钥的生成算法。

#### 分组模式以及 iv

分组模式以及 iv 的寻找相对简单，只要对 AES 加密流程以及几种加密模式的区别熟悉就可以在加密函数（RVA：0000000000040C30）中分析出加密模式以及向量。这里笔者不再赘述。

经过简单分析，总结之前分析成果，有如下清单：

```
除去文件头8个字节，剩余1024解密算法：
    秘钥算法：PBKDF2
    盐值：saltiest
    秘钥：小程序ID，wxf2a0156c0235fc4c
    摘要算法：sha1
    秘钥长度：32字节

    解密算法：aes-256-cbc模式
    初始化向量iv：74 68 65 20 69 76 3A 20 31 36 20 62 79 74 65 73 对应字符串：“the iv: 16 bytes” -_-||
1024字节之后的数据处理方式：
    解密方式：异或解密
    异或Key：微信appid字符串的第二个字符对应的ASCII码形式。
```

拿到解密出的文件后可以用相应的解包脚本进行解压，网上不乏解压脚本，遗憾的是笔者并没有找到能够彻底解压并且还原出微信开发者工具能够识别的对应各式的文件（微信开发者工具对 js 等的样式做了一层封装，想要能够调试源代码需要将解包后的文件还原成其能够识别的格式，网上确实有相关脚本，但大多比较老，微信开发工具对样式进行了更新，格式化出现了一些问题，笔者水平有限，就不去修复。）

下面给出不成熟的 C++ 解密脚本

```
#include "PKCS7.h"
#include <openssl/evp.h>
#include<openssl/aes.h>
#include <openssl/sha.h>
#include <openssl/crypto.h>
#include <iostream>
#include <string>
#include <fstream>
#include <filesystem>

using namespace std;

unsigned char iv[] = { 0x74,0x68,0x65,0x20,0x69,0x76,0x3A,0x20,0x31,0x36,0x20,0x62,0x79,0x74,0x65,0x73 };//iv
unsigned char recursive_keys[32] = { 0 };//计算AES秘钥
const unsigned char  salt[] = "saltiest";//盐值

int main()
{
	string app_id("wxf2a0156c0235fc4c");
	/*cout << "Plz enter the AppID:" << endl;
	cin >> app_id;*/

	//计算递归秘钥
	PKCS5_PBKDF2_HMAC_SHA1(app_id.c_str(),app_id.length(),salt,strlen((const char*)salt),1000,32, recursive_keys);
	//cout << recursive_keys << endl;
	
	string file_name("__APP__.wxapkg");
	/*cout << "Plz enter the name of the file you want to decrypt :" << endl;
	cin >> file_name;*/
	//读取文件
	int file_size = std::filesystem::file_size(file_name);
	char* file_buf = new char[file_size] {0};
	fstream fp(file_name.c_str(),std::ios::in|ios::binary);
	if (!fp.is_open())
	{
		cout << "Sorry,please check that you entered the correct file name" << endl;
		delete[] file_buf;
		file_buf = nullptr;
		return 0;
	}

	fp.read(file_buf, file_size);
	fp.close();

	//AES解密前1024字节内容（忽略文件头6个字节）
	AES_KEY aes_key;
	AES_set_decrypt_key((const unsigned char*)recursive_keys, 256, &aes_key);
	AES_cbc_encrypt((const unsigned char*)file_buf+0x6, (unsigned char*)file_buf+0x6, 1024, &aes_key, iv, AES_DECRYPT);
	PKCS7_unPadding* padding_result = removePadding(file_buf + 0x6, 1024);//解除填充

	size_t diff = 1024 - padding_result->dataLengthWithoutPadding;//得到解除填充后与保持填充时明文的差值

	//1024字节之后的密文解密
	//找到异或Key
	char xor_key = app_id.c_str()[app_id.length() - 2];

	for (int i = 0; i < file_size - 0x6 - 0x400-diff; ++i)
	{
		file_buf[0x406 + i - diff] = file_buf[0x406 + i] ^ xor_key;
	}

	//将解密后的数据写入
	fstream fp_out(file_name+"_plaintext",ios::out | ios::binary);
	if (!fp_out.is_open())
	{
		cout << "File create failed." << endl;
		fp_out.close();
		delete[] file_buf;
		file_buf = nullptr;
        freeUnPaddingResult(padding_result);
		return 0;
	}

	fp_out.write(file_buf+0x6,file_size-0x6-diff);
	fp_out.close();
	cout << "The file decryption is successful." << endl;

	delete[] file_buf;
	file_buf = nullptr;
	freeUnPaddingResult(padding_result);
	return 0;
}
```

WeChatAppEx_2.0.6609.4 下载：  
链接：https://pan.baidu.com/s/1N1f6DwOiIoElt9m1aN4uSw?pwd=vp03  
提取码：vp03  
供学习使用