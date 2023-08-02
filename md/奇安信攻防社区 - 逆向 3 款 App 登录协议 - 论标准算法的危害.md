> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [forum.butian.net](https://forum.butian.net/share/2371)

> 奇安信攻防社区 - 逆向 3 款 App 登录协议 - 论标准算法的危害

本文首先，使用 hook 技术，对两款采用 Java 层标准算法加密的 App 进行 hook ，实现较快速的对登录协议进行逆向分析。然后对 C/C++ 实现标准算法的步骤和代码进行研究，实现如何快速的找出 So 层用的是哪种加密方法，进而完成逆向分析。

hook 实现 Java 层协议通杀
------------------

### fridaUiTools 工具的使用

fridaUiTools 是一个界面化整理脚本的工具，本文的工作主要用到的就是 fridaUiTools 的 java 加解密部分。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-1548b8ab3ceaec2443f8c6645d8a603303ed1e06.png)

使用方法也很简单，提前勾选 java 加解密的选项框，然后附加进程就可以了，在输出日志中可以看到加解密相关信息。

### 某电影网 App 登录协议分析

在 App 的登录页面，输入手机号为 15026818188 密码为 123456789，然后点击登录。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-7c632779a8a1a54ffc833f64b60ff443ac28d93a.png)

登录的包是 GET 请求方式，所以这里只对 GET 请求方式的参数进行了加密，抓到的包如下：

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-f7a7a39583f9b021cc6b4a1705ce15af842d3170.png)

request 的值是加密过的，值是

xWonulEWGmnCtt3k0KzqxitgeBVGJxo53IW/llbwCCWoMOY3bQumRpdhOGr8X3M/

复制值的一部分去日志里面搜索，发现是可以定位到的。

而且可以看到原文

username=15026818188&password=123456789&bcode=

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-97822920a86151a521fdbd1295c8ec18a169cf9a.png)

还没有确定加密方法，所以日志继续往上翻。找到了秘钥，iv，和模式填充。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-cdddeccf88447461e4bc070452657e586225ca8c.png)

将找到的秘钥，iv，模式填充和原文，放到在线的 3DES 加密网址中，进行加密。

加密后的结果与 hook 到的结果完全一致！

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-4c6a10e7c4bfd9b7353ed9497d910cad775fc028.png)

另外 Http 请求头还有其他加密的参数，完整的 Http 请求头，如下图：

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-7c1e0ffb76477bf27d2e0b09b90f7cd28e291e77.png)

其中 Did 的值是：59c56da5611453a7da7aa1e283420286

Key 的值是：67964b8b10b073088a933a92e9d879f4

这两个值都是经过加密的，都是需要破解的。仍然将两个值复制到日志中，进行搜索。

首先复制 Did 的值进行查找。

找出 Did 的值就是 010067027161451 进行 MD5 加密后的结果。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-b4c1161609fe87f874299a2ed959dd420d4b2835.png)

在复制 Key 的值进行查找。

找出 Did 的值就是 59c56da5611453a7da7aa1e283420286m1905_2014 进行 MD5 加密后的结果。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-0c9679c03c89ca47d8bb82a266713d0128bec5f5.png)

而 59c56da5611453a7da7aa1e283420286 就是 Did 的值。

所以最终得出结论：首先将 010067027161451 进行 MD5 加密得出的值给 Did ，然后 Did + m1905_2014 得到一个新的字符串，在进行 MD5 加密，得到的值给 Key。

### 某设计 App 登录协议分析

仍然在 App 的登录页面，输入手机号为 15026818188 密码为 123456789，然后点击登录。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-b9675e5ed3ca27f3d4bd99151748c726b4cb4eb7.png)

登录的包是 POST 请求方式，加密的数据放在了 key 的值里面。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-af1fbfc1e41e5a6bba3d76f796bd775c07166798.png)

key 的值是：

