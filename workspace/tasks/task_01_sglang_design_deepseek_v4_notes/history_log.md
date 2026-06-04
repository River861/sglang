# task_01_sglang_design_deepseek_v4_notes History

<!-- METADATA:SESSION=27 -->

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

## Session 4

- 结合飞书笔记基准说明源码阅读选择：笔记基于 `origin/deepseek_v4 @ 1b497c7a0`，因此学习 DeepSeek V4 专项时优先阅读 `/work-agents/intern_sglang_learner/sglang_deepseek_v4/`。
- 建议将 `/work-agents/intern_sglang_learner/sglang/` 作为任务记录、PR 状态维护和必要时对照当前任务分支的目录。
- 给出建议阅读顺序：先读飞书索引和“总览与请求调用链”，再按入口、manager、model runner、DeepSeek V4 专项文件逐层对照源码。

## Session 5

- 用户要求答复模式改为简洁版：默认只回复答案，不打印过程日志；项目强制的 Session checklist 仍需保留。

## Session 6

- 用户已进入 `/work-agents/intern_sglang_learner/sglang_deepseek_v4`，询问熟悉仓库的第一步。
- 建议第一步阅读 `python/sglang/launch_server.py`，目标是先理解 SGLang server 启动入口如何解析参数并转入 SRT HTTP server。

## Session 7

- 响应用户在线确认，继续保持简洁答复模式。

## Session 8

- 诊断 VS Code Codex 插件报错 `Missing environment variable: LB_API_KEY`。
- 确认 `/root/.codex/config.toml` 使用 `model_provider = "lb"`，该 provider 的 `env_key` 要求插件进程环境中存在 `LB_API_KEY`。
- 发现 `LB_API_KEY` 已写在 shell 启动文件中，但 `.bashrc` 对非交互 shell 会提前返回，VS Code 远程扩展进程未继承该变量。
- 已创建 `/root/.vscode-server-insiders/server-env-setup` 和 `/root/.vscode-server/server-env-setup`，从现有 shell 配置导入 `LB_API_KEY`，不复制密钥明文；需要重启远程 VS Code Server 或重新连接窗口后生效。

## Session 9

- 回答用户关于 `sglang.srt` 中 `srt` 缩写的问题：未在本地源码/文档中发现明确展开；结合模块职责判断更可能指 `SGLang Runtime`。

## Session 10

- 排查用户收到 `[River861/sglang] PR run failed: PR Test Extra` 邮件的问题。
- 确认 PR #1 当前状态为 CLOSED，当前分支没有 queued/in_progress 的 GitHub Actions run。
- 失败来源是历史 `PR Test Extra` workflow：该 workflow 会在 PR push 时启动，并在缺少 `run-ci-extra` 标签时由 `call-gate / pr-gate` 失败，从而触发 GitHub 失败邮件。
- 当前 PR 只有 `documentation` 标签，没有 `run-ci`/`run-ci-extra`；PR 已关闭后该 PR 不会再因为后续分支 push 触发新的 PR workflow。
- 尝试取消当前用户对 issue/PR #1 的显式订阅，GitHub API 返回 404，表示没有显式 PR 订阅；repo 级通知订阅查询需要额外 `notifications` scope，未强行刷新 GitHub auth。

## Session 11

- 按用户请求升级 Intern Agent Helper 插件：在 `/work-agents/axis_intern_agents` 切到 `master` 并 `git pull --rebase origin master`，确认已是最新。
- 选择 `releases/` 中版本最高的 `/work-agents/axis_intern_agents/releases/intern-agent-helper-1.6.0.vsix` 并安装。
- 处理当前 shell 中过期的 `VSCODE_IPC_HOOK_CLI`，改用可响应的 VS Code Insiders IPC socket 完成安装，确认 `llm-intern-agents.intern-agent-helper@1.6.0`。
- 删除 `/work-agents/.github/hooks/.version`，让 hooks 在 Reload Window 后重新解压覆盖。

## Session 12

- 回答用户关于 gRPC server 与 HTTP server 区别的问题：gRPC 是基于 HTTP/2 和 protobuf 的 RPC 服务形态，HTTP server 通常指 REST/JSON 风格的通用 Web API 服务。

## Session 13

- 回答用户关于 ModelOpt 的问题：ModelOpt 指 NVIDIA Model Optimizer，是 NVIDIA 的模型优化库。
- 结合 SGLang deepseek_v4 代码说明：SGLang 主要把 ModelOpt 用在量化集成上，包括 `modelopt_fp8`、`modelopt_fp4`、`--modelopt-quant`、ModelOpt checkpoint restore/save/export 等路径。

