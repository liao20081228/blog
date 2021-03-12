---
title: MM_CB结构体解析
tags: 4G
---

------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

``` c
typedef struct
{	
	S_MM_CB_COMM		stMmCbComm;//MM模块公有的CB表信息，包含CB用户信息、位置信息，缓存机制等。n
	S_MM_CB_SDB			stMmCbSdb;
	S_MM_CB_MMCTRL			  stMmCbCtrl;   //MM主控模块的CB表信息，包含TAU/DETACH/ATTACH等流程信息?
	S_MM_ACCCONN_INFO		   stMmCbAcc[ACCCONN_CB_BUTT];  // 接入连接管理信息/
	S_MM_PAGING_INFO			stMmCbPaging;				 // 寻呼信息
	S_MM_SM_INFO				stMmCbSm;					 // 接口模块SM相关信息保存: SmCbIdx, SmSdbIdx
	S_MM_UDM_INFO			   stMmCbUdm;					// 接口模块UDM相关信息保存 
	S_MM_SMS_INFO			   stMmCbSms;					// 接口模块SMS相关信息保存 
	S_MM_MBMS_INFO			  stMmCbMbms;					 // 接口模块MBMS相关信息保存S
	S_MM_SECURITY_INFO		  stMmCbSec;			   // 子模块Security相关信息保存C
	//S_MM_CS_INFO				stMmCbCs;					  // 接口模块BSSAP+相关信息保存C
	S_MM_LCS_INFO			   stMmCbLcs;					// 接口模块LCS相关信息保存?
	S_MM_CAMEL_INFO			 stMmCbCamel;				  // 接口模块CAMEL相关信息保存?
	//S_MM_PEERCONN_INFO		  stMmCbPeerConn;			   // 接口模块GTPC相关信息保存?
	S_MM_SRVCC_INFO			 stMmCbSrvcc;
	//Modified for PN:V9R1C1 MessageTrc MST by z59661, 2009/2/20
	S_MM_OM_INFO				stMmOm;					// 维护模块OM相关信息保存
	S_MM_FAM_INFO			   stMmCbfam;				   // Fam 模块相关信息保存
	S_MM_CSSERV_INFO			stMmCbCsserv;				   // CSServ子模块相关信息保存

	/*added by lirongguo wx44965 for TTR1.1 eSCN231 MM 数据模型相关, 2012-7-4, begin*/
	S_MM_PTT_INFO			   stPttInfo;				   /* 存储PTT信息 */
	/*added by lirongguo wx44965 for TTR1.1 eSCN231 MM 数据模型相关, 2012-7-4, end*/   
}VOS_PACKED S_MM_CB;
```


------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
