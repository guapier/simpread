> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1741168-1-1.html)

> 免责声明: 本发布的内容，仅用于学习研究，请勿用于非法用途和商业用途！如因此产生任何法律纠纷，均与作者无关！使用说明：支持的版本 3.8.1.26。

![](https://avatar.52pojie.cn/images/noavatar_middle.gif)odmin _ 本帖最后由 odmin 于 2023-2-2 15:54 编辑_  
**免责声明:** 本发布的内容，仅用于学习研究，请勿用于非法用途和商业用途！如因此产生任何法律纠纷，均与作者无关！  
![](https://attach.52pojie.cn/forum/202302/02/153459s73lbxxtngzxntlx.png)

**123.png** _(367.4 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4NzgyMHw0ZTZjNTY1NXwxNjc1MzI4ODk2fDkzMDQwMnwxNzQxMTY4&nothumb=yes)

2023-2-2 15:34 上传

  
使用说明：  
支持的版本 3.8.1.26。 src: 主要的 dll 代码 tool：简单的注入工具。 python: 简单的服务器，用以接收消息内容。 release：编译好的 dll。  
0. 首先安装对应的微信版本对应 3.8.1.26 版本。  
1. 通过 cmake 构建成功后，将 wxhelper.dll 注入到微信，本地启动 tcp server，监听 19088 端口。  
2. 通过 http 协议与 dll 通信，方便客户端操作。  
3. **接口的 url 为 http://127.0.0.1:19088，注入成功后，直接进行调用即可。**  
4. 特别注意数据库查询接口需要先调用获取到句柄之后，才能进行查询。  
5. 相关功能只在 win11 环境下进行简单测试，其他环境无法保证。  
6. 具体参考源码。  
编译环境  
Visual Studio 2022(x86)  
Visual Studio code  
cmake  
vcpkg  
构建步骤  
以下是在 vscode 中操作，vs 中的操作类似。  
1. 安装 vcpkg，cmake，vscode  
2. 安装相应的库，如果安装的版本不同，则根据 vcpkg 安装成功后提示的 find_package 修改 CMakeLists.txt 内容即可。或者自己编译。  
    vcpkg  install mongoose    
    vcpkg  install nlohmann-json  
3.vscode 配置 CMakePresets.json, 主要设置 CMAKE_C_COMPILER 和 CMAKE_CXX_COMPILER 为 cl.exe. 参考如下  
[Asm] _纯文本查看_ _复制代码_

```
{
            "name": "x86-release",
            "displayName": "x86-release",
            "description": "Sets Ninja generator, build and install directory",
            "generator": "Ninja",
            "binaryDir": "${sourceDir}/out/build/${presetName}",
            "architecture":{
                "value": "x86",
                "strategy": "external"
            },
            "cacheVariables": {
                "CMAKE_C_COMPILER": "cl.exe",
                "CMAKE_CXX_COMPILER": "cl.exe",
                "CMAKE_BUILD_TYPE": "Release",
                "CMAKE_INSTALL_PREFIX": "${sourceDir}/out/install/${presetName}",
                "CMAKE_TOOLCHAIN_FILE": {
                    "value": "C:/soft/vcpkg/scripts/buildsystems/vcpkg.cmake",
                     "type": "FILEPATH"
                  }
            },
            "environment": {
 
            }
          
        }
```

4.vscode 中右键 configure all projects, 在 Terminal 中点击 Run Task，如没有先配置 build 任务，然后运行即可  
更新说明  
  
[Asm] _纯文本查看_ _复制代码_

```
2022-12-26 ： 增加3.8.1.26版本支持。
 
2022-12-29 ： 新增提取文字功能。
 
2023-01-02 ： 退出微信登录。
 
2023-01-31 ： 新增修改群昵称（仅支持3.8.1.26）。
 
2023-02-01 ： 新增拍一拍（仅支持3.8.1.26）。
```

[C++] _纯文本查看_ _复制代码_

```
#ifndef API_H_
#define API_H_
 
 
typedef enum WECHAT_HTTP_APISTag
{
    // login check
    WECHAT_IS_LOGIN = 0,
    // self info
    WECHAT_GET_SELF_INFO,
    // send message
    WECHAT_MSG_SEND_TEXT,
    WECHAT_MSG_SEND_AT,
    WECHAT_MSG_SEND_CARD,
    WECHAT_MSG_SEND_IMAGE,
    WECHAT_MSG_SEND_FILE,
    WECHAT_MSG_SEND_ARTICLE,
    WECHAT_MSG_SEND_APP,
    // receive message
    WECHAT_MSG_START_HOOK,
    WECHAT_MSG_STOP_HOOK,
    WECHAT_MSG_START_IMAGE_HOOK,
    WECHAT_MSG_STOP_IMAGE_HOOK,
    WECHAT_MSG_START_VOICE_HOOK,
    WECHAT_MSG_STOP_VOICE_HOOK,
    // contact
    WECHAT_CONTACT_GET_LIST,
    WECHAT_CONTACT_CHECK_STATUS,
    WECHAT_CONTACT_DEL,
    WECHAT_CONTACT_SEARCH_BY_CACHE,
    WECHAT_CONTACT_SEARCH_BY_NET,
    WECHAT_CONTACT_ADD_BY_WXID,
    WECHAT_CONTACT_ADD_BY_V3,
    WECHAT_CONTACT_ADD_BY_PUBLIC_ID,
    WECHAT_CONTACT_VERIFY_APPLY,
    WECHAT_CONTACT_EDIT_REMARK,
    // chatroom
    WECHAT_CHATROOM_GET_MEMBER_LIST,
    WECHAT_CHATROOM_GET_MEMBER_NICKNAME,
    WECHAT_CHATROOM_DEL_MEMBER,
    WECHAT_CHATROOM_ADD_MEMBER,
    WECHAT_CHATROOM_SET_ANNOUNCEMENT,
    WECHAT_CHATROOM_SET_CHATROOM_NAME,
    WECHAT_CHATROOM_SET_SELF_NICKNAME,
    // database
    WECHAT_DATABASE_GET_HANDLES,
    WECHAT_DATABASE_BACKUP,
    WECHAT_DATABASE_QUERY,
    // version
    WECHAT_SET_VERSION,
    // log
    WECHAT_LOG_START_HOOK,
    WECHAT_LOG_STOP_HOOK,
    // browser
    WECHAT_BROWSER_OPEN_WITH_URL,
    WECHAT_GET_PUBLIC_MSG,
    WECHAT_MSG_FORWARD_MESSAGE,
    WECHAT_GET_QRCODE_IMAGE,
    WECHAT_GET_A8KEY,
    WECHAT_MSG_SEND_XML,
    WECHAT_LOGOUT,
    WECHAT_GET_TRANSFER,
    WECHAT_GET_CONTACT_ALL,
    WECHAT_GET_CHATROOM_INFO,
    WECHAT_GET_IMG_BY_NAME,
    WECHAT_DO_OCR,
    WECHAT_SEND_PAT_MSG,
} WECHAT_HTTP_APIS,
    *PWECHAT_HTTP_APIS;
 
 
int http_start(int port);
#endif
```

### 接口文档：

#### 0. 检查微信登录 **

###### 接口功能

> 检查微信是否登录

###### 接口地址

> [/api/?type=0](/api/?type=0)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

```
{
            "name": "x86-release",
            "displayName": "x86-release",
            "description": "Sets Ninja generator, build and install directory",
            "generator": "Ninja",
            "binaryDir": "${sourceDir}/out/build/${presetName}",
            "architecture":{
                "value": "x86",
                "strategy": "external"
            },
            "cacheVariables": {
                "CMAKE_C_COMPILER": "cl.exe",
                "CMAKE_CXX_COMPILER": "cl.exe",
                "CMAKE_BUILD_TYPE": "Release",
                "CMAKE_INSTALL_PREFIX": "${sourceDir}/out/install/${presetName}",
                "CMAKE_TOOLCHAIN_FILE": {
                    "value": "C:/soft/vcpkg/scripts/buildsystems/vcpkg.cmake",
                     "type": "FILEPATH"
                  }
            },
            "environment": {
 
            }
          
        }
```

###### 返回字段

```
2022-12-26 ： 增加3.8.1.26版本支持。
 
2022-12-29 ： 新增提取文字功能。
 
2023-01-02 ： 退出微信登录。
 
2023-01-31 ： 新增修改群昵称（仅支持3.8.1.26）。
 
2023-02-01 ： 新增拍一拍（仅支持3.8.1.26）。
```

###### 接口示例

入参：

响应：

```
{
    "code": 1,
    "result": "ok"
}
```

#### 1. 获取登录用户信息 **

###### 接口功能

> 获取登录用户信息

###### 接口地址

> [/api/?type=1](/api/?type=1)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

```
#ifndef API_H_
#define API_H_
 
 
typedef enum WECHAT_HTTP_APISTag
{
    // login check
    WECHAT_IS_LOGIN = 0,
    // self info
    WECHAT_GET_SELF_INFO,
    // send message
    WECHAT_MSG_SEND_TEXT,
    WECHAT_MSG_SEND_AT,
    WECHAT_MSG_SEND_CARD,
    WECHAT_MSG_SEND_IMAGE,
    WECHAT_MSG_SEND_FILE,
    WECHAT_MSG_SEND_ARTICLE,
    WECHAT_MSG_SEND_APP,
    // receive message
    WECHAT_MSG_START_HOOK,
    WECHAT_MSG_STOP_HOOK,
    WECHAT_MSG_START_IMAGE_HOOK,
    WECHAT_MSG_STOP_IMAGE_HOOK,
    WECHAT_MSG_START_VOICE_HOOK,
    WECHAT_MSG_STOP_VOICE_HOOK,
    // contact
    WECHAT_CONTACT_GET_LIST,
    WECHAT_CONTACT_CHECK_STATUS,
    WECHAT_CONTACT_DEL,
    WECHAT_CONTACT_SEARCH_BY_CACHE,
    WECHAT_CONTACT_SEARCH_BY_NET,
    WECHAT_CONTACT_ADD_BY_WXID,
    WECHAT_CONTACT_ADD_BY_V3,
    WECHAT_CONTACT_ADD_BY_PUBLIC_ID,
    WECHAT_CONTACT_VERIFY_APPLY,
    WECHAT_CONTACT_EDIT_REMARK,
    // chatroom
    WECHAT_CHATROOM_GET_MEMBER_LIST,
    WECHAT_CHATROOM_GET_MEMBER_NICKNAME,
    WECHAT_CHATROOM_DEL_MEMBER,
    WECHAT_CHATROOM_ADD_MEMBER,
    WECHAT_CHATROOM_SET_ANNOUNCEMENT,
    WECHAT_CHATROOM_SET_CHATROOM_NAME,
    WECHAT_CHATROOM_SET_SELF_NICKNAME,
    // database
    WECHAT_DATABASE_GET_HANDLES,
    WECHAT_DATABASE_BACKUP,
    WECHAT_DATABASE_QUERY,
    // version
    WECHAT_SET_VERSION,
    // log
    WECHAT_LOG_START_HOOK,
    WECHAT_LOG_STOP_HOOK,
    // browser
    WECHAT_BROWSER_OPEN_WITH_URL,
    WECHAT_GET_PUBLIC_MSG,
    WECHAT_MSG_FORWARD_MESSAGE,
    WECHAT_GET_QRCODE_IMAGE,
    WECHAT_GET_A8KEY,
    WECHAT_MSG_SEND_XML,
    WECHAT_LOGOUT,
    WECHAT_GET_TRANSFER,
    WECHAT_GET_CONTACT_ALL,
    WECHAT_GET_CHATROOM_INFO,
    WECHAT_GET_IMG_BY_NAME,
    WECHAT_DO_OCR,
    WECHAT_SEND_PAT_MSG,
} WECHAT_HTTP_APIS,
    *PWECHAT_HTTP_APIS;
 
 
int http_start(int port);
#endif
```

###### 返回字段

```
LOCAL_IP = '192.168.1.102'  # 本地IP
 
def hook_msg():
    # hook 消息
    url = f"http://{WECHAT_IP}:19088/api/?type=9"
    data = {
        'port': "19099",
        'ip': LOCAL_IP
    }
    resp = requests.post(url, json=data)
    print(resp.json())
 
 
if __name__ == '__main__':
    print("Server Start : " + LOCAL_IP)
    # 发送消息
    send_msg('Hello! ', 'wxid')
```

###### 接口示例

入参：

响应：

```
{"code":1,"data":{"account":"xx","bigImage":"https://wx.qlogo.cn/mmhead/ver_1xx","city":"xx","country":"CN","currentDataPath":"xxx","dataRootPath":"C:\\xx","dataSavePath":"C:\\xx","mobie":"13949175447","name":"xx","province":"xx","smallImage":"xx","wxid":"xx"},"result":"OK"}
```

#### 2. 发送文本消息 **

###### 接口功能

> 发送文本消息

###### 接口地址

> [/api/?type=2](/api/?type=2)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>wxid</td><td>ture</td><td>string</td><td>接收人 wxid</td></tr><tr><td>msg</td><td>true</td><td>string</td><td>消息文本内容</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 不为 0 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "wxid": "filehelper",
    "msg": "1112222"
}
```

响应：

```
{"code":345686720,"result":"OK"}
```

#### 5. 发送图片消息 **

###### 接口功能

> 发送图片消息

###### 接口地址

> [/api/?type=5](/api/?type=5)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>wxid</td><td>ture</td><td>string</td><td>接收人 wxid</td></tr><tr><td>imagePath</td><td>true</td><td>string</td><td>图片路径</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 不为 0 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "wxid": "filehelper",
    "imagePath": "C:/Users/123.png"
}
```

响应：

```
{"code":345686724,"result":"OK"}
```

#### 6. 发送文件消息 **

###### 接口功能

> 发送文件

###### 接口地址

> [/api/?type=6](/api/?type=6)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>wxid</td><td>ture</td><td>string</td><td>接收人 wxid</td></tr><tr><td>filePath</td><td>true</td><td>string</td><td>文件路径</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "wxid": "filehelper",
    "filePath": "C:/Users/123.txt"
}
```

响应：

```
{"code":1,"result":"OK"}
```

#### 9.hook 消息 **

###### 接口功能

> hook 接收文本消息，图片消息，群消息. 该接口将 hook 的消息通过 tcp 回传给本地的端口

###### 接口地址

> [/api/?type=9](/api/?type=9)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>port</td><td>ture</td><td>string</td><td>本地服务端端口，用来接收消息内容</td></tr><tr><td>ip</td><td>ture</td><td>string</td><td>服务端 ip 地址，用来接收消息内容，可以是任意 ip, 即 tcp 客户端连接的服务端的 ip (3.8.1.26 版本)</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "port": "19099"
    "ip":"127.0.0.1"
}
```

