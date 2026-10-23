# 六位质量管理权威核心思想调研

> 调研日期：2026-10-23  
> 说明：所有观点和信息均来自公开可查的资料来源。标注"待确认"的部分表示未能通过多个独立来源交叉验证。

---

## 一、Watts S. Humphrey（1927-2010）——"软件质量之父"

### 基本背景
- 卡内基梅隆大学软件工程研究所（SEI）资深研究员，首位 SEI Fellow
- 1986年加入 SEI，建立软件过程项目，领导开发了软件能力成熟度模型（CMM）
- 曾在 IBM 工作27年，管理15个实验室、4000名软件专业人员
- 2003年获美国国家技术奖章（National Medal of Technology）
- 被誉为"Father of Software Quality"（软件质量之父）

> 来源：CMU SEI 官网 (sei.cmu.edu/authors/watts-humphrey)、Wikipedia "Watts Humphrey"、IEEE Computer Society 官方介绍

### 核心理论与主要贡献

1. **软件能力成熟度模型（CMM / CMMI）**
   - 1987年发布首个五级成熟度框架（CMU/SEI-87-TR-11）
   - 五级：初始级(Initial)、可重复级(Repeatable)、已定义级(Defined)、已管理级(Managed)、优化级(Optimizing)
   - 后发展为 CMMI（能力成熟度模型集成），成为全球软件过程改进的事实标准
   - 美国国防部要求其软件供应商至少达到 CMM 3级

   > 来源：Wikipedia "Watts Humphrey"；CMU/SEI-87-TR-11 技术报告；Managing the Software Process (1989)

2. **个人软件过程（PSP, Personal Software Process）**
   - 1995年正式提出，将 CMM 原则应用到个体工程师层面
   - 核心思想：软件工程师对自己产出的质量负有个人责任；通过数据驱动的自我管理提升质量和生产率
   - 包含 PSP0/PSP0.1 → PSP1/PSP1.1 → PSP2/PSP2.1 → PSP3 的渐进式过程改进路径
   - 关键实践：时间记录、缺陷记录、规模度量、个人评审、缺陷预防

   > 来源：A Discipline for Software Engineering (Addison-Wesley, 1995)；CMU/SEI-2000-TR-022 技术报告

3. **团队软件过程（TSP, Team Software Process）**
   - 1996年开发初始版本，将 PSP 训练有素的工程师组织成高效团队
   - TSP 提供团队启动（launch）流程、角色定义、质量计划、挣值跟踪等方法
   - 三大过程改进方法的关系：CMM 关注组织管理层面，TSP 关注团队产品层面，PSP 关注个人技能层面

   > 来源：CMU/SEI-2000-TR-023 技术报告；Introduction to the Team Software Process (2000)

### 关于四个核心主题的具体论述

#### 过程改进
- **三个维度的过程改进**：组织级（CMM）、团队级（TSP）、个人级（PSP）三者互补，缺一不可
- "CMM 提供组织改进框架，PSP 提供工程师个人纪律，TSP 指导工程师实际开展工作"
  > 来源：CMU/SEI-2000-TR-022, Chapter 11; CMU/SEI-2000-TR-023, Chapter 1

#### 质量成本
- "越早发现并修复缺陷，成本越低"（It costs less to find and fix defects earlier in a process than later.）
- PSP 数据显示：代码评审中平均每小时发现 6.52 个缺陷，设计评审中 2.96 个/小时，单元测试中仅 2.21 个/小时
- 移除100个缺陷：单元测试需要约45小时，代码评审仅需约15小时
  > 来源：CMU/SEI-2000-TR-022, Chapter 6 "PSP Quality Management"

#### 人的因素
- "要生产高质量产品，工程师必须对产品质量感到个人负责。优质产品不是偶然产生的；工程师必须努力做好质量工作。"
  （To produce quality products, engineers must feel personally responsible for the quality of their products. Superior products are not produced by mistake; engineers must strive to do quality work.）
  > 来源：CMU/SEI-2000-TR-022, Chapter 6

- PSP 训练通常可使工程师的缺陷注入率降低 40%-50%
  > 来源：CMU/SEI-2000-TR-023, 第27页

#### 预防胜于检测
- "管理缺陷最有效的方法是从一开始就预防其注入。"
  （The most effective way to manage defects is to prevent their initial introduction.）
- 三种缺陷预防方法：(1) 记录并分析每个缺陷的数据；(2) 使用有效的设计方法和表示法产出完整设计；(3) 更彻底的设计减少编码时间从而减少缺陷注入
  > 来源：CMU/SEI-2000-TR-022, Section 6.4 "Defect Prevention"

- "任何测试方法都只能发现大型复杂产品中缺陷的一小部分，产品越大越复杂，这个比例就越小。这意味着，要想从测试中得到高质量产品，就必须把高质量产品送进测试。"
  （First, no test yet devised can identify more than a fraction of the defects in a large and complex product, and the larger and more complex the product, the smaller this fraction becomes. This means that to get a high-quality product out of testing, one must put a high-quality product into testing.）
  > 来源：CMU/SEI-2006-TR-017 "Systems of Systems: Scaling Up the Development Process"

