
<!-- vim-markdown-toc GitLab -->

* [Xpra-Controller 程序 Json 数据交换协议](#xpra-controller-程序-json-数据交换协议)
  * [Q&A](#qa)
  * [一、格式：](#一格式)
    * [1.1 指令数据包（command packet）](#11-指令数据包command-packet)
      * [1.1.1 启动 MIV](#111-启动-miv)
      * [1.1.2 关闭 MIV](#112-关闭-miv)
      * [1.1.3 切换病人图像](#113-切换病人图像)
      * [1.1.4 请求触发事件](#114-请求触发事件)
    * [1.2 回复数据包（reply packet）](#12-回复数据包reply-packet)
    * [1.3 消息数据包（message packet）](#13-消息数据包message-packet)
      * [MIV 异常关闭](#miv-异常关闭)
      * [广播消息](#广播消息)
  * [二、数据包类型](#二数据包类型)
  * [三、指令类型](#三指令类型)
  * [三、事件类型](#三事件类型)
  * [四、错误码](#四错误码)

<!-- vim-markdown-toc -->

# Xpra-Controller 程序 Json 数据交换协议

## Q&A

* **Q：如何使用三维重建服务？**  
  A：通过调用 HTTP 或 Socket 接口执行以下流程：启动 MIV -> 通过返回的URL打开该页面 -> 发送指令切换浏览模式或切换患者图像

* **Q：如何测试？用什么服务器测试？**  
  A：公司内部 Http 协议可使用服务器：`http://10.68.137.21:12002`，Socket 协议使用：`tcp://10.68.137.21:3008`。

## 一、格式：

若使用 HTTP 协议，以下接口调用全部为`application/json` 格式的正文，请求方法为`POST`，`charset` 为 `UTF-8`。

HTTP 协议请忽略回复数据包。

### 1.1 指令数据包（command packet）

#### 1.1.1 启动 MIV

由客户端发出，服务端处理，用于启动云端 MIV。

* 参数：
  
  **packet_id : string**  
  *这个数据包的标识，一个不重复id*  
  
  **type : string**  
  *数据包类型，详见列表 2.1*  
  
  **reply : bool**  
  *是否返回回复数据包*
  
  **data : object**  
  *数据体*
  
  **command : string**  
  *指令类型，详见列表 3.1*  
  
  **user_token : string**  
  *用户token，标识用户*  
  
  **session_id : string**  
  *会话token，标识会话（session），即 http 的 Session_id*  
  
  **user_name : string**  
  *用户名，可选*

  **scale_factor : double**  
  *缩放比例，可选*  
  *值通常为 dpi / 96，默认为 1*

  **hospital_id : string**  
  *医院 id*

  **organization_id : string**  
  *机构 id*

  **host : string**  
  *云域名*

示例：
```
{
  "packet_id" : "uuiduuiduuiduuid",
  "type" : "command",
  "reply" : false, 
  "data" : {
    "command" : "start", 
    "data" : {
      "user_token" : "d26b65b5ec8a5d54e99f5159a9b04919",
      "session_id" : "f0d64d6cc4a53dc3eff8a9ccd2879dba",
      "user_name" : "Admin",
      "scale_factor" : 1.5,
      "hospital_id": "a53dc3eff8a9cc",
      "organization_id": "3effccaoeu1i45",
      "host": "pdrmyy.msunhis.com"
    }
  }
}
```

* 返回数据包：
  
  返回一个 reply 类型数据包，详见 1.2。

* 注意：

  当接口返回数据时，会返回参数 `display`，将该参数通过以下方式拼接，可获得一个 URL，通过该 URL 可打开 MIV 页面。  
  拼接规则：
  ```
  http://[ip]:[port]/index.html?username=msun&password0=[display]&password1=[display]&display=[display]&encoding=jpg&clipboard=true&keyboard=false&notifications=false&tray=false&video=true&sound=false&floating_menu=false
  ```


#### 1.1.2 关闭 MIV

由客户端发出，服务端处理，用于关闭云端 MIV。

* 参数：
  
  **packet_id : string**  
  *这个数据包的标识，一个不重复id*  
  
  **type : string**  
  *数据包类型*  
  
  **reply : bool**  
  *是否返回回复数据包*
  
  **data : object**  
  *数据体*
  
  **command : string**  
  *指令类型，详见列表 3.1*  
  
  **session_id : string**  
  *会话token，对应会话，即http Session_id*  


示例：
```
{
  "packet_id" : "847306e59b2868b4ba6248384075adb9",
  "type" : "command",
  "reply" : false, 
  "data" : {
    "command" : "stop", 
    "data" : {
      "session_id" : "f0d64d6cc4a53dc3eff8a9ccd2879dba"
    }
  }
}
```

* 返回数据包：
  
  返回一个 reply 类型数据包，详见 1.2。  


#### 1.1.3 切换病人图像 

由客户端发出，服务端处理，用于更新云端 MIV 图像。

* 参数：
  
  **packet_id : string**  
  *这个数据包的标识，一个不重复id*  
  
  **type : string**  
  *数据包类型*  
  
  **reply : bo**ol
  *是否返回回复数据包*
  
  **data : object**  
  *数据体*
  
  **command : string**  
  *指令类型，详见列表 3.1*  
  
  **session_id : string**  
  *会话token，对应会话，即http Session_id*  
  
  **type : string**  
  *dicom 图像获取途径，可以是文件夹和文件本地打开，也可以是Query/Retrieve，可能的值：files/folders/QueryRetrieve*  

    > Query/Retrieve  
    > 通过 QR 的方法获取图像。  
    >  
    > files/folders  
    > 通过文件或文件夹的方法获取路径，可以传入多个路径。  
  
  **ip : string**  
  *QueryRetrieve 地址*  
  
  **port : string**  
  *QueryRetrieve 端口*  

  **client_ae : string**  
  *QueryRetrieve Client AE*  

  **server_ae : string**  
  *QueryRetrieve Server AE*  

  **memo : string**  
  *QueryRetrieve Memo*  

  **http_address : string**  
  *MIV Http服务器，格式：[IP]:[PORT]*  
    
    > 通常为 PACS 服务器，可联系 PACS 获取。  
  
  **patient_id : string**  
  *病人ID*  
  
  **study_uid : string**  
  *study instance id*  
  
  **reg_id : string**  
  *检查登记ID*  
  
    > PACS 系统内部 ID,联系 PACS 获取。  

  **browse_mode : string**  
  *浏览模式*  
  
  **user_sys_id : string**  
  *用户ID*  
  
  **can_print : string**  
  *是否允许打印*  
  
  **hospital_id : string**  
  *医院ID*  
    
    > 联系 PACS 获取。  

  **is_clinic : string**  
  *是否是临床*  
  
    > 非临床均传入 0。  

  **card_code : string**  
  *就诊卡号*  

    > 联系 PACS 获取。  
  
  **mode : string**  
  *图像数据追加模式*  
  *可能的参数为"append"或者"new"，前者为追加模式，后者为清理再浏览*  

  > append  
  > 不清空当前正在浏览的图像，追加加载新的图像。  
  >
  > new  
  > 清空当前浏览的图像，加载新的图像。
  

示例：
```
示例一：

{
  "packet_id" : "847306e59b2868b4ba6248384075adb9",
  "type" : "command",
  "reply" : false, 
  "data" : {
    "command" : "update",
    "data" : {
      "session_id"     : "d26b65b5ec8a5d54e99f5159a9b04919",
      "type"           : "files",
      "data" : {
        "path" : ["/path/to/file/1", "/path/to/file/2", "/path/to/file/3"] 
      },
      "http_addr"      : "192.168.1.1:1333",
      "patient_id"     : "idididididid",
      "study_uid"      : "idididididid",
      "reg_id"         : "idididididid",
      "browse_mode"    : "0",
      "user_sys_id"    : "idididididid",
      "can_print"      : "0",
      "hospital_id"    : "idididididid",
      "is_clinic"      : "0",
      "card_code"      : "idididididid",
      "mode"           : "new"
    }
  }
}

示例二：

{
  "packet_id" : "847306e59b2868b4ba6248384075adb9",
  "type" : "command",
  "reply" : false, 
  "data" : {
    "command" : "update",
    "data" : {
      "session_id"     : "d26b65b5ec8a5d54e99f5159a9b04919",
      "type"           : "folders",
      "data" : {
        "path" : ["/path/to/folder/1", "/path/to/folder/2", "/path/to/folder/3"] 
      },
      "http_addr"      : "192.168.1.1:1333",
      "patient_id"     : "idididididid",
      "study_uid"      : "idididididid",
      "reg_id"         : "idididididid",
      "browse_mode"    : "0",
      "user_sys_id"    : "idididididid",
      "can_print"      : "0",
      "hospital_id"    : "idididididid",
      "is_clinic"      : "0",
      "card_code"      : "idididididid",
      "mode"           : "new"
    }
  }
}

示例三：

{
  "packet_id" : "847306e59b2868b4ba6248384075adb9",
  "type" : "command",
  "reply" : false, 
  "data" : {
    "command" : "update",
    "data" : {
      "session_id"     : "d26b65b5ec8a5d54e99f5159a9b04919",
      "type"           : "QueryRetrieve",
      "data" : {
        "ip" : "1.1.1.1",
        "port" : "1234",
        "client_ae" : "pacs",
        "server_ae" : "pacs_server",
        "memo" : "test"
      },
      "http_addr"      : "192.168.1.1:1333",
      "patient_id"     : "idididididid",
      "study_uid"      : "idididididid",
      "reg_id"         : "idididididid",
      "browse_mode"    : "0",
      "user_sys_id"    : "idididididid",
      "can_print"      : "0",
      "hospital_id"    : "idididididid",
      "is_clinic"      : "0",
      "card_code"      : "idididididid",
      "mode"           : "new"
    }
  }
}
```

* 返回数据包：
  
  返回一个 reply 类型数据包，详见 1.2。


#### 1.1.4 请求触发事件  

由客户端发出，服务端处理，用于触发影像浏览器事件。

* 参数：
  
  **packet_id : string**  
  *这个数据包的标识，一个不重复id*  

  **type : string**  
  *数据包类型，值为：command*  

  **reply : bool**  
  *是否返回回复数据包*  

  **data : object**  
  *数据体*  

  **command : string**  
  *命令类型，值为：ActionToggle*  

  **action : string**  
  *事件名称*  
  详见列表 4.1


示例：
```
{
  "packet_id" : "847306e59b2868b4ba6248384075adb9",
  "type" : "command",
  "reply" : false,
  "data" : {
    "command" : "ActionToggle",
    "data" : {
      "action" : "ModeAuto3D",
      "session_id" : "d26b65b5ec8a5d54e99f5159a9b04919"
    }
  }
}
```


* 返回数据包：
  
  返回一个 reply 类型数据包，详见 1.2。


### 1.2 回复数据包（reply packet）

仅对 Socket 通信方式有效。
仅在启用回复数据包时发送，即"reply"等于true时，格式如下：

* 参数：
  
  **type : string**  
  *数据包类型*  
  
  **result : string**  
  *返回结果，可以是两种值："success"或"fail"*  

  **error : int**  
  *错误码，见表 4.1*
  
  **packet_id : string**  
  *该回复数据包对应的 packet，表示该数据包回复的是之前该 id 的结果*
  
  **message : string**  
  *字符串消息，描述结果，例如如果失败，则描述失败原因*  


示例：
```
{
  "type" : "reply",
  "packet_id" : "847306e59b2868b4ba6248384075adb9",
  "data" : {
    "error" : 0, 
    "result" : "success", 
    "display" : 1001,
    "message" : "This is a message to describe what is the result or happened when request packet processed."
  }

}
```


### 1.3 消息数据包（message packet）

一次性消息数据包，用于传递消息。

#### MIV 异常关闭

在该应用场景下，由服务器发往客户端，通知客户端 MIV 异常关闭（通常是服务器或 MIV 自己的原因），该类型数据包不会返回回复数据包。

* 参数
  
  **type : string**  
  *数据包类型*  

  **data : object**  
  *数据体*  

  **session_id : string**  
  *MIV 异常退出对应的 session id*  


示例：
```
{
  "type" : "message",
  "data" : {
    "message": "miv_closed",
    "data" :{
      "session_id" : "5b2cac76836e7acdc3c33269cf27be06"
    }
  }
}
```


#### 广播消息

向服务器发送广播消息，广播消息将显示在当前服务器上的所有打开的影像浏览器上。

* 参数

  **type : string**  
  *数据包类型*  

  **data : object**  
  *数据体*  

  **session_id : string**  
  *会话token，对应会话，即http Session_id*  

  **title : string**  
  *标题内容*  

  **date : string**  
  *日期*  

  **from : string**  
  *发送人*  

  **message : string**  
  *广播消息内容*  

示例：
```
{
  "type" : "message",
  "data" : {
    "message": "broadcast",
    "data" :{
      "session_id" : "5b2cac76836e7acdc3c33269cf27be06",
      "title" : "标题", 
      "date" : "2020-01-01", 
      "from" : "管理员", 
      "message" : "你好"
    }
  }
}
```



## 二、数据包类型

| type    | 描述                         |
|---------|------------------------------|
| command | MIV control command request  |
| reply   | Reply when processed request |
| message | One-time message             |


## 三、指令类型

| command      | 描述     |
|--------------|----------|
| start        | 启动 MIV |
| stop         | 停止 MIV |
| update       | 加载图像 |
| ActionToggle | 触发事件 |

## 三、事件类型
| Action              | 描述                                                            | 工具栏         | 类型     |
|---------------------|-----------------------------------------------------------------|----------------|----------|
| ModeAuto3D          | 3D Panel 模式，此模式下加载的图像将自动三维重建                 | 包含完整工具栏 | MIV 模式 |
| ModeSingle3d        | 3D 独立模式，此模式下加载的图像将自动三维重建                   | 无工具栏       | MIV 模式 |
| ModeSingle3dMip     | 3D 独立 Mip 模式，此模式下加载图像自动三维重建并进入 Mip 模式   | 无工具栏       | MIV 模式 |
| ModeSingle3dMinp    | 3D 独立 Minp 模式，此模式下加载图像自动三维重建并进入 Minp 模式 | 无工具栏       | MIV 模式 |
| ActionFocusViewMip  | 将当前窗口切换到 MIP 模式，仅对当前图像生效                     |                | 渲染类型 |
| ActionFocusViewMinp | 将当前窗口切换到 MinP 模式，金对当前图像生效                    |                | 渲染类型 |

## 四、错误码

| error code | 描述               |
|------------|--------------------|
| 0          | 成功               |
| -1         | 启动失败           |
| -2         | 进程不存在或无相应 |
| -3         | 进程不存在         |
| -9         | json 解析失败      |
| 1          | 进程已存在         |