响应：

```
{"code":1,"result":"OK"}
```

#### 10. 取消 hook 消息 **

###### 接口功能

> 取消 hook 消息

###### 接口地址

> [/api/?type=10](/api/?type=10)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

响应：

```
{"code":1,"result":"OK"}
```

#### 11.hook 图片 **

###### 接口功能

> hook 图片原始内容，不推荐该接口，可以使用图片查询接口

###### 接口地址

> [/api/?type=11](/api/?type=11)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>imgDir</td><td>ture</td><td>string</td><td>图片保存的目录</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "imgDir":"C:\\other"
}
```

响应：

```
{"code":1,"result":"OK"}
```

#### 12. 取消 hook 图片 **

###### 接口功能

> 取消 hook 图片

###### 接口地址

> [/api/?type=12](/api/?type=12)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

响应：

```
{"code":1,"result":"OK"}
```

#### 17. 删除好友 **

###### 接口功能

> 删除好友

###### 接口地址

> [/api/?type=17](/api/?type=17)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>wxid</td><td>ture</td><td>string</td><td>好友 wxid</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "wxid":"wxid_o"
}
```

响应：

```
{"code":1,"result":"OK"}
```

#### 25. 获取群成员 **

###### 接口功能

> 获取群成员

###### 接口地址

> [/api/?type=25](/api/?type=25)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>chatRoomId</td><td>ture</td><td>string</td><td>群 id</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr><tr><td>data</td><td>object</td><td>返回内容</td></tr><tr><td>admin</td><td>string</td><td>群主 id</td></tr><tr><td>chatRoomId</td><td>string</td><td>群 id</td></tr><tr><td>members</td><td>string</td><td>群成员 id 以 ^ 分隔</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "chatRoomId":"123@chatroom"
}
```

响应：

```
{"code":1,"data":{"admin":"wxid","chatRoomId":"123@chatroom","members":"wxid_123^Gwxid_456^Gwxid_45677"},"result":"OK"}
```

#### 27. 删除群成员 **

###### 接口功能

> 删除群成员

###### 接口地址

> [/api/?type=27](/api/?type=27)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>chatRoomId</td><td>ture</td><td>string</td><td>群 id</td></tr><tr><td>memberIds</td><td>ture</td><td>string</td><td>成员 id，以, 分割</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "chatRoomId":"34932563384@chatroom",
    "memberIds":"wxid_oyb662qhop4422"
}
```

