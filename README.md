# 我的博客

基于 Hugo + PaperMod 的个人博客。**在线地址：[https://aswzblog.me](https://aswzblog.me)**

---

## 快速上手

### 环境准备

本机需安装 Hugo（Extended 版）：

```bash
# Windows
winget install Hugo.Hugo.Extended

# macOS
brew install hugo

# Linux
sudo apt install hugo
```

验证安装：`hugo version`，确保输出含 `extended` 字样。

### 克隆项目

```bash
git clone --recurse-submodules git@github.com:aswz123/blog-source.git
cd blog-source
```

> 如果你之前已经克隆但没带子模块，运行：`git submodule update --init --recursive`

---

## 管理后台（在线写作）

项目集成了 [Sveltia CMS](https://sveltiacms.app/)（Decap CMS 社区分支），可以直接在浏览器里写文章，像 WordPress 一样。

访问 **[https://aswzblog.me/admin/](https://aswzblog.me/admin/)** 进入后台。

### 登录

需要一个 GitHub Personal Access Token（Classic）：

1. 打开 https://github.com/settings/tokens/new
2. Note 填 `Blog CMS`，勾选 **`repo`** scope
3. 生成后复制 token，在后台登录页面输入即可

### 使用

- **写文章**：点击"文章"→"新建文章"，编辑好内容后点"保存"→"发布"
- **上传图片**：编辑器支持直接粘贴或拖拽图片
- **编辑"关于我"**：在"页面"→"关于我"里直接改
- 保存后会自动 commit 到 GitHub，约 30 秒部署上线

> 如果文章不想发布，在后台把"草稿"开关打开，保存后只会存在仓库里，不会出现在网站上。

---

## 本地写作

### 创建新文章

```bash
hugo new posts/文章的文件名.md
```

这会生成 `content/posts/文章的文件名.md`，内容如下：

```yaml
---
title: "文章的文件名"
date: 2026-05-23T22:30:00+08:00
draft: true          # ← 改为 false 才会发布
categories: []
tags: []
summary: ""
---
```

### 填写文章信息

| 字段 | 说明 | 示例 |
|------|------|------|
| `title` | 文章标题 | `"Go 语言并发编程入门"` |
| `date` | 发布日期 | `2026-05-23T22:30:00+08:00` |
| `draft` | `true`=草稿不发布，`false`=发布 | `false` |
| `categories` | 分类（通常 1 个） | `["技术"]` |
| `tags` | 标签（可多个） | `["Go", "并发", "goroutine"]` |
| `summary` | 摘要，显示在首页卡片上 | `"介绍 Go 语言 goroutine 的基本用法。"` |

### 写正文

下面直接写 Markdown：

```markdown
## 什么是 goroutine

Goroutine 是 Go 语言中的轻量级线程……

## 基本用法

```go
go func() {
    fmt.Println("Hello, goroutine!")
}()
```

## 总结

goroutine 让并发编程变得简单。
```

### Markdown 常用语法

| 效果 | 写法 |
|------|------|
| **粗体** | `**粗体**` |
| `行内代码` | `` `行内代码` `` |
| 链接 | `[文字](https://example.com)` |
| 图片 | `![描述](/images/图片名.png)` |
| 引用 | `> 引用内容` |
| 代码块 | <code>```go 换行 代码 换行 ```</code> |
| 列表 | `- 项目` 或 `1. 项目` |
| 分隔线 | `---` |

---

## 预览文章

```bash
hugo server -D
```

打开 http://localhost:1313，保存 Markdown 文件后浏览器自动刷新。-D 参数会显示草稿文章。

---

## 发布文章

确认 `draft: false`，然后：

```bash
git add content/posts/新文章.md
git commit -m "新文章: Go 语言并发编程入门"
git push origin main
```

推送后 **约 30 秒**自动部署到 https://aswzblog.me。

> 如果使用管理后台写作，保存后自动完成 commit + push，无需手动操作。

---

## 添加图片

把图片放到 `static/images/` 目录，文章中引用：

```markdown
![图片描述](/images/my-image.png)
```

---

## 编辑"关于我"页面

编辑 `content/about.md`，直接改内容，改完推送即可更新。

---

## 项目结构

```
blog-source/
├── content/
│   ├── posts/          # ← 文章都放这里
│   │   └── hello-world.md
│   ├── about.md        # ← 关于我页面
│   └── search.md       # ← 搜索页面（不要删）
├── static/
│   ├── admin/           # ← 管理后台（Sveltia CMS）
│   │   ├── index.html
│   │   └── config.yml
│   └── images/          # ← 图片等静态文件
├── archetypes/
│   └── default.md      # ← 新文章的默认模板
├── themes/
│   └── PaperMod/       # ← 主题（git 子模块，不要直接改）
├── hugo.yaml           # ← 网站配置
├── .github/workflows/
│   └── deploy.yml      # ← 自动部署配置
└── README.md
```

---

## 修改网站配置

编辑 `hugo.yaml` 可按需调整：

- `title` — 网站标题
- `homeInfoParams.Title / Content` — 首页欢迎语
- `socialIcons` — 社交链接（支持 GitHub、Twitter、Email 等）
- `paginate` — 每页显示文章数

改完推送即可生效。

---

## 常见问题

**Q: 推送后网站没更新？**
A: 到 https://github.com/aswz123/blog-source/actions 检查工作流运行状态，红色=失败，点进去看日志。

**Q: hugo server 报错？**
A: 检查是否在 `blog-source` 目录下运行，以及 Hugo 是否为 Extended 版本。

**Q: 怎么删除文章？**
A: 删除 `content/posts/xxx.md` 文件，提交推送即可。

**Q: 文章不想公开发布？**
A: 保持 `draft: true`，本地可以预览但不会部署到网站。

---

## 技术栈

- [Hugo](https://gohugo.io/) — Go 编写的静态站点生成器
- [PaperMod](https://github.com/adityatelange/hugo-PaperMod) — 轻量、快速的 Hugo 主题
- [GitHub Actions](https://github.com/features/actions) — 自动构建和部署
- Nginx — Web 服务器
