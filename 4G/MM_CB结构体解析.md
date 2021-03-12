---
title: MM_CB结构体解析
tags: 4G
---

------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 S_MM_CB结构体
``` c
//MM模块的上下文
typedef struct
{
	S_MM_CB_COMM			stMmCbComm;					// MM模块公有的CB表信息，包含CB用户信息、位置信息，缓存机制等
	S_MM_CB_SDB				stMmCbSdb;
	S_MM_CB_MMCTRL			stMmCbCtrl;					// MM主控模块的CB表信息，包含TAU/DETACH/ATTACH等流程信息
	S_MM_ACCCONN_INFO		stMmCbAcc[ACCCONN_CB_BUTT];	// 接入连接管理信息/
	S_MM_PAGING_INFO		stMmCbPaging;				// 寻呼信息
	S_MM_SM_INFO			stMmCbSm;					// 接口模块SM相关信息保存: SmCbIdx, SmSdbIdx
	S_MM_UDM_INFO			stMmCbUdm;					// 接口模块UDM相关信息保存 
	S_MM_SMS_INFO			stMmCbSms;					// 接口模块SMS相关信息保存 
	S_MM_MBMS_INFO			stMmCbMbms;					// 接口模块MBMS相关信息保存S
	S_MM_SECURITY_INFO		stMmCbSec;					// 子模块Security相关信息保存C
	//S_MM_CS_INFO			stMmCbCs;					// 接口模块BSSAP+相关信息保存C
	S_MM_LCS_INFO			stMmCbLcs;					// 接口模块LCS相关信息保存?
	S_MM_CAMEL_INFO			stMmCbCamel;				// 接口模块CAMEL相关信息保存?
	//S_MM_PEERCONN_INFO	stMmCbPeerConn;				// 接口模块GTPC相关信息保存?
	S_MM_SRVCC_INFO			stMmCbSrvcc;
	S_MM_OM_INFO			stMmOm;						// 维护模块OM相关信息保存
	S_MM_FAM_INFO			stMmCbfam;					// Fam 模块相关信息保存
	S_MM_CSSERV_INFO		stMmCbCsserv;				// CSServ子模块相关信息保存
	S_MM_PTT_INFO			stPttInfo;					// 存储PTT信息
}VOS_PACKED S_MM_CB;
```

# 2 S_MM_CB_COMM结构体

