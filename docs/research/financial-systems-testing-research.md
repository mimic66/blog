# 复杂金融系统测试的挑战（2020–2026）调研报告

> 调研日期：2026-10-27
> 时间范围：以 2020–2026 为主；对用户点名的历史基准案例（Knight Capital、BATS IPO、Facebook IPO 等 2012 年事件）作为"行业教科书案例"保留。
> 证据分级：
> - **【已核实】**：有监管文件、交易所/央行公告、司法判决书、机构一手工程博客或上市公司公告等可核查来源。
> - **【需进一步核实】**：仅有媒体报道、二手文章或单一来源，尚未找到一手文件。
> 本报告不编造案例与数字；凡媒体数据均标注来源性质。所有引用给出 URL 或正式文号。

---

## 0. 结论速览

复杂金融系统的测试难点不是"测试用例写得不够多"，而是由以下结构性因素叠加造成：

1. **正确性的时间属性与一致性属性极端**：实时撮合微秒级、7×24×365 清算（如 FedNow）、零停机迁移、金额不可有 1 分钱误差、日终批处理有严格 cutoff。
2. **系统是"系统的系统"**：一个核心银行改造牵动几十到几百个上下游（SWIFT、CNAPS/CIPS、交易所、征信、第三方支付、卡组织），环境搭建与第三方沙箱成为瓶颈。
3. **生产数据不能直接用于测试**：GDPR、中国《个人信息保护法》、JR/T 0171/0197 等使测试数据获取、脱敏、子集化本身成为合规工程。
4. **测试结论要对监管可审计**：SR 11-7 模型验证、MiFID II RTS 6/RTS 7 算法一致性测试、中国程序化交易"先报告后交易"均要求留痕、可复现、可独立验证。
5. **遗留核心（COBOL/大机）与新型分布式架构并存**：一边是"不敢改"的 40 年存量，一边是数千微服务、事件驱动、多云，测试范式被两头撕扯。
6. **真实故障反复证明：变更管理、环境一致性、回退预案、极端行情容量，是最常被监管点名的四类测试缺口**（Knight 2012、TSB 2018、TARGET2 2020、中国券商 2022–2023 系列罚单、NYSE 2024）。

---

## 1. 核心银行 / 交易 / 支付清算系统的测试复杂性

### 1.1 定义与技术机理

| 子领域 | 关键非功能/功能约束 | 为什么测试难 |
|---|---|---|
| 核心银行（core banking） | 账务强一致、借贷必相等、7×24 可用（部分系统）、日终批处理与 cutoff、计息精度 | 联机交易与批量作业共享账务数据；批处理窗口与 24h 服务冲突；跨年/跨结息日/闰年场景一年只出现一次，难以在测试期穷举 |
| 交易/撮合系统 | 微秒~毫秒延迟、价格/时间优先、开盘集合竞价、峰值并发、错误订单熔断 | 撮合是有状态确定性状态机，复现需要逐笔回放（tick-level replay）；生产行情分布在极端日（开盘、熔断、指数调仓）才出现 |
| 支付清算（RTGS/ACH/即时支付） | 最终一致/实时全额结算、不可撤销、流动性与排队机制、跨时区日历 | 一笔支付横跨多个参与方与结算账户；失败重放、重复提交、乱序、部分失败必须幂等；夜间批处理与 24h 即时支付并存 |

**金额精度机理（浮点 vs 定点）**：二进制浮点（IEEE 754 double）无法精确表示多数十进制小数（如 0.1），金融系统通常使用定点十进制（整数 + 隐含小数位）或任意精度十进制库（如 Java `BigDecimal`、ISO/IEC/IEEE 60559 十进制浮点）。测试难点在于：单测通过的金额在"汇总—分摊—舍入"链路上会出现尾差（rounding drift），必须定义舍入规则（HALF_UP/ banker's rounding）与尾差归属账户。此为通用工程机理，本报告不引用未经核实的"某行因浮点损失 X 元"传闻。

**日终/cutoff 与计息基础（day count convention）**：不同产品使用 30/360、Actual/365 Fixed、Actual/360、Actual/Actual ISDA 等不同计息天数惯例，加上工作日历、节假日调整（following/modified following）、闰年 2 月 29 日，计息结果可能因日历数据错误而偏差。这些属于"低频但必然发生"的时间型测试场景，常规回归容易遗漏。

### 1.2 一手证据

- **FedNow 的 24×7×365 设计目标**【已核实】：美联储明确 FedNow "designed to maintain uninterrupted 24x7x365 processing"，2023-07-20 上线，57 家早期采纳机构先完成正式测试与认证（2023-06-29 公告）。
  - https://www.federalreserve.gov/Paymentsystems/FedNow_about.htm
  - https://www.federalreserve.gov/newsevents/pressreleases/other20230720a.htm
  - https://www.federalreserve.gov/newsevents/pressreleases/other20230629a.htm
- **Temenos 自称其平台内部"每天运行数十万测试"，并把 release validation tests 随产品能力交付**（厂商一手材料，属营销口径，引用时注意立场）【已核实存在该说法】：
  - Temenos AG Interim Report 2023："running hundreds of thousands of tests daily" https://www.temenos.com/wp-content/uploads/2023/09/temenos-interim-report-2023-x30x401k8.pdf
- **银行软件"测试预言机问题"（oracle problem）学术研究**：IEEE IRI 2023 论文《An Approach to Testing Banking Software Using Metamorphic Relations》指出银行软件因复杂度高、缺乏可靠 oracle 而难以判定输出是否正确，建议用蜕变关系（资金守恒等不变式）测试。【已核实，IEEE/Computer Society】
  - https://www.computer.org/csdl/proceedings-article/iri/2023/345800a173/1Q258ftrQ6A
  - 预印本 PDF：https://www.cs.montana.edu/izurieta/pubs/IRI_2023.pdf

---

## 2. 遗留核心系统（COBOL / 大机）测试

### 2.1 定义与机理

