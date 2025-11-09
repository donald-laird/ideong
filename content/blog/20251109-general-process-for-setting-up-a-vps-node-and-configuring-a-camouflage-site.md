---
title: "通用的搭建 VPS 节点并配置伪装站流程"
date: 2025-11-09T23:11:15+08:00
description: "在部署 VPN 或代理服务器时，3x-ui 面板（基于 Xray 的管理工具）是一个流行选择。为了安全和隐藏面板 URL，我们通常使用 Nginx 作为反向代理，配置伪装站。本文我总结分享一下配置关键点、常见问题（如白屏、证书设置失败）和解决方法。"
cascade:
   type: blog
tags: 
   - VPS
   - 伪装站
   - VPN
   - 3X-UI
---

在部署 VPN 或代理服务器时，3x-ui 面板（基于 Xray 的管理工具）是一个流行选择。为了安全和隐藏面板 URL，我们通常使用 Nginx 作为反向代理，配置伪装站。本文我总结分享一下配置关键点、常见问题（如白屏、证书设置失败）和解决方法。

案例中，我使用的是 Ubuntu 系统，域名是托管在 Cloudflare 中，此时未开启 Proxy/CDN 状态（未开启小黄云）。
域名 `aaa.bbb.com`
面板端口 `41580`
面板路径basePath `/panel-sample/`

## VPS 安装依赖和软件

### 安装 3X-UI

``` bash
# 更新软件源
apt update
```

大部分 VPS 的 Ubuntu 版本中默认没安装 Cron，而后面申请证书时需要用到。先安装下 Cron，以免通过 `x-ui` 命令安装 SSL 证书时需要异常退出。

``` bash
# 安装 Cron
apt install cron
```

```
# 安装 3X-UI 面板
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```

### 在 x-ui 中启用 BBR  TCP 拥塞控制算法和安装证书

启用 BBR TCP 拥塞控制算法：
- `x-ui` -> `23` （`23. Enable BBR`）-> `1`（`1. Enable BBR`）

安装证书：
- `x-ui` -> `18` （`18. SSL Certificate Management`）-> `1`（`1. Get SSL`）

## 配置节点信息

因为我们的目标是充分利用 Nginx 和 Cloudflare CDN，实现路径分流和 IP 隐藏。所以我们在配置节点的协议上推荐两种配置：将节点配置为 **VLESS + WebSocket + TLS**

### VLESS + WebSocket + TLS

- **伪装域名/Host**: `aaa.bbb.com`
- **路径/Path**: `/015dd0b3-6c0b-4e07-8eb7-ac33c961cec4` (可以选取用户 id 的一段，或者整个用户 id，后续 Nginx 分流路径要与此配置一致)

**端口**按随机生成即可。

在不使用 Nginx 的情况下，可以使用 Reality 协议的传输层流量，因为 Reality 的流量**不会**经过我们的 Nginx 配置，也**不能**享受 Nginx 的路径分流功能。如果想使用 Reality，那只能将本地节点配置的地址使用 IP，而不能使用域名了。

> [!note]
> **Reality 协议的工作原理**：它是一种**直连协议**。客户端直接与服务器上 Xray 正在监听的端口（例如 `46074`）进行通信，**它不经过 Nginx，也不使用 WebSocket**。Reality 协议自身就包含了 TLS 加密和伪装的功能。

将节点信息导出到 v2rayN 本地客户端设置是否成功。未开启小黄云时，端口可以按随机生成的。如果开启了小黄云，则本地端口设置需要改为 `443`。

节点测试成功后。下面开始安装 Nginx。

**注意：** 在成功配置好 Nginx 分流后，需要回来 X-UI 这个节点的 Inbound 设置里，将底层传输安全 TLS 关闭！使之成为**VLESS + WebSocket**，原因在后面解释。

在后面安装完 Nginx 后，注意配置文件中 `server` 段落的 `location` 模板。

### VLESS + gRPC + TLS

**gRPC** 是由 **Google** 基于先进的 **HTTP/2** 协议开发的一个现代、高性能的**远程过程调用（RPC）框架**。在我们的使用场景中，它是 `WebSocket` 的一个更现代、性能更强的替代品。

> [!important]
> 使用 gRPC 传输协议需要确保托管域名的 Cloudflare 里的网络设置那里下打开了 gRPC 连接。因为它默认并未打开。

- **服务名称/Service Name**: `aaa.bbb.com` 后续 Nginx 分流路径要与此配置一致。
- **权限/Authority**: 可以默认保留为空。

**端口**按随机生成即可，但在后续设置 Nginx 的 `location` 里面的转发时记得保持一致。

虽然上面设置中将 `Service Name` 设置为域名，但不要混淆了他们是两个完全不同的概念：

- **Nginx 的 `location` 路径**：是 URL 的一部分，用于**路由**，告诉 Nginx 应该由哪个规则来处理这个请求。
- **gRPC 的 `Service Name`**：是 gRPC 协议**内部**的一个参数，用于标识具体的服务，它与 URL 路径无关。