## Session 14

- 回答用户关于 ModelOpt 是离线量化还是在线量化的问题：SGLang 支持两种模式。
- 离线路径是预先用 ModelOpt 生成/导出量化 checkpoint 或 HuggingFace 格式模型，然后 SGLang 直接加载；启动快，适合生产。
- 在线路径是通过 `--modelopt-quant`、`--quantize-and-serve` 等参数在启动时量化并直接 serving；方便实验，但启动慢且初始化显存压力更大。

## Session 15

- 回答用户关于 Intern Agent Helper 版本和飞书回复 UI 未变化的问题。
- 确认 VS Code Insiders 扩展目录中已安装 `llm-intern-agents.intern-agent-helper-1.6.0`。
- 同时发现当前运行中的 `feishu_daemon.py` 仍来自 `llm-intern-agents.intern-agent-helper-1.5.3` 路径，因此飞书回复 UI 仍表现为旧版本；需要 `Reload Window` 或重启 VS Code 远程扩展主机后生效。

## Session 16

- 用户 Reload Window 后再次询问当前是否使用 1.6.0。
- 确认 VS Code Insiders 当前扩展列表显示 `llm-intern-agents.intern-agent-helper@1.6.0`。
- 发现实际运行的 `feishu_daemon.py` 仍来自 1.5.3；尝试停止旧 daemon 并手动启动 1.6.0 daemon，但 1.6.0 daemon 因 relay credentials 接口返回 HTTP 404 而退出。
- 为避免飞书通道中断，已恢复启动 1.5.3 daemon；因此当前“安装版本”为 1.6.0，但“飞书 daemon 实际运行版本”为 1.5.3。

## Session 17

- 响应用户在线确认，继续保持简洁答复模式。

## Session 18

- 回答用户关于 `--enable-prefill-delayer` 的问题。
- 该参数启用 `PrefillDelayer`，用于 DP attention 场景下减少 prefill 阶段的空转/等待。
- 核心逻辑：各 DP rank 通过 CPU group 汇总本地是否有 prefillable 请求；如果所有 rank 都能 prefill 则放行，如果只有部分 rank 能 prefill 则最多延迟若干 forward pass，等待更多 rank 对齐后再 prefill。
- 约束条件：需要 `enable_dp_attention=True`，`disaggregation_mode="null"`，且不能禁用 overlap schedule。

## Session 19

- 回答用户关于 forward pass 和 token 使用率含义的问题。
- forward pass 指模型执行一次前向计算；在 SGLang scheduler 中可理解为一次调度出来的 batch 被送进模型跑一轮，prefill 可能处理一段 prompt token，decode 通常为每个活跃请求生成下一个 token。
- token usage 指 token/KV cache 池占用比例，普通路径中计算为 `num_used / max_total_num_tokens`，其中 `num_used = max_total_num_tokens - (available_size + evictable_size)`。
- 在 prefill delayer 中，`max_delay_passes` 是最多延迟多少个模型前向轮次；`token_usage_low_watermark` 是当 KV/token 池占用率很低时允许不再延迟 prefill。

## Session 20

- 回答用户关于 `--prefill-delayer-forward-passes-buckets` 和 `--prefill-delayer-wait-seconds-buckets` 的问题。
- 两个参数只配置 Prometheus histogram 分桶，用于观测 PrefillDelayer 等了多少 forward pass、等了多少秒，不直接改变 prefill delayer 的放行/延迟决策。
- forward passes 默认 buckets 为 `[5, 20, 50, 100, 200]`，会过滤掉 `>= max_delay_passes` 的值，并自动加入 `0` 和 `max_delay_passes - 1`。
- wait seconds 默认 buckets 为 `[1, 2, 5, 10, 20, 50, 100, 200, 500]`，并自动加入 `0`。

## Session 21

- 进一步解释 buckets 数组含义：数组是 histogram 的多个阈值边界，每个元素表示一个“<= 这个值”的统计桶上界。
- `--prefill-delayer-forward-passes-buckets 1 2 4 8` 表示统计等待 forward pass 数落在 `<=1`、`<=2`、`<=4`、`<=8` 等累计桶中。
- `--prefill-delayer-wait-seconds-buckets 0.01 0.05 0.1 0.5` 表示统计等待秒数落在 `<=0.01s`、`<=0.05s`、`<=0.1s`、`<=0.5s` 等累计桶中。
- 这些数组元素只改变监控统计粒度，不改变 PrefillDelayer 实际等待多久。

## Session 22

