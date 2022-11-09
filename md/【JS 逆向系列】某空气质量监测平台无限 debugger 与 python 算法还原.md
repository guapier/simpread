> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1595155-1-1.html)

> [md]@[TOC](【JS 逆向系列】某空气质量监测平台无限 debugger 与 python 算法还原)## 1. 前置阅读样品地址：aHR0cHM6Ly93d3cuYXFpc3R1ZHkuY24v 本篇......

 ![](https://avatar.52pojie.cn/data/avatar/000/55/71/95_avatar_middle.jpg) 漁滒

@[TOC](【JS 逆向系列】某空气质量监测平台无限 debugger 与 python 算法还原)

1. 前置阅读
-------

样品地址：aHR0cHM6Ly93d3cuYXFpc3R1ZHkuY24v  
本篇文章可能会省略某些过程，直接使用下面文章的结果，所以建议先阅读下面的文章。本篇文章也有与其他不一样的处理方法，以及单纯使用 python 来还原数据的加密和解密。  
[1. 某空气质量监测平台无限 debugger 以及数据动态加密分析](https://mp.weixin.qq.com/s/ithWRqLCIzAi-CKiD6mMdg)  
[2.Python 爬虫进阶必备 - 以 aqistudy 为例的无限 debugger 反调试绕过演示（附视频）](https://mp.weixin.qq.com/s/P9dbinE7Bua-7ULBQaIvRA)

2. 过反调试
-------

先打开 f12 后，打开网页，直接遇到 debugger

![](https://img-blog.csdnimg.cn/b4b847625705470989a25419a13c9a2e.png?)

此时通过调用堆栈查找最上层

![](https://img-blog.csdnimg.cn/a30feb7e8f1b441ca59a83b51dab71b5.png?)  
点击后跳转到主页源代码

![](https://img-blog.csdnimg.cn/cab3a9ace892489f9de99ea1313806c2.png?)  
debugger 是出现在 txsdefwsw 函数里面，如果有办法可以使得这个函数不执行，那是不是就相当于不会出现 debugger 了。

这里参考了志远哥的 fd 插件里面，hook 注入功能的想法。例如在网页的最前面加入自己的代码，或者修改源代码中的内容，就可以达到某些想要的效果。这里我是用的是 mitmproxy 这个软件

首先进行安装，直接使用 pip

```
pip install mitmproxy
```

安装完成后编写一个处理响应的脚本，并命名为 main.py

```
import mitmproxy.http
import re

print('脚本初始化成功')

def request(flow: mitmproxy.http.HTTPFlow):
    pass

def response(flow: mitmproxy.http.HTTPFlow):
    if 'https://www.aqistudy.cn/' == flow.request.url:
        html = flow.response.text
        html = html.replace('txsdefwsw();', '// txsdefwsw();')
        flow.response.text = html
```

然后在命令行启动 mitmproxy

```
mitmdump -q -p 8888 -s main.py
```

其中 -q 表示静默运行。-p 表示监听的端口，-s 表示处理的 python 脚本

然后打开【网络和 internet】设置，去到代 {过}{滤} 理

![](https://img-blog.csdnimg.cn/c445cec9ce4b4f918d0b4a09da5d749d.png?)

打开代 {过}{滤} 理服务器开关，地址填写【127.0.0.1】，端口填写刚刚上面的【8888】，然后点击保存。这是再次打开网页

![](https://img-blog.csdnimg.cn/e59574e346d1458b9f9d4d6a0df4c8ec.png?)

此时可以看到 txsdefwsw 函数已经被注释了，自然就没有出现 debugger 了。但是出现了其他的反调试情况。上面的【document.write('检测到非法调试, 请关闭调试终端后刷新本页面重试!');】重写了页面，为了不让它直接，在前面加上 return

```
def response(flow: mitmproxy.http.HTTPFlow):
    if 'https://www.aqistudy.cn/' == flow.request.url:
        html = flow.response.text
        html = html.replace('txsdefwsw();', '// txsdefwsw();')
        html = html.replace("document.write('检测到非法调试, 请关闭调试终端后刷新本页面重试!');",
                            "return; document.write('检测到非法调试, 请关闭调试终端后刷新本页面重试!');")
        flow.response.text = html
```

再次刷新网页

![](https://img-blog.csdnimg.cn/9cf60484079b4e30a72b2d5e04d09e7c.png?)  
可以看到最外层的页面没有被重写，说明这里的反调试已经过了。但是这时又出现了之前无限 debugger 的情况。再次通过调用堆栈的最上层

![](https://img-blog.csdnimg.cn/3531988ca5e543ac83519bf253497cfe.png?)  
发现是这个 eval 出来的，这里有两个 eval，只注释第一个有反调试检测的

```
def response(flow: mitmproxy.http.HTTPFlow):
    if 'https://www.aqistudy.cn/' == flow.request.url:
        html = flow.response.text
        html = html.replace('txsdefwsw();', '// txsdefwsw();')
        html = html.replace("document.write('检测到非法调试, 请关闭调试终端后刷新本页面重试!');",
                            "return; document.write('检测到非法调试, 请关闭调试终端后刷新本页面重试!');")
        flow.response.text = html
    elif 'html/city_realtime.php' in flow.request.url:
        html = flow.response.text
        js = re.findall('eval\(.+', html)[0]
        html = html.replace(js, '// ' + js)
        flow.response.text = html
```

最后再次刷新网页

![](https://img-blog.csdnimg.cn/6db008ebbf05467d9238d9f6b3274ee1.png?)  
可以看到注释以后，不会再出现反调试的情况，这时反调试已经过完，可以进行 js 代码分析

3.js 分析
-------

分析过程给出的文章中，均有详细的介绍，所以本篇文章对分析部分跳过一些内容，仅对关键地方介绍。

![](https://img-blog.csdnimg.cn/5c77758b92ca47ea93b55c5dcb483d12.png?)

数据是通过这个接口获取的，请求体和响应体都被加密了

![](https://img-blog.csdnimg.cn/8de80983037940a6888af10f063b7cab.png?)  
![](https://img-blog.csdnimg.cn/453b8cb4a9644031ab2e4b32f3d501b4.png?)  
通过调用堆栈，很容易定位到函数调用的地方，这里是通过 gHfcltaYLa11POt6 函数对参数进行加密，请求成功后，调用 ddI2ovOg52ToSyLv1nEa 函数进行解密。

但是直接扣代码的话，也没有办法快速解决，因为这个 js 是会不断改变了，按照前面文章的说法，这个 js 平均每 10 分钟变一次，所以并不能简单的把代码扣下来直接用。

经过我向多个大佬询问，这个网站的 js 会变，但是接口的加密逻辑是不变的。如果把逻辑改成 python，那么只有逻辑不变，就可以一直跑了。

4. 代码逻辑改写
---------

首先第一步就是获取加密的源代码，也就是 gHfcltaYLa11POt6 函数的改写

首先从 city_realtime.php?v=2.3 中提取带有【/js/encrypt_】的 js 链接，并获取 js 内容

```
requests = requests_html.HTMLSession()
    url = 'https://www.aqistudy.cn/html/city_realtime.php?v=2.3'
    headers = {
        'User-Agent': 'Mozilla/5.0(WindowsNT10.0;WOW64)AppleWebKit/537.36(KHTML,likeGecko)Chrome/69.0.3497.100Safari/537.36',
    }
    response = requests.get(url, headers=headers)
    url = filter(lambda n: '/js/encrypt_' in n.attrs['src'], filter(lambda n: 'src' in n.attrs.keys(), response.html.xpath('//script'))).__next__().attrs['src'].replace('../js/', 'https://www.aqistudy.cn/js/')
    print(url)
    response = requests.get(url, headers=headers)
    data = response.text
```

然后需要执行代码，获取 eval 之后的内容，eval 后如果存在 dswejwehxt，还需要进行 base64 解码

```
while data.startswith("eval("):
        with open('temp.js', 'w', encoding='utf-8') as f:
            f.write('console.log(' + data.strip()[5:-1] + ')')

        nodejs = subprocess.Popen('node temp', stderr=subprocess.PIPE, stdout=subprocess.PIPE)
        data = nodejs.stdout.read().decode().replace('\n', '')

        if 'dswejwehxt(dswejwehxt' in data:
            data = re.findall('(?<=dswejwehxt\(dswejwehxt\().+?(?=\))', data)[0][1:-1]
            data = base64.b64decode(base64.b64decode(data.encode())).decode()
        elif 'dswejwehxt(' in data:
            data = re.findall('(?<=dswejwehxt\().+?(?=\))', data)[0][1:-1]
            data = base64.b64decode(data.encode()).decode()
```

这时，data 已经是明文的 js 代码，但是这里的 js 有可能是压缩的，也有可能是没有压缩的。这样对后面使用正则匹配非常不友好。所以这里使用 ast 统一格式化代码

```
const parser = require("@babel/parser");
const generator = require("@babel/generator");
const fs = require("fs");

console.log(generator.default(parser.parse(fs.readFileSync('temp.js').toString('utf-8')), {
    compact: false,
    comments: false,
    jsescOption: {
        minimal: true
    }
}).code);
```

保存为【script.js】，并放到 py 同目录

```
# 格式化代码
    with open('temp.js', 'w', encoding='utf-8') as f:
        f.write(data)

    nodejs = subprocess.Popen('node script', stderr=subprocess.PIPE, stdout=subprocess.PIPE)
    data = nodejs.stdout.read().decode()
```

这个时候的 data 就是统一格式化后的 js 代码，这时可以使用正则提取里面的内容，为加密做准备。

```
appId = re.findall("(?<=var appId = ').+?(?=')", data)[0]
    clienttype = 'WEB'
    timestamp = int(time.time() * 1000)
    method = 'GETDATA'
    objectdata = {
        'city': '杭州'
    }
    param = {
        'appId': appId,
        'method': method,
        'timestamp': timestamp,
        'clienttype': clienttype,
        'object': objectdata,
        'secret': MD5.new((appId + method + str(timestamp) + clienttype + json.dumps(objectdata, ensure_ascii=False, separators=(',', ':'))).encode()).hexdigest()
    }
    print(param)
    param = base64.b64encode(json.dumps(param, ensure_ascii=False, separators=(',', ':')).encode()).decode()
    print(param)
```

这时 param 参数已经组包完成。但是上面文章中有提及。param 参数有三种可能，aes 加密，des 加密和不加密，所以还需要做一个判断

```
def encrypt_data_aes(text, key, iv):
    secretkey = MD5.new(key.encode()).hexdigest()[16:]
    secretiv = MD5.new(iv.encode()).hexdigest()[:16]
    crypto = AES.new(key=secretkey.encode(), mode=AES.MODE_CBC, iv=secretiv.encode())
    return base64.b64encode(crypto.encrypt(pad(text.encode(), AES.block_size))).decode()

def encrypt_data_des(text, key, iv):
    secretkey = MD5.new(key.encode()).hexdigest()[:8]
    secretiv = MD5.new(iv.encode()).hexdigest()[24:]
    crypto = DES.new(key=secretkey.encode(), mode=DES.MODE_CBC, iv=secretiv.encode())
    return base64.b64encode(crypto.encrypt(pad(text.encode(), DES.block_size))).decode()

if 'param = AES.encrypt' in data:
    keyid = re.findall('(?<=param = AES\.encrypt\(param, ).+?(?=,)', data)[0]
    key = re.findall('(?<=const )' + keyid + ' = ".+?(?=")', data)[0].split('"')[-1]
    ivid = re.findall("(?<=, )" + keyid + ', .+?(?=\))', data)[0].split(', ')[-1]
    iv = re.findall('(?<=const )' + ivid + ' = ".+?(?=")', data)[0].split('"')[-1]
    param = encrypt_data_aes(param, key, iv)
elif 'param = DES.encrypt' in data:
    keyid = re.findall('(?<=param = DES\.encrypt\(param, ).+?(?=,)', data)[0]
    key = re.findall('(?<=const )' + keyid + ' = ".+?(?=")', data)[0].split('"')[-1]
    ivid = re.findall("(?<=, )" + keyid + ', .+?(?=\))', data)[0].split(', ')[-1]
    iv = re.findall('(?<=const )' + ivid + ' = ".+?(?=")', data)[0].split('"')[-1]
    param = encrypt_data_des(param, key, iv)
```

然后到最关键的请求接口

```
dataid = re.findall('(?<=data: \{).+?(?=\})', data, re.S)[0].replace('\n', '').strip().split(':')[0]
    url = 'https://www.aqistudy.cn/apinew/aqistudyapi.php'
    postdata = {
        dataid: param
    }
    response = requests.post(url, headers=headers, data=postdata)
    print(response.text)
```

这时测试，可以成功获取到加密的响应体，接着就是还原解密的 ddI2ovOg52ToSyLv1nEa 函数，根据 js 是固定先进行 aes 解密，得到的结果再进行 des 解密，那么可以得到下面的 python 代码

```
def decrypt_data(text, data):
    keyid = re.findall('(?<=data = AES\.decrypt\(data, ).+?(?=,)', data)[0]
    key = re.findall('(?<=const )' + keyid + ' = ".+?(?=")', data)[0].split('"')[-1]
    ivid = re.findall('(?<=, )' + keyid + ', .+?(?=\))', data)[0].split(', ')[-1].split(',')[-1]
    iv = re.findall('(?<=const )' + ivid + ' = ".+?(?=")', data)[0].split('"')[-1]
    secretkey = MD5.new(key.encode()).hexdigest()[16:]
    secretiv = MD5.new(iv.encode()).hexdigest()[:16]
    crypto = AES.new(key=secretkey.encode(), mode=AES.MODE_CBC, iv=secretiv.encode())
    text = unpad(crypto.decrypt(base64.b64decode(text.encode())), AES.block_size)
    keyid = re.findall('(?<=data = DES\.decrypt\(data, ).+?(?=,)', data)[0]
    key = re.findall('(?<=const )' + keyid + ' = ".+?(?=")', data)[0].split('"')[-1]
    ivid = re.findall('(?<=, )' + keyid + ', .+?(?=\))', data)[0].split(', ')[-1].split(',')[-1]
    iv = re.findall('(?<=const )' + ivid + ' = ".+?(?=")', data)[0].split('"')[-1]
    secretkey = MD5.new(key.encode()).hexdigest()[:8]
    secretiv = MD5.new(iv.encode()).hexdigest()[24:]
    crypto = DES.new(key=secretkey.encode(), mode=DES.MODE_CBC, iv=secretiv.encode())
    text = unpad(crypto.decrypt(base64.b64decode(text)), DES.block_size)
    return base64.b64decode(text).decode()
```

测试解密正常，完整代码如下

```
import requests_html
import time
import json
import base64
import re
import subprocess
from Crypto.Util.Padding import pad, unpad
from Crypto.Cipher import AES, DES
from Crypto.Hash import MD5

def encrypt_data_aes(text, key, iv):
    secretkey = MD5.new(key.encode()).hexdigest()[16:]
    secretiv = MD5.new(iv.encode()).hexdigest()[:16]
    crypto = AES.new(key=secretkey.encode(), mode=AES.MODE_CBC, iv=secretiv.encode())
    return base64.b64encode(crypto.encrypt(pad(text.encode(), AES.block_size))).decode()

def encrypt_data_des(text, key, iv):
    secretkey = MD5.new(key.encode()).hexdigest()[:8]
    secretiv = MD5.new(iv.encode()).hexdigest()[24:]
    crypto = DES.new(key=secretkey.encode(), mode=DES.MODE_CBC, iv=secretiv.encode())
    return base64.b64encode(crypto.encrypt(pad(text.encode(), DES.block_size))).decode()

def decrypt_data(text, data):
    keyid = re.findall('(?<=data = AES\.decrypt\(data, ).+?(?=,)', data)[0]
    key = re.findall('(?<=const )' + keyid + ' = ".+?(?=")', data)[0].split('"')[-1]
    ivid = re.findall('(?<=, )' + keyid + ', .+?(?=\))', data)[0].split(', ')[-1].split(',')[-1]
    iv = re.findall('(?<=const )' + ivid + ' = ".+?(?=")', data)[0].split('"')[-1]
    secretkey = MD5.new(key.encode()).hexdigest()[16:]
    secretiv = MD5.new(iv.encode()).hexdigest()[:16]
    crypto = AES.new(key=secretkey.encode(), mode=AES.MODE_CBC, iv=secretiv.encode())
    text = unpad(crypto.decrypt(base64.b64decode(text.encode())), AES.block_size)
    keyid = re.findall('(?<=data = DES\.decrypt\(data, ).+?(?=,)', data)[0]
    key = re.findall('(?<=const )' + keyid + ' = ".+?(?=")', data)[0].split('"')[-1]
    ivid = re.findall('(?<=, )' + keyid + ', .+?(?=\))', data)[0].split(', ')[-1].split(',')[-1]
    iv = re.findall('(?<=const )' + ivid + ' = ".+?(?=")', data)[0].split('"')[-1]
    secretkey = MD5.new(key.encode()).hexdigest()[:8]
    secretiv = MD5.new(iv.encode()).hexdigest()[24:]
    crypto = DES.new(key=secretkey.encode(), mode=DES.MODE_CBC, iv=secretiv.encode())
    text = unpad(crypto.decrypt(base64.b64decode(text)), DES.block_size)
    return base64.b64decode(text).decode()

def main():
    requests = requests_html.HTMLSession()
    url = 'https://www.aqistudy.cn/html/city_realtime.php?v=2.3'
    headers = {
        'User-Agent': 'Mozilla/5.0(WindowsNT10.0;WOW64)AppleWebKit/537.36(KHTML,likeGecko)Chrome/69.0.3497.100Safari/537.36',
    }
    response = requests.get(url, headers=headers)
    url = filter(lambda n: '/js/encrypt_' in n.attrs['src'], filter(lambda n: 'src' in n.attrs.keys(), response.html.xpath('//script'))).__next__().attrs['src'].replace('../js/', 'https://www.aqistudy.cn/js/')
    print(url)
    response = requests.get(url, headers=headers)
    data = response.text
    while data.startswith("eval("):
        with open('temp.js', 'w', encoding='utf-8') as f:
            f.write('console.log(' + data.strip()[5:-1] + ')')

        nodejs = subprocess.Popen('node temp', stderr=subprocess.PIPE, stdout=subprocess.PIPE)
        data = nodejs.stdout.read().decode().replace('\n', '')

        if 'dswejwehxt(dswejwehxt' in data:
            data = re.findall('(?<=dswejwehxt\(dswejwehxt\().+?(?=\))', data)[0][1:-1]
            data = base64.b64decode(base64.b64decode(data.encode())).decode()
        elif 'dswejwehxt(' in data:
            data = re.findall('(?<=dswejwehxt\().+?(?=\))', data)[0][1:-1]
            data = base64.b64decode(data.encode()).decode()

    # 格式化代码
    with open('temp.js', 'w', encoding='utf-8') as f:
        f.write(data)

    nodejs = subprocess.Popen('node script', stderr=subprocess.PIPE, stdout=subprocess.PIPE)
    data = nodejs.stdout.read().decode()

    appId = re.findall("(?<=var appId = ').+?(?=')", data)[0]
    clienttype = 'WEB'
    timestamp = int(time.time() * 1000)
    method = 'GETDATA'
    objectdata = {
        'city': '杭州'
    }
    param = {
        'appId': appId,
        'method': method,
        'timestamp': timestamp,
        'clienttype': clienttype,
        'object': objectdata,
        'secret': MD5.new((appId + method + str(timestamp) + clienttype + json.dumps(objectdata, ensure_ascii=False, separators=(',', ':'))).encode()).hexdigest()
    }
    print(param)
    param = base64.b64encode(json.dumps(param, ensure_ascii=False, separators=(',', ':')).encode()).decode()
    print(param)
    if 'param = AES.encrypt' in data:
        keyid = re.findall('(?<=param = AES\.encrypt\(param, ).+?(?=,)', data)[0]
        key = re.findall('(?<=const )' + keyid + ' = ".+?(?=")', data)[0].split('"')[-1]
        ivid = re.findall("(?<=, )" + keyid + ', .+?(?=\))', data)[0].split(', ')[-1]
        iv = re.findall('(?<=const )' + ivid + ' = ".+?(?=")', data)[0].split('"')[-1]
        param = encrypt_data_aes(param, key, iv)
    elif 'param = DES.encrypt' in data:
        keyid = re.findall('(?<=param = DES\.encrypt\(param, ).+?(?=,)', data)[0]
        key = re.findall('(?<=const )' + keyid + ' = ".+?(?=")', data)[0].split('"')[-1]
        ivid = re.findall("(?<=, )" + keyid + ', .+?(?=\))', data)[0].split(', ')[-1]
        iv = re.findall('(?<=const )' + ivid + ' = ".+?(?=")', data)[0].split('"')[-1]
        param = encrypt_data_des(param, key, iv)

    dataid = re.findall('(?<=data: \{).+?(?=\})', data, re.S)[0].replace('\n', '').strip().split(':')[0]
    url = 'https://www.aqistudy.cn/apinew/aqistudyapi.php'
    postdata = {
        dataid: param
    }
    response = requests.post(url, headers=headers, data=postdata)
    print(response.text)
    data = decrypt_data(response.text, data)
    print(data)

if __name__ == '__main__':
    main()
```

![](https://img-blog.csdnimg.cn/9881c0412fad4607beab1aa7cbb47fbb.png?) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) 牛逼！  
大佬思路很清晰  
我一般都是到了 JS 解密那里就放弃了  
有时候运气好把解密搞出来了  
到后面算法又放弃了![](https://static.52pojie.cn/static/image/smiley/default/13.gif) ![](https://avatar.52pojie.cn/data/avatar/000/55/58/92_avatar_middle.jpg) Rbtl WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', TimeoutError('_ssl.c:980: The handshake operation timed out'))': /simple/mitmproxy/  
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', TimeoutError('_ssl.c:980: The handshake operation timed out'))': /simple/mitmproxy/  
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', TimeoutError('_ssl.c:980: The handshake operation timed out'))': /simple/mitmproxy/  
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', TimeoutError('_ssl.c:980: The handshake operation timed out'))': /simple/mitmproxy/  
WARNING: Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', TimeoutError('_ssl.c:980: The handshake operation timed out'))': /simple/mitmproxy/  
ERROR: Could not find a version that satisfies the requirement mitmproxy (from versions: none)  
ERROR: No matching distribution found for mitmproxy![](https://avatar.52pojie.cn/data/avatar/000/37/54/56_avatar_middle.jpg)geniusrot 前排占个位，哈哈 ![](https://avatar.52pojie.cn/data/avatar/000/15/24/83_avatar_middle.jpg) wakichie 哎，现在的 js 都挺难了![](https://avatar.52pojie.cn/images/noavatar_middle.gif)不苦小和尚 学到了！![](https://avatar.52pojie.cn/images/noavatar_middle.gif)删掉丶关于 n1 其实我本人一直很想学类似的逆向，但是不知道应该从何学起，看大佬写的文章能一知半解，可自己试的时候总感觉捉襟见肘 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) cyxnzb 占个位置 ![](https://avatar.52pojie.cn/data/avatar/000/93/53/52_avatar_middle.jpg) LQ6Htty 大佬就是牛逼，顶你 ![](https://avatar.52pojie.cn/data/avatar/000/43/16/52_avatar_middle.jpg) xixicoco

> [cyxnzb 发表于 2022-2-28 23:31](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=41740221&ptid=1595155)  
> 其实我本人一直很想学类似的逆向，但是不知道应该从何学起，看大佬写的文章能一知半解，可自己试的时候总感 ...

找大佬 报班吧，哈哈![](https://avatar.52pojie.cn/images/noavatar_middle.gif)似水流年 2015 插眼  学习下大佬操作