### 核心观点/名言
- "质量是按客户想要的方式完成工作。我同意戴明、朱兰和其他质量大师的定义。质量就是尽一切努力真正满足客户。"
  （Quality is doing the job the way it's wanted. I agree with Deming's and Juran's and the other quality guru's definitions. Quality is doing what it takes to truly satisfy the customer.）
  > 来源：Computer History Museum Oral History of Watts Humphrey, 2009

- "软件人员是非常出色的人，做着卓越的工作，但他们被当作失败者对待。"
  （Software people are really great people who are doing superb work, and they are treated like failures.）
  > 来源：同上

### 主要著作/论文（与软件质量相关）

| 序号 | 标题 | 类型 | 年份 | 出版社/会议 |
|------|------|------|------|-------------|
| 1 | Managing the Software Process | 专著 | 1989 | Addison-Wesley |
| 2 | A Discipline for Software Engineering | 专著 | 1995 | Addison-Wesley |
| 3 | Introduction to the Personal Software Process | 专著 | 1997 | Addison-Wesley |
| 4 | Introduction to the Team Software Process | 专著 | 2000 | Addison-Wesley |
| 5 | PSP: A Self-Improvement Process for Software Engineers | 专著 | 2005 | Addison-Wesley |
| 6 | Winning with Software: An Executive Strategy | 专著 | 2002 | Addison-Wesley |
| 7 | The Personal Software Process (PSP) | 技术报告 | 2000 | CMU/SEI-2000-TR-022 |
| 8 | The Team Software Process (TSP) | 技术报告 | 2000 | CMU/SEI-2000-TR-023 |
| 9 | "Using a Defined and Measured Personal Software Process" | 期刊论文 | 1996 | IEEE Software |
| 10 | "Introducing the Personal Software Process: Three Industry Case Studies" | 期刊论文 | 1997 | IEEE Computer (Ferguson et al., with Humphrey) |

---

## 二、W. Edwards Deming（1900-1993）——"质量管理之父"

### 基本背景
- 美国统计学家、管理顾问，日本战后经济复兴的关键人物之一
- 日本最高质量奖项以他命名——戴明奖（Deming Prize）
- 代表作：《转危为机》（Out of the Crisis, 1986）、《新经济观》（The New Economics, 1993）
- 提出"渊博知识体系"（System of Profound Knowledge, SoPK）和"十四点"管理原则

> 来源：The W. Edwards Deming Institute (deming.org)；ASQ 官网；Out of the Crisis (1986)

### 核心理论与主要贡献

1. **渊博知识体系（System of Profound Knowledge, SoPK）**
   - 四个相互关联的组成部分：
     - **对系统的欣赏**（Appreciation for a System）：理解组织各部分的相互依存关系
     - **关于变异的知识**（Knowledge of Variation）：区分共同原因与特殊原因的变异
     - **知识理论**（Theory of Knowledge）：知识建立在理论基础上，通过预测-检验循环获得
     - **心理学**（Psychology）：理解人的动机、恐惧、自豪感等心理因素
   - "十四点是渊博知识体系的自然应用"
     > 来源：The New Economics, 1993, Chapter 4；deming.org/explore/sopk

2. **十四点管理原则（14 Points for Management）**
   - 首次完整提出于 Out of the Crisis (1986)，第23-24页
   - 是从传统管理向优化型管理转型的行动指南
     > 来源：Out of the Crisis, MIT Press, 1986；deming.org/explore/fourteen-points

3. **PDSA 循环（Plan-Do-Study-Act）**
   - 基于 Shewhart 的思想发展而来，是持续改进的核心方法论
   - 强调学习（Study）而非简单的检查（Check）
     > 来源：The New Economics, 1993；Deming Institute 官方资料

### Deming 十四点中与软件开发最相关的几点

| 点 | 原文标题 | 与软件开发的关联 |
|----|---------|----------------|
| 1 | 树立改进产品和服务的恒久目标 | 软件组织需要长期质量愿景，不能只顾短期交付 |
| 3 | 停止依靠检验来获得质量 | 对应"内建质量"——不能靠测试来保证质量，要在开发过程中构建质量 |
| 5 | 持续不断地改进生产和服务的系统 | 软件过程改进（如 CMMI、Agile 持续改进）的核心理念 |
| 7 | 建立领导 | 管理者的角色从监督转为赋能，帮助开发人员更好地工作 |
| 8 | 消除恐惧 | 鼓励开发者坦诚面对问题，不因为报告缺陷而受惩罚；心理安全是敏捷团队核心 |
| 9 | 打破部门之间的壁垒 | 对应 DevOps、跨职能团队——开发、测试、运维协同 |
| 10 | 消除口号、训诫和目标 | 反对"零缺陷"等空洞口号，因为大多数质量问题源于系统而非个人 |
| 12 | 消除剥夺人们工作自豪感的障碍 | 开发者应该以写出高质量代码为荣，而非只看数量和速度 |
| 13 | 大力开展教育和自我改进 | 技术团队的持续学习和能力建设 |

> 来源：Out of the Crisis (1986)，pp. 23-24；Deming Institute 官网

