# 金融合规验证机制 与 AI 时代验收手段变革（调研笔记）

> 调研日期：2026-10-27。两份子代理调研合并存档。
> 证据分级：【已核实】= 官方域名/一手文件；【待核实】= 仅二手，已显式标注。
> 严禁编造：金额/文号/日期均以下列来源为准。

## A. 合规是怎么验证的

### A1. 三道防线
- IIA 2013 立场声明《Three Lines of Defense》；2020 更新《The IIA's Three Lines Model》（2020-07-20 发布）。
  https://www.theiia.org/globalassets/site/communication/2020/three-lines-model-updated.pdf
- 一线=业务/运营（控制首要责任）；二线=风险/合规/安全（独立监测）；三线=内审（独立客观保证，直达审计委员会）。
- BCBS《The internal audit function in banks》(2012, bcbs223)；《Compliance and the compliance function in banks》(2005, bcbs113)。
- 中国成文化：《金融机构合规管理办法》（金融监管总局令 2024 年第 7 号，2025-03-01 施行）第 35 条——业务/职能部门主体责任、合规部门管理责任、内审监督责任。
  https://www.gov.cn/gongbao/2025/issue_11866/202502/content_7004030.html

### A2. 控制测试方法论
- RCM 风险控制矩阵：法规义务→控制点→控制类型(预防/检查、人工/自动)→频率→责任人→测试方法→设计/运行结论→缺陷等级（行业方法论共识，非单一注册商标）。
- COSO 2013 内控整合框架：五要素、17 原则（原则16 持续/单独评价、17 缺陷沟通）。
- PCAOB AS 2201：设计有效性(.42-.43) vs 运行有效性(.44)；**仅询问 inquiry alone 不足以证明运行有效**。
  https://pcaobus.org/oversight/standards/auditing-standards/details/AS2201
- PCAOB AS 2315 审计抽样；AS 2301 控制测试；发现偏差三处理（扩样、测补偿性控制、判无效+加实质性程序）。
- 缺陷分级：控制缺陷 / 重要缺陷 Significant Deficiency / 重大缺陷 Material Weakness（一项重大缺陷即 ICFR 否定意见 Adverse Opinion）。

### A3. 关键领域
- KYC/CIP（爱国者法案326）；FinCEN CDD Final Rule（2016/2018，受益所有人）；CTA BOI（2024 生效，2024 曾被法院暂缓）。
- 中国《受益所有人信息管理办法》（人行、市监总局令〔2024〕第3号，2024-11-01）【文号待官网复核】。
- AML：FFIEC BSA/AML 手册——独立测试(Independent Testing)为法定支柱，独立测试充分时检查官可减少覆盖。
  https://bsaaml.ffiec.gov/manual/AssessingTheBSAAMLComplianceProgram/03
- NYDFS 3 NYCRR 504（2016）：交易监控与过滤项目+每年4/15前年度认证；OCC M.Y. Safra 同意令 EA-2020-005（模型与阈值定期独立验证）。
- FINRA Rule 3310 AML；年报高频缺陷=独立测试不充分。
- Reg BI（34-86031，2020-06-30 合规）四义务：披露/注意 Care/冲突/合规；执法 David Lerner Associates（3-22645，2026-05-27）。
- FCA Consumer Duty（PS22/9）：在售 2023-07、封闭产品 2024-07；仅收集数据/列指标**不足以**证明良好结果。

### A4. 独立鉴证与认证
- SOC1（ICFR/财务报告相关，SSAE18 AT-C320）/ SOC2（TSC：安全/可用/处理完整/保密/隐私）/ SOC3（公众可分发、不含测试明细）；各有 Type1(设计)/Type2(设计+运行)。
- PCI DSS v4.0(2022)/v4.0.1(2024-10)：ROC(QSA 现场) / SAQ(自评，多类型) / AOC(合规声明)。
- ISO/IEC 27001:2022（Annex A 93 项、SoA、9.2 内审）。
- 中国等保：网安法21/31条；GB/T 22239-2019、GB/T 28448-2019(测评)、GB/T 22240-2020(定级)；流程 定级→备案→建设整改→等级测评(持证机构)→监督检查。

