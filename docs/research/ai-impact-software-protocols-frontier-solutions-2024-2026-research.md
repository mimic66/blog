# AI 对软件系统依赖协议的影响与前沿解决办法（2024–2026）

> 调研日期：2026-11-10  
> 时间范围：以 2024–2026 年为主；对 Orca、vLLM、DCQCN、RoCEv2、CRAQ 等必要前置工作做历史回溯。  
> 研究问题：AI/大模型如何改变软件系统依赖的网络、应用、数据、存储、协调、安全与运维协议？目前有哪些前沿解决办法，哪些已经生产验证，哪些仍处于研究或炒作阶段？

## 证据分级

| 等级 | 定义 | 使用方式 |
|---|---|---|
| A | RFC/正式标准、官方协议规范、顶级会议同行评审论文、带 artifact 的系统论文 | 可直接支撑强结论 |
| B | 官方文档、官方仓库、正式发布说明、官方工程博客 | 可描述设计和实践；性能数字按实现方自报处理 |
| C | 社区材料、媒体、教程、厂商赞助调查、无法取得全文的摘要 | 只能作为线索，不单独支撑强结论 |

## 分类口径

- **协议重构**：消息边界、状态位置、顺序、可靠性、安全域或资源模型发生变化。
- **协议协同**：协议本身变化有限，但需要与调度、硬件、存储、策略和可观测性联动。
- **工程产品化**：已有思想在 AI 场景中变成可用产品或开源实现。
- **概念炒作**：缺少一手证据，或把连接、认证、事务、恢复等不同层次的问题混为一谈。

---

## 0. 执行摘要：12 个关键结论

1. **AI 没有产生一个“统一互联网式 Agent 协议”，而是推动协议栈重新分层。** 当前至少应区分：底层 fabric、集合通信、推理 API、流式传输、工具连接、Agent 任务委托、持久执行、数据/存储、身份授权和可观测性。MCP 与 A2A 位于不同层，不是互相替代关系。  
   证据：A；MCP Architecture（https://modelcontextprotocol.io/specification/2025-03-26/architecture）、A2A Specification（https://a2a-protocol.org/latest/specification/）

2. **训练网络的第一目标从“通用主机互联”转向“集合通信原语优化”。** AllReduce、AllGather、ReduceScatter、AllToAll 和 MoE expert dispatch 决定网络拓扑、拥塞控制、路由和调度；NCCL、SHARP、rail-aligned topology、NVLink、UEC 都围绕这些模式工作。  
   证据：A/B；NCCL docs（https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/index.html）、MegaScale（https://www.usenix.org/system/files/nsdi24-jiang-ziheng.pdf）

3. **InfiniBand、以太网 RDMA/RoCE、云厂商自定义传输仍并存，没有单一赢家。** InfiniBand 强在完整 HPC/AI 生态；RoCE 强在以太网基础和云运维；AWS EFA/SRD、Google ICI、阿里 HPN、字节 MegaScale 都说明超大规模厂商会选择深度定制路线。  
   证据：A/B；Meta RoCE（https://doi.org/10.1145/3651890.3672233）、AWS SRD（https://www.amazon.science/publications/a-cloud-optimized-transport-protocol-for-elastic-and-scalable-hpc）

4. **AI 以太网正在弱化“全局 PFC 无损”的单一路线。** Packet spraying、selective retransmission、adaptive multipath、sender window、receiver credit、端到端拥塞控制和细粒度遥测成为 UEC 方向的核心。  
   证据：A；IRN（https://doi.org/10.1145/3230543.3230559）、UEC Specification（https://ultraethernet.org/specification/）

5. **推理协议从无状态 HTTP 请求走向流式事件、显式会话和多阶段执行。** SSE 解决服务器到客户端的增量推送和重连，但不解决执行恢复、事务、幂等、背压或工具副作用。  
   证据：A；WHATWG SSE（https://html.spec.whatwg.org/multipage/server-sent-events.html）

6. **MCP 是工具和上下文连接层，不是完整安全层或事务层。** MCP 使用 JSON-RPC；其 Streamable HTTP 传输可返回单个 JSON 或 SSE。断线不等于取消，取消需显式 `CancelledNotification`；`Last-Event-ID` 只支持消息重放，不代表会话状态或副作用恢复。  
   证据：A；MCP Transports（https://modelcontextprotocol.io/specification/2025-06-18/basic/transports）

7. **A2A 面向独立、可能不透明的 Agent 之间的任务协作。** A2A 1.0 的规范定义 Agent Card、Task、Message、Part、Artifact、任务状态和流式事件，并提供 JSON-RPC、gRPC、HTTP/REST 绑定；它不要求共享对方内部记忆、状态或工具。  
   证据：A；A2A Specification（https://a2a-protocol.org/latest/specification/）

