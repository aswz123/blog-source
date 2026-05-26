# Cloudflare 免费接入指南

> 适用：Hugo 静态博客 + 自建服务器（2C2G），无需额外费用

---

## 第一步：添加站点到 Cloudflare

1. 注册 [Cloudflare](https://cloudflare.com)
2. 点击 **Add a Site** → 输入 `aswzblog.me`
3. 选择 **Free** 套餐
4. Cloudflare 会自动扫描现有 DNS 记录

## 第二步：修改 DNS 托管

1. Cloudflare 会给你两个 NS 地址（如 `alex.ns.cloudflare.com` + `sue.ns.cloudflare.com`）
2. 去你的域名注册商（Namecheap/GoDaddy/阿里云 等）把 DNS 服务器改成上面两个地址
3. 等待 DNS 生效（通常 5-30 分钟）

## 第三步：DNS 记录配置

在 Cloudflare DNS 面板添加以下记录：

| Type | Name | Content | Proxy |
|------|------|---------|-------|
| A | `@` | 你的服务器 IP |  Proxied |
| A | `www` | 你的服务器 IP |  Proxied |

>  Proxied = 橙色云朵开启，这样流量才会经过 Cloudflare CDN

## 第四步：SSL/TLS 设置

进入 **SSL/TLS → Overview**，选择：

- **Flexible**：源站没有 HTTPS 时使用（Cloudflare ↔ 用户加密，Cloudflare ↔ 源站不加密）
- **Full**：源站有自签证书时使用
- **Full (strict)**：源站有受信任证书时使用（推荐，但需要 Let's Encrypt）

> 推荐先用 **Flexible** 快速上线，然后配置 Let's Encrypt 后切到 **Full (strict)**

## 第五步：缓存规则（关键）

进入 **Caching → Cache Rules**，创建规则：

### 规则1：静态资源长缓存
```
Field: URI Path
Operator: ends with
Value: .css, .js, .png, .jpg, .webp, .svg, .woff2, .ico
Then: Cache - Eligible for cache
Edge TTL: 30 days
Browser TTL: 7 days
```

### 规则2：HTML 页面短缓存
```
Field: URI Path
Operator: ends with
Value: .html
Then: Cache - Eligible for cache
Edge TTL: 1 hour
Browser TTL: 30 minutes
```

### 规则3：RSS/Sitemap
```
Field: URI Path
Operator: contains
Value: index.xml, sitemap.xml, index.json
Then: Cache - Eligible for cache
Edge TTL: 1 hour
```

## 第六步：自动优化

进入 **Speed → Optimization**：

| 功能 | 设置 |
|------|------|
| **Auto Minify** | 勾选 HTML、CSS、JS |
| **Brotli** | 开启 |
| **Early Hints** | 开启 |
| **Rocket Loader** | 关闭（可能影响 Hugo 站点） |

进入 **Images → Polish**（Pro 功能，免费套餐不可用，跳过即可）

## 第七步：安全设置

进入 **Security → Settings**：

| 设置 | 值 |
|------|-----|
| Security Level | Medium |
| Challenge Passage | 30 min |
| Browser Integrity Check | On |

进入 **Security → Bots**：
- **Bot Fight Mode**：开启（免费）
- 这会阻挡大部分恶意爬虫

## 第八步：验证

1. 浏览器打开 `https://aswzblog.me`
2. 检查 Response Headers 是否有 `cf-cache-status: HIT`
3. `curl -I https://aswzblog.me` 确认返回 `server: cloudflare`

---

## 最终架构

```
用户 → Cloudflare CDN (边缘节点) → 你的 2C2G 服务器
        ↑ 缓存命中率 ~95%            ↑ 仅处理缓存未命中的请求
```

接入 Cloudflare 后，**95%+ 的请求会被 CDN 直接响应**，你的 2C2G 服务器只需要处理极少量的回源请求。日万级 UV 毫无压力。