遗留系统测试的典型痛点：
1. **知识断层**：COBOL/PL1/汇编专家退休，业务规则只存在于代码与老人记忆里，文档缺失。
2. **环境难复刻**：大机（IBM Z）、CICS、IMS、VSAM、特定中间件的测试环境许可与成本高；批处理依赖一夜的全量数据，造数困难。
3. **耦合深、变更半径不可知**：copybook 复用、隐式文件契约、JCL 作业流，改动一个字段可能影响数十个作业。
4. **"不敢改"文化**：因回归覆盖不足，默认策略是"不动"；而不动又导致技术债与安全债累积，形成悖论。
5. **迁移测试（big-bang vs strangler）**：新旧核心并行期必须做**双系统对账（parallel run / dual-core reconciliation）**、余额与历史交易核对、总账平衡验证。

### 2.2 证据

- **CIBC 三年升级 4 万个 COBOL 程序**【已核实，IBM 社区官方案例】：CIBC 用三年将 100%（40,000 个）大机应用升级到 Enterprise COBOL 6，涵盖批处理、COBOL/CICS 联机、COBOL/IMS、COBOL/Db2。
  - https://community.ibm.com/community/user/blogs/vikash-kumar/2025/04/23/cibcs-success-story-about-upgrading-40000-cobol-p
- **IBM：银行"害怕触碰大机技术债"**【已核实存在该表述，IBM Think】：
  - https://www.ibm.com/think/topics/mainframe-for-banking
- **BNP Paribas 的 COBOL 逆向文档化研究**【已核实，学术/行业论文】：2026 年 IEEE/ACM "Software Engineering Challenges in Financial Firms" 工作坊论文《LLM-Assisted Retro-Documentation for Legacy COBOL Applications in Banking》，明确说法国巴黎银行大量核心银行系统依赖 COBOL 大机，专家退休与 Java 迁移造成知识保存难题，用多 LLM 流水线在气隙（air-gapped）环境内自动生成功能/技术文档。
  - https://dl.acm.org/doi/10.1145/3786170.3788390
- **IBM IBV《The 94% core banking problem》（2025）**：报告称 69% 的受访银行在现代化过程中"维持充分风险管理变得更难"，系统更新受"庞大的测试需求（expansive testing requirements）"制约。【已核实存在该报告；百分比为厂商调研口径】
  - https://www.ibm.com/thought-leadership/institute-business-value/en-us/report/core-banking-modernization
- **TSB 迁移失败（2018，监管 2022 处罚）是"遗留核心迁移测试与外包治理失败"最权威的反面教材**【已核实】：
  - FCA/PRA 2022-12-20 通报：TSB 因 IT 升级项目（从 Lloyds 平台迁移至 Sabadell 的 Proteo4UK）的操作风险与外包治理失败，合计被罚 **48,650,000 英镑**（FCA 29.75m + PRA 18.9m），另已向客户支付 3,270 万英镑补偿；数据本身迁移成功，但平台切换后立即出现技术故障，直到 2018 年 12 月才恢复正常。
    - https://www.fca.org.uk/news/press-releases/tsb-fined-48m-operational-resilience-failings
    - https://www.bankofengland.co.uk/news/2022/december/tsb-fined-for-operational-resilience-failings
  - PRA 对前 CIO Carlos Abarca 的 Final Notice（2023）详述主迁移事件（MME, 2018-04-20/22 周末）与 Defender Plan：
    - https://www.bankofengland.co.uk/-/media/boe/files/prudential-regulation/regulatory-action/final-notice-from-pra-to-former-tsb-bank-plc-cio.pdf

---

## 3. 系统集成与测试环境挑战

### 3.1 机理

- **依赖拓扑庞大**：一个典型银行交易链路可能经过渠道层→核心→支付网关→SWIFT/CNAPS/CIPS→代理行→央行清算；证券交易还要经过柜台（如恒生电子柜台）→交易所网关→撮合→登记结算。
- **第三方沙箱不稳定/不真实**：外部机构沙箱常常限速、数据陈旧、行为与生产不一致、只在对方工作时间可用、不支持异常注入，导致集成测试"绿得假"。
- **环境数据与配置漂移**：测试环境无法承载生产规模，中间件版本、参数、证书、日历表不一致，"在我环境里是好的"。
- **服务虚拟化（service virtualization）与契约测试**成为应对手段：用虚拟服务模拟不可用的上下游；消费者驱动契约（CDC/Pact 类）固化接口预期。
- **全网测试（market-wide testing）**：交易所/中国结算在周末组织全市场联网测试（新业务、退市整理、两融等），所有券商必须参加，是少有的"全链路真实环境"机会。

### 3.2 证据

- **中国券商周末全网测试与宕机关联**【已核实，财经媒体+监管文书】：2023-06-19 中信证券故障前的周末，深交所、中国结算深圳分公司组织可转债退市整理期全网测试，北交所组织两融业务优化全网测试；业内分析券商宕机常与周末升级有关。
  - 财联社：https://m.cls.cn/detail/1382094
- **Temenos 实施商对集成测试复杂度的描述**【已核实，集成商白皮书】：Luxoft/Tricentis 指出 Temenos Transact 测试必须覆盖"core to interface to downstream system"端到端流，是主要复杂度来源。
  - https://www.luxoft.com/files/pdfs/whitepapers/Automated-Temenos-testing-with-Tricentis-and-Luxoft.pdf
- **ING 银行的云原生转型中，容器化首先解决的是"测试环境交付太重（每团队申请 VM）"的问题**【已核实，CNCF 官方案例】：
  - https://www.cncf.io/case-studies/ing
- **Monzo：约 3,000 个微服务、迁移 EKS 集群的工程规模**【已核实，Monzo 官方工程博客】：体现新一代银行集成测试的爆炸半径。
  - https://monzo.com/blog/the-engineering-behind-the-platform

---

## 4. 测试数据管理（TDM）与合规

### 4.1 机理

