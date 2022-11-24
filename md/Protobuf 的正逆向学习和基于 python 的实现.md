> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1692444-1-1.html)

> [md]## 1、起因 - 期初开始对 https://cat-match.easygame2021.com/sheep/v1/game/game_over_ex/ 抓到的包，很好奇，MatchPlayInfo 到底是个啥加密方式。

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)jingtai123 _ 本帖最后由 jingtai123 于 2022-9-26 09:06 编辑_  

1、起因
----

*   期初开始对 https://cat-match.easygame2021.com/sheep/v1/game/game_over_ex/ 抓到的包，很好奇，MatchPlayInfo 到底是个啥加密方式。
*   通过分析 JS 源码, 找到一个叫做 **_Protbuf_** 的东西，头一次 接触，学了一下，如何用 Python 实现这个玩意，所以有了这篇文章，算是对学习 protobuf 的记录。
*   欢迎大佬批评指正。

```
{
  "MatchPlayInfo" : "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",
  "Version" : "0.0.1",
  "MapSeed2" : "1666910531",
  "skin" : 1,
  "rank_time" : 144,
  "rank_role" : 1,
  "rank_state" : 1,
  "rank_score" : 1
}
```

2、protobuf 是什么
--------------

> protobuf 是 google 提供的一个 **开源序列化框架** ，**类似于 XML，JSON** 这样的数据表示语言，其最大的特点是 **基于二进制** ，因此比传统的 XML 表示高效短小得多。虽然是二进制数据格式，但并没有因此变得复杂，开发人员通过按照一定的语法定义结构化的消息格式，然后送给命令行工具，工具将自动生成 相关的类，可以支持 php、java、c++、python 等语言环境。通过将这些类包含在项目中，可以很轻松的调用相关方法来完成业务消息的序列化与反 序列化工作。
> 
> protobuf 在 google 中是一个比较核心的基础库，作为分布式运算涉及到大量的不同业务消息的传递，如何高效简洁的表示、操作这些业务消息在 google 这样的大规模应用中是至关重要的。而 protobuf 这样的库正好是在效率、数据大小、易用性之间取得了很好的平衡。

简单看来 就是一个序列化和反序列化工具。

3、protobuf 怎么用
--------------

#### 正向：

1.  定义一个. proto 文件的 message 结构体
2.  通过 proto 库提供的工具 生成所需语言的代码
3.  通过生成的代码 进行加解密

#### 逆向：

1.  通过 blackboxprotobuf 工具对密文解码
2.  通过读源码分析对应的参数
3.  构造 message 结构体
4.  通过生成的代码加解密

4、逆向实操
------

### step1

#### 对密文的预处理

1.  通过源码可以看到，MatchPlayInfo 是序列化后通过 base64 编码的
    
    ```
    var N ={        rank_score: 1,
                   rank_state: t,
                   rank_time: this.countdown,
                   rank_role: o,
                   skin: r,
                   MatchPlayInfo: S.default.base64_encode(b)
          };
    ```
    
