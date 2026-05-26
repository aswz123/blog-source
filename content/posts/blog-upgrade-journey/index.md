---
title: "博客升级血泪史：SEO、WebP、Cloudflare 踩坑全记录"
date: 2026-05-26T23:00:00+08:00
categories: ["技术"]
tags: ["博客", "Hugo", "SEO", "Nginx", "Cloudflare", "运维"]
---

原本只是想让博客"做大一点"，结果一个下午经历了 SEO 全栈升级、Nginx 调参、CDN 接入失败、DNS 反复横跳，最终回归朴实。

## 起点：2C2G 服务器能走多远？

我的博客架在腾讯云 2 核 2G 的轻量服务器上，Ubuntu 20.04 + Nginx + Hugo，域名是 `aswzblog.me`。日常访问量约等于零，但人要有梦想。

和 AI 搭档分析后发现，2C2G 跑静态博客完全够用——瓶颈不在硬件，在于没有做优化。于是开干。

## SEO 全家桶

PaperMod 主题的 SEO 底子不错，但需要 `--environment production` 才会输出 OG 标签和 JSON-LD 结构化数据。之前的 CI 部署脚本只跑了 `hugo --minify`，所以搜索引擎看到的页面是裸的。

改动：
- **JSON-LD 结构化数据**：BlogPosting + BreadcrumbList，搜索引擎富摘要
- **OpenGraph 默认图**：写了个 SVG 模板，没配封面图的文章也能优雅分享。覆盖了 PaperMod 的 `opengraph.html`，加了一行回退逻辑——没有文章封面 → 没有页面图片 → 用站点默认 OG 图
- **Meta Description**：文章没写 description 时自动用前 160 字摘要填充
- **RSS 全文输出**：`ShowFullTextinRSS: true`，RSS 读者不用跳转就能看完

这些改动一行服务器操作都不需要，纯 Hugo 模板 + 配置。

## WebP 自动转换

博客里的截图都是 PNG，一篇文章三张图加起来 400 多 KB。在 Hugo 的 markup render hook 里加了一层：

```go
{{ if in (slice "jpg" "jpeg" "png") $img.MediaType.SubType }}
  {{ $webp := $img.Process "webp q82" }}
  <picture>
    <source srcset="{{ $webp.RelPermalink }}" type="image/webp">
    <img src="..." loading="lazy" decoding="async">
  </picture>
{{ end }}
```

效果立竿见影——213KB 的 PNG 转成 69KB 的 WebP，体积减少 68%。Hugo 在构建时处理，不影响服务器性能。

## Nginx 上手段

原先的 Nginx 配置就是装完系统默认的，gzip 参数全注释、没有安全头、没有缓存策略。SSH 上去改了一波：

- gzip 全开 + `gzip_static on`
- 安全头：`X-Content-Type-Options`、`X-Frame-Options`、`Referrer-Policy`
- 指纹化资源缓存 1 年，图片缓存 30 天，RSS/sitemap 缓存 1 小时
- `server_tokens off` 隐藏版本号

配完 `sudo nginx -t && sudo systemctl reload`，平滑重载。

## Cloudflare 翻车

前面都顺风顺水，到了 CDN 环节开始出事。

域名接入 Cloudflare、DNS 切过去、代理打开，然后——网站挂了。

Cloudflare 返回 525 错误：SSL 握手失败。

排查过程：

1. **是不是端口没开？** 不是，80 和 443 都通，TCP 能连上。
2. **是不是 Nginx 配置有问题？** 不是，从服务器本地 curl 完全正常。
3. **是不是 SSL 证书有问题？** 不是，Let's Encrypt 证书有效，中国 IP 能正常访问。
4. **是不是腾讯云安全组？** 不是，端口放的是 `0.0.0.0/0`。

最后用 `openssl s_client` 对照测试才发现：**不带 SNI 的 SSL 连接成功，带 SNI 的被拦截**。

腾讯云防火墙在 DPI 层面检测到境外 IP 的 TLS SNI 握手，直接静默丢弃。Cloudflare 的边缘节点连源站时必然会发 SNI，所以必死。Flexible 模式也不优雅。

折腾了两小时，最终决定：**不搞 CDN 了**。

## 回归简单

把 DNS 从 Cloudflare 迁回阿里云 `dns31.hichina.com`，加上 A 记录指向服务器 IP。十分钟生效。

最终架构：

```
用户 → 阿里云 DNS → 腾讯云 2C2G → Nginx 直出静态文件
```

没有 CDN、没有中间层、没有代理。配合 Nginx 的 gzip + 缓存 + 安全头，现阶段完全够用了。

---

## 收获

1. **Hugo 的模板系统真的很强**：SEO、图片处理、RSS 全文，全部可以通过覆盖主题模板实现，不需要改主题源码。
2. **WebP 值得无脑开**：PNG → WebP 减少 60-80% 体积，而且 Hugo 构建时处理，零运行时开销。
3. **中国服务器 + Cloudflare = 看运气**：腾讯云/阿里云的安全产品可能在网络层拦截境外 SSL 流量，不是改 Nginx 能解决的。服务国内用户的话直连就够了。
4. **AI 搭档做运维是真的爽**：SSH、Nginx 配置、DNS 排查，全程对话完成，根本不用记命令。

以后流量大了再说 CDN，现在，先写文章。
