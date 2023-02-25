> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.null119.cn](https://www.null119.cn/index.php/archives/251/)

> 视频教程 https://www.bilibili.com/video/BV1aN4y1V7cb / 传统 RPC 插入 websocket 相关代码! function () { if (!window...

### 视频教程

> [https://www.bilibili.com/video/BV1aN4y1V7cb/](https://www.bilibili.com/video/BV1aN4y1V7cb/)

### 传统 RPC

#### 插入 websocket 相关代码

```
!function () {
    if (!window._makeRequest){
        window._makeRequest = makeRequest;
        var ws = new WebSocket("ws://127.0.0.1:16666");
        ws.open = function (param){};
        ws.onmessage=function(param){
            var mdata=param.data;
            var u=mdata.split('|')[0]
            var p=mdata.split('|')[1]
            var result = window._makeRequest(u,p, 7, false);
            ws.send(JSON.stringify(result))
        }
    }
}();

```

#### Python websocket Server

```
# -*- coding: utf-8 -*-
# @Author: Null119 微信公众号/网站：治廷君
# @Desc: { js RPC示例 }
# @Date: { 2022/8/18 }
import asyncio,websockets

async def recv_msg(websocket):
    while True:
        print(await websocket.recv())

async def main(websocket,path):
    await websocket.send('13388888888|testtest22222')
    await recv_msg(websocket)

server=websockets.serve(main,'127.0.0.1',16666)
asyncio.get_event_loop().run_until_complete(server)
asyncio.get_event_loop().run_forever()

```

### Sekiro 框架

#### 官方文档地址

> [https://sekiro.virjar.com/sekiro-doc/](https://sekiro.virjar.com/sekiro-doc/)

#### demoServer 下载地址

> [https://null119.lanzoul.com/iHdw709rydba](https://null119.lanzoul.com/iHdw709rydba)

#### 插入 JS 代码

```
if(!window._makeRequest){window._makeRequest = makeRequest}

```

#### 通过油猴加载 sekiro_web_client 及 SekiroClient 相关代码

```
// ==UserScript==
// @name         sekrio
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  try to take over the world!
// @author       You
// @match        https://weibo.com/*
// @icon         https://www.google.com/s2/favicons?sz=64&domain=jianshu.com
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // Your code here...
    var _mscript=document.createElement("script")
    _mscript.src="https://sekiro.virjar.com/sekiro-doc/assets/sekiro_web_client.js"
    document.body.appendChild(_mscript);
    function sek_start(){
        function guid() {
            function S4() {
                return (((1 + Math.random()) * 0x10000) | 0).toString(16).substring(1);
            }
            return (S4() + S4() + "-" + S4() + "-" + S4() + "-" + S4() + "-" + S4() + S4() + S4());
        }
        var client = new SekiroClient("ws://127.0.0.1:5620/business-demo/register?group=test&clientId=" + guid());
        client.registerAction("makeRequest", function (request, resolve, reject) {
            try {
                var _user=request['user']
                var _psw=request['psw']
                var result=JSON.stringify(window._makeRequest(_user,_psw,7,false))
                resolve(result);
            } catch (e) {
                reject("error: " + e);
            }
        });
    }
    setTimeout(sek_start,2000)
})();

```

#### RPC 调用（python 示例）

```
# -*- coding: utf-8 -*-
# @Author: Null119 微信公众号/网站：治廷君
# @Desc: { 微博登录Sekiro_RPC示例 }
# @Date: { 2022/8/18 }

import requests

r=requests.Session()
pdata={
    'group':'test',
    'action':'makeRequest',
    'user':'13255557777',
    'psw':'8888888567'
}

response=requests.get("http://127.0.0.1:5620/business-demo/invoke",params=pdata)
print(response.text)

```