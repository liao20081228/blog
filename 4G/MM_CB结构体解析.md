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
	VOS_LIST						pStoreDataList;					// 数据缓存链表头指针M
#endif
	VOS_UINT8						ucMmeS1ReleaseSendFlag;
	//VOS_UINT8						bt1IsRepBufferIniUEMsgFlag:1;
	//VOS_UINT8						btReserved:7;
	//S_MM_MSG_HEADER*				ptrBufferIniUEMsgData;			//缓存消息的头指针
	VOS_UINT8						ucDelSdbFlag;
	VOS_UINT8						ucSTMTypeWaitSppIdFromUip;		//E_STM_INSTANCE_NAME
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
#define		M_IMSI_LEN				8	/* IMSI长度 */
#define		M_IMEI_LEN				8
#define		M_TMSI_LEN				4
#define		M_PTMSI_SIG_LEN			3
#define		M_RAI_LEN				6
#define		M_LAI_LENGTH			5
#define		M_PLMN_LEN				3
#define		M_ENB_LEN				5


typedef		VOS_UINT8			IMSI[M_IMSI_LEN];
typedef		VOS_UINT8			IMEI[M_IMEI_LEN];
typedef		VOS_UINT8			TMSI[M_TMSI_LEN];
typedef		VOS_UINT8			TMGI[6];
typedef		VOS_UINT32			PTMSI;
typedef		VOS_UINT8			PTMSISIGN[M_PTMSI_SIG_LEN];
typedef		VOS_UINT8			RAI[M_RAI_LEN];
typedef		VOS_UINT8			LAI[M_LAI_LENGTH];
typedef		VOS_UINT8			ENBC[M_ENB_LEN];
typedef		VOS_UINT8			PLMN[M_PLMN_LEN];
typedef		VOS_UINT16			MMEGID; 
typedef		VOS_UINT8			MMEC;
typedef		VOS_UINT32			MTMSI;


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


/* 消息缓存序列，主要使用在Idle的场景 */
typedef struct
{
	VOS_UINT8*		pStoreNext;			//缓存节点指针
	VOS_UINT32		udwMsgIndex;
	//VOS_UINT8*		pMsg;			//缓存消息指针	
	VOS_UINT32		udwSenderPid;		//发送这个缓存消息的模块的PID

	/* 原始缓存信息 */
	VOS_UINT32		udwMsgBufferLen;	/* 缓存消息长度 */
	VOS_UINT8*		pucMsgBuffer;		/* 缓存消息指针 */

	/* 控制信息 */
	VOS_UINT8		aucMsgProcEnabled;	/*TRUE为处理,FALSE为不处理，注意初始化值*/
	VOS_UINT8		aucReserved[3];

	/* 辅助信息 */
	VOS_UINT32		udwInnerMsgId;		/* 统一映射后的内部消息字 */
}S_SPU_MM_CTRL_STORE_MSG_NODE; 

//缓存控制节点
typedef struct
{
	VOS_UINT8*			pStoreNext;		//缓存节点指针
	VOS_UINT8			ucData[1];		//指向存储数据的第一个字节
}S_SPU_STORE_DATA_NODE; 

/*enodeB id 实际占用4个字节，第5个字节填充enodeB Type
ENBC[0] ~ ENBC[3]填充的是enodeB id， ENBC[4]填充的是enodeB Type*/
typedef struct
{
	PLMN				PlmnCode;
	ENBC				EnbCode;
}VOS_PACKED S_ENBID;