响应：

```
{"code":1,"result":"OK"}
```

#### 28. 增加群成员 **

###### 接口功能

> 增加群成员

###### 接口地址

> [/api/?type=28](/api/?type=28)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>chatRoomId</td><td>ture</td><td>string</td><td>群 id</td></tr><tr><td>memberIds</td><td>ture</td><td>string</td><td>成员 id，以, 分割</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "chatRoomId":"34932563384@chatroom",
    "memberIds":"wxid_oyb662qhop4422"
}
```

响应：

```
{"code":1,"result":"OK"}
```

#### 31. 修改自身群昵称 **

###### 接口功能

> 修改群名片

###### 接口地址

> [/api/?type=31](/api/?type=31)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>chatRoomId</td><td>ture</td><td>string</td><td>群 id</td></tr><tr><td>wxid</td><td>ture</td><td>string</td><td>自己的 id，只能修改自己的群名片</td></tr><tr><td>nickName</td><td>ture</td><td>string</td><td>修改的昵称</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "chatRoomId":"34932563384@chatroom",
    "wxid":"wxid_272211111121112",
    "nickName":"昵称test"
}
```

响应：

```
{"code":1,"result":"OK"}
```

#### 32. 获取数据库句柄 **

###### 接口功能

> 获取 sqlite3 数据库句柄

