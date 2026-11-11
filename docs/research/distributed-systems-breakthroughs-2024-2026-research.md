# 2024–2026 年分布式底层技术真实突破研究报告

## 1. 研究范围与证据标准

### 1.1 时间范围

- **2024 年 1 月至 2026 年 11 月**。
- 若一项技术的早期版本早于 2024 年，但在该时间窗内发布关键版本、完成生产落地或形成重要演进，也纳入分析。

### 1.2 覆盖方向

1. 共识协议
2. 事务与一致性
3. 分布式存储
4. 数据处理与流批一体
5. 系统可靠性、可观测性与运维
6. 资源管理、调度与新型硬件互连

### 1.3 证据等级

| 等级 | 定义 | 可信度 |
|---|---|---|
| A | 顶级会议同行评审论文、正式标准、带 artifact 的学术系统 | 最高 |
| B | 官方工程博客、正式发布说明、官方规范文档、开源仓库一手材料 | 高 |
| C | 厂商技术演讲、行业会议材料、有明确来源的生产实践 | 中 |
| D | 媒体报道、二手分析、无一手材料支撑的说法 | 低 |

### 1.4 分类口径

- **范式级突破**：提出新的核心抽象、改变系统设计边界，或显著突破既有理论/工程权衡。
- **重要演进**：不是全新范式，但在关键机制、性能或可落地性上有实质推进。
- **工程优化**：主要是已有技术的实现优化、集成优化或部署成熟。
- **概念炒作/旧问题新包装**：缺少一手证据，或本质上只是既有思想换名。

---

## 2. 总体结论

2024–2026 年分布式系统领域**没有出现类似 MapReduce、Spanner、Dynamo、CRDT、RDMA disaggregation 那样数量密集的范式级革命**。多数核心问题仍然受限于 CAP、FLP、跨地域延迟、故障独立、复制与恢复成本等老约束。

但这一阶段出现了三条非常清晰的实质性演进主线：

1. **AI 工作负载正在反向重塑分布式底层系统**  
   3FS、FuseLink、Meta Ethernet RDMA、UEC、向量检索索引、GPU checkpoint、KVCache 存储等工作，都不是传统数据库或通用云存储的自然延伸，而是围绕大模型训练、推理、checkpoint、KVCache、集合通信和随机读取重新设计。

2. **时间、日志和 DAG 继续成为协调原语**  
   LazyLog、Tiga、Mako、Autobahn、Shoal++ 等工作表明，在高速网络、精确时钟和异步传播层成熟后，系统正在进一步合并“事务排序、复制、共识、并发控制”这些原本分离的层次。

3. **CXL、RDMA、DPU、用户态运行时正在把分布式系统推向微秒级**  
   Tigon、Skyloft、Okapi、Nos、Modal snapshot 等工作表明，新型硬件和用户态 I/O 改变了协调成本结构。但这些技术多数仍处于研究、有限生产或特定场景阶段，不能夸大为全面普及。

总体判断：

- **真正有范式意义的候选成果约 2–4 个**：3FS、Tigon/Tiga 所代表的 CXL/时钟协调方向、AI 时代以太网 RDMA/UEC 生态、LazyLog 的共享日志抽象。
- **更准确的表述是“重要演进密集，基础范式有限突破”**。
- **大量“Serverless 3.0”“智能运维革命”“零一致性成本”“AI Native 数据库”等说法证据不足**，应视为市场叙事而非已验证的技术突破。

---

## 3. 分方向结论

## 3.1 共识协议

### 总体判断

**传统 CFT 共识在 2024–2026 年没有出现新的统一天花板级协议。** MultiPaxos、Raft、EPaxos、Compartmentalized Paxos、NOPaxos 等基本思想仍主导实际系统。研究热点明显转向：

- BFT 与 DAG-BFT；
- 多实例、多分片 RSM 之间的高效通信；
- 共识与事务、时钟、日志和网络硬件的合并；
- 自动验证复杂协议不变式。

### 代表成果

#### 1. LazyLog：延迟绑定全局顺序的共享日志

| 字段 | 内容 |
|---|---|
| 年份 | 2024 |
| 提出者 | Xuhao Luo、Shreesha G. Bhat、Jiyu Hu、Ramnatthan Alagappan、Aishwarya Ganesan |
| 发表场景 | SOSP 2024，pp. 296–312 |
| 证据等级 | A |
| 分类 | 范式级候选 / 重要突破 |

LazyLog 重新设计共享日志抽象：写入时不立即强制跨分片记录获得全局可线性化位置，而是把跨分片顺序延迟绑定，直到读取或确有必要时才强制全局排序。其目标是降低多分片应用写入摄取路径上的延迟，同时保留需要时的强顺序语义。

重要性在于：它不是又一个 Paxos/Raft 变体，而是在**日志 API 的语义层**重新安排“何时必须知道全局顺序”。这反映了现代低延迟应用中“写入摄取”和“读取确认顺序”可以解耦的趋势。

链接：

- https://dl.acm.org/doi/10.1145/3694715.3695983

#### 2. Autobahn：兼顾低延迟、高吞吐和 blip 恢复的 BFT

| 字段 | 内容 |
|---|---|
| 年份 | 2024 |
| 提出者 | Neil Giridharan、Florian Suri-Payer、Ittai Abraham、Lorenzo Alvisi、Natacha Crooks |
| 发表场景 | SOSP 2024，pp. 1–23 |
| 证据等级 | A |
| 分类 | 重要演进 |