8. **推理系统的中心状态变成 KV Cache，但 KV Cache 缺少跨厂商可移植线格式。** Orca 的 iteration-level scheduling、vLLM 的 PagedAttention、SGLang 的 RadixAttention、DistServe/SplitWise/Mooncake 的阶段分离共同证明这一点；但跨 vLLM、SGLang、TensorRT-LLM、Mooncake 的统一格式尚未形成。  
   证据：A；Orca（https://www.usenix.org/conference/osdi22/presentation/yu）、PagedAttention（https://arxiv.org/abs/2309.06180）、Mooncake（https://www.usenix.org/conference/fast25/presentation/qin）

9. **AI 存储从“本地性优先的文件系统”转向“SSD/NIC 带宽聚合 + locality-oblivious 访问”。** DeepSeek 3FS 面向 data preparation、dataloader、checkpoint 和 KV Cache，使用 RDMA、现代 SSD、CRAQ 和文件接口；官方报告 180 个存储节点约 6.6 TiB/s 聚合读，但这是实现方配置和自报结果。  
   证据：B；3FS（https://github.com/deepseek-ai/3fs）

10. **Checkpoint 正在从“存文件”演变为状态迁移协议。** ByteCheckpoint 和 Universal Checkpointing 的重点是 parallelism-agnostic representation、load-time resharding、跨框架/后端/并行策略迁移；但跨所有训练框架的中立标准尚未出现。  
    证据：A；ByteCheckpoint（https://www.usenix.org/conference/nsdi25/presentation/wan-borui）、Universal Checkpointing（https://www.usenix.org/conference/atc25/presentation/lian）

11. **Agent 把传统“用户—应用—数据库”的授权模型变成跨用户、Agent、工具和第三方 API 的动态委托链。** OAuth 2.0、OIDC、RFC 8707 Resource Indicators、RFC 9728 Protected Resource Metadata、SPIFFE/SPIRE、mTLS、短时令牌和外部策略引擎比长期全局 API key 更适合该模型。  
    证据：A/B；RFC 8707（https://www.rfc-editor.org/rfc/rfc8707）、RFC 9728（https://www.rfc-editor.org/rfc/rfc9728）、MCP Authorization（https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization）

12. **AIOps 在分诊和根因假设上有真实进展，但“自治自愈数据中心”仍证据不足。** 更稳妥的路径是只读分析、建议方案、低风险可回滚自动操作、高风险人工审批四级分权，并用持久工作流承载状态、审计和回滚。  
    证据：A/B；AIOpsLab（https://www.microsoft.com/en-us/research/publication/building-ai-agents-for-autonomous-clouds-challenges-and-design-principles）、Triangle（https://www.microsoft.com/en-us/research/publication/triangle-empowering-incident-triage-with-multi-agents）

---

## 1. 协议栈的新分层

```text
应用与 Agent 层
  ├─ Chat / Responses / Batch API
  ├─ Tool / Function Calling
  ├─ MCP：模型宿主连接工具与上下文
  ├─ A2A：独立 Agent 之间的任务委托
  └─ Durable Execution：持久工作流、人工审批、重试与恢复

推理运行时层
  ├─ Iteration-level scheduling
  ├─ Continuous batching
  ├─ KV Cache paging / prefix reuse
  ├─ Prefill / decode disaggregation
  └─ Inference gateway / cache-aware routing

数据与状态层
  ├─ Parallel file system / object storage
  ├─ Checkpoint / state resharding
  ├─ Vector DB / RAG freshness
  ├─ Shared log / event stream
  └─ Table snapshot / atomic metadata switch

网络与硬件层
  ├─ Ethernet / UDP / TCP / IP
  ├─ RoCEv2 / InfiniBand / RDMA
  ├─ NCCL / RCCL / collective algorithms
  ├─ NVLink / UALink / CXL
  └─ UEC AI fabric

信任与观测层
  ├─ OAuth/OIDC、mTLS、SPIFFE
  ├─ Policy engine、human approval、audit
  ├─ OpenTelemetry / GenAI semantic conventions
  └─ Model/data provenance、signing、AIBOM
```

关键判断：**这些层次不能靠单一协议全部解决。** 连接规范解决“如何说话”，身份协议解决“谁在说话”，事务/工作流解决“动作是否可恢复”，可观测性解决“过程是否可追踪”，策略引擎解决“是否允许做”。

---

## 2. 网络与互连协议：从主机互通到 collective-aware fabric

### 2.1 传统协议假设为何受挑战

传统数据中心网络通常假设：

- 大多数请求是大量相对独立的 RPC；
- 连接的目的端和持续时间较容易理解；
- TCP 提供可靠有序字节流即可；
- ECMP 将流散列到不同路径；
- 网络主要追求平均吞吐、连通性和通用公平性。

大模型训练改变了这些假设。一个分布式训练 job 通常组合 data parallel、tensor parallel、pipeline parallel、sequence parallel 和 expert parallel。任何一个 rank 停顿，都可能阻塞 collective 中的其他 rank。

### 2.2 集合通信成为协议核心