```
dTdlMmtGQjZIQk45NEN3dTIzamdVdFduWGFQM2M0WWJySm1QU3hJeUxEMStKWTV2UUdlMVhjT05K%250AZmdIeGZjOTZGSWYxU3NkUkltQQpPYTg0Z0Z0TkFqV0oyckxHOFppUGZGdnpRMzkrcjBuc0VIRVFm%250AcUk0QWtnNHdTWmR2anNyU1JpYTBpVXZ3aFVtSlZUb1FZQng4Tk5XCnFtQjB2UjZjcVE3ZXB5elRV%250AZlZhc2x1RTFsYjlENk1UREpkVG9qL3hZVGN5eENIWUtBejI1THI3Y0wzNXpVb3ZjelMybWxoUFBo%250AdVUKZzJPSStkY1Q3SkxEelNreE5VYlczTUg1b0ZwaVAvSytlUW05a1N1NFdLU0dKUWoxdWVVRHZX%250ANjI1a1lheEpRMktrdzFJbHJoOUJGQQp0MEcvQVFoQU5uWGxkU1FhUC9LK2VRbTlrU3U0V0tTR0pR%250AajF1YXFadjRiRWE5L1pxeEpWalVRSE0wUTA5WlJRaHhlRTNEVCtyS1VOClowSmRCQ2FzdHpvbnJO%250AK29wWjhmZE5tdVRRPT0KP2tleUlkPTE%253D%250A


```

仍然将 key 值复制一部分放到日志里面进行搜索，可这次，发现是搜索不到的！

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-e72cda38269eabebceeb6e14576a313398f83068.png)

对上面的数据进行观察，有好多 % 这样的数据，于是猜测上面的数据可能经过了 UrlEncode，尝试对其进行解码。

发现可成功解码，第一次解码的数据是：

dTdlMmtGQjZIQk45NEN3dTIzamdVdFduWGFQM2M0WWJySm1QU3hJeUxEMStKWTV2UUdlMVhjT05K%0AZmdIeGZjOTZGSWYxU3NkUkltQQpPYTg0Z0Z0TkFqV0oyckxHOFppUGZGdnpRMzkrcjBuc0VIRVFm%0AcUk0QWtnNHdTWmR2anNyU1JpYTBpVXZ3aFVtSlZUb1FZQng4Tk5XCnFtQjB2UjZjcVE3ZXB5elRV%0AZlZhc2x1RTFsYjlENk1UREpkVG9qL3hZVGN5eENIWUtBejI1THI3Y0wzNXpVb3ZjelMybWxoUFBo%0AdVUKZzJPSStkY1Q3SkxEelNreE5VYlczTUg1b0ZwaVAvSytlUW05a1N1NFdLU0dKUWoxdWVVRHZX%0ANjI1a1lheEpRMktrdzFJbHJoOUJGQQp0MEcvQVFoQU5uWGxkU1FhUC9LK2VRbTlrU3U0V0tTR0pR%0AajF1YXFadjRiRWE5L1pxeEpWalVRSE0wUTA5WlJRaHhlRTNEVCtyS1VOClowSmRCQ2FzdHpvbnJO%0AK29wWjhmZE5tdVRRPT0KP2tleUlkPTE%3D%0A

发现解码后的数据仍然存在好多 % 这样的数据，所以 key 里面的数据可能经历了 2 次 UrlEncode，再次尝试对解码后的数据进行解码。

dTdlMmtGQjZIQk45NEN3dTIzamdVdFduWGFQM2M0WWJySm1QU3hJeUxEMStKWTV2UUdlMVhjT05KZmdIeGZjOTZGSWYxU3NkUkltQQpPYTg0Z0Z0TkFqV0oyckxHOFppUGZGdnpRMzkrcjBuc0VIRVFmcUk0QWtnNHdTWmR2anNyU1JpYTBpVXZ3aFVtSlZUb1FZQng4Tk5XCnFtQjB2UjZjcVE3ZXB5elRVZlZhc2x1RTFsYjlENk1UREpkVG9qL3hZVGN5eENIWUtBejI1THI3Y0wzNXpVb3ZjelMybWxoUFBodVUKZzJPSStkY1Q3SkxEelNreE5VYlczTUg1b0ZwaVAvSytlUW05a1N1NFdLU0dKUWoxdWVVRHZXNjI1a1lheEpRMktrdzFJbHJoOUJGQQp0MEcvQVFoQU5uWGxkU1FhUC9LK2VRbTlrU3U0V0tTR0pRajF1YXFadjRiRWE5L1pxeEpWalVRSE0wUTA5WlJRaHhlRTNEVCtyS1VOClowSmRCQ2FzdHpvbnJOK29wWjhmZE5tdVRRPT0KP2tleUlkPTE=

