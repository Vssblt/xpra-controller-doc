
<!-- vim-markdown-toc GitLab -->

* [Xpra-Controller 程序 Json 数据交换协议](#xpra-controller-程序-json-数据交换协议)
    * [Q&A](#qa)
    * [一、格式：](#一格式)
        * [1.1 指令数据包（command packet）](#11-指令数据包command-packet)
            * [1.1.1 启动 MIV](#111-启动-miv)
            * [1.1.2 关闭 MIV](#112-关闭-miv)
            * [1.1.3 加载患者图像](#113-加载患者图像)
            * [1.1.4 触发事件](#114-触发事件)
        * [1.2 回复数据包（reply packet）](#12-回复数据包reply-packet)
    * [二、数据包类型](#二数据包类型)
        * [2.1 数据包类型](#21-数据包类型)
    * [三、指令类型](#三指令类型)
        * [3.1 指令类型](#31-指令类型)
    * [四、事件类型](#四事件类型)
        * [4.1 事件类型](#41-事件类型)
    * [五、错误码](#五错误码)
        * [5.1 错误码](#51-错误码)

<!-- vim-markdown-toc -->

# Xpra-Controller 程序 Json 数据交换协议

## Q&A

* **Q：如何使用三维重建服务？**  
  A：通过调用 HTTP 或 Socket 接口执行以下流程：启动 MIV -> 通过返回的URL打开页面 -> 发送指令切换浏览模式（可选） -> 加载患者图像 -> 关闭 MIV

* **Q：如何测试？用什么服务器测试？**  
  A：公司内部 Http 协议可使用服务器：`http://10.68.137.21:12002`，Socket 协议使用：`tcp://10.68.137.21:3008`。

## 一、格式：

若使用 HTTP 协议，以下接口调用全部为`application/json` 格式的正文，请求方法为`POST`，`charset` 为 `UTF-8`。

### 1.1 指令数据包（command packet）

#### 1.1.1 启动 MIV

由客户端发出，服务端处理，用于启动云端 MIV。

* 参数：

| 参数名                    | 类型   | 详情                                                                                                       |
|---------------------------|--------|------------------------------------------------------------------------------------------------------------|
| type                      | string | 数据包类型，详见列表 2.1                                                                                   |
| data                      | object | 数据体                                                                                                     |
| data.command              | string | 指令类型，详见列表 3.1                                                                                     |
| data.data                 | object | 指令数据                                                                                                   |
| data.data.user_token      | string | 用户token，标识用户，用户 Token 是用户的唯一标识，任意值，但不同用户之间不能重复                           |
| data.data.session_id      | string | 会话token，标识会话（session），任意值，但每次会话不能重复。                                               |
| data.data.user_name       | string | 用户名，可选，用于显示在三维界面上，可以是中文                                                             |
| data.data.scale_factor    | double | 缩放比例，可选，值通常为 dpi / 96，默认为 1，大于 1 三维会放大显示，通常用于高分屏（手机或者苹果电脑等）。 |
| data.data.hospital_id     | string | 医院 ID，医院标识，任意值，不同医院之间不能重复。                                                          |
| data.data.organization_id | string | 机构 ID，机构标识，任意值，不同机构之间不能重复。如果机构是医院本身，可以传医院 ID。                       |

示例：
```
{
  "type" : "command",
  "data" : {
    "command" : "start", 
    "data" : {
      "user_token" : "d26b65b5ec8a5d54e99f5159a9b04919",
      "session_id" : "f0d64d6cc4a53dc3eff8a9ccd2879dba",
      "user_name" : "Admin",
      "scale_factor" : 1.5,
      "hospital_id": "a53dc3eff8a9cc",
      "organization_id": "3effccaoeu1i45"
    }
  }
}
```

* 返回数据包：

  返回一个 reply 类型数据包，详见 1.2。

* 注意：

  当接口返回数据时，会返回参数 `display`，将该参数通过以下方式拼接，可获得一个 URL，该 URL 60 秒内有效，通过该 URL 可打开三维阅片页面。  
  拼接规则：
  ```
  http://[ip]:12008/index.html?username=msun&password0=[display]&password1=[display]&display=[display]&encoding=jpeg&clipboard=true&keyboard=false&notifications=false&tray=false&video=true&sound=false&floating_menu=false
  ```

* 注意：

根据 error 返回的值不同，会有不同的后续处理方式。
- 如果 error 返回 -1，则发生错误，需要检查参数是否正确。
- 如果 error 等于 0，则需要打开返回的 url 连接。
- 如果 error 等于 1，说明当前服务端已存在启动的 MIV，通常是因上次打开未正确关闭服务端程序。这种情况下应分类讨论，如果当前已经通过 url 打开了页面，则不需要做额外的事情，你可以继续执行后续指令；如果当前 url 页面没有打开，则需要立刻通过 url 打开该三维页面。


#### 1.1.2 关闭 MIV

由客户端发出，服务端处理，用于关闭云端 MIV。

* 参数：

| 参数名          | 类型   | 详情                                   |
|-----------------|--------|----------------------------------------|
| type            | string | 数据包类型                             |
| data            | object | 数据体                                 |
| data.command    | string | 指令类型，详见列表 3.1                 |
| data.data       | object | 指令数据                               |
| data.session_id | string | 会话token，对应会话，即http Session_id |

示例：
```
{
  "type" : "command",
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


#### 1.1.3 加载患者图像 

由客户端发出，服务端处理，用于更新云端 MIV 图像。

* 参数：

| 参数名          | 类型   | 详情                                                                                                                                                                                           |
|-----------------|--------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type            | string | 数据包类型                                                                                                                                                                                     |
| data            | object | 数据体                                                                                                                                                                                         |
| data.command    | string | 指令类型，详见列表 3.1                                                                                                                                                                         |
| data.session_id | string | 会话token，对应会话，即http Session_id                                                                                                                                                         |
| data.type       | string | 图像来源类型，dicom 图像获取途径，可以是服务端文件夹和文件，也可以是Query/Retrieve（QR 协议，一种 Dicom 数据传输协议），还可以是 http 链接。可能的值：files/folders/QueryRetrieve/httpDownload |
| data.data       | object | 图像源的参数列表，根据上一个参数的值选择下列表格中的参数格式                                                                                                                                   |
| data.http_addr | string | 图灵医疗影像中心地址，例如：10.10.10.10:13333                                                                                                                                                   |
| data.patient_id | string | 患者 ID                                                                                                                                                                                        |
| data.study_uid  | string | study instance id                                                                                                                                                                              |
| data.mode       | string | 图像数据浏览模式，可能的参数为"append"或者"new"，append 不清空当前正在浏览的三维图像，直接加载新的图像。new 先清空当前正在浏览的图像，再加载新的图像。                                         |

* 如果 data.type 的值是 QueryRetrieve，data.data 使用以下参数列表

| 参数名              | 类型   | 详情                    |
|---------------------|--------|-------------------------|
| data.data.ip        | string | QueryRetrieve 地址      |
| data.data.port      | string | QueryRetrieve 端口      |
| data.data.client_ae | string | QueryRetrieve Client AE |
| data.data.server_ae | string | QueryRetrieve Server AE |
| data.data.memo      | string | QueryRetrieve Memo      |

* 如果 data.type 的值是 files，data.data 使用以下参数列表

| 参数名       | 类型   | 详情                                                                                                                       |
|--------------|--------|----------------------------------------------------------------------------------------------------------------------------|
| data.data.path           | array | 一个字符串数组，包含一个或多个文件地址                                                                                                         |

* 如果 data.type 的值是 foladers，data.data 使用以下参数列表

| 参数名       | 类型   | 详情                                                                                                                       |
|--------------|--------|----------------------------------------------------------------------------------------------------------------------------|
| data.data.path           | array | 一个字符串数组，包含一个或多个文件夹地址                                                                                                         |

* 如果 data.type 的值是 httpDownload，data.data 使用以下参数列表

| 参数名       | 类型   | 详情                                                                                                                       |
|--------------|--------|----------------------------------------------------------------------------------------------------------------------------|
| data.data.uri           | array | 一个字符串数组，包含一个或多个图像文件的Http链接                                                                                                         |

  

示例：
```
示例一：

{
  "type" : "command",
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
      "mode"           : "new"
    }
  }
}

示例二：

{
  "type" : "command",
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
      "mode"           : "new"
    }
  }
}

示例三：

{
  "type" : "command",
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
      "mode"           : "new"
    }
  }
}

示例(HTTP)：
{
  "type" : "command",
  "data" : {
    "command" : "update",
    "data" : {
      "session_id"     : "d26b65b5ec8a5d54e99f5159a9b04919",
      "type"           : "HttpDownload",
      "data" : {
        "uri" : ["http://1.1.1.1/1.dcm", "http://1.1.1.1/2.dcm", "http://1.1.1.1/3.dcm"] 
      },
      "http_addr"      : "192.168.1.1:1333",
      "patient_id"     : "idididididid",
      "study_uid"      : "idididididid",
      "mode"           : "new"
    }
  }
}
```

* 返回数据包：
  
  返回一个 reply 类型数据包，详见 1.2。


#### 1.1.4 触发事件  

由客户端发出，服务端处理，用于触发影像浏览器事件。

* 参数：

| 参数名               | 类型   | 详情                                                  |
|----------------------|--------|-------------------------------------------------------|
| type                 | string | 数据包类型，值为：command                             |
| data                 | object | 数据体                                                |
| data.command         | string | 命令类型，值为：ActionToggle                          |
| data.action          | string | 事件名称，详见列表 4.1                                |
| data.data.session_id | string | 会话token，标识会话（session），即 http 的 Session_id |


示例：
```
{
  "type" : "command",
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

* 参数：

| 参数名       | 类型    | 详情                                                            |
|--------------|---------|-----------------------------------------------------------------|
| type         | string  | 数据包类型                                                      |
| data         | object  | 数据体                                                          |
| data.error   | int     | 错误码，见表 4.1                                                |
| data.result  | string  | 返回结果，可以是两种值："success"或"fail"                       |
| data.message | string  | 字符串消息，描述结果，例如如果失败，则描述失败原因              |
| data.display | integer | 显示端口号                                                      |


示例：
```
{
  "type" : "reply",
  "data" : {
    "error" : 0, 
    "result" : "success", 
    "display" : 1001,
    "message" : "This is a message to describe what is the result or happened when request packet processed."
  }
}
```


## 二、数据包类型

### 2.1 数据包类型

| type    | 描述                         |
|---------|------------------------------|
| command | MIV control command request  |
| reply   | Reply when processed request |
| message | One-time message             |


## 三、指令类型

### 3.1 指令类型

| command      | 描述     |
|--------------|----------|
| start        | 启动 MIV |
| stop         | 停止 MIV |
| update       | 加载图像 |
| ActionToggle | 触发事件 |

## 四、事件类型

### 4.1 事件类型

| Action              | 描述                                                            | 工具栏         | 类型     |
|---------------------|-----------------------------------------------------------------|----------------|----------|
| ModeAuto3D          | 3D Panel 模式，此模式下加载的图像将自动三维重建                 | 包含完整工具栏 | MIV 模式 |
| ModeSingle3d        | 3D 独立模式，此模式下加载的图像将自动三维重建                   | 无工具栏       | MIV 模式 |
| ModeSingle3dMip     | 3D 独立 Mip 模式，此模式下加载图像自动三维重建并进入 Mip 模式   | 无工具栏       | MIV 模式 |
| ModeSingle3dMinp    | 3D 独立 Minp 模式，此模式下加载图像自动三维重建并进入 Minp 模式 | 无工具栏       | MIV 模式 |
| ActionFocusViewMip  | 将当前窗口切换到 MIP 模式，仅对当前图像生效                     |                | 渲染类型 |
| ActionFocusViewMinp | 将当前窗口切换到 MinP 模式，金对当前图像生效                    |                | 渲染类型 |

## 五、错误码

### 5.1 错误码

| error code | 描述               |
|------------|--------------------|
| 0          | 成功               |
| -1         | 启动失败           |
| -2         | 进程不存在或无相应 |
| -3         | 进程不存在         |
| -9         | json 解析失败      |
| 1          | 进程已存在         |

