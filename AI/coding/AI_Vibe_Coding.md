---
title: AI_Vibe_Coding
tags: AI
---

# 两种编程方式
vibe coding：通过对话迭代，将模糊需求逐步变成实际产品
- 需求探索
- 对话累积
- 迭代优化
- 上下文管理

spec coding：需求明确时，先写规范在让AI按规范执行
- vibe探索
- 需求整理
- 规范形成
- 规范执行


# Agent
AI Agent = LLM（核心引擎）+ 四大组件（Memory + Planning + Tools + Action）

# harness
**skills**：封装特定任务能力的可复用模块

**rules**：给agents设置的长期行为准则。**约束AI行为**的结构化指令，确保Agent的输出符合团队规范和安全要求。长期通用的行为准则。

**plans**：agents针对具体任务生成的执行步骤序列

**specs**：对任务、输出、流程的详细要求文档。**可被AI读取和执行的明确、结构化、可验证的标准**。短期具体的任务要求。
- API接口规范
- 编码规范
- 架构决策
 # 三大工程
 - 提示词
 - 上下文
 - harness