Autobahn 针对部分同步 BFT 协议中的长期权衡：传统 view-based HotStuff 类协议在正常区间延迟低，但领导者或网络出现短暂抖动后可能产生持续 hangover；DAG-BFT 从异常中恢复更平滑，但正常情况下延迟较高。

Autobahn 将高度并行的异步数据传播层与低延迟部分同步共识机制结合，声称：

- 吞吐可匹配 Bullshark 等 DAG-BFT；
- 延迟约为 DAG-BFT 的一半；
- 从 blip 恢复后可快速提交积压请求，避免协议自身造成的长尾。

重要性：它不是单点性能优化，而是试图同时解决 **latency、throughput、robustness** 三者的冲突。

链接：

- https://dl.acm.org/doi/10.1145/3694715.3695942
- https://arxiv.org/abs/2401.10369

#### 3. Shoal++：增强 DAG-BFT 的鲁棒性

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 发表场景 | NSDI 2025 |
| 证据等级 | A |
| 分类 | 重要演进 |

Shoal++ 属于 DAG-BFT 路线。DAG-BFT 通过把数据传播与共识排序解耦，让每个副本都可参与高吞吐传播，减少单领导者瓶颈；Shoal++ 进一步改善其在异常情况下的健壮性与提交延迟。

重要性主要体现在 BFT 系统工程化：DAG 不再只是区块链叙事，而成为高吞吐 BFT 复制的有效结构。但它对普通崩溃故障容忍的通用数据库系统影响仍有限。

链接：

- https://www.usenix.org/system/files/nsdi25-arun.pdf

#### 4. Picsou：让多个复制状态机高效通信

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 发表场景 | OSDI 2025 |
| 证据等级 | A |
| 分类 | 重要演进 |

Picsou 关注的不是单个 RSM 内部如何达成共识，而是多个 RSM 之间如何通信。它支持不同规模、不同通信模型甚至不同故障模型的 RSM 互连，例如 BFT 与 CFT 系统之间的互操作。

重要性在于，随着系统拆分为大量独立复制服务，传统“一个集群一个共识池”的模型不够用了，**跨 RSM 通信正在成为共识层的一等问题**。

链接：

- https://www.usenix.org/system/files/osdi25-frank.pdf

#### 5. Basilisk：自动证明不可判定协议的安全不变式

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 发表场景 | OSDI 2025 Best Paper |
| 证据等级 | A |
| 分类 | 可靠性方法突破，共识协议本身突破 |

Basilisk 不是新共识协议，而是自动为复杂分布式协议生成和证明 inductive invariants 的工具。它通过 Provenance Invariants 追踪主机状态来源，并通过 Atomic Sharding 静态分析协议步骤，自动推导可证明安全性的不变式集合。

重要性在于：协议设计的瓶颈往往不是提出协议，而是证明协议在所有异步交错下正确。Basilisk 把专家手工寻找不变式的过程部分自动化。

链接：

- https://www.usenix.org/conference/osdi25/presentation/zhang-tony
- https://cse.engin.umich.edu/stories/cse-researchers-win-best-paper-award-at-osdi-2025

### 本方向结论

- **有真实进展**：BFT、DAG-BFT、共享日志、跨 RSM 通信、自动验证。
- **无明显通用 CFT 范式革命**：尚未出现全面替代 Raft/Paxos 的生产级新共识范式。
- **需要警惕**：区块链语境中的“DAG 共识”常被包装成通用分布式系统突破，实际上多数系统仍主要适用于许可链或特定 BFT 部署。

---

## 3.2 事务与一致性

### 总体判断

2024–2026 年事务领域最重要的变化不是发明了新的 ACID，而是继续推动以下融合：

- 并发控制与共识合并；
- 事务执行与地理复制解耦；
- 用同步时钟预先确定事务顺序；
- 在 Aurora/Neon 式共享存储、存算分离架构上支持多区域写入；
- CRDT 从理论可用走向工程可用。

### 代表成果

#### 6. Tiga：用同步时钟在 1 个广域往返内提交事务

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 提出者 | Jinkun Geng、Shuai Mu、Anirudh Sivaraman、Balaji Prabhakar |
| 发表场景 | SOSP 2025，pp. 555–571 |
| 证据等级 | A |
| 分类 | 重要突破 / 事务与共识融合 |

Tiga 面向 Spanner 类地理复制事务数据库。核心机制是在事务提交时给它分配一个未来时间戳，使事务先于该时间戳到达相关服务器；服务器等到本地时钟越过该时间后，再按预分配顺序执行。

在多数情况下，它把严格可串行化执行与一致性复制压缩到 **1 个 wide-area round-trip time**；迟到事务走 1.5–2 WRTT 的慢路径。

重要性：Tiga 合并了并发控制和共识，使用“时间”预先确定顺序，是 Spanner TrueTime、Calvin 预定序、Tapir 融合事务/复制思路之后的重要推进。但它依赖现代时钟同步质量，不应被理解为无约束地消除了广域延迟。

链接：

- https://dl.acm.org/doi/10.1145/3731569.3764854
- https://arxiv.org/abs/2509.05759
- https://github.com/New-Consensus-Concurrency-Control/Tiga

#### 7. Mako：投机执行事务，后台地理复制

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 提出者 | Weihai Shen、Yang Cui、Siddhartha Sen、Sebastian Angel、Shuai Mu |
| 发表场景 | OSDI 2025，pp. 129–152 |
| 证据等级 | A |
| 分类 | 重要演进 |

