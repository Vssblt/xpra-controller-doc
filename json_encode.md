
<!-- vim-markdown-toc GitLab -->

* [Xpra-Controller 程序 Json 数据交换协议](#xpra-controller-程序-json-数据交换协议)
	* [一、格式：](#一格式)
		* [1.1 指令数据包（command packet）](#11-指令数据包command-packet)
			* [1.1.1 启动 MIV](#111-启动-miv)
			* [1.1.2 关闭 MIV](#112-关闭-miv)
			* [1.1.3 切换病人图像](#113-切换病人图像)
		* [1.2 回复数据包（reply packet）](#12-回复数据包reply-packet)
		* [1.3 消息数据包（message packet）](#13-消息数据包message-packet)
			* [MIV 异常关闭](#miv-异常关闭)
	* [二、数据包类型](#二数据包类型)
	* [三、指令类型](#三指令类型)
	* [四、错误码](#四错误码)

<!-- vim-markdown-toc -->

# Xpra-Controller 程序 Json 数据交换协议

## 一、格式：

### 1.1 指令数据包（command packet）

#### 1.1.1 启动 MIV

由客户端发出，服务端处理，用于启动云端 MIV。

* 参数：
	
	packet_id : string  
	*这个数据包的标识，一个不重复id*  
	
	type : string  
	*数据包类型，详见列表 2.1*  
	
	reply : bool
	*是否返回回复数据包*
	
	data : object  
	*数据体*
	
	command : string  
	*指令类型，详见列表 3.1*  
	
	user_token : string  
	*用户token，对应用户，可选*  
	
	session_id : string  
	*会话token，对应会话，即http Session_id*  
	
	user_name : string  
	*用户名，可选*

	scale_factor : double  
	*缩放比例，可选*  
	*值通常为 dpi / 96，默认为 1*

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
			"scale_factor" : 1.5
		}
	}
}
```

* 返回数据包：
	
	返回一个 reply 类型数据包，详见 1.2。


#### 1.1.2 关闭 MIV

由客户端发出，服务端处理，用于关闭云端 MIV。

* 参数：
	
	packet_id : string  
	*这个数据包的标识，一个不重复id*  
	
	type : string  
	*数据包类型*  
	
	reply : bool
	*是否返回回复数据包*
	
	data : object  
	*数据体*
	
	command : string  
	*指令类型，详见列表 3.1*  
	
	session_id : string  
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
	
	packet_id : string  
	*这个数据包的标识，一个不重复id*  
	
	type : string  
	*数据包类型*  
	
	reply : bool
	*是否返回回复数据包*
	
	data : object  
	*数据体*
	
	command : string  
	*指令类型，详见列表 3.1*  
	
	session_id : string  
	*会话token，对应会话，即http Session_id*  
	
	path_type : string  
	*dicom 图像路径类型，可以是文件夹，也可以是文件，可能的值：files/folders*  
	
	path : array  
	*dicom 图像的文件或文件夹路径*  
	
	http_address : string  
	*MIV Http服务器，格式：[IP]:[PORT]*  
	
	patient_id : string  
	*病人ID*  
	
	study_uid : string  
	*study instance id*  
	
	reg_id : string  
	*检查登记ID*  
	
	browse_mode : string  
	*浏览模式*  
	
	user_sys_id : string  
	*用户ID*  
	
	can_print : string  
	*是否允许打印*  
	
	hospital_id : string  
	*医院ID*  
	
	is_clinic : string  
	*是否是临床*  
	
	card_code : string  
	*就诊卡号*  
	
	mode : string  
	*图像数据追加模式*  
	*可能的参数为"append"或者"new"，前者为追加模式，后者为清理再浏览*  
	
	alternate_addr : string  
	*PACS1.0 IP，格式为-1或[IP]:[PORT]*  


示例：
```
{
	"packet_id" : "847306e59b2868b4ba6248384075adb9",
	"type" : "command",
	"reply" : false, 
	"data" : {
		"command" : "update",
		"data" : {
			"session_id"     : "d26b65b5ec8a5d54e99f5159a9b04919",
			"path_type"      : "files",
			"path"           : ["/path/to/file/1", "/path/to/file/2", "/path/to/file/3"],
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
			"mode"           : "new",
			"alternate_addr" : "192.168.1.2:5612"
		}
	}
}
```

* 返回数据包：
	
	返回一个 reply 类型数据包，详见 1.2。


### 1.2 回复数据包（reply packet）

仅一种格式，且仅在启用回复数据包时发送，即"reply"等于true时，格式如下：

* 参数：
	
	type : string  
	*数据包类型*  
	
	result : string  
	*返回结果，可以是两种值："success"或"fail"*  

	error : int  
	*错误码，见表 4.1*
	
	packet_id : string
	*该回复数据包对应的 packet，表示该数据包回复的是之前该 id 的结果*
	
	message : string  
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

一次性消息数据包，用于传递消息，在该应用场景下，由服务器发往客户端，通知客户端 MIV 异常关闭（通常是服务器或 MIV 自己的原因），该类型数据包不会返回回复数据包。

#### MIV 异常关闭

* 参数
	
	type : string  
	*数据包类型*  

	data : object  
	*数据体*  

	session_id : string  
	*MIV 异常退出对应的 session id*  


示例：
```
{
	"type" : "message",
	"data" : {
		"session_id" : "5b2cac76836e7acdc3c33269cf27be06"
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

| command | 描述                      |
|---------|---------------------------|
| start   | start a MIV               |
| stop    | stop a MIV                |
| update  | update dicom files to MIV |


## 四、错误码

| error code | 描述               |
|------------|--------------------|
| 0          | 成功               |
| -1         | 启动失败           |
| -2         | 进程不存在或无相应 |
| -3         | 进程不存在         |
| -9         | json 解析失败      |
| 1          | 进程已存在         |