- 回答用户关于 `--pp-max-micro-batch-size` 的问题。
- 该参数只在 pipeline parallelism 场景下有意义，用于限制单个 PP micro-batch 最多包含多少个请求。
- 若未设置，scheduler 会设置为 `max(max_running_requests // pp_size, 1)`。
- 调度时 `get_num_allocatable_reqs(running_bs)` 使用 `pp_max_micro_batch_size - running_bs` 限制本轮 micro-batch 还能加入多少请求，并受 `req_to_token_pool.available_size()` 进一步限制。
- 该值可通过 `/set_internal_state` 动态调整，合法范围为 `[1, max_running_requests // pp_size]`。

## Session 23

- 回答用户关于 `--constrained-json-whitespace-pattern` 和 `--constrained-json-disable-any-whitespace` 的问题。
- `--constrained-json-whitespace-pattern` 用于 constrained JSON 输出中 JSON 语法空白的 regex 控制，只传给 outlines 与 llguidance backend。
- `--constrained-json-disable-any-whitespace` 用于强制紧凑 JSON 表示，只传给 xgrammar 与 llguidance backend，代码里通过 `any_whitespace=not constrained_json_disable_any_whitespace` 生效。
- 这两个参数影响 JSON 结构 token 之间的空白生成，不改变 JSON schema 本身，也不影响字符串值内部内容。

## Session 24

- 回答用户关于 `--grammar-backend` 的问题。
- `--grammar-backend` 用于选择 grammar-guided constrained decoding backend，可选 `xgrammar`、`outlines`、`llguidance`、`none`。
- `ServerArgs._handle_grammar_backend()` 在未显式设置时把默认 backend 设为 `xgrammar`。
- `GrammarManager` 在请求带 `json_schema`、`regex`、`ebnf` 或 `structural_tag` 时使用该 backend 编译 grammar；若设置为 `none`，这些 grammar-based generation 请求会被 abort。
- 采样阶段通过 grammar object 生成 vocab mask，并在输出 token 后调用 `accept_token` 推进 grammar 状态。

## Session 25

- 回答用户关于 Speculative decoding 与 Speculative decoding (ngram) 区别的问题。
- SGLang 的 `SpeculativeAlgorithm` 包含 `EAGLE`、`EAGLE3`、`STANDALONE`、`NGRAM`、`NONE`；`NGRAM` 是 speculative decoding 的一种特殊算法。
- 普通 speculative decoding 通常由 draft model 或模型内 MTP/EAGLE draft 路径先猜多个 token，再由 target model 一次 forward 验证。
- NGRAM speculative decoding 不加载神经网络 draft model，而是通过 `NgramCache` 对当前上下文后缀做 ngram 匹配，取历史 continuation 作为候选 token tree，再由 target model 验证。
- deepseek_v4 分支中 NGRAM 仅支持 CUDA，且会禁用 overlap scheduler 与 mixed chunked prefill；其参数主要是 `--speculative-ngram-*`。

## Session 26

- 回答用户关于 EAGLE 与 MTP 区别的问题。
- EAGLE 是 speculative decoding 算法/服务端执行框架，负责 draft token 生成、token tree 组织、target model 验证与接受 token。
- MTP 是 Multi-Token Prediction，指模型自身带有用于预测未来多个 token 的 draft 能力/权重模块，例如 DeepSeek 的 NextN/MTP 模块。
- 在 SGLang DeepSeek 路径中，MTP 通过 `--speculative-algorithm EAGLE` 启用；`NEXTN` 会映射成 `EAGLE`，DeepSeek draft model path 未显式设置时会使用同一模型路径，并把 draft architecture 改写为 `DeepseekV3ForCausalLMNextN` 或 `DeepseekV4ForCausalLMNextN`。
- 因此 MTP 是 draft token 的来源之一，EAGLE 是使用这些 draft token 并验证加速的算法路径。

## Session 27

- 回答用户关于 MXFP4 含义的问题。
- MXFP4 指 microscaling FP4：每个数值用 FP4 表示，一小块数值共享一个 scale。
- deepseek_v4 分支代码中的 `MXFP4QuantizeUtil` 使用 E2M1 FP4 值域，默认 block size 为 32，并将两个 FP4 值打包到一个 `uint8`。
- scale 采用 E8M0 形式；反量化时近似为 `E2M1_value * 2 ** (scale - 127)`。
- SGLang 中 `mxfp4` 主要用于量化权重/激活，尤其是 MoE expert 权重、GPT-OSS/DeepSeek/FlashInfer MXFP4 kernel 路径，以降低显存和带宽成本。
