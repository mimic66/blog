# AI 大模型对分布式系统、网络、存储与调度的反向塑造（2022–2026）调研笔记

> 调研日期：2026-11-10
> 时间范围：重点覆盖 2022–2026；对 Megatron、ZeRO、DCQCN、DiskANN 等必要前置工作做历史回溯。
> 证据分级：
> - **A 级【已核实】**：顶会/期刊论文、正式标准、官方文档、官方仓库，可直接支撑结论。
> - **B 级【厂商/运营口径】**：官方工程博客、厂商技术白皮书、厂商赞助调查；可说明实践方向与样本，但需保留厂商立场。
> - **C 级【待核实】**：二手汇总、媒体、演讲材料；只能作为线索，不单独支撑强结论。
> - **证据缺口**：检索后未找到合格一手来源，不作为事实结论。

---

## 0. 执行摘要：10 个最重要结论

1. **万卡训练把“故障”从异常事件变成调度内常态**。Meta 两个共 24,000 张 A100 的研究集群、11 个月、400 万个 job、超过 150M A100 GPU 小时的数据表明：硬件归因故障只影响不到 1% 的 job，却影响约 18.7% GPU runtime；1,024 GPU job 的 MTTF 为 7.9 小时，16,384 GPU 的模型外推 MTTF 为 1.8 小时。
2. **训练系统的核心并行抽象正在反向决定网络拓扑和硬件互连**。NVLink/NVSwitch、InfiniBand、rail-aligned Clos、3D Torus、UALink、UEC 不再只是“给 GPU 配网”，而是围绕 AllReduce、AllGather、ReduceScatter、AllToAll 和 MoE expert dispatch 的通信模式重新划分 scale-up / scale-out 边界。
3. **LLM serving 从无状态 Web 服务变成“显存内状态管理系统”**。Orca 将调度粒度从 request 改为 iteration；vLLM/PagedAttention 将 KV Cache 分页；SGLang/RadixAttention 复用前缀；这些系统都在解决同一个问题：请求长度、生成长度和 KV Cache 生命周期高度动态，传统请求级 batch 会让 GPU 空转或因碎片浪费显存。
4. **推理架构正在从“单集群一个模型”走向阶段分离与 KV Cache 中心化**。DistServe、SplitWise、Mooncake 将 prefill/prompt 与 decode/token generation 拆分到不同资源池；Mooncake 进一步把 KV Cache 作为跨 prefill/decode 的一等对象，使用 CPU、DRAM、SSD、NIC 构建 GPU 外的分离式缓存层。
5. **AI 存储的关键矛盾不只是容量，而是高并发随机读、突发 checkpoint 写和 KV Cache 搬运**。DeepSeek 3FS 使用现代 SSD 与 RDMA，目标负载明确覆盖 data preparation、dataloader、checkpointing 和 KV Cache；官方仓库称 180 个存储节点在训练背景流量下达到约 6.6 TiB/s 聚合读。
6. **多租户 GPU 集群不能只靠“占满 GPU”衡量效率**。Crux 在生产 trace 中发现 job 间通信竞争会显著降低 GPU computation utilization；其测试床提升 8.3%–14.8%，生产 trace 仿真最高提升 23%。这说明算力调度、网络调度和任务放置必须联合优化。
7. **Gang scheduling、拓扑感知、抢占恢复、队列配额正在进入 AI 基础设施主干**。Slurm 长期承载 HPC/AI 训练；Volcano、Kueue、Kubeflow 把这些能力带入 Kubernetes；Kubernetes 自身在 2025 年 KEP-4671 中推进 Workload/PodGroup 与原生 gang scheduling。
8. **向量检索和 RAG 正在推动数据库从“存记录”扩展为“embedding、索引、过滤、权限和新鲜度”一体化系统**。Milvus、Pinecone、Weaviate 都采用不同程度的存算分离、分片、可调一致性或多租户设计；但 CDC → re-chunk → re-embed → atomic reindex 的生产收益仍缺少跨厂商、可比较的一手数据。
9. **GPU 多租户共享没有统一赢家：MIG、MPS、时间复用各自在隔离性、粒度和利用率之间取舍**。MIG 提供最强硬件隔离但配置粒度较粗；MPS 可共享 SM 但内存系统干扰更强；Google GASS 这类时间复用与按需分页可提升聚合吞吐，但需要实时监控和自适应调度避免性能异常。
10. **LLM agent 和 AIOps 已在局部生产环节落地，但“自治自愈数据中心”仍是早期叙事**。Microsoft Triangle 在 Azure 的局部分级场景中报告 agent 约 90% 准确率、单团队 TTM 降低 38%；学术论文还报告最高 97% 分级准确率和 TTE 降低 91%。但这些数字受组织、工具接入、事件分布和人工审批边界影响，不能外推为通用无人运维。

---

## 1. 研究问题与判断框架

### 1.1 要回答的问题

- 大模型训练和推理负载带来了哪些传统分布式系统中不突出的瓶颈？
- 网络、存储、调度、操作系统、加速器接口如何围绕这些负载重构？
- 哪些变化已经在生产系统中验证，哪些仍停留在论文、演示或厂商宣传？
- 训练侧和推理侧对底层系统的需求有何不同？
- AI for Systems / learned systems 在 2022–2026 年是否进入生产主干？

### 1.2 反向塑造的六个维度

| 维度 | 传统云服务常见假设 | 大模型负载带来的变化 |
|---|---|---|
| 计算单元 | 单节点或少量副本可独立推进 | 一个训练 job 需要数百到数万加速器同时存活；单卡故障可拖垮整个 gang |
| 通信模式 | 南北向 RPC、短连接、shuffle 相对阶段性 | AllReduce/AllGather/ReduceScatter 高频出现；MoE AllToAll 与 expert 负载均衡成为核心路径 |
| 服务状态 | 在线服务倾向无状态，状态放入数据库/缓存 | LLM 请求在 GPU 显存中维护持续增长的 KV Cache；serving scheduler 需管理显存块和前缀复用 |
| 存储访问 | 顺序日志、块存储、对象存储分层 | 高并发随机读、训练数据一次性读取、TB 级突发 checkpoint、KV Cache 跨节点搬运 |
| 调度目标 | CPU/内存利用率、副本弹性、微服务可用性 | 拓扑感知、gang placement、碎片治理、抢占恢复、通信竞争避免、SLO 与吞吐联合优化 |
| 运维自动化 | 规则告警、仪表盘、人工 runbook | LLM agent 接日志/指标/诊断工具，尝试跨事件生命周期执行分诊、根因分析和受控修复 |

---

## 2. 训练侧：万卡集群、并行策略与可靠性

### 2.1 关键证据