### 关于四个核心主题的具体论述

#### 过程改进
- "改进质量会自动提高生产率。"（Improvement of quality will automatically improve productivity.）
  > 来源：Deming's 14 Points, Point 12 相关论述 (University of Auckland lecture notes)

- 质量链式反应：**改进质量 → 成本降低（返工减少、错误减少、延迟减少）→ 生产率提高 → 以更好的质量和更低的价格占领市场 → 企业生存 → 提供更多就业**
  > 来源：Out of the Crisis；Deming Institute "Quality Chain Reaction" 资料

#### 质量成本
- "更高的质量花费更少，而非更多。"（Higher quality costs less, not more.）
  > 来源：Deming Institute 教学资料 (SAGE Notes, University of Auckland)

- 戴明估计："94%的麻烦和改进可能性属于系统（管理层责任），只有6%是特殊原因。"
  （I should estimate that in my experience most troubles and most possibilities for improvement add up to the proportions something like this: 94% belongs to the system (responsibility of management), 6% special.）
  > 来源：Out of the Crisis, p. 315

#### 人的因素
- "我们必须保留人们天生具有的内在动力、尊严、合作、好奇心和学习的快乐。"
  （We must preserve the power of intrinsic motivation, dignity, cooperation, curiosity, joy in learning, that people are born with.）
  > 来源：The W. Edwards Deming Institute 官方引述

- 关于"消除恐惧"的阐述："没有人能在缺乏安全感的情况下发挥最佳水平。安全感意味着不害怕——不怕表达想法，不怕提问。恐惧有很多表现形式。任何形式的恐惧的共同后果都是绩效受损和数据注水。"
  > 来源：Engineering LibreTexts "Deming's 14 Points" (引述自 Out of the Crisis)

- "你的工作就是你的自画像。"（Your work is your self-portrait.）
  > 来源：Deming Institute 资料

#### 预防胜于检测
- 第三点原文："停止依靠检验来获得质量。取消大规模检验的需要，代之以在生产过程中就把质量建造进去。"
  （Cease dependence on inspection to achieve quality. Eliminate the need for inspection on a mass basis by building quality into the product in the first place.）
  > 来源：Out of the Crisis, p. 23

- "发现错误不是过程的改进。"（Finding what is wrong is not improvement of the process.）
  > 来源：Deming Library Video 中的 Deming 原话（经 Deming Institute 确认）

### 核心观点/名言
- "仅仅尽力而为是不够的。你必须知道该做什么，然后尽力而为。"
  （Doing your best is not good enough. You have to know what to do. Then do your best.）
  > 来源：Deming Institute 官方引述

- "成功的秘诀是对目标的坚定不移。"
  （The secret of success is constancy to purpose.）—— 戴明引用 Benjamin Disraeli
  > 来源：SAGE Notes (University of Auckland)

### 主要著作

| 序号 | 标题 | 类型 | 年份 | 出版社 |
|------|------|------|------|--------|
| 1 | Out of the Crisis（《转危为机》/《走出危机》） | 专著 | 1986 | MIT Center for Advanced Engineering; MIT Press, 2000 |
| 2 | The New Economics for Industry, Government, Education（《新经济观》） | 专著 | 1993 | MIT Press |
| 3 | Quality, Productivity, and Competitive Position | 专著 | 1982 | MIT Center for Advanced Engineering Study |
| 4 | The Essential Deming: Leadership Principles from the Father of Quality | 文集 | 2012 | Joyce Orsini 编, McGraw-Hill |
| 5 | "Deming's 14 Points for Management" | 核心理论 | 1982/1986 | 出自 Out of the Crisis 第2章 |

---

## 三、Joseph M. Juran（1904-2008）——质量三部曲与帕累托原则

### 基本背景
- 罗马尼亚出生的美国质量管理学家，享年103岁
- 1951年出版《质量控制手册》（Quality Control Handbook），成为质量领域圣经
- 创立朱兰研究院（Juran Institute）
- 与戴明一同被认为是日本质量革命的关键推动者

> 来源：Juran Institute (juran.com)；New York Times 讣告 (2008-03-03)；Wikipedia "Joseph M. Juran"

### 核心理论与主要贡献

1. **质量三部曲（Juran Trilogy）**
   - **质量策划（Quality Planning）**：设计能够满足客户需求的产品和过程（又称"质量设计"/Quality by Design）
   - **质量控制（Quality Control）**：在运营中保持质量水平，通过监控发现偏差并纠正
   - **质量改进（Quality Improvement）**：取得突破性的绩效提升，达到前所未有水平
   - 三者关系：策划确定质量目标和实现路径，控制维持质量水平，改进实现突破
   - 以"劣质成本"（Cost of Poor Quality, COPQ）为纵轴展示三者关系
     > 来源：Juran Institute 官方博客 "The Juran Trilogy" (juran.com/blog/the-juran-trilogy)；Juran on Quality by Design (1992)