| 原语 | 典型用途 | 网络特征 |
|---|---|---|
| AllReduce | 数据并行梯度同步 | 周期性、同步边界明确；ring/tree 路径对慢节点敏感 |
| ReduceScatter | ZeRO/FSDP 等分片同步 | 归约后再分散，与 AllReduce 路径不同 |
| AllGather | 分片参数/状态聚合 | 大消息、多节点同步 |
| AllToAll | 张量重排、MoE token dispatch | 目的端多、熵高、易 incast |
| Expert dispatch/combine | MoE 专家路由 | 负载随输入内容动态变化 |

NCCL 对上给训练框架提供集合通信 API，对下使用 NVLink、shared memory、InfiniBand verbs、RoCE、socket、NetworkDirect 或插件；内部根据拓扑、消息大小、rank 数和平台能力选择 ring、tree、CollNet、NVLS、PAT 等算法。

### 2.3 Scale-up 与 scale-out 重新分层

| 层级 | 典型技术 | 适合的通信 | 主要关注点 |
|---|---|---|---|
| Package/server | HBM、PCIe、CXL、NVLink-C2C | CPU/GPU、本地内存扩展 | 带宽、延迟、一致性 |
| Rack/pod scale-up | NVLink/NVSwitch、UALink、ICI | tensor parallel、小规模 AllToAll | 高基数全互联、故障域、内存语义 |
| Multi-rack scale-out | InfiniBand、RoCE、UEC | data parallel、跨 pod collective | 路由、拥塞、容错、可扩展性 |
| 跨站点 | WAN、多集群调度 | 容灾、混合云、异地协同 | 广域延迟、安全域、一致性 |

NVLink 是强垂直整合方案；UALink 面向开放加速器 scale-up；CXL 重点是 coherency/memory semantics 和内存池化；UEC 更偏 AI/HPC scale-out fabric。它们不能被简单描述为互相替代。

### 2.4 前沿解决办法

1. **Rail-aligned topology**：GPU 的多 NIC 与网络 rail 对齐，降低跨 rail 流量和拓扑不对称。
2. **Adaptive routing / packet spraying**：把包分散到多条路径，降低 hash 极化和局部热点；代价是乱序、重复和接收状态复杂度。
3. **Selective retransmission**：只重传丢失片段，减少粗粒度超时造成的恢复延迟。
4. **In-network reduction**：把归约、广播、屏障等下沉到交换机；SHARP/NVLS 在 NVIDIA 垂直生态中有产品证据，UEC INC 仍属新规范阶段。
5. **Collective-aware scheduling**：把任务放置、网络拓扑、队列、拥塞和故障恢复联合优化。
6. **GPUDirect RDMA**：NIC 直接访问 GPU 注册内存，减少 CPU copy 和内核切换。
7. **SDN/OCS 故障绕行**：通过软件定义网络或光交换绕开故障机器、链路和网络设备。

### 2.5 成熟度判断

- **生产验证**：InfiniBand verbs/RDMA、RoCEv2、NCCL 基础算法、NVLink/NVSwitch、GPUDirect RDMA、rail-aligned topology、AWS EFA/SRD、Google ICI/OCS。
- **研究或早期标准**：UEC 完整多厂商 AI fabric、UEC In-Network Collectives、UALink 开放生态、CXL AI 内存池、新型可编程拥塞控制。
- **炒作/证据不足**：“以太网立即全面替代 InfiniBand/NVLink/ICI”、“UEC 规范发布等于大规模生产成熟”、“只靠 PFC 可解决万卡 RDMA 问题”。

---

## 3. 推理与 Agent 协议：从无状态 HTTP 到长会话、工具调用和任务委托

### 3.1 推理 API 的变化

传统 HTTP API 的基本模式是：

```text
HTTP Request → complete JSON Response
```

LLM 推理更常见的模式是：

```text
HTTP/JSON Request → SSE token events → final completion event
```

SSE 提供：

- 服务器到客户端的单向事件流；
- 浏览器自动重连；
- `Last-Event-ID`；
- 事件类型和数据载荷。

SSE 不定义：

- 请求是否已在服务端执行完成；
- 工具调用是否已经产生副作用；
- 断线后如何恢复会话；
- 多次提交如何去重；
- 慢客户端如何形成应用层背压；
- 多个事件是否构成事务。

### 3.2 MCP：工具与上下文连接协议

MCP 的角色是：

```text
Host / Model application
  └─ MCP Client
       └─ MCP Server
            ├─ Resources
            ├─ Tools
            ├─ Prompts
            └─ external systems
```

MCP 2025-06-18 传输规范定义 stdio 和 Streamable HTTP：

- stdio：客户端启动本地子进程，通过 stdin/stdout 交换 JSON-RPC；
- Streamable HTTP：单个 HTTP endpoint 支持 POST/GET；POST 请求可返回 `application/json` 或 `text/event-stream`；
- 有状态 session 可通过 `Mcp-Session-Id` 标识；
- 断线不应被解释为取消；取消应显式发送 `CancelledNotification`；
- `Last-Event-ID` 可用于 SSE 消息重放，但不是执行状态恢复。

MCP 的边界：

