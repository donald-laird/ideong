---
title: "3x-ui 安装和配置 ip 限制"
date: 2025-08-12T22:30:30+08:00
description: "通过在 3x-ui 安装 fail2ban 并进行正确设置后，便可进行入站用户的数量和 ip 限制等。"
cascade:
   type: blog
tags: 
    - 3x-ui
    - 用户限制
    - fail2ban
    - VPN
---

想要在 3x-ui 面板上设置 VPS 的用户限制（IP 限制），需要先安装 fail2ban 服务。

## 安装 fail2ban

在终端的 root 状态下，运行 `x-ui`，选择 `20. IP Limit Management`，进入菜单再选 `1`，安装 fail2ban 服务。

![](/images/2025/3x-ui-ip-limit-fail2ban-1.png)

![](/images/2025/3x-ui-ip-limit-fail2ban-2.png)

最后显示安装成功。

![](/images/2025/3x-ui-ip-limit-fail2ban-3.png)

## 3x-ui 日志设置

在 3x-ui 面板的 Xray Configs 配置中，基础设置-日志设置的下面找到 `Access Log` 日志入口项，选择里面的 `./access.log`，然后点上面的保存，再点重启 Xray 服务。

![](/images/2025/3x-ui-ip-limit-log.png)
## 禁用 fail2ban 对 `sshd` 监控规则

`fail2ban` 是一个独立的服务，它可以监控任何应用的日志文件。默认情况下，当我们安装 `fail2ban` 后，它会自带一些预设的监控规则，其中最常见的就是 `sshd`（SSH服务），用来防止SSH暴力破解。

因为我们主要是用于 3x-ui 的 IP 监控，而不是保护 `sshd` ，所以最简单的是直接禁用它。不然当 `fail2ban` 在启动时，首先尝试加载这个默认的 `sshd` 监控规则。而我们没在系统上配置 `sshd` 的日志文件导致 `fail2ban` 服务就启动失败。

现在我们配置一下禁用。

**步骤 1: 创建本地配置文件以禁用 sshd jail**

`fail2ban` 的最佳实践是不要修改主配置文件 (`jail.conf`)，而是创建一个 `.local` 文件来覆盖它。

1. 打开或创建一个新的本地配置文件 `jail.local`：
```
vim /etc/fail2ban/jail.local
```

2. 在该文件中，添加以下内容，明确告诉 `fail2ban` 禁用 `sshd` jail：
```
[sshd]
enabled = false
```

**注意：** 如果你的 `jail.local` 文件已经存在并且里面有 `[sshd]` 部分，只需确保 `enabled = false` 即可。如果文件是空的，直接粘贴上面两行。

3. 保存文件并退出编辑器 (在 `vim` 中，按 `:w!`，然后按 `Enter`)。

**步骤 2: 重启并检查 Fail2Ban 状态**

现在 `sshd` jail 已经被禁用，`fail2ban` 应该可以顺利启动了。

重启 `fail2ban` 服务：
```
systemctl restart fail2ban
```

再次进入 `x-ui` 命令，进入菜单 `20. IP Limit Management`，再执行菜单 `8. Service Status`，应该就能看到 `fail2ban` 运行状态正常。

![](/images/2025/3x-ui-ip-limit-fail2ban-4.png)

## 3x-ui 面板设置 

回到 3x-ui 面板中，在 `inbound` 里就可以在每一个想要设置的用户端上进入编辑进行设置。

![](/images/2025/3x-ui-ip-limit-setting.png)

默认为 `0` 表示不生效。自己可以填上想要进行的限制，在下面就有个 IP 列表。将来登录进来的 IP 就会出现。