```

```c
// 目前UDM和S1AP接口原因值中已经包含了协议原因值暂时不单独定义原因值
typedef enum
{
	USN_24301_CAUSE,		// 24301协议原因值
	USN_24008_CAUSE,		// 24008协议原因值
	USN_29274_CAUSE,		// 29274协议原因值
	USN_29060_CAUSE,		// 29060协议原因值
	USN_29272_CAUSE,		// 29272协议原因值
	USN_29002_CAUSE,		// 29002协议原因值
	USN_INNER_CAUSE,		// 内部原因值
	USN_CAUSE_BUTT
}E_USN_CAUSE_TYPE;
```

# 4 S_MM_CB_MMCTRL
``` c
typedef struct
{
	/* BEGIN COMM */
	VOS_UINT8				ucMmCbFreeFlag;	/*VOS_FALSE: CB不需要释放; VOS_TRUE: CB需要释放*/

	S_MM_FSM_INFO			stFsmInfo;		/*主控状态机*/

	
	S_EMM_PARALLEL_STATE	astParallelState[MAX_PARALLE_STATE_NUM];	/* 并行状态监控标识 */
	VOS_UINT8				ucInterOldMMEAuthFailureFlag;				//与Old MME鉴权失败标识
	VOS_UINT32				udwCause;				//原因
	VOS_UINT32				udwRejCause;			//拒绝原因
	S_SPU_STORE_DATA		stSmDataBuffer;			/*用于缓存Sm data*/

	E_ACCCONN_CB_TYPE		enCurrentAccTblNo;		/*当前的ACC表号:取值ACCCONN_CB_PRECEDENT or ACCCONN_CB_SUCCEDENT*/
	
	E_USN_MMPROC_TYPE		enProcedureType;		/*当前流程类型*/
	VOS_UINT8				ucIsInterUsn;			/*当前流程是否为Inter流程, 取值: VOS_TRUE, VOS_FALSE*/
	VOS_UINT8				ucSelectedNetwork;
	VOS_UINT8				ucIsInitUE;				/*此次主控流程触发消息的原始消息类型， 0xFF:无效 1:是initUE*/
	VOS_UINT8				ucUsrRadioInfoChngFlag;	/*用的无线信息是否改变的标识，如ENBID TAI ECGI*/
	VOS_UINT8				ucGutiRellocFlag;		/*临时标识重分配标识*/
	VOS_UINT8				ucTaiListAllocFlag;		/*是否分配TAI LIST标记*/
	VOS_UINT8				ucIsrFlg;
	E_RAT_TYPE				enPeerRatType;			/*对端网元的系统类型*/

	S_NAS_KEY_SET_ID		stKsiasme;
	S_NAS_KEY_SET_ID		stKsisgsn;	
	S_UE_NET_CAP			stUeNetCap;					/*UE network capability*/
	TAI						stLastVisitedRegisteredTai; /*Last visited registered TAI*/	
	VOS_UINT8				aucDrxPara[M_DRXPARA_LEN];	/*DRX parameter*/ 
	/* UE-AMBR协议跟进 */
	S_GTPV2_UE_AMBR			stUeAmbrSubscribed;
	S_GTPV2_UE_AMBR			stUeAmbrInUse; 
	VOS_UINT8				ucMsNetWorkCapLen;
	S_MS_NET_CAP			stMsNetCap;					/* MS network capability */
	S_MS_CLASSMARK2			stMobileStationClassmark2;	/* Mobile station classmark 2 */
	S_MS_CLASSMARK3			stMobileStationClassmark3;	/* Mobile station classmark 3 */
	VOS_UINT8				aucSupportedCodecs[M_MAX_SUPPORTED_CODEC_LIST_LEN]; /* Supported Codecs */
	VOS_UINT32				udwSupportedCodecsLen;
	VOS_UINT8				ucUeRadioCapInfoUpdateNeeded; /* UE radio capability information update needed */
	VOS_UINT16				uwEpsBearerContextStat;		/* EPS bearer context status */
	VOS_UINT8				ucEpsBearerContextStatLen;
	VOS_UINT8				ucDoubleCbFlag;				/*启动双CB表标记*/
	VOS_UINT32				udwPeerMmCbNo; 

	UE_RADIO_CAPABILITY		ueRadioCap;
	VOS_UINT32				udwRadioCapLen;
	VOS_UINT8				ucPsLcsCap;
	VOS_UINT8				ucVoiceDomainPreferenceAndUesUsageSetting; /* Voice domain preference and UEs usage setting */
	
	S_UE_CONTEXT_MODIFICATION_TYPE_FLAG stUeContextModificationTypeFlag;
	
	VOS_UINT8				bt1IsUeNetCapIncluded:1;
	VOS_UINT8				bt1IsLastVisitedRegisteredTaiIncluded:1;
	VOS_UINT8				bt1IsDrxParaIncluded:1;
	VOS_UINT8				bt1IsMsNetCapIncluded:1;	
	VOS_UINT8				bt1IsMobileStationClassmark2Included:1;
	VOS_UINT8				bt1IsMobileStationClassmark3Included:1;	
	VOS_UINT8				bt1IsSupportedCodecsIncluded:1;
	
	VOS_UINT8				bt1IsUeRadioCapInfoUpdateNeededIncluded:1;
	VOS_UINT8				bt1IsEpsBearerContextStatIncluded:1;	
	VOS_UINT8				bt1IsAdditionalGutiIncluded:1;	
	VOS_UINT8				bt1IsNonCurrentNativeKSIIncluded;
	VOS_UINT8				bt1IsRadioCapValid:1;
	VOS_UINT8				bt1CancelLocationRecFlag:1;		/*用来标识是否收到过cancel location消息*/

	VOS_UINT8				bt1RptLcsAftPaging:1;
	
	VOS_UINT8				bt1IsVoiceDomainPreferenceAndUesUsageSettingIncluded:1;

	VOS_UINT8				bt1Reserved:2;
	/* END COMM*/
	
	HTIMER					T3450ClockId;
	
	VOS_UINT8				ucAdditionalUpdateType; /* E_ADDITIONAL_UPDATE_RESULT_VALUE */
	
	/* BEGIN ATTACH*/
	VOS_UINT8				ucEpsAttachType;	/*Attach流程的类型*/
	VOS_UINT8				ucReAttachFlag;	/*标识attach冲突的标志*/
	/* END ATTACH*/

	/* BEGIN TAU*/
	HTIMER					T3ClockId;
	VOS_UINT8				ucEpsTauResult; /*EPS TAU result*/
	VOS_UINT8				ucActiveFlag;	
	VOS_UINT8				ucRauType;		/*RAU类型*/
	VOS_UINT8				ucSgwChangeIndication;
	VOS_UINT8				ucTauConflictReTauflag;
	/* END TAU*/
	
	/* BEGIN DETACH*/
	VOS_UINT8				ucIsImplictDetach ;		/*是否为隐式分离*/
	VOS_UINT8				ucDetachParallelStateFlag ; /*网络侧触发的detach并行状态机的标志位*/
	VOS_UINT8				ucSendDetachReqTimes ;		/*USN_SIG_MM: detach req消息的重发次数, 可以与ucMmCtrlTimeoutCounter合一*/
	VOS_UINT8				ucDetachSponsor;			/*detach发起方*/
	VOS_UINT8				ucPagingFlag;				/*是否正在寻呼的标志位(初始化为not start)*/	
	VOS_UINT8				ucDetachType;				/*Detach 类型*/	
	VOS_UINT8				ucIsPowerOff;				/*detach req消息中的DETACH TYPE信元的POWER OFF字段,代表是否是关机分离*/
	VOS_UINT32				udwCancelLocType;			/*CANCEL LOCATION消息中的CANCEL LOCATION TYPE*/	
	VOS_UINT32				udwCancelLocResult;		/*CANCEL LOCATION RSP消息中的原因值*/
	VOS_UINT32				udwSdbDetachType;			/*SDB分离的分离类型*/
	VOS_UINT32				udwSdbDetachCause;			/*SDB分离的分离原因*/
	VOS_UINT32				udwPagingCause;			/*寻呼原因*/
	
	VOS_UINT8				ucSmDetachType;	/*NW Detach的一种，参考枚举E_EMM_NETWORK_MS_DETACH_TYPE*/
	VOS_UINT8				ucPttDetachType; /*PTT Detach的一种，参考枚举PTTF_TUMMME_DETACH_TYPE_ENUM*/
	/* END DETACH*/

	/*service req begin*/
	/*这两个字段是用来存放SERVICE REQ消息里面的信元的值的*/
	VOS_UINT8				ucKsiAndSeqNumber;		/* KSI and sequence number */
	VOS_UINT16				uwMessageAuthCodeShort; /* Message authentication code short */
	VOS_UINT16				uwServiceReqTriggerCause; /*service req流程的发起原因*/
	/*service req end*/
	
	/*Extended service req*/
	VOS_UINT8				ucSeviceType;
	VOS_UINT8				ucCsfbResponse;
	VOS_UINT32				udwGtpVersion;		/*add for记录当前的GTPC的版本类型*/
	
	/* BEGINE HANDOVERE */
	S_TARGET_ID				stTargetId;		/*目标ENB ID (TAI + ENBID)*/
	S_RANAP_TARGET_RNC_ID	stTgtRncId;
	VOS_UINT32				udwHandoverType;			/*切换消息的类型*/
	VOS_UINT8				ucDirectFwdPathAvailPresent; /*是否启动标识*/
	VOS_UINT8				ucDirectForwardingSupported; /*源侧ENB是否支持直接转发*/
	VOS_UINT8				ucSmCancelFlag;		/*标识Sm侧的Cancel*/
	VOS_UINT8				ucPeerCancelFlag;	/*标识Mm的对侧的Cancel*/
	VOS_UINT8				ucConnCancelFlag;	/*标识接入连接Cancel状态*/
	VOS_UINT8				ucNeedSendCancelRsp; /*标识是否需要给对端发送Cancel Rsp*/
	VOS_UINT32				udwHoCasue;					/*记录handover的Cause值*/
	S_SPU_STORE_DATA		stSmContainerData;	/*用于缓存HO Container data*/
	S_SPU_STORE_DATA		stSmContainer1Data;	/*用于缓存HO Container1 data*/
	VOS_UINT8				ucNormalCancelFlag;			/*用于新侧记录收到Cancel的标识*/
	VOS_UINT8				ucDetachFlag;				/*用于新侧记录收到Detach的标识*/
	VOS_UINT8				ucRcvFrwReloCmpackFlag;		/*Added for HO_TAU interact*/
	VOS_UINT8				ucSgwChangeind;
	VOS_UINT8				ucHoCmdFlag;					/*用于判断老侧，发送过HandOverCmd消息*/
	VOS_UINT8				ucnewabnormaltype;	
	/* END HANDOVER */

	VOS_UINT32				udwPeerInterface;			/*用于记录对端接口类型*/
	
	VOS_UINT32				udwRevokeReqSenderCpuId;
	VOS_UINT32				udwRevokeReqSenderCbId;
	VOS_UINT8				ucMmCtrlNeedQueryUIPFlag;
	VOS_UINT8				ucImsiDetachIndFlag;
	VOS_UINT8				ucEpsDetachIndFlag;

	/* E_MM_UE_MODE_IN_CSFB */
	VOS_UINT8				ucUeEMMMode;	/*用于记录Extend ServReq中的UE状态*/
	
	VOS_UINT8				ucAttachResult;
	VOS_UINT8				ucRauResult;
	VOS_UINT8				ucKsiCksnInMsg;
	VOS_UINT8				ucR99CapOfUmtsAka;
	VOS_UINT8				bt1WaitPduNumList:1;
	VOS_UINT8				bt1GetImsiInProc :1; /*在RAU和ATTACH中是否携带了IMSI，在二次鉴权是需要判断*/
	VOS_UINT8				bt1RelocRaiChanged:1;
	VOS_UINT8				bt1IuClearIndFlag:1;	
	VOS_UINT8				btReceiveFwdSrnsCtxFlag:1;
	VOS_UINT8				btRauAfterReloc:1;
	VOS_UINT8				btMmCtrlReserve:2;	
	VOS_UINT8				ucRelocType;
	VOS_UINT8				ucReleaseCause;
	VOS_UINT32				udwSrcConnId;
	VOS_UINT32				udwTgtConnId;
	VOS_UINT8				ucPdpStatusLen;
	VOS_UINT8				aucPdpStatus[M_MAX_PDP_CONTEXT_STAT_LEN];
	VOS_UINT8				ucUplinkDataStatusValidFlag;
	VOS_UINT8				aucUplinkDataStatus[M_MAX_UPLINK_DATA_STAT_LEN];
	VOS_UINT8				ucMbMsStatusLen;
	VOS_UINT8				aucMbmsStatus[M_MAX_MBMS_CONTEXT_STAT_LEN];
	VOS_UINT8				ucCiphFlag;			/*是否加密*/
	VOS_UINT8				ucMcdrCloseCause;
	VOS_UINT8				ucSndComIdFlag;		/*Iu 连接下，是否向RNC发送了COMM ID消息*/
	VOS_UINT8				ucNeedUpdateSdb;		/*释放CB时是否需要更新SDB*/
	VOS_UINT8*				pSmNasMsg;

	/*用于指导MM填充MSG_MMEPTT_SERVICE_REQ: 是给TUM发送Trunk Service Req还是发送普通Service Req*/
	VOS_UINT8				ucTrunkServReqFlag;	/*VOS_TRUE: Trunk SR, VOS_FALSE: 普通SR*/

	/* 用于指导TAU结束后是否向PTT发送SR: Paging/SR流程中收到TAU的交叉，若是PTT Paging触发的Paging/SR，进入TAU流程，TAU结束后，不释放S1连接，发送MMEPTT_SERVICR_REQ给PTT */
	VOS_UINT8				bt1MmePttServReqAfterTauFlag:1;	/* VOS_TRUE : TAU流程结束后需发送MMEPTT_SERVICR_REQ给PTT */ 
																/* VOS_FALSE: 无需发送消息 */
	VOS_UINT8				bt7MmCtrlReserved:7;
	
	/* 二次鉴权获取不同用户Identity(IMSI)后, 会判断是否接入受限: 如果受限会返回鉴权失败并携带特殊内部原因值, 主控取CB中的以下原因作为流程拒绝原因. */
	VOS_UINT32				udwS1RestrictCause;	/* 保存接入限制原因 E_EMM_REJ_CAUSE */ 
}VOS_PACKED S_MM_CB_MMCTRL;