1. **生产数据不得直接进测试环境**：GDPR（欧盟 2016/679）、中国《个人信息保护法》（2021-11-01 施行）、《数据安全法》、《网络安全法》共同约束；金融业另有更细行标。
2. **脱敏（masking）的保真悖论**：静态脱敏要保持数据的格式合法、参照完整性（外键链条）、业务分布真实（如年龄与授信额度相关性），脱敏过度则测不出问题，脱敏不足则违规。
3. **数据子集（subset）与依赖链**：抽取一个客户的完整业务画像会沿账户→交易→贷款→抵押→担保→总账扩散，子集裁剪不当就断链。
4. **匿名化 vs 假名化**：PIPL 第四条明确"匿名化处理后的信息"不属于个人信息——这决定了测试数据能否脱离个信法约束，但要达到真正匿名化很难。
5. **合成数据（synthetic data）**是趋势，但合成数据对异常、长尾、欺诈模式的代表性需要验证。

### 4.2 一手法规/标准

- **GDPR**：Regulation (EU) 2016/679，官方文本 EUR-Lex：https://eur-lex.europa.eu/eli/reg/2016/679/oj
- **中国《个人信息保护法》**【已核实】：2021-08-20 通过、2021-11-01 施行；全国人大全文：
  - http://www.npc.gov.cn/npc/c2/c30834/202108/t20210820_313088.html
- **JR/T 0171—2020《个人金融信息保护技术规范》**【已核实】：人行 2020-02-13 发布实施，将个人金融信息按敏感程度分级（C1/C2/C3）。
  - 全国标准信息平台/金融标准全文公开系统：https://cfstc.pbc.gov.cn/bzgk/detail/?bzId=1873&id=0 （此为 JR/T 0197 页面；JR/T 0171 标准号见该行标体系）
- **JR/T 0197—2020《金融数据安全 数据安全分级指南》**【已核实】：2020-09-23 发布实施，金融数据 1–5 级，"重要数据"不低于 5 级；个人金融信息参照 JR/T 0171 且"从高考虑"。
  - https://std.samr.gov.cn/hb/search/stdHBDetailed?id=B081D125A6762DB8E05397BE0A0A5EA7
  - PDF：https://aicrc.cn:8521/upload/JRT%200197%E2%80%942020%E9%87%91%E8%9E%8D%E6%95%B0%E6%8D%AE%E5%AE%89%E5%85%A8%20%E6%95%B0%E6%8D%AE%E5%AE%89%E5%85%A8%E5%88%86%E7%BA%A7%E6%8C%87%E5%8D%97.pdf
- **JR/T 0223—2021《金融数据安全 数据生命周期安全规范》**【已核实】：覆盖采集、传输、存储、使用、删除、销毁全生命周期。
  - 引用与文本见：https://www.guilinbank.com.cn/api-file/busi/file/operation/getFileFromDB?fileInfoId=T_FILE_INFO_68B613958026484CBB19CC374C597373
- **工行 App 资产显示异常事件中"数据显示层 vs 真实资金"的区分**（见第 10 节案例），也说明测试数据/展示数据一致性验证的重要性。

---

## 5. 监管与合规驱动的测试

### 5.1 模型风险管理：SR 11-7

【已核实】美联储/OCC《Supervisory Guidance on Model Risk Management》（SR 11-7，2011-04-04）确立三大支柱：模型开发实施与使用、**独立模型验证（effective challenge、概念合理性、持续监控/结果分析 backtesting）**、治理与控制。
- 原文：https://www.federalreserve.gov/supervisionreg/srletters/sr1107.htm
- 附件 PDF：https://www.federalreserve.gov/supervisionreg/srletters/sr1107a1.pdf
- 【更新】2026-04-17 美联储/OCC/FDIC 发布修订版（SR 26-2），取代 SR 11-7，强调风险为本：https://www.federalreserve.gov/supervisionreg/srletters/SR2602.pdf

测试含义：模型必须有**独立验证团队**、上线前验证与投产后持续监控（性能监控、稳定性监控、基准对照 benchmarking），这就是金融 AI/ML 测试与普通软件测试最大的制度差异。

### 5.2 算法交易测试：MiFID II RTS 6 / RTS 7

【已核实】注意两个法规的常见混淆：
- **RTS 6 = Commission Delegated Regulation (EU) 2017/589**（2016-07-19），规定**投资公司**算法交易系统的组织要求；其 **Article 5** 要求交易算法/系统在部署前进行测试，包括**一致性测试（conformance testing）**、在非生产环境验证、受控部署（controlled deployment），并留存算法与变更记录。
  - 欧盟委员会 PDF：https://ec.europa.eu/finance/securities/docs/isd/mifid/rts/160719-rts-6_en.pdf
- **RTS 7 = Delegated Regulation (EU) 2017/584**，规定**交易场所**的系统韧性、容量与算法测试要求；要求场所提供 conformance testing 设施、测试环境与生产环境严格隔离或仅盘后测试，成员须认证其算法已测试。
  - EUR-Lex：https://eur-lex.europa.eu/eli/reg_del/2017/584/oj
- FCA《Algorithmic Trading Compliance in Wholesale Markets》对 RTS 6 的五章结构与盘前风控（市场/信用额度、最大订单量、自动节流、价格项圈 price collars）有实操解释：
  - https://www.fca.org.uk/publication/multi-firm-reviews/algorithmic-trading-compliance-wholesale-markets.pdf

### 5.3 中国程序化交易监管

【已核实】证监会公告〔2024〕8 号《证券市场程序化交易管理规定（试行）》，2024-05-11 公布、**2024-10-08 施行**：
- 实行**"先报告、后交易"**；
- 第二十二条：高频交易投资者除一般报告外，还须报告**系统测试报告、系统故障应急方案、服务器所在地**；
- 交易所实时监控异常交易，可对高频差异化收费、收撤单费。
- 证监会发布页：http://www.csrc.gov.cn/csrc/c100028/c7480577/content.shtml
- 公告页（含文号）：http://www.csrc.gov.cn/csrc/c101954/c7480579/content.shtml
- 全文 PDF：http://www.csrc.gov.cn/csrc/c101954/c7480579/7480579/files/%E9%99%84%E4%BB%B61%EF%BC%9A%E8%AF%81%E5%88%B8%E5%B8%82%E5%9C%BA%E7%A8%8B%E5%BA%8F%E5%8C%96%E4%BA%A4%E6%98%93%E7%AE%A1%E7%90%86%E8%A7%84%E5%AE%9A%EF%BC%88%E8%AF%95%E8%A1%8C%EF%BC%89.pdf
- 上交所 2024-06-07 实施细则征求意见稿提出高频量化阈值（单账户每秒申报/撤单最高 300 笔等）：https://m.cnstock.com/commonDetail/196105