Mako 是强一致地理复制、可水平扩展的事务 KV 存储。它将事务执行和复制解耦：事务先在本地投机执行，系统后台并行进行状态机地理复制。

关键创新是**投机式 2PC**：分布式事务不必等待决策被复制后才能继续，同时通过机制限制分片在复制完成前失败可能造成的无限级联中止。

论文报告在 Azure 上、10 个分片、每分片 24 线程时可处理 366 万 TPC-C TPS，相比面向地理复制优化的现有系统提高 8.6 倍。

重要性：它不是理论上消除复制，而是通过执行与复制解耦来隐藏广域复制延迟，代价是更复杂的中止、恢复和依赖管理。

链接：

- https://www.usenix.org/conference/osdi25/presentation/shen-weihai
- https://www.usenix.org/system/files/osdi25-shen-weihai.pdf

#### 8. SunStorm：让 Aurora 式共享存储系统支持地理分布式多写入者

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 提出者 | Cuong D. T. Nguyen、Pooja Nilangekar、Heikki Linnakangas、Daniel J. Abadi |
| 发表场景 | PVLDB 18(13) |
| 证据等级 | A |
| 分类 | 重要演进 |

Aurora 式系统通过共享存储和存算分离避免传统 shared-nothing 数据库中的许多分布式提交成本，但传统架构通常只有一个主写入者；远离主区域的写入仍要跨区域到主节点。

SunStorm 将共享存储按地理分区，使各区域拥有本地写入者：

- 单区域事务保持本地执行；
- 多区域事务只在提交时进行跨区域协调；
- 使用去中心化 2PC，在发起者和验证者之间完成一轮协调。

重要性：它把 Aurora/Neon 式存算分离架构从“单主写、多读副本”推向“地理分区、多区域写入”的设计空间。

链接：

- http://www.cs.umd.edu/~abadi/papers/SunStorm-vldb25.pdf
- https://dl.acm.org/doi/10.14778/3773731.3773733

#### 9. Automerge 3.0：CRDT 工程化成熟的重要节点

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 提出者 | Automerge 团队 |
| 证据等级 | B（官方博客/项目材料） |
| 分类 | 工程突破 / CRDT 实用化 |

Automerge 官方博客记录了 2024–2025 年的关键里程碑：

- 2024-04：2.2 Rich Text；
- 2024-08：Automerge Anywhere；
- 2025-05：Repo 2.0；
- 2025-07：Automerge 3.0，内存占用降低超过 10 倍，并保持向后兼容。

重要性：CRDT 的理论收敛性早已不是新问题，真正阻碍落地的是内存开销、同步协议、富文本、Repo/网络层、互操作和工程复杂度。Automerge 3.0 代表 CRDT 从“可用但重”走向“可在真实协作软件中大规模使用”。

链接：

- https://automerge.org/blog/

### 本方向结论

- **有实质突破**：Tiga 的时钟预排序和 Mako 的投机地理复制最具代表性。
- **没有消除一致性代价**：系统仍需在延迟、中止率、时钟依赖、复制开销和故障恢复之间取舍。
- **CRDT 进展主要是工程突破**，不是新的一致性理论。

---

## 3.3 分布式存储

### 总体判断

分布式存储在 2024–2026 年最显著的变化来自 AI 工作负载：

- 高吞吐随机读；
- checkpoint 并行写入；
- KVCache 高吞吐访问；
- 训练数据加载与小文件合并；
- RDMA/InfiniBand 全栈互联；
- 纠删码放置从 stripe 中解耦；
- CXL 内存池开始进入数据库和存储研究。

### 代表成果

#### 10. DeepSeek 3FS：面向 AI 训练与推理的开源高性能文件系统

| 字段 | 内容 |
|---|---|
| 开源时间 | GitHub 仓库创建于 2025-02-27；公开发布于 2025-02-28 前后 |
| 提出者 | DeepSeek-AI |
| 证据等级 | B（官方 GitHub/文档），A 级需要正式同行评审论文支撑 |
| 分类 | 范式级候选 / AI 存储重要突破 |

3FS（Fire-Flyer File System）是面向 AI 训练与推理工作负载的高性能分布式文件系统，利用现代 SSD 和 RDMA 网络提供共享存储层。

官方 README 描述的核心特性包括：

- **Disaggregated Architecture**：聚合大量 SSD 和存储节点网络带宽，允许应用以 locality-oblivious 方式访问存储；
- **Strong Consistency**：使用 Chain Replication with Apportioned Queries，即 CRAQ；
- **File Interfaces**：提供熟悉的文件接口，元数据服务无状态，底层由 FoundationDB 等事务 KV 支撑；
- 面向数据准备、数据加载、checkpoint、KVCache 等 AI 全流程。

公开材料中最常被引用的性能指标包括：

- 180 个存储节点、每节点 2×200Gbps InfiniBand 与 16×14TiB SSD，实现约 6.6 TiB/s 聚合读取吞吐；
- GraySort 排序 110.5 TiB 数据，8192 分区，约 30 分钟，平均 3.66 TiB/min；
- 单客户端 KVCache 峰值吞吐超过 40 GiB/s。

重要性：3FS 的关键不是单个 CRAQ 或 FoundationDB，而是将已有强一致复制、RDMA、SSD、文件接口与 AI 训练/推理负载进行端到端整合，并以开源方式填补了高性能 AI 并行文件系统长期被商业方案占据的空白。