typedef struct 
{
	S_MM_TRACE_FLAG			stTraceFlag; /* Modified by liubenchang(l00163414) for USN1.2 E2ETrc on 2010-04-12 */
	
	VOS_UINT16				uwE2eBufMsgHeadIdx;	/* 缓存的全网跟踪消息队列头下标 */
	
	VOS_UINT16				uwE2eBufMsgTailIdx;	/* 缓存的全网跟踪消息队列尾下标 */
	VOS_UINT32				audwMsgLocaInBuff[M_MAX_TRC_BUFF_MSG_NUM];
	
	
}VOS_PACKED S_MM_OM_TRC;
```

# S_MM_SECURITY_INFO
``` c
typedef struct
{ 
	//控制信息
	S_MM_FSM_INFO				stFsmInfo[ACCCONN_CB_BUTT]; //Null is need set respectively
	S_SECURITY_CONN_INFO		astSecConnInfo[ACCCONN_CB_BUTT];

	E_MM_SEC_SPONDER			aSecSponder[M_MAX_SEC_PROCEDURE];//安全流程的发起者，暂时定义为同时最大为4个
	// temp info for native: Phase(AKA).
	// Notice: Phase(SMC) use ctx in SDB
	S_MM_PRE_NATIVE_SEC_INFO	stPreNativeSecInfo;	//Null is 0xFF

	// temp info for mapped
	S_MM_PRE_MAPPED_SEC_INFO	stPreMappedSecInfo;	//Null is 0xFF

	VOS_UINT8					ucIsPreMappedSecInfoIncluded;	//是否有MAPPED安全上下文, {VOS_FALSE, VOS_TRUE}
	//USN2.0 ADD
	U_SDB_USN_AUTH_CONTEXT		uNeedCfnAuthVector;		//AKA阶段GU安全上下文中的鉴权集应该存储在这个地方
	
	U_SDB_USN_AUTH_CONTEXT		uUsedAuthVector;		//上下文中的鉴权集，GU使用

	VOS_UINT32					udwResult;							/* 记录返回给主控的udwSecResult */
	VOS_UINT8		ucKsiCksn;
	VOS_UINT8		aUsedKc[8];// 2G使用
	VOS_UINT8		aUsedCk[16];// 3G使用
	VOS_UINT8		aUsedIk[16];// 3G使用
	VOS_UINT8		ucSendAuthCiphReqMsgToAuthFlag;//发送AuthCIphReqMsg是否为了鉴权
	VOS_UINT8		ucCipherAlgorithm;// 2G 3G使用
	VOS_UINT8		ucIntegralityAlgorithm;// 3G使用
	VOS_UINT8		ucIsNeedWaitAIA;			//Null is 0xFF
	VOS_UINT8		ucIsAuthSucAtInterNew;		//Null is 0, 作为Inter New侧，是否成功执行AKA, {VOS_FALSE, VOS_TRUE}	

	// SMC info
	VOS_UINT8		ucSmcType;					//E_EPS_SMC_TYPE	

	VOS_UINT8		ucSecCtxValidFlagInLocalNE; //E_EPS_CTX_VALID_IN_LOCAL_NE
	
	S_MM_SEC_INFO_IN_MSG	stSecInfoInMsg;		// SEC info in Msg	
	
	VOS_UINT8		ucIsDoneAka;		//是否执行AKA, {VOS_FALSE, VOS_TRUE}
	VOS_UINT8		ucIsDoneSmc;		//是否执行SMC, {VOS_FALSE, VOS_TRUE}
	VOS_UINT8		ucIsDoneChkImei;	//是否执行Check IMEI, {VOS_FALSE, VOS_TRUE}
	VOS_UINT8		ucIsDoneGetImei;	//是否执行Get IMEI, {VOS_FALSE, VOS_TRUE}	

	VOS_UINT8		ucIsGutiTrust;		//GUTI附着，GUTI是否可信
	//TempTempTemp
}VOS_PACKED S_MM_SECURITY_INFO;
```

``` c
typedef struct 
{
	VOS_UINT8		ucMainState;		// 第一层状态*
	VOS_UINT8		ucSubState;			// 第二层子状态?
	VOS_UINT8		ucSubSubState;		// 第三层子子状态*
	VOS_UINT8		ucTimeoutCounter;	// 定时器超时次数计数*
	HTIMER			TimerId;			// 状态机的定时器名称*, void *
}VOS_PACKED S_MM_FSM_INFO;
```
```c
typedef enum
{
	ACCCONN_CB_PRECEDENT = 0, /* 第一个使用的ACC_CB */
	ACCCONN_CB_SUCCEDENT,	/* 第二个使用的ACC_CB */
	ACCCONN_CB_THIRD,
	ACCCONN_CB_BUTT,
	ACCCONN_CB_NOFOUND = ACCCONN_CB_BUTT
} E_ACCCONN_CB_TYPE;
```
```c
typedef struct
{
	E_AUTH_SET_TYPE		eNativeAuthType;
	E_AUTH_SET_TYPE		eUsedAuthType;
	VOS_UINT8			ucKeyStatus;// 3G使用
	VOS_UINT8			ucAuthCiphReference;
	VOS_UINT8			ucIdentityType;			//E_EMM_ID_TYPE 
	VOS_UINT8			ucAuthFailCount;
	VOS_UINT8			ucAkaMacFailCount;		//Null is 0
	VOS_UINT8			ucAkaSyncFailCount;		//Null is 0
	E_USN_MMPROC_TYPE	enProcedureType;
}VOS_PACKED S_SECURITY_CONN_INFO;