### 5.4 审计追踪与可复核性

金融测试不只是"过/不过"，还须证明：谁在什么环境、用什么数据/版本、执行了什么用例、结果如何、谁审批、回退方案是什么。中国监管对券商的处罚文书反复出现"升级论证测试不充分、回退方案不完备、未及时报告"等措辞（见第 10 节），说明测试过程证据本身就是合规对象。

---

## 6. 非功能性挑战：性能、容量、混沌、灾备、时间

### 6.1 性能/容量

- **峰值机理**：开盘集合竞价、9:30/13:00 开收盘、重大政策/行情日的登录与委托洪峰，是典型"秒杀式"流量；行情软件还要处理 Level-2 逐笔推送。容量测试若只用均匀负载，会完全错过洪峰与瞬时抖动。
- **中国监管对故障分级有时长量化标准**【已核实，媒体引述监管办法】：集中竞价交易系统以外的实时交易系统严重异常，**持续 30 分钟以上为重大事件、10 分钟以上为较大事件**（《证券期货业网络安全事件报告与调查处理办法》，证监会公告〔2021〕12 号）。
  - 财联社：https://m.cls.cn/detail/1179106
  - 证券时报罚单报道引述同一办法：https://www.stcn.com/article/detail/920153.html

### 6.2 混沌工程与灾备演练

- **工商银行**【已核实，InfoQ 访谈】：工行软件开发中心高级研究员王炳辉称建设混沌工程故障演练平台，对分布式应用实施故障演练；强调金融行业对可靠性要求高于非涉账类系统，且大行处于"大机+分布式转型"并存阶段，生产环境注入故障须审慎（先有高可用、监控、应急预案三前提）。
  - https://www.infoq.cn/article/AAdvTFs2XayEu7hdv4FL
- **北京银行**【已核实，《金融电子化》署名文章】：2021-11 起对云原生统一技术底座做混沌测试，**执行 175 个故障场景、500+ 测试用例、发现 37 项平台稳定性问题**，云原生成熟度由 2 级提升至 4 级；2022 年建设本地化混沌工程平台。
  - http://www.fintechinchina.com/viewpoints/6806
- **信通院《分布式系统稳定性建设指南（2022）》**：混沌工程实验室成员含工行软件开发中心（副理事长）、华泰证券等；并引用《银行业金融机构数据治理指引》对数据应急预案演练的要求。
  - http://www.caict.ac.cn/kxyj/qwfb/ztbg/202206/P020220621683623705917.pdf
- **摩根大通**【已核实，Grafana ObservabilityCON 2021 官方议程】：用 k6 模拟负载、Gremlin 做混沌场景、OpenTelemetry 观测，把韧性测试"左移"。
  - https://grafana.com/events/observabilitycon/2021/observability-driven-development-jpmc/
- **Monzo Stand-in**【已核实，Monzo 官方博客，2025-02】：建设完全独立的备份银行基础设施 Stand-in，在云全面中断时仍能提供关键服务——灾备从"同构双活"走向"异构降级生存"。
  - https://monzo.com/blog/topic/technology/page/3 （索引页；文章标题 "Tolerating full cloud outages with Monzo Stand-in"）

### 6.3 时区、日历、闰年与计息基础

- 跨境组合估值、SWIFT 报文、期货交割都涉及时区切日（如纽约/伦敦/东京）与本地节假日；测试需要"日历数据"作为受版本控制的测试夹具，而非硬编码。
- 2020 是闰年、2024 是闰年，2 月 29 日计息/对账是必测点；此类时间型缺陷四年才暴露一次，属于测试设计中的"日期矩阵"问题（工作日历 + 产品计息基础 + 调整惯例的组合）。本节为**机理性结论**，未引用具体损失数字。

---

## 7. 新技术带来的测试挑战

### 7.1 微服务、契约测试与事件驱动

- **挑战机理**：服务数量爆炸（Monzo 约 3,000 微服务）后，端到端测试组合爆炸、环境不稳定、异步事件的最终一致使断言时机难定；行业应对为**消费者驱动契约测试 + 服务虚拟化 + 生产可观测性**组合，用线上观测补足预生产覆盖。
- 证据：
  - Goldman Sachs Developer《Observability at Scale》（2021）与《SecDb Observability Journey》（2023）：https://developer.gs.com/blog/posts/observability-at-scale ；https://developer.gs.com/blog/posts/secdb-observability-journey
  - Goldman Sachs × GitLab 客户案例（工程团队 52k 测试用例/11k 测试类、千级日构建）：https://about.gitlab.com/customers/goldman-sachs
  - 微服务测试系统性文献综述（74 篇一手研究，Information & Software Technology，2025）：https://www.sciencedirect.com/science/article/abs/pii/S0950584925002095
  - ING 工程团队公开其低代码 Playwright 框架 INGenious（250+ 内部用户、80+ 应用）：https://medium.com/ing-blog/from-silos-to-synergy-how-ingenious-is-transforming-ings-testing-approach-4a010a4f2586

### 7.2 上云与外包/集中度风险

- **CrowdStrike 2024-07-19 全球宕机**【已核实】：Falcon 一个 Windows 内容更新缺陷导致约 850 万台设备蓝屏，波及银行（TD、BofA、JPMorgan Chase、Wells Fargo、AmEx 等）、航空（Delta 五天取消约 7,000 航班、损失逾 5 亿美元并起诉 CrowdStrike/微软）；**不是网络攻击**。
  - CrowdStrike 官方说明：https://www.crowdstrike.com/en-us/blog/to-our-customers-and-partners
  - 美国国会 CRS FAQ R48135（列出受影响银行）：https://www.congress.gov/crs_external_products/R/PDF/R48135/R48135.1.pdf
  - CRS Insight IN12392：https://www.congress.gov/crs-product/IN12392
  - 测试含义：**第三方组件/强制更新的金丝雀发布、变更窗口、客户端可回滚性**成为金融机构无法完全自控的风险；供应商集中度需要在恢复演练中假设"同区域多家机构同时挂"。