- 不提供事务性 tool call；
- 不统一 KV Cache 格式；
- 不自动解决跨 Agent 幂等；
- 不保证工具描述和返回内容可信；
- 不把应用从授权、审计、隔离和策略控制中解放出来。

### 3.3 A2A：独立 Agent 的任务协议

A2A 的角色是：

```text
Client Agent → Remote Agent
                 ├─ Agent Card
                 ├─ Task
                 ├─ Message / Part
                 └─ Artifact
```

A2A 1.0 的核心抽象包括：

- **Agent Card**：声明身份、技能、端点、安全方案和能力；
- **Task**：有状态的工作单元；
- **Message / Part**：消息及文本、文件、结构化内容；
- **Artifact**：任务产物；
- **Streaming events**：任务状态和产物的增量更新；
- **Push notification**：异步任务更新。

A2A 支持 JSON-RPC、gRPC 和 HTTP/REST 绑定。其规范明确 Send Message 可以用 `messageId` 做重复检测，但不是强制完整幂等；Cancel Task 要求幂等。因此跨 Agent 的副作用去重仍需要应用层或持久工作流补足。

### 3.4 推理运行时对协议的反向影响

| 系统 | 关键变化 | 协议含义 |
|---|---|---|
| Orca | iteration-level scheduling、selective batching | 请求不再是唯一调度粒度 |
| vLLM/PagedAttention | KV Cache 分页 | 显存状态像虚拟内存一样被管理 |
| SGLang/RadixAttention | 公共前缀 KV 复用 | 路由和上下文组织影响命中 |
| DistServe | prefill/decode 分离 | 阶段间需要 KV 传输和资源协同 |
| SplitWise | prompt/generation 分到不同机器 | 不同硬件匹配不同阶段 |
| Mooncake | KVCache-centric 分离式架构 | KV Cache 成为跨资源池的一等对象 |
| Llumnix | 请求级重调度和迁移 | 推理负载均衡从副本级进入请求级 |

### 3.5 推理网关

Kubernetes Gateway API Inference Extension 通过 Envoy External Processing Protocol 扩展支持推理负载的网关，核心资源包括 `InferencePool` 和 `InferencePoolImport`。它主要标准化路由扩展点，使网关能够进行推理负载感知路由，而不是定义统一模型格式、KV 传输协议或完整 Agent 事务。

### 3.6 持久执行的作用

Agent 长任务常包含：

- 多次模型调用；
- 工具和第三方 API 副作用；
- 人工审批；
- 条件等待；
- 失败重试；
- 多 Agent 协作；
- 进程崩溃后的恢复。

Temporal 一类 durable execution 的基本模式是：

```text
确定性 Workflow：可 replay
非确定性 Activity：LLM、工具、外部 API
Activity 结果：写入 workflow history
```

这能解决协调逻辑的持久化、等待、恢复和审计，但不自动让第三方 API 变成 exactly-once，也不自动撤销已发生的外部副作用。

### 3.7 成熟度判断

- **生产验证**：HTTP/JSON、SSE、JSON-RPC、OpenAPI、WebRTC、Batch API、Temporal 风格持久执行。
- **快速演进**：MCP、A2A、Gateway API Inference Extension、vLLM/SGLang 生态、KV transfer connectors。
- **证据不足**：统一 Agent 协议、跨厂商会话迁移、跨引擎可恢复执行、事务性 tool call、多 Agent 自动协商标准。

---

## 4. 数据、存储、复制、共识与事务协议

### 4.1 AI 数据路径的新负载

| 负载 | 访问模式 | 主要风险 |
|---|---|---|
| Data preparation | sort/shuffle、混合读写、大量中间结果 | 聚合带宽和阶段屏障 |
| Dataloader | 大量 worker 随机、小到中块读取 | GPU 等数据，页缓存收益低 |
| Checkpoint save | 周期性 TB 级突发大写 | 与训练通信和前台 I/O 冲突 |
| Checkpoint load | 全量读取并 reshard | RTO 高，拓扑变化导致转换成本 |
| KV Cache | 随机读写、迁移、驱逐、复用 | 缓存热点、GC、跨阶段传输 |
| Vector index | 构建、更新、删除、压缩、检索 | 新鲜度、召回率和重建成本难兼顾 |

### 4.2 3FS：AI workload 驱动的并行文件系统

3FS 面向四类 AI 负载：data preparation、dataloaders、checkpointing、KVCache。其公开材料显示：

- disaggregated architecture；
- locality-oblivious 访问；
- CRAQ 强一致复制；
- 文件接口；
- 元数据服务无状态，底层由事务 KV 支撑；
- RDMA 和现代 NVMe SSD。

3FS 的重要性不是单个 CRAQ、RDMA 或文件接口，而是把已有机制围绕 AI 全数据路径做端到端整合。限制是：公开性能来自 DeepSeek 指定硬件、网络和 workload，缺少足够的第三方独立复测。

### 4.3 Checkpoint：状态迁移协议

ByteCheckpoint 的关键设计是：

