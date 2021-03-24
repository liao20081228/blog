---
title: SMC与CHECK IMEI
tags: 4G
---

------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 SMC
```mermaid!
sequenceDiagram
title: 序列图sequence(示例)
participant A
participant B
participant C
participant D as test

note left of A: A左侧说明
note over B,C: 覆盖B,C的说明
note right of C: C右侧说明

A->A:自己到自己
A->B:实线不带箭头
A-->B:虚线不带箭头
A->>B:实线带箭头
A-->>B:虚线带箭头
A-xB:带x带箭头实线（异步）
A--xB:带x带箭头虚线（异步）
C->D:激活
activate C
C->D:不激活
deactivate C

C->+D:激活
C->+D:激活
C->-D:不激活
C->-D:不激活

loop 循环
B->C:测试循环
B->>C:测试循环
B-->C:测试循环
B-->>C:测试循环
end

A->>D:首选路径

alt 替代路径1
A->>C: 
C->>D: 
else 替代路径2
A->>B: 
B->>D: 
end

A-->>C: 首选路径

opt 一种替代路径
A-->>B: 
B-->>D: 
end
```
## 1.1 信令流程

```mermaid!
sequenceDiagram

title: SMC信令流程
participant s1 as s1ap《modul》
participant A as MM_AccConnCtrl
participant M as MM_MainCtrl
participant S as MM_SecCtrl

note over  M: 上级流程触发

M->>S:US_SEC_SMC_REQ
note over S:MM_SecurityInitRecvSmcReq
S->>A:US_AC_NAS_TRANSFER(EMM_SECURITY_MODE_CMD)
A->>s1:SPU_S1AP_DOWNLINK_NAS_TRANSPORT
s1->>A:S1AP_SPU_UPLINK_NAS_TRANSPORT
A->>S:AC_US_NAS_TRANSFER(EMM_SECURITY_MODE_CMP/EMM_SECURITY_MODE_REJECT)
S->>M:SEC_US_SMC_RSP
```
# 2 CHECK IMEI
## 2.1 信令流程
```mermaid!
sequenceDiagram

title: CHECK IMEI信令流程
participant s1 as s1ap《modul》
participant A as MM_AccConnCtrl
participant M as MM_MainCtrl&emsp;
participant S as MM_SecCtrl
participant US as MM_UdmServer
participant U as Udm
note over M: 某个状态
note over of M:某个状态
note over US:空闲态
note over S: 初始状态
M->>S:US_SEC_CHECKIMEI_REQ
note over M:WAIT_CHECKIMEI_RSP
opt 没有取得IMEI或IMEI无效
S-->>A:US_AC_NAS_TRANSFER(EMM_IDEN_REQ)
note over S:等待终端IMEI响应
A-->>s1:SPU_S1AP_DOWNLINK_NAS_TRANSPORT
s1-->>A:S1AP_SPU_UPLINK_NAS_TRANSPORT
A-->>S:AC_US_NAS_TRANSFER(EMM_IDEN_RSP)
end

alt 当g_udwMmImeiChkSwitch==MM_GET_IMEI_YES
S->>US:MM_UDM_NOTIFY_REQ

else 当g_udwMmImeiChkSwitch==MM_CHECK_IMEI_YES
S->>U:MM_UDM_CHK_IMEI_REQ
note over S:等待UDM SERVER检查IMEI响应
U->>US:UDM_MM_CHK_IMEI_RSP
US->>S:UDMSERV_MM_CHECK_IMEI_RSP
end
note over US:空闲态
S->>M:SEC_US_CHECKIMEI_RSP
note over S:初始状态
note over M:某个状态


```




------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