| 编号 | 结论 | 关键数字/机制 | 来源 | 年份 | 等级 |
|---|---|---|---|---|---|
| TR1 | 万卡级别生产训练已能实现较高 MFU，但必须依赖系统级协同优化 | 12,288 GPUs 训练 175B 模型；MFU 55.2%；较 Megatron-LM 提升 1.34× | MegaScale, NSDI 2024，https://www.usenix.org/system/files/nsdi24-jiang-ziheng.pdf | 2024 | A |
| TR2 | 大集群中的故障影响具有明显“放大效应” | 两个 24,000 A100 集群；11 个月；400 万 jobs；>150M A100 GPU 小时；硬件归因故障影响 0.2% jobs、18.7% runtime | Meta, HPCA 2025 / arXiv:2410.21680，https://arxiv.org/abs/2410.21680 | 2024/2025 | A |
| TR3 | Job MTTF 随 GPU 数近似反比例下降 | 8 GPU：47.7 天；1,024 GPU：7.9 小时；16,384 GPU：1.8 小时（外推）；131,072 GPU：0.23 小时（外推） | Meta, arXiv:2410.21680 | 2024/2025 | A |
| TR4 | MoE 模型可以在较低激活参数下实现高训练效率，但稳定性是系统工程结果 | 671B 总参数、37B 激活；总训练 2.788M H800 GPU 小时；无不可恢复 loss spike、无 rollback | DeepSeek-V3 Technical Report，https://arxiv.org/abs/2412.19437 | 2024/2025 | A |
| TR5 | 训练 checkpoint 不只是故障恢复，还服务于评估、阶段转换、并行策略变化和资源变更 | Parallelism-agnostic checkpoint representation；load-time resharding；多框架/多后端支持 | ByteCheckpoint，https://arxiv.org/abs/2407.20143 | 2024/2025 | A |
| TR6 | Tensor parallelism 是大模型跨 GPU 切分的基础能力 | 512 GPUs 训练 8.3B 参数；持续 15.1 PetaFLOPs；扩展效率 76% | Megatron-LM，https://arxiv.org/abs/1909.08053 | 2019 | A |
| TR7 | ZeRO 通过分区 optimizer states、gradients、parameters 降低显存冗余 | ZeRO-1/2/3；64 GPU 时最高 64× 显存降低，通信量约增加 50% | Microsoft ZeRO/DeepSpeed，https://www.microsoft.com/en-us/research/blog/zero-deepspeed-new-system-optimizations-enable-training-models-with-over-100-billion-parameters | 2020 | B |
| TR8 | In-memory checkpoint 可把恢复状态先写入主机 CPU 内存，规避远端存储带宽瓶颈 | 模型论文同名系统“Gemini”，非 Google Gemini 模型 | Gemini, SOSP 2023，https://dl.acm.org/doi/10.1145/3600006.3613145 | 2023 | A |
| TR9 | Checkpoint-free recovery 已进入工业研究系统验证，但尚不能视为生产主干 | 4,800 devices；150 秒恢复；秒级检测；单步恢复；规模近似无关 | FlashRecovery，https://arxiv.org/abs/2509.03047 | 2025 | A（论文/研究系统） |

### 2.2 机制解释

#### 2.2.1 为什么单卡故障会放大成万卡停顿

大模型训练通常采用 data parallel、tensor parallel、pipeline parallel、expert parallel 的组合。任一 rank 失败后：

1. 参与同一个 collective 的其他 rank 无法继续当前 step。
2. Pipeline parallel 的上下游 stage 可能被阻塞。
3. Gang scheduler 需要重新分配或替换节点。
4. 应用从上一个 checkpoint 恢复，重新执行 checkpoint 之后的计算。
5. 通信组、并行拓扑和数据加载器需要重新初始化。

因此故障成本不是“坏一张卡修一张卡”，而是 **故障检测时间 + 节点替换/重排队时间 + 重启初始化时间 + checkpoint 重放时间 + checkpoint 自身开销**。Meta 论文将 ETTR 作为衡量 productive runtime 占 wallclock time 的核心指标，正是为了把这些成本统一建模。

#### 2.2.2 为什么并行策略会反向决定网络

- Tensor parallelism要求层内算子频繁交换，适合放在 NVLink/NVSwitch、ICI、UALink 等高带宽 scale-up 域。
- Data parallelism周期性同步梯度或 partitioned states，对 AllReduce/ReduceScatter/AllGather 延迟和带宽敏感。
- Pipeline parallelism按 stage 切分，通信颗粒较大但对 bubble 和调度敏感。
- Expert parallelism在 MoE 中产生动态 AllToAll，expert 负载不均会让网络拥塞与计算空闲同时出现。

所以网络设计从“主机之间可达”转为“让 collective 在特定层级上以最小代价执行”：机架内全互联、rail-optimized topology、adaptive routing、in-network reduction、packet spraying、selective retransmission 都是该趋势的体现。

### 2.3 训练侧成熟度判断

| 能力 | 判断 | 依据 |
|---|---|---|
| 千卡级生产训练 | 已生产验证 | MegaScale、Meta、DeepSeek 均提供大规模样本 |
| 多维并行组合 | 已生产验证 | Megatron、ZeRO、DeepSpeed、DeepSeek-V3 等 |
| 高频故障与自动 requeue | 已生产验证 | Meta Slurm/健康检查/requeue 数据 |
| Checkpoint 跨并行策略 resharding | 已有工业系统证据 | ByteCheckpoint |
| Checkpoint-free / 单步恢复 | 仍偏研究或早期验证 | FlashRecovery 等方案尚缺少独立大规模第三方验证 |
| 十万卡级稳定训练的公开可比数据 | 证据缺口 | 公开资料多为厂商口径或模型外推 |

---

## 3. 推理侧：LLM Serving、KV Cache 与阶段分离

### 3.1 关键证据