这次终于没有了 UrlEncode 的样子，但是字符串后面有 = 号，所以尝试可能还有 Base64 编码，于是再对上面的数据进行 Base64 解码。发现可成功解码。

```
u7e2kFB6HBN94Cwu23jgUtWnXaP3c4YbrJmPSxIyLD1+JY5vQGe1XcONJfgHxfc96FIf1SsdRImA
Oa84gFtNAjWJ2rLG8ZiPfFvzQ39+r0nsEHEQfqI4Akg4wSZdvjsrSRia0iUvwhUmJVToQYBx8NNW
qmB0vR6cqQ7epyzTUfVasluE1lb9D6MTDJdToj/xYTcyxCHYKAz25Lr7cL35zUovczS2mlhPPhuU
g2OI+dcT7JLDzSkxNUbW3MH5oFpiP/K+eQm9kSu4WKSGJQj1ueUDvW625kYaxJQ2Kkw1Ilrh9BFA
t0G/AQhANnXldSQaP/K+eQm9kSu4WKSGJQj1uaqZv4bEa9/ZqxJVjUQHM0Q09ZRQhxeE3DT+rKUN
Z0JdBCastzonrN+opZ8fdNmuTQ==
?keyId=1


```

解码后的数据发现又有 == 号，再次尝试对其进行 Base64 解码，发现是解不动的，乱码的。

所以这部分的数据，应该是加密后的结果。

综合上面的分析可知，这款 App 将加密后的数据进行了 Base6 编码，然后又进行了 2 次 UrlEncode。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-fd486cf7463296352fec1c5c86fa001f6fe6449d.png)

复制这部分的数据，在日志中进行查找，发现是可以找的到的。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-39b62b07d88457283c4e51bbfcdc3d7cba4ff060.png)

在往上查找就会查找到加密的秘钥，由于这串字符串解密后会出现主机域名等信息，所以对其秘钥进行打码处理。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-40cd37b7ac6f73df5dd9ba125c775b54cbaf9f05.png)

某追星 App 的协议分析
-------------

### 抓包

这款 App 并没有做任何的抓包防御。

所以使用 HttpCanary 抓包即可。

在登录界面输入手机号 15026818188 密码 1234t6789

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-a9fbf29001c3cce5fb362920344fa2e7c7ebe069.png)

然后点击登录，抓到的包如下：

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-32ef3dc9c6d0d18568b48f075d5dc1ee93eb39cf.png)

### Java 层逆向

#### 定位关键代码

从上面抓到的包得知，加密的只有 pa 的值。

pa 的值是 MTY4OTUxNTM3Mzk0Nyw5YTI3MDE4NmU2YzA0YmU0YmM4MDgyMzRiZTdmNzU5OCwwNzMxMDk2ZDU4ZTg5OGNmNTg5YWEwYzI5YjRlOGVhOSw=

Base64 编码后的字符数，是 4 的倍数，编码的字节数是 3 的倍数时，不需要填充，如果不够字节数会填 0 补充，也就是后面会出现 = 号。

pa 值的字符数是 108 个，是 4 的 27 倍，并且后面有 = 号，所以推测 pa 的值是进行了 Base64 编码。

对其进行 Base64 解码：

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-8baa8cb019653edbd76c5cda2172e8c8f5c07616.png)

成功解码成功，证明 App 调用了 Base64 加密方法。

于是对 android.util.Base64.encodeToString 方法进行 hook，并打调用栈。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-f6d669199e28a3878a2f9ae53d4cb5fad0705851.png)

hook 后再次点击登录，成功获取到调用栈和返回值。

返回值就是 pa 的值。于是目光定位到上一层的方法 xxx.xxxxxx.xxxxx.xxxx.xxxx.kotlin.net.KTokenHeaderInterceptor.intercept 方法。

#### Java 层协议分析

将 Apk 拖入到 GDAE 中进行反编译。并找到 intercept 方法。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-5cd98689243575c4f5579a1b981f2563bca2e4df.png)

可以看到 str1 就是编码前的数据。

str1 又由四部分组成。

str1 = str1+','+str2+','+EncryptlibUtils.MD5(鼕鷙癵簾爩龘籲. 簾籲蠶爩 (), str1, str2, uCharSequenc1)+','+str;

第一部分：String str1 = String.valueOf(System.currentTimeMillis()); 这只是一个时间戳。