2.  需要下述将 MatchPlayInfo 中的内容先 base64 解码
    
    > CAMiBQiKARAIIgUIiwEQCCIFCJABEAgiBQjRARAFIgUI0gEQBSIFCNMBEAUiBQi9ARAGIgUIvwEQBiIFCL4BEAYiBQijAhABIgUIpgIQASIFCKcCEAEiBQikAhABIgUIqAIQASIFCKUCEAEiBQibAhABIgUImAIQASIFCJoCEAEiBQiZAhABIgUIoQIQASIFCKICEAEiBQicAhABIgUInQIQASIFCJ8CEAEiBQieAhABIgUIoAIQASIFCJUCEAEiBQiJAhACIgUIigIQAiIFCIsCEAIiBQiMAhACIgUIjQIQAiIFCJQCEAIiBQiTAhACIgUIkgIQAiIFCJECEAIiBQiQAhACIgUIgAIQAiIFCIECEAIiBQiGAhACIgUIhQIQAiIFCIQCEAIiBQj5ARADIgUI+gEQAyIFCPsBEAMiBQj8ARADIgUI\/gEQAyIFCP8BEAMiBQjzARADIgUI9AEQAyIFCO0BEAMiBQjsARADIgUI6wEQAyIFCPIBEAMiBQjgARAEIgUI4QEQBCIFCOIBEAQiBQjmARAEIgUI3AEQBCIFCN0BEAQiBQjIARAFIgUIyQEQBSIFCNABEAUiBQjPARAFIgUI1AEQBSIFCMEBEAUiBQjAARAGIgUIswEQBiIFCLQBEAYiBAgwEA0iBAgxEA0iBAguEA0iBAgvEA0iBAglEA0iBAgmEA0iBAgiEA4iBAgjEA4iBAgkEA4iBAgXEA4iBAgYEA4iBAgZEA4iBQipARAHIgUIpwEQByIFCKgBEAciBAgGEA8iBAgFEA8iBAgJEA8iBAgIEA8iBAgHEA8iBAgLEA8iBQiWAhABIgUIhwIQAiIFCIgCEAIiBQiDAhACIgUI9QEQAyIFCPgBEAMiBQjxARADIgUI6gEQBCIFCNsBEAQiBQjfARAEIgUIlwIQASIFCI4CEAIiBQiPAhACIgUIggIQAiIFCPYBEAMiBQj9ARADIgUI9wEQAyIFCOcBEAQiBQjoARAEIgUI6QEQBCIFCNYBEAQiBQjeARAEIgUI5QEQBCIFCMIBEAUiBQjVARAFIgUIxwEQBSIFCLIBEAYiBQjuARADIgUI7wEQAyIFCPABEAMiBQjXARAEIgUI4wEQBCIFCOQBEAQiBQjYARAEIgUI2QEQBCIFCNoBEAQiBQjDARAFIgUIxAEQBSIFCMUBEAUiBQjGARAFIgUIygEQBSIFCM0BEAUiBQjLARAFIgUIzgEQBSIFCMwBEAUiBQi1ARAGIgUItgEQBiIFCLcBEAYiBQi4ARAGIgUIvAEQBiIFCLEBEAYiBQiwARAGIgUIuwEQBiIFCLoBEAYiBQivARAGIgUIrgEQBiIFCLkBEAYiBQiZARAHIgUImwEQByIFCKsBEAciBQiqARAHIgUInAEQByIFCJoBEAciBQieARAHIgUIoQEQByIFCJ8BEAciBQigARAHIgUIpAEQByIFCKYBEAciBQijARAHIgUIpQEQByIFCJgBEAciBQiMARAIIgUIjgEQCCIFCJUBEAgiBQiWARAIIgUIlAEQCCIFCJMBEAgiBQiFARAIIgUIgwEQCCIFCIcBEAgiBQiPARAIIgUIjQEQCCIFCIkBEAgiBQiCARAIIgUIrAEQBiIFCK0BEAYiBQidARAHIgUIlwEQByIFCKIBEAciBQiRARAIIgUIkgEQCCIECH8QCSIFCIEBEAkiBAh9EAkiBAgOEA8iBAgQEA8iBAgPEA8iBAh1EAkiBAh2EAkiBAh3EAkiBAh8EAkiBAhwEAkiBAhxEAkiBAh0EAkiBAh7EAkiBAhvEAkiBAhrEAoiBAhqEAoiBAhpEAoiBQiEARAIIgUIhgEQCCIFCIgBEAgiBAhiEAoiBAhhEAoiBAhnEAoiBQiAARAJIgQIfhAJIgQIeRAJIgQIeBAJIgQIehAJIgQIchAJIgQIcxAJIgQIbhAJIgQIbRAJIgQIZhAKIgQIYxAKIgQIbBAKIgQIZBAKIgQIZRAKIgQIXRAKIgQIVhALIgQIWhALIgQIWRALIgQITxALIgQISRALIgQIUhALIgQIWxAKIgQIXBAKIgQIXhAKIgQIQxAMIgQIRBAMIgQIPRAMIgQIVBALIgQIVRALIgQIUxALIgQIThALIgQITBALIgQITRALIgQIRxAMIgQIOxAMIgQIPBAMIgQINBANIgQIMxANIgQINRANIgQISBAMIgQIQhAMIgQIOhAMIgQIJxANIgQIMhANIgQIKRANIgQIKBANIgQIHRAOIgQIHBAOIgQIGxAOIgQIEhAPIgQIERAPIgQIDRAPIgQIXxAKIgQIaBAKIgQIYBAKIgQIWBALIgQIVxALIgQIURALIgQIUBALIgQISxALIgQIShALIgQIRhAMIgQIRRAMIgQIPxAMIgQIPhAMIgQINxAMIgQIQRAMIgQIQBAMIgQIORAMIgQIOBAMIgQINhANIgQILRANIgQILBANIgQIIBAOIgQIIRAOIgQIGhAOIgQIKhANIgQIKxANIgQIExAOIgQIHxAOIgQIFhAOIgQIHhAOIgQIBBAPIgQIFBAOIgQIFRAOIgQIAhAPIgQIAxAPIgQIARAPIgQIDBAPIgQIChAPIgQIABAB
    