| 编号 | 结论 | 关键数字/机制 | 来源 | 年份 | 等级 |
|---|---|---|---|---|---|
| IN1 | 请求级 batch 不适合自回归生成模型 | Iteration-level scheduling；scheduler 每次只执行一个 iteration；selective batching | Orca, OSDI 2022，https://www.usenix.org/system/files/osdi22-yu.pdf | 2022 | A |
| IN2 | Orca 对 GPT-3 175B 的吞吐相对 NVIDIA FasterTransformer 显著提升 | 同等延迟下 36.9× throughput | Orca, OSDI 2022 | 2022 | A |
| IN3 | KV Cache 可采用虚拟内存式分页管理，降低碎片和冗余复制 | PagedAttention；block-based KV memory；提升 batch size 和吞吐 | vLLM, SOSP 2023，https://arxiv.org/abs/2309.06180 | 2023 | A |
| IN4 | Prefill/decode 资源需求不同，合并部署会损害 goodput | Prefill/decode disaggregation；示例中 2 GPU prefill + 1 GPU decode，per-GPU goodput 提升 2.1×；总体可服务 7.4× requests 或支持 12.6× tighter SLO | DistServe, OSDI 2024，https://www.usenix.org/system/files/osdi24-zhong-yinmin.pdf | 2024 | A |
| IN5 | Prompt processing 与 token generation 可拆分到不同机器 | SplitWise；compute-intensive prompt 与低算力 generation 分开 | SplitWise, ISCA 2024，https://www.computer.org/csdl/proceedings-article/isca/2024/265800a118/1Z3pCOHPSuc | 2024 | A |
| IN6 | KV Cache 可成为跨资源池的中心对象 | Prefill/decode 集群分离；GPU 外 CPU/DRAM/SSD/NIC 构建 PB 级 KV Cache；RDMA 最高 8×400 Gbps | Mooncake, FAST 2025，https://www.usenix.org/system/files/fast25-qin.pdf | 2025 | A |
| IN7 | 前缀 KV Cache 复用可显著提升 agent、结构化输出和多轮任务 | RadixAttention；compressed finite state machines；最高 6.4× throughput | SGLang, NeurIPS 2024，https://proceedings.neurips.cc/paper_files/paper/2024/hash/724be4472168f31ba1c9ac630f15dec8-Abstract-Conference.html | 2024 | A |
| IN8 | Chunked prefill 与 stall-free batching 可缓解吞吐/延迟冲突 | Chunked-prefills；stall-free batching；Mistral-7B serving capacity 最高 2.6×；Falcon-180B pipeline 场景最高 5.6× | Sarathi-Serve, OSDI 2024，https://www.usenix.org/system/files/osdi24-agrawal.pdf | 2024 | A |
| IN9 | Speculative decoding 在 serving 中并不总是收益为正 | 自动 profile；feedback-based 调整 intra-request parallelism；goodput 控制 | TurboSpec，https://arxiv.org/abs/2406.14066 | 2024/2025 | A |
| IN10 | 通用 serving 框架正在把动态 batching、流式响应、autoscaling 和多模型组合产品化 | Ray Serve 支持 dynamic batching、streaming、autoscaling、model composition | Ray Serve docs，https://docs.ray.io/en/latest/serve | 2026 页面 | A（文档） |

### 3.2 为什么推理系统的中心资源是 KV Cache

Transformer decoder 在生成第 `t` 个 token 时，需要使用之前 token 的 key/value。若每次重新计算历史 token，成本会随上下文长度快速增长；因此系统把每层、每请求、每 token 的 KV 张量保存在 GPU 显存或更外层介质中。

这带来四个系统问题：

1. **生命周期动态**：请求开始时分配，生成过程中增长，请求结束后释放；长度不可完全预测。
2. **空间碎片**：请求级连续显存预留会产生内部碎片和外部碎片。
3. **复用机会**：系统提示词、公共文档、多轮对话、agent 子任务可能共享前缀。
4. **跨阶段迁移**：prefill 集群生成 KV 后，decode 集群需要读取；跨 GPU、跨节点迁移会消耗 NIC、CPU 和显存。

Orca、vLLM、SGLang、Mooncake 可被理解为沿同一条路径演进：**调度迭代化 → KV 块化 → 前缀复用 → 阶段分离 → KV Cache 中心化**。

### 3.3 Prefill/decode 分离的动因

| 阶段 | 计算特征 | 主要瓶颈 | 适合的资源策略 |
|---|---|---|---|
| Prefill / prompt processing | 一次性处理大量输入 token，矩阵乘 batch 大，计算密集 | 算力、长上下文峰值、与其他请求的排队延迟 | 高算力 GPU；chunked prefill；独立资源池 |
| Decode / token generation | 每次迭代生成少量 token，逐 token 自回归，算术强度低 | HBM 带宽、KV Cache、TPOT、batch 组织 | 内存带宽友好；大 batch；独立 decode 池 |
| KV transfer | prefill 结果迁移到 decode | RDMA/NIC、CPU、序列化和缓存容量 | KV pool；RDMA；统一索引和引用管理 |

DistServe 与 SplitWise 证明“资源需求不同”后，Mooncake 进一步把架构重点从“把任务放到哪类 GPU”推进到“KV Cache 如何作为持久/半持久对象在分离式资源之间流动”。

### 3.4 Speculative decoding 的边界

Speculative decoding 通过小模型、草稿头或树状候选一次产生多个候选 token，再由目标模型验证，以降低小 batch、低负载下的延迟。但 TurboSpec 指出，其收益受以下因素影响：

- 当前 batch size 和请求达到率；
- 草稿模型/候选树的额外计算；
- token acceptance rate；
- 模型、硬件和请求类型；
- KV Cache 额外占用。

因此它不是必然加速，也不应只在 batch size=1 的离线实验中评估。**在高负载、低接受率或显存受限时，speculation 可能降低 goodput**。

---

## 4. 存储：从通用文件系统到 AI 数据路径

### 4.1 关键证据

| 编号 | 结论 | 关键数字/机制 | 来源 | 年份 | 等级 |
|---|---|---|---|---|---|
| ST1 | AI 分布式文件系统需要覆盖训练和推理全数据路径 | Data preparation、dataloader、checkpointing、KV Cache | DeepSeek 3FS，https://github.com/deepseek-ai/3fs | 2025+ | A（官方仓库） |
| ST2 | 3FS 利用 SSD 与 RDMA 实现高吞吐共享存储 | 180 storage nodes；训练背景流量下约 6.6 TiB/s aggregate read | 3FS README | 2025+ | A（官方仓库） |
| ST3 | 3FS 面向大规模数据排序和 shuffle | 25 storage nodes + 50 compute nodes；110.5 TiB；8,192 partitions；30m14s；3.66 TiB/min | 3FS README | 2025+ | A（官方仓库） |
| ST4 | 3FS 使用强一致复制与文件接口简化上层应用 | CRAQ；stateless metadata service；transactional key-value store，如 FoundationDB | 3FS README | 2025+ | A（官方仓库） |
| ST5 | 训练恢复可利用 CPU 内存作为高速 checkpoint 层 | In-memory checkpoint，避免远端存储带宽限制 | Gemini, SOSP 2023 | 2023 | A |
| ST6 | Checkpoint 管理需要支持跨框架、跨后端、跨并行策略 resharding | Industrial-grade checkpointing；parallelism-agnostic representation | ByteCheckpoint | 2024/2025 | A |
| ST7 | 大规模集群通常提供多档存储：易用 POSIX、高带宽数据集缓存、对象存储 | NFS/flash、AirStore、ObjectStore 三类 | Meta reliability paper | 2024/2025 | A |