### A5. 报送数据与现场检查
- BCBS 239（2013）风险数据汇总与报告14原则，G-SIB 2016 全面实施，强调 independent validation。
  https://www.bis.org/publ/bcbs239.pdf
- 美联储 SR 13-13/CA13-10 附件：MRA(Matters Requiring Attention) vs MRIA(Immediate)；需验证满意整改方可关闭。
- FINRA 年度监管报告（2026 报告 2025-12-09 发布）。

### A6. RegTech / SupTech
- FCA+BoE 数字监管报送 DRR 试点（2017 TechSprint；Phase1 2018；Phase2 2019，7家银行）；MRR(机器可读)/MER(机器可执行)。
  https://www.fca.org.uk/innovation/regtech/digital-regulatory-reporting
- CCM 持续控制监测；FSB SupTech/RegTech 报告（2020-10）https://www.fsb.org/uploads/P091020.pdf
- FINRA RegTech 报告（2018）。

### A7. 合规失败真实处罚（均可核查）
- HSBC 2012-12 DPA：DOJ 没收12.56亿美元，合计约19.2亿美元（BSA/IEEPA/TWEA）。
- BNP Paribas 2014：认罪，89.736亿美元（88.336亿没收+1.4亿罚金），OFAC另结9.63亿。
- Danske Bank 2022-12：SDNY 共谋银行欺诈认罪，DOJ没收20.59亿美元；SEC和解约4.13亿（Release 2022-220）。
- SEC off-channel（WhatsApp 等未留痕，Rule17a-4/交易所法17(a)）：2022-09 十六家合计>11亿（2022-174）；2023-08 十一家2.89亿（2023-149）；2023-09 十家7900万（2023-212）。
- Bank of America 2026-10：FinCEN 12亿(净付约4.5亿)、OCC 7.5亿(史上最高)、OFAC 1166.293万；SAR lookback 自2022起，漏报约8900份/近400亿交易；独立测试未发现缺陷是关键。
  https://www.fincen.gov/news/news-releases/fincen-assesses-12-billion-penalty-against-bank-america
- SEB 2026-10-15 NYDFS 罚3300万美元（AML/监控积压）。
- 首创证券删日志：北京证监局〔2022〕64号（落款2022-04-01），违反证监会令第152号第36、54条；2021-05-18中断约20分钟，删日志/数据库未备份致根因无法认定。
  https://www.csrc.gov.cn/beijing/c105547/c2319935/content.shtml

### A8. 电子证据
- SEC Rule 17a-4 传统要求 WORM（不可重写/不可擦除）；2022-10 修订（34-96034，合规日2023-05-03）新增 audit-trail alternative（可重建原始记录+时间戳完整审计追踪）。
  https://www.sec.gov/files/rules/final/2022/34-96034.pdf
- Scottrade AWC（2015-11-16）未按 WORM 保存。

### A9. 合规验证 vs 功能测试根本区别
- 目标：功能正确 vs 满足外部强制义务且控制真实运行；基准：需求文档 vs 法律法规；证据：用例记录 vs 可呈堂证据链；
  独立性：可自测 vs 必须独立（二/三线/第三方）；时间：发布前 vs 持续/回溯 lookback/CCM；风险导向；必须分设计/运行有效性；
  失败：修缺陷 vs 罚款/刑责/个人禁业/牌照；结论：pass/fail vs 合理保证/审计意见。

---

## B. AI 时代验收手段的改变