限制：截至本报告整理时，它更应被定位为**有真实生产背景的开源系统突破**，而非已经被顶级会议完整同行评审的新理论范式。

链接：

- https://github.com/deepseek-ai/3FS
- https://github.com/deepseek-ai/3fs
- 早期相关系统论文线索：https://arxiv.org/html/2408.14158v2

#### 11. Okapi：解耦数据条带与冗余分组

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 提出者 | CMU 与 Google 联合团队 |
| 发表场景 | OSDI 2025，pp. 897–914 |
| 证据等级 | A |
| 分类 | 重要架构演进 |

传统集群文件系统通常把数据如何分布到磁盘上的 data striping，与哪些块一起纠删编码的 redundancy grouping 绑定。这样同一个 stripe width 同时承担 I/O 效率和耐久目标，导致显著不灵活。

Okapi 将两者解耦：

- striping 根据 I/O 性能和访问模式配置；
- redundancy grouping 根据可靠性和空间效率配置；
- 更换 EC 方案时可以不必重写全部数据；
- 原型系统显示元数据增长低于 1%；
- 读吞吐最高提升 80%，每秒寻道最多降低 70%，冗余转换开销最高降低 70%。

重要性：它指出“读优化布局”和“故障恢复组”不是同一个概念。这个思想在 HDD 集群、冷热分层、磁盘故障率变化和大规模转码场景中都有实际价值。

链接：

- https://www.usenix.org/conference/osdi25/presentation/athlur
- https://www.usenix.org/system/files/osdi25-athlur.pdf

#### 12. Nos/Nostor：无条带纠删码内存存储

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 提出者 | Jian Gao、Jiwu Shu、Bin Yan、Yuhao Zhang、Keji Huang |
| 发表场景 | OSDI 2025，pp. 821–838 |
| 证据等级 | A |
| 分类 | 重要演进 |

Nos 面向高速内存存储。传统纠删码以 stripe 决定数据和校验块放置，但在内存级系统中，组织 stripe、等待同一条带写入、处理边界和对齐都会产生不可忽略的协调开销。

Nos 允许每个节点独立把数据复制到其他节点，并把接收到的数据副本通过 XOR 编码为 parity；它使用 SBIBD（symmetric balanced incomplete block design）确定主副本到备份节点的亲和关系。

基于 Nos 构建的 Nostor 在评估中相比基于 stripe 的纠删码基线获得 1.61 倍和 2.60 倍吞吐。

重要性：它说明当底层介质从磁盘和 SSD 进一步走向内存池时，传统 stripe 抽象本身可能成为瓶颈。

链接：

- https://www.usenix.org/conference/osdi25/presentation/gao
- https://www.usenix.org/system/files/osdi25-gao.pdf
- https://github.com/IcicleF/Nos

### 本方向结论

- **最强实践突破是 3FS**，但应准确表述为 AI workload-driven open-source parallel file system，而非通用存储理论革命。
- **Okapi 与 Nos 是高质量学术演进**，分别从条带/冗余组解耦、消除 stripe 抽象两个方向推进。
- 传统 Ceph、Lustre、DAOS、HDFS 在该时期更多是持续优化和适配 AI，没有发现足够证据支持“新一代通用文件系统范式已统一”。

---

## 3.4 数据处理与流批一体、Lakehouse

### 总体判断

该方向的关键词是：

- Flink 2.0 的异步执行与状态后端解耦；
- Spark 4.0 的 ANSI SQL、Variant、Spark Connect 和多语言客户端；
- Iceberg v3 用 deletion vectors 替代 position delete files；
- Paimon 1.0 推动流式湖仓和 LSM lake format 成熟；
- 流处理系统继续向增量物化视图和事件驱动服务演进。

该方向更多是**生态与工程成熟**，不是新的计算范式。

### 代表成果

#### 13. Apache Flink 2.0：异步执行模型与解耦状态后端

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 证据等级 | B（官方 release notes） |
| 分类 | 重要工程演进 |

Flink 2.0 release notes 显示其关键变化包括：

- 异步执行模型；
- disaggregated state backend；
- 异步状态访问；
- Materialized Table；
- 与云存储和对象存储更紧密的集成。

重要性：Flink 的瓶颈长期不只是 SQL 或算子，而是状态访问、检查点、恢复和远程 I/O。2.0 的方向是把状态访问从同步线程模型中解出来，适配对象存储、远端状态和更高并行度。

链接：

- https://apache.googlesource.com/flink/+/master/docs/content/release-notes/flink-2.0.md

#### 14. Apache Spark 4.0：ANSI SQL 默认化与 Variant

| 字段 | 内容 |
|---|---|
| 发布时间 | 2025-05-28 |
| 证据等级 | B（Databricks 官方博客） |
| 分类 | 重要生态演进 |

Spark 4.0 的代表性更新包括：

- Spark Connect 增强；
- ANSI SQL 默认开启；
- VARIANT 类型；
- Go、Swift、Rust 客户端；
- 流处理能力增强。

重要性：Spark 正在从 JVM-centered monolithic engine 进一步转向多语言、远程连接、标准 SQL 和半结构化 AI 数据友好的运行时。它不是新的 MapReduce，但对生态兼容性和现代化很重要。

链接：

- https://www.databricks.com/blog/introducing-apache-spark-40