### 4.2 AI 存储负载与传统负载的差异

| 负载 | 访问模式 | 主要风险 |
|---|---|---|
| Data preparation | 大量中间结果、sort/shuffle、混合读写 | 网络和存储聚合带宽不足；阶段屏障拖慢 |
| Dataloader | 大量并发、随机、小到中块读取；数据常只读一次 | 页缓存收益低；预取不准会让 GPU 等数据 |
| Checkpoint save | 周期性突发大写，TB 级状态 | 写带宽与训练通信/前台流量冲突 |
| Checkpoint load | 重启时全量读取并 reshard | RTO 高；并行拓扑变化导致转换成本 |
| KV Cache | 随机读、写、回收、跨节点迁移 | 缓存层成为新的热点；GC 和一致性复杂 |
| Embedding/vector index | 索引构建、压缩、删除、更新、检索 | 新鲜度、recall、删除语义和重建成本难兼顾 |

### 4.3 3FS 的架构含义

3FS 的关键价值不是单一文件系统 benchmark，而是把 AI 生命周期中的多类 I/O 放入同一高带宽、强一致、RDMA/SSD 友好的共享层：

- 对 dataloader，减少对数据本地性的依赖；
- 对 checkpoint，提供并行写入和重载能力；
- 对 data preparation，支撑大规模 sort/shuffle；
- 对推理 KV Cache，提供高吞吐后端；
- 对应用，保留文件接口并通过强一致语义降低推理成本。

需要注意：README 中的性能来自 DeepSeek 特定硬件、网络和测试配置；不能直接推出在任意 SSD、NIC、交换机或 workload 中都能达到相同吞吐。

---

## 5. 网络与硬件协同：scale-up、scale-out 的重新分层

### 5.1 关键证据

| 编号 | 结论 | 关键数字/机制 | 来源 | 年份 | 等级 |
|---|---|---|---|---|---|
| NW1 | LLM 训练网络需要专门针对突发大流和 collective 通信设计 | Alibaba HPN；前端网络 1:1 oversubscription；训练/推理混部 | Alibaba HPN, SIGCOMM 2024，https://ennanzhai.github.io/pub/sigcomm24-hpn.pdf | 2024 | A |
| NW2 | Meta 已在生产 AI 训练网络中运营 RoCE | Distributed AI training over RoCE；设计、实现与运营经验 | Meta RoCE, SIGCOMM 2024，https://dl.acm.org/doi/10.1145/3651890.3672233 | 2024 | A |
| NW3 | Ultra Ethernet 正在把 AI 负载需求写入开放标准 | Packet spraying、selective retransmission、AI/HPC API、modern lossless delivery | UEC Specification v1.0，https://ultraethernet.org/wp-content/uploads/sites/20/2025/06/UE-Specification-6.11.25.pdf | 2025 | A |
| NW4 | UALink 面向 scale-up pod，定位在 UEC scale-out 之内/之外的另一层 | 单端口最高 800Gbps；一个 scale-up pod 最多 1,024 accelerators | UALink 1.0 webinar，https://ualinkconsortium.org/wp-content/uploads/2025/04/UALink-1.0-Specification-Webinar_FINAL.pdf | 2025 | B（联盟材料） |
| NW5 | NVIDIA NVLink 5 把 rack-scale scale-up 域扩大到 72 GPU，并宣称可扩展到 576 GPU | GB200 NVL72：72 GPUs、30TB unified memory、>1PB/s total NVLink bandwidth；单 GPU 1.8TB/s bidirectional；576 GPU domain 为设计能力 | NVIDIA blog，https://developer.nvidia.com/blog/nvidia-gb200-nvl72-delivers-trillion-parameter-llm-training-and-real-time-inference/ | 2024 | B |
| NW6 | InfiniBand Quantum-X800 进入 800Gb/s end-to-end 互连 | Quantum-X800；ConnectX-8；144-port Q3400；800Gb/s per port | NVIDIA docs，https://networking-docs.nvidia.com/nvidia-quantum-x800-xdr-clusters | 2025/2026 页面 | A（官方文档） |
| NW7 | NCCL 正在扩展网络卸载和更多 collective | SHARP 扩展到 AllGather/ReduceScatter；SM 占用从 16+ 降至 ≤6 | NVIDIA NCCL 2.27 blog，https://developer.nvidia.com/blog/enabling-fast-inference-and-resilient-training-with-nccl-2-27 | 2025 | B |
| NW8 | Google Jupiter 使用 OCS、WDM、SDN，并把网络作为 building-scale 基础设施 | 2022 博客称 flow completion time 降低 10%、吞吐提升 30%、功耗降低 40%、成本降低 30%、停机时间减少 50×；Trillium 页面称 13 Pb/s Jupiter | Google，https://cloud.google.com/blog/topics/systems/the-evolution-of-googles-jupiter-data-center-network | 2022/2024 | B |
| NW9 | TPU Trillium 使用 256-chip pod、ICI 与多 pod 扩展 | 256 TPU/pod；数百 pods；最高 91 exaflops cluster；Llama-2-70B 36-pod 99% scaling efficiency | Google Trillium，https://cloud.google.com/blog/products/compute/trillium-tpu-is-ga | 2024 | B |
| NW10 | RoCE/RDMA 拥塞控制依赖 PFC、ECN 等机制，但其生产部署并非简单开关 | DCQCN；Meta DCTCP 大规模部署经验 | DCQCN，https://conferences.sigcomm.org/sigcomm/2015/pdf/papers/p523.pdf；DCTCP NSDI 2024，https://www.usenix.org/system/files/nsdi24-dhamija.pdf | 2015/2024 | A |

### 5.2 Scale-up 与 scale-out 的新边界

| 层级 | 典型技术 | 适合的通信 | 设计重点 |
|---|---|---|---|
| Package/chip 内 | GPU 内 HBM、片上互连 | 最高密度算子和内存访问 | 带宽、功耗、HBM 容量 |
| Board/server | NVLink-C2C、PCIe、CXL | CPU-GPU、少量加速器、内存扩展 | 一致性、成本、延迟 |
| Rack/pod scale-up | NVLink/NVSwitch、ICI、UALink | Tensor parallel、小规模 AllToAll、共享内存域 | 全互联、高基数、故障域 |
| Multi-rack scale-out | InfiniBand、Ethernet/UEC、RoCE | Data parallel、跨 pod collective、大规模 MoE | 路由、拥塞控制、容错、成本 |
| 跨站点/多集群 | 广域、多集群调度 | 容灾、资源池化、混合云 | 延迟、一致性、安全和调度边界 |