### B1. 欧盟 AI 法案（Reg (EU) 2024/1689）
- 2024-06-13 登 OJ、2024-08-01 生效。
- **时间表（重要：已被 AI Omnibus 推迟）**：Reg (EU) 2026/1744（2026-07-27 生效，政治协议2026-05-07）：
  - 2025-02-02 禁止类(Art.5)+AI素养；2025-08-02 GPAI+治理+罚则；2026-08-02 法案总体适用+透明度(Art.50)执法；
  - **Annex III 高风险推迟至 2027-12-02**；Annex I 嵌入式产品高风险推迟至 **2028-08-02**；新增第9项禁止 2026-12 适用。
  https://digital-strategy.ec.europa.eu/en/news/ai-omnibus-enters-force
- 四级风险；金融高风险：Annex III 5(b) 信用度/信用评分（**反欺诈除外**）、5(c) 人寿与健康保险风险评估与定价。
- 高风险七义务（法定验收清单）：风险管理(Art.9)、数据治理(Art.10)、日志可追溯(Art.12)、Annex IV 技术文档、对部署方信息(Art.13)、人工监督(Art.14)、准确性稳健性网安(Art.15)；上市前合格评定(Art.43)+CE+EU数据库+上市后监测(Art.72)+严重事件报告(Art.73)。
- GPAI：技术文档/版权政策/训练内容摘要；系统性风险(>10^25 FLOP 推定)；GPAI Code of Practice(2025-07-10)。
- EBA 2025-11-20《AI Act implications for EU banking sector》。

### B2. NIST
- AI RMF 1.0（AI 100-1，2023-01）四功能 GOVERN/MAP/MEASURE/MANAGE；TEVV。
- GenAI Profile（AI 600-1，2024-07）12 类风险（含 confabulation 幻觉、提示注入）、200+ 行动。
- 2025 回摆：EO14110 被撤、America's AI Action Plan(2025-07) 令修订 AI RMF（修订文本未发布）。

### B3. ISO
- ISO/IEC 42001:2023（AIMS 可认证）；23894:2023（AI 风险指南）；**TS 4213:2022（ML 分类性能评估：准确率/F1/混淆矩阵/ROC 使用原则，正转 DIS）**；5259 数据质量；TS 5723:2024 偏见减少。
- 2026-10-19 首批 AI 法案协调标准登 OJ。

### B4. 中国
- 算法推荐规定（四部门令第9号，2022-03-01）；深度合成规定（令第12号，2023-01-10）；生成式暂行办法（七部门令第15号，2023-08-15）。
- 备案：截至2025-12-31 累计748款生成式服务备案、435款应用登记，须公示备案号。
- TC260-003《生成式人工智能服务安全基本要求》（2024-02-29）= 备案技术基线：语料/关键词库/**生成内容测试题库+拒答测试题库**/31种风险；已升 GB/T 45654-2025（2025-11-01 实施）。
- 金融行标：**JR/T 0221—2021《人工智能算法金融应用评价规范》**（2021-03，安全性/可解释性/精准性/性能四维，含对抗/逆向攻击测试）——标准号名称已核实。
  https://cfstc.pbc.gov.cn/bzgk/detail?bzId=1912&id=0
- JR/T 0263—2022 机器学习金融应用技术指南；**JR/T 0287—2023 AI算法金融应用信息披露指南**（七维披露，中国版模型卡）。
- 金融监管总局《关于银行业保险业人工智能安全开发应用的指导意见》（金发〔2026〕8号【文号待官网复核】）：全生命周期、分类分级、谁使用谁负责、董事会专委会。
- 证监会李超 2025-10-29：关键环节人类把关、最终决策由人作出。

### B5. 验收方法差异（传统 vs ML）
- 对象：代码符合规格 vs 模型行为/数据/性能分布；判据：确定用例 vs 指标分布+阈值容差+群体差异；数据：固定 vs 训练/验证/测试独立+泄漏检查；时点：一次 vs 上市前+持续再验证；缺陷：功能bug vs 偏见/不可解释/OOD/对抗/幻觉；验收方：UAT vs 独立验证(effective challenge)。
- 指标：ISO TS4213 流程；信贷除 AUC/KS 外须报告校准度 ECE；EBA《ML for IRB Models Follow-up》(2023-08)。
- 公平性：ECOA/Reg B + HMDA；CFPB Circular 2022-03（黑箱不免具体拒贷原因义务）；Upstart 独立监督(2020-2024，LDA 分析)；
  **2025-2026 回摆**：CFPB 2026-04-22 Reg B 终规明确 ECOA 不授权 disparate impact——但欧/中方向相反。
