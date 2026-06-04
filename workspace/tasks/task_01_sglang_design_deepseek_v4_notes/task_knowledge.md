# task_01_sglang_design_deepseek_v4_notes Knowledge

<!-- METADATA:SESSION=3 -->

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
