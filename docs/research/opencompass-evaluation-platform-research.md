# OpenCompass（司南）大模型测评框架 —— 桌面调研

调研日期：2026-11-25
目的：为介绍/选型文章提供一手证据。纯桌面调研，未运行代码。

证据级别约定：
- **[A]** 官方来源：GitHub 仓库/官方文档/官网/上海人工智能实验室（上海AI实验室，下称 Shanghai AI Lab）官方新闻
- **[B]** 权威二手：学术论文、权威媒体、信通院报告
- **[C]** 二手聚合：CSDN、商城商品页、工具导航站等（仅作旁证，不作为关键事实唯一来源）

---

## 一、基本信息

### 1.1 它是什么

OpenCompass（中文名"司南"）是**上海人工智能实验室**推出的、面向大语言模型与多模态大模型的**一站式开源评测平台**。提供从数据集/模型配置、推理、评测到结果汇总的完整工作流，支持本地开源模型与 API 模型，支持 HuggingFace、LMDeploy、vLLM 等推理后端，支持零样本、少样本、思维链、规则评测与 LLM Judge 等多种评测方式。[A]

- 主仓库：https://github.com/open-compass/opencompass
- 官网：https://opencompass.org.cn/
- 文档：https://opencompass.readthedocs.io/
- 榜单 CompassRank：https://rank.opencompass.org.cn/home
- 基准社区 CompassHub：https://hub.opencompass.org.cn/home
- 安装：`pip install -U opencompass`（可选 `[full]`/`[lmdeploy]`/`[vllm]`/`[api]`/`[vlm]` extras）；支持 Python 3.12，代码执行类数据集（APPS/TACO/LiveCodeBench Code Generation）因 `pyext==0.7` 依赖需用 Python 3.10。[A]

### 1.2 仓库关键数字（GitHub API 实测，2026-11-25 抓取）[A]

| 项目 | 值 |
|---|---|
| Star | 7,639 |
| Fork | 921 |
| Open issues | 332 |
| 许可证 | Apache-2.0 |
| 主语言 | Python |
| 仓库创建时间 | 2023-06-15 |
| 最近 push | 2026-11-24 |
| 最新 release | v0.5.5（2026-11-06 发布，据此前抓取的 Releases 页） |
| 组织 | open-compass（GitHub Organization） |

官方口径"集成超过 100 个数据集"[A]；实测 `opencompass/configs/datasets/` 目录下约 **250+ 个数据集配置目录**（含同一基准的多配置与大量垂直领域基准，2026-11-25 GitHub API 列举）[A]。

### 1.3 关键时间线 [A]（除标注外）

- **2023-06-15**：GitHub 仓库创建（索引时间，早于官宣；具体首次对外宣布日期未见专门公告，**首次开源的确切官宣日期未查到**）。
- **2023-10-19**：上海AI实验室正式对外推出 OpenCompass 开放评测体系；当时口径为 5 大能力维度（知识/语言/理解/推理/考试）、70+ 数据集、40 万+ 题目、支持 70+ 开源模型，宣称千亿参数模型最快 3 小时完成评测。来源：上海AI实验室新闻 https://www.shlab.org.cn/news/5443498
- **2023-12-22**：开源 T-Eval（工具调用分步评测，后被 ACL 2024 收录），仓库 https://github.com/open-compass/T-Eval （313 stars，Apache-2.0，2026-11-25 实测）[A]
- **2024-01-30**：发布 **OpenCompass 2.0**"铁三角"：CompassRank（榜单）+ CompassHub（基准社区）+ CompassKit（工具链，含 OpenCompass、VLMEvalKit、Code-Evaluator、MixtralKit）；公布 2023 年度榜单（GPT-4 Turbo 第一，GLM-4、Qwen-Max、文心一言 4.0 紧随）。截至当时榜单涉及 150+ 模型，30 余家企业/机构（含 Meta、阿里、腾讯、百度）采用。来源：https://www.shlab.org.cn/news/5443858
- **2024 年内**：被 **Meta AI Llama 官方入门文档**列为大模型标准测试（validation）工具之一，OpenCompass README 长期置顶该背书，链接 https://ai.meta.com/llama/get-started/#validation [A]
- **2024-07**：VLMEvalKit 技术报告发布（arXiv:2407.11691，后发表于 ACM MM 2024 Demo，pages 11198–11201）[B]
- **2024-12-13**：联合上海财经大学等发布《金融大模型应用评测报告（2024）》，14 家机构 20 个模型参评（财联社报道）[B]
- **2026-05~06**：OpenCompass 主技术报告《OpenCompass: A Universal Evaluation Platform for Large Language Models》挂出（arXiv:2605.19276，2026-05-19 提交、v3 2026-06-08）[B]
- **2026-09-30**：发布 **AgentCompass**（智能体评测基础设施）并上架 PyPI v1.0.0；技术报告 arXiv:2607.13705（2026-07-13 挂出，曾上 Hugging Face Daily Papers），被 **EMNLP 2026 Demo Track** 接收（pages 253–263）[A][B]
- **2026-11-06**：主仓 release v0.5.5（据 Releases 页此前抓取）[A]

> 时效提示：本笔记中 arXiv 编号、2026 年的模型名（如 Qwen3.7-Plus、GPT-5.5、Gemini-3.1-Pro、Kimi-K2.6 等）均为 2026-11-25 从官方页面原样抓取，引用时应保留抓取日期，勿向更早时间外推。

---

## 二、架构与运行机制

