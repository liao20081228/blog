---
title: MME状态机编程框架 
tags: 4G
---

------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 框架定义
## 1.1 状态机原型描述
**状态机（FSM）的定义**：

FSM：有限状态机。通常，状态机是一种记录给定时刻状态的设备，并根据输入，对每个给定的状态，改变其现有状态或者引发一个动作。
 
 整个SPM分成多个模块，各模块有的又分成各个子模块。各子模块在处理事务时，都会起相应的状态机。由于对应流程复杂度，处理事务的平行性等要求不一，状态机的复杂性和设计也不一样。


**FSM的总体思路**：

状态机机制包括状态的存储，状态的读取与设置，状态的分级，状态的定位与各状态收到事件后的处理，暂态的保护等机制。

* 状态的存储：各模块的稳定状态存储在USD里，便于备份与核查。暂态一般存储在CB或临时上下文中。
* 状态的读取与设置： 
	* MM，SM，UDM原SPU进程的模块，状态和对象较复杂，对状态的读取与设置，有专门的函数，并和状态机容器存储在一起。
	* S1AP对状态的读取与设置，使用专门的函数，但没有使用状态机容器的概念。
	* 而USD和SIPU的状态处理比较简单，采用的是直接读取更改的方式。
* 状态的分级，这方面主要考虑的是对象状态的复杂度与各流程之间的并行性。
	* MM，SM，S1AP，UDM处理的对象较复杂，采用两级状态机的方式，主状态为对象的稳态或较复杂的流程，子状态为更深更细的暂时状态。有些情况下，MM在某些复杂流程（如多MME时的inter TAU）过程中，使用了三级状态。
	* SIPU和USD的机制较简单，可以不采用分层状态的机制。
* 状态的定位与各状态收到事件后的处理。
	* 各模块的子模块都有自己的状态机。模块消息处理函数，在收到外部/内部事件后，根据触发事件的源以及消息类型，来决定将事务交给哪个子模块的状态机进行处理。
	* 各实际处理模块，在处理完事务后，会决定是否维持状态还是发生状态变更，并决定下一步的动作。
* 暂态的保护： 各状态机的暂态，都应起相应的定时器进行保护。定时器超时后，按照相应流程失败处理。定时器的长度，需要根据具体的业务流程来决定。

模块内各子模块的状态机处理机制应该统一，而各模块之间可以不同，由所处理的对象来决定。如MM使用状态机容器的方式，将状态的读取设置，定时器句柄等信息组合起来，便于维护。而USD使用的状态机机制就可以十分简单。

此外，MM和SM在每个子状态都会起定时器，而不是一个流程的大定时器保护。

## 1.2 事件处理机制描述
事件包括外部消息，内部消息，定时器。事件的处理，包括事件定位，预处理，消息处理（定时器事件也是消息）。
 * 每个事件都需要映射到具体的对象上面。即先进行定位。
	* 为了和外部接口解耦，外部消息可以将消息类型转换成模块内部的消息类型。
	* 为了更快速的将消息转发给相应子模块，内部消息类型需要考虑将子模块标识和内部消息类型进行快速绑定（如采用子模块名为前缀的做法）。
	* 同样，采用内部消息后，消息可以携带模块内的此对象内部标识（如CB number）
* 事件预处理目的是找到此事件的实际处理函数。这部分处理不一定需要专门的函数。
	* 需要进行一定的初步参数检查。
	* 根据发送方标识，消息类型，处理对象的当前状态，进行处理函数的定位。有可能因为流程交叉，而导致新事件的缓存，丢弃或对老事件的抢占。
	* 可能有些事件，对应处理简单，**直接在预处理函数里进行**。
* 消息处理为事件的实际处理，处理包括内部数据的变更，外部流程的触发，定期器的启动，状态的变迁等。
* 暂态应该有保护，消息处理失败，则流程中暂时分配的资源应该释放掉（包括可能触发外部模块分配的资源）。

内部消息队列机制：
* 对MM，SM，UDM而言，各模块内子模块间的通信为内部消息队列。
* 内部消息队列为模块申请的一个队列，在初始化时进行资源的申请和配置。各模块采用相同的队列机制来管理。各模块的内部消息队列独立，其余模块不能操作。
* 各模块初步处理完外部事件后，从内部消息队列中读取此流程引发的内部消息。直到处理完所有的内部消息

# 2 实现

## 2.1 状态机定义

