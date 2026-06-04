# task_01_sglang_design_deepseek_v4_notes Knowledge

<!-- METADATA:SESSION=32 -->

## 编写规则

- 记录与 SGLang 架构、调用链、deepseek_v4 实现和官方文档相关的事实
- 代码阅读结论需尽量落到具体文件或类/函数
- 不记录无关环境噪声

## 知识条目

- `python/sglang/launch_server.py` 是服务入口，普通 HTTP 服务最终进入 `sglang.srt.entrypoints.http_server.launch_server`。
- `python/sglang/srt/entrypoints/engine.py` 的 docstring 明确 SRT 三组件模型：TokenizerManager 主进程、Scheduler 子进程、DetokenizerManager 子进程，组件间用 ZMQ IPC。
- OpenAI chat 请求主要经 `http_server.py` -> `serving_chat.py` -> `TokenizerManager.generate_request` -> `Scheduler` -> `TpModelWorker` -> `ModelRunner` -> `DetokenizerManager`。
- `python/sglang/srt/managers/schedule_batch.py` 给出关键数据流：`ScheduleBatch -> ModelWorkerBatch -> ForwardBatch`。`ScheduleBatch` 是 scheduler 侧高层调度数据，`ModelWorkerBatch` 是 worker 输入，`ForwardBatch` 是 ModelRunner/GPU 执行数据。
- Prefix cache 主路径是 `Req.init_next_round_input(tree_cache)` 查询 radix tree，`radix_cache.cache_finished_req/cache_unfinished_req` 在请求完成或 chunk 后插入新 KV。
- Structured outputs 由 `GrammarManager` 异步编译 grammar，`SamplingBatchInfo.update_regex_vocab_mask` 在 sample 前生成 logits mask，输出 token 在 scheduler output processor 中 `grammar.accept_token` 推进状态。
- Attention backend 由 `attention_registry.py` 注册；官方文档中的 `flashinfer`、`triton`、`fa3/fa4`、`trtllm_mha/mla`、`flashmla`、`nsa`、`compressed` 等名字在这里落地。
- DeepSeek V4 在 `server_args.py` 中强制使用 `attention_backend='compressed'`、`page_size=256`，`kv_cache_dtype` 默认为 `fp8_e4m3`，speculative 仅支持 EAGLE 且 topk=1。
- DeepSeek V4 OpenAI chat 编码走 `serving_chat.py` 的 `chat_encoding_spec='dsv4'` 分支，并调用 `encoding_dsv4.encode_messages`；DSML tool-call 外层 block 是 `<｜DSML｜tool_calls>`。
- `python/sglang/srt/models/deepseek_v4.py` 的模型结构是 `DeepseekV4ForCausalLM -> DeepseekV4Model -> DeepseekV4DecoderLayer -> MQALayer + DeepseekV2MoE`，并引入 HC pre/post/head、多路 hidden mixing、C4/C128 compressor 和 C4 indexer。
- DeepSeek V4 专用 KV cache 在 `deepseekv4_memory_pool.py` 中组合 `swa_kv_pool`、`c4_kv_pool`、`c128_kv_pool`、`c4_indexer_kv_pool` 和压缩状态池，按 `compress_ratios` 建 `layer_mapping`。
- DeepSeek V4 专用 attention backend 是 `deepseek_v4_backend_radix.py::DeepseekV4BackendRadix`；它构造 `DSV4AttnMetadataRadix`，同时管理 SWA、C4 sparse、C128 metadata，并调用 FlashMLA entrypoint。
- `compressed/compressor.py` 负责 C4/C128 压缩 KV 的 fused compress、norm、rope 和写入；`compressed/indexer.py` 负责 C4 indexer logits 与 top-k page 选择。
- DeepSeek V4 JIT kernel 集中在 `python/sglang/jit_kernel/deepseek_v4.py` 和 `python/sglang/jit_kernel/csrc/deepseek_v4/*.cuh`，覆盖 compress、topk、hash_topk、store、paged MQA metadata、Silu/mul quant、MegaMoE、HiSparse transfer。
- DeepSeek V4 MoE 复用大量 `deepseek_v2.py` 逻辑；V4 top-k 路由在 `layers/moe/deepseek_v4_topk.py`，DeepGEMM runner 在 `layers/moe/moe_runner/deep_gemm.py`。
- PD disaggregation 的 V4 路径要求 PP=1，且 prefill/decode TP size 保持一致，因为 `DeepSeekV4TokenToKVPool` 的 buffer 按类型组织，不适合普通 PP 切分方式。
- 飞书文档产出：
  - 索引：https://feishu.cn/docx/MK7HdqZiaouoMtxbLFGczumvnCh
  - 总览与请求调用链：https://feishu.cn/docx/RMYfdhZVyoKeBCxgZpWck2innWh
  - 核心模块逐文件笔记：https://feishu.cn/docx/Cg5bdBxyWourzrxKXzWctWKAnOg
  - DeepSeek V4 分支专项：https://feishu.cn/docx/GtLMdr4EvoSRaQxzS4zcGmrMnuh