- parallelism-agnostic checkpoint representation；
- load-time resharding；
- 多框架和多存储后端支持；
- save/load 工作流和监控工具。

Universal Checkpointing 也把 checkpoint 与 tensor/pipeline/sequence/ZeRO distributed training 等并行策略解耦，使训练能在不同硬件和并行配置之间重配置。

这说明 checkpoint 的语义已经从“故障后恢复”扩展到：

- 训练阶段转换；
- 并行策略变化；
- 资源规模变化；
- 评估和调试；
- 跨集群迁移；
- 任务抢占与恢复。

### 4.4 派生状态改变事务边界

AI 系统中存在大量不同于权威业务数据的状态：

- KV Cache；
- embedding；
- vector index；
- intermediate activations；
- materialized retrieval result。

这些状态通常可以由源数据、模型权重和输入重新计算。丢失主要影响性能，不一定影响正确性。因此更自然的发布模式是：

```text
后台构建新一代状态 → 校验质量/召回率/延迟 → 原子切换 metadata pointer → 保留旧版本回滚
```

这与 Iceberg 的 snapshot、manifest 和 metadata pointer 思想相似：不是对每条记录做长事务，而是用不可变数据对象和原子元数据发布形成版本边界。

### 4.5 RAG 与向量数据库

RAG 的完整链路包括：

1. source DB 变更捕获；
2. 文档解析与切块；
3. embedding；
4. 写入向量集合；
5. 索引构建或更新；
6. hybrid search、过滤和 rerank；
7. source、文档、向量、权限之间的新鲜度对齐。

Milvus、Weaviate、Pinecone 等系统提供不同程度的存算分离、分级一致性、多租户、后台修复或对象存储卸载。但不能把这些机制简单等同于关系数据库中的多对象 ACID。

Debezium 官方文档说明其默认提供 at-least-once；在 Kafka Connect 中可利用 Kafka transactions 获得 exactly-once 支持，但端到端正确性还取决于 sink 幂等、offset 提交、schema 演进、索引提交和失败恢复。

### 4.6 协调协议前沿

| 路线 | 代表系统 | 核心思想 | 成熟度 |
|---|---|---|---|
| 同步时钟预排序 | Tiga | 事务获得未来时间戳，多数路径压缩到 1 个广域往返 | 研究/开源原型 |
| 执行与复制解耦 | Mako | 本地投机执行，后台地理复制 | 研究 |
| CXL 内存原子 | Tigon | 用 CXL pod 内共享内存和原子操作协调跨主机访问 | 研究 |
| 共享日志延迟排序 | LazyLog | append 时只保证持久化，读前绑定全局顺序 | 研究 |
| 链式强一致复制 | CRAQ/3FS | 写沿链传播，读由链上节点分摊 | 3FS 等生产采用 |

这些方案不是“消除一致性成本”，而是把排序、复制、冲突检测从关键路径上移走、延后或与硬件合并。

### 4.7 成熟度判断

- **生产验证**：对象存储/S3 API、HDFS/Ceph/Lustre、GPUDirect Storage、Parquet/Arrow、Iceberg、PyTorch DCP、vLLM、Milvus/Weaviate/pgvector、Debezium/Kafka、Raft/Paxos、Saga/TCC、Safetensors。
- **研究或早期采用**：3FS 的第三方泛化证据、ByteCheckpoint/UCP 跨框架统一、checkpoint-free recovery、Tiga、Mako、Tigon、LazyLog、CXL 内存池。
- **炒作/证据不足**：“AI 数据平台无条件替代数据湖/消息/文件系统/向量库”、“CDC 端到端 exactly-once 开箱即用”、“向量数据库已普遍具备 SQL 级 ACID”、“零成本强一致”。

---

## 5. 安全、身份、治理、可观测性与运维协议

### 5.1 授权模型从静态凭证变为动态委托链

Agent 场景必须回答：

- 当前动作代表哪个用户；
- Agent 的工作负载身份是什么；
- 能访问哪个工具或资源服务器；
- 允许读还是写；
- 令牌是否能被再次委托；
- 是否需要人工审批；
- 动作如何审计和撤销。

推荐基线：

1. 用户委托使用 OAuth 2.0/OIDC + PKCE；
2. 每个资源服务器使用明确 audience/resource；
3. 使用 RFC 9728 Protected Resource Metadata；
4. token 短时有效并可撤销；
5. Agent 工作负载使用 SPIFFE/SPIRE 和 mTLS；
6. scope 细化到工具、资源和动作；
7. 高风险 API 使用 mTLS client credentials、DPoP、PAR 或 FAPI 2.0；
8. 权限判断由外部策略引擎和资源服务器强制执行。

RFC 8707 的 Resource Indicators 允许客户端显式声明目标资源，帮助授权服务器限制令牌的 audience；它能降低 token redirect 和部分 confused deputy 风险，但不能单独解决所有多 Agent 委托问题。

### 5.2 Prompt injection 使“数据/命令边界”成为协议问题