typedef enum 
{
	SDB_AUTHSET_TRIPLET_TYPE,	//三元组
	SDB_AUTHSET_QUINTET_TYPE,	//五元组
	SDB_AUTHSET_QUATERNION_TYPE,	//四元组
	SDB_AUTHSET_BUTT
}VOS_PACKED E_AUTH_SET_TYPE;

typedef enum 
{
	MM_SEC_KEY_STATUS_NEW,
	MM_SEC_KEY_STATUS_OLD,

	MM_SEC_KEY_STATUS_BUTT = 0xff
}E_SEC_KEY_STATUS;

typedef enum
{
	EMM_ID_NONE					= 0x00,
	EMM_ID_IMSI					= 0x01,
	EMM_ID_IMEI					= 0x02,
	EMM_ID_IMEISV				= 0x03,
	EMM_ID_TMSI					= 0x04,
	EMM_ID_TMGI					= 0x05,
	EMM_ID_GUTI					= 0x06,
	EMM_ID_BUTT
}E_EMM_ID_TYPE;



typedef enum
{
	MMPROC_ATTACH = 0,
	MMPROC_UE_DETACH,
	MMPROC_MME_DETACH,
	MMPROC_HSS_DETACH,
	MMPROC_SERVICE_REQ,
	MMPROC_S1RELEASE,
	MMPROC_EXTEND_SERVICE_REQ,
	MMPROC_INTRA_TAU = 0x84,
	MMPROC_INTER_TAU,
	MMPROC_INTRA_HANDOVER,
	MMPROC_INTRA_HANDOVER_OLD,
	MMPROC_PAGING,
	MMPROC_S1SEC,
	MMPROC_UE_CTX_MODIFICATION,
	MMPROC_PTT_HANDOVER,
	MMPROC_PTT_BLOCK_UE,
	MMPROC_TRUNK_SERVICE_REQ,
	MMPROC_BPTT_BLOCK_UE,
	MMPROC_BUTT
}E_USN_MMPROC_TYPE;