``` c
//CB表公共部分
typedef struct
{
	IMSI							Imsi;							// MM用户IMSI标识
	IMEI							Imei;							// MM用户IMEI标识
	TMSI							Tmsi;							// MM用户TMSI
	VOS_UINT8						ucTmsiStat;						// TMSI status 
	TMGI							Tmgi;
	PTMSI							Ptmsi;							// MM用户当前P-TMSI标识
	PTMSISIGN						PtmsiSign;						// MM用户当前P-TMSI签名标识
	PTMSI							OldPtmsi;						// MM用户old P-TMSI标识
	PTMSISIGN						OldPtmsiSign;					// MM用户old P-TMSI签名标识

	S_GUTI							CurrentGuti;					// USN_SIG_MM: 可以和P-TMSI合一，使用联合?
	S_GUTI							OldGuti;						// USN_SIG_MM: 可以和Old P-TMSI合一，使用联合?
	S_GUTI							AdditionalGuti;					// 附加GUTI

	VOS_UINT8						ucFollowOnReq;					/*是否有后续信令和数据指示标志 */
	VOS_UINT16						uwReadyTimerVal;				/*Requested READY timer value*/
	VOS_UINT8						bt1IsReadyTimerIncluded:1;		/*用户附着请求中是否携带ReadyTimer长度 */
	VOS_UINT8						bt1TmsiReallocFlag:1;			/*是否已TMSI重分配标志*/
	VOS_UINT8						bt1PtmsiReallocFlag:1;			/*是否已P-TMSI重分配标志*/
	VOS_UINT8						bt1SdbRelocPtmsi:1;				/*SDB发起PTMSI分配*/
	VOS_UINT8						bt1ImsiInTmsiRealloc:1;			/*位置更新过程中，VLR送imsi，要求邋MS删除旧TMSI*/
	VOS_UINT8						bt1FlexUser:1;
	VOS_UINT8						bt1IsS6Available:1;				/*用户是否可以使用S6d接口*/
	VOS_UINT8						btMmComCbReserved:1;

	VOS_UINT8						ucMsIdType;						// MM用户标识的类型: IMSI, P-TMSI, GUTI
	VOS_UINT8						ucSystemType;
	
	S_USN_CAUSE_INF					stInnerCause;

	RAI								TargetRai;						// MM用户所在目标路由区 
	RAI								OldRai;							// MM用户所在 old 路由区 
	RAI								SuspendRai;						//suspend流程中的RAI
	//SAI							CurrentSai;						// MM用户当前所在服务区, 仅在3G时使用 
	LAI								OldLocation;					// Old location area identification 
	TAI								CurrentTai;						// USN_SIG_MM: 可以和RAI合一，使用联合?
	TAI								CurentTaiList[M_TALIST_LEN];	// USN_SIG_MM: TA list没有必要保存在MM CB表中?
	TAI								OldTaiList[M_TALIST_LEN];		// USN_SIG_MM: TA list没有必要保存在MM CB表中?
	VOS_UINT8						ucCurentTaiListNum;
	VOS_UINT8						ucOldTaiListNum;
	LAI								Lai;
	
	VOS_UINT8						bt1IsPtmsiSigIncluded:1;
	VOS_UINT8						bt1IsOldPtmsiSigIncluded:1;
	VOS_UINT8						bt1IsPtmsFlag:1;
	VOS_UINT8						bt1IsOldPtmsiFlag:1;
	VOS_UINT8						bt1IsCurentTaiListValid:1;
	VOS_UINT8						bt1IsOldTaiListValid:1;
	VOS_UINT8						bChangeInd:1;
	VOS_UINT8						bt1TmsiStatValid:1;
	
	VOS_UINT8						bt1ACCCResult:1;
	VOS_UINT8						bt1IsCombinedAttached:1;		/*UE是否已经联合附着成功, DTS2015010609666, added by pengyi, 2015-1-23*/
	VOS_UINT8						btExReserved:6;
  
	PLMN							EquivalentPlmn;					// FFS
	PLMN							SelectedPlmn;
	PLMN							TargetPlmn;						// USN1.2 协议跟进 by wushizhong

	VOS_UINT32						udwTraceFlag;					// 用户跟踪时使用?
	VOS_UINT8						ucOAMTraceFlag;					// O&M跟踪标志 */

	/* 消息缓存序列，主要使用在Idle的场景 */
	S_SPU_MM_CTRL_STORE_MSG_NODE*	pStoreFwdMsgList;
#ifndef MM_COMM_BUFF_TEST
	S_SPU_MM_CTRL_STORE_MSG_NODE*	pStoreMsgList;
#else
	VOS_LIST						pStoreMsgList;					// 消息缓存链表头指针
#endif

#ifndef MM_COMM_BUFF_TEST
	
	S_SPU_STORE_DATA_NODE*			pStoreDataList;
	
#else
	/* 数据缓存序列, 主要使用大的数据缓存和消息重发的场景*/
	VOS_LIST	pStoreDataList;										// 数据缓存链表头指针M
#endif
	VOS_UINT8						ucMmeS1ReleaseSendFlag;
	//VOS_UINT8						bt1IsRepBufferIniUEMsgFlag:1;
	//VOS_UINT8						btReserved:7;
	//S_MM_MSG_HEADER*				ptrBufferIniUEMsgData;			//缓存消息的头指针
	VOS_UINT8						ucDelSdbFlag;
	VOS_UINT8						ucSTMTypeWaitSppIdFromUip;		//E_STM_INSTANCE_NAME	
	//add by chenheng for USN2.0
	VOS_UINT8						ucOffLoadSgsnIndex;
	VOS_UINT8						ucAttachOrTauFailInSecPhaseFlag;
	VOS_UINT8						ucAttachFailbetweenAccAndCmpLiFlag;//合法监听中用户attach acc和attach cmp间失败后，不再上报detach使用
	VOS_UINT16						uwRFSPIndexInUse;				/* RFSP Index in Use */
	VOS_UINT32						udwCrtTimeStamp;
	
	S_ENBID							stCurrentEnbId;					/* 存储用户当前驻留的eNB的eNB ID */
	S_ENBID							stOldEnbId;						/* 存储用户原来驻留的eNB的eNB ID */
	TAI								aucOldTai;						/* 存储用户原来驻留的TA的TAI */
}VOS_PACKED S_MM_CB_COMM;
```
```c

#define		M_STMSI_LEN				11
#define		M_PTMSI_SIG_STR_LEN		9
#define		M_PTMSI_STR_LEN			11

#define		M_IMSI_LEN				8  /* IMSI长度 */
#define		M_IMEI_LEN				8
#define		M_TMSI_LEN				4
#define		M_PTMSI_SIG_LEN			3
#define		M_PLMN_LEN				3

typedef		VOS_UINT8			IMSI[M_IMSI_LEN];
typedef		VOS_UINT8			IMEI[M_IMEI_LEN];
typedef		VOS_UINT8			TMSI[M_TMSI_LEN];
typedef		VOS_UINT8			TMGI[6];
typedef		VOS_UINT32			PTMSI;
typedef		VOS_UINT8			PTMSISIGN[M_PTMSI_SIG_LEN];
typedef     VOS_UINT8           RAI[M_RAI_LEN];
typedef     VOS_UINT8           LAI[M_LAI_LENGTH];
typedef struct
{
	PLMN		PlmnCode;
	MMEGID		MmeGroupId;
	MMEC		MmeCode;
	MTMSI		Mtmsi;
}VOS_PACKED S_GUTI;

// 原因值组成
typedef struct
{
	VOS_UINT32		udwCauseType;		// 原因值类型, 参考枚举:E_USN_CAUSE_TYPE
	VOS_UINT32		udwCauseValue;		// 原因值取值
	VOS_UINT32		udwFileNo;			// 设置原因值的文件号
	VOS_UINT32		udwLineNo;			// 设置原因值的行号
} VOS_PACKED S_USN_CAUSE_INF;

typedef		VOS_UINT8           PLMN[M_PLMN_LEN];
typedef		VOS_UINT16			MMEGID; 
typedef		VOS_UINT8			MMEC;
typedef		VOS_UINT32			MTMSI;


typedef		VOS_UINT8			MSISDN[M_MSISDN_LEN];   /* MSISDN */

typedef		VOS_UINT8			TRUSERDN[M_USERDN_LEN];   /* USERDN */


typedef		VOS_UINT32			LOG_CID;   


typedef		VOS_UINT8			IMEISV[M_IMEI_LEN];	/*DEFINE BY CHENHENG ,TO BE CHECK*/

typedef		VOS_UINT8			TID[M_TID_LEN]; 
typedef		VOS_UINT32			IP;
typedef		VOS_UINT8			TMGI[6];
typedef		VOS_UINT32			TEID;
typedef		VOS_UINT16			FLOWLABEL; 
typedef		VOS_UINT8			LI_TARGETID[M_LI_TARGETID_LEN];
typedef		VOS_UINT8			ENODEBID[M_GLB_ENBID_LEN];
typedef		VOS_UINT8			TAI[M_TAI_LEN];

typedef		VOS_UINT8			RANID[M_RANID_LEN];
typedef		VOS_UINT8			GUTI[M_GUTI_LEN];

typedef		VOS_UINT16			MMEGID; 
typedef		VOS_UINT8			MMEC; 

typedef		VOS_UINT8			PTMSISIGN[M_PTMSI_SIG_LEN]
```
```
// 目前UDM和S1AP接口原因值中已经包含了协议原因值暂时不单独定义原因值
typedef enum
{
    USN_24301_CAUSE,           // 24301协议原因值
    USN_24008_CAUSE,           // 24008协议原因值
    USN_29274_CAUSE,          // 29274协议原因值
    USN_29060_CAUSE,          // 29060协议原因值
    USN_29272_CAUSE,           // 29272协议原因值
    USN_29002_CAUSE,         // 29002协议原因值
    USN_INNER_CAUSE,         // 内部原因值
    USN_CAUSE_BUTT
}E_USN_CAUSE_TYPE;
```
------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
