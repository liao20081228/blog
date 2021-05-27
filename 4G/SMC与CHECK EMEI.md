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
participant C as Cm

note over M: 某个状态
note over US:空闲态
note over S: 初始状态
note over U:空闲态
M->>S:US_SEC_CHECKIMEI_REQ
note over M:等待检查IMEI响应
opt attach流程或CB中的IMEI无效（全1）
S-->>A:US_AC_NAS_TRANSFER(EMM_IDEN_REQ)
note over S:等待终端IMEI响应
A-->>s1:SPU_S1AP_DOWNLINK_NAS_TRANSPORT
s1-->>A:S1AP_SPU_UPLINK_NAS_TRANSPORT
A-->>S:AC_US_NAS_TRANSFER(EMM_IDEN_RSP)
end

alt 当g_udwMmImeiChkSwitch==MM_GET_IMEI_YES且CB、SDB中的IMEI有效
S->>U:MM_UDM_NOTIFY_REQ
U->>C:SP_CM_NOTIFY_REQ

else 当g_udwMmImeiChkSwitch==MM_CHECK_IMEI_YES
S->>U:MM_UDM_CHK_IMEI_REQ
note over S:等待UdmServer检查IMEI响应
U->>US:UDM_MM_CHK_IMEI_RSP
US->>S:UDMSERV_MM_CHECK_IMEI_RSP
end
note over US:空闲态
S->>M:SEC_US_CHECKIMEI_RSP
note over S:初始状态
note over M:某个状态


```
U-->>S:UDM_MM_NOTIFY_RSP(发送给MM模块)
# test

```mermaid!
sequenceDiagram

title: CHECK IMEI信令流程
participant s1 as s1ap《modul》
participant A as MM_AccConnCtrl
participant M as MM_MainCtrl&emsp;
participant S as MM_SecCtrl
participant US as MM_UdmServer
participant U as Udm
participant C as Cm

note over M: 某个状态
note over US:空闲态
note over S: 初始状态
note over U:空闲态
M->>S:US_SEC_CHECKIMEI_REQ
note over M:等待检查IMEI响应

alt g_udwMmImeiChkSwitch==MM_GET_IMEI_YES
opt CB中的IMEI无效（全1）
S-->>A:US_AC_NAS_TRANSFER(EMM_IDEN_REQ)
note over S:等待终端IMEI响应
A-->>s1:SPU_S1AP_DOWNLINK_NAS_TRANSPORT
s1-->>A:S1AP_SPU_UPLINK_NAS_TRANSPORT
A-->>S:AC_US_NAS_TRANSFER(EMM_IDEN_RSP)
end
opt  SDB、CB中的IMEI不相等
S-->>U:MM_UDM_NOTIFY_REQ
U-->>C:SP_CM_NOTIFY_REQ
note over U:等待Notify响应
C-->>U:CM_SP_NOTIFY_RSP
end

else g_udwMmImeiChkSwitch==MM_CHECK_IMEI_YES
opt attach流程或CB中的IMEI无效（全1）
S-->>A:US_AC_NAS_TRANSFER(EMM_IDEN_REQ)
note over S:等待终端IMEI响应
A-->>s1:SPU_S1AP_DOWNLINK_NAS_TRANSPORT
s1-->>A:S1AP_SPU_UPLINK_NAS_TRANSPORT
A-->>S:AC_US_NAS_TRANSFER(EMM_IDEN_RSP)
end
S->>U:MM_UDM_CHK_IMEI_REQ
U->>C:SP_CM_CHK_IMEI_REQ
note over U:等待CHECK EMEI响应
C->>U:CM_SP_CHK_IMEI_RSP
note over S:等待UdmServer检查IMEI响应
U->>US:UDM_MM_CHK_IMEI_RSP
US->>S:UDMSERV_MM_CHECK_IMEI_RSP

end
note over U:空闲态
note over US:空闲态
S->>M:SEC_US_CHECKIMEI_RSP
note over S:初始状态
note over M:某个状态


```

# main
```mermaid!
sequenceDiagram

title: 分析平台调用算法流程
participant P as 算法
participant V as 分析平台
participant S as 流平台

V->>P:初始化请求
P-->>V:OK
V->>P:创建线程
P-->>V:线程ID
V->>S:开始传送图片流
S-->>V:图片流分析


```
# rw
```shell




```
------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