- 摩根大通公开说明其混合多云策略的首要原因是**集中度风险**（关键基础设施不能受制于单一云厂商）【已核实，DCD 转述高管】：
  - https://www.datacenterdynamics.com/en/analysis/the-it-wingspan-of-jpmorgan-chase-co
- FIS 披露 Profile 核心"全部开发与测试都在云中"，2024 年首个美国客户在 Azure 上线【已核实，FIS 白皮书】：
  - https://www.fisglobal.com/-/media/fisglobal/files/pdf/white-paper/cloudenabled-core-banking-white-paper.pdf

### 7.3 AI/ML 模型在金融中的测试

- 与传统测试的本质差别：输入空间开放、分布会漂移（concept/data drift）、输出是概率而非确定值、需要**可解释性（XAI）与公平性**评估；监管上落入 SR 11-7/SR 26-2 的模型验证框架与"独立挑战"要求。
- 实时反欺诈/风控还要求在毫秒级决策链路中做冠军/挑战者（champion-challenger）、影子模式（shadow）与阈值回退测试。
- 工行设备反欺诈系统（设备指纹+机器学习+知识图谱+智能决策引擎）的建设实践可作一手参考【已核实，安全内参转载工行软件开发中心署名文章】：
  - https://www.secrss.com/articles/61720
- AI 银行应用综述（Springer，2022/2023，44 篇 SLR）反映行业关注点变化：
  - https://link.springer.com/article/10.1057/s41264-022-00176-7

---

## 8. 组织与流程挑战

1. **业务-科技-测试协同与领域门槛**：核心银行、清算、衍生品定价都需要多年领域知识；测试人员要懂会计分录、清算规则、监管口径，培养周期长。
2. **发布窗口少、变更冻结期长**：岁末年初、两会、重大会议、国庆/春节、"双十一"支付保障期等均有封网；需求堆积到窗口集中发布，放大风险。
3. **外包测试质量与责任不可外包**：TSB 案中 FCA/PRA 明确处罚了**外包治理失败**——银行不能因系统由供应商（Sabadell/Proteo）建设而免责；PRA 还依 SMCR 个人问责制罚了前 CIO。
   - FCA 通报（含外包风险表述）：https://www.fca.org.uk/news/press-releases/tsb-fined-48m-operational-resilience-failings
   - Norton Rose Fulbright 对 CIO 个人罚单的分析：https://www.nortonrosefulbright.com/en/knowledge/publications/5511e670/it-migration-and-outsourcing-lessons-from-the-latest-smcr-enforcement-case
4. **监管问责到人**：中国券商宕机罚单中，中信证券首席信息官与两名运维/机房负责人同收警示函（2023-07）【已核实，证券时报/财联社】：
   - https://www.stcn.com/article/detail/920153.html
5. **事件报告义务本身被考核**：首创证券 2021 年事件后，应急人员**删除日志与数据库信息且未备份，导致真实原因无法认定**；同花顺 2021 年故障后未及时报告，均单独受罚【已核实，每日经济新闻/中国基金报引述监管决定书】：
  - http://www.nbd.com.cn/rss/toutiao/articles/2209127.html
  - https://www.chnfund.com/article/AR2021102918154959212692

---

## 9. 量化金融特有测试

### 9.1 定价模型回测、P&L 验证、VaR 回测、XVA

- **模型验证的两类结果分析**（SR 11-7 框架）：
  - *Benchmarking*：与替代模型/第三方定价（vendor price）比较；
  - *Ongoing performance monitoring*：用已实现结果检验预测，例如 VaR 例外（exception/backtesting）。
- **VaR 回测的监管"红绿灯"**：巴塞尔框架以 250 个交易日、99% 置信度下约 1–4 次为例外绿区，例外过多触发资本加成（traffic-light approach）。这是把"模型测试"直接变成资本计量规则的典型。
  - 巴塞尔框架 MAR 章节（BIS 官方整合框架，含 backtesting 与 CVA 框架）：https://www.bis.org/basel_framework/
  - CVA 框架 MAR50（2023-01-01 生效）：https://www.bis.org/basel_framework/chapter/MAR/50.htm
- **P&L 验证（P&L attribution / IPV）**：交易账簿风险因子不可观察输入（IPV）、前端—风控—财务三套定价的对账（front office vs risk vs finance P&L），FRTB 还要求 P&L attribution test（NMRF 等），测试失败会导致模型资格问题。
- **XVA 家族**【已核实，BIS/IACPM 等】：CVA（对手方信用估值调整）、DVA、FVA（资金）、MVA（保证金）、KVA（资本）等使一笔衍生品的"正确价格"依赖蒙特卡洛模拟、抵押协议与资本规则，回归测试必须重放整条估值链并控制数值容差（蒙特卡洛噪声 vs 真实缺陷的区分）。
  - IACPM/Fintegral 白皮书指出 MVA/KVA 是当时参与机构面临的最大挑战：https://iacpm.org/wp-content/uploads/2018/06/IACPM-Fintegral-Making-the-Most-of-XVA-2018-White-Paper.pdf
  - BIS MAR50 对监管 CVA 与会计 CVA 差异的官方界定：https://www.bis.org/basel_framework/chapter/MAR/50.htm
- **测试特性**：数值型 oracle 多为容差比较而非精确相等；同一模型不同实现（重写迁移时）要做**数值对齐（numerical equivalence / parity testing）**；希腊值（Greeks）可用有限差分与伴随微分（AAD）交叉验证。

---

## 10. 近年真实故障案例库（按时间）

> 下表"证据等级"中，**一手**指监管/司法/机构文件；**媒体**指权威媒体报道但尚未取得一手公告。