###### 接口地址

> [/api/?type=32](/api/?type=32)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr><tr><td>data</td><td>array</td><td>返回数据</td></tr><tr><td>databaseName</td><td>string</td><td>数据库名称</td></tr><tr><td>handle</td><td>int</td><td>数据库句柄</td></tr><tr><td>tables</td><td>array</td><td>表信息</td></tr><tr><td>name</td><td>string</td><td>表名</td></tr><tr><td>rootpage</td><td>string</td><td>rootpage</td></tr><tr><td>sql</td><td>string</td><td>sql</td></tr><tr><td>tableName</td><td>string</td><td>tableName</td></tr></tbody></table>

###### 接口示例

入参：

响应：

```
{
  "data": [
    {
      "databaseName": "MicroMsg.db",
      "handle": 119561688,
      "tables": [
        {
          "name": "Contact",
          "rootpage": "2",
          "sql": "CREATE TABLE Contact(UserName TEXT PRIMARY KEY ,Alias TEXT,EncryptUserName TEXT,DelFlag INTEGER DEFAULT 0,Type INTEGER DEFAULT 0,VerifyFlag INTEGER DEFAULT 0,Reserved1 INTEGER DEFAULT 0,Reserved2 INTEGER DEFAULT 0,Reserved3 TEXT,Reserved4 TEXT,Remark TEXT,NickName TEXT,LabelIDList TEXT,DomainList TEXT,ChatRoomType int,PYInitial TEXT,QuanPin TEXT,RemarkPYInitial TEXT,RemarkQuanPin TEXT,BigHeadImgUrl TEXT,SmallHeadImgUrl TEXT,HeadImgMd5 TEXT,ChatRoomNotify INTEGER DEFAULT 0,Reserved5 INTEGER DEFAULT 0,Reserved6 TEXT,Reserved7 TEXT,ExtraBuf BLOB,Reserved8 INTEGER DEFAULT 0,Reserved9 INTEGER DEFAULT 0,Reserved10 TEXT,Reserved11 TEXT)",
          "tableName": "Contact"
        }
      ]
    }
  ],
  "result": "OK"
}
```