#### 15. Apache Iceberg v3：Deletion Vectors 与 Variant

| 字段 | 内容 |
|---|---|
| 规范落地 | 2025 年起随 Iceberg 1.8 等版本逐步落地 |
| 证据等级 | B（官方规范、发布说明） |
| 分类 | 表格式重要演进 |

Iceberg v3 增加：

- Variant 类型；
- deletion vectors；
- 默认值；
- row lineage；
- 更丰富的类型和加密相关能力。

其中最重要的是 deletion vectors。v2 中 position delete files 会在频繁行级更新场景产生大量小删除文件，读取时需要将数据文件和多个删除文件进行归并。v3 使用 bitmap 编码某个数据文件中被删除的位置，通常以 Puffin blob 存储；每个数据文件最多一个 deletion vector。

v3 禁止新增 position delete files；后续 v4 进一步禁止新的 equality deletes 和 delete manifests，并把 deletion vector metadata 放入 data file manifest entry。

重要性：它降低了 lakehouse 行级 DML 的读放大和元数据复杂度，是 Iceberg 从批式表格式走向实时更新表格式的关键一步。

需要注意：Iceberg v3 是**规范版本**，不是一个单独软件发行版；各引擎对 v3 的读写支持是逐步成熟的。

链接：

- https://iceberg.apache.org/spec
- https://opensource.googleblog.com/2025/08/whats-new-in-iceberg-v3.html

#### 16. Apache Paimon 1.0：流式湖仓成熟节点

| 字段 | 内容 |
|---|---|
| 1.0.0 归档时间 | 2024-12-31 / 官方发行材料显示 2025-01-03 发布 |
| 证据等级 | B（Apache release catalog、董事会记录、官方文档） |
| 分类 | 工程成熟 / 流式湖仓重要节点 |

Paimon 是 lake storage format 与 LSM 结构结合的流式湖仓系统，支持：

- 流式更新；
- 主键表；
- LSM merge；
- 流批读写；
- CDC 摄取；
- 与 Flink、Spark 等引擎集成。

Apache 软件基金会董事会记录显示：

- Paimon 1.0.0 于 2025-01-03 发布；
- 1.0.1 于 2025-02-10 发布。

重要性：它不是简单的 Iceberg 替代品，而是把 LSM、changelog、主键更新和流式消费放到湖仓底层，降低实时链路从 Kafka/专用存储再同步到离线表的复杂度。

链接：

- https://release-catalog.apache.org/paimon/index.html
- https://www.apache.org/foundation/board/minutes/Paimon.html
- https://paimon.apache.org/docs/1.0

### 本方向结论

- **有重要工程突破，但无新计算模型**。
- Lakehouse 的竞争焦点从“能不能在对象存储上做表”转向“高频更新、删除向量、状态、CDC、流批一致性和多引擎兼容”。
- 厂商宣传中的“统一批流、实时离线一体”在架构上可行，但具体一致性、延迟和运维成本仍需逐项验证。

---

## 3.5 系统可靠性、可观测性与运维

### 总体判断

该方向的主要进展不是“LLM 自动运维”被证明可靠，而是：

- 形式化验证从协议走向生产系统组件；
- 生产系统静默故障检测开始利用从测试推导的语义检查器；
- sidecarless service mesh 走向 GA；
- OpenTelemetry 继续统一信号，但 profiling signal 仍未完全成熟；
- eBPF、用户态运行时和高频遥测扩大落地。

### 代表成果

#### 17. Verus：面向真实系统的实用化验证

| 字段 | 内容 |
|---|---|
| 年份 | 2024 |
| 发表场景 | SOSP 2024 |
| 证据等级 | A |
| 分类 | 可靠性方法重要突破 |

Verus 用 Rust 风格语言和 SMT 支持系统代码验证。SOSP 2024 论文展示其验证了多个复杂系统组件，包括：

- OS page table；
- 并发内存分配器；
- 面向生产存储的 persistent circular log；
- distributed lock 等。

重要性：它证明形式化验证不只适用于极小算法，而可以进入真实系统中容易造成严重后果的核心模块。

链接：

- https://verus-lang.github.io/paper-sosp24-artifact/assets/paper-20240921-162720-b7db935.pdf

#### 18. CCF “smart casual verification”：验证生产级分布式账本协议

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 发表场景 | NSDI 2025 |
| 证据等级 | A |
| 分类 | 生产协议验证 |

Microsoft Confidential Consortium Framework（CCF）支撑 Azure Confidential Ledger。NSDI 2025 工作介绍了对 CCF 独特分布式共识协议和客户端一致性模型的“smart casual verification”：

- 用 TLA+ 编写形式规范；
- 结合 model checking；
- 将形式规范与 C++ 实现通过自动化测试绑定。

重要性：这是一种介于完整形式化证明和普通测试之间的折中路径，更适合生产团队长期维护。

链接：

- https://www.usenix.org/conference/nsdi25/technical-sessions

#### 19. Istio Ambient Mode GA：Sidecarless Service Mesh 生产化

| 字段 | 内容 |
|---|---|
| GA 时间 | Istio 1.24，2024-11-07 发布 |
| 证据等级 | B（官方发布说明） |
| 分类 | 工程成熟 |

Istio 1.24 官方发布说明确认 Ambient mode generally available。核心组件：

- 节点级 ztunnel；
- namespace/工作负载级 waypoint；
- Ambient API 被标记为 Stable。

