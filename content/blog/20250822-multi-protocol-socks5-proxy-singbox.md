---
title: "一键实现多协议转Socks5代理方案2：跨境电商多账号独立IP方案"
date: 2025-08-22T22:58:00+08:00
description: "告别端口冲突！本次以 sing-box 配置为例，将 SS/Vmess/Vless 等任意节点转化为独立 Socks5 通道，解决指纹浏览器多账号IP隔离难题"
cascade:
   type: blog
tags: 
   - VPN
---

许多跨境电商运营人员或开发者拥有多个代理节点，但常用的代理客户端（如 Clash, v2RayN）默认只允许一个全局出口。这意味着无法简单地在本地电脑为不同的应用程序或浏览器实例，特别是使用指纹浏览器等，分配不同地区或者不同节点的 Socks 出口 IP，让每个浏览器实例或者 ip 各不相同。之前我介绍过 [【一键实现多协议转Socks5代理】](https://post.ideong.top/blog/20250613-multi-protocol-socks5-proxy/)，在那里是生成 Clash 配置文件，这次我们来生成 `sing-box` 内核配置。虽然 `sing-box` 内核支持通过复杂的配置实现“多端口分流”，但手动编写 JSON 配置文件对非技术用户门槛高、易出错。

开源 Github 项目[【SingMP-Gen】](https://github.com/donald-laird/SingMP-Gen)，本工具通过一个直观的图形化界面，将这个复杂的过程自动化，让任何人都能在几分钟内创建出稳定、防泄漏的多端口代理环境配置。而且这个代理环境还可以让本地局域网的其它电脑分享使用。

## ✨ 核心功能

- **轻松导入节点**: 支持直接粘贴 `outbounds` 数组或上传完整的配置文件。
- **智能端口分配**: 可设置起始端口号，为所有节点自动分配递增的端口，默认为 `50000` 开始，并支持手动修改默认端口号。
- **一键生成配置**: 根据用户设置，快速生成一份完整、可用、防 DNS 泄漏的 `sing-box` 配置文件。

## 🚀 如何使用

最简单的办法就是直接在 [Demo](https://singmp.hotrue.cc/) 网站上使用，因为这个工具仅通过**前端**原生 HTML, JavaScript (ESM) 实现，并不保存数据。所有操作均在您的浏览器本地完成，无需后端服务器，确保节点信息的绝对安全和隐私。

### 部署

**方式一：**
当然可以直接 Fork 项目，再部署到自己的 Cloudflare Pages 或者 Github Pages 上。

**方式二：**
如果是想在自己电脑本地使用，由于浏览器安全策略的限制，本项目不能直接通过 `file://` 协议打开 `index.html` 文件运行。您需要通过一个本地 Web 服务器来访问它，可以**使用 Python**。

1. 确保您的电脑已安装 Python。
2. 将本项目的所有文件 (`index.html`, `main.js`, `en.yml`, `zh.yml`, `sing-box-template.json.tpl`) 放置在同一个目录下。
3. 打开终端或命令行工具，使用 `cd` 命令进入该目录。
4. 运行以下命令启动一个简单的 Web 服务器：
```
# 如果您使用 Python 3
python -m http.server 8000

# 如果您使用 Python 2
python -m SimpleHTTPServer 8000
```
5. 打开您的浏览器，访问 `http://localhost:8000`。

现在，您应该可以看到工具界面并开始使用了，如图：

![](/images/2025/Sing-Box-Multi-port-Config-Generator-01.png)

### 使用方法

打开工具网页，如上图。

1. 粘贴 sing-box 订阅链接或者直接粘贴 sing-box 格式的节点信息。

> [!note]
> 目前我觉得最好用的订阅节点管理和转换工具可以推荐 [Sub-Store](https://github.com/sub-store-org/Sub-Store)，可以参考[我这篇文章](https://post.ideong.top/blog/20250816-deploy-vps-node-subscription-tool-sub-store/)配置使用。

2. 设置起始端口，默认从 50000 开始。
3. 设置默认节点，可以按默认，或者根据自己需要设置一个默认节点。比如我们使用 v2rayN 最常用 10808 作本地监听默认端口。
4. 最后生成配置后，直接复制或者下载 `config.json` 到本地，导入客户端便可使用。

![](/images/2025/Sing-Box-Multi-port-Config-Generator-02.png)

5. 导入客户端，以 v2rayN 为例，通过 `配置文件` - `添加自定义配置文件` 导入配置

![](/images/2025/Sing-Box-Multi-port-Config-Generator-03.png)

6. 配置文件可以按需起个别名，`Core 类型` 那里需要选 `sing_box`。最后确定，再在主界面选择添加的这个配置激活就可以了。

![](/images/2025/Sing-Box-Multi-port-Config-Generator-04.png)