- 配合上述飞书笔记学习时，源码优先使用 `/work-agents/intern_sglang_learner/sglang_deepseek_v4/`，因为该 worktree 指向笔记基准 `origin/deepseek_v4 @ 1b497c7a0`；`/work-agents/intern_sglang_learner/sglang/` 用于任务记录和 PR 状态维护。
- 熟悉仓库的第一步建议从 `python/sglang/launch_server.py` 开始，它是 server 启动入口，读它可以建立 CLI 参数、ServerArgs 和 HTTP server 启动链路的整体框架。
- `sglang.srt` 的 `srt` 未在本地源码/文档中发现明确展开；结合目录职责，它更可能表示 `SGLang Runtime`。
- gRPC server 通常用 protobuf 定义服务接口，运行在 HTTP/2 之上，适合强类型、高性能、服务间调用；HTTP server 通常暴露 REST/JSON 或浏览器友好的接口，适合人工调试和通用客户端访问。
- ModelOpt 是 NVIDIA Model Optimizer；SGLang deepseek_v4 代码中主要通过 `modelopt_fp8`、`modelopt_fp4`、`--modelopt-quant` 等配置把它用于模型量化和量化 checkpoint 的加载/保存/导出。
- SGLang 的 ModelOpt 集成同时支持离线量化和在线量化：离线量化先生成/导出量化 checkpoint 或 HF 格式模型再加载，适合生产；在线量化用 `--quantize-and-serve` 在启动时量化并 serving，适合开发验证。
- Intern Agent Helper 安装新版本后，飞书 UI 是否变化取决于正在运行的扩展/daemon 是否已重启；如果 `feishu_daemon.py` 仍来自旧扩展目录，需要 `Reload Window` 让新版本接管。
- 2026-06-04 检查发现：Intern Agent Helper 1.6.0 已被 VS Code 识别为安装版本，但 1.6.0 `feishu_daemon.py` 会因 relay credentials HTTP 404 退出；为保持飞书通道可用，当前 daemon 回退运行 1.5.3。
- `--enable-prefill-delayer` 启用 `managers/prefill_delayer.py::PrefillDelayer`，在 DP attention 下跨 DP rank 协商是否允许 prefill；mixed prefillable 状态会延迟 prefill，等待 rank 对齐，以减少 prefill idle，最多延迟 `--prefill-delayer-max-delay-passes`。
- SGLang 中 forward pass 可理解为一次 scheduler batch 的模型前向执行；token usage 是 token/KV cache 池占用率，普通路径由 `_get_token_info()` 计算为 `num_used / max_total_num_tokens`。
- `--prefill-delayer-forward-passes-buckets` 和 `--prefill-delayer-wait-seconds-buckets` 只配置 PrefillDelayer metrics histogram 分桶；前者统计等待的 forward pass 数，后者统计等待秒数，不改变调度决策。
- PrefillDelayer buckets 数组中的每个元素是 histogram 的桶上界：例如 forward passes buckets 的 `4` 表示统计“等待 forward pass 数 <= 4”的累计桶，wait seconds buckets 的 `0.1` 表示统计“等待秒数 <= 0.1”的累计桶。
- `--pp-max-micro-batch-size` 是 pipeline parallelism 下单个 PP micro-batch 的请求数上限；未设置时默认为 `max(max_running_requests // pp_size, 1)`，调度中用于限制 `pp_max_micro_batch_size - running_bs` 个可新增请求。
- `--constrained-json-whitespace-pattern` 控制 constrained JSON 输出中语法空白的 regex，适用于 outlines/llguidance；`--constrained-json-disable-any-whitespace` 强制紧凑 JSON 表示，适用于 xgrammar/llguidance，并通过 `any_whitespace=False` 生效。
- `--grammar-backend` 选择 structured output 的 grammar-guided decoding backend；可选 `xgrammar`、`outlines`、`llguidance`、`none`，未显式设置时 `_handle_grammar_backend()` 会设为 `xgrammar`。
- `SpeculativeAlgorithm.NGRAM` 是 speculative decoding 的 ngram 变体：普通 EAGLE/STANDALONE 路径依赖 draft model/MTP 猜 token，NGRAM 路径依赖 `NgramCache` 从历史 token 模式匹配 continuation，再用 target model 验证候选。
- EAGLE 是 speculative decoding 算法/执行框架；MTP 是模型内 Multi-Token Prediction draft 能力。SGLang 中 DeepSeek MTP 通过 `--speculative-algorithm EAGLE` 使用，draft architecture 会切到 `DeepseekV3ForCausalLMNextN`/`DeepseekV4ForCausalLMNextN`。
- MXFP4 是 microscaling FP4：E2M1 FP4 数值按小 block 共享 E8M0 scale。SGLang 代码默认 block size 32，两个 FP4 packed 到一个 `uint8`，常用于 MoE expert 权重量化和 FlashInfer/DeepSeek/GPT-OSS MXFP4 kernel 路径。
- MXFP4 的 shared scale 可理解为每 32 个数共用一个“单位”：先把原始数除以该单位压到 FP4 小值域，存储 FP4 小数值和一个 scale；使用时再用 `FP4_value * scale` 近似还原。
- DeepEP 是 SGLang MoE expert parallel 的 A2A/token dispatcher 后端，通过 `--moe-a2a-backend deepep` 启用；核心流程为 dispatch token 到专家所在 rank、执行 local expert compute、combine 聚合结果，`--deepep-mode auto` 对 prefill 使用 normal、decode 使用 low_latency。
- EP redundant expert 是 logical expert 的额外 physical replica：SGLang 用 `num_logical_experts + ep_num_redundant_experts` 建物理专家数，EPLB 复制热点专家并在 dispatch 阶段把 logical top-k expert id 映射到某个 physical copy，用于负载均衡而不是改变模型能力。
- `--expert-distribution-recorder-mode` 启用 MoE expert 分布记录器；`stat`/`stat_approx` 输出聚合统计用于 EPLB 和 balancedness，`per_pass` 逐 forward pass 保存聚合分布，`per_token` 额外保存 token 级 top-k expert ids；启动 EPLB 或 expert distribution metrics 时默认补为 `stat`。
- `--hicache-write-policy` 控制 HiCache 从快层到慢层的 KV 写入时机：`write_through` 命中即备份，`write_through_selective` 基于 hit count 只备份热点节点，`write_back` 仅在 eviction 时写回；默认是 `write_through`。
