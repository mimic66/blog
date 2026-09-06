# 模型测评实验室

> 专注于大语言模型深度测评的独立博客，基于 GitHub Pages 构建。

## 快速开始

### 本地预览

直接在浏览器中打开 `index.html` 即可预览，无需任何构建工具。

或使用本地服务器：

```bash
# Python 3
python3 -m http.server 8080

# Node.js (npx)
npx serve .
```

然后访问 http://localhost:8080

### 部署到 GitHub Pages

1. 在 GitHub 上创建一个新仓库，命名为 `blog`（或其他名称）
2. 将本目录下的所有文件推送到仓库的 `main` 分支
3. 进入仓库 Settings → Pages
4. Source 选择 `Deploy from a branch`
5. Branch 选择 `main`，目录选择 `/ (root)`
6. 点击 Save，等待几分钟后即可通过 `https://<你的用户名>.github.io/blog/` 访问

如果使用 `<username>.github.io` 命名的仓库，则部署后访问 `https://<你的用户名>.github.io/`

## 目录结构

```
blog/
├── index.html              # 首页（文章列表）
├── about.html              # 关于页面
├── css/
│   └── style.css           # 全局样式
├── posts/                  # 文章目录
│   ├── gpt-4o-vs-claude-opus.html
│   ├── qwen2-5-72b-review.html
│   ├── doubao-seed-2-1-review.html
│   └── gemini-1-5-pro-1m-context.html
└── README.md
```

## 写新文章

1. 在 `posts/` 目录下新建一个 `.html` 文件，参考现有文章的格式
2. 复制现有文章的 HTML 结构作为模板，修改标题、日期、内容
3. 在 `index.html` 的 `.post-list` 中添加一条新的文章卡片链接

## 自定义

- 修改 `css/style.css` 中的 CSS 变量（`:root` 块）可以快速调整主题色、背景色等
- 修改 `index.html` 的 hero 区域可以自定义博客标题和副标题
- 修改 `about.html` 可以自定义关于页面内容