第二部分：String str2 = StringsKt__StringsJVMKt.replace$default(UUID.randomUUID().toString(), "-", "", false, 4, null);

看方法名中有 Strings replace 等字样，像是实现一个字符串替换的功能。

此时不妨用 frida hook 一下这个方法，观察一下参数和返回值即可。

```
setImmediate(function () {
    Java.perform(function () {
        var targetClass = decodeURIComponent('kotlin.text.StringsKt%5f%5fStringsJVMKt');
        var methodName = 'replace$default';
        var gclass = Java.use(targetClass);
        gclass[methodName].overload('java.lang.String', 'java.lang.String', 'java.lang.String', 'boolean', 'int', 'java.lang.Object').implementation = function (arg0, arg1, arg2, arg3, arg4, arg5) {
            console.log('\n[Hook replace$default(java.lang.String,java.lang.String,java.lang.String,boolean,int,java.lang.Object)]' + '\n\targ0 = ' + arg0 + '\n\targ1 = ' + arg1 + '\n\targ2 = ' + arg2 + '\n\targ3 = ' + arg3 + '\n\targ4 = ' + arg4 + '\n\targ5 = ' + arg5);
            var i = this[methodName](arg0, arg1, arg2, arg3, arg4, arg5);
            console.log('\treturn ' + i);
            return i;
        }
    })
})


```

hook 后得到的结果是，str2 就是生成了一串 UUID 然后删除了 - 符号后，重新拼接成了一串新的字符串。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-1929e6d7e495cda56268d8f28103ea35ed98a1b3.png)

第三部分：EncryptlibUtils.MD5(鼕鷙癵簾爩龘籲. 簾籲蠶爩 (), str1, str2, uCharSequenc1)

点 MD5 方法继续跟踪，发现 鼕鷙癵簾爩龘籲. 簾籲蠶爩 () 其实就是一个 Context 对象。然后其他三个就是字符串，前两个字符串一个是时间戳，一个是 UUID 。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-118a5647fed71e91f4c0a24a9ac289250c94754e.png)

第四部分：从解码后就可以得到是一个空字符串。

### So 层逆向

#### Md5 详解

MD5 摘要算法大概计算过程可以描述如下： MD5 将 “输入信息” 分为 N_512bit 的数据分组，每一 512bit 分组又分为 16 个 子分组，每个子分组为 32bit 的原始数据，16 个子分组分别命名为 M0~M15，每个子分组都要进行 4 次运算，运算公式分别为 FF、GG、HH、II，总的运算次数为 N_16*4（运算均为位运算）。

输入信息分组计算情况如下图所示：

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-a5445651b83a388311d8d4385ee6d1cf4a6160b8.png)

#### 标准的 Md5 开发

根据上面的原理，用 C 语言写了一份 Md5