Indirect prompt injection 中，外部内容可能被模型解释成指令。下列内容都可能成为攻击载体：

- RAG 检索结果；
- 网页、邮件、工单；
- issue、PR、代码评论；
- 工具返回值；
- 长期 memory；
- Agent Card 或工具描述；
- 其他 Agent 的响应。

可靠控制包括：

- 不可信内容显式标记为 data；
- 工具 allowlist；
- 读写工具分离；
- 网络出口和 DLP；
- tool-level quota；
- 多步任务预算；
- 高风险操作人工批准；
- policy decision 和 audit log；
- 攻击回放和 red teaming。

关键原则：**授权判断必须在模型外部。** 不能把“模型会遵守 system prompt”当作安全边界。

### 5.3 人类批准的边界

审批界面必须展示：

- 实际工具名；
- 完整参数；
- 目标资源；
- 读写属性；
- 代表的用户；
- 外发目的地；
- 与原始目标的关系；
- 是否引用了不可信内容；
- 可回滚性和副作用。

仅提供“是否允许调用工具”的确认框不足以构成充分控制。

### 5.4 机密计算

CPU TEE 和机密 VM 已较成熟；GPU confidential computing 仍在快速演进。TEE 能降低宿主、运维和内存窥探风险，但不解决：

- prompt injection；
- 越权工具调用；
- RAG 污染；
- Agent 错误决策；
- 应用层授权；
- 跨 CPU/GPU/NIC 的完整端到端证明。

采用时应要求明确 TCB 范围、attestation 内容、跨设备数据保护、调试/profiling 限制、KV Cache 保护方式和侧信道缓解。

### 5.5 可观测性协议

Agent 最小观测模型：

```text
Agent run
├─ plan / workflow
├─ LLM inference
│  ├─ provider/model/version
│  ├─ prompt version
│  ├─ token usage / cost
│  └─ status / error
├─ retrieval / embedding
│  ├─ source/document/chunk id
│  ├─ score
│  └─ retrieval parameters
├─ memory read/write
└─ tool execution
   ├─ tool name/type/id
   ├─ target resource
   ├─ arguments hash/reference
   ├─ result status
   └─ approval metadata
```

OpenTelemetry/OTLP 的核心 traces、metrics、logs 管道成熟，但 GenAI semantic conventions 在 2026 年仍应视为快速演进。生产系统应固定 instrumentation 版本，在 Collector 层统一字段映射。

Prompt、工具参数、RAG 文本可能包含 PII、商业秘密或凭证，默认应记录长度、哈希、引用 ID 或脱敏摘要，而不是完整原文。

### 5.6 AIOps 与受控修复

建议把 Agent 权限分成四级：

| 级别 | 能力 | 控制要求 |
|---|---|---|
| L1 | 只读分析：日志、指标、trace、变更记录 | 访问审计 |
| L2 | 建议诊断、runbook、补丁或配置 | 人工阅读和验证 |
| L3 | 低风险可回滚动作：重启、扩容、隔离实例 | 幂等、灰度、自动回滚 |
| L4 | 高风险动作：删除、权限变更、数据库写入、生产发布 | 双人审批、变更单、完整审计 |

所有写操作应具备 dry run、幂等键、执行计划、实际 diff、回滚方案、完成后验证和事件关联。

### 5.7 成熟度判断

- **生产验证**：OAuth/OIDC、RFC 8707、RFC 9728、mTLS、SPIFFE/SPIRE、OPA、API gateway、OTLP 核心管道、SLSA/Sigstore、CPU TEE、变更审批/灰度/回滚。
- **快速演进**：MCP/A2A 授权、Agent 专属策略、GenAI OTel semantic conventions、模型签名、AIBOM、GPU CC、多 Agent 分诊。
- **炒作/证据不足**：“接入 MCP/A2A 即有安全层”、“LLM 自己能判断越权”、“确认框能解决工具滥用”、“TEE 能端到端保护全部 AI 链路”、“无人自治数据中心已成熟”。

---

## 6. 推荐参考架构

### 6.1 分层而不是单协议统一

```text
User / Client
  │ OAuth/OIDC + PKCE
  ▼
AI Gateway / Agent Orchestrator
  ├─ Policy Engine：外部授权
  ├─ Inference Router：模型、LoRA、prefix/KV-aware
  ├─ Durable Workflow：状态、重试、审批、恢复
  ├─ MCP Clients：工具和资源
  └─ A2A Client：远程 Agent
       │
       ├─ Prefill Pool ── KV transfer ── Decode Pool
       ├─ Vector DB / Search / Document Store
       ├─ Parallel File / Object Store / Checkpoint Store
       └─ Observability + Audit + Cost/Quota
```

### 6.2 协议设计清单