### 2.1 总体架构：配置驱动的流水线

OpenCompass 核心是一条**配置驱动（Python 配置文件）的离线评测流水线**：

1. **模型配置**：`opencompass/configs/models/`，区分开源权重模型（HF/LMDeploy/vLLM 后端）与 API 模型。
2. **数据集配置**：`opencompass/configs/datasets/<数据集名>/`，文件名编码评测方式，如 `gen`（生成式）、`ppl`（困惑度判别式）、`rawprompt`、few-shot 数与配置哈希；每个配置包含 `reader_cfg`（数据读取）、`infer_cfg`（推理提示/推理器）、`eval_cfg`（评测器）三段。官方提供 `python tools/list_configs.py mmlu gsm8k` 命令检索配置。[A，官方文档"数据集选择与配置"]
3. **推理（Inferencer）**：任务切分、并发、分布式执行；2026-03-20 新增跨任务并发推理（`openicl_infer_concurrent.py`）与评测监听（`openicl_eval_watch.py`）+ 心跳机制，推理完成即触发评测。[A，README What's New]
4. **评测（Evaluator）**：规则精确匹配、LLM Judge 等。2025-02-15 新增 `GenericLLMEvaluator`（通用 LLM 评判器）与 `MATHVerifyEvaluator`（数学等价判定）；2025-04-01 新增 `CascadeEvaluator`（多个评估器级联编排）。[A]
5. **结果汇总**：CSV/HTML 报告、按维度聚合。

部分代码修改自 OpenICL、chain-of-thought-hub、instruct-eval（README 致谢）。[A]

### 2.2 评测范式

- **客观评测**：判别式（PPL/LogProb 选择题）与生成式（自由生成 + 精确匹配/规则/LLM 答案抽取）。
- **主观评测**：模型对战（pairwise）、LLM Judge、人类标注；配置目录中有 `subjective/`、`judge/`、`hardmtbench`（硬任务 MT-Bench）等。[A，目录实测]
- **提示词范式**：零样本、少样本（README 早期版本称支持 7 种上下文样例提取方案）、思维链；与模型绑定的 Meta Template；2026-03-17 新增 `RawPromptTemplate`，可透传 benchmark 原始 prompt 与结构化对话，避免格式被意外转换。[A]
- **防污染**：目录中有独立 `contamination/` 与 `gsm8k_contamination` 配置；2.0 起官方宣传具备数据污染检查能力。[A]

### 2.3 模型生态（README 2026-09 版口径）[A]

- 开源模型配置：DeepSeek、Gemma、GLM、Intern-S（Intern-S1/S1-Pro）、Kimi、Llama、MiniMax、Mistral、Qwen3 等。
- API 模型：OpenAI（含 2026-07 新增的 Responses API）、Anthropic Claude（最新 SDK）、Google Gemini（最新 SDK）、ByteDance 豆包、xAI Grok，并新增 **LiteLLM AI Gateway**（一家网关对接多家）。
- 推理模型支持：2025-02-28 官方配置支持 DeepSeek-R1 系列（流式 reasoning 配置）。

### 2.4 "铁三角"生态（2024-01-30 确立，后续演进）[A]

| 组件 | 定位 | 现状 |
|---|---|---|
| **CompassKit** | 评测工具链集合 | 核心 opencompass 仓 + VLMEvalKit + code-evaluator（https://github.com/open-compass/code-evaluator ，29 stars，Docker 化多语言代码执行沙箱，2026-11-25 实测）等 |
| **CompassHub** | 高质量评测基准**导航/社区**站 | hub.opencompass.org.cn，用户可提交自定义基准；偏"导航 + 投稿"，并非传统意义的数据集托管市场（"数据集市场"式交易机制**未查到**） |
| **CompassRank** | 榜单平台 | 见第四节 |

2026 年生态进一步分化为：**文本/通用能力 → OpenCompass 主仓；多模态 → VLMEvalKit；智能体 → AgentCompass**。2026-08-25 起 VLMEvalKit 被集成进主仓，可在统一流程中加载多模态数据集、走 OpenAI 兼容接口推理并使用 VLMEvalKit 官方评测逻辑（MMBench/MMMU-Pro 示例）。[A]

---

## 三、测评维度与数据集（重点）

### 3.1 能力维度框架的演进

- **2023-10（1.0 官宣）**：知识、语言、理解、推理、考试 5 大维度，70+ 数据集、40 万+ 题目，特色含长文本、安全、代码。[A]
- **2024-01（2.0）**：中英文双语基准，覆盖语言与理解、常识与逻辑推理、数学计算与应用、多编程语言代码、智能体、创作与对话；方法上首次系统化主观对战评测（500+ 高质量中文问题）。[A]
- **2024-04 媒体口径**：基础能力 + 综合能力两个层级，**12 个一级能力维度、50 余个二级维度**（语言、知识、理解、数学、代码、长文本、智能体等）。来源：界面新闻专访 https://www.jiemian.com/article/11100699.html [B]
- **2026 年 README 口径**：知识、推理、代码、科学、语言、长上下文、安全（已不含"考试"字样，科学与安全被显式列出）。[A]

### 3.2 实测数据集清单（按能力归类，2026-11-25 主仓 `configs/datasets/` 目录）[A]

下表只收录目录中**确实存在**的配置（名称为目录原样名），用于核实"它到底能测什么"。

**知识 / 综合考试**
- 英文/国际：`mmlu`、`mmlu_pro`、`mmlu_cf`、`mmmlu`、`mmmlu_lite`、`MMLUArabic`、`cmmlu`、`ceval`、`agieval`、`GaokaoBench`、`gaokao_math`、`xiezhi`（獬豸）、`kaoshi`、`supergpqa`（SuperGPQA，2025-03-11 接入，官方称覆盖 285 个研究生学科）、`gpqa`、`SimpleQA`、`chinese_simpleqa`、`ARC_c`、`ARC_e`、`openbookqa`（obqa）、`hellaswag`、`winogrande`、`piqa`、`siqa`、`commonsenseqa`、`commonsenseqa_cn`、`nq`/`nq_cn`、`triviaqa`、`narrativeqa`、`qasper`、`drop`、`race`、`musr`、`strategyqa`、`bbh`（BIG-Bench Hard）、`bbh` 之外还有 `bbh` 扩展 `bbeh`。
- 中国语言类：CLUE/FewCLUE/SuperGLUE 全家桶（`CLUE_cmnli`、`CLUE_afqmc`、`CLUE_C3`、`CLUE_CMRC`、`CLUE_DRCD`、`FewCLUE_*`、`SuperGLUE_*`）、`XCOPA、`LogiQA`、`lambada`、`wikitext` 等。

**数学与推理**
- 经典：`gsm8k`、`gsm_hard`、`math`、`SVAMP`、`mgsm`、`game24`、`TheoremQA`、`NPHardEval`、`ZebraLogic`、`anli`。
- 竞赛/高难（2025-2026 新接入明显加密）：`aime2024`、`aime2025`、`aime2026`、`hmmt2026`、`mastermath2024v1`、`BeyondAIME`、`OlymMATH`、`OlympiadBench`、`korbench`、`reasonzoo`、`ProcessBench`（过程奖励评测）、`cruxeval`、`livemathbench`、`math401`、`omni_math`、`MathBench`、`livereasonbench`、`R_Bench`、`mrcr`。

**代码**
- `humaneval`、`humaneval_cn`、`humaneval_multi`、`humanevalx`、`humaneval_plus`、`humaneval_pro`、`mbpp`/`mbpp_cn`/`mbpp_plus`/`mbpp_pro`、`apps`、`taco`、`ds1000`（数据科学代码）、`bigcodebench`、`livecodebench`、`livecodebench_pro`、`multipl_e`、`scicode`、`codecompass`、`ojbench`、`py150`、`lcbench`、`leetcode` 类 `LCBench`。
- 注：**主仓 configs 中没有 SWE-bench 目录**（2026-11-25 实测）；智能体式软件工程任务（SWE-bench Verified/Pro/Pro Verified/Multilingual/Atlas、Terminal-Bench、GAIA、BrowseComp 等）全部收敛到独立的 **AgentCompass**（见 3.5）。

**长上下文**
- `longbench`、`longbenchv2`、`infinitebench`（∞Bench）、`ruler`（RULER）、`needlebench`、`needlebench_v2`（大海捞针，2024-01 已支持多针版本）、`babilong`、`leval`、`lveval`、`lgbench150`、`QuALITY`、`qaspercut`、`summscreen`、`llm_compression`。

**科学 / 垂直领域**
- 通用科学：`scibench`、`ScienceQA`、`SciEval`、`SciReasoner`/`scireasoner1_5`（2025-12-08 接入 SciReasoner）、`Intern-S1-Pro` 科学基准（2026-02-05）、`livestembench`、`SciKnowEval`、`s3eval`、`SciDocBench`、`srbench`、`PHYBench`、`PHYSICS`、`GrandPhysics`、`CMPhysBench`、`matbench`、`Earth_Silver`（地球科学）、`ClimaQA`、`maritimebench`（海事）、`molculariq`、`MolInstructions_chem`、`ChemBench`、`ProteinLMBench`。
- 医学/生命：`MedBench`、`MedQA`、`medmcqa`、`PubMedQA`、`Medbullets`、`MedCalc_Bench`、`MedXpertQA`、`nejm_ai_benchmark`、`CARDBiomedBench`、`ClinicBench`、`HealthBench`、`biodata`、`CIBench`、`MicroVQA`（显微镜 VQA；CIBench 具体领域**未核实**，以官方配置说明为准）。
- 金融：`BuySideFinBench`、`FinanceIQ`、`OpenFinData`，以及独立站点 **CNFinBench**（https://cnfinbench.opencompass.org.cn/ ，5 个公开 + 11 个自建数据集、约 5.4 万条，五大维度；金融 Agent 评测 4 大维度 12 项长程任务；子项含 Fin_Basics、Fin_Cert_Exams、Fin_Info_Extract、Credit_Score、Quant_Invest；论文 ACL Anthology pages 8765–8776 (2026)；联系邮箱 cnfinbench@pjlab.org.cn）[A]。
- 法律：`lawbench`（与南京大学共建，记忆/理解/应用 3 认知维度 20 任务）、`PromptCBLUE`（医疗 prompt）、`govrepcrs`（政务）、`PJExam`。
- 其他行业：`CritPt`、`General365`、`weddoo_bench`、`RebuttalBench`（学术审稿）、`CHARM`、`CLBench`、`PMMEval`、`qabench`、`rpcbench`、`s2_tomg_bench`。

**智能体 / 工具调用 / 多轮**
- 主仓：`teval`（T-Eval，ACL 2024）、`MultiIF`（Multi-IF 多轮指令遵循，2026-07-27 接入，`GenInferencer` 同步支持多轮推理）、`internsandbox`、`aa_lcr`、`aa_omniscience`、`anthropics_evals`（含 Anthropic 官方若干 eval）、`advancedIF`、`IFEval`、`IFBench`、`InverseIFEval`。
- 智能体重度任务统一走 AgentCompass（见 3.5）。

**安全 / 对齐 / 可信**
- `safety`（安全合集）、`cvalues`（中文价值观）、`realtoxicprompts`、`jigsawmultilingual`、`civilcomments`、`crowspairs`/`crowspairs_cn`（偏见）、`truthfulqa`、`Ethics`、`adv_glue`（对抗）、`fake_alignment`（伪对齐）、`perspectivegap`、`promptbench`（鲁棒性/对抗 prompt）、`chhallu_src`（幻觉）、`dingo`、`SFE`。
- 早期资料提到的 **TurboSquirrel** 安全集在当前主仓目录中**未直接见到同名目录**，可能归入 `safety` 合集，**未核实**。

**多模态（主仓侧为集成入口，本体在 VLMEvalKit）**
- 主仓配置：`MMBench`、`MMMU_Pro`、`ChartQAPro`、`SeedBench`、`MicroVQA` 等；2026-08-25 起经 VLMEvalKit 原生加载。[A]

**主观 / 创作 / 翻译 / 摘要**
- `subjective`、`hardmtbench`、`rolebench`、`wmt19`、`flores`、`iwslt2017`、`tydiqa`、`Xsum`、`XLSum`、`lcsts`、`summedits`、`SumSurvey`、`storycloze`、`calm`、`fever` 类等。

### 3.3 官方自测基准 CompassBench 等

- 目录中存在 `compassbench_v1_3`、`compassbench_20_v1_1`、`compassbench_20_v1_1_public`、`collections`（官方维度合集配置）。CompassBench 是 OpenCompass 团队自建的中英双语综合主观/开放任务基准（2.0 时期发布），具体题目构成以配置文件为准。[A]

### 3.4 VLMEvalKit（多模态评测子系统）

仓库 https://github.com/open-compass/VLMEvalKit ，pip 包名 `vlmeval`，Apache-2.0，报告 arXiv:2407.11691（ACM MM 2024）。[A][B]

- 方法论：**全部采用 generation-based 评测**（不让模型打 logprob），同时给"精确匹配"与"LLM 答案抽取"两套分数；统一 prompt 模板，因此官方明确说明**不保证复现第三方基准论文里 PPL 式评测的原始数字**。[A，README "Goal" 节]
- 规模口径（官方飞书 Features 表，README 链接）：支持 **70+ 图文/视频基准、200+ 模型**（含商业 API 与开源模型）。[A]
- 2025-2026 重要更新 [A]：
  - 2025-02-20：大批新模型（InternVL2.5、Qwen2.5VL、Doubao-VL、Janus-Pro 等）+ MMMU-Pro、WeMath、CC-OCR、CMMMU 等基准。
  - 2025-05-24：支持 **LMDeploy / vLLM 多机分布式推理**；接入 HLE-Bench、MMVP、MM-IFEval、OmniDocBench、Video-MMLU、QBench-Video、MME-CoT 等大批基准与 InternVL3、Gemini-2.5-Pro、Kimi-VL、LLaMA4、Grok 等模型。
  - 2025-09-12：大更新——thinking 模式模型支持自定义 `split_thinking`（`SPLIT_THINK=True`）；长回复（>16k/32k token）支持 TSV 存储防 xlsx 截断（`PRED_FORMAT=tsv`）；MCQ 更多路由到 LLM 选择抽取器。
  - 2026-04-08：支持 **Video-MME-v2**。
  - 2026-11-10：新增**音频模态**评测，首个基准 MMSU。
  - Judge 模型更替预告：OpenAI 将在 2027-01-14 下线 gpt-4o/gpt-4.1 系列，VLMEvalKit 届时把默认 judge 换为 `gpt-6-luna`，并提示换 judge 会影响跨期分数可比性。
- 榜单：OC 多模态榜（rank.opencompass.org.cn/leaderboard-multimodal）、HF OpenVLM Leaderboard、HF **视频榜** openvlm_video_leaderboard；评测记录开放下载（OpenVLM.json）。[A]

### 3.5 AgentCompass（智能体评测子系统，2026 年最重要新增）

仓库 https://github.com/open-compass/AgentCompass ，`pip install agentcompass`（v1.0.0，2026-09-30），Apache-2.0，Python 3.12+，文档 https://agent-compass.mintlify.app/ 。[A]

- 核心设计：把 **Model / Benchmark / Harness / Environment 四者解耦**，可组合；提供任务调度、隔离执行（本地进程 / Docker / Daytona / Modal / OpenSandbox）、评测、结果持久化（增量、断点续跑、失败重试）、轨迹分析（trajectory/tool call/token/latency 全记录）。
- 规模：README 徽章/正文口径为 **40+ 公开基准、10+ Harness**（主 OpenCompass README 2026-09-30 条目写"30+ 公开基准"，两处口径略有出入，以 AgentCompass 自身文档"40+"为准）[A]。
- 已集成基准（文档 llms.txt 全量列举，2026-11-25 抓取）[A]：
  - 软件工程：SWE-bench Verified、SWE-bench Pro、**SWE-Bench Pro Verified**（团队自研，arXiv:2609.08149，2026-09-08，针对 reward hacking 与任务质量问题）、SWE-bench Multilingual、SWE Atlas、Frontier-SWE、Frontier Engineering、DeepSWE、GDPval-AC。
  - 终端/计算机使用：Terminal-Bench 2 / 2 Verified / 2.1 / 4 / Science、OSWorld、OSWorld-G、AndroidWorld、ScreenSpot、WorkspaceBench、SkillsBench、AutomationBench、CyberGym、PinchBench、AA-Briefcase-Lite-AC。
  - 浏览/深度研究：BrowseComp、BrowseComp-ZH、GAIA、HLE、HLE-Verified、DeepResearch Bench、DeepSearchQA、xbench-DeepSearch、WideSearch、SGI Deep Research、ScholarQABench、ResearchClawBench、ResearchRubrics、DiscoveryBench。
  - 科学/其他：ScienceAgentBench、SciCode、FrontierScience、CORE-Bench、LLM-SRBench、BrainArena、Harbor、SealQA、TauBench（τ³）、WildClawBench。
- Harness：Claude Code、Codex、OpenHands、OpenClaw、mini-SWE-agent、NaiveSearchAgent、CORE-Agent、Discovery Agent、OpenEvolve、ResearchHarness、SciCode Tool-Use、Stirrup、Terminus-2、Qwen3-VL GUI，以及裸 OpenAI Chat。[A]
- 裁判：普通 LLM Judge + Agentic Judge；跑前对模型做多模态探针（文本回 OK、红图认红、音频听音）。[A]
- 学术身份：EMNLP 2026 Demo（arXiv:2607.13705），与 SWE-Bench Pro Verified（arXiv:2609.08149）均出自 open-compass 团队。[A][B]

### 3.6 官方对"维度"设计的方法论表态

- **反刷榜机制**（2024-04 界面新闻专访）[B]：借鉴高考"公布考纲不公布考题"——每期榜单发布前只公开**上一期**题目；题目尽量原创、互联网搜不到原题；强调题目动态更新。
- **中立性**：CompassRank 官方反复声明"不受任何商业利益干扰"；榜单提交规则明确**不接受内部迭代研发模型或有邀请访问限制的 API**（old-home 页脚）[A]。
- 局限自承：VLMEvalKit README 明确写"不设计为复现所有第三方基准论文数字"（generation vs PPL 差异、模型专属模板缺失）[A]。

---

## 四、CompassRank 榜单体系

来源：https://rank.opencompass.org.cn/old-home?lang=zh-CN （JS 渲染，借 old-home 抓取，2026-11-25）[A]；2.0 发布稿 [A]。

### 4.1 榜单结构

**（1）官方自建榜单**（按官方评测规则、官方跑模型）：
- 大语言模型榜（客观）
- 多模态模型榜（客观）
- 模型对战榜（主观 pairwise）
- 月度榜（如"26 年 05 月榜"），区分开源/闭源、New 标记、rank 升降箭头。
- 抓取时多模态月榜前列出现 Qwen3.7-Plus、Gemini-3.1-Pro-Preview、Qwen3.5-397B-A17B、GPT-5.5(high)、Kimi-K2.6 等条目（**原样记录，未独立核实这些模型的公开发布情况**）；另一榜单快照含 GPT-5-20250807、Gemini-2.5-Pro、Doubao-1.5-Pro 等。[A，抓取日内容]

**（2）特色能力榜**（单基准垂直榜）：
- MMBench（20 项细粒度视觉能力，约 3000 题）
- OpenVLM Video Leaderboard（MVBench、MMBench-Video 等，时序理解）
- OpenVLM Subjective Leaderboard（Creation-MMBench、MM-AlignBench 等，主观/人类场景）
- VBench / VBench2.0（视频**生成**质量，15 方向，2.0 强调内在真实性）

**（3）技术社区共建榜**（合作伙伴出基准，官方出榜）：

| 榜单 | 领域 | 共建方 |
|---|---|---|
| LawBench | 法律 | 上海AI实验室 + 南京大学 |
| SecBench | 网络安全 | 腾讯 + 上海AI实验室 + 清华 + 港理工 |
| CFBenchmark | 金融 | 同济大学 + 上海AI实验室 + 东方财富 |
| Flames | 价值对齐（高对抗，公平/安全/道德/数据保护/合法 5 维） | 上海AI实验室 + 复旦大学 |
| Shell | 教育/金融/管理领域隐式风险 | 华东师范大学 + 上海AI实验室 |
| TransBench | 多语言翻译 | 阿里国际 + 北京语言大学 + 上海AI实验室 |
| Civil-Bench | 土木工程 | 同济大学 + 上海AI实验室 |
| PriceSeer | 投资顾问/价格预测 | 上海交大 + 上海AI实验室 |
| LifeOmni | 生命健康（基础科学/临床/工具调用/安全） | 上海交大医学院 + 上海AI实验室 |
| AIDABench | AI 数据分析工具（端到端办公分析） | 商汤 + 上海AI实验室 |
| RAVEN-Eval | AI 视频生成（多评委成对比较） | 上海AI实验室 + 央视超高清国重 + 中国美院 + 芒果TV |

另：**CNFinBench**（金融，含金融 Agent 长程任务榜）独立子站运行。[A]

### 4.2 参与方式与规则

- 模型方提供模型仓库地址或标准 API 接口至 opencompass@pjlab.org.cn 申请入榜；不支持内部迭代模型/邀请制 API。[A]
- 客观题由 CompassKit 工具链跑；主观题采用 LLM 对战 + 人工抽检（2.0 发布稿披露 500+ 中文题对战设计）。[A]
- 榜单提供团队"模型解读"；定期更新。[A]
- 完整书面《榜单规则》页面为 JS 渲染，本次**未逐条抓取细则**（打分聚合公式、评委模型版本、抽样数量等细则**未查到全文**）。

---

## 五、2025–2026 新增内容（重点）

### 5.1 主仓 OpenCompass What's New 全条目（README 实测）[A]

- **2025.02.15**：`GenericLLMEvaluator`（通用 LLM 评判器）+ `MATHVerifyEvaluator`（数学评测）。
- **2025.02.28**：支持 DeepSeek-R1 系列（流式 reasoning 模型配置）——标志对"推理模型/reasoning 模型"的一等支持。
- **2025.03.11**：SuperGPQA（285 个研究生学科）。
- **2025.04.01**：`CascadeEvaluator` 多评估器级联。
- **2025.07.26**：Intern-S1 通用与科学基准。
- **2025.12.08**：SciReasoner 科学推理评测（上游 InternScience/SciReason）。
- **2026.02.05**：Intern-S1-Pro 通用与科学基准。
- **2026.03.17**：`RawPromptTemplate`（原始 prompt/结构化对话透传，支持 ChatML 数据集与 API 模型）。
- **2026.03.20**：跨任务并发推理 + 评测监听 + 心跳（大规模评测效率）。
- **2026.05.25**：重复输出/循环输出分析工具 `tools/analyze_repeat.py`（针对推理模型复读退化）。
- **2026.07.27**：`GenInferencer` 支持**多轮推理**；接入 Multi-IF（多轮指令遵循）。
- **2026.07.28**：新增 OpenAI **Responses API**、**LiteLLM AI Gateway**；Gemini/Anthropic 升级到最新 SDK。
- **2026.08.25**：主仓集成 VLMEvalKit（多模态原生评测流程）。
- **2026.09.21**：主流厂商旗舰模型 API 推荐配置更新；中英文文档体系全面重构。
- **2026.09.30**：AgentCompass 发布（EMNLP 2026 Demo，40+ 基准/10+ harness）。

同期数据集目录新增的高密度方向：竞赛数学（aime2025/2026、hmmt2026、BeyondAIME、OlymMATH、ZebraLogic、korbench、NPHardEval）、高难综合（HLE、HealthBench、SimpleQA/chinese_simpleqa）、过程评测（ProcessBench）、行业（医学 MedCalc/MedXpertQA/NEJM、金融 BuySideFinBench/FinanceIQ、海事/气候/材料）、多轮指令（MultiIF/advancedIF）。[A]

### 5.2 多模态侧 2025–2026 主线 [A]

- thinking/reasoning 多模态模型适配（split_thinking）、长输出 TSV、多机分布式推理（2025-05/09）。
- 视频：Video-MME-v2（2026-04）、视频榜持续运营。
- 音频：MMSU（2026-11），从"视觉语言模型工具包"实质走向全模态。
- HLE-Bench 等新一代高难基准与数十个 2025 新基准（OmniDocBench、MM-IFEval、EMMA、TDBench、VisuLogic 等）。

### 5.3 智能体侧 2026 主线 [A][B]

- AgentCompass 技术报告（2026-07）→ EMNLP 2026 Demo（2026-08 接收）→ PyPI v1.0.0（2026-09-30）。
- 自研 SWE-Bench Pro Verified（2026-09，arXiv:2609.08149），直击编码智能体现有 SWE 评测的 reward hacking 与任务质量问题，数据挂 HuggingFace `opencompass/SWEBench-Pro-Verified`。
- 覆盖 GUI/安卓/终端/浏览器/深度研究/科研/网络安全等环境的 40+ 基准矩阵。

### 5.4 学术产出

- 主平台技术报告：arXiv:2605.19276（2026-05/06）[B]。
- VLMEvalKit：arXiv:2407.11691（ACM MM 2024）[B]。
- AgentCompass：arXiv:2607.13705（EMNLP 2026 Demo）[B]。
- T-Eval：ACL 2024 [A]。
- CNFinBench：ACL Anthology 2026（pages 8765–8776）[B，据 CNFinBench 站点此前抓取]。

### 5.5 商业化/交付线索（弱证据）

- 火山引擎万有商城存在"司南 OpenCompass 大模型评测体系"SaaS 商品页（SaaS 交付、合作咨询、按天计费，页面主体为官方介绍文案）。[C]
- 招聘与合作邮箱 opencompass@pjlab.org.cn、cnfinbench@pjlab.org.cn 长期有效。[A]
- **明确的"企业版"功能差异、私有化部署报价/SLA 未查到官方说明。**

---

## 六、适用场景与局限

### 6.1 适合谁用

1. **模型研发团队做回归测试**：配置即代码、可断点续跑、支持分布式，千亿模型全量评测小时级（官方早期口径 3 小时）[A]；2026 年并发监听进一步提效。
2. **需要"一套框架测文本+多模态+API+开源"的团队**：主仓 + VLMEvalKit 集成 + LiteLLM 网关，覆盖最广的国产模型矩阵（Qwen/GLM/DeepSeek/Kimi/MiniMax/豆包/Intern 系），这是相对海外框架最实际的优势。[A]
3. **中文与垂直行业评测**：C-Eval/CMMLU/GAOKAO/CLUE/高考/法律/金融/医疗/政务中文基准齐全，并有 LawBench、CNFinBench、SecBench 等共建行业榜可对标。[A]
4. **要发论文/送榜**：CompassRank 是国内事实上的主流送榜渠道之一；Meta Llama 文档背书使其在国际语境也有能见度。[A]
5. **2026 年评测智能体**：AgentCompass 提供 Model/Benchmark/Harness/Environment 解耦的工程化方案，覆盖 SWE/终端/GUI/Deep Research，且隔离执行、轨迹留存、可审计，比自己搭评测脚手架成熟。[A]

### 6.2 局限与注意点

1. **工程门槛偏高**：纯配置文件体系、依赖组合复杂（pyext 需 3.10、vLLM/LMDeploy 要独立环境、flash-attn/transformers 版本随模型矩阵碎片化，VLMEvalKit README 列了十几套版本建议）。[A]
2. **榜单分数的跨期可比性问题**：题目按年滚动不公开当季题、judge 模型会换代（VLMEvalKit 自己提示 gpt-6-luna 换代影响分数）[A]；月度榜分数不宜直接跨月横比。
3. **官方自承不保证复现第三方论文原始分**（评测范式差异、模型专属模板不全）。[A]
4. **"高分低能"/刷题争议**是所有公开静态基准的通病；OpenCompass 的应对（滚动原创题、HLE/AgentCompass 动态任务）缓解但不能消除。其榜单仍以选择题/静态集为主干，真实任务评测权重依赖 AgentCompass/VLM 主观榜补充。[A][B]
5. **智能体评测与主仓割裂**：SWE-bench、Terminal-Bench 等不在 opencompass 包里，需另学 AgentCompass 一套 CLI/配置；"一站式"目前是组织层面的一站式，不是单包。[A]
6. **数据集数字口径不一**：README 称 100+，配置目录实测 250+（含多配置/废弃/垂直集），引用时应注明口径。[A]
7. 332 个 open issues（2026-11-25）[A]，维护活跃但长尾问题不少；部分基准依赖外部数据下载，离线环境准备成本高（社区常见反馈，C 级，**未系统核实**）。
8. 榜单中立性虽有制度声明，但运营方为国家队研究机构（上海AI实验室），在部分海外用户眼中存在感知层面的信任门槛；这是感知问题而非已证实的操纵问题。[B，界面新闻报道其对刷榜乱象的自我定位]

---

## 七、同类框架对比

### 7.1 对比表

| 维度 | **OpenCompass**（+VLMEvalKit/+AgentCompass） | **lm-evaluation-harness**（EleutherAI） | **HELM**（Stanford CRFM） | **FlagEval 天秤**（智源 BAAI） |
|---|---|---|---|---|
| 仓库 | github.com/open-compass/opencompass | github.com/EleutherAI/lm-evaluation-harness | github.com/stanford-crfm/helm | github.com/FlagOpen/FlagEval |
| Star（2026-11-25 实测） | 7,639 [A] | **14,535** [A] | 未本次抓取 | 未本次抓取 |
| 许可证 | Apache-2.0 [A] | MIT [A] | Apache-2.0（一般认知，**本次未核**） | 未本次核实 |
| 首次发布 | 2023-06 建仓 / 2023-10 官宣 [A] | 2020-08（仓库创建）[A]，LLM 评测事实标准前身 | 2022（HELM 论文/项目）[B，信通院报告] | 2023 [B，信通院报告] |
| 定位 | 一站式平台 + 榜单 + 基准社区 + 行业共建 | 学术社区事实标准的**轻量评测库**，复现论文数字最方便 | "整体评测"研究项目：同一模型跑全维度并公开权重误差/效率等 | "能力-任务-指标"三维框架的开放评测平台 |
| 评测范式 | PPL + 生成 + CoT + LLM Judge + 主观对战 + 多轮 + 智能体 | 以 PPL/loglikelihood 与 generation 为主，few-shot 评测标杆 | 标准化 prompt 下多指标（准确率、校准、鲁棒性、偏见、毒性、效率等） | 客观+主观；"能力-任务-指标"细粒度认知边界 |
| 数据集规模口径 | 配置目录 250+；官方"100+"[A] | 60+ task group（社区贡献，数字随版本变；**具体数本次未核**） | 按 scenario 滚动（HELM→HEIM→HRS 等）[B] | 6 大评测任务、近 30 数据集（信通院 2024 口径）[B] |
| 中文/国产模型 | **最强项**：C-Eval/CMMLU/高考/CLUE 齐，国产 API 一等支持 [A] | 中文任务靠社区提交，深度一般 | 英文为主，多语种覆盖有限 | 中文强（智源本土） |
| 多模态 | VLMEvalKit（70+ 基准/200+ 模型，图文/视频/音频）[A] | 有 lmms-eval 等姊妹项目（**本次未核**） | HEIM（文生图 12 维 62 场景）、HRS-Bench [B] | 有多模态评测模块（**细节本次未核**） |
| 智能体 | **AgentCompass：40+ 基准/10+ harness，沙箱执行** [A] | 基本不覆盖端到端 Agent 任务 | 不侧重 | 不侧重 |
| 分布式/推理后端 | HF/LMDeploy/vLLM、任务切分、跨任务并发 [A] | 支持 accelerate/vLLM 等，单机并行常见 | 自有批处理 | 未本次核实 |
| 榜单 | CompassRank（官方自建+月度+社区共建+垂直行业榜）[A] | 无官方总榜（被各家榜单引用为底层工具） | crfm 官网 HELM leaderboard | FlagEval 官方榜 |
| 上手成本 | 中高（配置体系） | **低**（一条命令跑标准任务） | 中高 | 中 |
| 典型用户 | 国内厂商研发/送榜、垂直行业评测、2026 年 Agent 评测 | 论文作者、快速复现 MMLU/GSM8K 数字 | 学术整体性比较、公平性/效率研究 | 国内学术与产业 |

### 7.2 其他相关参照

- **SuperCLUE**：中文大模型评测基准与榜单品牌（CLUE 团队），与 OpenCompass 是**相互独立的两套体系**；OpenCompass 会包含部分中文基准，但 SuperCLUE 不在 open-compass 组织下（组织关系基于公开事实，**SuperCLUE 最新形态本次未做专项调研**）。
- **PandaLM**（WeOpenML）：以"LLM 当裁判"模型为核心的评测思路，对应 OpenCompass 里的 LLM Judge/GenericLLMEvaluator 能力层，非平台级竞品（**细节本次未核**，B/C 级背景）。
- **信通院《大模型基准测试体系研究报告（2024）》**把 HELM（文本，多维度含偏差/毒性/效率）、HEIM（文生图 12 维）、HRS-Bench（KAUST，13 技能/50+ 场景）、OpenCompass（CompassKit+CompassHub+CompassRank，产业影响力大）、FlagEval（能力-任务-指标三维，6 任务近 30 集）并列为代表性体系。来源 PDF：https://www.caict.ac.cn/kxyj/qwfb/ztbg/202407/P020240711534708580017.pdf [B]
- **垂直/新型基准**（多被 OpenCompass/AgentCompass 收编为数据集而非框架竞品）：HLE、SWE-bench、Terminal-Bench、BrowseComp、GAIA、SuperGPQA、MMMU/MMMU-Pro、Video-MME 等。

### 7.3 一句话选型建议

- 要**快速复现论文标准数字 / 海外英文基准**：lm-evaluation-harness。
- 要**学术整体性、公平性效率多指标**：HELM。
- 要**中文 + 国产模型 + 行业垂直 + 官方榜单 + 2026 年智能体/多模态一体化**：OpenCompass 生态目前是国内覆盖最全的一套，代价是学习成本与跨子系统割裂。

---

## 八、未查到 / 待补充清单

1. OpenCompass 2023 年**首次开源的具体官宣日期**（仓库创建 2023-06-15 已确认，但首次公开 release/公告未见）。
2. CompassRank《榜单规则》全文细则（聚合公式、抽样量、judge 版本、人工标注比例）。
3. CompassHub 是否存在数据集版本托管/市场式机制（目前仅能确认"导航 + 投稿"）。
4. 官方企业版/私有化部署的功能边界与报价（仅有火山引擎商城 C 级商品页）。
5. HELM、FlagEval、SuperCLUE、PandaLM 的最新数据（star、数据集数、2026 动态）——本次对比表中已逐项标注"未本次核实"，成文引用前需补抓。
6. TurboSquirrel 等早期安全基准在当前版本的确切位置（疑并入 `safety/`）。
7. opencompass 主仓 release v0.5.5 的 changelog 细节（版本日期据此前抓取，建议成稿前再核 Releases 页）。

---

## 附：主要来源清单

**[A] 官方**
1. https://github.com/open-compass/opencompass （README_zh-CN、configs/datasets 目录、GitHub API 元数据，2026-11-25 抓取）
2. https://github.com/open-compass/AgentCompass （README；文档 llms.txt 与 benchmarks 页，agent-compass.mintlify.app）
3. https://github.com/open-compass/VLMEvalKit （README，2026-11-25 抓取）
4. https://github.com/open-compass/T-Eval 、 https://github.com/open-compass/code-evaluator （GitHub API 元数据）
5. https://rank.opencompass.org.cn/old-home?lang=zh-CN （榜单结构与共建榜名单）
6. https://www.shlab.org.cn/news/5443498 （2023-10-19 官宣）
7. https://www.shlab.org.cn/news/5443858 （2024-01-30 OpenCompass 2.0 发布稿）
8. https://cnfinbench.opencompass.org.cn/ （CNFinBench，此前会话抓取）
9. https://opencompass.readthedocs.io/ （数据集配置文档，此前会话抓取）
10. https://ai.meta.com/llama/get-started/#validation （Meta Llama 官方推荐，经 README 引用）

**[B] 论文 / 权威媒体 / 研究机构**
11. arXiv:2605.19276 OpenCompass 技术报告（2026-05/06）
12. arXiv:2607.13705 AgentCompass（EMNLP 2026 Demo）
13. arXiv:2609.08149 SWE-Bench Pro Verified（2026-09）
14. arXiv:2407.11691 VLMEvalKit（ACM MM 2024）
15. 界面新闻，2024-04-26，《上海人工智能实验室要做大模型技术的"度量衡"》 https://www.jiemian.com/article/11100699.html
16. 财联社，2024-12-13，金融大模型应用评测报告报道 https://www.cls.cn/detail/1888223 （此前会话抓取）
17. 中国信通院《大模型基准测试体系研究报告（2024）》 https://www.caict.ac.cn/kxyj/qwfb/ztbg/202407/P020240711534708580017.pdf
18. T-Eval，ACL 2024（ACL Anthology，经 GitHub 仓库页确认）

**[C] 二手聚合（仅旁证）**
19. CSDN OpenCompass 官方博客《一文读懂司南大模型评测体系》（2024-07-26）
20. 火山引擎万有商城"司南 OpenCompass"SaaS 商品页
