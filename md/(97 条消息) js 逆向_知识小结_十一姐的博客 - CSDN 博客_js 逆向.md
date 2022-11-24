> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43411585/article/details/117969306?spm=1001.2014.3001.5502#_domcanvas_dom_jquery_505)

### 目录

*   *   *   *   [一、Chrome 之调试小结](#Chrome_1)
            *   *   [① chrome 查看资源文件](#_chrome_3)
                *   [② chrome 关联本地文件夹](#_chrome_6)
                *   [③ chrome 重写 js 文件并替换](#_chromejs_9)
                *   [④ chrome 新建 js 文件并执行](#_chromejs_13)
                *   [⑤ Console 打印输出勾选](#_Console_16)
                *   [⑥ 断点（DOM、事件、xhr、debugger）](#_DOMxhrdebugger_20)
                *   [⑦ 调用栈 Call Stack](#_Call_Stack_30)
                *   [⑧ 清缓存、清 cookie](#_cookie_34)
                *   [⑨ 过无限 debugger](#_debugger_39)
                *   [⑩ hook 之 js](#_hookjs_79)
            *   [二、Fiddler 之抓包小结](#Fiddler_117)
            *   *   [① 模拟 postman 请求](#_postman_118)
                *   [② 重放再次请求](#__121)
                *   [③ 本地文件替换网页 js 文件](#_js_124)
                *   [④ 查看 tls 指纹](#_tls_130)
            *   [三、js 代码之知识点小结](#js_137)
            *   *   [① js 基础（数据类型、对象、数组）](#_js_140)
                *   [② 函数（构造、有名、匿名、自执行、arguments）](#_arguments_262)
                *   [③ 作用域（局部、全局、this、apply、call、bind）](#_thisapplycallbind_350)
                *   [④ 常见的函数方法 （escape、eval、slice、splice、charCodeAt、JSON.stringify、parseInt、 String.fromCharCode()）](#__escapeevalslicesplicecharCodeAtJSONstringifyparseInt_StringfromCharCode_435)
                *   [⑤ js 补环境介绍 BOM、DOM](#_jsBOMDOM_455)
                *   [⑥ js 原型与原型链](#_js_481)
                *   [⑦ 逻辑运算符与移位运算符](#__482)
            *   [四、浏览器指纹](#_483)
            *   *   [① 全局相关：window、document](#_windowdocument_486)
                *   [② 环境相关： navigator(包括经纬度在内都在这个接口里)，screen，history](#__navigatorscreenhistory_503)
                *   [③ 请求相关：XMLHttpRequest fetch worker](#_XMLHttpRequest__fetch_worker_504)
                *   [④ dom 相关：canvas ，所有对 dom 节点操作，包括 jquery 等三方库以及自设导入接口](#_domcanvas_dom_jquery_505)
                *   [⑤ 其他：caches WebGL AudioContext WebRTC](#_caches_WebGL_AudioContext__WebRTC_506)
                *   [⑤ 数据库相关： Storage IndexedDB cookie](#__Storage__IndexedDB__cookie_507)
            *   [五、node 之运行 js 小结](#nodejs_508)
            *   *   [① node 与 vscode 搭配模拟谷歌调试 js](#_nodevscodejs_509)
                *   [② aes、des、rsa、base64、md5、hash](#_aesdesrsabase64md5hash_510)
                *   [③ node 起服务运行 js](#_nodejs_511)
            *   [六、python 之执行 js 小结](#pythonjs_513)
            *   *   [① execjs、miniracer](#__execjsminiracer_514)
                *   [② md5、base64、aes、des、rsa](#__md5base64aesdesrsa_515)
                *   [③ requests 请求封装](#__requests_516)
                *   [④ soup、xpath](#__soupxpath_520)
                *   [⑤ excel、word、pdf、img](#_excelwordpdfimg_521)
                *   [⑥ mongodb、mysql](#__mongodbmysql_522)
                *   [⑦ 时间戳与日期的转换格式化](#__523)
                *   [⑧ urlencode、quote](#_urlencodequote_524)

#### 一、Chrome 之调试小结

*   `很早之前的笔记，先发出来，后面有很多没有完善起来`

##### ① chrome 查看资源文件

*   chrome 查看资源文件：Sources>Page  
    ![](https://img-blog.csdnimg.cn/7656f8e611fd411aa8dfdb0f9c52e034.png)

##### ② chrome 关联本地文件夹

*   chrome 关联本地文件夹：Sources>Filesystem>Add folder to workspace  
    ![](https://img-blog.csdnimg.cn/0eeace243b1a43edae3df17bf4a2629b.png)

##### ③ chrome 重写 js 文件并替换

*   `chrome重写js文件并替换`：第一步先添加文件夹：Sources>Overrides>Select folder for overrides；第二步去请求包 Network 页面选择 js 文件 > 右击 Open in Sources panel > 重新编辑 js 文件 > 刷新即可运行  
    ![](https://img-blog.csdnimg.cn/d6b15a9990234af08a2aee97a879e034.png)  
    ![](https://img-blog.csdnimg.cn/4cee8e1e41994ba4a24ab99b9bf19a19.png)

##### ④ chrome 新建 js 文件并执行

*   `chrome新建js文件并执行`：Sources>Snippets>New snippet；[Chrome 浏览器中执行 js](https://www.runoob.com/js/js-chrome.html)  
    ![](https://img-blog.csdnimg.cn/784aa14499544f9da03ad3876cc83d50.png)

##### ⑤ Console 打印输出勾选

*   Console 打印输出勾选：下面右侧设置齿轮，勾选如下 6 个，可以查看 xhr 请求的打印日志  
    ![](https://img-blog.csdnimg.cn/788734e9ac034a7c99ecb43cfdbb49cc.png)
    *   `如果页面多次打印某个函数`，可对该函数进行置空，如 console.log = function(){}

##### ⑥ 断点（DOM、事件、xhr、debugger）

*   断点：DOM 断点、DOM 事件下断点、xhr 断点、代码行断点 (自己下的 js 断点)、代码断点 (代码里的 debugger)、捕获异常断点
*   `DOM断点`：右击元素 attribute modifications 下断点（不推荐）  
    ![](https://img-blog.csdnimg.cn/442713bcc7134893b9ad1bcd7b30b441.png)
*   `DOM事件下断点`：Event Listeners、点击、Canvas、load 等事件断点  
    ![](https://img-blog.csdnimg.cn/d6d504db931c448bad874e209e299a3e.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70)![](https://img-blog.csdnimg.cn/52cac7d999564c1b91bd4da19e5af8e8.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70)
*   `xhr断点`：只针对 xhr 请求  
    ![](https://img-blog.csdnimg.cn/59b8389da510422480bac9d364068b9e.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70)
*   `捕获异常断点`：右侧暂停按钮，勾选 Pause on caught exceptions，可捕获全局断点  
    ![](https://img-blog.csdnimg.cn/4f70b00d54b74cae93eba40ea829cd7a.png)

##### ⑦ 调用栈 Call Stack

*   调用栈：Network 里面 Initiator 或者是断点调试 Source 面板下的 Call Stack  
    ![](https://img-blog.csdnimg.cn/06acfa5018a445ebb9080b3a8ce1eb99.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70)  
    ![](https://img-blog.csdnimg.cn/83472b0954db4bb7b4ed99ecc3ca2f28.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70)

##### ⑧ 清缓存、清 cookie

*   清缓存、清 cookie：Application 下 Clear site data 或者 cookie 的清除符号  
    ![](https://img-blog.csdnimg.cn/829acc75b9eb4ab4b248226b80475b0e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![](https://img-blog.csdnimg.cn/10fab16a55b84601bf50c66c50987a99.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)

##### ⑨ 过无限 debugger

*   `针对静态网页`：可以用 fiddler 替换网页 js 文件，并将其中的 debugger 删掉
*   `针对动态网页`：
    *   `常用方式过debugger`：鼠标右击选择 Never pause here  
        ![](https://img-blog.csdnimg.cn/cb9c947c45b44b878c84187decacc235.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)
    *   `Function原理的debugger`，可以重写函数构造器
        
        ```
        Function.prototype.constructor_bc = Function.prototype.constructor
        Function.prototype.constructor = function() {
            if (arguments[0] === "debugger") {} 
            else {
                Function.prototype.constructor_bc.apply(this, arguments)
            }
        }
        ```
        
    *   `eval类型的debugger`，可以重构 eval 函数
        
        ```
        eval_bc = eval
        eval = function(a) {
            if (a === '(function() {var a = new Date(); debugger; return new Date() - a > 100;}())') {} 
            else {
                return eval_bc(a)
            }
        }
        ```
        
    *   `(function() {var a = new Date(); debugger; return new Date() - a > 100;}())`：定时器的无限 debugger，可用上面的方法，或者以下两种方法  
        ![](https://img-blog.csdnimg.cn/d39f213f28ca4516b582fde8033f8223.png)
        
        ```
        setinval_b= setInterval
        setInterval = function(a, b) {
            if (a.toString().indexOf('debugger') == -1) {
            	console.log(a);
                return setinval_b(a, b)
            }
        }
        ```
        
        ```
        for (var i = 1; i < 99999; i++)window.clearInterval(i);
        ```
        
    *   `向上找堆栈，在进入无限debugger之前打上断点将触发无限debugger的函数置空`

##### ⑩ hook 之 js

*   `hook之window的属性`
    
    ```
    (function() {
        'use strict';
        var pre = "";
        Object.defineProperty(window, '_pt_', {
            get: function() {
                console.log('Getting window.属性');
                return pre
            },
            set: function(val) {
                console.log('Setting window.属性', val);
                debugger ;
                pre = val;
            }
        })
    })();
    ```
    
*   `hook之cookie`
    
    ```
    (function() {
       'use strict';
        var _cookie = ""; // hook cookie
        Object.defineProperty(document, 'cookie', {
            set: function(val) {
                console.log('cookie set->', new Date().getTime(), val);
                debugger;
                _cookie = val;
                return val;
            },
            get: function(val) {
                return _cookie;
            }
       });
    })()
    ```
    

#### 二、[Fiddler](https://so.csdn.net/so/search?q=Fiddler&spm=1001.2101.3001.7020) 之抓包小结

##### ① 模拟 postman 请求

*   `模拟请求`，复制请求头 raw 的数据在 Composer 页面 raw 可以重新执行 Execute，类似于 postman 工具  
    ![](https://img-blog.csdnimg.cn/20210616230626826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70)

##### ② 重放再次请求

*   `重放再次请求`，响应请求右击 Replay>Reissue Sequentially 或者直接点上方 Replay；即进行重新请求，可选择多个请求同时操作，重放攻击如果成功则模拟请求，失败则找原因  
    ![](https://img-blog.csdnimg.cn/43c19a18be3b46fcb21f253465a366cd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_17,color_FFFFFF,t_70,g_se,x_16)

##### ③ 本地文件替换网页 js 文件

*   `通过Fiddler用本地js文件替换源网页的js文件` ，将网页端的 js 文件保存到本地，并修改 js 内容里面想要改的地方，比如保存为 ticket.js
*   打开 Fiddler 选择 AutoResponder 进行文件替换
*   刷新一下网页，查看 js 文件是否已经被替换成你想替换的内容，并做相应的测试
*   刷新后 fiddler 替换 js 文件如果失败，可如下操作，网页端谷歌开发者工具勾选 Network>Disable cache 或 js 文件与网页一致取压缩状态的文件  
    ![](https://img-blog.csdnimg.cn/522e03bd841c48f4a2f4b27c93c52745.png?x-oss-process=image,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)

##### ④ 查看 tls 指纹

*   可以看到 Ciphers 为 tls 指纹的 key，
*   [复制每个 TLS 分别去该网找对应的 value，将找到的 value 用冒号拼接起来](https://www.openssl.org/docs/man1.1.1/man1/openssl-ciphers.html)，然后赋值给 requests.packages.urllib3.util.ssl_.DEFAULT_CIPHERS 即可，可以应对 tls 指纹的网站  
    ![](https://img-blog.csdnimg.cn/101cf66bed9849669a7f13a616336a8d.png)

![](https://img-blog.csdnimg.cn/2cf43e18e8c94155ad58aab1a4e8256d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5Y2B5LiAKFNocmltYXkxKQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 三、js 代码之知识点小结

*   [推荐教程](https://www.runoob.com/js/js-htmldom-events.html)
*   [推荐文章](https://blog.csdn.net/qq_33254766/article/details/107894664)

##### ① js 基础（数据类型、对象、数组）

*   （1）js 的使用：内部 js 放在 <script></script > 标签中；外部 js 通过 src 属性引入：<script src=“javascript.js”></script>；onclick、href 等属性绑定执行 js  
    ![](https://img-blog.csdnimg.cn/20210701083202167.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70)
    
*   （2）数据类型：基本数据类型：`字符串（String）、数字(Number)、布尔(Boolean)、空（Null）、未定义（Undefined`）；引用数据类型：`对象(Object)、数组(Array)、函数(Function)`，可通过 typeof() 检查数据类型  
    ![](https://img-blog.csdnimg.cn/20210701083833919.png)
    
*   （3）当你通过 var 声明变量时，可以使用关键词 “new” 来声明其类型
    
    ```
    var carname=new String;
    var x=      new Number;
    var y=      new Boolean;
    var cars=   new Array;
    var person= new Object;
    ```
    
*   （4）变量声明提前，使用 var 关键字声明的变量，会在所有代码执行之前被声明 (但不会被赋值)，但是如果声明变量不使用 var 关键字，则变量不会被声明提前
    
*   （5）在函数中，不使用 var 声明的变量都会成为全局变量
    
*   （6）对象 Object：对象属性名可加双引号可不加双引号，只能是字符串，值可以是任意的数据类型
    
    *   `内建对象`：由 ES 标准中定义的对象，在任何的 ES 的实现中都可以使用，比如：Math，String，Number，Boolean，Function，Object
    *   `宿主对象`：由 JS 运行环境提供的对象，目前来讲主要是浏览器提供的对象，比如 BOM，DOM
    *   `自定义对象`：由开发人员自定创建的对象
    
    ```
    // 定义一个对象
    let obj = {hello: 'world'};
    // 浅拷贝，只复制对象的内存地址，类似于指针，只要其中之一改变，则另一个也随之改变
    let obj2 = obj;
    // 深拷贝，完全克隆，生成一个新对象
    let obj3 = JSON.parse(JSON.stringify(obj));
    // 定义
    let me = {
        name: 'shir',
        age: "18",
        fullname: function() {
            return
        }
    };
    // 赋值也可以取值
    me.education = 'College'
    me['sex'] = '女'
    me[love] = 'flower'
    // 取值
    console.log(Object.keys(me))  // ["name", "age", "education", "sex", "love"]
    console.log(Object.values(me))  // ["shir", "18", "College", "女", "flower"]
    // 删除属性
    delete me.education
    // 遍历
    for (let key in me) {
        if (me.hasOwnProperty(key)){
            const value = me[key]
            console.log(key, value)  // name shir  age 18   
        }
    }
    ```
    
*   （7）`数组Array`：值可以是任意的数据类型，map() 和 forEach() 方法遍历数组
    
    *   创建一个数组的三种方式:
        
        ```
        // 常规方式
        var myCars=new Array();
        myCars[0]="Saab";      
        myCars[1]="Volvo";
        myCars[2]="BMW";
        // 简洁方式
        var myCars=new Array("Saab","Volvo","BMW");
        // 字面方式
        var myCars=["Saab","Volvo","BMW"];
        ```
        
    *   数组的长度`.length`，数组的追加`push()`，数组的尾部删除`pop()`，数组的头部增加`unshift()`，数组的头部删除`shift()`，数组排序`sort()`，数组逆序排序`reverse()`
        
        ```
        let items = [1, 2, 3, 4, 5]
        console.log(items[2]);   // 3
        items[3] = "4shir";   // 第4个位置赋值
        items.length;   // 5
        items.indexOf(5);  //  读索引值
        items.push(6);  // 相当于python的append()，追加一个元素
        items.pop();  // 删除最后一个元素
        items.unshift("Lemon","Pineapple") // 头部增加第一个元素
        items.shift(); // 删除第一个元素
        items.sort();  // 正序排序
        items.reverse();  // 反序排序
        ```
        
    *   数组的插入`splice(index, 0, 元素)`与删除`splice(index,个数)`
        
        ```
        let items = [1, 2, 3, 4, 5]
        // 指定索引添加或删除
        items.splice(
            0, // 指定索引添加
            0, // 不删除即为添加
            "0", // 添加的元素值
        );
        items.splice(
            0, // 指定索引删除
            2, // 删除多少个
        );
        ```
        
    *   数组的遍历`.map()`与`.forEach()`
        
        ```
        //数组遍历方式1，map会返回新数组
        var items = [1,2,3,4,5];
        var result = items.map(function( value ) {
            return value * 2;
        });
        console.log(result);
        	
        // 数组遍历方式2，forEach不会有新数组返回，也没有break和continue，可通过some和every实现
        var arr = [1, 2, 3, 4, 5];
        arr.forEach(function (item) {
            if (item === 3) {
                return;  // return语句实现continue 关键字的效果：
            }
            console.log(item);
        });
        ```
        
    *   数组的合并`concat()`
        
        ```
        var parents = ["Jani", "Tove"];
        var brothers = ["Stale", "Kai Jim", "Borge"];
        var children = ["Cecilie", "Lone"];
        var family = parents.concat(brothers, children); //["Jani", "Tove", "Stale", "Kai Jim", "Borge", "Cecilie", "Lone"]
        ```
        
*   （8）逗号运算符使用逗号`,` ，可以分割多个语句，一般在声明多个变量时使用
    
*   （9）在 JS 中可以`使用{}来为语句进行分组`，一个 {} 中的语句我们称为一组语句或代码块，它们要么都执行，要么都不执行，在代码块后边就不用再写分号; 了
    
*   （10）三元表达式：`条件表达式?语句1：语句2`  
    ![](https://img-blog.csdnimg.cn/20210701232208140.png)
    

##### ② 函数（构造、有名、匿名、自执行、arguments）

*   （1）函数定义：
    
    *   声明式函数定义：function mytest(){console.log(‘haha’)}
    *   函数表达式定义：let fun = function(){console.log(‘haha’)}
    *   new Function 形式：var fun1 = new Function (arg1 , arg2 ,arg3 ,…, argN , body)
*   （2）`构造函数`：使用 new 关键字调用的函数，最后一个位置为函数执行体，前面的位置可加形参
    
    ```
    let sum = new Function('a', 'b', 'return a + b');
    sum(1, 2)
    
    let sayHi = new Function('console.log("Hello")');
    sayHi()
    
    function Person(){
        this.;
        this.age=18;
    }
    var p1=new Person();
    console.log(p1, p1.name,p1.age);
    ```
    
*   （3）`有名函数`：function 关键字后面指定名字的，如下案例，指定函数名 myNameFun
    
    ```
    function myNameFun(a,b){
        console.log(a+b)
    }
    myNameFun(1, 1)
    ```
    
*   （4）`匿名函数`：function 关键字后面没有名字的
    
    ```
    // 匿名函数格式
    function(a,b){
        console.log(a+b)
    }
    ```
    
*   （5）`自执行函数`：顾名思义，就是立即自动执行的函数，`有名函数和匿名函数都可以自执行`，举例几个匿名函数自执行的方式如下；因为 js 是函数作用域，所以如果`想实现某个功能又不想污染全局变量的时候可以使用自执行函数`；**样例见某盾 core.js 一个典型的实际用例，逐步调试看看就懂了**
    
    *   `自执行方式1：变量=匿名函数(传入参数)`，把匿名函数赋给一个变量自执行，
        
        ```
        // 自执行方式1
        var myNamefun = function(a,b){
            console.log(a+b)
        }(1,1)
        ```
        
    *   `自执行方式2：(匿名函数)(传入参数)`，样例如下
        
        ```
        // 自执行方式2
        (function(a,b){
            console.log(a+b)
        })(1,1);
        ```
        
    *   `自执行方式3：(匿名函数(传入参数))`，样例如下
        
        ```
        (function(a,b){
            console.log(a+b)
        }(1,1));
        ```
        
    *   [更多自执行方式详见](https://blog.csdn.net/weixin_42703239/article/details/88371031)
        
        ```
        !function () { /* code */ } ();  
        !(function () { /* code */ } )();  
        ~function () { /* code */ } ();  
        -function () { /* code */ } ();  
        +function () { /* code */ } ();
        ```
        
*   （6）`箭头函数`：(参数 1, 参数 2, …, 参数 N) => 表达式 (单一)
    
    ```
    // ES5
    var x = function(x, y) {
         return x * y;
    }
    // ES6
    const x = (x, y) => x * y;
    ```
    
*   （7）函数的参数：显式参数 (Parameters) 与隐式参数(Arguments)，`JavaScript 函数有个内置的对象 arguments 对象`，argument 对象包含了函数调用的参数数组，arguments 是个类数组对象； `在调用函数时浏览器每次都会传递两个隐含的参数：函数的上下文对象this、封装实参的对象arguments`
    
    ```
    funciton test(){
    	var m = arguments[0]; //第一个传入的参数
    	var n = arguments[1]; //第二个传入的参数，如果只传入一个参数，那么该值为undefined
    	var l = argument.length;//该值对应的函数调用时传入参数的个数
    	var ll = fun.length;//该值对应的函数定义的参数个数
    }
    ```
    
*   （8）函数的特征熟悉
    
    *   函数声明提前，使用 function 声明创建的函数，function 函数 (){}，会在所有代码执行前创建，可以在函数声明前提前调用；使用函数表达式即定义变量的形式创建的函数，不会被提前创建
    *   函数也可以作为对象的属性值
    *   函数的实参可以是任意的数据类型，可以是对象，也可以是函数
    *   函数的返回值可以是任意数据类型，可以是对象，也可以是函数
    *   调用函数时创建函数作用域，调用结束就销毁，在函数中要访问全局变量，可以使用 window. 变量

##### ③ 作用域（局部、全局、this、apply、call、bind）

*   （1）作用域：在 JS 中一共有两种作用域，`一种是全局作用域，一种是函数作用域`
    
*   （2）`局部作用域`：在函数内通过 var 声明的变量，变量为局部变量，所以只能在函数内部访问它，该变量的作用域是局部的；
    
*   （3）`全局作用域`：在函数外通过 var 声明的变量为全部变量，不使用 var 声明的变量默认都是全局变量
    
    *   直接编写在 script 标签中的 Js 代码，都在全局作用域
    *   在全局作用域中有一个**全局对象 window**，它代表的是一个浏览器的窗口，它由浏览器创建，我们可以直接使用
    *   在全局作用域中创建的变量都会作为 window 对象的属性保存
    *   在全局作用域中创建的函数都会作为 window 对象的方法保存
*   （4）`this`：表示当前对象的一个引用，不是固定不变的，它会随着执行环境的改变而改变，`换言之当前谁是this的拥有者，this就指向谁`
    
    *   全局作用域中，this 指向全局对象 window
    *   以方法的形式调用时，this 是调用方法的对象
    *   以函数形式调用时，this 永远都是 window
    *   以构造函数形式调用时，this 是新创建的那个对象
    *   使用 call 和 apply 和 bind 调用是，this 就是指定的那个对象
    *   `全局中的this`：全局作用域中，this 指向全局对象 window
        
        ```
        //在全局作用域中声明变量num
        var num=1;
        console.log(window.num);//1
        //修改this对象身上的num属性,将num属性的值修改为2
        this.num=2;
        console.log(window.num);//2
        console.log(this===window);//true
        ```
        
    *   `方法中的this`：this 是调用方法的对象；在方法中，this 指向该方法所属的对象，如下 this 指向了 p1 对象，因为 p1 对象是 speak 方法的所有者
        
        ```
        var p1={
            name:"11",
            speak:function(){
                console.log("我是"+this.name);  // 我是11
                console.log(this===p1);//true
            }
        }
        p1.speak();
        ```
        
    *   `普通函数中的this`：this 永远都是 window；函数的所属者默认绑定到 this 上，即谁拥有这个函数，即指向了 window，因为在全局作用域中创建的函数都会作为 window 对象的方法保存
        
        ```
        function Person(){
           this.;
           console.log(this===window); // true
        }
        Person();//实际上相当于window.Person()
        ```
        
        ```
        var p1={
            friend:"小明",
            speak:function(){
                console.log(this===p1);//false
                console.log(this===window);//true
                console.log("我的朋友是"+this.friend);
                //我的朋友是undefined,因为此时this指向window,
                //window身上并没有friend这个属性,所以读取结果是undefined
                //speak2属于全局变量，全局变量作为window对象属性保存，所以指向window
            }
        }
        
        var speak2=p1.speak;
        speak2();//此处相当于window.speak2()
        ```
        
    *   `构造函数中的this`：this 是新创建的那个对象  
        ![](https://img-blog.csdnimg.cn/20210703085148833.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70)
*   （5）`call()、apply()、bind()：可以强制改变函数内部this的指向`，this 就是指定的那个对象；执行结果一致，如下案例 this 本来应该指向 obj 对象，但是最终指向了了 obj2 对象
    
    *   call 、bind 、 apply 这三个函数的第一个参数都是 this 的指向对象，`差别在于第二个以及后面的参数`
        
    *   call 的参数是直接放进去的，`第二第三第 n 个参数全都用逗号分隔`，直接放到后面 obj.func.call(obj2,‘成都’, … ,’）
        
    *   apply 的所有参数都必须`放在一个数组里面传进去`obj.func.apply(obj2,[‘成都’, …,])
        
    *   bind 除了**返回是函数以外，它的参数和 call 一样**
        
        ```
        let obj = {
            name:'张三',
            age:18,
            func:function(fm, t){
                console.log(this.name + "年龄" + this.age + "来自" + fm + "去向" + t);
            }
        }
        let obj2 = {
            name:'李四',
            age:20
        }
        obj.func.call(obj2, '成都', '上海') //李四年龄20来自成都去向上海
        obj.func.apply(obj2, ['成都', '上海']); //李四年龄20来自成都去向上海
        obj.func.bind(obj2, '成都', '上海')(); //李四年龄20来自成都去向上海
        ```
        
*   （6） [关于 JS 中作用域的销毁和不销毁的情况总结](https://www.cnblogs.com/qinmengjiao123-123/p/5219424.html)
    

##### ④ 常见的函数方法 （escape、eval、slice、splice、charCodeAt、JSON.stringify、parseInt、 String.fromCharCode()）

*   （1）`escape()`：对普通字符串编码。`unescape()`：对字符串普通解码。`encodeURIComponent()`：对 URL 进行编码，编码范围大于`encodeURI()`。`decodeURIComponent()`：对 URL 进行解码，解码范围大于`decodeURI()`  
    ![](https://img-blog.csdnimg.cn/20210703152947151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70)
    
*   （2）`eval()`：用来执行一个字符串表达式，并返回表达式的值  
    ![](https://img-blog.csdnimg.cn/20210703153719862.png)
    
*   （3）`JSON.stringify()`：类似 python 的 json.dumps()，将 json 对象转换成 json 字符串。 `JSON.parse()`：类似 python 的 json.loads()，将 json 字符串转换成 json 对象  
    ![](https://img-blog.csdnimg.cn/20210701221609372.png)
    
*   （4）`slice()`：不会改变原始数组，截取某段索引范围的数据 。`splice()`：指定数组某个索引值是添加元素还是删除多少个元素，第二个位置参数数字代表删除多少个。`substr()`： 的参数指定的是子串的开始位置和长度  
    ![](https://img-blog.csdnimg.cn/20210701230712254.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70)  
    ![](https://img-blog.csdnimg.cn/c6afac68eefb463c9419c515b2077b8c.png)
    
*   （5）`parseInt()`：其中之一作用是将字符串中有效的整数取出来转换为 Number 类型  
    ![](https://img-blog.csdnimg.cn/20210701222221831.png)
    
*   （6）`charCodeAt()`：返回指定位置的字符的 Unicode 编码，`String.fromCharCode()`：返回 Unicode 编码对应的字符，`charAt()`：可返回指定位置的字符  
    ![](https://img-blog.csdnimg.cn/294470151e774f859d495ddf54cc17a6.png)
    
*   (7) Math.random，Math.ceil，Math.floor，Math.abs，Math.log
    

##### ⑤ js 补环境介绍 BOM、DOM

*   如 window、navigator、location、document 脱离浏览器、在外部不能直接调用，当你从全局复制 js，则需要补齐环境
    
*   （1）`window`是一个全局变量，由浏览器提供的，一般缺的是浏览器环境  
    ![](https://img-blog.csdnimg.cn/01e9b7c706b14816919dfdddc7f629cd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   （2）`navigator`：navigator.plugins  
    ![](https://img-blog.csdnimg.cn/80736d4719074f6ca09b4da1f2787c03.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)![](https://img-blog.csdnimg.cn/e6cf64c34e6c4df7b59b8ceea7900052.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   （3）`location`  
    ![](https://img-blog.csdnimg.cn/bc9bdfca0a6942dd99919866ef234108.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   （4）`document`
    
    ```
    document = {
        write:function(){}
    }
    document.write.toString = function(){return "function write() { [native code] }"}
    ```
    
    ![](https://img-blog.csdnimg.cn/6eee9cce62c54ec0aab95204d8c2bf66.png)
    
*   （5）[html 事件](https://www.runoob.com/js/js-htmldom-events.html)
    
    *   当用户点击鼠标时：登录
    *   当网页已加载时：浏览器指纹、收集是否是浏览器环境
    *   当图像已加载时：滑块图片还原、canvas
    *   当鼠标移动到元素上时：浏览器指纹、无感验证

##### ⑥ js 原型与原型链

##### ⑦ 逻辑运算符与移位运算符

#### 四、浏览器指纹

*   `WebApI`: [详细](https://developer.mozilla.org/zh-CN/docs/Web/API)
*   `nodejsAPI`: [详细](http://nodejs.cn/api/path.html)

##### ① 全局相关：window、document

*   [window](https://developer.mozilla.org/zh-CN/docs/Web/API/Window)
    *   window.XMLHttpRequest
    *   `window.ActiveXObject`：用来判断浏览器是否支持 ActiveX 控件，只有 IE 才能装 ActiveX 插件，所以通过此方法判断是不是 ie 浏览器
    *   `window.msCrypto`：IE11 的 Web Crypto 位于 window.msCrypto 内部, 而对于 Firefox 或 Chrome, 它可以在 window.crypto 中访问，Web Crypto 提供了一些加密
    *   `window.addEventListener(event, function, useCapture)`：事件监听函数，第一个参数是事件的类型 (如 “click” 或 “mousedown”)，第二个参数是事件触发后调用的函数，第三个参数是个布尔值用于描述事件是冒泡（false，先内后外）还是捕获，该参数是可选的
    *   `window.removeEventListener(event, function)`：移除事件监听
    *   `window.localStorage`：[localStorage 不能被爬虫抓取到](https://www.runoob.com/jsref/prop-win-localstorage.html)
    *   `window.unload`：
    *   window.top.location
    *   window.setTimeout
    *   window.setInterval
    *   window.eval
    *   escape, Number
    *   decodeURIComponent
*   [document](https://developer.mozilla.org/zh-CN/docs/Web/API/Document)
    *   `document.characterSet`和`document.charset`：返回当前文档的字符编码

##### ② 环境相关： navigator(包括经纬度在内都在这个接口里)，screen，history

##### ③ 请求相关：XMLHttpRequest fetch worker

##### ④ dom 相关：canvas ，所有对 dom 节点操作，包括 jquery 等三方库以及自设导入接口

##### ⑤ 其他：caches WebGL AudioContext WebRTC

##### ⑤ 数据库相关： Storage IndexedDB cookie

#### 五、node 之运行 js 小结

##### ① node 与 vscode 搭配模拟谷歌调试 js

##### ② aes、des、rsa、base64、md5、hash

##### ③ node 起服务运行 js

#### 六、python 之执行 js 小结

##### ① execjs、miniracer

##### ② md5、base64、aes、des、rsa

##### ③ requests 请求封装

*   `requests基础代码`：至少要包含请求头、timeout，resp.json()
*   `请求头保持顺序不变`：用 session.headers=headers 可以防止请求头的顺序乱掉，而 requests 请求（由于 python 字典特性）会让 headers 的请求头乱
*   `响应请求头快速转换`，右击开发者工具，[copy all as cURL, 复制到转换为 python 代码](http://tool.yuanrenxue.com/curl)

##### ④ soup、xpath

##### ⑤ excel、word、pdf、img

##### ⑥ mongodb、mysql

##### ⑦ 时间戳与日期的转换格式化

##### ⑧ urlencode、quote