将节点信息导出到 v2rayN 本地客户端设置是否成功。未开启小黄云时，端口可以按随机生成的。如果开启了小黄云，则本地端口设置需要改为 `443`。

**注意：** 在成功配置好 Nginx 分流后，需要回来 X-UI 这个节点的 Inbound 设置里，将底层传输安全 TLS 关闭！使之成为**VLESS + gRPC**，原因在后面解释。

节点测试成功，在后面安装完 Nginx 后，注意配置文件中 `server` 段落的 `location` 模板。

### 两者比较

- **如果追求极致的稳定和最广泛的兼容性**：`VLESS + WebSocket + TLS` 依然是无可挑剔的王者。
- **如果您希望探索更高的性能和更好的伪装性**：`VLESS + gRPC + TLS` 代表了更现代的技术方向。

| 特性          | VLESS + WebSocket + TLS      | VLESS + gRPC + TLS                      |
| ----------- | ---------------------------- | --------------------------------------- |
| **底层协议**    | HTTP/1.1                     | HTTP/2                                  |
| **性能效率**    | **良好**。成熟稳定，足以满足绝大多数需求。      | **优秀**。延迟更低，尤其在高并发下优势明显。                |
| **伪装效果**    | **优秀**。WebSocket 流量在网络中非常普遍。 | **极佳**。流量特征与现代网站的 API 通信无异。             |
| **CDN 兼容性** | **完美**。所有 CDN 都支持 WebSocket。 | **完美**。Cloudflare 等主流 CDN 均已支持 gRPC。    |
| **配置复杂度**   | **较低**。配置直观，是目前最主流的方案。       | **中等**。需要额外配置 ServiceName，Nginx 配置稍有不同。 |
| **客户端支持**   | **非常广泛**。几乎所有客户端都支持。         | **广泛**。绝大多数主流客户端都已支持。                   |

## 安装并配置 Nginx

### 安装 Nginx
```bash
# 安装 Nginx
apt install nginx

# 启动 Nginx
systemctl start nginx

# 设置 Nginx 开机自启动
systemctl enable nginx

# 检查 Nginx 运行状态
systemctl status nginx
```

### 安装 Nginx 证书

由于已经在安装 3X-UI 中已经申请了域名证书，将已申请的证书安装到 Nginx 有权访问的标准目录 `/etc/nginx/ssl/` 下，注意替换域名到实际域名。
```
# 创建证书目录
mkdir -p /etc/nginx/ssl/aaa.bbb.com

# 安装证书到这个目录
acme.sh --install-cert -d aaa.bbb.com \
--key-file       /etc/nginx/ssl/aaa.bbb.com/privkey.pem \
--fullchain-file /etc/nginx/ssl/aaa.bbb.com/fullchain.pem \
--reloadcmd     "sudo systemctl force-reload nginx"
```

可能需要确保 Nginx 的工作进程（`www-data` 用户）有读取 SSL 私钥证书（`privkey.pem`）权限，可以按照下面方法解决此问题。

我们将为证书目录和文件设置最安全、最规范的权限，确保 Nginx 能够正常读取。

**1. 将证书目录的所有者递归地更改为 `root`**： 

证书文件由 `root` 用户拥有是更安全的做法。
```
chown -R root:root /etc/nginx/ssl/
```

**2. 为目录和文件设置正确的读写权限**：

- `/etc/nginx/ssl` 目录及其子目录需要 `755` 权限，以便其他用户（如 `www-data`）可以进入并读取。
- 公开的证书文件 (`fullchain.pem`) 需要 `644` 权限。
- **私钥文件 (`privkey.pem`) 必须是 `640` 或 `600` 权限**，并且其所属组需要是 `www-data`，或者让所有用户都能读（虽然不太安全，但能保证 Nginx 读到）。最安全的方式是让 `root` 用户拥有，但让 `www-data` 用户能读。

- **确认 Nginx 工作进程的用户**：
```
ps aux | grep nginx
```
_(我们期望看到用户 `www-data`)_

- **检查证书文件的当前权限和所有者**：
```
ls -l /etc/nginx/ssl/aaa.bbb.com/
```
_(我们需要查看文件的读写权限和所有者)_

- **模拟 Nginx 工作进程读取证书（最关键的测试）**： 这条命令会尝试以 `www-data` 用户的身份去读取证书和私钥文件。如果它返回 “Permission denied”，我们就找到了问题的直接证据。
```
sudo -u www-data head -n 1 /etc/nginx/ssl/aaa.bbb.com/fullchain.pem && echo "证书文件读取成功"
sudo -u www-data head -n 1 /etc/nginx/ssl/aaa.bbb.com/privkey.pem && echo "私钥文件读取成功"
```
按实际测试得来的经验，大概率是私钥文件读取不成功的。