```
#include <memory.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef struct
{
    unsigned int count[2];
    unsigned int state[4];
    unsigned char buffer[64];
}MD5_CTX;

#define F(x,y,z) ((x & y) | (~x & z))
#define G(x,y,z) ((x & z) | (y & ~z))
#define H(x,y,z) (x^y^z)
#define I(x,y,z) (y ^ (x | ~z))
#define ROTATE_LEFT(x,n) ((x << n) | (x >> (32-n)))
#define FF(a,b,c,d,x,s,ac) \
          { \
          a += F(b,c,d) + x + ac; \
          a = ROTATE_LEFT(a,s); \
          a += b; \
          }
#define GG(a,b,c,d,x,s,ac) \
          { \
          a += G(b,c,d) + x + ac; \
          a = ROTATE_LEFT(a,s); \
          a += b; \
          }
#define HH(a,b,c,d,x,s,ac) \
          { \
          a += H(b,c,d) + x + ac; \
          a = ROTATE_LEFT(a,s); \
          a += b; \
          }
#define II(a,b,c,d,x,s,ac) \
          { \
          a += I(b,c,d) + x + ac; \
          a = ROTATE_LEFT(a,s); \
          a += b; \
          }
void MD5Init(MD5_CTX* context);
void MD5Update(MD5_CTX* context, unsigned char* input, unsigned int inputlen);
void MD5Final(MD5_CTX* context, unsigned char digest[16]);
void MD5Transform(unsigned int state[4], unsigned char block[64]);
void MD5Encode(unsigned char* output, unsigned int* input, unsigned int len);
void MD5Decode(unsigned int* output, unsigned char* input, unsigned int len);

unsigned char PADDING[] = { 0x80,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
                         0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
                         0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
                         0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 };

void MD5Init(MD5_CTX* context)
{
    context->count[0] = 0;
    context->count[1] = 0;
    context->state[0] = 0x67452301;
    context->state[1] = 0xEFCDAB89;
    context->state[2] = 0x98BADCFE;
    context->state[3] = 0x10325476;
}
void MD5Update(MD5_CTX* context, unsigned char* input, unsigned int inputlen)
{
    unsigned int i = 0, index = 0, partlen = 0;
    index = (context->count[0] >> 3) & 0x3F;
    partlen = 64 - index;

    context->count[0] += inputlen << 3;
    if (context->count[0] < (inputlen << 3))
        context->count[1]++;
    context->count[1] += inputlen >> 29;

    if (inputlen >= partlen) {
        memcpy(&context->buffer[index], input, partlen);
        MD5Transform(context->state, context->buffer);
        for (i = partlen; i + 64 <= inputlen; i += 64)
            MD5Transform(context->state, &input[i]);
        index = 0;
    }
    else {
        i = 0;
    }
    memcpy(&context->buffer[index], &input[i], inputlen - i);
}
void MD5Final(MD5_CTX* context, unsigned char digest[16])
{
    unsigned int index = 0, padlen = 0;
    unsigned char bits[8];
    index = (context->count[0] >> 3) & 0x3F;
    padlen = (index < 56) ? (56 - index) : (120 - index);
    MD5Encode(bits, context->count, 8);
    MD5Update(context, PADDING, padlen);
    MD5Update(context, bits, 8);
    MD5Encode(digest, context->state, 16);
}
void MD5Encode(unsigned char* output, unsigned int* input, unsigned int len)
{
    unsigned int i = 0, j = 0;
    while (j < len) {
        output[j] = input[i] & 0xFF;
        output[j + 1] = (input[i] >> 8) & 0xFF;
        output[j + 2] = (input[i] >> 16) & 0xFF;
        output[j + 3] = (input[i] >> 24) & 0xFF;
        i++;
        j += 4;
    }
}
void MD5Decode(unsigned int* output, unsigned char* input, unsigned int len)
{
    unsigned int i = 0, j = 0;
    while (j < len) {
        output[i] = (input[j]) |
            (input[j + 1] << 8) |
            (input[j + 2] << 16) |
            (input[j + 3] << 24);
        i++;
        j += 4;
    }
}
void MD5Transform(unsigned int state[4], unsigned char block[64])
{
    unsigned int a = state[0];
    unsigned int b = state[1];
    unsigned int c = state[2];
    unsigned int d = state[3];
    unsigned int x[64];
    MD5Decode(x, block, 64);
    FF(a, b, c, d, x[0], 7, 0xd76aa478); 
    FF(d, a, b, c, x[1], 12, 0xe8c7b756); 
    FF(c, d, a, b, x[2], 17, 0x242070db); 
    FF(b, c, d, a, x[3], 22, 0xc1bdceee); 
    FF(a, b, c, d, x[4], 7, 0xf57c0faf); 
    FF(d, a, b, c, x[5], 12, 0x4787c62a); 
    FF(c, d, a, b, x[6], 17, 0xa8304613); 
    FF(b, c, d, a, x[7], 22, 0xfd469501); 
    FF(a, b, c, d, x[8], 7, 0x698098d8); 
    FF(d, a, b, c, x[9], 12, 0x8b44f7af); 
    FF(c, d, a, b, x[10], 17, 0xffff5bb1); 
    FF(b, c, d, a, x[11], 22, 0x895cd7be); 
    FF(a, b, c, d, x[12], 7, 0x6b901122); 
    FF(d, a, b, c, x[13], 12, 0xfd987193); 
    FF(c, d, a, b, x[14], 17, 0xa679438e); 
    FF(b, c, d, a, x[15], 22, 0x49b40821); 

    
    GG(a, b, c, d, x[1], 5, 0xf61e2562); 
    GG(d, a, b, c, x[6], 9, 0xc040b340); 
    GG(c, d, a, b, x[11], 14, 0x265e5a51); 
    GG(b, c, d, a, x[0], 20, 0xe9b6c7aa); 
    GG(a, b, c, d, x[5], 5, 0xd62f105d); 
    GG(d, a, b, c, x[10], 9, 0x2441453); 
    GG(c, d, a, b, x[15], 14, 0xd8a1e681); 
    GG(b, c, d, a, x[4], 20, 0xe7d3fbc8); 
    GG(a, b, c, d, x[9], 5, 0x21e1cde6); 
    GG(d, a, b, c, x[14], 9, 0xc33707d6); 
    GG(c, d, a, b, x[3], 14, 0xf4d50d87); 
    GG(b, c, d, a, x[8], 20, 0x455a14ed); 
    GG(a, b, c, d, x[13], 5, 0xa9e3e905); 
    GG(d, a, b, c, x[2], 9, 0xfcefa3f8); 
    GG(c, d, a, b, x[7], 14, 0x676f02d9); 
    GG(b, c, d, a, x[12], 20, 0x8d2a4c8a); 

    
    HH(a, b, c, d, x[5], 4, 0xfffa3942); 
    HH(d, a, b, c, x[8], 11, 0x8771f681); 
    HH(c, d, a, b, x[11], 16, 0x6d9d6122); 
    HH(b, c, d, a, x[14], 23, 0xfde5380c); 
    HH(a, b, c, d, x[1], 4, 0xa4beea44); 
    HH(d, a, b, c, x[4], 11, 0x4bdecfa9); 
    HH(c, d, a, b, x[7], 16, 0xf6bb4b60); 
    HH(b, c, d, a, x[10], 23, 0xbebfbc70); 
    HH(a, b, c, d, x[13], 4, 0x289b7ec6); 
    HH(d, a, b, c, x[0], 11, 0xeaa127fa); 
    HH(c, d, a, b, x[3], 16, 0xd4ef3085); 
    HH(b, c, d, a, x[6], 23, 0x4881d05); 
    HH(a, b, c, d, x[9], 4, 0xd9d4d039); 
    HH(d, a, b, c, x[12], 11, 0xe6db99e5); 
    HH(c, d, a, b, x[15], 16, 0x1fa27cf8); 
    HH(b, c, d, a, x[2], 23, 0xc4ac5665); 

    
    II(a, b, c, d, x[0], 6, 0xf4292244); 
    II(d, a, b, c, x[7], 10, 0x432aff97); 
    II(c, d, a, b, x[14], 15, 0xab9423a7); 
    II(b, c, d, a, x[5], 21, 0xfc93a039); 
    II(a, b, c, d, x[12], 6, 0x655b59c3); 
    II(d, a, b, c, x[3], 10, 0x8f0ccc92); 
    II(c, d, a, b, x[10], 15, 0xffeff47d); 
    II(b, c, d, a, x[1], 21, 0x85845dd1); 
    II(a, b, c, d, x[8], 6, 0x6fa87e4f); 
    II(d, a, b, c, x[15], 10, 0xfe2ce6e0); 
    II(c, d, a, b, x[6], 15, 0xa3014314); 
    II(b, c, d, a, x[13], 21, 0x4e0811a1); 
    II(a, b, c, d, x[4], 6, 0xf7537e82); 
    II(d, a, b, c, x[11], 10, 0xbd3af235); 
    II(c, d, a, b, x[2], 15, 0x2ad7d2bb); 
    II(b, c, d, a, x[9], 21, 0xeb86d391); 
    state[0] += a;
    state[1] += b;
    state[2] += c;
    state[3] += d;
}

int main(int argc, char* argv[])
{
    int i;
    unsigned char encrypt[] = "admin";
    unsigned char decrypt[16];
    MD5_CTX md5;
    MD5Init(&md5);
    MD5Update(&md5, encrypt, strlen((char*)encrypt));
    MD5Final(&md5, decrypt);
    printf("Before encryption:%s\nAfter encryption:", encrypt);
    for (i = 0; i < 16; i++) {
        printf("%02x", decrypt[i]);
    }

    return 0;
}


```

