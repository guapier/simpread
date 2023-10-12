> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/xiaoweigege/p/15755794.html)

sekiro 部署及 js-rpc-demo
======================

1. sekiro 服务端部署[#](#1-sekiro服务端部署)
----------------------------------

*   [项目地址](https://github.com/virjar/sekiro)
*   [项目文档](https://sekiro.virjar.com/sekiro-doc/index.html)
*   [项目文件](https://oss.virjar.com/sekiro/sekiro-demo/sekiro-release-demo-20210823.zip?download=true)

运行启动脚本:

1.  bin/sekiro.sh :mac or linux
2.  bin/sekiro.bat :windows

到此项目就运行成功了, 但是这不是 https 项目 js rpc 是通过 websocket 进行调用的, 在 https 站点中, 不是 wss 链接 是不成功的, 所以我们要配合 nginx 进行 搭建

2. nginx 搭建[#](#2-nginx-搭建)
---------------------------

nginx 采用 docker 进行安装

### 拉取 nginx 镜像[#](#拉取-nginx-镜像)

```
docker pull nginx


```

### 使用 openssl 生成证书[#](#使用-openssl-生成证书)

1.  安装 openssl

```
yum -y install openssl openssl-devel


```

2.  生成 ca 秘钥

```
openssl genrsa -out local.key 2048


```

3.  生成 ca 证书请求

```
openssl req -new -key local.key -out local.csr


```

4.  生成 ca 根证书

```
openssl x509 -req -in local.csr -extensions v3_ca -signkey local.key -out local.crt


```

5.  根据 CA 证书创建 server 端证书

```
openssl genrsa -out my_server.key 2048 

openssl req -new -key my_server.key -out my_server.csr

openssl x509 -days 365 -req -in my_server.csr -extensions v3_req -CAkey local.key -CA local.crt -CAcreateserial -out my_server.crt


```

### 创建 nginx 挂载目录[#](#创建nginx挂载目录)

```
mkdir -p /nginx/config /nginx/config /nginx/logs /nginx/ssl


```

```
将刚才生成证书 `local.crt`, `local.key` 移动到 /nginx/ssl 目录下


```

### 创建 nginx 配置文件[#](#创建-nginx-配置文件)

在 /nginx/config 目录下创建 nginx.conf 文件

```
events {

    worker_connections  1024;

}



http{

upstream sekiro_business_netty {

  # sekiro 项目 地址

  server 127.0.0.1:5620;

}



server {

        listen       4431 default  ssl;  #监听4331端口

        keepalive_timeout 100;  #开启keepalive 激活keepalive长连接，减少客户端请求次数



        ssl_certificate      /ssl/local.crt;   #server端证书位置

        ssl_certificate_key  /ssl/local.key;   #server端私钥位置



        ssl_session_cache    shared:SSL:10m;         #缓存session会话

        ssl_session_timeout  10m;                    # session会话    10分钟过期



       ssl_ciphers  HIGH:!aNULL:!MD5;

       ssl_prefer_server_ciphers  on;



        server_name   test.com;

        charset utf-8;



        location /business-demo {    #后续获取加密参数时候用到的路由



          gzip on;

          gzip_min_length 1k;

          gzip_buffers 4 16k;

          gzip_http_version 1.0;

          gzip_comp_level 2;

          gzip_types application/json text/plain application/x-javascript text/css application/xml;

          gzip_vary on;



          proxy_read_timeout      500;

          proxy_connect_timeout   300;

          proxy_redirect          off;



          proxy_http_version 1.1;

          proxy_set_header    Upgrade $http_upgrade;

          proxy_set_header    Host                $http_host;

          proxy_set_header    X-Real-IP           $remote_addr;

          proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;

          proxy_set_header    X-Forwarded-Proto   $scheme;



          proxy_pass http://sekiro_business_netty;

        }

  

        location /register {      #后续页面或者手机进行wss连接用到的路由

          proxy_http_version 1.1;

          proxy_set_header Upgrade $http_upgrade;

          proxy_set_header Connection "Upgrade";

          proxy_set_header X-Real-IP $remote_addr;

          proxy_pass http://sekiro_business_netty;

        }



        location / {

          client_max_body_size 0;



          proxy_read_timeout      300;

          proxy_connect_timeout   300;

          proxy_redirect          off;



          proxy_http_version 1.1;



          proxy_set_header    Host                $http_host;

          proxy_set_header    X-Real-IP           $remote_addr;

          proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;

          proxy_set_header    X-Forwarded-Proto   $scheme;



          proxy_pass http://sekiro_business_netty;

        }

    }

}




```

### 启动 nginx[#](#启动-nginx)

```
docker run --detach --name nginx-test -p 4431:4431 -v /nginx/data:/usr/share/nginx/html:rw -v /nginx/config/nginx.conf:/etc/nginx/nginx.conf/:rw -v /nginx/logs:/var/log/nginx/:rw -v /nginx/ssl:/ssl/:rw -d nginx


```

3. JS RPC DEMO[#](#3-js-rpc-demo)
---------------------------------

```
function SekiroClient(wsURL) {

    this.wsURL = wsURL;

    this.handlers = {};

    this.socket = {};

    // check

    if (!wsURL) {

        throw new Error('wsURL can not be empty!!')

    }

    this.webSocketFactory = this.resolveWebSocketFactory();

    this.connect()

}

SekiroClient.prototype.resolveWebSocketFactory = function() {

    if (typeof window === 'object') {

        var theWebSocket = window.WebSocket ? window.WebSocket : window.MozWebSocket;

        return function(wsURL) {

            function WindowWebSocketWrapper(wsURL) {

                this.mSocket = new theWebSocket(wsURL);

            }

            WindowWebSocketWrapper.prototype.close = function() {

                this.mSocket.close();

            }

            ;

            WindowWebSocketWrapper.prototype.onmessage = function(onMessageFunction) {

                this.mSocket.onmessage = onMessageFunction;

            }

            ;

            WindowWebSocketWrapper.prototype.onopen = function(onOpenFunction) {

                this.mSocket.onopen = onOpenFunction;

            }

            ;

            WindowWebSocketWrapper.prototype.onclose = function(onCloseFunction) {

                this.mSocket.onclose = onCloseFunction;

            }

            ;

            WindowWebSocketWrapper.prototype.send = function(message) {

                this.mSocket.send(message);

            }

            ;

            return new WindowWebSocketWrapper(wsURL);

        }

    }

    if (typeof weex === 'object') {

        // this is weex env : https://weex.apache.org/zh/docs/modules/websockets.html

        try {

            console.log("test webSocket for weex");

            var ws = weex.requireModule('webSocket');

            console.log("find webSocket for weex:" + ws);

            return function(wsURL) {

                try {

                    ws.close();

                } catch (e) {}

                ws.WebSocket(wsURL, '');

                return ws;

            }

        } catch (e) {

            console.log(e);

            //ignore

        }

    }

    //TODO support ReactNative

    if (typeof WebSocket === 'object') {

        return function(wsURL) {

            return new theWebSocket(wsURL);

        }

    }

    throw new Error("the js environment do not support websocket");

}

;

SekiroClient.prototype.connect = function() {

    console.log('sekiro: begin of connect to wsURL: ' + this.wsURL);

    var _this = this;

    // if (this.socket && this.socket.readyState === 1) {

    //     this.socket.close();

    // }

    try {

        this.socket = this.webSocketFactory(this.wsURL);

    } catch (e) {

        console.log("sekiro: create connection failed,reconnect after 2s");

        setTimeout(function() {

            _this.connect()

        }, 2000)

    }

    this.socket.onmessage(function(event) {

        _this.handleSekiroRequest(event.data)

    });

    this.socket.onopen(function(event) {

        console.log('sekiro: open a sekiro client connection')

    });

    this.socket.onclose(function(event) {

        console.log('sekiro: disconnected ,reconnection after 2s');

        setTimeout(function() {

            _this.connect()

        }, 2000)

    });

}

;

SekiroClient.prototype.handleSekiroRequest = function(requestJson) {

    console.log("receive sekiro request: " + requestJson);

    var request = JSON.parse(requestJson);

    var seq = request['__sekiro_seq__'];

    if (!request['action']) {

        this.sendFailed(seq, 'need request param {action}');

        return

    }

    var action = request['action'];

    if (!this.handlers[action]) {

        this.sendFailed(seq, 'no action handler: ' + action + ' defined');

        return

    }

    var theHandler = this.handlers[action];

    var _this = this;

    try {

        theHandler(request, function(response) {

            try {

                _this.sendSuccess(seq, response)

            } catch (e) {

                _this.sendFailed(seq, "e:" + e);

            }

        }, function(errorMessage) {

            _this.sendFailed(seq, errorMessage)

        })

    } catch (e) {

        console.log("error: " + e);

        _this.sendFailed(seq, ":" + e);

    }

}

;

SekiroClient.prototype.sendSuccess = function(seq, response) {

    var responseJson;

    if (typeof response == 'string') {

        try {

            responseJson = JSON.parse(response);

        } catch (e) {

            responseJson = {};

            responseJson['data'] = response;

        }

    } else if (typeof response == 'object') {

        responseJson = response;

    } else {

        responseJson = {};

        responseJson['data'] = response;

    }

    if (Array.isArray(responseJson)) {

        responseJson = {

            data: responseJson,

            code: 0

        }

    }

    if (responseJson['code']) {

        responseJson['code'] = 0;

    } else if (responseJson['status']) {

        responseJson['status'] = 0;

    } else {

        responseJson['status'] = 0;

    }

    responseJson['__sekiro_seq__'] = seq;

    var responseText = JSON.stringify(responseJson);

    console.log("response :" + responseText);

    this.socket.send(responseText);

}

;

SekiroClient.prototype.sendFailed = function(seq, errorMessage) {

    if (typeof errorMessage != 'string') {

        errorMessage = JSON.stringify(errorMessage);

    }

    var responseJson = {};

    responseJson['message'] = errorMessage;

    responseJson['status'] = -1;

    responseJson['__sekiro_seq__'] = seq;

    var responseText = JSON.stringify(responseJson);

    console.log("sekiro: response :" + responseText);

    this.socket.send(responseText)

}

;

SekiroClient.prototype.registerAction = function(action, handler) {

    if (typeof action !== 'string') {

        throw new Error("an action must be string");

    }

    if (typeof handler !== 'function') {

        throw new Error("a handler must be function");

    }

    console.log("sekiro: register action: " + action);

    this.handlers[action] = handler;

    return this;

}

;



function guid() {

    function S4() {

        return (((1 + Math.random()) * 0x10000) | 0).toString(16).substring(1);

    }

    return (S4() + S4() + "-" + S4() + "-" + S4() + "-" + S4() + "-" + S4() + S4() + S4());

}





// group 为分组名称(调用时需要用到)

// clientId 客户端id

var client = new SekiroClient("wss://主机地址:4431/business-demo/register?group=xwgg&clientId=" + guid());



client.registerAction("get_sign", function(request, resolve, reject) {

    // 请求时 传递的参数

    var info = request['info']

    // 这里可以调用浏览器中的方法

    // var result = md5(info)

    var result = 'get_sign:' + info

    resolve(result)

})




```

4.  Python 调用 sekiro 进行 rpc

```
import requests



url = 'https://主机地址:4431/business-demo/invoke'



# group 分组名   action js中对于的注册的函数  info 具体传参

data = {"group": "xwgg", "action": "get_sign", "info": "我需要被签名"}



response = requests.get(url, params=data, verify=False)

print(response.text)


```