所以按下面操作获取权限：
```
# 允许 www-data 用户读取私钥，基本上这两个命令就可以了
sudo chgrp www-data /etc/nginx/ssl/aaa.bbb.com/privkey.pem
sudo chmod 640 /etc/nginx/ssl/aaa.bbb.com/privkey.pem

# 设置其他文件和目录的权限，下面这几个权限可以不用，经过上面设置基本就满足下面状态
sudo chmod 755 /etc/nginx/ssl
sudo chmod 755 /etc/nginx/ssl/aaa.bbb.com
sudo chmod 644 /etc/nginx/ssl/aaa.bbb.com/fullchain.pem
```

**3. 应用了新的权限后，请彻底重启 Nginx**

```
sudo systemctl restart nginx
```

### 配置 Nginx

建议将配置分为两部分，主配置文件将作为 Nginx 的框架，**它本身不包含任何网站信息**。
主配置文件的路径：`/etc/nginx/nginx.conf`

```conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 1024;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    # Gzip Settings
    gzip on;

    # Log Settings
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ##
    # Virtual Host Configs
    # 这一行是核心，它会告诉 Nginx 去加载下面我们创建的网站配置文件
    ##
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

自己 VPS 节点网站的配置文件，这个文件里**只包含我们网站的所有配置**。
自己 VPS 节点的配置文件路径：`/etc/nginx/sites-enabled/my-site.conf`

``` conf
server {
    listen 80;
    server_name aaa.bbb.com;  # 这里是自己域名
    
    location /.well-known/ {
           root /var/www/html;
    }
    location / {
            rewrite ^(.*)$ https://$host$1 permanent;
    }
}

server {
    listen 443 ssl http2;
    server_name aaa.bbb.com;  # 这里是自己域名
    
    # --- 确保使用的是这个正确的证书路径 ---
    ssl_certificate      /etc/nginx/ssl/aaa.bbb.com/fullchain.pem; # 注意替换域名
    ssl_certificate_key  /etc/nginx/ssl/aaa.bbb.com/privkey.pem;   # 注意替换域名
    
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;
    ssl_session_tickets off;
    ssl_protocols    TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    # --- 伪装站点 ---
    location / {
        proxy_pass https://fakeweb.com/; #伪装网址
        proxy_redirect off;
        proxy_ssl_server_name on;
        sub_filter_once off;
        sub_filter "fakeweb.com" $server_name;
        proxy_set_header Host "fakeweb.com";
        proxy_set_header Referer $http_referer;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header User-Agent $http_user_agent;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header Accept-Encoding "";
        proxy_set_header Accept-Language "zh-CN";
    }

    # --- Xray WebSocket 节点分流路径模板 ---
    location /ac33c961cec4 {  # 注意路径后无斜杠`/`
        proxy_redirect off;
        proxy_pass http://127.0.0.1:21052; #Xray Inbound 端口，注意这里不需要 Https
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
    
    # --- Xray gRPC 节点分流路径模板 ---
    location /aaa.bbb.com {     # 注意这里对应 Service Name
        if ($content_type !~ "application/grpc") {
            return 404;
        }
        client_max_body_size 0;
        client_body_timeout 1071906480m;
        grpc_read_timeout 1071906480m;
        grpc_pass grpc://127.0.0.1:55723;   # 注意对应端口号
    }

    # --- 3x-ui 面板分流路径 ---
    location /panel-sample/ {  # 面板路径与 x-ui 配置一致，注意路径后需要有斜杠`/`
        proxy_redirect off;
        proxy_pass https://127.0.0.1:41580; #注意如果面板已启用证书，则需要 `https`
        proxy_ssl_verify off; # 如果是 `https`，则忽略上游证书验证，如果非证书，这句删除
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

配置完成后，通过下面命令检查并重启 Nginx。
``` bash
# 再次运行语法检查 
nginx -t 

# 如果成功，请彻底重启 Nginx 服务来应用全新的配置结构
systemctl restart nginx
```

仅重启配置，可以用
```
systemctl reload nginx
```

此时，访问 3X-UI 面板，就可以通过下面网址正常登录了：
`https://aaa.bbb.com/panel-sample/`

**注意：** 在成功配置好 Nginx 分流后，需要回去 X-UI 对应节点的 Inbound 设置里，将安全 TLS 关闭！使之成为**VLESS + WebSocket**。**VLESS + gRPC** 同理需要回去关闭传输安全 TLS。

> [!note]
> **为什么？** 因为我们配置的 Nginx 代理模式是： `客户端 --(HTTPS)--> Nginx --(普通 HTTP)--> Xray` TLS 加密/解密的工作已经完全由 Nginx 负责了。如果 Xray 自己也开启了 TLS，那么当 Nginx 发送一个普通的 HTTP 请求给它时，Xray 会因为收到了一个它不期望的协议而返回一个错误响应，从而导致 Nginx 报 `400` 错误。

## 开启 Cloudflare 的代理加速

至此，就可以回到托管域名的 Cloudflare，将域名的小黄云开启。同时，回到本地节点配置那里进行编辑，将节点端口修改为 `443`。

这样就可以后续使用优选 IP 的方式，进行更多节点的玩法。同时，节点，又非常安全，伪装度又非常高。