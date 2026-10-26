# PSP 完整体系调研：从 Humphrey 的过程纪律到 AI 编程时代

> 调研日期：2026-10-26
> 说明：本文所有数据尽量来自一手原始资料（SEI 技术报告、论文原文、官方出版物），并标注报告编号与页码。凡未能取得原文核实的内容（主要是 Humphrey 四本书的具体页码与部分网页正文），均标注"待确认"。

---

## 目录

- [第一部分：Watts S. Humphrey 与 PSP 的诞生背景](#第一部分watts-s-humphrey-与-psp-的诞生背景)
- [第二部分：PSP 完整体系详解](#第二部分psp-完整体系详解)
- [第三部分：TSP —— 从个人到团队](#第三部分tsp--从个人到团队)
- [第四部分：PSP 在 AI 编程时代的适用性](#第四部分psp-在-ai-编程时代的适用性)
- [参考资料](#参考资料)

---

## 第一部分：Watts S. Humphrey 与 PSP 的诞生背景

### 1.1 人物简介

Watts S. Humphrey（1927-07-04 ～ 2010-10-28），美国软件工程先驱，被称为"软件质量之父"（Father of Software Quality）。

- 二战期间在美国海军服役；战后在芝加哥大学获物理学学士学位，曾受教于 Enrico Fermi；后在伊利诺伊理工学院获物理学硕士学位（具体学位信息见 SEI 官方讣告）。
- 在 IBM 工作 27 年，曾管理 IBM 多个软件/工程实验室。
- 1986 年加入卡内基梅隆大学软件工程研究所（SEI），成为首位 SEI Fellow，建立软件过程项目（Software Process Program）。
- 2003 年获美国国家技术奖章（National Medal of Technology）；授奖词肯定其"对工程纪律的远见及其对软件产业的影响"。

> 来源：SEI 官方讣告《National Medal of Technology Winner Watts Humphrey, 1927-2010》（sei.cmu.edu，2010-10-28）；SEI 作者页 sei.cmu.edu/authors/watts-humphrey；National Science and Technology Medals Foundation（nationalmedals.org/laureate/watts-humphrey）。

### 1.2 从 CMM 到 PSP 的逻辑

Humphrey 在 SEI 领导开发了软件能力成熟度模型（CMM，1987 年首次发布五级框架），CMM 面向**组织层面**，回答的是组织"应该具备哪些过程能力"的问题。但他很快发现一个缺口：

> "PSP 和 TSP 之所以被开发出来，是为了向个人开发者及其团队展示如何在自己的个人工作中应用这些 [CMM] 原则。"

> 来源：Computerworld《Sidebar: Watts Humphrey on Software Quality》，2004-03-08。

TR-022 记录了 PSP 的直接起源：在开发模块级小程序的场景下，Humphrey **亲自把软件 CMM 中直到第 5 级的所有实践全部使用了一遍**，以检验这些组织级原则缩放到个人层面是否可行、如何落地。

> 来源：CMU/SEI-2000-TR-022《The Personal Software Process (PSP)》，第 1 章（亦见 DTIC ADA387268 文本）。

时间线：

| 年份 | 事件 |
|---|---|
| 1986 | Humphrey 加入 SEI，创立软件过程项目 |
| 1987 | 首版 CMM 成熟度框架发布 |
| 1989 | 出版《Managing the Software Process》 |
| 1993 左右 | Humphrey 在 SEI 课程与个人实践中试验 PSP |
| 1995 | 出版《A Discipline for Software Engineering》（PSP 教材）；PSP 正式提出 |
| 1996 | 开发 TSP 初始版本；《IEEE Software》13(3): 77-88 发表 "Using a Defined and Measured Personal Software Process" |
| 1997 | 出版《Introduction to the Personal Software Process》；《Computer》30(5): 24-31 发表 Ferguson 等 "Results of Applying the Personal Software Process" |
| 2000 | 发布 CMU/SEI-2000-TR-022（PSP）、TR-023（TSP） |
| 2002 | 出版《Winning with Software: An Executive Strategy》 |
| 2005 | 出版《PSP: A Self-Improvement Process for Software Engineers》；出版《TSP: Leading a Development Team》 |
| 2009 | 发布 PSP Body of Knowledge v2 |
| 2010 | 逝世；身后出版《Leadership, Teamwork, and Trust》 |

> 来源：SEI 官网出版物页；IEEE Software Vol.13 No.3 目录（sigmod.org 镜像 dblp 数据）；ACM Digital Library 书籍条目。各书具体章节页码：待确认（未获取到电子书全文）。

### 1.3 PSP 的核心哲学

TR-022 明确列出 PSP 设计所依据的计划与质量原则（第 2 章 "PSP Principles"），核心可归纳为：

1. **每个人都是不同的**：工程师必须使用自己的历史数据来做计划和估算，别人的数据只在自己没有数据时作初始参照。
2. **个人责任**：要生产高质量产品，工程师必须对产品质量感到个人负责。"优质产品不是偶然产生的；工程师必须努力做好质量工作。"
3. **数据驱动的自我改进**：过程改进的基本输入是个人的时间（time）、规模（size）、缺陷（defect）三类基础度量；改进循环依赖"定义过程 → 度量执行 → 分析偏差 → 调整过程"。
4. **质量不是靠测试测出来的**："任何已发明的测试方法都只能识别大型复杂产品中一小部分缺陷，产品越大越复杂，比例越小。要想从测试中得到高质量产品，就必须把高质量产品送进测试。"
5. **零缺陷目标**："工程师使用 PSP 时，推荐的过程目标是按进度、在计划成本内生产零缺陷产品。"（TR-022 摘要）

> 来源：CMU/SEI-2000-TR-022 第 2、6 章；CMU/SEI-2006-TR-017（质量三断言，见 2006-TR-017 第 3 章附近）。2006-TR-017 原文措辞见第 34 页（报告内页码）。

Humphrey 在 2009 年计算机历史博物馆（CHM）口述史中对"质量"的定义：

> "质量是按客户想要的方式完成工作。我同意戴明、朱兰和其他质量大师的定义。质量就是尽一切努力真正满足客户。"

他同时以"缺陷密度"为尺度给出他个人的质量标杆判断：**每千行代码 5 个缺陷就算差质量**，质量人应该谈的是每百万件的缺陷数；并提到他手中有"数千名工程师的缺陷数据"。

> 来源：Computer History Museum，Oral History of Watts Humphrey，2009-12-17（archive.computerhistory.org，文档编号 102702107.05.01.acc.pdf）。

---

## 第二部分：PSP 完整体系详解

### 2.1 总体结构：七个过程版本、一条渐进路径

PSP 的方法不是一次性全部灌输，而是通过 **7 个过程版本（PSP0 → PSP0.1 → PSP1 → PSP1.1 → PSP2 → PSP2.1 → PSP3）** 渐进引入。每一级都包含类似的日志（logs）、表单（forms）、脚本（scripts）与标准（standards），新版本在旧版本上增量叠加。

| 版本 | 新增内容 |
|---|---|
| **PSP0** | 当前个人过程；基本度量——时间记录（按阶段）、缺陷记录日志、规模度量；项目计划摘要（Project Plan Summary） |
| **PSP0.1** | 编码标准（coding standard）；规模度量方法；过程改进提案（PIP, Process Improvement Proposal） |
| **PSP1** | 基于 PROBE 的规模估算（size estimating）；测试报告 |
| **PSP1.1** | 任务计划（task planning）与进度计划（schedule planning） |
| **PSP2** | 代码评审（code reviews）；设计评审（design reviews） |
| **PSP2.1** | 设计模板（design templates） |
| **PSP3** | 循环式开发（cyclic development），把个人过程扩展到较大规模程序 |

> 来源：CMU/SEI-2000-TR-022 图 5 "PSP/TSP Course Structure"（报告第 29 页，PDF 第 43 页）；第 3 章对各版本表单/脚本的描述。PSP0 与 PIP 的细节另见 CMU/SEI-97-TR-001《Introduction to the Personal Software Process》。

### 2.2 课程形态：10 个练习程序

- **高校高阶课程**：学生编写 **10 个程序**、完成 **5 份数据分析报告**；通常持续一整学期；课程总投入约 **120–150 工程小时**，一般分布在 **14 个工作日**以上（工业课程排期）。
- 学生从 PSP0 起步（保留自己当前的编程习惯，只增加时间与缺陷记录），每个 PSP 版本写 1–2 个程序，逐级叠加，课程结束时写满 10 个程序并使用全部 PSP 方法。
- 另有面向初学者的入门课程（基于《Introduction to the Personal Software Process》，1997），要求不如高阶课程严格，重在从第一次作业起养成习惯。

TR-022 第 10 章报告的核心效果数据，来自 **SEI 工业课程 298 名学生的平均值**（对应练习共产生 2,386 个程序版本）。

> 来源：CMU/SEI-2000-TR-022 第 9 章（PDF 第 43-44 页，报告第 29-30 页）、第 10 章（"All these data are averages for 298 students in SEI industrial courses"）。

### 2.3 基础度量：时间、规模、缺陷

PSP 的一切分析建立在三类基础数据上（与 TR-022 第 4 章及《The Software Quality Profile》所述一致）：

1. **时间（time）**：按过程阶段记录每个任务的实际分钟数（如计划、设计、设计评审、编码、代码评审、编译、单元测试、复盘）。
2. **规模（size）**：以新增及修改的代码行（new and changed LOC）为基本规模单位，要求可自动化测量；估算通过 PROBE 完成。
3. **缺陷（defect）**：每个缺陷记录类型、注入阶段、移除阶段、修复时间等。

**PROBE（PRoxy-Based Estimating，基于代理对象的估算）**：工程师先识别待建对象（面向对象语言中以对象及其方法的相对大小为代理；过程式语言中以函数/过程为代理），用历史数据把代理的相对规模换算成 LOC。代理必须与工作量相关、能在计划阶段估算、最终可自动测量。

> 来源：CMU/SEI-97-TR-001 对 PROBE 的描述（"PROBE stands for PRoxy-Based Estimating"）；CMU/SEI-2000-TR-022 第 4-5 章规模与计划部分。

### 2.4 质量管理：评审杠杆率与关键指标

PSP 质量论点的核心是**早期阶段的缺陷移除效率远高于测试**。TR-022 第 6 章（报告第 20-22 页）给出 298 名学生的关键数据：

**缺陷发现效率（个/小时）——TR-022 Table 4：**

| 阶段 | 缺陷发现率（个/小时） |
|---|---|
| 设计评审（Design Review） | **2.96** |
| 代码评审（Code Review） | **6.52** |
| 单元测试（Unit Test） | **2.21** |

缺陷注入率：详细设计阶段平均注入 **1.76 个/小时**；编码阶段平均注入 **4.20 个/小时**。

据此可推出评审的"杠杆"：设计评审发现缺陷的效率约为单元测试的 3.06/1.71 ≈ **1.79 倍**（TR-022 原文计算值）；在 2006 年的质量报告语境中，Humphrey 反复强调"先聚焦质量，才能最大化生产率"。

**评审速度准则**：PSP 数据表明，当工程师以快于约 **150–200 新增/修改 LOC/小时** 的速度评审设计或代码时，会漏掉大量缺陷。

> 来源：CMU/SEI-2000-TR-022 第 6 章（PDF 第 32-33 页；Table 4 在报告第 21 页）。评审速度原文："when engineers review designs or code faster than about 150 to 200 new and changed LOC per hour, they miss many defects"。

**时间比例准则：**

- 设计评审时间 / 设计时间：课程中据注入/发现率推算约为 60%（1.76 vs 2.96）；
- 代码评审时间 / 编码时间：建议约 50% 以上（按 4.20 vs 6.52 推算约 65%，PSP 取 50% 作为近似准则）；
- 缺陷比率准则：代码评审发现缺陷数 / 编译发现缺陷数应 ≥ 2；设计评审发现缺陷数 / 单元测试发现缺陷数应 ≥ 2。

> 来源：CMU/SEI-2000-TR-022 第 6 章（"The PSP data also suggest that the design review to unit test defect ratio be two or greater"）。

### 2.5 Yield 与 A/FR：两个核心质量指标

**Yield（收益率）**，分两种：

- *阶段 yield（phase yield）*：进入该阶段时存在的缺陷中，被该阶段移除的百分比。例如进入单元测试时有 20 个缺陷、测试发现 9 个，则阶段 yield 为 45%。
- *过程 yield（process yield）*：在编译之前移除的缺陷占全部缺陷的百分比。**PSP 建议的过程 yield 准则为 70% 以上。**

Yield 在程序生命周期结束前无法精确计算，但可用"假设残留缺陷数 ≈ 后置测试发现缺陷数"等方法做早期估计。

**A/FR（Appraisal to Failure Ratio，鉴定/故障成本比）**，基于朱兰的质量成本框架：

- A（鉴定成本）：花在质量鉴定活动上的时间占比，PSP 中主要是设计评审与代码评审（含修复评审发现缺陷的时间）；
- F（故障成本）：花在故障恢复上的时间，即编译、单元测试等阶段处理缺陷的时间；
- **PSP 课程要求工程师按 A/FR ≥ 2.0 做计划。**

> 来源：CMU/SEI-2000-TR-022 第 6 章（process yield "suggested guideline ... is 70% better"；"engineers are told to plan for A/FR values of 2.0 or higher"）。BOK 2009 另列 5.2.8 Percent appraisal COQ、5.2.11 COQ A/FR 等指标条目。

### 2.6 缺陷预防

Humphrey 认为"管理缺陷最有效的方法是从一开始就预防其注入"，PSP 的预防手段有三：(1) 记录并分析每个缺陷的数据（含 PIP）；(2) 使用有效的设计方法与表示法产出完整设计；(3) 设计越彻底，编码时间越短，从而减少编码阶段（注入率最高，4.20 个/小时）的缺陷注入。

> 来源：CMU/SEI-2000-TR-022 第 6.4 节 "Defect Prevention"（报告第 25-26 页附近，PDF 约第 38-39 页；引文段落 "PSP data for 298 ..." 在 PDF 第 39 页）。

### 2.7 PSP 的实测效果（一手数据）

**(1) TR-022 引用的 Hayes & Over 研究（[Hayes 97]）**：估算准确度与早期缺陷移除显著改善；**生产率没有受到显著影响**（即质量提升不以牺牲生产率为代价）。

**(2) CMU/SEI-97-TR-001 的重复测量方差分析（Repeated Measures ANOVA）**，对 PSP0/1/2 三个层级：

- **工作量估算**：三层级整体差异显著（p < .0005），改善主要发生在 PSP0→PSP1（PROBE 引入）；PSP1→PSP2 不显著。
- **缺陷密度**：三层级整体差异显著（p < .0005），PSP0→1 与 PSP1→2 均显著（p < .0005）。

即：估算能力在引入 PROBE 时跃升，之后趋稳；而缺陷密度在引入评审的 PSP2 阶段继续显著下降。

**(3) Ferguson 等，《Computer》30(5), 1997（"Results of Applying the Personal Software Process"）**：报告 Advanced Information Services、Motorola Paging Products Group、Union Switch & Signal 三家工业组织的实践，所有 PSP 项目均按期或提前交付，且只有一个项目出现客户报告缺陷。

> 来源：CMU/SEI-2000-TR-022 第 10 章；CMU/SEI-97-TR-001 ANOVA 章节；IEEE Computer Society 页面 computer.org/csdl/magazine/co/1997/05/r5024（DOI 10.1109/2.589907）。三组织具体数字以原文为准，本文未逐一复核全部数值。

---

## 第三部分：TSP —— 从个人到团队

### 3.1 TSP 是什么

Team Software Process（TSP）由 Humphrey 于 1996 年开发初始版本，目标是把经过 PSP 训练的工程师组成团队，为团队提供"持续做好质量工作"的**操作性过程**。设计适用规模为 **2–20 人**的团队（2000 年报告时更大规模的 TSPm 多团队版本尚在规划中；Wiley 条目记录 TSPm 后来用于更大项目）。

TSP 明确承认："人开发软件，人会犯错，这些错误就是缺陷的来源；目前没有防止缺陷注入的通用方法。因此 TSP 的质量重点放在**缺陷管理（defect management）**上。"其质量战略源自戴明与朱兰，并由 Michael Fagan 1976 年的软件检查（inspection）方法延伸到软件领域。

> 来源：CMU/SEI-2000-TR-023《The Team Software Process (TSP)》摘要、第 1-2 章。官方页：sei.cmu.edu/library/the-team-software-process-tsp/。

### 3.2 TSP 的基本要素

- **团队启动（launch）**：团队在启动过程中共同理解目标、自下而上制定计划，而非被动接受不现实的工期；
- **角色定义**：如计划经理、质量经理、设计经理等；
- **质量计划**：工程师依据个人缺陷注入/移除历史，估算各阶段会注入多少缺陷、在何处移除，质量经理检查参数是否合理、能否达成目标，否则迭代调整；
- **挣值（earned value）跟踪**：按周排团队可用时间并跟踪进度；
- **个人评审 + 团队检查 + 缺陷评审（defect review）** 的多层缺陷移除网。

Humphrey 的总结论之一：**PSP 训练通常使工程师的缺陷注入率降低 40%–50%**；TSP 中设计经理等角色还能进一步压低注入率。

> 来源：CMU/SEI-2000-TR-023（"engineers typically learn how to reduce their defect injection rates by 40% to 50%"，报告第 27 页附近，PDF 约第 41 页）。

### 3.3 TSP 质量指南（Table 3 全文要点）

TR-023 Table 3 "TSP Quality Guidelines"（报告第 20 页，PDF 第 34 页；注释说明：这些初始准则只在团队拥有自己的历史 TSP 数据前使用）：

**Percent Defect Free (PDF) 目标：**

| 阶段 | PDF 目标 |
|---|---|
| 编译 Compile | > 10% |
| 单元测试 Unit Test | > 50% |
| 集成测试 Integration Test | > 70% |
| 系统测试 System Test | > 90% |

**缺陷密度（Defects/KLOC）：**

- 总缺陷注入：75–150（未经 PSP 训练者按 100–200）；
- 编译阶段 < 10；单元测试 < 5（全部主要缺陷）；集成测试 < 0.5；系统测试 < 0.2。

**缺陷比率：** 详细设计评审缺陷/单元测试缺陷 > 2.0；代码评审缺陷/编译缺陷 > 2.0。

**时间比例：** 需求检查/需求时间 > 0.25；高层设计检查/高层设计时间 > 0.5；详细设计/编码时间 > 1.0；详细设计评审/详细设计时间 > 0.5；代码评审/编码时间 > 0.5。

**评审/检查速度：** 需求 < 2 页/小时；高层设计 < 5 页/小时；详细设计 < 100 伪码行/小时（伪码约等于 3 LOC）；代码 < 200 逻辑 LOC/小时。

**阶段 Yield：** 需求检查约 70%；设计评审/检查约 70%；代码评审/检查约 70%；编译约 50%（90%+ 语法缺陷）；编译前 > 75%；单元测试前 > 85%；集成测试前 > 97.5%；系统测试前 > 99%。

> 来源：CMU/SEI-2000-TR-023 Table 3，完整内容见 PDF 第 34 页。

### 3.4 组织级案例数据（TR-023 与 2003-TR-014）

**Teradyne（TR-023 引 [Musson 99]）：** 使用 TSP 前，集成测试、系统测试、现场测试与客户使用中的缺陷平均约 **20 个/KLOC**；首个 TSP 项目降至 **1 个/KLOC**；按平均每个缺陷耗费 12 个工程小时计算，每千行代码节省约 **228 个工程小时**。

**Hill AFB（TR-023 引 [Webb 00]）：** Hill 是首个被评为 CMM 5 级的政府组织；首个 TSP 项目团队**生产率提升 123%**，测试时间占项目进度比例从组织平均的 **22% 降至 2.7%**。

**Boeing（TR-023）：** 大型航电项目上，PSP/TSP 训练后（Release 9 vs 此前版本）缺陷数降低 **75%**，系统测试时间大幅下降。

**CMU/SEI-2003-TR-014（Davis & Mullaney，《TSP in Practice》，汇总 20 个 TSP 项目、13 家组织）：**

- 交付平均比计划晚 **6%**（误差范围：提前 20% ～ 晚 27%）；对照行业数据，超过一半软件项目延期 100% 以上或被取消；
- 生产率平均提升 **78%**；
- 产品缺陷比典型软件产品少 **10–100 倍**；平均质量水平 **5.2 sigma（约 60 个缺陷/百万行，即 0.06 个/KLOC）**，多例零缺陷交付；
- 对比基线：典型项目约 7.5 个缺陷/KLOC——即质量差距**超过两个数量级**；
- 各组织自报：系统测试缺陷密度平均降低 **8 倍**（范围 4–10 倍）；系统测试时长平均缩短 **4 倍**。

> 来源：CMU/SEI-2000-TR-023 第 7 章（Teradyne/Hill/Boeing）；CMU/SEI-2003-TR-014 执行摘要（PDF 第 13 页）与正文结果章节（"0.06 defects/KLOC versus 7.5 defects/KLOC"、"8 times reduction"）。官方页 DOI 10.1184/R1/6585302.v1。

### 3.5 安全维度：从质量到安全

CMU/SEI-2006-TR-017 把质量逻辑延伸到安全，提出质量三断言（报告第 34 页附近）：

1. 在进入测试前移除所有缺陷；
2. 过程必须提供质量度量；
3. 从高管到工程师全员参与。

报告指出：**缓冲区溢出约占已知互联网安全漏洞的一半**（引 [Kay 03, Seacord 06]；正文另一处表述为"超过 50% 的此类缺陷与缓冲区溢出相关"）。SEI 在 2004 年 TSP 汇报材料中给出量级推算：若 5% 的缺陷是潜在安全漏洞，那么 TSP 产品（0.02 缺陷/KLOC 级别）约为每百万行 1 个漏洞，而一般产品（2 个/KLOC）约为每百万行 100 个。

> 来源：CMU/SEI-2006-TR-017（buffer overflows "responsible for about half of the known Internet security" 漏洞）；SEMA 演示 SMU_Presentation.pdf，2004-01-29（publications.ri.cmu.edu）。另见 Humphrey《Safety-Critical Systems and the TSP》，CMU/SEI-2005-TN-011。

---

## 第四部分：PSP 在 AI 编程时代的适用性

### 4.1 时代背景：AI 编程工具的普及与证据冲突

到 2025 年，AI 编程助手已大规模普及。Stack Overflow 2025 开发者调查：约 **84%** 的受访者已经使用或计划使用 AI 工具；信任度上 46% 不信任、33% 信任（仅 3% 高度信任）。但关于"AI 是否真的提升了质量与生产率"，实证研究结论高度分裂，恰好为 PSP 式度量提供了舞台。

> 来源：Stack Overflow Developer Survey 2025（具体百分比以 survey.stackoverflow.co 原文为准；本文依据搜索摘要，精确措辞待确认）。

### 4.2 生产率：加速、减速与"感觉更快"

**(1) GitHub/Peng 等 Copilot 受控实验（正面证据）：** 95 名专业开发者，任务为用 JavaScript 实现 HTTP 服务器；使用 Copilot 组完成任务快 **55%**（中位数约 71 分钟 vs 161 分钟），完成率 78% vs 70%。

> 来源：GitHub Blog《Research: quantifying GitHub Copilot's impact on developer productivity and happiness》，2022-09-07；Peng et al., "The Impact of AI Pair Programmer..."，Microsoft Research，2023-02。本文未复核原文全部数字，71/161 分钟为该研究广为引用的数据。

**(2) METR 2025 RCT（反面证据）：** Becker, Rush, Barnes, Rein（METR），arXiv:2507.09089，2025-07：

- **16 名**有经验的开源开发者，在自己平均参与 5 年的成熟项目上完成 **246 个真实任务**（平均 2 小时/任务）；
- 工具为 Cursor Pro + Claude 3.5/3.7 Sonnet（2025 年 2–6 月前沿水平）；
- 结果：允许使用 AI 使完成时间**增加 19%**（置信区间 +2% ～ +39%）；
- 而开发者事前预测 AI 会提速 **24%**，事后仍估计自己被提速 **20%**；经济学专家预测提速 39%、ML 专家 38%，均与实测相反；
- METR 于 2026-02-24 发文说明：仍认为该估计对"当时的工具与人群"准确；新一轮（2025-08 起，57 名开发者）实验原始数据出现一些提速迹象，但因参与者反馈与调查问题，信号被判定不可靠，因此更改实验设计。

> 来源：arXiv:2507.09089 摘要（原文已核）；METR 博客 metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study；METR《We are Changing our Developer Productivity Experiment Design》，2026-02-24。

**与 PSP 的呼应**：METR 研究最惊人的发现不是慢，而是**主观感知与客观度量的系统性背离**——这正是 Humphrey 在 1990 年代坚持要求工程师逐任务记录时间的理由：没有个人级、阶段级的度量，"快了/慢了"只是错觉。

**(3) DORA 2024 报告：** AI 采用率每增加 25%，文档质量约 +7.5%、代码质量约 +3.4%、代码评审速度约 +3.1%，但交付吞吐量约 −1.5%、交付稳定性约 −7.2%。即组织层面呈现"局部活动加速、端到端结果未必改善"的格局。

> 来源：DORA《State of AI-Assisted Software Development》2024 报告 PDF（dora.dev）。具体系数为回归估计，本文依据报告摘要数据，引用时建议回查 PDF 原表。

### 4.3 质量与安全：AI 可能在放大"后期缺陷"

**(1) Stanford 用户实验：** Perry, Srivastava, Kumar, Boneh，《Do Users Write More Insecure Code with AI Assistants?》，ACM CCS 2023（pp. 2785-2799；arXiv:2211.03622）。基于 codex-davinci-002：能使用 AI 助手的参与者在 5 个任务中的 4 个里写出不安全方案的频率更高，**代码显著更不安全，却更倾向于相信自己的代码安全**——过度自信效应。对 AI 信任度更低、在提示上更用心的参与者漏洞更少。

> 来源：arXiv:2211.03622 / ACM DL 10.1145/3576915.3623157（摘要已核）。

**(2) 漏洞复制研究：** Asare et al.（arXiv:2308.06587）等针对 Copilot 建议的实证研究，观察到相当比例开发者会直接复制含漏洞代码（文献中常引约 33% 复制漏洞代码、25% 复制修复代码的比例，该具体数字对应《Is GitHub's Copilot as Bad as Humans at Introducing Vulnerabilities?》，*Empirical Software Engineering* 2023；**百分比待与原文逐一核对**）。

**(3) AI 生成补丁的大规模安全研究：** arXiv:2507.02976（2025），覆盖 20,000+ 真实 issue 上的 AI 补丁安全性分析，用于评估 AI 代码的端到端安全表现（具体结论待全文核实）。

**(4) GitClear 代码 churn 数据：** 《Coding on Copilot》2024/2025 报告称，代码 churn（短期内被重写的代码比例）在 2024 年相对 2021 基线接近翻倍，2024 实际重复代码块比例创历史新高——指向 AI 生成代码"写得快、返工也多"的模式（具体百分比待回查 GitClear PDF）。

**PSP 视角的解读**：这些证据共同指向 Humphrey 早已警告的模式——当缺陷在**后期**（测试、评审、生产）才暴露时，返工成本会吞没前期的时间节省。AI 提升了"编码"这一最可见阶段的速度，却可能同时提高缺陷注入率与评审负担；没有 yield、A/FR、缺陷注入/移除率的度量，团队无法判断自己是真的变好，还是只是"更快地产生缺陷"。

### 4.4 Copilot 接受率与使用者分布

GitHub《Sea Change in Software Development》报告中 Copilot 建议整体接受率约 **30%**；并报告一个反直觉分布：使用 AI 最多的五分位与最少的五分位在接受率上差异有限（约 31.9% vs 26.2%），说明"装了工具"不等于"有效使用"，个体实践差异才是关键变量。

> 来源：GitHub《Sea Change in Software Development》PDF（具体版本与页码待确认）。

### 4.5 直接结合 PSP 与生成式 AI 的最新探索

这是与本文主题最直接相关的一手文献，但均为 2025-2026 年新成果，证据等级有限：

1. **《Trials of Using Generative AI in Personal Software Process Education》**（IISA 2026）：在 PSP 教学中引入生成式 AI 的试验，关注 AI 对学生度量数据质量、练习完成方式与学习效果的影响（具体数据待全文核实）。
2. **Kusakabe 等，《Toward a Process Improvement Framework for Software Development Using Generative AI》**（IIAI AAI Winter 2025）：尝试把 PSP/TSP 的过程改进框架扩展到生成式 AI 开发场景，提出面向 AI 协作的过程度量与改进框架（待全文核实）。
3. 新手开发者研究《The Widening Gap: The Impact of Generative AI on the Skill Development of Novice Programmers》（arXiv:2405.17739，2024）：关注 AI 辅助对新手技能形成路径的影响——与 PSP"先用个人数据建立自我认知"的教育逻辑直接相关（具体结论待全文核实）。

### 4.6 PSP 哲学在 AI 时代的适用与失效

**适用（且价值上升）的部分：**

1. **个人度量基础设施**：AI 使产出速度与缺陷来源更难凭直觉判断，时间日志、缺陷日志（含"缺陷由 AI 生成还是本人编写"这一新维度）成为区分真实收益与错觉的唯一手段。METR 的 19% 减速与 20% 自我感知提速之差，就是该论点的当代实证。
2. **PROBE 与任务计划**：AI 改变的是编码速率的分布而非估算逻辑本身；工程师仍需基于**自己**的历史数据（含 AI 辅助下的数据）校准规模与工期估算，且更需要任务级计划来防止"接受建议 → 不断返工"的漂移。
3. **评审纪律与 yield/A/FR**：AI 倾向于在编码阶段大量注入缺陷（Stanford 研究的安全过度自信、GitClear 的 churn），Humphrey"质量在进入测试前决定"的原则更显重要；A/FR 的内涵可扩展为"对 AI 产出的鉴定成本 vs 故障返工成本"，评审速度上限（<200 LOC/小时）对 AI 生成的大段代码同样成立甚至更严格。
4. **PIP 与缺陷预防**：缺陷日志分析可用于识别"我常被 AI 在哪类问题上误导"，驱动个人检查清单与提示策略改进——这正是 PSP 缺陷预防循环的自然延伸。
5. **管理层责任**：TR-014 与 2006-TR-017 强调的高管参与、质量目标与不切实际工期问题，在"AI 让开发快 X 倍"的叙事压力下反而更突出。

**需要修正或受到挑战的部分：**

1. **规模度量以 LOC 为中心**：AI 使"写了多少行"与投入/价值进一步脱钩，LOC 作为生产率口径和 PROBE 代理的有效性下降，可能需要以任务、行为规格或交付结果为补充口径。
2. **过程是"个人手写"的隐含假设**：PSP 脚本假设每个阶段由工程师本人执行；AI 介入后，设计、编码、评审都可能是人-机混合活动，脚本需要显式记录 AI 使用点与采纳决策。
3. **零缺陷目标的经济权衡**：AI 大幅降低了"再生成一版"的成本，也改变了评审 vs 重写的成本结构，部分场景下"生成-测试-再生成"可能比"彻底设计 + 评审"更经济（这恰是 Cleanroom/Humphrey 路线与现代快速迭代路线的争议点，目前缺乏高质量对照证据）。
4. **练习式技能习得路径**：若新手通过 AI 完成 PSP 练习，度量数据仍在但"技能内化"环节可能被架空（arXiv:2405.17739 关注的核心风险），PSP 教育设计需要新的约束。

**综合判断**：AI 改变的是 PSP 各阶段的**速率与执行者**，而不是其底层认识论——*用个人的、阶段化的、可核验的度量来驱动改进*。在研究证据互相冲突、主观感知系统性失真的环境里，PSP 的度量纪律不是过时，而是从"良好习惯"变成了"判别工具是否真正有效的前提条件"；但 PSP 的具体表单与 LOC 口径需要围绕"AI 参与度"重新参数化。这一判断目前由 METR、Stanford、DORA 等间接证据支撑，**直接检验"PSP 实践者使用 AI 的效果"的对照研究尚属空白**（IISA 2026、IIAI 2025 是早期尝试），属于值得实证研究的开放问题。

---

## 参考资料

### 2026-10-26 更新：新增证据（METR 单点结论已被修正）

经补充检索，2025 年 7 月 METR 的"慢 19%"不应再作为当前事实引用：

- **METR 自行撤回时效性**：原页面标注 "These results are out of date"，并在 2026-02-24 更新中说明历史结果"不再反映当前 AI 模型的影响"；新一轮实验（2025-08 起，57 名开发者、143 个仓库、800+ 任务）原始数据显示老参与者约 -18%、新参与者约 -4%（任务时间下降），但区间跨零且存在招募选择效应，METR 判定信号不可靠，仅定性判断 2026 年初 AI 很可能确实提速。
- **Management Science（2026-02-27，DOI: 10.1287/mnsc.2025.00535）**：微软/埃森哲/某财富100强三场实地 RCT，共 4,867 名开发者，完成任务量 +26.08%（标准误 10.3%）；经验越少收益越大。
- **Faros AI 2026（约 22,000 开发者/4,000 团队）**：任务量 +33.7%，同时代码评审时间 +441%。
- **CACM 2026-07-21《How to Cross the AI Code Productivity Divide》**：任务 +21%、合并 PR +98%，但 PR 评审时间 +91%，端到端交付时间未缩短。
- **METR 2026-05-11 问卷（349 名技术工作者）**：自报工作价值中位数提升 1.4–2 倍，METR 提醒调查结果未必符合现实。
- **MIT Sloan 报道（2026-09-02）**：基于 10 万+ GitHub 开发者的研究（Demirer, Musolff 等），AI 工具主要改善写代码等早期阶段，对后续阶段帮助有限。

**修正后的综合判断**：AI 提速为真；瓶颈由生成端迁移至评审/集成端；感知偏差仍在但幅度收窄。PSP 的论点（个人度量、评审优先、质量在进入测试前决定、个人责任）被新证据强化，但其事实基础应建立在多研究综合证据上，而非 METR 单点数据。

### 一手文献（已下载并提取原文，存档于临时工作目录）

1. Humphrey, W. S. *The Personal Software Process (PSP)*. CMU/SEI-2000-TR-022, ESC-TR-2000-022, November 2000. DOI: 10.1184/R1/6585197.v1. https://www.sei.cmu.edu/documents/644/2000_005_001_13751.pdf
2. Humphrey, W. S. *The Team Software Process (TSP)*. CMU/SEI-2000-TR-023, ESC-TR-2000-023, November 2000. 官方页：https://www.sei.cmu.edu/library/the-team-software-process-tsp/ （本文 PDF 取自镜像 http://www.uml.org.cn/SoftWareProcess/pdf/tsp.pdf 与 SEI documents/645）
3. Humphrey, W. S. 等. CMU/SEI-2006-TR-017（质量三断言、缓冲区溢出数据）, 2006. DOI: 10.1184/R1/6584591.v1. https://www.sei.cmu.edu/documents/773/2006_005_001_14789.pdf
4. Humphrey, W. S. *Introduction to the Personal Software Process*. CMU/SEI-97-TR-001, 1997. DOI: 10.1184/R1/6585230.v1. https://www.sei.cmu.edu/documents/1166/1997_005_001_16565.pdf
5. *PSP Body of Knowledge, Version 2*. SEI, 2009. https://www.sei.cmu.edu/documents/1867/2009_003_001_15029.pdf
6. Davis, N., Mullaney, J. *The Team Software Process (TSP) in Practice: A Summary of Recent Results*. CMU/SEI-2003-TR-014, September 2003. DOI: 10.1184/R1/6585302.v1.
7. Computer History Museum. *Oral History of Watts Humphrey*, 2009-12-17. archive.computerhistory.org（102702107.05.01.acc.pdf）

### 书籍（未取得电子版，引用细节标注待确认）

8. Humphrey, W. S. *A Discipline for Software Engineering*. Addison-Wesley, 1995.
9. Humphrey, W. S. *PSP: A Self-Improvement Process for Software Engineers*. Addison-Wesley, 2005.
10. Humphrey, W. S. *Introduction to the Personal Software Process*. Addison-Wesley, 1997.
11. Humphrey, W. S. *Winning with Software: An Executive Strategy*. Addison-Wesley, 2002.
12. Humphrey, W. S. *TSP: Leading a Development Team*. Addison-Wesley, 2005.
13. Humphrey, W. S. *Safety-Critical Systems and the TSP*. CMU/SEI-2005-TN-011.

### 论文与行业报告（AI 部分）

14. Becker, J., Rush, N., Barnes, E., Rein, D. *Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity*. arXiv:2507.09089, 2025.
15. METR. *We are Changing our Developer Productivity Experiment Design*, 2026-02-24.
16. Perry, N., Srivastava, M., Kumar, D., Boneh, D. *Do Users Write More Insecure Code with AI Assistants?* ACM CCS 2023, pp. 2785-2799. arXiv:2211.03622.
17. Peng, S. 等. GitHub Copilot 生产率实验. GitHub Blog, 2022-09-07；Microsoft Research, 2023.
18. DORA. *State of AI-Assisted Software Development*, 2024.
19. GitClear. *Coding on Copilot*（2024、2025 版报告）.
20. Asare, O. 等. arXiv:2308.06587, 2023；及 *Empirical Software Engineering* 2023 Copilot 漏洞研究.
21. *AI patch security* 研究. arXiv:2507.02976, 2025.
22. *The Widening Gap*. arXiv:2405.17739, 2024.
23. *Trials of Using Generative AI in Personal Software Process Education*. IISA 2026.
24. Kusakabe 等. *Toward a Process Improvement Framework for Software Development Using Generative AI*. IIAI AAI Winter 2025.
25. Stack Overflow Developer Survey 2025.
26. Thomas, N. *The SPACE of AI*. ACM Queue（抓取时 403，原文措辞待确认）.