UALink 材料把 3–4 racks 等近距 scale-up 场景与 UALink 关联，把多 rack 扩展交给 UEC；NVIDIA 则通过 NVLink + Quantum/Spectrum 形成垂直栈。这说明行业尚未形成唯一分层，但共识是：**scale-up 和 scale-out 不能再用同一种网络抽象粗略处理**。

### 5.3 以太网反攻与 InfiniBand 的差异

- InfiniBand 优势在端到端生态成熟、低延迟、RDMA、in-network computing、子网管理和 AI/HPC 长期部署经验。
- UEC 方向强调开放以太网、packet spraying、selective retransmission、端到端拥塞控制、多供应商互操作和 AI/HPC API。
- RoCE 路线希望保留以太网基础设施，同时通过 RDMA、PFC/ECN、DCQCN 和运营经验满足 AI 训练。

需要谨慎的是：标准发布不等于大规模生产成熟。UEC 1.0 是强标准证据，但不同厂商实现、拥塞控制、运维工具、故障定位和多代网络互操作仍需生产周期验证。

### 5.4 CXL 与内存池

CXL 3.x switch 已被用于 GPU/内存池、RAG 向量库内存池等演示。联盟和厂商材料展示了通过 CXL switch 扩展内存、池化资源和降低 GPU 本地内存压力的可能。

但截至本次调研，公开证据主要是演示、联盟博客和厂商方案，缺少与 NVLink、统一内存、对象/文件缓存方案相比的大规模生产收益。因此 CXL 内存池应归类为 **有明确系统机会、但生产成熟度未充分证明**。

---

## 6. 调度、利用率、混部与多租户隔离

### 6.1 关键证据

| 编号 | 结论 | 关键数字/机制 | 来源 | 年份 | 等级 |
|---|---|---|---|---|---|
| SC1 | 多训练 job 共享网络时，通信竞争会造成 GPU 浪费 | 36.3% DLT jobs 可能遭遇 job 间通信竞争；测试床提升 8.3%–14.8%；trace 仿真最高提升 23% | Crux, SIGCOMM 2024，https://cs.stanford.edu/~keithw/sigcomm2024/sigcomm24-final380-acmpaginated.pdf | 2024 | A |
| SC2 | GPU 集群调度必须处理不可预测运行时长、all-or-nothing allocation 和抢占恢复 | Tiresias；GPU cluster scheduler；preemption and placement | Tiresias, NSDI 2019 / project，https://github.com/symbioticlab/tiresias | 2019 | A（早期基础） |
| SC3 | Slurm 是 Meta RSC 使用的调度底座 | Bare-metal allocations；topology co-location；2 小时后可抢占；最长 7 天；健康检查失败自动 requeue | Meta reliability paper | 2024/2025 | A |
| SC4 | Volcano 将 gang scheduling、队列、GPU 共享和统一调度带入云原生 AI | Gang plugin 满足 all-or-nothing；支持 Ray/PyTorch/MPI/Spark 等 | Volcano docs，https://volcano.sh/en/docs/v1-10-0/plugins | 2024 | A（文档） |
| SC5 | Kueue 定位为 Kubernetes-native job queueing 与配额/资源治理 | Fair sharing、topology-aware placement、preemption、multi-cluster；支持 JobSet、KubeRay、Kubeflow、LeaderWorkerSet | Kueue docs，https://kueue.sigs.k8s.io/ | 2025/2026 页面 | A（文档） |
| SC6 | Kubernetes 原生 Workload/PodGroup/gang scheduling 已进入正式增强流程 | KEP-4671；Workload API、PodGroup；AI training/inference workloads；v1.35 alpha、v1.38 stable 规划 | Kubernetes KEP-4671，https://www.kubernetes.dev/resources/keps/4671 | 2025 | A |
| SC7 | Google 通过细粒度时间复用和按需内存分页提升 GPU 聚合吞吐 | GASS；aggregated GPU throughput +50%；支持 19% more GPU jobs | GASS, IEEE Cloud 2024，https://store.computer.org/csdl/proceedings-article/cloud/2024/685300a439/1ZMel2e2RTa | 2024 | A/B |
| SC8 | Inference/training colocation 可通过空间共享提升资源利用，但必须保证延迟 SLO | GPUColo；57 天、6,500 GPU trace；仿真显示可节省 74.9% GPUs | ICDCS 2024，https://ieeexplore.ieee.org/document/10630927；代码 https://github.com/halegchen/GPU-Co-location | 2024 | A |

### 6.2 为什么 GPU 调度比 CPU 调度更强调“整体性”

GPU job 的常见约束包括：

- 一个 job 的多个 task 必须同时启动，否则已启动部分空等；
- 并行 rank 之间存在网络拓扑依赖；
- GPU、NIC、NVLink、CPU/内存和存储通道可能同时成为瓶颈；
- 运行时长和输出长度难以预测；
- 抢占成本高，需要 checkpoint；
- 故障影响整个 gang，而不是单个 task；
- 大 job requeue 会引发大量低优先级 job 的二次抢占。

因此 AI 调度不是单纯 bin packing，而是 **资源配额、队列公平、拓扑、通信、碎片、弹性、SLO、故障恢复** 的多目标优化。

### 6.3 MIG、MPS、时间复用与隔离边界

| 技术 | 隔离方式 | 优点 | 主要限制 |
|---|---|---|---|
| MIG | SM、显存、cache、内存带宽等硬件分区 | 隔离最强；错误边界清晰；适合强多租户 | 分区粒度固定；动态重配有成本；不适合所有并行框架 |
| MPS | 多进程共享 SM，内存系统仍共享 | 粒度灵活；利用率高；部署较方便 | 干扰和故障边界弱；内存/缓存/TLB 竞争明显 |
| Time-slicing | GPU 时间片切换 | 兼容性好；易提升任务密度 | 上下文切换和 QoS 复杂；显存与性能隔离弱 |
| GASS 类方案 | 细粒度时间复用 + on-demand paging + 监控重调度 | 聚合吞吐提升明显 | 需要实时异常检测和自适应调度 |
| SR-IOV / 虚拟化 | 设备虚拟化和隔离 | 云化和权限隔离 | 对 AI collective、拓扑和性能损耗敏感 |

MIG 并非绝对无干扰。2024 年关于 MIG TLB 的研究指出，A100 MIG 的 L3 TLB 未完全按实例分区，多租户 workload 中可能出现 TLB contention。该结论说明即使硬件隔离也存在共享微架构资源，需要按具体代际和 workload 验证。

### 6.4 混部的主要模式