3.  base64 解码：
    
    ```
    import base64
    mydate = "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"
    out_b = base64.b64decode(mydata)
    ```
    
    得到的就是 protobuf 序列化后的结果
    
    > b'\x08\x03"\x05\x08\x8a\x01\x10\x08"\x05\x08\x8b\x01\x10\x08"\x05\x08\x90\x01\x10\x08"\x05\x08\xd1\x01\x10\x05"………………
    

### step2

#### 使用 blackboxprotobuf 对密文反序列化

1.  安装 blackboxprotobuf ， protobuf
    
    ```
    pip install blackboxprotobuf protobuf
    ```
    
2.  解析为 json
    
    ```
    import blackboxprotobuf
    deserialize_data, message_type = blackboxprotobuf.protobuf_to_json(out_b)
    print(f"原始数据: {deserialize_data}")
    print(f"消息类型: {message_type}")
    ```
    
    得到结果：
    

> 原始数据: {"1": "3","4": [ { "1": "138","2": "8"},{"1": "139","2": "8"},{"1": "144","2": "8"},…………]}  
> 消息类型: {'1': {'type': 'int', 'name': ''},'4': {'type':'message','message_typedef': {'1': {'type':'int','name':''}, '2': {'type': 'int', 'name': ''}},'name':''}}

### step3

#### 分析源码构造 message 的. proto 文件