2. **质量的定义：适用性（Fitness for Use）**
   - "质量就是适用性。"（Quality is fitness for use.）
   - 与 Crosby 的"符合要求"（conformance to requirements）形成对比
   - 强调从客户视角定义质量，包括设计质量、一致性、可用性、安全性和现场使用等维度
     > 来源：Juran's Quality Control Handbook；National Academy of Engineering 传记文章

3. **帕累托原则（Pareto Principle / 80-20 Rule）**
   - 朱兰是第一个将帕累托原则应用于质量领域的人
   - "关键的少数和次要的多数"（the vital few and the trivial many / useful many）
   - 在质量语境下：80%的问题由20%的原因导致
   - 朱兰曾说，这个原则后来被错误地归到帕累托名下，但50多年后有人问他是否要改名，他说"不用，但也许等我走了之后可以"
     > 来源：National Academy of Engineering 传记；Juran Institute "A Guide to the Pareto Principle"

### 关于四个核心主题的具体论述

#### 过程改进
- 质量改进的本质是取得"突破性"（breakthrough）成果，而非渐进式改善
- "突破意味着有组织地创造有益变革，达到前所未有的绩效水平。"
  （Breakthrough means the organized creation of beneficial change and the attainment of unprecedented levels of performance.）
  > 来源：Juran Institute 官方博客 "The Juran Trilogy"

- 质量改进应采取"项目-by-项目"的方法，聚焦关键少数（vital few）项目
  > 来源：Juran Trilogy 官方说明

#### 质量成本
- 首创"质量成本"概念——如果所有人工作完美，这些成本就会消失
- "在美国，我们所做的工作中约有三分之一是重做以前做过的工作。"
  （In the U.S.A., about a third of what we do consists of redoing work previously done.）
  > 来源：Quality Quotes, ASQ Quality Press, 1996（由 Hélio Gomes 记录的朱兰言论）

- 劣质成本（COPQ）：所有因缺陷、错误、返工、故障等导致的成本
  > 来源：Juran Institute 官方资料

#### 人的因素
- 朱兰将质量管理从工厂车间扩展到更广泛的管理和服务领域
- 克服组织变革阻力是质量改进的关键挑战
- "对于那些从事质量管理职业的人，感谢你们的幸运星。你们的领域在有生之年会大幅发展，尤其是在我们三大产业——医疗、教育和政府——中。"
  （To those whose careers are in the field of managing for quality, thank your lucky stars. Your field will grow extensively during your lifetime, especially in three of our giant industries—health, education and government.）
  > 来源：朱兰回忆录（引述自 PMC 论文 "Joseph Juran: overcoming resistance to organisational change"）

#### 预防胜于检测
- 质量三部曲中，"质量策划"是第一步，强调在设计阶段就把质量建造进去
- "控制质量"这个概念的含义从早期的事后检验（detection control）演变为"预防（proactive control）"
  > 来源：Juran Institute "Quality Leadership: Broader than Quality Management" 博客文章

### 核心观点/名言
- "质量就是适用性。"（Quality is fitness for use.）
  > 来源：Juran's Quality Control Handbook；NAE 传记

- "关于劣质成本：在美国，我们所做的工作中约有三分之一是重做以前做过的工作。"
  > 来源：Quality Quotes (ASQ Quality Press, 1996)

- "所有质量问题中，相对少数的贡献者（关键的少数）造成了绝大部分的影响。"
  > 来源：Juran Institute 帕累托原则指南

### 主要著作

| 序号 | 标题 | 类型 | 年份 | 出版社 |
|------|------|------|------|--------|
| 1 | Quality Control Handbook（《质量控制手册》） | 手册 | 1951（首版） | McGraw-Hill |
| 2 | Juran on Quality by Design: The New Steps for Planning Quality into Goods and Services | 专著 | 1992 | Free Press |
| 3 | Juran on Leadership for Quality: An Executive Handbook | 专著 | 1989 | Free Press |
| 4 | Quality Planning and Analysis（与 Frank M. Gryna 合著） | 教材 | 多版 | McGraw-Hill |
| 5 | Architect of Quality: The Autobiography of Dr. Joseph M. Juran | 自传 | 2004 | McGraw-Hill |

---

## 四、Philip B. Crosby（1926-2001）——零缺陷与质量免费

### 基本背景
- 从流水线工人起步，后在 Martin-Marietta 提出"零缺陷"概念
- 曾任 ITT 公司质量总监，后创立 Philip Crosby Associates
- 1979年出版《质量免费》（Quality is Free），成为全球畅销书（引用超过2500次）
- 2001年因食道癌去世，享年75岁

> 来源：New York Times 讣告 (2001-08-22)；ASQ Quality Progress 文章 "Philip B. Crosby's Mark on Quality" (2001年10月)；IndustryWeek 访谈 (1995)；Wikipedia "Zero Defects"

### 核心理论与主要贡献

1. **质量管理四绝对（Four Absolutes of Quality Management）**
   1. **质量的定义是符合要求**，而非"好"或"优良"（Quality is conformance to requirements, not goodness.）
   2. **质量的系统是预防**，而非检验（The system of quality is prevention, not appraisal.）
   3. **质量的执行标准是零缺陷**，而非"可接受质量水平"（The performance standard is Zero Defects, not acceptable quality levels.）
   4. **质量的度量是不符合要求的代价**，而非质量指数（The measurement of quality is the price of nonconformance, not indexes.）
   
   > 来源：Quality is Free (1979)；Wikipedia "Zero Defects" 条目；ASQ Quality Progress, 2001

