---
title: 静态博客接入R2存储
published: 2026-09-09
description: 在静态博客中使用Cloudflare R2存储图片，10G空间、全球加速、流量免费；Cloudflare赛博大善人。
image: "https://cloudflare-r2.4w.ink/2026/09/09/1788961210228-9131.png"
tags: [对象存储,白嫖,Cloudflarer]
category: 白嫖日记
slug: baipiao/cloudflarer2
---

# Cloudflare R2 介绍

## 一、R2 是什么

R2 是 Cloudflare 推出的对象存储服务，和 AWS S3 是同类产品，支持 S3 API。你可以把它当成一个“无限扩容的硬盘”，用来存图片、视频、附件、静态资源等。

与普通对象存储不同的是，R2 天生长在 Cloudflare 全球网络上，所以访问同一份资源时，不需要再额外接 CDN。绑定一个域名，就能让内容通过 Cloudflare 的全球边缘节点加速分发。

## 二、R2 的核心亮点

### 1. 没有出口流量费

对象存储通常有两种收费：**存储费 + 流量费**。很多云厂商的流量费甚至比存储费还贵。R2 的“杀手锏”就是：**出站流量费为 0**。不管你是 1TB 流量还是 10TB 流量，只要从 R2 里读取出来，都不收流量钱。

### 2. 永久免费额度

R2 每个账户每月有：

```text
10GB 存储
100 万次写入请求（Class A）
1000 万次读取请求（Class B）
```

这个额度是“持久免费”，不是 12 个月试用。对大多数个人博客来说，只要图片总量不超过 10GB，基本等于不花钱。

### 3. 自带 CDN，全球加速

R2 存储桶绑定自定义域名后，Cloudflare 会自动把内容分发到边缘节点。第一次请求后，后续访问直接由 CDN 响应，速度快，而且不重复消耗 R2 的读取次数。对静态博客的图片加载来说非常友好。

### 4. S3 兼容

R2 提供 S3 API，所以现有支持 S3 的工具都能用：AWS CLI、rclone、PicGo、uPic、Typora 等。你只需要把 endpoint 指向 Cloudflare 的 R2 地址即可，迁移成本非常低。

### 5. 与 Cloudflare 生态无缝集成

R2 可以配合 Cloudflare Workers 做图片处理、防盗链、签名鉴权等；也可以直接绑定到 Pages/Static Assets 项目中使用。

## 三、R2 定价一览

| 项目 | 免费额度 | 超出后单价 |
| --- | --- | --- |
| 存储 | 10GB / 月 | $0.015 / GB / 月，约 ¥0.11 / GB / 月 |
| Class A（写/列举/删除等） | 100 万次 / 月 | $0.36 / 百万次 |
| Class B（读/取响应头等） | 1000 万次 / 月 | $0.036 / 百万次 |
| 出口流量 | 免费 | 0 |

简单估算：如果你的博客图片总大小不到 10GB，每月读取次数不超过 1000 万次，那么你每月需要支付的费用是 **0 元**。即使涨到 50GB，存储费也仅为：

```text
50 × $0.015 = $0.75 / 月
```

约 5 元人民币。

## 四、R2 与亚马逊 S3 / 传统 OSS 对比

| 维度 | Cloudflare R2 | AWS S3 | 阿里云 OSS |
| --- | --- | --- | --- |
| 免费额度 | 10GB 永久免费 | 新用户 5GB，12 个月 | 有限时长试用，通常不长期 |
| 出口流量费 | 0 | 按流量付费，价格较高 | 按流量付费，国内 CDN 流量也收费 |
| 加速方式 | 自带 Cloudflare CDN | 需搭配 CloudFront，另行计费 | 自带 CDN，但流量费不能省 |
| S3 兼容 | 兼容 | 原生 | 兼容 |
| 门槛 | 注册 Cloudflare 即可 | 需要绑定信用卡 | 需要实名认证 |

## 五、静态博客接入 R2 的完整思路

如果你的博客是用 Hugo/Hexo/Vitepress 等静态站点生成的，R2 非常适合作为“图床”或“附件桶”。一般步骤：

1. 在 Cloudflare 控制台创建 R2 Bucket。
2. 在 R2 API Tokens 页面创建一个 Token，拿到 Access Key ID / Secret Access Key。
3. 在 Bucket 的「Public Access」设置中绑定自定义域名，比如 `cdn.example.com`。
4. 上传图片，得到类似 `https://cdn.example.com/2026/09/example.png` 的 URL。
5. 在博客 Markdown 中直接引用该 URL。

上传方式可以用：

```bash
# rclone 示例
rclone config create r2 s3 provider=Cloudflare \
  access_key_id=<你的AccessKey> \
  secret_access_key=<你的SecretKey> \
  endpoint=https://<account_id>.r2.cloudflarestorage.com

# 同步本地图片文件夹到 R2
rclone sync ./images r2:my-bucket/images
```

如果你习惯使用 PicGo，可以选择“S3”插件，填写：

- Endpoint：`https://<account_id>.r2.cloudflarestorage.com`
- Bucket：`my-bucket`
- Access Key / Secret Key

## 六、实战接入 R2 对象存储

获取`AccessKey`和`SecretKey`

进入cloudflare控制台后找到存储和数据库，R2对象存储，没有存储桶的可以创建一个新的存储桶

![image.png](https://cloudflare-r2.4w.ink/2026/09/09/1788959841458-3776.png)

进入存储桶-设置

![image.png](https://cloudflare-r2.4w.ink/2026/09/09/1788960020043-2941.png)

首先先去绑定自定义域名，虽然R2也提供免费 r2.dev的公共域名。
>但是这个域名有速率限制，不建议用于生产。诸如 Access 和 Caching 等 Cloudflare 功能不可用。

绑定完自定义域名后回到R2对象存储界面，鼠标向下滑动，右侧底部找到管理 API 令牌。

![image.png](https://cloudflare-r2.4w.ink/2026/09/09/1788960244894-962.png)

创建帐户 API 令牌，权限选择对象读和写: 允许读取、写入和列出特定存储桶中的对象，是否指定存储可选，有多个桶的话可以分别用不同的 API 令牌，防止Access Key / Secret Key泄露后，所有桶受到数据泄露风险。

![image.png](https://cloudflare-r2.4w.ink/2026/09/09/1788960364505-5569.png)

设置完成后点击创建令牌，访问密钥 ID是Access Key，机密访问密钥是Secret Key，注意保存好后在点击完成，此信息只会显示一次。

![image.png](https://cloudflare-r2.4w.ink/2026/09/09/1788960561624-7229.png)

对接到S3存储，部分工具里面没有为R2单独增加选项，那么选择S3兼容、MinIO、OSS兼容端点均可，所有需要填写信息君和在上一步页面中获取到。

![image.png](https://cloudflare-r2.4w.ink/2026/09/09/1788960758577-892.png)

## 总结：R2 是真香，但不是真免费

严格来讲，R2 不是无限免费。10GB 存储加每月百万级请求，对普通静态博客来说足够了。但如果你是一个重度图片博主，每天的读请求几十万次，那就要认真评估了。

不过，在“成本低廉、全球加速、无流量费”这个赛道上，Cloudflare R2 确实是把其他图床按在地上摩擦。

如果你也是静态博客玩家，强烈建议试一下。配置一次，终身受益。