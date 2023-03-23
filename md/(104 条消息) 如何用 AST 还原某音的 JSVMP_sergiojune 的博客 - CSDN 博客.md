> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/sergiojune/article/details/129279613)

##### 1. 什么是 JSVMP

vmp 简单来说就是将一些高级语言的代码通过自己实现的编译器进行编译得到字节码，这样就可以更有效的保护原有代码，而 jsvmp 自然就是对 JS 代码的编译保护，具体的可以看看 [H5 应用加固防破解 - JS 虚拟机保护方案](https://blog.ntan520.com/article/11179)。

如何区分是不是 jsvmp？看代码里面有没有很长的一串字符串或者是数组，并且代码里面没有具体的逻辑，还有就是有一个循环不断地在跑，里面的就是一些指令，比如 x 乎，x 音，x 讯滑块等等都是 jsvmp，也都是基于堆栈的栈式虚拟机实现的。

这是某乎的：

![](https://img-blog.csdnimg.cn/b58f38c729ea4988b2b904c91c399a49.png#pic_center)

这是某音的，还原了一些算术混淆、[三目运算](https://so.csdn.net/so/search?q=%E4%B8%89%E7%9B%AE%E8%BF%90%E7%AE%97&spm=1001.2101.3001.7020)和 if else 控制流混淆的：

![](https://img-blog.csdnimg.cn/2afec443042444608e9f95b393c1a033.png#pic_center)

![](https://img-blog.csdnimg.cn/6f07cc5ac82e4cf58aa1191f71c20e54.png#pic_center)

这是某讯滑块的：

![](https://img-blog.csdnimg.cn/8e376576d78f4f55a718d9279cde6500.png#pic_center)

##### 2. 如何去还原 JSVMP

1.  调试 vmp 代码，分析循环代码里面每个操作数对应的代码的意义
    
2.  修改源代码中的 vmp [解释器](https://so.csdn.net/so/search?q=%E8%A7%A3%E9%87%8A%E5%99%A8&spm=1001.2101.3001.7020)，添加 AST 的对应代码
    
3.  运行代码，生成最终代码。
    

##### 3. 分析具体网站

样品网站：aHR0cHM6Ly90cmVuZGluc2lnaHQub2NlYW5lbmdpbmUuY29tL2FyaXRobWV0aWMtaW5kZXgvYW5hbHlzaXM/a2V5d29yZD0lRTklODAlODYlRTUlOTAlOTEmc291cmNlPW9jZWFuZW5naW5lJmFwcE5hbWU9YXdlbWU=

这个网站要找的是 **X-Bogus** 和 **_signature** 这两个参数的生成，怎么定位的这里不说了，

很容易就可以看到这个函数就是在通过操作码去运行逻辑

![](https://img-blog.csdnimg.cn/e88b793db20040e6a9a8ca66bdf37e44.png#pic_center)

所以我们要分析下这个函数，看到有算术混淆和一些逗号混淆，不方便调试，可以使用 AST 写个逻辑去掉 这个很简单，还原效果如下：

![](https://img-blog.csdnimg.cn/89ffad4d996844c4b6075ad49f773abc.png#pic_center)

可以看到，逻辑清晰了很多，但是因为是 if else 混淆，里面还有一些中间变量，如果调试的话会需要点击多几次才能到目标语句，所以我们可以考虑将这个 if else 转换为 switch 语句，这样调试的时候直接一步到位。

![](https://img-blog.csdnimg.cn/eaac2e44a40a4eb3ad048fa32ff9d8c5.png#pic_center)

上图可以看到每次都从操作码中读取两个字符，并且是十六进制的，所以 **_0x1383c7** 这个变量的范围就是 0-255，这样 switch 的条件就有了，可以编写代码从 0 迭代到 255，就可以得到每个指令对应的代码了。最后还原结果如下：

![](https://img-blog.csdnimg.cn/1ffb7d731a134f609d90b2a93cfd6ca3.png#pic_center)

可以看到逻辑很清晰了，但是有个问题是 0-255 的 case 里面有些是重复的，需要删除，可以继续使用 AST，在每个 case 的第一行插入 **june.push(case_num)**，如这样：

![](https://img-blog.csdnimg.cn/eabc81ccaea948398d88a0744fa40b7f.png#pic_center)

然后映射文件到网站上，运行一次，获取 june 数组里面的值，保存到代码里，继续编写 AST 代码，读取 case 条件值，将不在数组中存在的 case 值全部删除掉，最后就可以愉快地分析了！

通过调试分析出，该函数的每个参数意义如下：

<table><thead><tr><th align="left">参数</th><th>意义</th></tr></thead><tbody><tr><td align="left">_0x307ee4</td><td>字节码</td></tr><tr><td align="left">_0x5b7220</td><td>函数起始位置</td></tr><tr><td align="left">_0x237dce</td><td>函数长度</td></tr><tr><td align="left">_0x3f8a47</td><td>不知道干啥，不影响分析</td></tr><tr><td align="left">_0x4372f0</td><td>内外部变量区，存储了当前函数的参数、定义的变量和外部变量</td></tr><tr><td align="left">_0x264fdc</td><td>函数调用对象</td></tr><tr><td align="left">_0x1f863f</td><td>无意义</td></tr><tr><td align="left">_0xb80186</td><td>vmp 分支，混淆分析的</td></tr></tbody></table>

不过我们只需要知道哪个参数是存储变量的，和栈变量即可，其他的不影响分析。

我们要想还原出逻辑，就要找到对变量的定义，赋值，然后将其转为对应的语句即可。

调试代码：

![](https://img-blog.csdnimg.cn/179437d465e641e4a04317ccbe421b44.png#pic_center)

![](https://img-blog.csdnimg.cn/50433078fd9c4609bc201ab89afde466.png#pic_center)

这个就是在获取变量区 **$0 **的变量，**$0 **代表的是本函数内定义的变量，**$1 **就代表上一层，**$2** 就继续往上推等等，但是其最外面的也会是本函数内的变量，所以需要需注意 (这个如果有误，望赐教)

继续跑下去，可以看到

![](https://img-blog.csdnimg.cn/acfd3bb12af0463dba511fa817b70c14.png#pic_center)

![](https://img-blog.csdnimg.cn/fbdf0d5bc5ca496ea0530f942cf7ab61.png#pic_center)

他将刚在获取的变量区 index 为 24 的值 push 进了栈，是一个函数参数，继续往下看

![](https://img-blog.csdnimg.cn/e379a4f719a340f0a15b14a77c23ac89.png#pic_center)

通过分析可知，**_0x9ac2c2** 为栈，此处是将字符串 **dfp** 入栈，而我们要还原语句的话，入栈就不能这个了，需要改为对应的 ast 语句，将字符串 **dfp** 直接放上这网站 [astexplorer](https://astexplorer.net/)

![](https://img-blog.csdnimg.cn/ac1dd6237d364557a566ffa0942aadcd.png#pic_center)

可以看到对应的 ast 结构，所以此处代码改为：

![](https://img-blog.csdnimg.cn/957ca84b68ae4caeb439ea023c0248df.png#pic_center)

继续往下调试：

![](https://img-blog.csdnimg.cn/60cd96c6cccc4cc7b727bd90113bbd97.png#pic_center)

这里是将刚才两个入栈的数据进行属性获取，对应语句为这样： yyy[“dfp”] (这里 yyy 为乱写的，方便大家看的)

所以我们需要将源代码改为对应的 ast 代码：

![](https://img-blog.csdnimg.cn/d33202af4fd44f2e96e3cb9ea492984d.png#pic_center)

![](https://img-blog.csdnimg.cn/df144b299f9b425f9d0979b18a179ced.png#pic_center)

接下来继续遇到一些常量进栈的，和上面的做法一样的，就不继续说了，说下变量定义：

![](https://img-blog.csdnimg.cn/2abf962cc5db4ee8b1f3b1b49a019e26.png#pic_center)

上面这个就是变量赋值，这里面获取的是变量区的值，如下：

![](https://img-blog.csdnimg.cn/c0241267b9fa492ba4d2befb0bfc2f21.png#pic_center)

![](https://img-blog.csdnimg.cn/eff8ad620bcc432f9d4fd7113a284c3a.png#pic_center)

可以看到 index 26 的位置还没有，所以可以判断这是一个新变量，第一次赋值，所以语句为这个：var a = “xxx”;

![](https://img-blog.csdnimg.cn/43876bb820c84cde8caa4e9ab7f58131.png#pic_center)

如果当前这个 “26” 已经存在的话，就不需要定义变量了，而是赋值就行，语句是这样：a = “xxx”;

![](https://img-blog.csdnimg.cn/cba28f41623a4f3e92c923710d9e1401.png#pic_center)

所以我们需要判断下这个语句是否存在，修改代码如下：

![](https://img-blog.csdnimg.cn/4e4a4e1d086a4384a47e0b691b21c327.png#pic_center)

可以看到入栈的数据是一个 **Identifier** 类型的变量，这个就是变量名了，之后代码需要操作的话，就获取这个变量就行，而不是获取直接的值。

最后记得将盖语句 push 进 body，因为赋值就是一个完整的语句了，需要还原成代码。

理解了上面的剩下的就没什么问题了，就可以边调试边改对应的 case，有些 case 是不需要操作的，比如上面的第一步获取变量区 **$0**

**小技巧**：在每一个 case 前面添加一个语句：throw Error(“未更改”)；然后直接运行代码，抛出异常的就是未处理的 case，然后在浏览器调试看该 case 的作用并修改代码即可，运行到没有异常就代表没啥问题了！

另外说下 if else 和 for 这种语句怎么还原：

![](https://img-blog.csdnimg.cn/ca9f14aea5c04600926839bb8c1ae666.png#pic_center)

上面这个就是进行判断的，可以看到 **_0x9ac2c2[_0x47144c–] **的结果不一样，**_0x1383c7** 就不一样，即为下一个运行的指令不一样

我的处理思路：

1.  进入这个分支就直接生成 **if** 判断，
    
    2.  然后保存当前栈，当前栈指针，以及当前函数所有参数变量的值，和当前指令位置 **_0x1383c7**，  
        3. 将 else 分支的 **_0x1383c7** 作为 if 分支的结束值，因为当 if 分支的指令值大于或者等于 else 分支的指令值的时候 证明两个分支运行的语句已经一致了 (不过如果遇到三目语句，这种情况又需要另外讨论了)

然后直接调用当前函数，返回语句后将 if 结束后返回的指令值 **_0x1383c7** 作为 else 语句的结束指令，最后用 else 分支保存的函数进行调用

![](https://img-blog.csdnimg.cn/0a4e49211f6b431188c39f13472d445b.png#pic_center)

最后就是判断循环了，循环的也会进入上面这个 case，只需要在进入的时候判断当前的指令值和 if 语句的起始指令值是否相同，相同即为循环，就可以修改之前保存的 if 分支为循环，我这里修改为 while 循环，比较好改

![](https://img-blog.csdnimg.cn/f49f1f7820c642c09ee871e5d79b3f53.png#pic_center)

最后一个就是 vmp 内的函数定义，调试代码分析知道这个就是定义 vmp 函数的：

![](https://img-blog.csdnimg.cn/6034da1d322b406c9cc3b639aeaba730.png#pic_center)

我是在他定义完了之后直接调用该函数进行还原，这样可以防止他只定义没有调用到，不好的地方在于参数个数不知道，所以这个需要在代码还原之后手动优化下，问题不大，如果你有更好的方法，希望能指导下。

![](https://img-blog.csdnimg.cn/5362d981ed5842f28942719358f4ecd8.png#pic_center)

##### 4. 开始编写 AST 代码

由于该网站的 vmp 只是部分 vmp，而我们还原代码是在原有的解释器上修改的，如果直接整完整代码进行运行，需要补一定的环境，这个不行，所以我这里将这个 **_$webrt_1670312749** 函数扣下来修改即可

首先导入我们需要用到的库：

```
const escodegen = require('escodegen');  // 用于将ast转为代码的
const esprima = require('esprima');  // 将代码转为ast
const estraverse = require('estraverse');  // 遍历ast的，这里用不到，用到了下面这个常量值
const Syntax = estraverse.Syntax;
var funs = {};  // 记录vmp函数的，可以方便调用
```

![](https://img-blog.csdnimg.cn/b6c97c04b19a4a70ac1072a5b42b779a.png#pic_center)

然后在这个函数里面修改传过来的函数参数，将他改为 ast 代码的格式，而不是一个字面量，因为真正程序调用的时候，传到这里表面就是个变量了，当然也可以在 **_0x207ec8** 这个循环函数内修改，但是不好确定哪些是函数参数，所以我在他上一个函数这里修改了

接下的就是按照上面第三步的分析开始写代码即可，如有不懂，可以在评论区讨论。

还原 jsvmp 代码这个第一次需要耐心点，等你熟悉起来了，后面的就会越来越快，里面主要就是一些 ast 代码的定义，需要传入栈的数据和存在变量区的值，最后就是函数定义和判断分支的实现，判断分支会比较难，处理好判断分支，剩下的就不是问题了。

如果觉得这个太难，可以试试这个大佬写的编译器：[给 "某音" 的 js 虚拟机写一个编译器](https://bbs.kanxue.com/thread-261414.htm)

可以编译自己的代码或者里面例子的代码，然后试着还原也不错！

##### 5. 分析生成后的代码

```
function _0x5b7a61_vmp(args_0, args_1, args_2, args_3, args_4, args_5, args_6, args_7, args_8, args_9, args_10, args_11, args_12, args_13, args_14, args_15, args_16, args_17, args_18, args_19, args_20, args_21, args_22) {
        var var_233 = 'X-Bogus';
        var var_24 = '_signature';
        var var_25 = window['XMLHttpRequest']['prototype'];
        var var_26 = var_25['open'];
        var var_27 = var_25['setRequestHeader'];
        var var_28 = var_25['send'];
        var var_29 = var_25['overrideMimeType'];
        if (var_25['_ac_intercepted']) {
            return;
        }
        var_25['_ac_intercepted'] = !0;
        var_25['setRequestHeader'] = function (args_0, args_1) {
            if (!this['_send']) {
                var var_9999 = new window['Object']();
                var_9999['func'] = 'setRequestHeader';
                var_9999['arguments'] = arguments;
                this['_byted_intercept_list']['push'](var_9999);
                if (_0xc5dbaf(window['RegExp'], _0x1a373c(['^content-type$', 'i']))['test'](args_0)) {
                    this['_byted_content'] = args_1['toString']()['toLowerCase']()['split'](';')[0];
                }
            }
            return var_27.apply(this, arguments);
        };
        var_25['overrideMimeType'] = function () {
            this['_overrideMimeTypeArgs'] = arguments;
            return var_29.apply(this, this['_overrideMimeTypeArgs']);
        };
        var_25['open'] = function (args_0, args_1, args_2) {
            this['_byted_intercept_list'] = [];
            var var_10000 = new window['Object']();
            var_10000['func'] = 'open';
            var_10000['arguments'] = arguments;
            this['_byted_intercept_list']['push'](var_10000);
            this['_byted_method'] = args_0['toUpperCase']();
            this['_byted_url'] = args_1;
            return var_26.apply(this, arguments);
        };
        var var_30 = [
            'onabort',
            'onerror',
            'onload',
            'onloadend',
            'onloadstart',
            'onprogress',
            'ontimeout'
        ];
        var var_31 = [
            'GET',
            'POST'
        ];
        var_25['send'] = function fun_4(args_0) {
            var var_6 = var_31['indexOf'](this['_byted_method']) !== 0 - 1;
            if (args_2(this['_byted_url'])) {
                if (var_6) {
                    if (this['_byted_url']['indexOf']('_signature=') > 0 - 1) {
                        return var_28(this, arguments);
                    }
                    this['_byted_body'] = args_0;
                    var var_7 = this['onreadystatechange'];
                    var var_8 = this['onabort'];
                    var var_9 = this['onerror'];
                    var var_10 = this['onload'];
                    var var_11 = this['onloadend'];
                    var var_12 = this['onloadstart'];
                    var var_13 = this['onprogress'];
                    var var_14 = this['ontimeout'];
                    var var_15 = new window['Object']();
                    var var_50 = 0;
                    while (var_50 < var_30['length']) {
                        var_15[var_30[var_50]] = this['upload'][var_30[var_50]];
                        ++var_50;
                    }
                    var var_16 = args_3['msStatus'];
                    var var_17 = args_3['__ac_testid'];
                    if (var_17 == '') {
                        var var_18 = ['msToken', args_3['msToken']];
                    } else {
                        var_18 = ['msToken', args_3['msToken'], '__ac_testid', var_17];
                    }
                    var var_19 = args_4(args_5(this['_byted_url']), var_18);
                    var var_20 = args_6(var_19);
                    var var_21 = args_7(var_20, this['_byted_body']);
                    var var_22 = args_4(var_19, [var_233, var_21]);
                    var var_23 = '';
                    if (args_8['v']) {
                        var_23 = var_22;
                    } else {
                        var var_10002 = new window['Object']();
                        var_10002['url'] = args_9(null, var_22);
                        var var_100 = var_10002;
                        if (this['_byted_method'] === 'POST') {
                            if (args_10(this['_byted_content'])) {
                                args_11(var_100, this['_byted_content'], this['_byted_body']);
                                var var_101 = args_12(var_100, args_13, 'forreal');
                                var_23 = args_4(var_22, [var_24, var_101]);
                            } else {
                                var_23 = var_22;
                            }
                        } else {
                            var var_251 = args_12(var_100, args_13, 'forreal');
                            var_23 = args_4(var_22, [var_24, var_251]);
                        }
                    }
                    if (this['_byted_intercept_list']) {
                        if (this['_byted_intercept_list'][0]['func'] !== 'open') {
                            return null;
                        }
                    }
                    var var_244 = this['_byted_intercept_list'];
                    var var_182 = 0;
                    while (var_182 < var_244['length']) {
                        if (var_182 === 0) {
                            var_244[var_182].arguments[1] = var_23;
                            this['_send'] = !0;
                            var_26.apply(this, var_244[var_182].arguments);
                        } else {
                            this[var_244[var_182]['func']].apply(this, var_244[var_182].arguments);
                        }
                        ++var_182;
                    }
                    if (this['_overrideMimeTypeArgs']) {
                        this['overrideMimeType']();
                    }
                    delete this['_byted_intercept_list'];
                    if (args_8['sdi']) {
                        this['setRequestHeader'](args_14['secInfoHeader'], args_15());

                    }
                    this['onreadystatechange'] = var_7;
                    this['onabort'] = var_8;
                    this['onerror'] = var_9;
                    this['onload'] = function () {
                        var var_6 = 0;
                        if (!this['responseURL']) {
                            if (!this['_byted_url']) {
                                var var_7 = '';
                                if (args_16(var_7)) {
                                    var_6 = 1;
                                }
                                if (var_7['indexOf'](window['location']['host']) !== 0 - 1) {
                                    var_6 = 2;
                                }
                                if (var_6 > 0) {
                                    var var_8 = this['getResponseHeader']('x-ms-token');
                                    if (var_8) {
                                        var var_9 = args_17(this['_byted_url']);
                                        if (var_9 === args_18['sec']) {
                                            args_3['msToken'] = var_8;
                                            args_3['msStatus'] = var_9;
                                            args_19('msToken', var_8);
                                            args_20(var_8);
                                            if (var_9 > var_16) {
                                                if (args_3['msNewTokenList']['length'] > 0) {
                                                    args_21(args_22, 2);
                                                }
                                            }
                                        } else if (var_16 >= args_3['msStatus']) {
                                            args_3['msToken'] = var_8;
                                        }
                                        if (var_16 === args_18['init']) {
                                            if (args_3['msNewTokenList']['length'] < 10) {
                                                args_3['msNewTokenList']['push'](var_8);
                                                if (args_3['msNewTokenList']['length'] === 1) {
                                                    args_20(var_8);
                                                    args_19('msToken', var_8);
                                                }
                                            }
                                        }
                                    }
                                }
                                if (var_10) {
                                    var_10(args_0);
                                }
                                return;
                            } else {
                                var_7 = this['_byted_url'];
                                if (args_16(var_7)) {
                                    var_6 = 1;
                                }
                                if (var_7['indexOf'](window['location']['host']) !== 0 - 1) {
                                    var_6 = 2;
                                }
                                if (var_6 > 0) {
                                    var_8 = this['getResponseHeader']('x-ms-token');
                                    if (var_8) {
                                        var_9 = args_17(this['_byted_url']);
                                        if (var_9 === args_18['sec']) {
                                            args_3['msToken'] = var_8;
                                            args_3['msStatus'] = var_9;
                                            args_19('msToken', var_8);
                                            args_20(var_8);
                                            if (var_9 > var_16) {
                                                if (args_3['msNewTokenList']['length'] > 0) {
                                                    args_21(args_22, 2);
                                                }
                                            }
                                        } else if (var_16 >= args_3['msStatus']) {
                                            args_3['msToken'] = var_8;
                                        }
                                        if (var_16 === args_18['init']) {
                                            if (args_3['msNewTokenList']['length'] < 10) {
                                                args_3['msNewTokenList']['push'](var_8);
                                                if (args_3['msNewTokenList']['length'] === 1) {
                                                    args_20(var_8);
                                                    args_19('msToken', var_8);
                                                }
                                            }
                                        }
                                    }
                                }
                                if (var_10) {
                                    var_10(args_0);
                                }
                            }
                        } else if (!this['responseURL']) {
                            var_7 = '';
                            if (args_16(var_7)) {
                                var_6 = 1;
                            }
                            if (var_7['indexOf'](window['location']['host']) !== 0 - 1) {
                                var_6 = 2;
                            }
                            if (var_6 > 0) {
                                var_8 = this['getResponseHeader']('x-ms-token');
                                if (var_8) {
                                    var_9 = args_17(this['_byted_url']);
                                    if (var_9 === args_18['sec']) {
                                        args_3['msToken'] = var_8;
                                        args_3['msStatus'] = var_9;
                                        args_19('msToken', var_8);
                                        args_20(var_8);
                                        if (var_9 > var_16) {
                                            if (args_3['msNewTokenList']['length'] > 0) {
                                                args_21(args_22, 2);
                                            }
                                        }
                                    } else if (var_16 >= args_3['msStatus']) {
                                        args_3['msToken'] = var_8;
                                    }
                                    if (var_16 === args_18['init']) {
                                        if (args_3['msNewTokenList']['length'] < 10) {
                                            args_3['msNewTokenList']['push'](var_8);
                                            if (args_3['msNewTokenList']['length'] === 1) {
                                                args_20(var_8);
                                                args_19('msToken', var_8);
                                            }
                                        }
                                    }
                                }
                            }
                            if (var_10) {
                                var_10(args_0);
                            }
                            return;
                        } else {
                            var_7 = this['responseURL'];
                            if (args_16(var_7)) {
                                var_6 = 1;
                            }
                            if (var_7['indexOf'](window['location']['host']) !== 0 - 1) {
                                var_6 = 2;
                            }
                            if (var_6 > 0) {
                                var_8 = this['getResponseHeader']('x-ms-token');
                                if (var_8) {
                                    var_9 = args_17(this['_byted_url']);
                                    if (var_9 === args_18['sec']) {
                                        args_3['msToken'] = var_8;
                                        args_3['msStatus'] = var_9;
                                        args_19('msToken', var_8);
                                        args_20(var_8);
                                        if (var_9 > var_16) {
                                            if (args_3['msNewTokenList']['length'] > 0) {
                                                args_21(args_22, 2);
                                            }
                                        }
                                    } else if (var_16 >= args_3['msStatus']) {
                                        args_3['msToken'] = var_8;
                                    }
                                    if (var_16 === args_18['init']) {
                                        if (args_3['msNewTokenList']['length'] < 10) {
                                            args_3['msNewTokenList']['push'](var_8);
                                            if (args_3['msNewTokenList']['length'] === 1) {
                                                args_20(var_8);
                                                args_19('msToken', var_8);
                                            }
                                        }
                                    }
                                }
                            }
                            if (var_10) {
                                var_10(args_0);
                            }
                        }
                    };
                    this['onloadend'] = var_11;
                    this['onloadstart'] = var_12;
                    this['onprogress'] = var_13;
                    this['ontimeout'] = var_14;
                    var var_216 = 0;
                    while (var_216 < var_30['length']) {
                        this['upload'][var_30[var_216]] = var_15[var_30[var_216]];
                        ++var_216
                    }
                }
                return var_28.apply(this, arguments);
            }
        };
    }
```

上面代码是在 vmp 还原后经过手动小优化的，因为分支处理不够好，会有重复分支，手动删除即可。

上面代码不调试，直接看，逻辑也都很清晰了，加密是在 **send** 函数里面，这一小段：

![](https://img-blog.csdnimg.cn/7dee1b1f85344ee5b24de1c916230b54.png#pic_center)

![](https://img-blog.csdnimg.cn/f49c4e75f0104841b4509fc37f22176d.png#pic_center)

![](https://img-blog.csdnimg.cn/d4474e093c204a62b9ea9eeb58966dc2.png#pic_center)

浏览器上也是可以得到结果的，证明是没问题的

最后我也将还原的代码将算法扣出来，然后用 python，去请求也没问题

![](https://img-blog.csdnimg.cn/295162385fee4dfd89c0ff93ab85218d.png#pic_center)

从还原的代码来看，对环境的检测也挺多的，检测了一些 dom 操作，还检测了一堆异步对象，都是获取设备的一些信息的。

##### 6. 参考文章

[**给 "某音" 的 js 虚拟机写一个编译器**](https://bbs.kanxue.com/thread-261414.htm)

[**某乎 x96 参数与 jsvmp 初体验**](https://www.52pojie.cn/thread-1619464-1-1.html)