2. **质量免费（Quality is Free）**
   - "质量是免费的。但它不是礼物。"（Quality is free, but it's not a gift.）
   - 质量本身不花钱——花钱的是"不符合要求"（nonconformance）——即返工、报废、保修、客户流失等
   - 通过预防投入来降低不符合要求的代价，总体上是省钱的
     > 来源：Quality is Free (1979)；IndustryWeek 1995年访谈

3. **零缺陷（Zero Defects）与"第一次就把事情做对"（DIRFT）**
   - 零缺陷不是要求人完美无缺，而是"按照约定的要求去执行，每次都做对"
   - "零缺陷与完美无关。它的全部含义是：履行你同意的要求，每次都做对。"
     （Zero Defects has nothing to do with being perfect. All ZD means is: Perform to the requirement you have agreed to, and do it right every time.）
   - 核心口号：Do It Right the First Time（DIRFT）
     > 来源："The myths of Zero Defects" (Philip B. Crosby)；Quality is Free

### 关于四个核心主题的具体论述

#### 过程改进
- 提出**质量改进十四步**（14-Step Quality Improvement Program）：
  1. 管理层承诺 2. 成立质量改进团队 3. 建立质量度量 4. 评估质量成本
  5. 提高质量意识 6. 纠正措施 7. 建立零缺陷委员会 8. 主管培训
  9. 零缺陷日 10. 员工目标设定 11. 消除错误原因 12. 表彰（非金钱）
  13. 质量理事会 14. 循环重复
  > 来源：Quality is Free (1979)

- 提出**质量疫苗**（Quality Vaccine）的五个成分：正直、系统、沟通、运营、政策
  > 来源：Quality is Free

#### 质量成本
- 使用"不符合要求的代价"（Price of Nonconformance）替代"质量成本"，以避免"高质量需要高成本"的误解
- 质量成本 = 符合要求的代价（预防、鉴定） + 不符合要求的代价（返工、报废、保修等）
- "质量免费"的含义：预防投入减少了不符合要求的代价，最终总成本降低
  > 来源：Quality is Free, 1979；Wikipedia "Zero Defects" 条目

#### 人的因素
- "质量是精心构建的文化环境的产物。它必须是组织的结构，而不是结构的一部分。"
  （Quality is the result of a carefully constructed cultural environment. It has to be the fabric of the organisation, not part of the fabric.）
  > 来源：Quality.org 资料（引述自 Crosby 著作）

- 管理层承诺是第一要素："当你把质量委派出去的时候，质量就开始完蛋了。我说的承诺，是指 CEO 亲自参与做事，而不是只说'好的，我祝福，给你点钱去做'。"
  （Quality starts to go to hell when you delegate it. So when I say commitment, I mean CEOs in there working and doing things, not just saying, 'Yes, I bless this thing, and here's some money to do it.'）
  > 来源：IndustryWeek 1995年访谈 "Quality is Still Free"

#### 预防胜于检测
- "预防的秘诀在于审视过程，识别出错的机会。"
  （The secret of prevention is to look at the process and identify opportunities for error.）
  > 来源：Quality is Free, p. 68 (引述自 Wikipedia "Zero Defects")

- 用疫苗比喻："我们想给公司接种质量预防哲学的疫苗。与其建造世界上最大的天花医院，不如给人们接种疫苗，这样我们就不需要天花医院了。经营公司也是一样，要预防问题。"
  （What we want to do is vaccinate the company with the quality philosophy of prevention. So instead of setting up the world's largest smallpox hospital, we vaccinate people and then we don't need a smallpox hospital. That's the same way you have to run a company, by preventing the problems.）
  > 来源：IndustryWeek 1995年访谈

### 核心观点/名言
- "质量是符合要求，而不是好。"（Quality is conformance to requirements, not goodness.）
  > 来源：Quality is Free (1979)

- "第一次就把事情做对总是更便宜。"（It is always cheaper to do the job right the first time.）
  > 来源：Crosby 质量哲学官方资料

- "没有质量问题这种东西。"（There is no such thing as a quality problem.）
  —— 意思是质量问题都是管理问题、设计问题、流程问题
  > 来源：Philip B. Crosby 质量哲学（Chapter 5 - Philip B. Crosby 教学资料）

### 主要著作

| 序号 | 标题 | 类型 | 年份 | 出版社 |
|------|------|------|------|--------|
| 1 | Quality Is Free: The Art of Making Quality Certain（《质量免费》） | 专著 | 1979 | McGraw-Hill |
| 2 | Quality Without Tears: The Art of Hassle-Free Management | 专著 | 1984 | McGraw-Hill |
| 3 | Let's Talk Quality | 专著 | 1989 | McGraw-Hill |
| 4 | The Absolutes of Leadership | 专著 | 1996 | Jossey-Bass |
| 5 | Quality is Still Free: Making Quality Certain in Uncertain Times | 专著 | 1996 | McGraw-Hill |