1.  源码：
    
    ##### MatchPlayInfo 部分
    
    ```
    o.MatchPlayInfo = function () {
                    function t(t) {
                        if (this.stepInfoList = [], t) for (var e = Object.keys(t), o = 0; o < e.length; ++o) null != t[e[o]] && (this[e[o]] = t[e[o]]);
                    }
    
                    return t.prototype.gameType = 0, t.prototype.mapId = 0, t.prototype.mapSeed = 0,
                        t.prototype.stepInfoList = r.emptyArray, t.create = function (e) {
                        return new t(e);
                    }, t.encode = function (t, e) {
                        if (e || (e = a.create()), null != t.gameType && Object.hasOwnProperty.call(t, "gameType") && e.uint32(8).int32(t.gameType),
                        null != t.mapId && Object.hasOwnProperty.call(t, "mapId") && e.uint32(16).int32(t.mapId),
                        null != t.mapSeed && Object.hasOwnProperty.call(t, "mapSeed") && e.uint32(24).int32(t.mapSeed),
                        null != t.stepInfoList && t.stepInfoList.length) for (var o = 0; o < t.stepInfoList.length; ++o) c.protocol.MatchStepInfo.encode(t.stepInfoList[o], e.uint32(34).fork()).ldelim();
                        return e;
                    }, t.decode = function (t, e) {
                        t instanceof i || (t = i.create(t));
                        for (var o = void 0 === e ? t.len : t.pos + e, n = new c.protocol.MatchPlayInfo(); t.pos < o;) {
                            var a = t.uint32();
                            switch (a >>> 3) {
                                case 1:
                                    n.gameType = t.int32();
                                    break;
    
                                case 2:
                                    n.mapId = t.int32();
                                    break;
    
                                case 3:
                                    n.mapSeed = t.int32();
                                    break;
    
                                case 4:
                                    n.stepInfoList && n.stepInfoList.length || (n.stepInfoList = []), n.stepInfoList.push(c.protocol.MatchStepInfo.decode(t, t.uint32()));
                                    break;
    
                                default:
                                    t.skipType(7 & a);
                            }
                        }
                        return n;
                    }, t;
                }()
    ```
    
    ##### MatchStepInfo 部分
    
    ```
    o.MatchStepInfo = function () {
                    function t(t) {
                        if (t) for (var e = Object.keys(t), o = 0; o < e.length; ++o) null != t[e[o]] && (this[e[o]] = t[e[o]]);
                    }
    
                    return t.prototype.chessIndex = 0, t.prototype.timeTag = 0, t.create = function (e) {
                        return new t(e);
                    }, t.encode = function (t, e) {
                        return e || (e = a.create()), null != t.chessIndex && Object.hasOwnProperty.call(t, "chessIndex") && e.uint32(8).int32(t.chessIndex),
                        null != t.timeTag && Object.hasOwnProperty.call(t, "timeTag") && e.uint32(16).int32(t.timeTag),
                            e;
                    }, t.decode = function (t, e) {
                        t instanceof i || (t = i.create(t));
                        for (var o = void 0 === e ? t.len : t.pos + e, n = new c.protocol.MatchStepInfo(); t.pos < o;) {
                            var a = t.uint32();
                            switch (a >>> 3) {
                                case 1:
                                    n.chessIndex = t.int32();
                                    break;
    
                                case 2:
                                    n.timeTag = t.int32();
                                    break;
    
                                default:
                                    t.skipType(7 & a);
                            }
                        }
                        return n;
                    }, t;
               }()
    ```
    
2.  根据源码及解析结果构建. proto 文件
    

```
syntax = 'proto3';

message MatchPlayInfo{
  int32 gameType = 1;
  int32 mapId = 2;
  int32 mapSeed = 3;
  repeated StepInfoList stepInfoList = 4;
}
message StepInfoList{
  int32 chessIndex = 1;
  int32 timeTag = 2;
}
```

存为 yang.proto 文件

### step4

#### 通过. proto 文件生成对应的 python 代码

1.  安装 grpcio-tools 库
    
    ```
    pip install grpcio-tools
    ```
    
2.  终端运行 生成对应的 yang_pb2.py
    
    ```
    python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. yang.proto
    ```
    

### step5

#### 引用 yang_pb2.py 进行加解密

#### 解密：

```
import yang_pb2
mydata = "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"
out_b = base64.b64decode(mydata)

matchPlayInfo_out = yang_pb2.MatchPlayInfo()
matchPlayInfo_out.ParseFromString(out_b)
print(matchPlayInfo_out)
```

打印结果：

```
gameType: 3
stepInfoList {
  chessIndex: 138
  timeTag: 8
}
stepInfoList {
  chessIndex: 139
  timeTag: 8
}
stepInfoList {
  chessIndex: 144
  timeTag: 8
}
stepInfoList {
  chessIndex: 209
  timeTag: 5
}
………………
```

#### 构造加密：