| 问题 | 推荐机制 |
|---|---|
| 用户身份 | OAuth 2.0/OIDC、PKCE、短时 token |
| 服务身份 | SPIFFE/SPIRE、mTLS |
| 资源绑定 | RFC 8707 Resource Indicators |
| 元数据发现 | RFC 9728 Protected Resource Metadata |
| 工具连接 | MCP |
| Agent 间协作 | A2A |
| 长任务恢复 | Durable execution / workflow history |
| 推理路由 | Inference gateway、InferencePool、cache-aware routing |
| KV Cache | 引擎内分页、前缀复用、显式 connector；不假设跨引擎兼容 |
| Checkpoint | 并行策略无关表示、load-time resharding |
| RAG 新鲜度 | CDC + 消息系统 + 版本化索引 + 原子切换 |
| 高风险动作 | 人工审批、策略引擎、审计、灰度、回滚 |
| 观测 | OTLP + GenAI/Agent 属性 + prompt/tool 内容脱敏 |
| 供应链 | 模型/数据/插件 provenance、签名、AIBOM |

### 6.3 不应采用的假设

- 不应假设 SSE 重连等于任务恢复；
- 不应假设 MCP 提供事务性 tool call；
- 不应假设 A2A 的消息投递天然 exactly-once；
- 不应假设 KV Cache 可跨引擎无缝迁移；
- 不应假设 CDC + Kafka 自动保证端到端 exactly-once；
- 不应让模型输出本身成为授权判断；
- 不应把长期全局 API key 分发给 Agent 工具；
- 不应把 TEE 视为 prompt injection 和越权动作的解决方案；
- 不应把厂商特定 benchmark 当作跨场景通用事实。

---

## 7. 证据缺口清单

1. **UEC 多厂商互操作和长期生产数据不足。** 规范存在不等于网卡、交换机、GPU 平台、拥塞控制和运维工具已经大规模成熟。
2. **KV Cache 缺少跨厂商线格式/ABI。** 论文证明 KV Cache 是核心资源，但没有证据表明 vLLM、SGLang、TensorRT-LLM、Mooncake 等已统一可移植格式。
3. **MCP 不提供事务和执行迁移标准。** 消息重放不能证明工具副作用没有发生，也没有统一提交/回滚/快照协议。
4. **A2A 的跨组织生产样本不足。** 规范完整，但独立生产流量、故障模式、委托撤销和权限降级数据仍少。
5. **3FS 缺少第三方公平复测。** 6.6 TiB/s 等数字依赖 DeepSeek 指定硬件和测试配置。
6. **Checkpoint 跨框架无损互操作仍缺标准。** Safetensors 只覆盖 tensor，不覆盖 optimizer、scheduler、RNG、并行拓扑和完整运行时状态。
7. **RAG 端到端事务语义不足。** source DB、chunk、embedding version、vector index、ACL 和 CDC offset 的原子切换仍缺少跨产品标准。
8. **Agent 专属策略模型不成熟。** OPA 成熟，但如何为 memory、plan、多 Agent 委托和工具预算建立统一策略仍在早期。
9. **GPU TEE 缺少独立端到端测量。** KV Cache、Agent state、跨 CPU/GPU/NIC 路径和侧信道仍需更多证据。
10. **自主修复缺少跨组织生产证据。** 现有强证据多来自特定云厂商、受控环境或局部分诊场景。
11. **GenAI 语义约定仍在变化。** 生产 instrumentation 需要版本固定和 Collector 层兼容。
12. **审批 UI 的安全有效性缺少系统研究。** 用户是否真正理解工具参数、副作用和外发目的地仍缺少可靠数据。

---

## 8. 参考来源索引

### 标准与规范

1. RFC 8707: Resource Indicators for OAuth 2.0 — https://www.rfc-editor.org/rfc/rfc8707
2. RFC 9728: Protected Resource Metadata for OAuth 2.0 — https://www.rfc-editor.org/rfc/rfc9728
3. JSON-RPC 2.0 Specification — https://www.jsonrpc.org/specification
4. WHATWG Server-sent Events — https://html.spec.whatwg.org/multipage/server-sent-events.html
5. OpenAPI Specification 3.1.0 — https://spec.openapis.org/oas/v3.1.0
6. WebRTC W3C Recommendation — https://www.w3.org/TR/webrtc/
7. Apache Iceberg Specification — https://iceberg.apache.org/spec/
8. Ultra Ethernet Consortium Specification — https://ultraethernet.org/specification/
9. UALink Whitepaper — https://ualinkconsortium.org/whitepaper/

### MCP / A2A / Inference Gateway

10. MCP Architecture — https://modelcontextprotocol.io/specification/2025-03-26/architecture
11. MCP Transports — https://modelcontextprotocol.io/specification/2025-06-18/basic/transports
12. MCP Authorization — https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization
13. A2A Specification — https://a2a-protocol.org/latest/specification/
14. Gateway API Inference Extension API Overview — https://gateway-api-inference-extension.sigs.k8s.io/concepts/api-overview/
15. Envoy External Processing Filter — https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/ext_proc_filter

### 推理系统论文