---

## 五、周震漪——国内软件测试与质量保障领域推动者

### 基本背景
- 上海同思廷软件技术有限公司（TST）总经理
- 国际软件测试资质认证委员会中国分会（CSTQB®）副理事长
- 国际 TMMi® 基金会中国分会（TMMiCN）核心发起人及副理事长
- 国际需求工程委员会中国分会（CREB）核心发起人及副理事长
- TMMi® 主任评估师、ISTQB® 专家级证书获得者
- 上海杉达学院兼职教授
- 曾任 imbus Shanghai 总经理（imbus 是德国软件测试服务商）
- 深度参与多项国家标准与团体标准编制

> 来源：中国电子商会信息工程测试专业委员会官网 (ceietn.com)；第四届电子信息测试产业大会官方报道 (2026-07)；imbus AG 官网新闻 (2019-07)

### 核心理论与主要贡献

1. **国际标准体系的中国本土化推广**
   - 推动 ISTQB（国际软件测试资质认证）、TMMi（测试成熟度模型集成）、IREB（国际需求工程委员会）三大国际知识体系在中国大陆的引入与落地
   - 同思廷（TST）是上述三大协会在大中华地区唯一授权的考试和认证机构
   - 深度参与《系统与软件工程 生存周期过程 需求工程》等国家标准编制
   - 参与三项软件供应链安全团体标准编制
   
   > 来源：同思廷官网 (tsting.cn)；ceietn.com 大会报道；CSTQB 官网

2. **AI 时代软件质量保障的三大支柱框架**
   - **标准体系**：ISTQB、TMMi、IREB 等国际知识体系在 AI 时代的演进与融合
   - **过程体系**：AI 重塑软件全生命周期的质量保障流程——从需求、开发到测试、运维的全链路质量管控
   - **人才培养**：产教融合背景下软件质量人才的能力模型与培养路径
   
   > 来源：第四届电子信息测试产业大会主题演讲《AI时代软件质量保障的破局之道——标准、过程和人才培养体系》(2026-07)

3. **软件测试过程改进实践**
   - 作为 TMMi 主任评估师，推动企业测试过程从混乱到成熟的改进
   - 共同发起并组织 ISQE（国际软件质量工程论坛），参会规模约千人，是全球最大软件质量会议之一
   
   > 来源：imbus AG 官网 2019年报道（第10届 ISQE 论坛）；TMMi Foundation 官网

### 关于重点研究方向的观点

#### 软件测试过程改进
- 推动 TMMi 模型在中国企业的落地实践，TMMi 五级模型：初始级(1) → 受管理级(2) → 已定义级(3) → 已度量级(4) → 优化级(5)
  > 来源：TMMi Foundation 官网；TMMi 模型中文版发布

#### 软件质量度量
- 待确认（公开资料中未见系统阐述，需查阅其标准编制参与内容）

#### AI 与测试
- "人工智能不是取代人，而是在赋能人。未来属于那些能够将人类专业能力与人工智能能力相结合，从而交付可信、高效且高质量软件的专业人士和组织。"
  > 来源：第四届电子信息测试产业大会主题演讲 (2026-07)

- "在软件质量保障领域，AI 已通过自动化测试用例生成、智能回归测试、智能缺陷分析等手段，切实为企业提质增效。"
  > 来源：同上

- 2026年6月亚洲联盟质量大会（首尔）主题演讲《AI如何重塑软件全生命周期的质量》
  > 来源：ceietn.com 大会报道 (2026-07-28)

#### 质量文化
- "仅仅依靠测试是无法完全保障软件质量的——许多问题源于需求阶段。正因如此，质量必须贯穿软件生命周期的每一个阶段，结合先进的人工智能，并形成全员的质量文化，而不应被视为最后的检查环节。"
  > 来源：第四届电子信息测试产业大会主题演讲 (2026-07)

### 公开观点/言论汇总
- "质量必须贯穿软件生命周期的每一个阶段，结合先进的人工智能，并形成全员的质量文化，而不应被视为最后的检查环节。"
- "人工智能不是取代人，而是在赋能人。"
- "标准、过程与人才培养体系的深度融合，是 AI 时代软件质量保障破局的关键答案。"
  > 全部来源：第四届电子信息测试产业大会主题报道 (2026-07)

### 主要参与编制的标准和出版物

| 序号 | 标题 | 类型 | 年份 | 备注 |
|------|------|------|------|------|
| 1 | 《系统与软件工程 生存周期过程 需求工程》 | 国家标准 | 待确认 | 核心编制人 |
| 2 | 软件供应链安全团体标准（共三项） | 团体标准 | 待确认 | 参与编制 |
| 3 | TMMi® 模型中文版（R1.3） | 译著 | 待确认 | TMMiCN 组织翻译 |
| 4 | 《人工智能系统测试与 TMMi®》中文版 v1.0 | 白皮书/译著 | 待确认 | TMMi Foundation 发布 |
| 5 | 《DevOps 世界中的 TMMi®》中文版 | 白皮书/译著 | 待确认 | TMMi Foundation 发布 |