#### 34. 查询数据库 **

###### 接口功能

> 查询数据库

###### 接口地址

> [/api/?type=34](/api/?type=34)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>dbHandle</td><td>ture</td><td>int</td><td>句柄</td></tr><tr><td>sql</td><td>ture</td><td>string</td><td>sql 语句</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr><tr><td>data</td><td>array</td><td>返回数据</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "dbHandle": 219277920,
    "sql":"select * from MSG where MsgSvrID=8985035417589024392"
}
```

响应：

```
{"code":1,"data":[["localId","TalkerId","MsgSvrID","Type","SubType","IsSender","CreateTime","Sequence","StatusEx","FlagEx","Status","MsgServerSeq","MsgSequence","StrTalker","StrContent","DisplayContent","Reserved0","Reserved1","Reserved2","Reserved3","Reserved4","Reserved5","Reserved6","CompressContent","BytesExtra","BytesTrans"],["6346","24","8985035417589024392","1","0","0","1670897832","1670897832000","0","0","2","1","778715089","wxid_1222","112","","0","2","","","","","","","CgQIEBAAGkEIBxI9PG1zZ3NvdXJjZT4KCTxzaWduYXR1cmU+djFfSFFyeVAwZTE8L3NpZ25hdHVyZT4KPC9tc2dzb3VyY2U+ChokCAISIDU5NjI1NjUxNWE0YzU2ZDQxZDJlOWMyYmIxMjFhNmZl",""]],"result":"OK"}
```

#### 44. 退出登录 **

###### 接口功能

> 退出登录微信，相当于直接退出微信，跟手动退出比，少了重新打开登录的一步，dll 注入后也会随微信关闭而关闭。调用后不能再继续操作 dll。

###### 接口地址

> [/api/?type=44](/api/?type=44)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 非 0 成功</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

响应：

```
{"code":4344,"result":"OK"}
```

#### 46. 联系人列表 **

###### 接口功能

> 联系人列表

###### 接口地址

> [/api/?type=46](/api/?type=46)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr><tr><td>data</td><td>array</td><td>返回内容</td></tr><tr><td>customAccount</td><td>string</td><td>自定义账号</td></tr><tr><td>delFlag</td><td>int</td><td>删除标志</td></tr><tr><td>type</td><td>int</td><td>好友类型</td></tr><tr><td>userName</td><td>string</td><td>用户名称</td></tr><tr><td>verifyFlag</td><td>int</td><td>验证</td></tr><tr><td>wxid</td><td>string</td><td>wxid</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "wxid": "filehelper",
    "msgid":7215505498606506901
}
```