#### C 层逆向 - 判断 Md5

通过 Java 层的逆向分析，所以要找到 libencryptlib.so 库，然后找到 MD5 函数，因为是静态注册，所以直接找 包类函数名 即可。

找到函数后，进行反编译。

反编译后前面的内容不进行讲解，只说 Md5 部分。

因为前面已经讲解了 Md5 是如何开发的，所以 MD5_CTX::MD5_CTX(&48) 看起来就很像，Md5 的初始化。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-965ce3f5786d018daee4b6a492df62ee1f6446c4.png)

点进去 MD5_CTX ，可以看到初始化了运算的数字 0xEFCDAB8967452301，0x1032547698BADCFE。

并且由于 Md5 的数据要填充到 448 字节，后面先填充 1 在填充 0 ，1000 0000 0000…… 对应的前面两个字节就是 0x80 了。所以 his->PADDING[0] = 0x80 也能看出来这是一个 Md5。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-b93fce275a36ce88a13184261717f8775bc21027.png)

后面还有 MD5_CTX::MakePassMD5 方法，这很可能就是 Md5 的运算部分了。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-360a688688a9e9d4f0f290368aaf842ce764ece9.png)

点进去 MakePassMD5 ，由对 Md5 的详解可知，这里面的运算一定有，下面写好的值在里面。