> 注：周震漪作为行业推动者，主要贡献在于国际标准的本土化推广、评估实践和人才培养，学术论文发表较少，以上主要基于公开活动报道和官方介绍。其参与的具体论文章节数量待确认。

---

## 六、郝丹——北京大学软件测试领域教授

### 基本背景
- 北京大学计算机学院博雅特聘教授、博士生导师
- 国际电气和电子工程师协会会士（IEEE Fellow）
- ACM 杰出会员（ACM Distinguished Member）
- 2002年哈尔滨工业大学学士，2008年北京大学博士
- 2015年获国家优秀青年科学基金
- 2016年入选"长江学者奖励计划"青年学者
- 2021年成为长江学者特聘教授
- 2020年度日本大川研究助成奖获得者
- 主要研究方向：软件工程，特别是软件测试与调试领域
- 研究成果应用于航天、电力、税务、搜索等多个领域

> 来源：北京大学计算机学院官网 (cs.pku.edu.cn/info/1234/2129.htm)；百度百科；Google Scholar (h-index 约41，引用约6800次)

### 核心理论与主要贡献

1. **测试用例优先级与约简（Test Case Prioritization & Reduction）**
   - 提出**统一测试优先级框架**（A Unified Test Case Prioritization Approach），系统整合多种优先级策略
   - 提出**按需测试约简**（On-demand Test Suite Reduction）技术，通过分析故障检测损失与覆盖率减少的关系来控制故障检测损失
   - 对回归测试优先级进行了大规模实证研究，揭示了实际软件演化中的性能表现
   
   > 来源：北大计算机学院官网；TOSEM 2014；ICSE 2012；TSE 2016

2. **测试预言（Test Oracle）**
   - 提出**蜕变关系自动推理**技术，首次实现蜕变关系（一种测试预言）的自动推断
   - 提出基于静态分析的预言数据识别技术，帮助开发者确定测试中应观察哪些变量
   
   > 来源：北大计算机学院官网研究成果介绍

3. **编译器测试（Compiler Testing）**
   - 对编译器测试技术进行系统实证研究，揭示核心影响因素
   - 提出多种优先级排序技术加速编译器测试
   - 提出学习排序测试程序的方法加速编译器测试
   
   > 来源：北大计算机学院官网；ICSE 2016; ICSE 2017

4. **变异测试（Mutation Testing）**
   - 提出**预测性变异测试**（Predictive Mutation Testing），通过预测模型大幅降低变异测试成本
   - 通过实证研究系统性分析变异体选择问题
   
   > 来源：北大计算机学院官网；ISSTA 2016

### AI与大模型驱动的软件测试研究成果

郝丹团队近年来在 LLM 与软件测试结合方向发表了一系列顶会论文：

1. **TestTailor：基于路径邻近测试的 LLM 高覆盖率测试生成**
   - 提出了一个神经符号框架，利用细粒度、面向路径的指导来引导 LLM 测试生成
   - 在不同 LLM 骨干网络上均表现出鲁棒性
   - FSE 2026 论文
   > 来源：Dan Hao 个人主页 (sites.google.com/view/danhao)

2. **ConTested：基于一致性辅助的 LLM 测试代码生成**
   - 提出轻量级交互框架，结合用户反馈有效引导一致性
   - 实验表明，只需最小的人力投入，性能即可显著提升
   - ISSTA 2025 论文
   > 来源：同上；ACM Digital Library (DOI: 10.1145/3728902)

3. **学习式顺序测试套件生成（From Greedy Steps to Global Optimization）**
   - 提出 TestDecision 框架，包含推理框架和强化学习训练流水线
   - 在分支覆盖率上提升 38.15%-52.37%，执行通过率提升 298.22%-558.88%
   - 7B 大小模型达到与更大的专有模型 GPT-5.2 相当的性能
   - ISSTA 2026 论文
   > 来源：Dan Hao 个人主页

4. **CausalRepair：通过双切片弥合 LLM 自动程序修复中的因果鸿沟**
   - 针对大语言模型自动程序修复中的因果性问题
   - ISSTA 2026 论文
   > 来源：Dan Hao 个人主页

### 其他重要研究方向

#### 公平性测试（Fairness Testing）
- 发表 TOSEM 2024 论文《Fairness Testing of Machine Translation Systems》，提出 FairMT 方法对机器翻译系统进行自动化公平性测试
- TOSEM 2026 论文《From Cryptic to Clear - Training on LLM Explanations to Detect Smart Contract Vulnerabilities》，利用 LLM 解释训练来检测智能合约漏洞
  > 来源：Dan Hao 个人主页期刊论文列表

#### 编译器自动调优
- PDCAT: Preference-Driven Compiler Auto-Tuning（FSE 2025，获 ACM SIGSOFT Distinguished Paper Award）
  > 来源：Dan Hao 个人主页

#### 自动化程序修复与回归测试
- 200万补丁上的大规模实证研究，揭示 APR 与回归测试之间的关系，提供实用指导
  > 来源：alphaXiv 个人档案

### 主要论文精选（按重要性排序）

