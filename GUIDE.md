# 博客改功能指南

本文档记录本站（Hugo + PaperMod + GitHub Pages）后续增改功能的操作方法。

---

## 一、日常流程：改完怎么看到效果

### 本地预览（推荐，边改边看，秒级刷新）

```bash
cd D:\OneDrive\Desktop\zedward
hugo server -D
```

然后浏览器打开 http://localhost:1313/ 。改动配置、文章、样式都会**自动热重载**，不用手动刷新。
`-D` 表示同时预览草稿（`draft: true`）的文章。按 `Ctrl+C` 停止。

> 本地预览用 `hugo server`，**不要**用 `hugo`（那个只生成 `public/`，不启服务器）。

### 发布上线

```bash
git add -A
git commit -m "描述你改了什么"
git push origin main
```

推送到 `main` 分支后，GitHub Actions 会自动构建并部署，**1～2 分钟**后线上生效。
在仓库 **Actions** 标签页可以看到构建进度，绿色勾代表成功。

### 构建报错怎么排查

先本地跑一次，错误会直接打在终端里（本地能过，CI 基本就能过）：

```bash
hugo --minify
```

---

## 二、目录结构：每个文件夹管什么

| 路径 | 作用 | 能改吗 |
|---|---|---|
| `hugo.yaml` | **站点总配置**，开关功能主要靠它 | ✅ 常改 |
| `content/` | **文章内容**，Markdown 文件 | ✅ 常改 |
| `static/` | 静态文件，原样复制到网站根目录（图片、`css/`、`favicon.ico`） | ✅ 常改 |
| `layouts/` | **覆盖主题模板**，同路径同名文件会替换主题的 | ✅ 可改 |
| `i18n/` | 界面文案汉化 | ✅ 可改 |
| `assets/` | 需要 Hugo 处理的资源（图片压缩、SCSS 编译） | ✅ 可改 |
| `themes/PaperMod/` | **主题本体（git submodule）** | ❌ **别直接改** |

### ⚠️ 最重要的一条：不要直接改 `themes/PaperMod/`

`themes/PaperMod/` 是 git submodule，指向主题的官方仓库。你在这里面的任何修改：

- **不会被提交**到你的仓库（push 上去也是空的）
- 下次更新主题（`git submodule update`）会**全部丢失**
- 别人 clone 你的仓库时**看不到**这些改动

**正确做法**：在站点根目录建同路径的文件来覆盖它。

---

## 三、改东西的四个层次（从简单到复杂）

### 层次 1：改开关 —— 只动 `hugo.yaml`

绝大多数需求（字号、显示阅读时间、换高亮配色、加社交图标）都属于这类。

例如关掉分享按钮、把代码高亮换成 `github` 配色：

```yaml
params:
  ShowShareButtons: false          # 关掉分享按钮

markup:
  highlight:
    style: github                  # 换高亮配色
```

改完用 `hugo server -D` 看效果，满意就 commit + push。

### 层次 2：改单篇文章的行为 —— 只动该文章的 front matter

文章顶部的 `---` 之间是 front matter，可以**单独**控制这一篇：

```markdown
---
title: "我的文章"
date: 2026-02-01
draft: false                # true = 草稿，不会发布
tags: ["Hugo", "教程"]
categories: ["技术"]
summary: "列表页显示的摘要"
ShowToc: false              # 这篇不显示目录（覆盖全局设置）
ShowBreadCrumbs: false      # 这篇不显示面包屑
weight: 1                   # 排序权重，越小越靠前
cover:                      # 封面图
  image: "images/cover.jpg"
  hiddenInList: true        # 列表页不显示封面
---

正文从这里开始……
```

新建文章用：

```bash
hugo new content posts/文章标题.md
```

### 层次 3：改外观细节 —— 加自定义 CSS（最常用）

**不要**去改主题的 CSS。本站已经配好了自定义样式入口：

1. 把样式写进 `static/css/custom.css`
2. 它已被 `layouts/_partials/extend_head.html` 自动引入，全站生效

```css
/* static/css/custom.css */
.post-content {
    font-size: 17px;
}
```

> 这个文件目前已有代码块字体、正文行高的设置，追加即可。

### 层次 4：改模板结构 —— 覆盖主题模板

想把文章页布局改一改、加个侧边栏、改首页结构时用这招。

**原理**：Hugo 查找模板时，站点 `layouts/` 的优先级**高于**主题的 `themes/PaperMod/layouts/`。

**做法**：把主题里要改的文件，复制到站点根目录的**相同相对路径**下再改。

```bash
# 例：要改文章页模板
copy themes\PaperMod\layouts\single.html layouts\single.html
# 然后编辑 layouts\single.html
```

PaperMod 常用的可覆盖文件：

| 主题里的路径 | 复制到 | 控制什么 |
|---|---|---|
| `layouts/single.html` | `layouts/single.html` | 文章页整体结构 |
| `layouts/list.html` | `layouts/list.html` | 列表页 / 首页结构 |
| `layouts/404.html` | `layouts/404.html` | 404 页面 |
| `layouts/_partials/head.html` | `layouts/_partials/head.html` | `<head>` 全部内容 |
| `layouts/_partials/footer.html` | `layouts/_partials/footer.html` | 页脚 |