重要性：服务网格从每个 Pod 注入 sidecar 的模型，转向按需、分层、sidecarless/sidecar-optional 的数据平面。这样可降低部分基础设施开销，并把 L4 与 L7 处理分离。

限制：Ambient GA 不等于所有场景都优于 sidecar；策略复杂、L7 功能、迁移成本和性能表现仍需按场景评估。

链接：

- https://istio.io/latest/news/releases/1.24.x/announcing-1.24/
- https://istio.io/latest/blog/2024/ambient-reaches-ga/

### OpenTelemetry 状态

OpenTelemetry 官方 2024 年材料显示：

- tracing、metrics、logs、resources 等核心能力已在不同程度上稳定或广泛生产使用；
- profiling signal 当时仍在开发中，不建议作为稳定生产标准；
- OTLP profiles 在相关版本中仍不稳定；
- Elastic Continuous Profiling Agent 被捐赠为 eBPF profiling agent。

结论：OpenTelemetry 是可观测性事实标准之一，但“四大信号完全统一成熟”的说法在该时间点仍需谨慎。

链接：

- https://opentelemetry.io/blog/2024/state-profiling

### 本方向结论

- **真实突破在验证和生产协议核验**，而非 AIOps 概念。
- Sidecarless 服务网格已进入生产可用阶段，但不是无成本替代。
- 自动化根因分析、LLM 运维、智能混沌工程缺少足够一手证据证明其已成为可靠基础范式。

---

## 3.6 资源管理、调度与新型硬件互连

### 总体判断

该方向最值得关注的是三类变化：

1. CXL 从硬件互连概念进入数据库系统设计；
2. 用户态调度利用 Intel user interrupt 等能力达到微秒级抢占；
3. RDMA over Ethernet 与 Ultra Ethernet 正在为 AI 训练构建新一代网络生态。

### 代表成果

#### 20. Tigon：为 CXL Pod 设计的分布式数据库

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 提出者 | Yibo Huang、Haowei Chen、Newton Ni、Yan Sun、Vijay Chidambaram、Dixin Tang、Emmett Witchel |
| 发表场景 | OSDI 2025，pp. 109–128 |
| 证据等级 | A |
| 分类 | 范式级候选 / CXL 数据库探索 |

Tigon 是较早明确面向 CXL pod 设计的分布式内存数据库。它不把 CXL 仅当作更快网络，而是利用共享 CXL memory 上的原子操作同步跨主机并发数据访问。

CXL pod 介于共享内存多处理器和传统分布式系统之间：

- 每个主机有本地 DRAM；
- 多主机连接共享 CXL memory；
- 跨主机硬件缓存一致性支持有限且昂贵；
- CXL memory 延迟高于本地 DRAM、带宽低于本地内存。

Tigon 采用 Pasha 架构，数据初始分区在各主机上，并在本地 DRAM 与 CXL memory 间动态移动。论文报告：

- TPC-C/YCSB 场景中，相比两个把 CXL 当传输层的优化 shared-nothing 数据库，吞吐最高提升 2.5 倍；
- 相比基于 RDMA 的分布式数据库，吞吐最高提升 18.5 倍。

重要性：它展示了一种不同于传统 shared-nothing + 2PC、也不同于完全共享内存机器的 CXL pod 数据库设计。

限制：作者也指出 CXL 硬件、跨主机一致性和商业可用性仍受限；当前主要是研究原型和硬件过渡期探索。

链接：

- https://www.usenix.org/conference/osdi25/presentation/huang-yibo
- https://yibo-huang.github.io/papers/tigon_osdi25.pdf
- https://github.com/ut-datasys/tigon

#### 21. Skyloft：用户态微秒级调度框架

| 字段 | 内容 |
|---|---|
| 年份 | 2024 |
| 提出者 | Yuekai Jia、Kaifu Tian、Yuyang You、Yu Chen、Kang Chen |
| 发表场景 | SOSP 2024，pp. 265–279 |
| 证据等级 | A |
| 分类 | 调度系统重要演进 |

Skyloft 利用 Intel user-mode interrupt 在用户空间直接投递和处理硬件定时器，实现微秒级抢占。它提供可支持抢占式与非抢占式策略的调度接口，并与 Linux、DPDK 等高性能 I/O 框架兼容。

论文报告：

- 用户态定时器显著降低 wake-up latency；
- 相比 ghOSt，Latency Critical 应用最大吞吐提高约 1.2 倍；
- RocksDB 场景中结合 5μs 抢占和 work-stealing，相比 Shenango 性能提高 1.9 倍。

重要性：它把微秒级调度从专用运行时推广为更通用的用户态框架。

链接：

- https://dl.acm.org/doi/10.1145/3694715.3695973
- https://madsys.cs.tsinghua.edu.cn/publication/skyloft-a-general-high-efficient-scheduling-framework-in-user-space/SOSP24-Jia.pdf

#### 22. Meta：AI 训练规模的 Ethernet RDMA

| 字段 | 内容 |
|---|---|
| 年份 | 2024 |
| 发表场景 | SIGCOMM 2024 |
| 证据等级 | A |
| 分类 | AI 网络重要生产实践 |

Meta 在 SIGCOMM 2024 发表面向分布式 AI 训练的 RDMA over Ethernet 实践。论文讨论了在 400G 以太网部署中支撑大规模训练的网络机制。

一个值得注意的经验是：在超过一年的部署中，其网络主要依赖 PFC，而未使用 DCQCN，更多依靠上层 collective-library control 来处理流量调度和拥塞协同。