| 序号 | 标题 | 类型 | 年份 | 会议/期刊 | 备注 |
|------|------|------|------|-----------|------|
| 1 | A Unified Test Case Prioritization Approach | 期刊 | 2014 | TOSEM 24(2) | 引用约103次 |
| 2 | To Be Optimal or Not in Test-Case Prioritization | 期刊 | 2016 | IEEE TSE 42(5) | 引用约101次 |
| 3 | Predictive Mutation Testing | 会议 | 2016 | ISSTA 2016 | 引用约79次 |
| 4 | On-demand Test Suite Reduction | 会议 | 2012 | ICSE 2012 | 引用约77次 |
| 5 | A Survey of Compiler Testing | 期刊 | 2020 | ACM Computing Surveys 53(1) | 综述论文 |
| 6 | A Survey on Regression Test-Case Prioritization | 期刊 | 2018 | Advances in Computers | 综述论文 |
| 7 | TestTailor: Generating High-Coverage Tests via Path-Proximal Tests with LLMs | 会议 | 2026 | FSE 2026 | LLM 测试生成 |
| 8 | ConTested: Consistency-Aided Tested Code Generation with LLM | 会议 | 2025 | ISSTA 2025 | LLM 测试生成 |
| 9 | From Greedy Steps to Global Optimization: Learning Sequential Test Suite Generation | 会议 | 2026 | ISSTA 2026 | 强化学习+测试生成 |
| 10 | PDCAT: Preference-Driven Compiler Auto-Tuning | 会议 | 2025 | FSE 2025 | 杰出论文奖 |
| 11 | Fairness Testing of Machine Translation Systems | 期刊 | 2024 | TOSEM 33(6) | 公平性测试 |
| 12 | Compiler Bug Isolation via Enhanced Test Program Mutation | 会议 | 2024 | ASE 2024 | 编译器调试 |

> 来源：Dan Hao 个人主页 (sites.google.com/view/danhao)；Google Scholar；OpenAlex

### 学术服务
- 国际会议 ASE 2021 程序委员会联合主席
- 国际会议 SANER 2022 程序委员会联合主席
- FSE 2024 New Faculty Co-Chair
- TSE 期刊编委 (2019-2023)
- STVR 期刊编委 (2019-2021)
- 多次担任 ICSE、FSE、ASE、ISSTA 等顶级会议程序委员会委员
  > 来源：Dan Hao 个人主页

### 公开观点（待补充直接引述）
- 待确认（公开学术访谈或演讲中的直接引述较少，主要观点通过论文和学术服务体现）

---

## 附录：六位专家比较概览

| 维度 | Humphrey | Deming | Juran | Crosby | 周震漪 | 郝丹 |
|------|----------|--------|-------|---------|--------|------|
| 领域 | 软件工程/过程 | 通用质量管理 | 通用质量管理 | 通用质量管理 | 软件测试/质量工程 | 软件测试/学术研究 |
| 核心方法 | CMM/PSP/TSP | 十四点/SoPK | 质量三部曲 | 四绝对/零缺陷 | 标准+过程+人才 | 测试优化/LLM测试 |
| 质量定义 | 满足客户 | 持续改进 | 适用性 | 符合要求 | 全生命周期质量 | 故障检测能力 |
| 预防观 | 个人缺陷预防 | 内建质量 | 质量策划 | 预防系统 | 全链路质量管控 | 测试左移/优先级 |
| 人的因素 | 个人责任与自豪 | 消除恐惧/自豪感 | 管理层领导 | 文化与承诺 | 人才培养体系 | （学术研究为主） |

---

## 信息来源汇总

### 主要机构官网
- CMU Software Engineering Institute: sei.cmu.edu
- The W. Edwards Deming Institute: deming.org
- Juran Institute: juran.com
- TMMi Foundation: tmmi.org
- 北京大学计算机学院: cs.pku.edu.cn
- 上海同思廷: tsting.cn
- 中国电子商会信息工程测试专业委员会: ceietn.com

### 主要书籍
- Humphrey, W.S. Managing the Software Process. Addison-Wesley, 1989.
- Humphrey, W.S. A Discipline for Software Engineering. Addison-Wesley, 1995.
- Deming, W.E. Out of the Crisis. MIT Press, 1986/2000.
- Deming, W.E. The New Economics. MIT Press, 1993.
- Juran, J.M. Quality Control Handbook. McGraw-Hill, 1951.
- Juran, J.M. Juran on Quality by Design. Free Press, 1992.
- Crosby, P.B. Quality Is Free: The Art of Making Quality Certain. McGraw-Hill, 1979.

### 主要技术报告
- CMU/SEI-2000-TR-022: The Personal Software Process (PSP), Watts S. Humphrey
- CMU/SEI-2000-TR-023: The Team Software Process (TSP), Watts S. Humphrey
- CMU/SEI-2006-TR-017: Systems of Systems: Scaling Up the Development Process, Watts Humphrey

### 主要学术数据库
- Google Scholar (Dan Hao profile)
- OpenAlex (Dan Hao: 146 works, 4808 citations, h-index 41)
- ACM Digital Library
- IEEE Xplore