响应：

```
{"code":1,"data":[{"customAccount":"custom","delFlag":0,"type":8388611,"userName":"昵称","verifyFlag":0,"wxid":"wxid_123pcqm22"}]}
```

#### 47. 群详情 **

###### 接口功能

> 获取群详情

###### 接口地址

> [/api/?type=47](/api/?type=47)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>chatRoomId</td><td>ture</td><td>string</td><td>群 id</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr><tr><td>data</td><td>object</td><td>返回内容</td></tr><tr><td>admin</td><td>string</td><td>群主 id</td></tr><tr><td>chatRoomId</td><td>int</td><td>群 id</td></tr><tr><td>notice</td><td>int</td><td>通知</td></tr><tr><td>xml</td><td>string</td><td>xml</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "wxid": "filehelper",
    "msgid":7215505498606506901
}
```

响应：

```
{"code":1,"data":{"admin":"123","chatRoomId":"123@chatroom","notice":"1222","xml":""},"result":"OK"}
```

#### 48. 获取解密图片 **

###### 接口功能

> 获取解密图片

###### 接口地址

> [/api/?type=48](/api/?type=48)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>imagePath</td><td>ture</td><td>string</td><td>图片路径</td></tr><tr><td>savePath</td><td>ture</td><td>string</td><td>保存路径</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, 0 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "imagePath":"C:\\3a610d7bc1cf5a15d12225a64b8962.dat",
    "savePath":"C:\\other"
}
```

