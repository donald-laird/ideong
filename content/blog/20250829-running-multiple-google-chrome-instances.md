---
title: "谷歌 Chrome 浏览器多开"
date: 2025-08-29T12:39:20+08:00
description: "简单几步在 windows 和 mac 电脑让谷歌 Chrome 浏览器实现多开独立运行"
cascade:
   type: blog
tags: 
   - 技巧
   - 浏览器
   - VPN
---

某些场景特别是使用数字货币的需要谨慎使用指纹浏览器，或者想自己实现指纹浏览器一样的功能，简单几步在 windows 和 mac 电脑让谷歌 Chrome 浏览器实现多开独立运行，而且还能通过开源软件实现可本地同步操作。

## 使用多个独立 Chrome 实例的好处

```
1.	账号隔离（多个 Google/Facebook/Twitter 账号互不干扰）。
2.	独立 Cookie 和缓存（防止自动登录错账户）。
3.	避免网站指纹追踪（适合营销、电商、广告推广）。
4.	防止崩溃影响所有 Chrome（每个实例独立运行）。
5.	提高工作效率（为不同任务创建不同的 Chrome 配置）。
6.	独立代理（使用不同 VPN 或 IP 访问不同站点）。
7.	适用于 Web3、加密货币交易、Dapp（多个钱包账户隔离）。
```

下面是在不同操作系统的实现方法。

## macOS

Chrome 不支持像 Windows 那样直接创建 `.lnk` 快捷方式，但可以通过 终端命令 +AppleScript 实现多个独立环境的 Chrome 实例。以下是详细步骤：

### 步骤 1：

创建用户数据文件夹，打开终端（快捷键 Command + Space，搜索 `终端` 或 `terminal` ）。根据实际需要，创建用户数据目录：

``` shell
mkdir -p ~/Chrome_Profiles/Profile_1
mkdir -p ~/Chrome_Profiles/Profile_2
mkdir -p ~/Chrome_Profiles/Profile_3
mkdir -p ~/Chrome_Profiles/Profile_4
mkdir -p ~/Chrome_Profiles/Profile_5
mkdir -p ~/Chrome_Profiles/Profile_6
mkdir -p ~/Chrome_Profiles/Profile_7
mkdir -p ~/Chrome_Profiles/Profile_8
mkdir -p ~/Chrome_Profiles/Profile_9
mkdir -p ~/Chrome_Profiles/Profile_10
```

这样会在 `~/Chrome_Profiles` 目录下创建 10 个独立的 Chrome 配置文件夹。

这样会在 ~/Chrome_Profiles 目录下创建 10 个独立的 Chrome 配置文件夹。

### 步骤 2：

创建 Chrome 启动脚本创建文件（在终端中输入 `nano` 或 `vim` 命令如下）：

``` shell
nano ~/chrome_profiles.sh
```

复制并粘贴以下脚本：

``` shell
#!/bin/bash
# macOS Chrome 多开脚本
for i in {1..10}; do
    open -na "Google Chrome" --args --user-data-dir="$HOME/Chrome_Profiles/Profile_$i"
done
```

保存文件

按 Control + X 退出。 按 Y 确认保存。 按 Enter 确认文件名。

赋予执行权限：

``` shell
chmod +x ~/chrome_profiles.sh
```

### 步骤 3：

运行脚本，在终端中输入：

``` shell
~/chrome_profiles.sh
```

回车后，你会看到 10 个独立环境的 Chrome 浏览器实例被打开，每个都使用不同的 Profile 目录。

### 步骤 4：

创建桌面快捷方式 方法 ：使用 Automator finder 程序（即自动操作程序）中点击左侧 `应用程序` 找到 Automator 选择 “应用程序” 类型。

![](images/2025/chrome-multi-browser-opening-1.png)

然后在左侧搜索 Shell 脚本，然后双击它。

在脚本框中输入：

```
~/chrome_profiles.sh
```

![](images/2025/chrome-multi-browser-opening-2.png)

点击 “文件” → “存储”：

选择你想保存的位置，比如 `桌面` 作为存储位置。