- 案例：Apple Card NYDFS 调查 2021-03 报告未发现违法歧视但呼吁模型现代化；DOJ v. Meta 住房广告算法(2022，VRS 方差削减系统+量化指标+第三方验证)。
- 稳健性：JR/T0221 第6章对抗/逆向攻击测试；NIST 2024-01 对抗 ML 指引。
- 部署：shadow、champion-challenger、金丝雀、A/B、回退（EBA 2021；FCA ML research note）。
- LLM 特有：groundedness/幻觉率（NIST 600-1）；Morgan Stanley+OpenAI（RAG 接地到约10万份已审核研报+人工核验，2023-09 全员）；金标准/拒答集（TC260）；提示注入/越权投资建议；FINRA RN 24-09（2024-06-27）；ESMA 2024-05-30 声明（人工监督、数据准确充分）。

### B6. 持续验证
- SR 11-7(2011) 三要素（概念合理性/持续监测/结果分析）+模型库存+有效挑战；
  **SR 26-2（2026-04-17，OCC2026-13）取代 SR11-7 与 SR21-8，主要适用总资产>300亿美元银行组织，明确不设强制标准；关键：把生成式 AI 与智能体 AI 排除在范围外**——GenAI 验收落入技术中立规则(FINRA3110、顾问法206、Marketing Rule)+自愿框架。
- 模型清单：EU 数据库；BoE/FCA 2024 调查（75%已用AI、仅2%完全自主、仅34%完全理解所用AI、1/3用例第三方交付）。
- 模型卡 Model Cards(Mitchell 2019)/数据卡 Datasheets(Gebru, CACM 2021) ↔ EU Annex IV / JR-T 0287。

### B7. 监管检查新做法
- DORA(EU 2022/2554，2025-01-17 适用) + Delegated Reg (EU) 2025/1190：一般 ICT 测试至少每年；**TLPT 威胁导向渗透测试至少每3年，在活体生产系统红队**（TIBER-EU 2025-02 更新）。
- IOSCO《Supervisory Toolkit for AI Use in Capital Markets》FR/02/2026（2026-05-25，三层工具）。
- FSB：AI 金融稳定影响(2024-11)；AI 采用监测(2025-10)；**《Sound Practices for Responsible Adoption of AI》最终报告(2026-10-15，12项实践)**。
- 美国回摆：SEC 2025-06-12 撤 PDA 规则等14项；206条/Marketing Rule 执法继续。
- 英国 PRA SS1/23（2024-05 生效）。

### B8. 人在回路如何可验证
- AI Act Art.14(设计可监督)/Art.13(正确使用说明)/Art.26(部署方监督义务)；证据=监督程序、干预/停用 kill switch、报警阈值、培训记录。
- 可核查落地证据：override 率日志、升级工单、影子比对记录、输出与最终决策留痕映射、停用演练。
- 中国：算法推荐规定第11条人工干预机制；证监会2025讲话。

### B9. 范式迁移十转变
时点(一次UAT→分类+评定+上市后监测)、对象(代码→模型+数据+标注+基础模型供应链+人因)、判据(功能→分布/校准/公平/稳健/可解释/幻觉多门限)、
主体(自测→独立验证+公告机构+监管备案)、环境(测试环境→生产活体TLPT/影子/挑战者)、文档(手册→Annex IV/模型卡/数据卡/七维披露)、
监控(无→漂移/衰减/群体差异/事件再验证)、资产(系统清单→含第三方嵌入式模型全量清单/备案号)、人(例外处理→制度化监督与override留痕)、
地缘(欧盟硬法/美国去规则化但MRM仍在/中国备案+金融评价双轨，FSB/IOSCO汇合)。