``` c
/* 状态机容器结构体定义，状态机放在容器中，封装了设置和访问状态机的句柄 */
typedef struct
{
	S_SPU_STATE							*pstStateMachine;		/* 状态机头指针 */
	VOS_UINT32							udwPid;					/* 该实例所属的PID , 枚举*/
	PF_SPU_GET_STATE_CALLBACK			pfnGetState;			/* GET 状态回调 */
	PF_SPU_SET_STATE_CALLBACK			pfnSetState;			/* SET 状态回调 */
	PF_STM_GET_TIMER_PTR_CALLBACK		pfnGetTimer;			/* 定时器回调 */
	PF_SPU_CHANGE_TIMER_LEN_CALLBACK	pfnChangeTimerHandle;	/*此回调函数用来进行定时器长度转换，
																解决2/3G主控代码合一但是定时器配置分开的问题*/
} VOS_PACKED S_SPU_STATE_CONTAINER;
```

```c
#define M_STM_MAX_STATE_LEVEL	(3) /* 最大状态层数 */
#define M_STM_MAX_MSG_PROC		(50) /* 每个状态处理的最大消息数 */
/* 状态 */
typedef struct
{
	VOS_UINT8		aucStates[M_STM_MAX_STATE_LEVEL];	/* 当前状态，根据下标依次为第一层，第二层，第三层 */
	VOS_UINT32		udwTimerLen;						/* 定时器长度 */
	VOS_UINT3		udwTimerName;						/* 定时器名,枚举*/
	VOS_UINT32		udwMsgProcLen;						/* 消息处理函数表长度 */
	S_STM_MSG_PROC	astMsgProc[M_STM_MAX_MSG_PROC];		/* 消息处理函数表 */
}VOS_PACKED S_SPU_STATE;

/* 消息处理结构 */
typedef struct
{
	VOS_UINT32			udwMsgType;		/* 消息类型 */
	VOS_UINT32			udwMid;			/* 子模块ID */
	PF_SPU_STATE_FUNC	pfnStateFunc;	/* 状态处理函数 */
	S_STM_RET_STATE		*pstRetState;	/* 返回值列表 */
}VOS_PACKED S_STM_MSG_PROC;	


/* 返回值列表。如果下一个主状态，下一个子状态，下一个子子状态均为NULL，则状态不发生改变*/
typedef struct
{
	VOS_UINT32	udwRetCode;		/* 返回码 */
	VOS_UINT8	ucStartTimer;	/* 是否启动定时器 */
	VOS_UINT8	ucMainState;	/* 下一个主状态 */
	VOS_UINT8	ucSubState;		/* 下一个子状态 */
	VOS_UINT8	ucSubSubState;	/* 下一个子子状态 */
}VOS_PACKED S_STM_RET_STATE;


```

``` c
/* 消息处理函数 根据返回值在返回值列表中找到对应项，迁移到下一个状态 */
typedef VOS_UINT32 (*PF_SPU_STATE_FUNC)(VOS_UINT32 udwCbNo, VOS_VOID *pMsg);


/* GET状态回调，获取状态机的主状态、子状态、子子状态*/
typedef VOS_UINT32 (*PF_SPU_GET_STATE_CALLBACK)(VOS_UINT32  udwCbNo,
												VOS_UINT8  *pucMainState,
												VOS_UINT8  *pucSubState,
												VOS_UINT8  *pucSubSubState);

/* SET状态回调，设置获取状态机的主状态、子状态、子子状态*/
typedef VOS_UINT32 (*PF_SPU_SET_STATE_CALLBACK)(VOS_UINT32 udwCbNo,
												VOS_UINT8  ucNextMainState,
												VOS_UINT8  ucNextSubState,
												VOS_UINT8  ucNextSubSubState);
											

/* 取得定时器句柄指针，失败返回VOS_NULL_PTR*/
typedef HTIMER * (*PF_STM_GET_TIMER_PTR_CALLBACK)(VOS_UINT32 udwCbNo);

/* 状态机通知业务定时器长度修改函数 */
typedef VOS_VOID (*PF_SPU_CHANGE_TIMER_LEN_CALLBACK)(VOS_UINT32 udwCbNo,
														VOS_UINT32 udwStmName,
														VOS_UINT32 *pudwTimerName,
														VOS_UINT32 *pudwTimerLen);

typedef struct RelTimerType *  HTIMER;
```

## 2.2 状态机操作