### 层次 4.5：优先用「扩展钩子」而不是整份覆盖（推荐）

整份覆盖模板有个缺点：**主题升级后你不会自动获得改进**，还可能冲突。
PaperMod 提供了 3 个空钩子，专门给你插内容，**不覆盖任何东西**：

| 钩子文件 | 插入位置 | 适合放什么 |
|---|---|---|
| `layouts/_partials/extend_head.html` | `</head>` 之前 | 自定义 CSS、统计代码、meta 验证标签 |
| `layouts/_partials/extend_footer.html` | 页脚 | 自定义 JS、页脚脚本 |
| `layouts/_partials/extend_post_content.html` | 文章正文之后 | 文章末尾的推广位、打赏、相关阅读 |

**本站已创建 `layouts/_partials/extend_head.html`**（用来引入 `custom.css`），另外两个需要时自建即可。

例：想在每篇文章末尾加一行版权声明，新建 `layouts/_partials/extend_post_content.html`：

```html
<hr>
<p style="font-size: 0.85em; opacity: 0.7;">
    本文作者：zedward ｜ 转载请注明出处
</p>
```

---

## 四、常见需求速查

### 加社交图标

编辑 `hugo.yaml` 的 `params.socialIcons`。图标名必须是主题支持的（见 `themes/PaperMod/layouts/_partials/svg.html`，共 140+ 个，如 `github`、`x`、`twitter`、`bilibili`、`zhihu`、`email`、`rss`、`telegram`）。

```yaml
params:
  socialIcons:
    - name: github
      url: "https://github.com/zedward-0417"
    - name: bilibili
      url: "https://space.bilibili.com/你的ID"
```

### 换首页风格：简介模式 ↔ 头像模式

**当前是简介模式**（`homeInfoParams`）。想换成居中的头像 + 按钮模式：

```yaml
params:
  # 先把 homeInfoParams 整段注释掉
  profileMode:
    enabled: true
    title: "zedward"
    subtitle: "记录学习与踩坑"
    imageUrl: "images/avatar.jpg"     # 放到 static/images/avatar.jpg
    imageWidth: 150
    imageHeight: 150
    buttons:
      - name: 文章
        url: /posts/
      - name: 标签
        url: /tags/
```

> 两种模式**不要同时开**，`profileMode.enabled: true` 会优先。

### 加网站统计

推荐用 Cloudflare 或 Umami 这类不需要 Cookie 提示的。以 Cloudflare Web Analytics 为例，
在 `layouts/_partials/extend_head.html` 里追加（把 token 换成你自己的）：

```html
<script defer src='https://static.cloudflareinsights.com/beacon.min.js'
        data-cf-beacon='{"token": "你的TOKEN"}'></script>
```

Hugo 内置的 Google Analytics 也可用（主题已支持），在 `hugo.yaml` 加：

```yaml
services:
  googleAnalytics:
    ID: "G-XXXXXXXXXX"
```

### 加评论区

主题的 `comments.html` 是**空文件**，需要自己接第三方服务（Giscus 最省事，基于 GitHub Discussions，无广告）。
新建 `layouts/_partials/comments.html` 写入服务商给的代码，然后在 `hugo.yaml` 把 `params.comments` 改成 `true`。

### 换 favicon

把图标文件放到 `static/`，命名保持主题期望的名字即可覆盖：

```
static/favicon.ico
static/favicon-16x16.png
static/favicon-32x32.png
static/apple-touch-icon.png
```

### 加自定义页面（如「友链」）

```bash
hugo new content links.md
```

写入：

```markdown
---
title: "友链"
url: "/links/"
ShowToc: false
hidemeta: true
---

内容……
```

再在 `hugo.yaml` 的 `menu.main` 里加一项 `url: /links/`。

### 更新主题到最新版

```bash
git submodule update --remote themes/PaperMod
git add themes/PaperMod
git commit -m "更新 PaperMod 主题"
git push
```

> 因为你的自定义都放在站点根目录（`layouts/`、`static/`、`i18n/`、`hugo.yaml`），
> 更新主题**不会**覆盖掉这些改动，这就是分层的好处。

---

## 五、排查清单

| 现象 | 先查这里 |
|---|---|
| 本地改了没反应 | 是不是没在跑 `hugo server -D`？确认地址是 `localhost:1313` |
| push 了线上没变 | 去仓库 **Actions** 看构建是否失败（红叉） |
| 线上 404 | **Settings → Pages** 的 Source 是否为 `GitHub Actions` |
| 构建报 `failed to load config` | `hugo.yaml` 的缩进 / 语法错误，注意 YAML 不能用 Tab，只能用空格 |
| 目录、阅读时间显示英文 | `i18n/zh-cn.yaml` 是否被删或改坏 |
| 样式改了没用 | 浏览器缓存，`Ctrl+F5` 强刷；或 `custom.css` 路径写错 |
| 首页空白没文章 | 文章 `draft: true`？或不在 `content/posts/` 下（`mainSections` 只认 `posts`） |