| 日期 | 机构 | 事件与技术/测试根因 | 损失/处罚 | 证据 |
|---|---|---|---|---|
| 2012-03-23 | BATS（BZX 交易所） | 自家 IPO 开盘拍卖软件 bug：开盘订单不可访问，并触发 Apple 等股票错误成交，BATS 取消 IPO | IPO 撤销（无罚款；另 2015-01 Direct Edge 因订单类型描述不实罚 1,400 万美元，系不同事件） | BATS S-1 自述（一手）：https://www.sec.gov/Archives/edgar/data/1659228/000104746915009232/a2226675zs-1.htm ；SEC 2015-2：https://www.sec.gov/newsroom/press-releases/2015-2 ；WSJ：https://www.wsj.com/articles/SB10001424052702304636404577299560502440118 |
| 2012-05-18 | NASDAQ（Facebook IPO） | IPO 匹配设计缺陷；取消/改单积压超 2 小时；未经授权错误账户形成 300 多万股、约 1.29 亿美元空头；"Code Blue"期间在未搞清根因时恢复交易 | SEC 罚 **1,000 万美元**（当时对交易所最大罚单）；NASDAQ 另设 6,200 万美元补偿计划 | SEC 2013-95：https://www.sec.gov/newsroom/press-releases/2013-2013-95htm ；处罚决定书 34-69655 PDF：https://www.sec.gov/files/litigation/admin/2013/34-69655.pdf |
| 2012-08-01 | Knight Capital | SMARS 路由系统为 NYSE 零售流动性项目（RLP）升级时，**代码只人工部署到 8 台服务器中的 7 台**；残留的废弃 Power Peg 代码在第 8 台被标志位激活；45 分钟内为 212 笔零售小单发出 400 多万笔订单、成交 3.97 亿股；盘前 97 封报警邮件无人处理 | 公司 8-K 称**税前损失约 4.4 亿美元**；SEC 行政令称损失超 4.6 亿美元；2013 年依市场准入规则（Rule 15c3-5）罚 **1,200 万美元** | 8-K：https://www.sec.gov/Archives/edgar/data/1060749/000119312512332176/d391111dex991.htm ；SEC 处罚令 34-70694：https://www.sec.gov/files/litigation/admin/2013/34-70694.pdf ；SEC PR 2013-222：https://www.sec.gov/newsroom/press-releases/2013-222 |
| 2018-04-20/22 | TSB Bank | 核心银行平台迁移（Proteo4UK）切换后立即大面积故障，客户数周无法访问，数据虽迁移成功但平台技术失败；测试、容量、外包治理、回退均不足 | FCA+PRA 合计罚 **4,865 万英镑**（2022-12）；客户补偿 3,270 万英镑；总成本超 4 亿英镑（卫报口径） | 见第 2 节 FCA/PRA 链接 |
| 2020-10-01 | 东京证券交易所（arrowhead） | 共享磁盘装置（NAS）1 号设备故障后，主备切换因数据不一致未能完成；08:36 决定全天停市——史上首次因系统故障全天取消股票交易 | 全天停市（无直接罚款数字在本报告引用） | TSE 官方故障报告（一手，英文）：https://www.jpx.co.jp/english/corporate/news/news-releases/0060/b5b4pj000003r769-att/trading_system.pdf |
| 2020-10-23 | 欧元体系 TARGET2 | 网络设备软件缺陷导致 RTGS 中断近 10 小时；2020 年 TARGET Services 共 5 起重大 IT（非网络攻击）事件，暴露变更发布、业务连续性、故障切换与恢复测试、应急沟通短板；Deloitte 独立审查提 18 条建议 | 无罚款；18 项整改 | ECB 2020-11-16：https://ecb.europa.eu/press/pr/date/2020/html/ecb.pr201116~7b08f0a3c5.sk.html ；ECB 2021-07-28：https://www.ecb.europa.eu/press/pr/date/2021/html/ecb.pr210728~cb0848ee42.en.html |
| 2021-05-18 | 首创证券 | 集中交易系统部分中断约 20 分钟（达较大事件标准）；应急人员删除日志/数据库且未备份，根因无法认定，首次报告不如实 | 北京证监局责令改正 | 每经：http://www.nbd.com.cn/rss/toutiao/articles/2209127.html |
| 2021-10-28 | 同花顺 | App 大面积登录/行情异常；未及时向浙江证监局报告 | 浙江证监局警示函 | 证券时报：https://www.stcn.com/article/detail/463069.html |
| 2022-03-14 / 05-16 | 招商证券 | 3 月集中交易回报系统处理延迟（当日 280 万笔成交回报大延迟、影响约 48.7 万客户——上交所罚单口径）；5 月 PC/APP 均无法登录；监管认定**系统设计与升级变更未经充分论证和测试、压力测试场景不足、回退方案不完备** | 4 月深圳证监局责令改正；7 月证监会警示函（公司+3 名责任人） | 每经：http://www.nbd.com.cn/rss/toutiao/articles/2360908.html ；证券时报 3·15 综述：https://stcn.com/article/detail/1147408.html |
| 2022-03-08 | LME 镍 | 镍价数小时翻倍至 10 万美元/吨上方，LME 暂停交易并取消当日交易（名义约 120 亿美元）；Elliott/Jane Street 司法复核索赔（Elliott 主张利润损失约 4.56 亿美元） | 高等法院 2023-11-29、上诉法院 2024-10-07 均判 LME 合法 | LME 官方：https://www.lme.com/lme-nickel-litigation ；判决 PDF：https://www.lme.com/-/Media/Files/News/CO19952022-AC2022LON000890-CO20072022-AC2022LON000901-Final-Judgment-291123.pdf ；Reuters：https://www.reuters.com/business/hedge-fund-elliott-loses-appeal-against-dismissal-lme-nickel-lawsuit-2024-10-07 |
| 2023-03-21 | 东方财富证券 | 网络安全事件中"信息系统升级论证测试不充分"、未及时报告 | 西藏证监局责令改正 | 财联社：https://m.cls.cn/detail/1382094 |
| 2023-06-19 | 中信证券 | 早盘约 10 时交易系统瘫痪约半小时（挂单"已报待撤"、无法撤单）；监管认定机房基础设施安全性不足、设备可靠性管理疏漏；疑与周末升级/全网测试有关 | 深圳证监局对公司及 CIO 等 3 人警示函（2023-07-13） | 证券时报：https://www.stcn.com/article/detail/920153.html ；财联社：https://m.cls.cn/detail/1403890 |
| 2023-11-02/03 | The Clearing House（EPN/ACH） | 单个 ACH 文件处理人为错误：客户姓名与账号被掩码后发给银行，银行无法入账；影响约 85–90 万笔（< 当日全美 ACH 量 1%，当日约 1.3 亿笔），BofA/Chase/Wells Fargo/US Bank/Truist/PNC 等客户工资延迟数日 | 未见监管罚款（本报告未查到） | Reuters：https://www.reuters.com/business/finance/bofa-notifies-customers-temporary-industry-wide-deposit-delays-2023-11-03/ ；Yahoo/BI 数据：https://finance.yahoo.com/news/why-some-us-bank-deposits-are-held-up-days-after-processing-error-delayed-850000-payments-164651814.html ；TCH 一手信件链接（Fortune 引用）：https://media.theclearinghouse.org/-/media/New/TCH/Documents/Payment-Systems/ACH-Processing-Letter_Nov-06-2023.pdf |
| 2024-06-03 | NYSE | 早盘软件更新相关技术问题，行业级价格区间（Consolidated Tape Association 的 SIP/价格带）数据异常，错误触发约 40 只证券 LULD 熔断；Berkshire A 显示从约 62.7 万美元跌至 185.10 美元（-99%+）；NYSE 取消 603,718.30 美元以下的 BRK.A 成交 | 交易取消、当日复盘；本报告未查到 SEC 罚款 | CNN：https://www.cnn.com/2024/06/03/investing/new-york-stock-exchange-technical-issue ；NYT：https://www.nytimes.com/2024/06/03/business/nyse-trading-glitch-berkshire-hathaway.html ；Fortune（软件更新起因）：https://www.mailto:irena.raltcheva@fortune.com/2024/06/03/berkshire-stock-drop-nyse-glitch-malfunction/ |
| 2024-07-19 | CrowdStrike 生态（含多家银行） | Falcon 内容更新缺陷致 Windows 主机崩溃，银行 App/ATM/呼叫中心与资本市场受扰；非网络攻击 | 行业级间接损失；Delta 单家 >5 亿美元 | 见第 7.2 节 |
| 2025-01-31~02-02 | Barclays（英国） | 支付系统间歇性错误，部分客户近 24 小时无法访问账户/转账；恰逢发薪日与个税申报截止 | 服务中断、补偿承诺；本报告未查到监管罚款 | BBC：https://www.bbc.co.uk/news/articles/cd9qzg92g72o ；Guardian：https://www.theguardian.com/business/2025/feb/01/barclays-it-glitch-locks-customers-out-of-accounts-for-almost-24-hours |
| 2025-06-06 | NatWest | 手机 App 登录故障数小时；英国议会数据显示其 2023–2025 有 13 起"重大"事件、累计中断 194 小时、赔付约 34.8 万英镑 | 媒体口径赔付 | Guardian：https://www.theguardian.com/business/2025/jun/06/natwest-apologises-to-millions-of-customers-locked-out-of-app |
| 2025-11 | CME Group | 报道称数据中心故障导致 CME 多市场（股指、外汇、债券、商品）交易长时间中断 | **需进一步核实**：目前仅见 American Banker 2026-03 回顾性报道与 Pure Storage 二手报告，未取到 CME 官方 incident notice | American Banker：https://www.americanbanker.com/news/tech-outage-halts-trading-at-ubs |