16. Orca: A Distributed Serving System for Transformer-Based Generative Models — https://www.usenix.org/conference/osdi22/presentation/yu
17. PagedAttention / vLLM — https://arxiv.org/abs/2309.06180
18. SGLang: Efficient Execution of Structured Language Model Programs — https://proceedings.neurips.cc/paper_files/paper/2024/hash/724be4472168f31ba1c9ac630f15dec8-Abstract-Conference.html
19. DistServe: Disaggregating Prefill and Decoding — https://arxiv.org/abs/2401.09670
20. SplitWise: Efficient Generative LLM Inference Using Phase Splitting — https://arxiv.org/abs/2311.18677
21. Mooncake: A KVCache-centric Architecture — https://www.usenix.org/conference/fast25/presentation/qin
22. Llumnix: Dynamic Scheduling for LLM Serving — https://arxiv.org/abs/2406.03243

### 网络、存储与训练恢复

23. NCCL Documentation — https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/index.html
24. NVIDIA SHARP Documentation — https://docs.nvidia.com/networking/display/SHARPv314
25. RoCEv2 Whitepaper — https://network.nvidia.com/pdf/whitepapers/RoCEv2.pdf
26. DCQCN — https://dl.acm.org/doi/10.1145/2829988.2787504
27. IRN — https://doi.org/10.1145/3230543.3230559
28. MegaScale — https://www.usenix.org/system/files/nsdi24-jiang-ziheng.pdf
29. Meta Distributed AI Training over Ethernet — https://doi.org/10.1145/3651890.3672233
30. Alibaba HPN — https://doi.org/10.1145/3651890.3672259
31. AWS SRD — https://www.amazon.science/publications/a-cloud-optimized-transport-protocol-for-elastic-and-scalable-hpc
32. DeepSeek 3FS — https://github.com/deepseek-ai/3fs
33. ByteCheckpoint — https://www.usenix.org/conference/nsdi25/presentation/wan-borui
34. Universal Checkpointing — https://www.usenix.org/conference/atc25/presentation/lian
35. GEMINI In-Memory Checkpoints — https://dl.acm.org/doi/10.1145/3600006.3613145
36. Bamboo — https://www.usenix.org/conference/nsdi23/presentation/thorpe
37. Oobleck — https://arxiv.org/abs/2309.08125

### 数据、RAG 与治理

38. Milvus Consistency — https://milvus.io/docs/consistency.md
39. Weaviate Consistency — https://docs.weaviate.io/weaviate/concepts/replication-architecture/consistency
40. Debezium Exactly Once — https://debezium.io/documentation/reference/stable/configuration/eos.html
41. OpenLineage Specification — https://github.com/OpenLineage/OpenLineage/blob/main/spec/OpenLineage.md
42. Safetensors — https://github.com/huggingface/safetensors
43. SPIFFE/SPIRE Concepts — https://spiffe.io/docs/latest/spire-about/spire-concepts
44. OpenTelemetry Specification Status — https://opentelemetry.io/docs/specs/status/
45. OpenTelemetry GenAI Spans — https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-spans.md

### 安全与运维

46. OWASP Agentic AI Threats and Mitigations — https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/
47. Indirect Prompt Injection — https://arxiv.org/abs/2302.12173
48. PoisonedRAG — https://arxiv.org/abs/2402.07867
49. Invariant Labs GitHub MCP Analysis — https://invariantlabs.ai/blog/mcp-github-vulnerability
50. AIOpsLab — https://www.microsoft.com/en-us/research/publication/building-ai-agents-for-autonomous-clouds-challenges-and-design-principles
51. Triangle: Incident Triage with Multi-agents — https://www.microsoft.com/en-us/research/publication/triangle-empowering-incident-triage-with-multi-agents
52. RCACopilot — https://www.microsoft.com/en-us/research/publication/automatic-root-cause-analysis-via-large-language-models-for-cloud-incidents
53. NIST AI 600-1 — https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf
54. EU AI Act — https://eur-lex.europa.eu/legal-content/EN/ALL/?uri=CELEX%3A32024R1689
55. SLSA Specification — https://slsa.dev/spec/v1.0/levels

---

## 9. 最终判断

AI 对软件系统协议的核心影响不是“把旧协议全部换掉”，而是：

1. **通信主体变化**：从独立 RPC 变成 collective、KV Cache 迁移和动态 Agent 消息；
2. **状态位置变化**：大量状态进入 GPU 显存、分离式缓存、并行文件系统和版本化索引；
3. **时间边界变化**：请求持续数秒到数天，必须支持流式、取消、审批、恢复和重放；
4. **信任边界变化**：模型、工具、记忆、RAG 内容和其他 Agent 都可能影响动作；
5. **协调成本变化**：一次停顿会被大量 GPU idle 和 WAN wait 放大；
6. **治理要求变化**：协议必须与身份、策略、审计、provenance、成本和观测联动。

因此，2024–2026 年最可靠的技术路线不是等待一个万能协议，而是采用 **collective-aware fabric + KV-aware inference runtime + MCP/A2A 分层互操作 + durable execution + 外部授权/审计 + 全链路 GenAI observability** 的组合架构。