```c
/* 开始声明状态机 */
#define BEGIN_STATE_MACHINE(PID,                \
                            STATE_MACHINE_NAME, \
                            GET_STATE_CALLBACK, \
                            SET_STATE_CALLBACK, \
                            TIMER_HANDLER)      \
    if (E_STM_IC_SUCCESS != StmBeginDeclare((PID), \
                                            (STATE_MACHINE_NAME), \
                                            (GET_STATE_CALLBACK), \
                                            (SET_STATE_CALLBACK), \
                                            (TIMER_HANDLER))) \
    {                                           \
        return VOS_FALSE;                       \
    }                                           \
    


/* 添加注册项 */
/* 模块ID， 主状态， 子状态， 子子状态， 消息类型， 处理函数， 返回值列表 */
#define AddStateHandler(SUB_MODULE_ID, MAIN_STATE, SUB_STATE, SUB_SUB_STATE, MESSAGE, MSG_HANDLER, P_RET_STATE) \
    if (E_STM_IC_SUCCESS != StmAddStateHandler((SUB_MODULE_ID), \
                                               (MAIN_STATE), \
                                               (SUB_STATE), \
                                               (SUB_SUB_STATE), \
                                               (MESSAGE), \
                                               (MSG_HANDLER), \
                                               (P_RET_STATE))) \
    {                                           \
        return VOS_FALSE;                       \
    }                                           \
    


/* 结束声明状态机 */
#define END_STATE_MACHINE()                     \
    StmEndDeclare();                            \
    return VOS_TRUE;                    \
```

