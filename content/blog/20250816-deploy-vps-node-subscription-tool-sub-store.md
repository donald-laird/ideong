---
title: "部署节点订阅工具 sub-store"
date: 2025-08-16T18:07:24+08:00
description: "通过部署免费的节点订阅工具，方便管理及分享多种方式的节点，包括自定义节点和机场节点。"
cascade:
   type: blog
tags: 
   - VPS
   - VPN
   - 订阅
---


## sub-store 是什么

[Sub-Store](https://github.com/sub-store-org/Sub-Store) 是一个强大的订阅管理工具，可以用于过滤、合并、重写订阅节点。

该工具支持四大功能：

- 纯节点信息之间的转换（不包含 clash 的分流规则、配置等），将同一个节点的信息，转换为不同的代理内核支持的格式，可以方便的导入使用。
- 订阅格式的整理（正则表达式过滤、脚本过滤），过滤无效/特定地区/倍率的节点、重命名节点名称等。
- 整理订阅网址中的节点信息，可以过滤、重命名、合并等。将多个机场的节点整合到一个订阅链接、提取多个机场中的特定信息的节点等。
- 将使用的各种数据同步到其他地方：网盘、gitlist等。

该工具分为前后端，前端界面如下，主要是提供可视化界面。后端一般搭建在公网服务器上，在前端中输入后端的 api 链接，即可对接到后端，组成完整功能的 sub-store。

![](/images/2025/sub-store-01.png)

任何人都可以自建前后端，网络上也有免费的站点，但考虑到节点信息都储存在别人的服务器上，等同于暴露了自己所有的节点给别人，所以自建的节点都不会使用公共提供的后端。后端的样式如下图。

![](/images/2025/sub-store-02.png)

## 部署 sub-store

### VPS 非 Docker 部署

可以参考【[v2rayssr：sub-store 服务宝塔面板搭建](https://v2rayssr.com/sub-store.html)】

### VPS Docker 部署

可以参考【[IOS 软件部署-VPS Docker 部署](https://wiki.repcz.link/substore/install/#ios)】

### Claw Cloud Run App Docker 部署

[Claw Cloud Run](https://link.vanke.uk/clawcloudrun) 是阿里云青春版 Claw Cloud 旗下，类似于 Vercel、Netlify 的在线开发平台，可以快速部署 Alist、Dify、frp 等程序，目前只需要绑定一个注册超过 180 天的 GitHub 账号，即可永久免费赠送 5 刀/月额度，可在每个可用区使用最多 4 vCPU、8 GiB 内存、10 GiB 硬盘资源，共提供 10GB 流量。

我觉得这种部署是最轻松的。但是如果要使用自定义域名的话，貌似无法成功，我测试过经过 2 天了后台还一直在 Pending。

进入 Claw Cloud Run，打开 App Launchpad，点右上角的 Create App。

`Image Name` 填入镜像 `xream/sub-store:http-meta`
服务器配置选最低的 CPU 0.2，Memory 256M 基本能满足要求。

![](/images/2025/sub-store-clawcloudrun-01.png)

端口设置为 `3001`

![](/images/2025/sub-store-clawcloudrun-02.png)

环境变量 `Environment Variables` 填入下面内容
```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TIME_ZONE=Asia/Shanghai
SUB_STORE_FRONTEND_BACKEND_PATH=/【自己定义的密钥】
SUB_STORE_BACKEND_SYNC_CRON=0 0 * * *
SUB_STORE_BACKEND_UPLOAD_CRON=0 1 * * *
```

![](/images/2025/sub-store-clawcloudrun-03.png)

## sub-store 的使用

sub-store 只支持**不包含 clash 的分流规则**的纯节点信息转换，但 sub-store 提供了**文件管理**功能。结合 clash.meta 的 `proxy-providers` 功能，可以完美提供节点信息向 clash 配置文件的订阅转换。下面分两种使用方法做介绍，具体也可以看油管视频【[最强节点/订阅管理工具 sub-store 节点/订阅聚合 全平台客户端订阅生成 sub-store使用教程](https://youtu.be/JEWeApalKow?si=NuZXmwQ-5o3qrePT)】

### 1. 节点信息订阅管理

Sub-store 支持将节点信息转换为其他代理软件支持的格式，用于在不同的软件总分享。

点击左边的 `+` 号添加，就可以看到如下界面，单条订阅和组合订阅的区别是针对于订阅链接是否是单个而言的，一个订阅中可以包含有多条节点信息，可以在单条订阅中添加多个节点。

![](/images/2025/sub-store-03.png)

填入时选择本地订阅，如果是机场的链接，就选择远程订阅。

![](/images/2025/sub-store-04.png)

导入后，就可以选择分享链接，

![](/images/2025/sub-store-05.png)

或者直接点击进行预览。

![](/images/2025/sub-store-06.png)

### 2. 文件管理

可以利用这个功能，导出适合 Clash 和 Sing-box 等客户端的配置文件。

- 将 clash 模板文件存放在 sub-store 中，并填入 sub-store 的节点分享链接。以文件的形式分享 clash 订阅。
- 将 sing-box 模板文件存放在 sub-store 中，并使用 sing-box 转换脚本，自动在订阅时将节点信息转换到配置中。

具体可以看上面说的油管视频【[最强节点/订阅管理工具 sub-store 节点/订阅聚合 全平台客户端订阅生成 sub-store使用教程](https://youtu.be/JEWeApalKow?si=NuZXmwQ-5o3qrePT)】

至此，关于 sub-store 的功能就简单介绍到这里。