| 模式 | 思路 | 风险 |
|---|---|---|
| Training + offline inference | 用离线推理填充训练空闲 | 批量任务峰值和 I/O 干扰 |
| Online inference + training | 空间共享，利用训练任务空隙满足在线 SLO | decode/prefill 延迟尖峰；隔离难 |
| Prefill + decode 分离 | 不同阶段独立资源池 | KV 迁移和容量规划复杂 |
| Big data + AI | Spark/Flink 与 AI 任务共享集群 | CPU/内存/网络瓶颈不同 |
| Spot/preemptible + checkpoint | 用低成本资源承受抢占 | checkpoint 频率和恢复成本决定收益 |

---

## 7. 向量检索、RAG 与数据新鲜度

### 7.1 关键证据

| 编号 | 结论 | 关键数字/机制 | 来源 | 年份 | 等级 |
|---|---|---|---|---|---|
| VEC1 | SSD 支持的 ANN 可在单节点上处理十亿级向量 | DiskANN；64GB RAM + SSD；10 亿点；>5,000 QPS；<3ms 平均延迟；95%+ 1-recall@1 | NeurIPS 2019，https://proceedings.neurips.cc/paper_files/paper/2019/hash/09853c7fb1d3f8ee67a61b6bf4a7f8e6-Abstract.html | 2019 | A |
| VEC2 | 流式向量索引需要同时处理插入、删除和搜索 | FreshDiskANN；实时 insert/delete/search；>95% 5-recall@5；维护成本降低 5–10× | arXiv，https://arxiv.org/abs/2105.09613 | 2021 | A |
| VEC3 | Milvus 支持多级别一致性 | Strong、Bounded、Eventually、Session；默认 Bounded；DataNode/QueryNode，object storage | Milvus docs，https://milvus.io/docs/v2.5.x/consistency.md | 2025/2026 页面 | A（文档） |
| VEC4 | Milvus 采用存算分离和分布式微服务架构 | Shared-storage；storage/computing disaggregation；Kubernetes；流式摄入 | Milvus architecture，https://milvus.io/docs/v2.2.0/architecture_overview.md | 2022+ | A（文档） |
| VEC5 | Pinecone serverless 将对象存储作为索引 source of truth，并分离读写计算 | Blob storage source of truth；log；freshness layer；index builder；读写独立扩缩容 | Pinecone architecture，https://www.pinecone.io/blog/serverless-architecture/；docs https://docs.pinecone.io/guides/get-started/architecture | 2024 | B |
| VEC6 | Weaviate 将元数据复制与数据复制分开 | Metadata 使用 Raft；data replication 为 leaderless、tunable consistency；优先可用性 | Weaviate docs，https://docs.weaviate.io/weaviate/concepts/replication-architecture | 2025/2026 页面 | A（文档） |
| VEC7 | 租户分片和冷数据卸载是向量数据库成本控制手段 | 每租户一个 shard；inactive/offloaded；S3 offload | Weaviate docs，https://docs.weaviate.io/weaviate/concepts/cluster | 2025/2026 页面 | A（文档） |

### 7.2 RAG 数据路径中的系统问题

生产 RAG 不只是“文档切块 + 调 embedding + 写入向量库”。完整链路包括：

1. 数据源变更捕获：CDC、事件、批量扫描或对象存储事件；
2. 文档解析和切块：结构、表格、图片、代码块和跨块语义；
3. Embedding：模型版本、批处理、失败重试、成本和延迟；
4. 向量写入：upsert、删除、版本号、命名空间和租户隔离；
5. 索引更新：在线增量、后台重建、compaction、recall 保障；
6. 查询：hybrid search、metadata/ACL 过滤、rerank、上下文组装；
7. 一致性：source DB、文档索引、向量索引和权限缓存之间的新鲜度对齐。

### 7.3 证据边界

本次检索找到大量关于 CDC → Kafka/Debezium → embedding service → vector DB 的教程和架构文章，但多数是个人博客、厂商内容或课程材料，可说明常见模式，不能支撑“分钟级新鲜度已在多数企业生产验证”或“某架构普遍降低多少成本”。

更高等级的系统证据主要来自：

- FreshDiskANN 对动态索引机制和维护成本的研究；
- Milvus/Pinecone/Weaviate 官方架构文档；
- 各数据库支持的一致性级别、存算分离和租户模型。

因此应把 **RAG 数据新鲜度工程** 归类为“架构模式清晰、产品能力快速成熟，但跨行业生产收益数据不足”。

---

## 8. Learned Systems、AIOps 与 LLM Agent

### 8.1 Learned systems 的现状

| 方向 | 当前判断 | 证据 |
|---|---|---|
| Learned query optimizer | 在研究和特定 benchmark 中持续改进，但不是生产数据库默认替代 | GLO, ICDE 2024，在未见过的 TPC-DS 查询上较 LOGER 快 1.4×、较 Balsa 快 2.1×，https://www.computer.org/csdl/proceedings-article/icde/2024/171500e843/1YOtHMz7Eha |
| Learned index | 学术影响大，但生产落地受 workload、更新和退化边界限制 | 本次未找到 2024–2026 主流数据库全面替换 B-tree 的强一手证据 |
| Learned scheduler/cache/拥塞控制 | 有研究价值，但生产系统更偏好可控、可解释、可回滚机制 | Meta DCTCP 部署经验表明生产网络变更需长期灰度和监控 |
| LLM for database operations | 在诊断、推荐、SQL/代码辅助上更现实 | Microsoft RCA、Triangle 等提供生产 incident 数据集和局部部署证据 |

结论：**learned systems 更可能先在窄接口、可回滚、离线评估充分的模块中落地，而不是一次性替换数据库、调度器或拥塞控制核心。**

### 8.2 AIOps 与 AgentOps