按你喜欢的方式取名，如 “Chrome 多开.app”。 文件格式选择 “应用程序”，然后保存。 双击该应用，即可同时打开 10 个独立 Chrome 实例！

## Windows

### 步骤 1：

在d盘（或其他盘）创建两个文件夹：

• **D:\fenliulanqi2\Chrome_UserData** （存放 Chrome 用户数据）

• **D:\fenliulanqi2\Chrome_ShortCuts** （存放快捷方式）

### 步骤 2：

创建 PowerShell 脚本

1. **打开记事本**（快捷键 Win + R，输入 notepad，回车）。
2. **复制以下代码**（已修改成生成 10 个 Chrome 分身）：

``` bash
# -*- coding: utf-8 -*-  
# Title: 自动生成多个具有独立环境的 Chrome 浏览器  
# Describe: d盘新建一个记事本文件，复制以下代码，保存为 chrome.ps1 ，  
# 菜单栏以管理员运行 PowerShell 运行，输入 d: 然后输入Set-ExecutionPolicy RemoteSigned 回车后获取权限输入y，  
# 输入命令 .\chrome.ps1 即可  
# 先建立两个文件夹并复制其路径，替换以下两个路径  

# 存放 Chrome 用户数据
$UserDataPath = "D:\fenliulanqi2\Chrome_UserData"

# 存放快捷方式图标，从这个文件夹里打开浏览器分身
$FilePath = "D:\fenliulanqi2\Chrome_ShortCuts"
  
# 右键打开你桌面上的 Chrome 浏览器快捷方式，复制“目标”一栏的内容，替换下方路径  
# （注意：只复制 C:\Users\....\chrome.exe ，chrome.exe 后面的比如“--profile-directory”等字符不要复制）  
$TargetPath = "C:\Program Files\Google\Chrome\Application\chrome.exe"  
  
# 复制 Chrome 浏览器快捷方式的“起始位置”一栏的内容，替换下方路径  
$WorkingDirectory = "C:\Program Files\Google\Chrome\Application"  
  
# 设置生成分身的数量（从1到10）  
$array = 1..10  
  
foreach ($n in $array)  
{  
    $x = $n.ToString()  
  
    $ShortcutFile = $FilePath + "\Chrome_" + $x + ".lnk" #  
  
    $WScriptShell = New-Object -ComObject WScript.Shell  
  
    $Shortcut = $WScriptShell.CreateShortcut($ShortcutFile)  
  
    $Shortcut.TargetPath = $TargetPath  
  
    $Shortcut.Arguments = "--user-data-dir=" + $UserDataPath + "\" + $x  
  
    $Shortcut.WorkingDirectory = $WorkingDirectory  
  
    $Shortcut.Description = "Chrome" #备注，可以随便写  
  
    $Shortcut.Save()
}
```

3. **保存文件**

• 点击 **文件 → 另存为**。
• **文件名：** chrome.ps1
• **保存类型：** 选择 所有文件 (_._)
• **存放位置：** D:\

### 步骤 3：

运行 PowerShell 脚本

1. **以管理员权限运行 PowerShell**

• **按 Win + X**，选择 **Windows 终端（管理员）** 或 **PowerShell（管理员）**。

2. **进入 D 盘、PowerShell 脚本文件所在目录**

在 PowerShell 中进入到脚本文件所在目录。

3. **允许执行 PowerShell 脚本**

输入：

``` bash
Set-ExecutionPolicy RemoteSigned
```

**如果提示是否更改执行策略，输入 Y 并回车**。

![](images/2025/chrome-multi-browser-opening-3.png)

4. **运行脚本**

输入：

```
.\chrome.ps1
```

**回车运行脚本**，等待完成。

### 步骤 4：

运行 Chrome 分身

打开 D:\fenliulanqi2\Chrome_ShortCuts，你会看到 10 个 Chrome 快捷方式，例如：

• Chrome_1.lnk
• Chrome_2.lnk
• …
• Chrome_10.lnk

**双击任意一个快捷方式，即可打开独立的 Chrome 环境！**

![](images/2025/chrome-multi-browser-opening-4.png)

至此，本地多开浏览器完成。
