# 故障复盘实验室

> 专注于 AI / 大模型领域技术故障分析与深度复盘的独立博客。
> 基于 GitHub Pages 构建，纯 HTML/CSS，无构建依赖。

## 项目简介

本博客围绕「9·3 大模型服务集体故障」事件展开深度复盘，从技术根因、业务影响、架构启示、行业趋势等多个维度进行分析。

## 快速预览

```bash
# 本地启动（Python 3）
python3 -m http.server 8080

# 然后访问 http://localhost:8080
```

或直接用浏览器打开 `index.html`。

## 文章列表

### 核心复盘
- **[9·3 大模型服务集体故障：全链路深度复盘](posts/deep-dive-postmortem.html)** — 事件全貌、时间线、原因分析与启示

### 配套分析
- **[技术根因推测：为什么多家模型同时故障？](posts/root-cause-analysis.html)** — 从基础设施到软件栈的多维度分析
- **[影响评估：一次故障让多少业务停摆？](posts/business-impact.html)** — 经济损失与行业影响量化分析
- **[架构启示：LLM 应用如何设计多模型容灾？](posts/architecture-lessons.html)** — 高可用架构设计指南
- **[行业观察：大模型基础设施的脆弱性与未来](posts/industry-insights.html)** — 集中化风险与行业演进趋势

## 目录结构

```
llm-outage-postmortem/
├── index.html              # 首页（时间线 + 文章列表）
├── about.html              # 关于页面
├── css/
│   └── style.css           # 全局样式（暗色主题）
├── posts/                  # 文章目录
│   ├── deep-dive-postmortem.html
│   ├── root-cause-analysis.html
│   ├── business-impact.html
│   ├── architecture-lessons.html
│   └── industry-insights.html
└── README.md
```

## 部署到 GitHub Pages

1. 将本目录内容推送到 GitHub 仓库
2. 进入 Settings → Pages
3. Source 选择 `Deploy from a branch`
4. Branch 选择 `main`，目录选择 `/ (root)`
5. 等待部署完成

## 写新文章

1. 在 `posts/` 下新建 `.html` 文件，参考现有文章的 HTML 结构
2. 在 `index.html` 中添加文章卡片链接

## 自定义

- 修改 `css/style.css` 中的 CSS 变量（`:root` 块）调整主题色
- 暗色主题为主，适合技术内容阅读