### 中国案例的规律总结（已核实部分）
- 2022 年以来至少 9–11 家券商或行情/交易系统出现故障（中信证券、招商证券、东方财富、国元、华西、国信、首创、东兴、华宝等；行情系统涉及同花顺、大智慧、通达信、万得）【已核实，证券时报 2024-03-15 综述】：https://stcn.com/article/detail/1147408.html
- 监管处罚高频措辞高度一致：**"升级论证测试不充分""压力测试场景不足""回退方案不完备""未及时报告"**——与 Knight/TSB 的国际教训指向同一组根因（变更管理、环境真实度、容量、回退）。
- 行业应对：中证协《证券公司网络和信息安全三年提升计划（2023–2025）》要求券商信息科技平均投入不少于 2023–2025 平均净利润 10% 或平均营收 7%【已核实，证券时报引述】：https://www.stcn.com/article/detail/920153.html

### 需进一步核实 / 谨慎引用的案例
1. **工商银行 App 2025-10-29/30 资产"清零"显示异常**：多家门户转载"信号新闻"报道，称工行总行工作人员回应为偶发系统故障、仅资产负债数据显示异常、不影响真实资金。属**媒体来源、未见监管公告**，建议引用时标注。
  - https://sd.china.com/cjzx/20000936/20251101/25980116.html
2. **同花顺 2025-08-25、09-01、10-09 卡顿及 2026 年多次故障**：仅见新浪/21 财经等媒体与自媒体叙述，**未见监管文书**，具体日期与影响范围需以浙江证监局或交易所公告为准。
  - https://cj.sina.com.cn/articles/view/7879922977/1d5ae152106801glww
3. **CME 2025-11 宕机、LME 2026-03 撮合引擎故障、UBS 2026-03 交易中断、Lloyds 2026 串号显示**：目前仅 American Banker 单篇回顾提及，超出本报告核心时间窗，待一手公告。
  - https://www.americanbanker.com/news/tech-outage-halts-trading-at-ubs
4. **FTX**：属风控/客户资产挪用与治理失败（已破产、创始人被定罪），**不是软件测试失败案例**，不应混入本主题；如需引用应明确区分性质。
5. **英国邮政 Horizon 系统**：性质是系统缺陷+组织性错误追诉造成的司法丑闻，与"金融交易系统测试"不同类，但对"系统输出被默认为绝对正确、缺乏独立质疑机制"有极强警示意义。Horizon 公开调查第一卷最终报告 2025-07-08 发布（Sir Wyn Williams），称至少 13 人自杀可能与丑闻相关【已核实】：
  - 调查官网：https://www.postofficehorizoninquiry.org.uk/reports-and-statements
  - 第一卷：https://www.postofficehorizoninquiry.org.uk/inq00002037-volume-1-post-office-horizon-it-inquirys-final-report

---

