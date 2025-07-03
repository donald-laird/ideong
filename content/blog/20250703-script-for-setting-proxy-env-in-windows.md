---
title: "在 windows 设置 PROXY 环境变量的执行脚本"
date: 2025-07-03T17:34:35+08:00
author: Donald
description: 在 Powershell 执行一些**开发者工具 (Git, npm, pip, curl)** 的命令，但由于国内网络 GFW 的原因网络访问不了的解决办法
cascade:
   type: blog
tags: 
   - 代理
   - PowerShell
   - Windows
---

经常需要在 Powershell 执行一些**开发者工具 (Git, npm, pip, curl)** 的命令，但由于国内网络 GFW 的原因往往很多开发者网络访问不了，可以通过下面方法来解决：

## 临时命令

``` shell
$env:HTTP_PROXY="http://127.0.0.1:10808"
$env:HTTPS_PROXY="http://127.0.0.1:10808"
```
注：端口号请以实际代理端口号填入，我的环境是 10808。

## 通过执行脚本

也可以通过脚本方式来解决，更加方便。在 PowerShell 的用户目录下，我喜欢在当前用户目录下进行。通过记事本或者 IDE 工具，新建一个后缀为 `.ps1` 的脚本文件，比如我是保存为 `setProxy.ps1`，将下面脚本复制进去。

``` shell
# --- 配置你的代理信息 ---
# 注意：如果代理需要用户名和密码，格式为: "http://username:password@your_proxy_server:port"
$httpProxy = "http://127.0.0.1:10808"
$httpsProxy = "http://127.0.0.1:10808" # 如果 HTTPS 代理和 HTTP 相同，则使用相同的值
$noProxy = "localhost,127.0.0.1,*.baidu.com" # 不需要代理的地址，用逗号隔开

# --- 永久设置环境变量(为当前用户) ---
[System.Environment]::SetEnvironmentVariable("HTTP_PROXY", $httpProxy, "User")
[System.Environment]::SetEnvironmentVariable("HTTPS_PROXY", $httpsProxy, "User")
[System.Environment]::SetEnvironmentVariable("NO_PROXY", $noProxy, "User")

Write-Host "代理相关的环境变量已为当前用户永久设置。" -ForegroundColor Green

Write-Host "重要提示：你必须重启 PowerShell 窗口才能使这些新设置生效！" -ForegroundColor Yellow
```

保存后在命令终端执行 `.\setProxy.ps1` 就可以了。

![](/images/2025/250703-setProxy.png)

以后要移除它们时，只需将变量的值设置为空字符串即可。

## PowerShell 输出中文乱码问题

如果你输出显示为乱码，主要是 PowerShell 处理中文字符时最常见的一个“坑”。问题出在 **PowerShell 读取文件时使用的解码方式** 上。可以按下面方法解决输出有中文的问题。

### 仅解决当前文件

**如何操作 (以 VS Code 为例):**

1. 用 VS Code 打开您的 setProxy.ps1 文件。
2. 看编辑器右下角的状态栏，您会看到 "UTF-8" 字样。
3. **点击这个 "UTF-8"**。
4. 在顶部弹出的菜单中，选择 **"通过编码保存 (Save with Encoding)"**。
5. 在接下来的列表中，选择 **"UTF-8 with BOM"**。
6. 保存文件。

**如何操作 (以 Windows 记事本为例):**

1. 用记事本打开您的 .ps1 文件。
2. 选择“文件” -> “另存为”。
3. 在弹出的对话框底部，将“编码”选项从 "UTF-8" 改为 **"UTF-8 (带 BOM)"**。
4. 保存并覆盖原文件。

完成以上操作后，**不需要做任何其他改动**，直接在 PowerShell 中再次运行 .\setProxy.ps1，输出就会完全正常了。

### 系统解决 PowerShell Profile

如果你不想每次都手动保存为 "UTF-8 with BOM"，可以修改 PowerShell 的配置文件，让它在执行命令时默认使用 UTF-8 编码。

1. 在 PowerShell 中运行以下命令，打开你的配置文件（如果文件不存在，它会自动创建）：
``` shell
notepad $PROFILE
```

2. 在打开的记事本文件中，添加下面这行代码：
``` shell
$PSDefaultParameterValues['*:Encoding'] = 'utf8'
```

3. 保存并关闭记事本。
4. **关闭并重新打开 PowerShell 窗口**，让配置生效。
