# task_01_sglang_design_deepseek_v4_notes History

<!-- METADATA:SESSION=3 -->

## Session 0

- 创建任务 `task_01_sglang_design_deepseek_v4_notes`
- 任务内容：阅读 SGLang deepseek_v4 相关代码和官方文档，输出中文飞书学习笔记
- 接受任务并创建 PR：https://github.com/River861/sglang/pull/1
- 基于 `origin/deepseek_v4 @ 1b497c7a0` 阅读核心链路、runtime、cache、attention、MoE、speculative、PD disaggregation 和 DeepSeek V4 专项实现
- 创建飞书笔记索引：https://feishu.cn/docx/MK7HdqZiaouoMtxbLFGczumvnCh
- 创建飞书主体文档 1：SGLang 架构学习指南 1：总览与请求调用链：https://feishu.cn/docx/RMYfdhZVyoKeBCxgZpWck2innWh
- 创建飞书主体文档 2：SGLang 架构学习指南 2：核心模块逐文件笔记：https://feishu.cn/docx/Cg5bdBxyWourzrxKXzWctWKAnOg
- 创建飞书主体文档 3：SGLang 架构学习指南 3：DeepSeek V4 分支专项：https://feishu.cn/docx/GtLMdr4EvoSRaQxzS4zcGmrMnuh
- 已用 `get_doc.py --format text` 读回索引文档，确认链接和目录已写入

## Session 1

- 诊断本机 Claude Code 未响应问题：`claude -p '只回复 OK'` 超时，debug 日志显示请求被发往 `ANTHROPIC_BASE_URL=http://10.100.193.54:8006` 后持续重试。
- 确认 API key/网关链路可用：`/health` 和 `/v1/models` 均正常，`claude-sonnet-4-6` 与 `claude-opus-4-7` 精确模型均能返回 `OK`。
- 定位根因：`~/.claude/settings.json` 中 `"model": "opus"` 被 Claude Code 解析为 `claude-opus-4-8`，但当前网关模型列表没有该模型，返回 `model_not_found`，因此表现为长时间无响应。
- 修复方式：将本机 `~/.claude/settings.json` 的默认模型改为网关支持的 `claude-opus-4-7`，并用默认 `claude -p` 复测返回 `OK`。

## Session 2

- 回答用户关于 SGLang 源码路径的问题：当前源码根目录为 `/work-agents/intern_sglang_learner/sglang/`。

## Session 3

- 核查 `/work-agents/intern_sglang_learner/sglang_deepseek_v4`：它是主仓库 `/work-agents/intern_sglang_learner/sglang` 的 Git linked worktree，不是独立 clone。
- 该 worktree 当前处于 detached HEAD，指向 `origin/deepseek_v4 @ 1b497c7a0c5a2951ae86f21f4cfebe4678bbc7e4`，提交信息为 `little fix deepgemm hash on gb image`。
- `sglang_deepseek_v4` 工作区干净，适合作为 DeepSeek V4 分支代码阅读目录；常规任务状态与 PR 仍维护在 `/work-agents/intern_sglang_learner/sglang/` 当前分支。