## 11. 按测试活动归纳：难点 → 行业做法 → 仍未解决

| 测试活动 | 在金融场景的特殊难点 | 主流做法（有证据） | 仍未解决 |
|---|---|---|---|
| 单元/组件测试 | 金额舍入、计息日历、状态机 | 定点十进制、属性测试/蜕变测试（IEEE 2023 银行 MT 论文）、黄金文件 | 数值容差与 MC 噪声区分 |
| 集成/契约测试 | 上下游数十、沙箱不稳 | 服务虚拟化、CDC 契约、容器化环境（ING CNCF 案例） | 异步事件时序、第三方行为保真 |
| 系统/E2E | 核心+渠道+清算全链路 | 双核心并行对账（TSB 反面教训）、交易所全网测试 | 生产规模数据与极端场景 |
| 性能/容量 | 开盘洪峰、极端行情 | 全链路压测、周末全网演练 | 真实"秒杀"流量模型、容量与成本平衡 |
| 混沌/灾备 | 涉账系统不敢在生产炸 | 先测试后生产、最小爆炸半径（工行、北京银行、JPMC、Monzo Stand-in） | 多地/多供应商同时失效演练 |
| 数据管理 | 合规与保真矛盾 | 静态脱敏+子集+合成数据、生命周期分级（JR/T 0197/0223） | 脱敏后分布保真、跨表依赖完整 |
| 模型验证 | 概率输出、漂移 | 独立验证、backtesting、benchmark、champion/challenger（SR 11-7） | 非结构化模型（生成式 AI）验证标准 |
| 验收/回归 | 领域知识门槛高 | 厂商 Model Bank 测试资产（Temenos/Tricentis）、低代码框架（ING INGenious） | 监管规则频繁变更下用例维护 |
| 发布/回退 | 窗口少、冻结期长 | 灰度、蓝绿、可回滚内容更新（CrowdStrike 教训）、变更冻结期管控 | 大机批处理型系统的灰度天然困难 |
| 事后/合规 | 证据链与问责 | 强制事件报告（证监会 12 号令）、SMCR 个人问责、审计留痕 | 瞒报/删日志（首创证券案）仍需靠监管威慑 |

---

## 12. 核心来源清单（按类型）

### 监管/央行/司法（一手）
- SEC：Knight 34-70694、NASDAQ 34-69655、BATS/Direct Edge 2015-2、Knight PR 2013-222（链接见第 10 节）
- FCA/PRA/BoE：TSB 罚单与 Final Notice（第 2 节）
- ECB：TARGET2 事件与 Deloitte 独立审查（第 10 节）
- Federal Reserve：SR 11-7、SR 26-2、FedNow（第 1、5 节）
- BIS：Basel Framework MAR50 CVA（第 9 节）
- EUR-Lex：MiFID II RTS 6 (2017/589)、RTS 7 (2017/584)、GDPR (2016/679)
- 中国证监会：公告〔2024〕8 号程序化交易规定；〔2021〕12 号网络安全事件报告办法（媒体引述文号）
- 人行金融行标：JR/T 0171—2020、JR/T 0197—2020、JR/T 0223—2021
- 全国人大：个人信息保护法（2021）
- JPX/TSE：2020-10-01 arrowhead 故障英文报告
- LME/英国法院：镍交易取消判决书（2023 EWHC 2969；2024 EWCA Civ 1168）
- CrowdStrike 官方事故说明；美国国会 CRS R48135/IN12392
- Post Office Horizon Inquiry 官方报告页

### 机构工程博客/厂商一手
- Goldman Sachs Developer（observability、SecDb）；GitLab GS 案例
- JPMorganChase Technology blog（JET、云扩展）；Grafana ObservabilityCON 议程
- Monzo 工程博客（Stand-in、EKS 迁移）；Starling Technology/Engine by Starling
- ING（CNCF 案例、INGenious、ING blog）
- Temenos（Interim Report 2023、Workbench、Transact）；FIS（Profile cloud 白皮书）；IBM（mainframe、CIBC、IBV）
- Luxoft/Tricentis（Temenos 测试白皮书）；北京银行/工行（混沌工程，经《金融电子化》/InfoQ 发表）

### 学术
- IEEE IRI 2023《An Approach to Testing Banking Software Using Metamorphic Relations》
- ACM/IEEE FinTech-SE 工作坊 2026《LLM-Assisted Retro-Documentation for Legacy COBOL Applications in Banking》
- 微服务测试 SLR（Information & Software Technology, 2025）
- Crépey 等 XVA 学术论文（BIS MAR50 为监管口径）

### 权威媒体（用于事件经过，均已标注）
Reuters、NYT、WSJ、BBC、Guardian、CNN、CNBC、Fortune、American Banker；
中国：证券时报·券商中国、财联社、每日经济新闻、中国基金报、21 世纪经济报道。

---

## 13. 使用提醒（避免引用错误）

1. **Knight 的金额有两个口径**：公司 8-K 新闻稿为"约 4.4 亿美元税前损失"；SEC 2013 年行政令正文为"超 4.6 亿美元交易损失"。严谨写作应写"约 4.4 亿（公司披露）/超 4.6 亿（SEC 认定），1,200 万美元和解金"，不要只写一个数。
2. **RTS 6 与 RTS 7 不要混**：公司侧算法测试义务在 2017/589（RTS 6）Article 5；场所侧一致性测试设施在 2017/584（RTS 7）。
3. **BATS 2012 IPO 没有 SEC 罚款**；2015 年 1,400 万美元罚的是 Direct Edge 的订单类型规则描述问题（并购后实体相关但不同事由）。
4. **TSB 罚单主体拆分**：总额 4,865 万英镑 = FCA 2,975 万 + PRA 1,890 万；另有 3,270 万客户补偿；不要把补偿算入罚款。
5. **中国券商故障的时长（如"半小时"）多为投资者感知口径**，监管分级以正式事件调查为准；引用以证监局决定书措辞最稳。
6. 2025 年后的部分银行/交易所宕机（CME、UBS、LME、工行显示异常、同花顺 2025/2026 事件）**本报告发布时一手文件尚不完整**，对外使用前应再查交易所 status/incident 页与监管公告。