```

``` c
typedef enum
{
	E_SEC_SPONDER_MM_CTRL,
	E_SEC_SPONDER_MM_ACC,//目前只有在3G/2G 寻呼时发起
	E_SEC_SPONDER_SM,
	E_SEC_SPONDER_SMS,
	E_SEC_SPONDER_BUTT
}E_MM_SEC_SPONDER;
#define M_MAX_SEC_PROCEDURE 4
```

``` c
typedef struct
{
	S_NAS_KEY_SET_ID	stKsiasme;
	S_SDB_EPS_VECTOR	stEpsAV;	
}VOS_PACKED S_MM_PRE_NATIVE_SEC_INFO;

/*24301: 9.9.3.21	NAS key set identifier，网络分配，需要编码*/
typedef struct
{
	VOS_UINT8 ucTSC;/*Type of security context flag:0 cached security context,1 mapped security context*/
	VOS_UINT8 ucNasKeyID;/*NAS key set identifier */
}VOS_PACKED S_NAS_KEY_SET_ID;

//Mm Auth 数据定义
typedef struct
{
	VOS_UINT8	aucRand[16];
	VOS_UINT8	ucXresLen;
	VOS_UINT8	aucXres[16];
	VOS_UINT8	ucAUTNLen;			//AUTN的长度应该是16， =	SQN * AK || AMF || MAC，不应该是可变长度的。
	VOS_UINT8	aucAutn[16];
	VOS_UINT8	aucKasme[M_MAX_KASME_LEN];
}VOS_PACKED S_SDB_EPS_VECTOR	;	//define the same as S_SDB_QUINTUPLET
```

``` c
typedef struct
{
	S_SEC_INFO_MAPPED_KASME	stMapKasme; 
	S_NAS_KEY_SET_ID		stKsisgsn;
	VOS_UINT8				aucNH[M_EMM_KENB_LEN];		//Null is 0
	VOS_UINT8				ucNasItgAlgthm;	//E_MM_SECURITY_EIA,	Null is 0xFF 
	VOS_UINT8				ucNasEncAlgthm;	//E_MM_SECURITY_EEA,	Null is 0xFF 
	VOS_UINT8				aucKnasenc[MM_SEC_NAS_ALG_KEY_LEN];	// NULL is 0	
	VOS_UINT8				aucKnasint[MM_SEC_NAS_ALG_KEY_LEN];	// NULL is 0	
	VOS_UINT32				udwDlNasCount;	//因SMCommand消息重传，为MAPPED安全上下文临时保存的DL NAS COUNT,NULL is 0
	VOS_UINT32				udwUlNasCount;	//因SMCommand消息重传，为MAPPED安全上下文临时保存的UL NAS COUNT,NULL is 0
}VOS_PACKED S_MM_PRE_MAPPED_SEC_INFO;