重要性：它挑战了“AI 训练网络必须依赖 InfiniBand 完整生态”的单一路线，表明以太网 RDMA 在严格工程控制和上层协同下可以支撑超大规模训练。

链接：

- https://cs.stanford.edu/~keithw/sigcomm2024/sigcomm24-final246-acmpaginated.pdf
- https://doi.org/10.1145/3651890.3672233

#### 23. Ultra Ethernet Consortium：面向 AI 的新一代以太网生态

| 字段 | 内容 |
|---|---|
| 时间窗 | 2024–2026 规范推进 |
| 证据等级 | B/C（规范组织材料） |
| 分类 | 标准与生态演进 |

UEC 的目标是为 AI/HPC 工作负载构建现代化以太网，包括改进传输、拥塞控制、遥测、多路径、可扩展 fabric 和端到端协同。

公开规范历史显示：

- UEC 1.0.3 于 2026-07-16 发布；
- UEC 1.1 于 2026-10-12 发布。

但 UEC 1.0 初始发布日期、真实芯片/交换机/生产集群部署范围，仍需要进一步核查。

重要性：如果 UEC 被网卡、交换机、GPU 平台和云厂商广泛采用，它可能成为 AI 数据中心网络从 InfiniBand 主导走向开放以太网生态的重要基础。但截至本报告，不能仅凭规范发布推断大规模生产成功。

链接：

- https://ultraethernet.org/specification-history/

#### 24. Modal：基于 gVisor/runsc 内存快照的亚秒级 Serverless 冷启动

| 字段 | 内容 |
|---|---|
| 年份 | 2025 |
| 证据等级 | B（Modal 官方工程博客） |
| 分类 | 工程优化 / Serverless 冷启动突破 |

Modal 工程博客介绍了基于 gVisor/runsc 的 memory snapshot 与 checkpoint-restore 技术，使函数可以利用预构建内存状态实现亚秒级冷启动，并与 lazy loading 路线进行比较。

重要性：Serverless 的核心痛点之一是冷启动和运行时初始化。内存快照不是全新思想，但在 gVisor、微 VM、语言运行时和云平台调度结合后，变得更加实用。

限制：这是典型工程突破，不是资源调度理论范式变化。

链接：

- https://modal.com/blog/mem-snapshots

### 本方向结论

- **CXL 是最值得跟踪但也最容易被夸大的方向**。Tigon 证明其数据库价值，但距离通用生产普及仍早。
- **用户态微秒级调度已在高性能系统中成熟**，但主要影响低延迟基础设施，而非所有普通业务。
- **AI Ethernet/UEC 是网络生态级机会**，但实际落地仍取决于硬件、拥塞控制和上层集合通信协同。

---

## 4. 10–15 个最具代表性成果精选

若从全部候选中压缩为 12 个最值得记录的成果，建议如下。

| 排名 | 成果 | 年份 | 方向 | 等级 | 分类 |
|---|---|---:|---|---|---|
| 1 | DeepSeek 3FS | 2025 | 分布式存储 | B | AI 并行文件系统重要突破 |
| 2 | LazyLog | 2024 | 共识/日志 | A | 共享日志语义突破 |
| 3 | Tiga | 2025 | 事务/一致性 | A | 时钟预排序，一往返提交 |
| 4 | Mako | 2025 | 事务/地理复制 | A | 投机事务与后台复制 |
| 5 | Tigon | 2025 | CXL/数据库 | A | CXL pod 数据库范式候选 |
| 6 | Okapi | 2025 | 存储/纠删码 | A | 条带与冗余组解耦 |
| 7 | Nos/Nostor | 2025 | 内存存储 | A | 无条带纠删码 |
| 8 | Autobahn | 2024 | BFT 共识 | A | 高吞吐低延迟 BFT |
| 9 | Basilisk | 2025 | 协议验证 | A | 自动不变式证明 |
| 10 | Meta Ethernet RDMA | 2024 | AI 网络 | A | 以太网 RDMA 生产实践 |
| 11 | Flink 2.0 | 2025 | 流处理 | B | 异步执行与状态解耦 |
| 12 | Iceberg v3 | 2025 | Lakehouse | B | deletion vectors 与 Variant |

若必须进一步压缩为“真正可能具有范式意义”的 4 个：

1. **3FS**：AI workload 驱动的开源高性能并行文件系统；
2. **LazyLog**：重新安排共享日志中全局顺序的绑定时机；
3. **Tiga/Tigon 所代表的时间/CXL 协调路线**：把时间和共享 CXL memory 用作更低成本协调媒介；
4. **AI 时代 Ethernet RDMA/UEC 生态**：可能改变 AI 数据中心网络的开放标准路线。

---

## 5. 真突破与“旧问题新包装”的区分

### 5.1 可认为有真实突破或重要演进的方向

#### AI 驱动的存储系统

3FS 的突破点不是“又一个分布式文件系统”，而是围绕 AI 训练和推理的具体瓶颈进行端到端重设计：

- 不追求传统小文件优化，而通过 FFRecord 合并样本；
- 强调高吞吐随机读；
- 强化 KVCache 和 checkpoint；
- 全组件 RDMA；
- 文件接口保持通用；
- 元数据建立在事务 KV 上。

这属于新负载驱动的系统设计，而非单纯换名。

#### 时钟作为事务协调原语

