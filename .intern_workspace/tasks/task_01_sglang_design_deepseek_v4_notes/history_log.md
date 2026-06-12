# task_01_sglang_design_deepseek_v4_notes History

<!-- METADATA:SESSION=40 -->

## Session 0-38

- 原任务由 `intern_sglang_learner` 执行，历史记录保存在远端分支 `origin/intern_sglang_learner/task_01_sglang_design_deepseek_v4_notes` 的 `workspace/tasks/task_01_sglang_design_deepseek_v4_notes/history_log.md`。
- 已创建飞书索引和三份主体文档：
  - 索引：https://feishu.cn/docx/MK7HdqZiaouoMtxbLFGczumvnCh
  - 总览与请求调用链：https://feishu.cn/docx/RMYfdhZVyoKeBCxgZpWck2innWh
  - 核心模块逐文件笔记：https://feishu.cn/docx/Cg5bdBxyWourzrxKXzWctWKAnOg
  - DeepSeek V4 分支专项：https://feishu.cn/docx/GtLMdr4EvoSRaQxzS4zcGmrMnuh
- 笔记基准源码为 `origin/deepseek_v4 @ 1b497c7a0c5a2951ae86f21f4cfebe4678bbc7e4`。
- learner 期间已回答过关于 SRT、HTTP/gRPC、ModelOpt、prefill delayer、PP micro-batch、grammar backend、speculative decoding、EAGLE/MTP、MXFP4、DeepEP、redundant expert、expert distribution recorder、HiCache、Double Sparsity、NCCL NVLS、Piecewise CUDA Graph 等问题。

## Session 39

- `intern_sglang_guider` 查询 `intern_sglang_learner` 状态：正式 `.intern_workspace` 显示 Idle、无当前任务、无 PR；tmux 会话存在。
- 读取 learner 的旧 `workspace/tasks/task_00_update_plugin` 和远端分支 `origin/intern_sglang_learner/task_01_sglang_design_deepseek_v4_notes`，确认 `task_01_sglang_design_deepseek_v4_notes` 是需要接手的主要上下文。
- 尝试通过 peer-send 请求 learner 交接摘要，daemon 返回 `session_not_running`；随后用 tmux pane 残留输出补充最近上下文：最近一次问答聚焦 `http_server.py` 中 `obj` 与 `request` 参数来源、FastAPI 注入、`GenerateReqInput` 和 `TokenizerManager.generate_request` 的断连检查。
- 在新版 `.intern_workspace/tasks/` 下创建本任务接手记录，并将 assignee 切换为 `intern_sglang_guider`。
- 创建接手 PR：https://github.com/River861/sglang/pull/2

## Session 40

- 主管询问是否可以删除 `intern_sglang_learner`。
- 按 `internctl-operations` 流程只读复核：`internctl status intern_sglang_learner --project sglang --json` 显示 learner 为 Idle、无当前 task、无 PR；`internctl session status` 显示 tmux session 仍存在且 running。
- 结论：交接上下文已经写入 `intern_sglang_guider` 的任务 metadata 和 PR #2；若主管确认不再需要 learner 的本地 clone/debug/session，可以用 `internctl delete intern_sglang_learner --project sglang --confirm` 删除。不要手动删除目录。
