# task_01_sglang_design_deepseek_v4_notes

<!-- METADATA:STATUS=InProgress,ASSIGNEE=intern_sglang_guider -->

## 背景

主管希望系统学习 SGLang 框架的整体设计和调用流程，并以 `deepseek_v4` 分支为例，结合代码文件和官方文档整理中文阅读笔记。

本任务原由 `intern_sglang_learner` 承接；`intern_sglang_guider` 于 2026-06-12 接手。接手时，learner 的正式 `.intern_workspace` 状态为 Idle，但远端分支 `origin/intern_sglang_learner/task_01_sglang_design_deepseek_v4_notes` 保留了详细历史、知识和飞书文档链接。

## 目标

- 阅读 `origin/deepseek_v4` 分支中的 SGLang 代码结构。
- 按关键代码文件总结整体架构、调用流程、特性设计和实现细节。
- 结合官方高级特性文档：https://docs.sglang.io/docs/advanced_features/overview
- 维护中文阅读笔记和问答上下文，必要时补充飞书文档。
- 继续回答主管围绕 SGLang、DeepSeek V4、SRT runtime、scheduler、attention、MoE、HiCache、speculative decoding 等方向的问题。

## 已有产出

- 飞书索引：https://feishu.cn/docx/MK7HdqZiaouoMtxbLFGczumvnCh
- 总览与请求调用链：https://feishu.cn/docx/RMYfdhZVyoKeBCxgZpWck2innWh
- 核心模块逐文件笔记：https://feishu.cn/docx/Cg5bdBxyWourzrxKXzWctWKAnOg
- DeepSeek V4 分支专项：https://feishu.cn/docx/GtLMdr4EvoSRaQxzS4zcGmrMnuh

## 验收标准

- 能接续 learner 已有飞书文档、任务历史和代码阅读上下文。
- 后续回答应落到具体源码路径、类、函数或参数，不只给概念解释。
- 如继续产出文档，最终回复中给出飞书文档链接和覆盖范围说明。