Tiga、Mako 等工作延续 Spanner/Calvin/TAPIR 路线，但把“未来时间戳”“后台复制”“事务与共识合并”推到更实用的位置。它们没有消除物理延迟，但显著减少常见路径中的协调轮次。

#### CXL pod 上的数据库

Tigon 的独特之处是利用 CXL pod 这种介于共享内存和分布式系统之间的结构。它不是简单把网络换成 CXL，而是重新设计数据放置、原子操作和缓存一致性假设。

#### 微秒级用户态运行时与调度

Skyloft、Modal snapshot、gVisor checkpoint 等属于工程机制成熟。它们虽然不是基础理论革命，但对尾延迟、冷启动和高密度多租户有直接影响。

### 5.2 应谨慎对待的叙事

#### “新共识彻底替代 Paxos/Raft”

未发现足够证据。多数新协议仍是：

- BFT 场景；
- DAG 传播；
- 特定硬件假设；
- 时钟假设；
- 或已有 Paxos/Raft 思想的模块化重排。

通用生产系统仍主要依赖经过长期验证的 Paxos/Raft 谱系。

#### “零成本强一致”

不可能无代价。系统可以：

- 把成本移出关键路径；
- 用后台复制承担；
- 用时钟预排序减少冲突；
- 用缓存和日志降低延迟；
- 用更强硬件降低通信成本。

但不能同时无延迟、无中止、无复制、无依赖且在所有故障模型下保证强一致。

#### “AI Native 数据库革命”

许多说法只是把向量类型、RAG、JSON/Variant、物化视图或流触发器重新包装。真正值得认可的部分是：

- Variant；
- vector index；
- event-driven materialized views；
- KVCache 存储；
- 训练/推理负载感知调度。

不能仅凭产品页上的“AI Native”认定为底层突破。

#### “AIOps 已实现自动根因分析”

目前更可靠的进展是：

- 更好的 traces/logs/metrics；
- eBPF 持续剖析；
- 遥测关联；
- 故障注入；
- 形式化检查；
- LLM 辅助总结和检索。

自动闭环决策在高风险生产系统中仍远未成为可信基础范式。

---

## 6. 证据缺口清单

以下事项没有找到足够一手来源，不能做确定性结论。

### 6.1 共识协议

- 2024–2026 年是否存在可在通用 CFT 场景中全面替代 Raft/Paxos 的新协议；
- Raft leader offload、负载分摊、leaderless SMR 在工业界的真实部署规模；
- Compartmentalized Paxos 之后是否形成新的公认生产框架；
- Ladon 目前主要检索到 EuroSys 2025 幻灯片，缺少正式论文 PDF 与完整 artifact 评估。

### 6.2 事务与一致性

- FoundationDB 7.3/7.4 是否存在范式级事务新特性，目前只证实短事务、冲突范围、read-your-writes 等既有模型和维护性变更；
- Yjs 在 2024–2026 的关键版本里程碑尚未完整核查；
- ElectricSQL、Riak 等系统在该时段是否有底层一致性突破；
- 多区域事务在云厂商商业产品中的真实一致性语义与内部实现差异。

### 6.3 分布式存储

- DAOS、Ceph、Lustre、VAST、Weka 在该时期是否存在可被定义为范式突破的新机制；
- CXL memory pooling 的真实生产部署范围；
- 静默数据损坏检测是否出现跨系统的新一代通用方案；
- 3FS 是否有完整、正式发表并通过同行评审的系统论文。

### 6.4 数据处理

- Delta Lake、Apache Hudi、Paimon、Iceberg 各版本规范和关键特性的精确时间线；
- Materialize、RisingWave 的增量维护模型是否出现底层新突破；
- Ray 在分布式调度和 shuffle 上是否形成新的基础范式；
- 新一代向量化执行引擎相对既有 Volcano/Vectorized model 的实际创新边界。

### 6.5 可靠性与运维

- Homa 在生产环境中的广泛部署证据仍不足；
- BBRv3 的正式标准化状态和 Linux 主线集成状态需要继续核查；
- UEC 1.0 的初始发布日期与商业部署情况仍需补证；
- LLM 驱动自动故障修复、自动变更决策缺少高可信生产证据。

### 6.6 资源与调度

- Kubernetes 之外是否出现被广泛采纳的新一代通用分布式调度器；
- CXL 内存池如何与现有 Kubernetes/数据库调度器集成，仍缺少成熟事实标准；
- 异构 GPU/NPU/DPU 调度是否已形成跨厂商统一抽象。

---

## 7. 最终判断

2024–2026 年分布式底层技术的真实图景可以概括为：

> **基础理论少有颠覆性突破，但 AI 工作负载、新型硬件和微秒级工程栈正在推动系统接口重新分层。**

最值得重视的不是“某个协议彻底解决一致性”，而是：

- 存储围绕 AI 训练和推理重构；
- 事务通过时钟、日志、DAG 和投机执行减少协调轮次；
- CXL 在本地内存和分布式网络之间创造新的 pod 级结构；
- RDMA over Ethernet/UEC 可能形成开放 AI fabric；
- 形式化验证和自动不变式推导开始进入复杂生产协议；
- Lakehouse 与流处理通过 deletion vectors、状态解耦、LSM 和物化视图继续收敛。

因此，若要为这一时期贴一个准确标签，不应是“分布式系统范式革命”，而更适合称为：

> **AI 与硬件驱动的分布式系统再设计阶段。**