| 编号 | 结论 | 数字/机制 | 来源 | 年份 | 等级 |
|---|---|---|---|---|---|
| AI1 | LLM agent 可通过工具动态获取日志、指标和诊断信息，改进 RCA | ReAct agent + retrieval tools；Microsoft production incidents；factual accuracy 提升 | FSE 2024，https://arxiv.org/abs/2403.04123 | 2024 | A |
| AI2 | 多 agent 可用于 incident triage | Analyzer、Decider、Team Manager agents；协商、信息收集和缓解 | ASE 2025，https://www.microsoft.com/en-us/research/wp-content/uploads/2025/02/TRIANGLE_ASE25.pdf | 2025 | A |
| AI3 | Azure 局部分级系统已有生产部署 | 2024 年中起生产；2025-01 有 6 teams in production、15+ onboarding；约 90% accuracy；一个团队 TTM 降低 38% | Microsoft Azure blog，https://azure.microsoft.com/en-us/blog/optimizing-incident-management-with-aiops-using-the-triangle-system | 2025 | B |
| AI4 | 论文系统在生产环境中报告更高分级效率 | Triage accuracy up to 97%；TTE reduced up to 91% | Triangle paper | 2025 | A/B |
| AI5 | AIOps agent 评估需要故障注入、工作负载、遥测和 agent-cloud interface | AIOpsLab；100 problems；多 agents 评估；开源框架 | MLSys 2025，https://proceedings.mlsys.org/paper_files/paper/2025/hash/d1f9e4a9f109b6e8b75ed362736f22ec-Abstract-Conference.html | 2025 | A |
| AI6 | 企业对 AI 影响 observability 有高预期，但生产成熟度不能由预期替代 | 96% 受访者预期 AI 五年内影响 observability；厂商赞助调查 | Elastic/Dimensional Research，https://www.elastic.co/pdf/dimensional-research-elastic-the-2024-observability-landscape.pdf | 2024 | B/C |
| AI7 | 多 agent 系统适合广度优先研究，但成本和协调复杂度显著上升 | Orchestrator-worker；简单事实查找约 1 agent/3–10 tool calls；复杂研究可超过 10 subagents | Anthropic Engineering，https://www.anthropic.com/engineering/multi-agent-research-system | 2025 | B |

### 8.3 LLM agent 作为系统组件带来的新问题

Anthropic 对多 agent research system 的工程总结指出，agent 是在循环中持续使用工具的系统，长任务、状态累积、工具失败和错误传播会带来明显工程挑战。与传统工作流相比，agent 系统还会引入：

- 长上下文和多子 agent 上下文管理；
- 工具调用次数和 token 成本的高方差；
- 工具失败后的重试、降级和状态恢复；
- 子任务结果可验证性；
- 人机审批和权限边界；
- 长事务/长运行流程的持久化执行；
- 评估从单次输出扩展到多步骤状态变化。

这类系统与 Temporal、Airflow 等传统工作流的关系不是简单替代：传统工作流强在持久化、重试、超时和确定性控制；agent 强在语义理解、工具选择和开放式推理。生产架构更可能是 **确定性工作流引擎承载状态与边界，LLM agent 承担受限决策**。

---

## 9. 训练侧 vs 推理侧：架构变化对比

| 维度 | 训练侧倒逼 | 推理侧倒逼 |
|---|---|---|
| 主要目标 | 扩展到更大模型和更多 token；维持 MFU/ETTR | 满足 TTFT/TPOT/吞吐/成本；提升单 GPU goodput |
| 典型规模 | 单个 job 数百到数万 GPU/加速器 | 单实例到多资源池；请求并发和上下文长度波动大 |
| 通信模式 | AllReduce、AllGather、ReduceScatter、pipeline、MoE AllToAll | KV transfer、跨 prefill/decode 通信、检索调用、流式响应 |
| 网络重点 | Collective 性能、rail 拓扑、无阻塞/低拥塞、故障恢复 | KV Cache 迁移、低延迟、前后端分离、推理集群弹性 |
| 存储重点 | Dataloader 随机读、checkpoint 突发写、resharding | KV Cache、模型加载、向量索引、多轮/前缀复用 |
| 调度重点 | Gang、拓扑、抢占、quota、长 job、故障重排队 | SLO、continuous batching、prefill/decode、autoscaling、cache-aware routing |
| 故障成本 | 大量 GPU 同时停摆，checkpoint 重放 | 单请求失败/卡顿、cache 丢失、尾延迟和雪崩 |
| 隔离问题 | 多训练 job 通信竞争、资源抢占 | 多租户请求、模型/LoRA、KV/显存、权限和数据隔离 |
| 架构趋势 | 多维并行 + 高速互连 + 可靠 checkpoint | 迭代调度 + KV 分页/复用 + 阶段分离 + KV 中心化 |

---

## 10. 生产验证、研究中与炒作分类

### 10.1 已生产验证或有强系统证据

- 千卡级大模型训练；
- Tensor/data/pipeline/expert 并行组合；
- GPU gang scheduling 和拓扑感知放置；
- 基于健康检查的节点隔离、requeue 和作业恢复；
- Iteration-level / continuous batching；
- PagedAttention 式 KV Cache 管理；
- Prefill/decode 分离的生产和系统论文证据；
- 高吞吐 RDMA/SSD 存储支撑 dataloader、checkpoint 等 AI 数据路径；
- 向量数据库的分片、存算分离、多租户和多一致性级别；
- AIOps agent 在局部 incident triage/RCA 场景中的受限生产部署。

### 10.2 仍是研究、早期产品或局部验证

- Checkpoint-free / 单步恢复；
- 跨引擎、跨集群 KV Cache 迁移和统一缓存层的通用标准；
- CXL 大规模 GPU 内存池；
- UALink/UEC 多供应商完整生态；
- 训练与在线推理的大规模无风险混部；
- 动态 MIG 重配置；
- Learned scheduler、learned congestion control、learned cache 的生产替代；
- Agent 自主执行高风险修复；
- 跨数据中心/跨区域十万卡级统一调度；
- RAG 的端到端事务一致性和自动 embedding 版本迁移。

### 10.3 更接近炒作或证据不足的说法

- “AI 将让分布式系统彻底不需要传统数据库/工作流/调度器”；
- “LLM agent 已能普遍自治数据中心”；
- “某 serving 技术在所有场景下线性提升吞吐”；
- “CXL 会立即替代 GPU HBM/NVLink”；
- “以太网/InfiniBand 一方已无条件胜出”；
- “GPU 利用率普遍只有某个固定低值”——缺少统一口径和可比样本；
- “RAG 只要接入向量库即可保证新鲜和准确”；
- “Speculative decoding 必然降低延迟/成本”。

---

## 11. 证据缺口清单

