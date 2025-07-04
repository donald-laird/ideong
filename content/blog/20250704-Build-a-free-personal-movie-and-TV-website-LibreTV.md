---
title: "搭建免费的个人影视网站-LibreTV"
date: 2025-07-04T15:30:14+08:00
description: "LibreTV 是一款轻量免费的视频搜索平台，整合了多个视频源，支持随时观看，无需注册且免会员限制。这里我们来自行搭建一个这样的网站。"
cascade:
   type: blog
tags: 
   - Github
   - 视频网站
---

> 来源：[【5分钟搭建免费的个人影视网站 | Hans Blog】](https://hansvlss.top/post/libretv/)

本教程将手把手教你如何用5分钟**快速免费搭建属于你自己的影视聚合门户网站！**

**LibreTV** 是一款轻量免费的视频搜索平台，整合了多个视频源，支持随时观看，无需注册且免会员限制。它不承担视频流量，只提供聚合链接，能去广告但画质一般，非常适合临时快速观看电影或电视剧片段，方便又实用。

现在我们就一起部署一个**无需备案、可控、安全、零广告**的影视聚合平台！

在正式部署前，你需要准备以下内容：

| 项目              | 说明                                                                                   |
| --------------- | ------------------------------------------------------------------------------------ |
| ✅ GitHub 账号     | 用于 Fork 项目并自动部署代码（[https://github.com](https://github.com\)/)）                       |
| ✅ Cloudflare 账号 | 用于 Pages 免费部署和绑定自定义域名（[https://dash.cloudflare.com](https://dash.cloudflare.com\)/)） |
| 🌐 自定义域名（可选）    | 如果你希望使用自己的域名，如 `tv.hans.com`，可提前注册好                                                  |

**小贴士：**

- **无需服务器**：所有资源托管在 Cloudflare Pages，完全静态部署，无需服务器或数据库。
- **零费用**：GitHub + Cloudflare 均有免费额度，部署和使用不收取任何费用。
- **无代码基础也能完成**：只需复制粘贴，跟着教程操作即可。

## 第一步：Fork LibreTV 仓库

1. 打开官方仓库：
👉 [https://github.com/LibreSpark/LibreTV](https://github.com/LibreSpark/LibreTV)
2. 点击右上角 `Fork`，→ 将项目复制到你自己的 GitHub 账户
3. 建议保持名称为 `LibreTV`，方便后续自动同步

## 第二步：部署到 Cloudflare Pages

1. 打开 Cloudflare Pages：  
👉 [https://cloudflare.com/](https://cloudflare.com/)
2. 点击「创建项目」→ 选择你刚 Fork 的 `LibreTV` 仓库
3. 设置构建参数（默认即可）
4. 点击「部署」开始构建，几分钟后即可完成部署

## 第三步：设置密码保护（可选）

保护你的站点隐私，设置两个环境变量即可：

在 `Cloudflare Pages` 设置：

防止陌生人访问你的站点，可设置访问密码：

1. 进入 Cloudflare Pages → 找到你的项目 → 打开左侧「Settings」→「Environment Variables」
2. 添加以下两个环境变量：

| 变量名             | 说明                        |
| --------------- | ------------------------- |
| `PASSWORD`      | 用户访问网站时输入的密码（例如 `123456`） |
| `ADMINPASSWORD` | 管理员后台设置密码（右上角齿轮入口）        |
## 第四步：访问你的 LibreTV

访问地址类似于：`https://libretv.pages.dev`

你现在已经成功搭建一个 IPTV/影视门户网站！

## 第五步：接入第三方影视 API 资源接口(可选)

LibreTV 开放接入其它影视数据源，你需要手动添加。

1. 打开你的网站 → 点击右上角齿轮图标（设置）
2. 找到「自定义接口」功能 → 填入可用的影视接口地址

推荐影视接口整理：

|名称|地址|
|---|---|
|影视工厂|`https://cj.lziapi.com/api.php/provide/vod/`|
|七七资源|`https://www.qiqidys.com/api.php/provide/vod/`|
|饭团影视|`https://www.fantuan.tv/api.php/provide/vod/`|
## 第六步：每天自动同步官方更新（可选）

新建一个 GitHub Actions 文件 `.github/workflows/sync-upstream.yml`：

``` yaml
name: LibreTV Sync Upstream  

on:  
  schedule:  
    - cron: "0 3 * * *"  
  workflow_dispatch:  

permissions:  
  contents: write  

jobs:  
  sync:  
    runs-on: ubuntu-latest  
    steps:  
      - uses: actions/checkout@v4  
        with:  
          fetch-depth: 0  
      - name: Add upstream  
        run: |  
          git remote add upstream https://github.com/LibreSpark/LibreTV.git  
          git fetch upstream  
          git merge upstream/main --allow-unrelated-histories -m "🔄 Sync from upstream"  
          git push origin main
```

部署后每天凌晨自动更新官方代码，无需手动操作。

## 第七步：绑定自定义域名（可选但推荐）

将你的站点绑定到自己的域名，如 `tv.hans.com`，更专业好记！

🔧 步骤如下：

1. 打开 你的项目 → 点击「自定义域」
2. 输入托管到CloudFlare域名」绑定即可

通过本教程，你已经掌握了使用 Cloudflare Pages 快速部署 LibreTV 的方法。后续也可以尝试添加更多 API 接口，实现更强大的聚合播放体验。
