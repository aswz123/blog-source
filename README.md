# 我的博客

基于 [Hugo](https://gohugo.io/) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 主题的个人博客。

在线地址：http://122.51.94.127

## 本地开发

```bash
# 克隆仓库（含主题子模块）
git clone --recurse-submodules git@github.com:aswz123/blog-source.git
cd blog-source

# 启动开发服务器
hugo server -D

# 访问 http://localhost:1313
```

## 发布文章

```bash
# 创建新文章
hugo new posts/新文章标题.md

# 编辑 content/posts/新文章标题.md
# 将 draft: true 改为 draft: false

# 本地预览
hugo server -D

# 提交并推送，自动部署
git add content/
git commit -m "feat: new post"
git push origin main
```

## 技术栈

- Hugo (Static Site Generator)
- PaperMod (Theme)
- GitHub Actions (CI/CD)
- Nginx (Web Server)