typedef struct
{
	//Condition
	VOS_UINT8	aucCK[MM_SEC_CK_LEN];		// Null is 0;
	VOS_UINT8	aucIK[MM_SEC_IK_LEN];		// Null is 0;
	VOS_UINT32	udwNonceue;				// Null is 0xFF;
	VOS_UINT32	udwNoncemme;				// Null is 0xFF;

	//Output
	VOS_UINT8	aucMappedKasme[M_KASME_LEN]; // Null is 0;
}VOS_PACKED S_SEC_INFO_MAPPED_KASME;

#define M_KASME_LEN	32
#define MM_SEC_CK_LEN				(16)
#define MM_SEC_IK_LEN				(16)

#define MM_SEC_NAS_ALG_KEY_LEN		(16)
#define M_EMM_KENB_LEN				(32)
```
```c
typedef union
{
	S_SDB_EPS_VECTOR		stEAuthContext;
	S_SDB_UMTS_VECTOR		stUAuthContext;
}VOS_PACKED U_SDB_USN_AUTH_CONTEXT; 

/* 五元组鉴权集*/
typedef struct
{
	VOS_UINT8	aucRand[16];
	VOS_UINT8	ucXresLen;
	VOS_UINT8	aucXres[16];
	VOS_UINT8	aucCk[16];
	VOS_UINT8	aucIk[16];
	VOS_UINT8	ucAutnLen;
	VOS_UINT8	aucAutn[16];
}VOS_PACKED S_SDB_UMTS_VECTOR;	//define the same as S_SDB_QUINTUPLET
```

``` c
typedef struct
{
	VOS_UINT32			udwSecHdrType;	//MM_SEC_SECURITY_HEADER_TYPE
	S_NAS_KEY_SET_ID	stKsi;	
}VOS_PACKED S_MM_SEC_INFO_IN_MSG;

/* 安全头枚举 */
typedef enum 
{
	EPS_SEC_PLAIN_NAS_MSG		= 0,
	EPS_SEC_INTE_ONLY			= 1,
	EPS_SEC_INTE_AND_CIPH		= 2,
	EPS_SEC_INTE_WITH_NEW		= 3,
	EPS_SEC_INTE_AND_CIPH_WITH_NEW = 4,
	EPS_SEC_SERVICE_REQUEST_MSG	= 12,
	EPS_SEC_TRUNK_SERVICE_REQ	= 13,
	EPS_SEC_RESERVER_MSG_ONE	= 14,
	EPS_SEC_RESERVER_MSG_TWO	= 15,
	EPS_SEC_RESERVER_MSG_THREE	= 16,

	EPS_SEC_OTHER_MSG_BUTT
} EPS_NAS_SEC_SECURITY_HEADER_TYPE;

```

------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