``` c
VOS_UINT32 StmBeginDeclare(VOS_UINT32 udwPid,
                           VOS_UINT32 udwStmName,
                           PF_SPU_GET_STATE_CALLBACK pfnGetState,
                           PF_SPU_SET_STATE_CALLBACK pfnSetState,
                           PF_STM_GET_TIMER_PTR_CALLBACK pfnGetTimer)
{
    S_SPU_STATE *pstStateMachine = VOS_NULL_PTR;

    if ((M_STM_MAX_INSTANCE_NUM <= udwStmName)
        || (VOS_NULL_PTR == pfnGetState)
        || (VOS_NULL_PTR == pfnSetState))
        /* || VOS_NULL_PTR == pfnGetTimer ) */
    {
        SPP_COM_TRACEEX(SPP_COM_ERR_0012, SMP_ERROR_LOG, SPP_COM, 
            "Invalid Para, PID: 0x%x, STM Name: 0x%x, pfnGetState = %p, pfnSetState = %p.",
            udwPid, udwStmName, (VOS_UINTPTR)pfnGetState, (VOS_UINTPTR)pfnSetState);

        return E_STM_IC_INVALID_PARAM;
    }
    
    StmInit();  /* 初始化状态机容器 ，将状态机容器置为0，并设置全局初始化标志为true*/

    g_udwCurrentStmName = udwStmName; //全局指明要声明的状态机
    //为状态机分配空间
    pstStateMachine = g_astStmContainer[udwStmName].pstStateMachine;
    if (VOS_NULL_PTR == pstStateMachine)
    {
        pstStateMachine = VOS_MemAlloc(udwPid, STATIC_MEM_PT, sizeof(S_SPU_STATE) *  (M_STM_MAX_STATE_INDEX));
        if (VOS_NULL_PTR == pstStateMachine)
        {
            SPP_COM_TRACE(SPP_COM_ERR_0013, SMP_ERROR_LOG, SPP_COM, 
                "Alloc Mem Fail, PID: 0x%x, STM Name: 0x%x.", udwPid, udwStmName);

            return E_STM_IC_SYSTEM_FAILURE;
        }
    }
    else 
    {
        SPP_COM_TRACE(SPP_COM_ERR_0014, SMP_ERROR_LOG, SPP_COM, 
            "STM Reinit, PID: 0x%x, STM Name: 0x%x.", udwPid, udwStmName);
    }

    memset_s((VOS_UINT8 *)pstStateMachine, sizeof(S_SPU_STATE) * (M_STM_MAX_STATE_INDEX),0, sizeof(S_SPU_STATE) * (M_STM_MAX_STATE_INDEX));
    //设置状态机容器地址以及访问状态机的句柄
    g_astStmContainer[udwStmName].udwPid = udwPid;
    g_astStmContainer[udwStmName].pstStateMachine = pstStateMachine;
    g_astStmContainer[udwStmName].pfnGetState = pfnGetState;
    g_astStmContainer[udwStmName].pfnSetState = pfnSetState;
    g_astStmContainer[udwStmName].pfnGetTimer = pfnGetTimer;

    g_astStmContainer[udwStmName].pfnChangeTimerHandle = VOS_NULL_PTR;
    return E_STM_IC_SUCCESS;
}


VOS_VOID StmInit(VOS_VOID)
{
    if (VOS_FALSE == g_bIsStmContainerInitialized)
    {
        /* 状态机容器重未被初始化过 */
        VOS_MemSet((VOS_UINT8 *) g_astStmContainer, 0, sizeof(g_astStmContainer));
        g_bIsStmContainerInitialized = VOS_TRUE;
    }

    
}


VOS_UINT32 StmAddStateHandler(VOS_UINT32 udwMid,
                              VOS_UINT8 ucMainState,
                              VOS_UINT8 ucSubState,
                              VOS_UINT8 ucSubSubState,
                              VOS_UINT32 udwMsgType,
                              PF_SPU_STATE_FUNC pfnStateFunc,
                              S_STM_RET_STATE *pstRetState)
{
    S_SPU_STATE *pstStateMachine = VOS_NULL_PTR;//指向容器中的状态机
    VOS_UINT32 udwIndex = VOS_NULL_DWORD;
    S_STM_MSG_PROC *pstMsgProc = VOS_NULL_PTR;//消息处理函数列表
    VOS_UINT8 aucStates[M_STM_MAX_STATE_LEVEL];//临时状态机状态，[0]主状态，[1]子状态,[2]子子状态
    VOS_INT32 lRet = VOS_ERR;

    memset_s((VOS_UINT8 *) aucStates, sizeof(aucStates),(VOS_INT8)STATE_NULL, sizeof(aucStates));
                       
    if (VOS_NULL_DWORD == g_udwCurrentStmName)
    {
        return E_STM_IC_NAME_NOT_SPECIFIC;
    }

    pstStateMachine = g_astStmContainer[g_udwCurrentStmName].pstStateMachine;//取得容器中的状态机
    if (VOS_NULL_PTR == pstStateMachine)
    {
        return E_STM_IC_NULL_PTR;
    }


    /* 现在临时状态数组中设置状态机的状态，再检查状态是超过最大值*/
    aucStates[0] = ucMainState;
    aucStates[1] = ucSubState;
    aucStates[2] = ucSubSubState;
    
    udwIndex = GetStateIndex(pstStateMachine, aucStates, M_STM_MAX_STATE_LEVEL);
    if (M_STM_MAX_STATE_INDEX <= udwIndex)
    {
        return E_STM_IC_UNKNOWN_STATE;
    }
    
    if (M_STM_MAX_MSG_PROC <= pstStateMachine[udwIndex].udwMsgProcLen)
    {
        
        SPP_COM_TRACEEX(SPP_COM_ERR_0010, SMP_ERROR_LOG, SPP_COM, 
            "MsgProcLen reachs MAXIMUM(%d)! Current FSM:%d, Current PID:%d.MsgType:0x",
            (M_STM_MAX_MSG_PROC), g_udwCurrentStmName, 
            udwMid, udwMsgType);
        SPP_COM_TRACEEX(SPP_COM_ERR_0011, SMP_ERROR_LOG, SPP_COM, 
            "%X,StateFunc:0x%p,States:%d %d %d. ",
            (VOS_UINTPTR)pfnStateFunc, aucStates[0], aucStates[1], aucStates[2]);
   
        
        /* 已经到达最大可以处理的消息数 */
        return E_STM_IC_OVERFLOW;
    }

    //设置状态机的状态
    lRet = memcpy_s(pstStateMachine[udwIndex].aucStates, sizeof(pstStateMachine[udwIndex].aucStates),aucStates, sizeof(aucStates));
    if (VOS_OK != lRet)
    {
        SPP_COM_TRACE(SPP_COM_ERR_FFFF, SMP_ERROR_LOG, SPP_COM, 
            "StmAddStateHandler memcpy_s lRet = %d.",lRet, VOS_NULL_PTR);
    }
    
    pstStateMachine[udwIndex].udwTimerLen = 0;
    pstStateMachine[udwIndex].udwTimerName = 0;
    pstMsgProc = (pstStateMachine[udwIndex].astMsgProc + pstStateMachine[udwIndex].udwMsgProcLen);
    pstMsgProc->udwMsgType = udwMsgType;
    pstMsgProc->udwMid = udwMid;
    pstMsgProc->pfnStateFunc = pfnStateFunc;
    pstMsgProc->pstRetState = pstRetState;
    pstStateMachine[udwIndex].udwMsgProcLen ++;

    return E_STM_IC_SUCCESS;
}

VOS_VOID StmEndDeclare(VOS_VOID)
{
    g_udwCurrentStmName = VOS_NULL_DWORD;
}



```



------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