```
improt yang_pb2
matchPlayInfo=yang_pb2.MatchPlayInfo()
matchPlayInfo.gameType=3
stepInfoList = yang_pb2.StepInfoList()
for i in range(1,297):  
    stepInfoList.chessIndex=i
    stepInfoList.timeTag=i
    matchPlayInfo.stepInfoList.append(stepInfoList)
out_b = matchPlayInfo.SerializeToString()
```

out_b 结果：

```
b'\x08\x03"\x04\x08\x01\x10\x01"\x04\x08\x02\x10\x02"\x04\x08\x03\x10\x03"\x04\x08\x04\x10\x04"\x04\x08\x05\x10\x05"\…………
```

对 out_b 进行 base64 编码即得到结果

```
b64out_b = base64.b64encode(out_b)
print(b64out_b)
```

> b'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'

#### 用上述的解密方法对构造的加密解密：

```
matchPlayInfo_out = yang_pb2.MatchPlayInfo()
matchPlayInfo_out.ParseFromString(out_b)
print(matchPlayInfo_out)
```

matchPlayInfo_out 结果：

```
gameType: 3
stepInfoList {
  chessIndex: 1
  timeTag: 1
}
stepInfoList {
  chessIndex: 2
  timeTag: 2
}
stepInfoList {
  chessIndex: 3
  timeTag: 3
}
stepInfoList {
  chessIndex: 4
  timeTag: 4
}
…………
stepInfoList {
  chessIndex: 295
  timeTag: 295
}
stepInfoList {
  chessIndex: 296
  timeTag: 296
}
```![](https://avatar.52pojie.cn/images/noavatar_middle.gif)jingtai123 这篇帖子的目的只是探讨 protobuf  不是为了刷次数 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) gugouo163

> [子诺 发表于 2022-9-27 10:59](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=44167449&ptid=1692444)  
> 请问下楼主，这个方法我能够成功提交。但是为什么自己构造的数据不能刷通关次数，它们的通关次数 是什么原 ...

同样情况，感觉还是数据有不对![](https://avatar.52pojie.cn/images/noavatar_middle.gif)子诺 请问下楼主，这个方法我能够成功提交。但是为什么自己构造的数据不能刷通关次数，它们的通关次数 是什么原理？有没有知道的？![](https://avatar.52pojie.cn/images/noavatar_middle.gif)zhaohyperion 感谢分享，blackboxprotobuf, 这个玩意木有听说过，看看 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) wkunzhi 学习了，谢谢 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) buggur 有没有 php 版本对这个进行实现的 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) buggur 为什么我解出来是这样的 [image.png](forum.php?mod=attachment&aid=MjU1Njc5OXw5ZTRiYTI4YnwxNjY5MjkwNTg3fDkzMDQwMnwxNjkyNDQ0&nothumb=yes) _(18.84 KB, 下载次数: 3)_

[下载附件](forum.php?mod=attachment&aid=MjU1Njc5OXw5ZTRiYTI4YnwxNjY5MjkwNTg3fDkzMDQwMnwxNjkyNDQ0&nothumb=yes)

2022-9-26 19:01 上传 [![](https://static.52pojie.cn/static/image/common/rleft.gif)](javascript:;) [![](https://static.52pojie.cn/static/image/common/rright.gif)](javascript:;)

![](https://attach.52pojie.cn/forum/202209/26/190159gsgnrs3cvk3zbvgk.png) ![](https://avatar.52pojie.cn/images/noavatar_middle.gif)blacksails 感谢楼主分享 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) shutegame 厉害了。。。DUBBO 的东西也破解了 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) mfvpnhaha 看不懂啊这个咋使用的哦？![](https://avatar.52pojie.cn/images/noavatar_middle.gif)jingtai123

> [shutegame 发表于 2022-9-26 20:32](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=44162725&ptid=1692444)  
> 厉害了。。。DUBBO 的东西也破解了

算不上破解  正常使用 ![](https://avatar.52pojie.cn/images/noavatar_middle.gif) BySiHan 好像并未成功计算次数 楼主可以自己再试一下提交