响应：

```
{"code":1,"result":"OK"}
```

#### 49. 提取文字 **

###### 接口功能

> 提取图片中的文字

###### 接口地址

> [/api/?type=49](/api/?type=49)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>imagePath</td><td>ture</td><td>string</td><td>图片路径</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 0 成功, -1 失败，1 2 则是缓存或者正在进行中需再调用一次</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr><tr><td>text</td><td>string</td><td>提取的相应文字</td></tr></tbody></table>

#### 50. 拍一拍 **

###### 接口功能

> 群里拍一拍用户

###### 接口地址

> [/api/?type=50](/api/?type=50)

###### HTTP 请求方式

> POST  JSON

###### 请求参数

<table><thead><tr><th>参数</th><th>必选</th><th>类型</th><th>说明</th></tr></thead><tbody><tr><td>chatRoomId</td><td>ture</td><td>string</td><td>微信群聊 id</td></tr><tr><td>wxid</td><td>ture</td><td>string</td><td>要拍的用户 wxid，如果使用用户自定义的微信号，则不会显示群内昵称</td></tr></tbody></table>

###### 返回字段

<table><thead><tr><th>返回字段</th><th>字段类型</th><th>说明</th></tr></thead><tbody><tr><td>code</td><td>int</td><td>返回状态, 1 成功, -1 失败</td></tr><tr><td>result</td><td>string</td><td>成功提示</td></tr></tbody></table>

###### 接口示例

入参：

```
{
    "chatRoomId":"123331@chatroom",
    "wxid":"wxid_123456"
}
```

响应：

```
{"code":1,"result":"OK"}
```

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[hook_wechat.zip](forum.php?mod=attachment&aid=MjU4NzgyNnw1NWExYTE3OXwxNjc1MzI4ODk2fDkzMDQwMnwxNzQxMTY4)

2023-2-2 15:40 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB  

99.86 KB, 下载次数: 6, 下载积分: 吾爱币 -1 CB

编译好的 dll

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[hook_wechat-3.8.1.26-V3.zip](forum.php?mod=attachment&aid=MjU4NzgyNXxlMDM1NzdkY3wxNjc1MzI4ODk2fDkzMDQwMnwxNzQxMTY4)

2023-2-2 15:39 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB  

164.96 KB, 下载次数: 7, 下载积分: 吾爱币 -1 CB

源码 + 注入器 + TCPserver(python)![](https://avatar.52pojie.cn/images/noavatar_middle.gif)2513002960 大佬牛逼！![](https://avatar.52pojie.cn/images/noavatar_middle.gif)odmin python client test !  
[Python] _纯文本查看_ _复制代码_

```
LOCAL_IP = '192.168.1.102'  # 本地IP
 
def hook_msg():
    # hook 消息
    url = f"http://{WECHAT_IP}:19088/api/?type=9"
    data = {
        'port': "19099",
        'ip': LOCAL_IP
    }
    resp = requests.post(url, json=data)
    print(resp.json())
 
 
if __name__ == '__main__':
    print("Server Start : " + LOCAL_IP)
    # 发送消息
    send_msg('Hello! ', 'wxid')
```

![](https://attach.52pojie.cn/forum/202302/02/160223y44phi1mhmpmpz3f.png)

**123.png** _(49.01 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjU4NzgzM3w4ZjUyYjM1YnwxNjc1MzI4ODk2fDkzMDQwMnwxNzQxMTY4&nothumb=yes)

2023-2-2 16:02 上传![](https://avatar.52pojie.cn/images/noavatar_middle.gif)无夜滴滴 膜拜大佬~~![](https://static.52pojie.cn/static/image/smiley/mogu/dyj.gif)![](https://avatar.52pojie.cn/data/avatar/000/14/22/63_avatar_middle.jpg)dayer hook 图片没啥效果啊？还有群里面接收的图片能定位到哪个人发的吗？