1. **跨厂商 GPU/TPU 年化故障率与 MTBF 的可比数据**：GPU、HBM、NVLink、NIC、交换机故障定义不统一，厂商数据少。
2. **十万卡到百万卡级训练的公开生产数据**：现有强数据多在数千到一万多 GPU，更大规模常为预测或厂商口径。
3. **Checkpoint I/O 成本的细粒度公开数据**：缺少不同模型、并行策略、存储后端下的写入/加载/resharding 成本对比。
4. **GPU 利用率“20%–40%”等流行说法的权威来源**：DCGM 利用率、SM active、MFU、goodput、ETTR 是不同指标，不能混用。
5. **训练/推理混部在长期生产中的 SLO 和故障率数据**：短期实验较多，跨团队长期运营样本少。
6. **MIG/MPS/SR-IOV 在云生产中的安全隔离与性能干扰对比**：微架构共享资源、错误边界和权限模型需逐代验证。
7. **Speculative decoding 的生产收益分布**：需要按请求长度、到达率、acceptance rate、模型和 batch 分位数公开数据。
8. **KV Cache 跨厂商、跨引擎标准化程度**：vLLM、SGLang、TensorRT-LLM、Mooncake 等实现和语义尚未完全统一。
9. **Milvus/Pinecone/Weaviate 的独立生产基准**：官方文档能证明架构能力，但成本、可用性、新鲜度和召回率缺少统一第三方测试。
10. **RAG CDC pipeline 的事务语义**：source DB、文档内容、embedding 版本、向量索引、ACL 更新之间如何原子切换，仍缺少标准和跨产品强证据。
11. **Learned systems 在主流生产数据库/调度器中的默认部署证据**：论文 benchmark 与生产默认能力之间仍有距离。
12. **LLM agent 执行修复动作的风险模型**：权限分级、审批、回滚、误操作成本和责任边界尚缺成熟工程标准。

---

## 12. 参考来源索引

1. MegaScale: Deploying a 10,000-GPU Cluster for Large-Scale DL Training, NSDI 2024 — https://www.usenix.org/system/files/nsdi24-jiang-ziheng.pdf
2. Revisiting Reliability in Large-Scale Machine Learning Research Clusters, Meta/HPCA 2025 — https://arxiv.org/abs/2410.21680
3. DeepSeek-V3 Technical Report — https://arxiv.org/abs/2412.19437
4. ByteCheckpoint — https://arxiv.org/abs/2407.20143
5. Gemini: in-memory checkpoint training recovery, SOSP 2023 — https://dl.acm.org/doi/10.1145/3600006.3613145
5a. FlashRecovery — https://arxiv.org/abs/2509.03047
6. Megatron-LM — https://arxiv.org/abs/1909.08053
7. ZeRO / DeepSpeed — https://www.microsoft.com/en-us/research/blog/zero-deepspeed-new-system-optimizations-enable-training-models-with-over-100-billion-parameters
8. Orca, OSDI 2022 — https://www.usenix.org/system/files/osdi22-yu.pdf
9. vLLM/PagedAttention, SOSP 2023 — https://arxiv.org/abs/2309.06180
10. DistServe, OSDI 2024 — https://www.usenix.org/system/files/osdi24-zhong-yinmin.pdf
11. SplitWise, ISCA 2024 — https://www.computer.org/csdl/proceedings-article/isca/2024/265800a118/1Z3pCOHPSuc
12. Mooncake, FAST 2025 — https://www.usenix.org/system/files/fast25-qin.pdf
13. SGLang, NeurIPS 2024 — https://proceedings.neurips.cc/paper_files/paper/2024/hash/724be4472168f31ba1c9ac630f15dec8-Abstract-Conference.html
14. Sarathi-Serve, OSDI 2024 — https://www.usenix.org/system/files/osdi24-agrawal.pdf
15. TurboSpec — https://arxiv.org/abs/2406.14066
16. DeepSeek 3FS — https://github.com/deepseek-ai/3fs
17. Alibaba HPN, SIGCOMM 2024 — https://ennanzhai.github.io/pub/sigcomm24-hpn.pdf
18. Meta RoCE, SIGCOMM 2024 — https://dl.acm.org/doi/10.1145/3651890.3672233
19. Ultra Ethernet Specification v1.0 — https://ultraethernet.org/wp-content/uploads/sites/20/2025/06/UE-Specification-6.11.25.pdf
20. UALink 1.0 Webinar — https://ualinkconsortium.org/wp-content/uploads/2025/04/UALink-1.0-Specification-Webinar_FINAL.pdf
21. NVIDIA GB200 NVL72 — https://developer.nvidia.com/blog/nvidia-gb200-nvl72-delivers-trillion-parameter-llm-training-and-real-time-inference/
22. NVIDIA Quantum-X800 — https://networking-docs.nvidia.com/nvidia-quantum-x800-xdr-clusters
23. NCCL 2.27 — https://developer.nvidia.com/blog/enabling-fast-inference-and-resilient-training-with-nccl-2-27
24. Google Jupiter — https://cloud.google.com/blog/topics/systems/the-evolution-of-googles-jupiter-data-center-network
25. Google Trillium GA — https://cloud.google.com/blog/products/compute/trillium-tpu-is-ga
26. DCQCN, SIGCOMM 2015 — https://conferences.sigcomm.org/sigcomm/2015/pdf/papers/p523.pdf
27. Meta DCTCP deployment, NSDI 2024 — https://www.usenix.org/system/files/nsdi24-dhamija.pdf
28. Crux, SIGCOMM 2024 — https://cs.stanford.edu/~keithw/sigcomm2024/sigcomm24-final380-acmpaginated.pdf
29. Tiresias — https://github.com/symbioticlab/tiresias
30. Volcano plugins — https://volcano.sh/en/docs/v1-10-0/plugins
31. Kueue — https://kueue.sigs.k8s.io/
32. Kubernetes KEP-4671 — https://www.kubernetes.dev/resources/keps/4671
33. GASS — https://store.computer.org/csdl/proceedings-article/cloud/2024/685300a439/1ZMel2e2RTa
34. DiskANN, NeurIPS 2019 — https://proceedings.neurips.cc/paper_files/paper/2019/hash/09853c7fb1d3f8ee67a61b6bf4a7f8e6-Abstract.html
35. FreshDiskANN — https://arxiv.org/abs/2105.09613
36. Milvus consistency — https://milvus.io/docs/v2.5.x/consistency.md
37. Pinecone serverless architecture — https://www.pinecone.io/blog/serverless-architecture/
38. Weaviate replication — https://docs.weaviate.io/weaviate/concepts/replication-architecture
39. Microsoft LLM-based RCA Agents, FSE 2024 — https://arxiv.org/abs/2403.04123
40. Microsoft Triangle, ASE 2025 — https://www.microsoft.com/en-us/research/wp-content/uploads/2025/02/TRIANGLE_ASE25.pdf
41. Microsoft Azure Triangle blog — https://azure.microsoft.com/en-us/blog/optimizing-incident-management-with-aiops-using-the-triangle-system
42. AIOpsLab, MLSys 2025 — https://proceedings.mlsys.org/paper_files/paper/2025/hash/d1f9e4a9f109b6e8b75ed362736f22ec-Abstract-Conference.html
43. Ray Serve — https://docs.ray.io/en/latest/serve
44. Elastic 2024 observability survey — https://www.elastic.co/pdf/dimensional-research-elastic-the-2024-observability-landscape.pdf
45. Anthropic multi-agent research system — https://www.anthropic.com/engineering/multi-agent-research-system