除了初始化的四个值能判断 Md5，64 个参与运算的值和左移的位数，也可判断是 Md5。

```
FF(a, b, c, d, x[0], 7, 0xd76aa478); 
FF(d, a, b, c, x[1], 12, 0xe8c7b756); 
FF(c, d, a, b, x[2], 17, 0x242070db); 
FF(b, c, d, a, x[3], 22, 0xc1bdceee); 
FF(a, b, c, d, x[4], 7, 0xf57c0faf); 
FF(d, a, b, c, x[5], 12, 0x4787c62a); 
FF(c, d, a, b, x[6], 17, 0xa8304613); 


```

点进函数，一直追踪到 MD5Transform 函数，对数字按 H 键可以转换为 16 进制。

可以看到 -0x28955B88 虽然不是 +0xd76aa478 ，但是 +0xd76aa478 是 0xffffffff - -0x28955B88 的结果。

0x242070db 就对应了 FF(c, d, a, b, x[2], 17, 0x242070db)。所以这是一个标准的 Md5。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-0a63d67c5c8ad30a8b06e3e9239772b9b118f81f.png)

#### C 层逆向 - 协议分析

通过对上面的分析，这就是一个标准的 Md5 算法，只不过 C/C++ 没有提供加密库，只能手动来完成。

观察 MakePassMD5 函数可知，第 1 个参数应该就是输入的原始字符串，第 2 个参数是输入的长度，第 3 个参数应该就是加密后的结果了。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-03788256110afb2754de03df33aa23211ae7d033.png)

为了验证猜想，编写 hook 代码。

```
var funcAddr = Module.findExportByName("libencryptlib.so", "_ZN7MD5_CTX11MakePassMD5EPhjS0_");
console.log(funcAddr);
Interceptor.attach(funcAddr, {
    onEnter: function (args) {
        console.log("funcAddr onEnter args[1]: ", hexdump(args[1]));
        console.log("funcAddr onEnter args[2]: ", args[2].toInt32());
        this.args3 = args[3];
    }, onLeave: function (retval) {
        console.log("funcAddr onLeave args[3]: ", hexdump(this.args3));
    }
});


```

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-f04a445e4678c36e1f66cd30058ffe90a7e8d755.png)

果然参数 1 取 Md5 就是参数 3。

![](https://shs3.b.qianxin.com/attack_forum/2023/07/attach-49d8f44d90b5ef0fa8a1a9bb68278128f3aac528.png)

协议分析到此结束，pa 的值就是 时间戳 + UUID + C 层 Md5 运算（时间戳 + UUID）。

通过对上面 3 个 App 的协议逆向分析可知，hook Java 标准加密库的方法，可以拿到加密方式，秘钥，iv，加密模式等信息。拿到这些信息就相当于拿到了明文，App 的数据就处于一种 “裸奔” 的状态，是非常不安全的。

所以在开发中尽量不要靠调用 Java 的标准加密库来对数据进行加密，尽量在 SO 层自己实现算法完成加密。

即使不能这样做，也可以参考第二种 App 的方式，对加密后的数据或原数据，进行一定量的反复编码操作，此时如果数据流足够大，hook 到的加解密信息日志中，由于无法搜索匹配到抓包中的数据，也会提高安全性。

对 So 层的加密分析虽然困难了一些，但只要对标准加密算法足够了解，知道加密的步骤和标准加密算法一定用到的常数据和位移数据，虽然分析会耗时长一些，但是也是可以分析出来的。